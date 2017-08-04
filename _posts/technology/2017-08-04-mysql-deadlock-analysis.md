---
layout: post
title: 一个 MySQL 死锁案例分析
modified:
categories: technology
description:
tags: [sql, mysql, deadlock]
date: 2017-08-04T15:43:44+08:00
---

对 MySQL 死锁不是特别擅长，业务中遭遇了，就尝试分析一下了。

首先有篇不错的[关于 MySQL 加锁机制的文章](http://yeshaoting.cn/article/database/mysql%20insert%E9%94%81%E6%9C%BA%E5%88%B6/#8-查看innodb状态-包含最近的死锁日志)以及一个[很有意思的 INSERT 死锁例子](http://blog.csdn.net/and1kaney/article/details/51214001)，Mark 一下。

业务需求，写出了类似这样的 SQL：

```sql
INSERT INTO site(third_party_id, data) VALUES(%s, %s)
ON DUPLICATE KEY UPDATE data = %s
```

其中 `site` 表主键 `id` 自增，唯一索引 `third_party_id`。

SQL 目的很简单，就是从一张三方表中导数据到自己业务的一张表中间。
就这样简单的一句 SQL，没有复杂的事务，只是会出现大量的并发。

<p id="read-more-anchor"/>

并发跑起来之后，很快就会发现程序报出大量死锁，看下死锁记录，可以抓到一条死锁信息大致如下：

```
*** (1) TRANSACTION:
TRANSACTION 4185716323, ACTIVE 0.021 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1184, 3 row lock(s)
LOCK BLOCKING MySQL thread id: 252573051 block 235860837
MySQL thread id 235860837, OS thread handle 0x7f6fcc2a4700, query id 1581628064 172.16.7.71 prod update
...
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 794 page no 8 n bits 128 index `PRIMARY` of table ``.`` trx id 4185716323 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 35; compact format; info bits 0
...
*** (2) TRANSACTION:
TRANSACTION 4185716326, ACTIVE 0.014 sec inserting
mysql tables in use 1, locked 1
5 lock struct(s), heap size 1184, 3 row lock(s)
MySQL thread id 235818269, OS thread handle 0x7f6d88fbe700, query id 1581628068 172.16.7.71 prod update
...
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 794 page no 8 n bits 128 index `PRIMARY` of table ``.`` trx id 4185716326 lock_mode X locks gap before rec
Record lock, heap no 2 PHYSICAL RECORD: n_fields 35; compact format; info bits 0
...
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 794 page no 8 n bits 128 index `PRIMARY` of table ``.`` trx id 4185716326 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 35; compact format; info bits 0
```

这里事务 2 由于是 `INSERT with ON UPDATE`，在 constraint check 中会直接上 X 锁，参见[这里](https://bugs.mysql.com/bug.php?id=7975)。
而同时我们这里主键让其自增，针对 `third_party_id` 字段来做的操作，于是导致主键索引产生的 gap 锁而不是 record 锁。
接下来，与[这里](https://bugs.mysql.com/bug.php?id=52020)类似，事务2在有了 X gap 锁之后，又申请了个其实没有必要的 insert intention lock，
这一请求排在了事务1的 insert intention lock 请求之后，于是带来了死锁。

这里对于这一导入数据的业务需求，其实一次导入中并不会出现多个相同的 `third_party_id`，
于是我们直接 SELECT 出来，判断一下再 INSERT 或者 UPDATE 即可，两条语句间也不需要做事务，因为不会中途被改掉。