---
title: 反转数组
date: 2018-02-04
categories: 算法
tags: 算法
---
反转数组，反转字符串均可，有两种思路

### 第一种
#### 思路
```
1. 使用for循环
2. 循环到i/2处，将每个值和count-i进行交换
3. 返回原数组
```

#### 实现
```php
function reverge_array($arr){
    $count = count($arr) - 1;
    for ($i=0; $i<=$count/2; $i++){
        $temp = $arr[$i];
        $arr[$i] = $arr[$count-$i];
        $arr[$count-$i] = $temp;
    }
    return $arr;
}
```

### 第二种
#### 思路
```
1. 使用while循环
2. 设置left和right，分别为第一个和数组最后一个的下标
3. while判断right大于left，然后将arr[left]和arr[right]的值进行交换
4. left++，right--
```

#### 实现
```php
function reverge_array($arr){
    $left = 0;
    $right = count($arr)-1;
    while ($right > $left){
        $tmp = $arr[$left];
        $arr[$left] = $arr[$right];
        $arr[$right] = $tmp;
        $right --;
        $left ++;
    }
    return $arr;
}
```