---
title: composer命令及其配置
date: 2017-08-10
categories: php
tags: composer
---





### 配置文件

```shell
查看所有配置项，一些看不懂的没有列出来

composer config -l

[repositories.packagist.org.type] composer
//仓库地址
[repositories.packagist.org.url] https://packagist.org
[repositories.packagist.org.allow_ssl_downgrade] true
...
[github-protocols] [https, ssh]
[vendor-dir] vendor (/Users/user/Sites/atom/packages/tuling/vendor)
[bin-dir] {$vendor-dir}/bin (/Users/user/Sites/atom/packages/tuling/vendor/bin)
[cache-dir] /Users/user/.composer/cache
[data-dir] /Users/user/.composer

//缓存地址
[cache-files-dir] {$cache-dir}/files (/Users/user/.composer/cache/files)
[cache-repo-dir] {$cache-dir}/repo (/Users/user/.composer/cache/repo)
[cache-vcs-dir] {$cache-dir}/vcs (/Users/user/.composer/cache/vcs)
[cache-ttl] 15552000
[cache-files-ttl] 15552000
[cache-files-maxsize] 300MiB (314572800)
...
//composer全局配置地址
[home] /Users/user/.composer
```



### require包的版本号规则

```
常用版本号规则，确定版本号、版本号范围、版本号通配符，2、3、4是常用的版本号规则
>= 是范围符号，大于某个

//指定确定版本号
1. 1.0.2    

//在某个版本号的范围之内，可以加逗号、或运算符，>=1.0,<2.0  >=1.0|<2.0  逗号表示and，两个范围之内的版本号，|表示或者
2. >=1.0    

// *星号表示匹配任意值，例如：表示1.0.*下的任意版本，等效于>=1.0,<1.1
3. 1.0.*    

//~波浪号表示在某一个重要版本下的小版本升级
例如：在1.2.2版本之上升级，但是不升级到1.3.0版本  1.2 >= 版本号 < 1.3.0
避免高版本不兼容低版本
4. ~1.2.2     

//^和~类似， 1.2.3 <= 版本号 < 2.0.0
//比~的范围更广，匹配下一个重要版本（官网上没有，但实际中经常见到）
5. ^1.2.3
```



### composer.json示例文件

```json
// composer.json配置文件

"type": "library",  //默认为library
"require": {
    //对php有版本限制
    "php": ">=5.6",
    //要求php扩展，必须安装ext-mbstring
    "ext-mbstring": "*",
    //要求php库，必须安装lib-curl
    "lib-curl": "*"
},
"require-dev": {},
"autoload": {
     "psr-4": {
         "App\\": "app/",
          "Monolog\\": ["src/", "lib/"]
     },
     "psr-0": {
         "Monolog\\": ["src/", "lib/"] 
     },
     "classmap": {     //不遵循psr-0/4规范
         "database" 
     },
     "files": {     //作为函数库引入
         ["url/to/your/helpers.php"] 
     }
},
"repositories": {
     "packagist": {
         "type": "composer",
         "url": "https://packagist.phpcomposer.com"
     }
},
//最低版本要求，尽量不要安装dev-master版本的包，安装稳定版
"minimum-stability": "stable"
```



### 常用命令

```shell
//查看软件包被哪些包依赖
composer depends guzzlehttp/guzzle

composer init
composer require
composer update
composer install 
     —no-dev 跳过require-dev中列出的包
     —dev 安装require-dev列出的包
     
//重建索引
composer dump-autoload
```

