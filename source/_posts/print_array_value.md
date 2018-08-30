---
title: 打印符合规则的数组的值
date: 2018-7-29
categories: 算法
tags: 算法
---


遍历数组里的值，如果两数之和等于key，则打印这两个数，并且这两个值只能用一次


```php
$arr = range(-100, 100);

$key = 2;

function check($arr, $key) {
    $count = count($arr);
    for ($i=0; $i<$count; $i++){
        for($j=$i+1; $j<$count; $j++){
            if ($arr[$i] + $arr[$j] == $key) {
                print $arr[$i]. ', '.  $arr[$j]. "\n";
                $arr[$i] = null;
                $arr[$j] = null;
                break;
            }
        }
    }
}

check($arr, $key);
```

本来用的unset删掉数组的键值，后来发现这种方法不清晰，只会给自己挖坑，所以，改用$arr[$i] = null的方式。