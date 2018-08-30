---
title: 约瑟夫环
date: 2018-02-04
categories: 算法
tags: 算法
---
猴子问题，n个猴子围成圈，每第m个踢掉，最后剩下哪个

#### 思路
```
1. 因为不知道具体的循环次数，所以使用while循环
2. 使用临时变量i来计数，循环中每次取出第一个值
3. 判断i对m取余是否为0，如果为0，则证明该值应该去掉，否则将其重新添加到队尾
```

#### 实现
```php
function joseph_ring($n, $m){
    $arr = range(1, $n);
    $i = 0;
    while (count($arr)>1){
        $i ++:
        $left = array_shift($arr);
        if ($i % $m != 0) {
            array_push($arr, $left);
        }
    }
    return $arr[0];
}
```