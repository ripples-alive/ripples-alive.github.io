---
layout: post
title: SQL 行转列 实现二维表格
modified:
categories: technology
description:
tags: [sql, mysql, view]
date: 2016-04-26T11:59:38+08:00
---

前段时间，看别个写的 view 的时候，看到个二维表格的实现，感觉甚是不错，效果比我之前[那个方案](/通过SQL实现二维表格/)感觉还要好。

还是就上次的例子来说，上次本来是说『印象中SQL里似乎没有什么函数可以统计某一字段为特定值的记录数』，不过这里其实我们是有办法可以实现的。

```sql
SELECT `date`, SUM(IF(`state` = 1, 1, 0)) state1, SUM(IF(`state` = 2, 1, 0)) state2
FROM `order`
GROUP BY `date`
ORDER BY `date` DESC
```

<p id="read-more-anchor"/>

这里通过 if 和 sum 操作的结合，只统计了满足指定条件的行数，不得不说很是巧妙。
不过实际测试的时候，却比我那个 join assist 跑的慢，然而只要稍微改下，就比我那个快了：

```sql
SELECT `date`, SUM(IF(`state` = 1, `total`, 0)) state1, SUM(IF(`state` = 2, `total`, 0)) state2
FROM (
    SELECT `date`, `state`, COUNT(*) `total`
    FROM `order`
    GROUP BY `date`, `state`
)
GROUP BY `date`
ORDER BY `date` DESC
```

大概是由于通过子查询先做 group 减少了 if 判断的次数，从而提升了性能（有时候子查询还是有好处的:unamused:）。

不得不说，这个方案确实是个很好的解决方案，写起来也很方便，感觉确实是有必要记录一个:flushed:。
