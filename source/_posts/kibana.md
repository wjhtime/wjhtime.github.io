---
title: kibana使用技巧
date: 2018-03-23
categories: 学习笔记
---


### kibana搜索规则

`"key word"`
关键词搜索，加上引号可以查询整个短语



`status:504`
按关键词搜索，查找结果中包含504的数据



`?`匹配单个字符
`*` 匹配多个字符



`AND`
`OR`
逻辑操作，必须大写



`+`	结果中必须有
`-`	结果中去掉


统计nginx访问日志

状态码是200，不是资源类型的请求

```
status:200 AND NOT request:png AND NOT request:gif AND NOT request:jpg AND NOT request:ico AND NOT request:css AND NOT request:js
```

