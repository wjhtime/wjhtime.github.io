---
title: 括号匹配问题
date: 2018-07-25
categories: 算法
tags: 算法
---



验证括号是否匹配 `{1+2}+[(3+4)*5]*6-1` ，这种类型的字符串，括号必须成对出现，否则不符合要求

### 实现

```php
$str = "{1+2}+[(3+4)*5]*6-1";
$search = "[](){}";

function check_match($str, $search){
    $result = [];
    $length = strlen($str);
    for ($i=0; $i<$length; $i++){
        if (strstr($search, $str[$i])){
            $tmp = array_pop($result);
            if ($tmp){
                $pos = strpos($search, $tmp);
                if ($search[$pos+1] == $str[$i]) {
                    continue;
                }
                array_push($result, $tmp);
            }
            array_push($result, $str[$i]);
        }
    }
    if (count($result)>0) return false;
    return true;
}

$result = check_match($str, $search);
var_dump($result);
```

