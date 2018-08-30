---
title: swoole扩展安装
date: 2018-8-13
categories: php
tags: php
---



github上下载swoole的release包



```shell
1. tar -zxvf swoole-src-4.0.3.tar.gz
解压包

2. cd swoole-src-4.0.3
进入目录

3. phpize
生成编译检测脚本

4. ./configure --with-php-config=/usr/bin/php-config
编译配置检测

5. make
编译

6. make install
安装，之后会打印扩展位置

Installing shared extensions:     /usr/local/Cellar/php@7.0/7.0.29_1/pecl/20151012/
Installing header files:           /usr/local/Cellar/php@7.0/7.0.29_1/include/php/

7. 修改php.ini，也可以在conf.d目录下创建扩展文件
[swoole]
extension="/usr/local/Cellar/php@7.0/7.0.29_1/pecl/20151012/swoole.so"
```



相关工具： 

```
phpize    编译安装扩展工具 

php-config    获取php的配置信息
```

