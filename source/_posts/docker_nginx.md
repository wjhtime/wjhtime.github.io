---
title: docker实现nginx负载均衡
date: 2018-05-10
tags: docker
categories: docker
---



linux环境下docker实现nginx负载均衡环境搭建



## 思路

启动1个php容器，处理php请求

启动3个nginx容器，nginx1，nginx2，nginx_main，1和2是应用服务器，main是反向代理服务器

所有对nginx_main的请求都会分布到nginx1和nginx2上面

最后通过观察nginx1，2的访问日志可以看到效果

## 目录说明

```
/var/www/html/laravel	主机的网站地址
/usr/share/nginx/html	容器内的网站地址
/root/docker/nginx/conf.d	主机的nginx配置目录
/etc/nginx/conf.d	容器内的nginx配置目录
/root/docker/nginx/nginx.conf	主机的nginx配置文件
/etc/nginx/nginx.conf	容器内的nginx配置文件
/root/docker/nginx/nginx_main.conf	主机的nginx代理服务器配置文件
```

## 步骤

1.启动php服务器


```shell
# 拉取镜像
docker pull php:7.0-fpm

# 启动容器
docker run --name php -d -p 9001:9000 \
-v /var/www/html/laravel:/usr/share/nginx/html php:7.0-fpm

#安装pdo_mysql扩展
docker-php-ext-install pdo_mysql
```



2.启动nginx1，2，关联php容器

```shell
# 拉取nginx镜像
docker pull nginx

# 启动nginx1容器
docker run --name nginx1 -d -p 81:80 \
-v /root/docker/nginx/conf.d:/etc/nginx/conf.d \
-v /root/docker/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /var/www/html/laravel:/usr/share/nginx/html --link php:php nginx

# 启动nginx2容器
docker run --name nginx2 -d -p 82:80 \
-v /root/docker/nginx/conf.d:/etc/nginx/conf.d \
-v /root/docker/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /var/www/html/laravel:/usr/share/nginx/html --link php:php nginx
```



nginx.conf配置

```nginx
user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer"'
        '"$http_user_agent" "$http_x_forwarded_for"'
    access_log /var/log/nginx/access.log main;
    sendfile on;
    keepalive_timeout 65;
    include /etc/nginx/conf.d/*.conf;
}
```



default.conf配置

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html/public;
    index index.php index.html index.htm;
    
    location ~* \.(gif|jpg|png|css|js|flv|swf|ico|woff|ttf|woff2)(.*)$ {
        access_log off;
        expires 30d;
    }
    
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
        # 连接php容器的9000端口
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_read_timeout 600;
        include fastcgi_params;
    }
}
# fastcgi_pass里的php为php容器
```



3.启动nginx_main反向代理服务器，关联nginx1和nginx2

```shell
docker run --name nginx_main -d -p 83:80 \
-v /root/docker/nginx/nginx_main.conf:/etc/nginx/nginx.conf --link nginx1:nginx1 --link nginx2:nginx2 nginx
```



nginx_main反向代理服务器配置

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# include /usr/share/nginx/modules/*.conf;

event {
	worker_connections 1024;
}

http {
	# 反向代理配置，nginx1的权重是2，nginx2的权重是3
	upstream php-fpm {
		server nginx1 weight=2;
		server nginx2 weight=3;
	}
	log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';
	access_log  /var/log/nginx/access.log  main;
	sendfile            on;
	tcp_nopush          on;
	tcp_nodelay         on;
	keepalive_timeout   65;
	types_hash_max_size 2048;

	include             /etc/nginx/mime.types;
	default_type        application/octet-stream;

	server {
		listen 80;
		server_name localhost;
		root /usr/share/nginx/html;
		location / {
			# 此处的php-fpm为上面upstream模块配置的php-fpm
			proxy_pass http://php-fpm;
		}
	}
}
# proxy_pass的php-fpm必须加上http://，否则报错
```


