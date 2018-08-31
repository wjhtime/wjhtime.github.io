---
title: nginx location配置
date: 2018-08-08
categories: nginx
tags: nginx
---



location按照最大匹配规则进行匹配

```
= 精确匹配
/ 通用匹配，任何请求都会匹配到
~ 区分大小写的匹配
~* 不区分大小写
```



例子：

```shell

location ~* ^/x/([a-zA-Z0-9_]+)/(.+)$ {
    proxy_pass http://$1.dns.alibaba.com/$2$is_args$args;
}
```

