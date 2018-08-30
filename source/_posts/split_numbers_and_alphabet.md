---
title: 将字符串和数字分组
date: 2018-02-04
categories: 算法
tags: 算法
---

分组格式化字符串
1a3bb44a2ac => 1:a 3:bb 44:a 2:ac

#### 思路
```
1. 前提是两者匹配出现，不会在最后面有多余
2. 使用preg_split函数，根据\d和[a-zA-Z]进行分隔
3. 循环数组，根据key进行字符串拼接
4. implode(' ')，将数组合并
```

#### 实现
```php
function number_alphabet($str){
    $result = [];
    //注意？防止使用贪婪匹配，否则不会截断字符串
    $number = preg_replace("/[0-9]+?/", $str, PREG_SPLIT_NO_EMPTY);
    $alphabet = preg_replace("/[a-zA-Z]+?/", $str, PREG_SPLIT_NO_EMPTY);
    foreach ($number as $key=>$vaule){
        array_push($result, $value.":".$alphabet[$key]);
    }
    return array_implode(" ", $result);
}
```