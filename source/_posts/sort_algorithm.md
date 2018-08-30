---
title: 排序算法
date: 2018-02-04
categories: 算法
tags: 算法
---

### 冒泡排序
#### 思路
```
1. 设计算法复杂度为n方的嵌套循环
2. i和j都从0开始，i<count，j<count-i-1
3. 比较arr[j]和arr[j+1]，如果arr[j]大于arr[j+1]，则交换顺序
```

#### 实现
```php
function bubble_sort($arr) {
    $count = count($arr);
    for ($i=0; $i<$count; $i++) {
        for ($j=0; $j<$count-$i-1; $j++) {
            if ($arr[$j] > $arr[$j+1]){
                $tmp = $arr[$j];
                $arr[$j] = $arr[$j+1];
                $arr[$j+1] = $tmp;
            }
        }
    }
    return $arr;
}
```

### 快速排序
#### 思路
```
1. 取数组第一个值为标准值
2. 设置比标准值大的数组为right，比标准值小的数组为left
3. for循环数组所有变量，如果比标准值大则放到right，反之，放到left
4. 分别对left和right使用递归，最后分别赋值给left和right
5. 返回合并数组，顺序是left, [key], right
```

### 实现
```php
function quick_sort($arr) {
    $count = count($arr);
    if ($count <= 1) return $arr;
    $key = $arr[0];
    $left = [];
    $right = [];
    for ($i=1; $i<$count; $i++) {
        if ($arr[$i] > $key){
            array_push($right, $arr[$i]);
        }else {
            array_push($left, $arr[$i]);
        }
    }
    $left = quick_sort($left);
    $right = quick_sort($right);
    return array_merge($left, [$key], $right);
}
```


