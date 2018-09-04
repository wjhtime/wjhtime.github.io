---
title: 跨域
date: 2018-09-04
tags: 网络
categories: 网络
---



什么样的请求算是跨域?
端口号必须相同，必须是同一子域名下

```
URL                                      说明                    是否允许通信
http://www.domain.com/a.js
http://www.domain.com/b.js         同一域名，不同文件或路径           允许
http://www.domain.com/lab/c.js

http://www.domain.com:8000/a.js
http://www.domain.com/b.js         同一域名，不同端口                不允许

http://www.domain.com/a.js
https://www.domain.com/b.js        同一域名，不同协议                不允许

http://www.domain.com/a.js
http://192.168.4.12/b.js           域名和域名对应相同ip              不允许

http://www.domain.com/a.js
http://x.domain.com/b.js           主域相同，子域不同                不允许
http://domain.com/c.js

http://www.domain1.com/a.js
http://www.domain2.com/b.js        不同域名                         不允许

```

