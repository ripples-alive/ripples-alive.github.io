---
title: Laravel Eloquent ORM 数据更新
author: Ripples
layout: post
permalink: /lavarel-eloquent-orm-数据更新/
views:
  - 511
categories:
  - coding
tags:
  - laravel
  - ORM
  - php
  - 数据库
---
<p style="margin-left: 0em; text-indent: 2em;">
  最近在用laravel写后台，不得不说<span style="text-indent: 32px;">laravel</span>用的真爽，彻底不同于php之前给我的感觉，虽说还是对于弱类型有时候感觉有点不爽。
</p>

<p style="margin-left: 0em; text-indent: 2em;">
  然而，今天用ORM更新数据库的时候却遇到个神奇的问题，<span style="text-indent: 32px;">laravel</span>的文档中对于模型的更新是这样举例的：
</p>

<pre class="brush:php;toolbar:false">$user&nbsp;=&nbsp;User::find(1);
$user-&gt;email&nbsp;=&nbsp;&#39;john@foo.com&#39;;
$user-&gt;save();</pre>

<!--more-->

<p style="margin-left: 0em; text-indent: 2em;">
  方法很明确，和模型插入的例子是相似的：
</p>

<pre class="brush:php;toolbar:false">$user&nbsp;=&nbsp;new&nbsp;User;
$user-&gt;name&nbsp;=&nbsp;&#39;John&#39;;
$user-&gt;save();</pre>

<p style="margin-left: 0em; text-indent: 2em;">
  然而，自己在使用的时候，却发生了一个神奇的现象，对于模型的插入，数据库正常插入了，可是对于模型的更新，数据库却没有正常的更新。
</p>

<p style="margin-left: 0em; text-indent: 2em;">
  首先的第一反应当然是检查save的返回值，可是却发现返回的是true，就是说<span style="text-indent: 32px;">laravel</span>是认为save的操作是成功执行了的。
</p>

<p style="margin-left: 0em; text-indent: 2em;">
  然后我尝试了类似如下的代码：
</p>

<pre class="brush:php;toolbar:false">$user&nbsp;=&nbsp;new&nbsp;User;
$user-&gt;NAME&nbsp;=&nbsp;&#39;A&#39;;
$user-&gt;save();
$user-&gt;NAME&nbsp;=&nbsp;&#39;B&#39;;
$user-&gt;save();</pre>

<p style="margin-left: 0em; text-indent: 2em;">
  执行之后发现，第二个save确实成功更新了数据库，那就更让我疑惑了。
</p>

<p style="margin-left: 0em; text-indent: 2em;">
  最后折腾了半天之后，在一个朋友的帮助下，大概找出来问题所在。
</p>

<p style="margin-left: 0em; text-indent: 2em;">
  我们的数据库里面字段名都是用的大写，把主键名改为小写即可解决这个问题，不得不说真是神奇，检查ORM的log会发现，在主键名大写的时候，SQL语句的是类似：
</p>

<pre class="brush:sql;toolbar:false">select&nbsp;*&nbsp;from&nbsp;`table_name`&nbsp;where&nbsp;`id`&nbsp;is&nbsp;null</pre>

<p style="margin-left: 0em; text-indent: 2em;">
  而主键名小写的时候，SQL语句是类似：
</p>

<pre class="brush:sql;toolbar:false">select&nbsp;*&nbsp;from&nbsp;`table_name`&nbsp;where&nbsp;`id`&nbsp;=&nbsp;?</pre>

<p style="margin-left: 0em; text-indent: 2em;">
  也就是说，大写的时候，ORM没能成功找到主键的条件，具体ORM的源码也没时间研究，但出这种问题真是有点让人无言以对，看来这真是天都来谴责我使用ORM了，不过就是看到文档里专门大费章节写以为是推荐这样才用的，不然直接SQL语句多好！！！
</p>
