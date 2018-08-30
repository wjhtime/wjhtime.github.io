---
title: cors跨域问题
date: 2018-01-15
tags: 网络
categories: 网络
---



以前碰到过跨域问题，这里整理了一下，主要是通过程序或者nginx配置实现的。



## 分析思路
cross-origin resource sharing，跨域资源共享，简称为CORS，html5下很容易解决
假设有两个网站，客户端：www.site.com ，服务器：www.example.com 
客户端要访问服务器的资源，发送ajax GET [www.example.com/ajax/resource](http://www.example.com/ajax/resource) ，请求头中会增加Origin:www.site.com。 

服务器返回头中要增加Access-Control-Allow-Orgin: *，这样客户端才会正常接收返回数据，没有的话会在console下报错。 

还有这两项：Access-Control-Allow-Methods，Access-Control-Allow-Headers


```
query-user-information:1 
Failed to load http://www.proton.work/test/ajax: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.dep.work' is therefore not allowed access.

现在没有Access-Control-Allow-Origin头在请求的资源上，源站点因此不能正常访问
```

## 解决办法

1.通过php程序里面处理 

```php
//目前采用的php原生代码，可以实现简单的跨域访问
处理如下:
//允许所有的域名
header('Access-Control-Allow-Origin: *');
//允许post，get请求
header('Access-Control-Allow-Methods: POST,GET ');
//允许的header
header('Access-Control-Allow-Headers: content-type, x-requested-with');
$json = ['succ'=>true,'info'=>['name'=>'my-name']];
//返回json字符串，如果直接用return返回的话，会报错
echo json_encode($json);
return ;
```



2.在laravel中增加cors处理 

给路由增加中间件处理，实际上也是通过增加header头实现的，github上有barryvhd-cors包可以直接使用 

```php
//路由之前增加cors中间件处理

Route::get('ajax', function (\Illuminate\Http\Request $request) { 
    $json = [ 
        'succ' => true, 
        'info' => [ 
            'name' => 'wjh' 
        ] 
    ];
    echo json_encode($json); 
})->middleware('cors');

//中间件文件
Cors.php
public function handle($request, \Closure $next)
{
    header('content-type:application:json;charset=utf8'); 
    header('Access-Control-Allow-Origin: *'); 
    header('Access-Control-Allow-Methods: GET,POST'); 
    header('Access-Control-Allow-Headers: content-type,x-requested-with'); 
    return $next($request); 
}
```



3.nginx中处理cors 

```nginx
if ($request_method = OPTION){
                add_header 'Access-Control-Allow-Orgin' *;
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'DNT, X-Mx-ReqToken, Keep-Alive, User-Agent, X-Requested-With, If-Modified-Since, Cache-Control, Content-Type';
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 200;
            }
```

