---
title: 概率输出字符
date: 2018-08-31
categories: 算法
tags: 算法
---

​	执行方法输出字符，每次输出字符a,b,c,d中的一个，当量到一定程度，例如循环1000000次之后，统计每个字符的出现次数，满足a：10%,b：20%,c：30%,d：40%



```php
//a出现的概率是10%，b是20%，c是30%，d是40%
$pro = [
    'a' =>10,
    'b' =>20,
    'c' =>30,
    'd' =>40
];

function gailv($arr){
    $number = mt_rand(1, array_sum($arr));
    switch ($number){
        case $number<=10:
            return 'a';
            break;
        case $number>10 && $number<=30:
            return 'b';
            break;
        case $number>30&&$number<=60:
            return 'c';
            break;
        case $number>60&&$number<=100:
            return 'd';
            break;
    }
}
```



测试方法：

```php
for($i=0; $i<1000000; $i++){
    $r[] = gailv($pro);
}
print_r(array_count_values($r));

//输出结果，基本满足概率
Array
(
    [a] => 99958
    [d] => 400218
    [c] => 299746
    [b] => 200078
)
```

遗留问题：
后期扩展比较麻烦，修改一个字符的概率，要把程序重写一遍

---

上面的方法太蠢了，网上搜到的更好的方法，且易于扩展

```php
function proRand($arr) {
    $sum = array_sum($arr);
    foreach ($arr as $k => $v) {
        $rand = mt_rand(1, $sum);
        if ($rand <= $v) return $k;
        else {
            $sum -= $v;
        }
    }
}
```





