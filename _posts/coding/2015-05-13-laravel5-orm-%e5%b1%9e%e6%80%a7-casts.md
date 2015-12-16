---
title: Laravel5 ORM 属性 Casts
author: Ripples
layout: post
views:
  - 333
categories:
  - coding
tags:
  - casts
  - laravel
  - ORM
---
<p style="text-indent: 2em;">
  之前用laravel的时候，对于laravel自带的ORM，也就是Eloquent，一直都有一点很不爽，那就是所有的字段对应过来的属性全部都是字符串，这一点真的是让人很不爽，本来我就对弱类型感觉很蛋疼，觉得很容易出事，结果这地方还非要弄个和心里期望类型不一致。
</p>

<!--more-->

<p style="text-indent: 2em;">
  当然，这个问题，如果硬要做也是可以通过getter/setter来做，但麻烦不说，指不定哪地方laravel又会没按照<span style="text-indent: 32px;">getter/setter来取，那就只能呵呵了……</span>
</p>

<p style="text-indent: 2em;">
  然后，最近在laravel上看文档时，突然发现laravel5新加了一个<a href="http://laravel.com/docs/5.0/eloquent#attribute-casting" target="_blank">casts特性</a>，通过这个特性，我们相当于是显式指定了属性的类型，laravel在内部存取属性的时候，会自动帮我们把类型转换做好，这样我们就不用担心数据类型和心里预期不一致了，可以放心大胆的用===做比较了。
</p>

<p style="text-indent: 2em;">
  然后查了下laravel源代码，发现类型转换是下面这段：
</p>

<pre class="brush:php;toolbar:false">/**
&nbsp;*&nbsp;Cast&nbsp;an&nbsp;attribute&nbsp;to&nbsp;a&nbsp;native&nbsp;PHP&nbsp;type.
&nbsp;*
&nbsp;*&nbsp;@param&nbsp;&nbsp;string&nbsp;&nbsp;$key
&nbsp;*&nbsp;@param&nbsp;&nbsp;mixed&nbsp;&nbsp;&nbsp;$value
&nbsp;*&nbsp;@return&nbsp;mixed
&nbsp;*/
protected&nbsp;function&nbsp;castAttribute($key,&nbsp;$value)
{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(is_null($value))&nbsp;return&nbsp;$value;

&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;($this-&gt;getCastType($key))
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;int&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;integer&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;(int)&nbsp;$value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;real&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;float&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;double&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;(float)&nbsp;$value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;string&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;(string)&nbsp;$value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;bool&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;boolean&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;(bool)&nbsp;$value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;object&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;json_decode($value);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;array&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;json&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;json_decode($value,&nbsp;true);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;&#39;collection&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$this-&gt;newCollection(json_decode($value,&nbsp;true));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$value;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

<p style="text-indent: 2em;">
  通过这段我们也可以很清楚的发现支持指定了哪些类型。
</p>

<p style="text-indent: 2em;">
  然后本来以为这样就完事了，结果没想到今天突然发现了一个神奇的现象，到现在也没能想明白是怎么回事。我本地在没有加上casts的情况下，取出来的数据竟然自动识别了整数类型，然后刚感叹完竟然能自动识别了，放服务器上跑了下，立马就又打回原形了。本来还以为是laravel版本不同，结果看了下composer.lock，发现完全一致，都是v5.0.27，于是就惊呆了。然后又查了一下php版本，两个确实还不一样，本地是5.5.17，服务器是5.5.9-1ubuntu4.9，但不太能想象这会是PHP版本导致的不同，毕竟要知道数据类型必须是从数据库中查询出来。
</p>

<p style="text-indent: 2em;">
  在思前想后没想明白的情况下，又试了一下，如果我在本地，给一个被自动识别出是int的字段赋值为一个字符串，还是不会自动转为int，于是觉得还是显式用casts来指定类型的好，这样不管怎么操作，最后都保证了数据类型确定，然后返回的json中也不至于出现一下是字符串，一下是数字了。
</p>

<p style="text-indent: 2em;">
  总之，这个特性简直不能更喜欢，现在再剩下的类型蛋疼的地方就是输入的参数了，也不知道会不会哪天也能有好的解决方案~~~
</p>
