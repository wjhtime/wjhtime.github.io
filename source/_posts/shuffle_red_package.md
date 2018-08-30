---
title: 随机红包算法
date: 2018-8-30
categories: 算法
tags: 算法
---



每次生成红包，金额随机，不太大，也不会太小，红包总金额不会超出限额。



```php
class MoneyPackage {
    public $leftMoney;
    public $leftSize;

    public function __construct($leftMoney, $leftSize)
    {
        $this->leftMoney = $leftMoney * 100;
        $this->leftSize = $leftSize;
    }
}

function getRandomMoney(MoneyPackage $package) {
    if ($package->leftSize <= 0 || $package->leftMoney <= 0) {
        return '超出限额';
    }
    if ($package->leftSize == 1) {
        $package->leftSize--;
        $money = $package->leftMoney;
        $package->leftMoney = 0;
        return $money/100;
    }

    $min = 1;
    $max = $package->leftMoney / $package->leftSize * 2;
    $rand = mt_rand()/mt_getrandmax();
    $money = floor($rand * $max);
    $money = $money >= $min ? $money : 1;
    $package->leftSize--;
    $package->leftMoney -= $money;
    return $money/100;
}

$n = 10;
$r = [];
$package = new MoneyPackage(10, $n);
for ($i=0; $i<$n; $i++) {
    $r[] = getRandomMoney($package);
}

print_r($r);
print_r(array_sum($r));
```



不预先生成所有的红包，每次实时计算红包金额 

红包算法： 

```
红包金额=当前余额/剩余个数*2*(0-1之间随机数)
如果所得金额小于0.01，则将金额置为0.01 
```


