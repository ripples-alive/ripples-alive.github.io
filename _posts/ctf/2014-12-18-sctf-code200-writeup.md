---
title: SCTF Code200 WriteUp
author: Ripples
layout: post
permalink: /sctf-code200-writeup/
views:
  - 245
categories:
  - CTF
tags:
  - CTF
  - SCTF
---
<p style="text-indent: 2em;">
  code200这道题应该算是一道送分的大水题吧，题目意思很明确，不需要去猜，然后做法也很简单。
</p>

<p style="text-indent: 2em;">
  题目描述如下：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">在218.2.197.248:10007运行了一个ATM程序，但是这个ATM有一个特点，每次只能存2^i（i为偶数）元或者取2^i(i为奇数)元，0&lt;=i&lt;99，且每个i只能使用一次。给出任意一定的金额（正数代表取出，负数代表存进），怎样操作这个ATM才能满足给定的金额?
eg:
13
4&nbsp;3&nbsp;2&nbsp;0
983
10&nbsp;5&nbsp;3&nbsp;1&nbsp;0</pre>

<p style="text-indent: 2em;">
  话说这道题，赛后看样例的时候，突然发现，题目写反了，4 3 2 0 对应的应该是 &#8211; 2^4 + 2^3 &#8211; 2^2 &#8211; 2^0 = -13。
</p>

<p style="text-indent: 2em;">
  不要在意这些细节，我们继续看……
</p>

<p style="text-indent: 2em;">
  很显然，我们可以先考虑正数的情况，负数就是类似的做即可。
</p>

<p style="text-indent: 2em;">
  拿到的第一想法肯定是对要求的金额进行二进制分解，这样得到的东西，显然是不能直接过的，其中2^i(i为奇数)的情况是只能取，不能存。那么，我们就要将2^i(i为奇数)替换成存2^(i+1)并取2^i，然后这样一路向上传递上去，便得以解决。
</p>

<p style="text-indent: 2em;">
  得到程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;socket

s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
s.connect((&#39;218.2.197.248&#39;,&nbsp;10007))

round_count&nbsp;=&nbsp;1
while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;aim&nbsp;=&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;aim&nbsp;=&nbsp;&#39;-809&#39;
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;aim&nbsp;==&nbsp;&#39;&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;aim&nbsp;+=&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;aim&nbsp;=&nbsp;int(aim)
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;aim

&nbsp;&nbsp;&nbsp;&nbsp;sign&nbsp;=&nbsp;int(aim&nbsp;&lt;&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;aim&nbsp;=&nbsp;abs(aim)
&nbsp;&nbsp;&nbsp;&nbsp;bin_aim&nbsp;=&nbsp;bin(aim)[2:][::-1]
&nbsp;&nbsp;&nbsp;&nbsp;bin_aim_len&nbsp;=&nbsp;len(bin_aim)
&nbsp;&nbsp;&nbsp;&nbsp;bin_aim&nbsp;=&nbsp;[bool(c&nbsp;==&nbsp;&#39;1&#39;)&nbsp;for&nbsp;c&nbsp;in&nbsp;(bin_aim&nbsp;+&nbsp;&#39;0&#39;&nbsp;*&nbsp;(100&nbsp;-&nbsp;len(bin_aim)))]
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;print&nbsp;bin_aim

&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;[]
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(99):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;bin_aim[i]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;i&nbsp;%&nbsp;2&nbsp;==&nbsp;sign:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pass
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;j&nbsp;=&nbsp;i&nbsp;+&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;bin_aim[j]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bin_aim[j]&nbsp;=&nbsp;False
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;j&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bin_aim[j]&nbsp;=&nbsp;True

&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(99,&nbsp;-1,&nbsp;-1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;bin_aim[i]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;+=&nbsp;str(i)&nbsp;+&nbsp;&#39;&nbsp;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;ans

&nbsp;&nbsp;&nbsp;&nbsp;s.send(ans[:-1])
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;break
&nbsp;&nbsp;&nbsp;&nbsp;round_count&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;Round&#39;,&nbsp;round_count</pre>
