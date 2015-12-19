---
title: Laravel 没有自动加载类
author: Ripples
layout: post
permalink: /2014/12/14/laravel%e6%b2%a1%e6%9c%89%e8%87%aa%e5%8a%a8%e5%8a%a0%e8%bd%bd%e7%b1%bb/
views:
  - 335
categories:
  - coding
tags:
  - autoload
  - composer
  - laravel
---
<p style="text-indent: 2em;">
  今天为了做作业，又新建了一个laravel的项目，但是运行的时候却发现自己写在models中的类没有自动加载，很是郁闷。
</p>

<p style="text-indent: 2em;">
  检查了半天，和之前的那个laravel的项目对比了下，没发现什么config有问题。
</p>

<!--more-->

<p style="text-indent: 2em;">
  于是乎，在网上查，找到了<a href="http://segmentfault.com/q/1010000000583540" target="_blank">这个</a>：
</p>

<pre class="brush:plain;toolbar:false">由于Laravel是使用composer加载类的，如果不是使用命令创建的类是需要更新autoload的，正如@kankana说的&nbsp;：composer&nbsp;dump-autoload，推荐看下这里http://segmentfault.com/a/1190000000355928</pre>

<p style="text-indent: 2em;">
  创建项目我是使用的是：
</p>

<pre class="brush:bash;toolbar:false">composer&nbsp;create-project&nbsp;laravel/laravel&nbsp;--prefer-dist</pre>

<p style="text-indent: 2em;">
  不知道这个是否满足上面所说的条件，但是，既然没有自动加载，那我们就还是上面说的，运行一下：
</p>

<pre class="brush:bash;toolbar:false">composer&nbsp;dump-autoload</pre>

<p style="text-indent: 2em;">
  或者是使用：
</p>

<pre class="brush:bash;toolbar:false">omposer&nbsp;dump-autoload&nbsp;--optimize</pre>

<p style="text-indent: 2em;">
  运行完之后再尝试，便会发现程序可以自动加载各个类了，至少表面上看起来，问题是得以解决了。
</p>
