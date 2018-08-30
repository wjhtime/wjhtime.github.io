---
title: 打乱数组
date: 2018-02-04
categories: 算法
tags: 算法
---

相当于shuffle函数

#### 思路
```
1. for循环数组的每一个变量
2. 使用随机数函数取出数组的另外一个随机值
3. 将两者交换
```

#### 实现
```php
function shuffle_array($arr) {
    $count = count($arr) -1 ;
    for ($i=0; $i<=$count; $i++){
        $key = mt_rand(0, $count);
        if ($arr[$i] != $arr[$key]){
            $tmp = $arr[$key];
            $arr[$key] = $arr[$i];
            $arr[$i] = $tmp;
        }
    }
    return $arr;
}
```