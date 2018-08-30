---
title: 二分查找
date: 2018-02-04
categories: 算法
tags: 算法
---

二分查找的前提是数组已经排序好

#### 思路
```
1. 取出最小下标low和最大下标high
2. 使用while循环
3. 取出中间位置
4. 再拿target和中间位置的数做比较，如果大于，则将low = middle+1，降低数据查找的范围。
```

#### 实现
```php
function binary_search($arr, $target) {
    $high = count($arr);
    $low = 0;
    while ($high >= $low){
        $middle = floor(($high + $low) / 2);
        if ($target > $arr[$middle]){
            $low = $middle + 1;
        }elseif ($target < $arr[$middle]) {
            $high = $middle - 1;
        }else {
            return $middle;
        }
    }
}
```