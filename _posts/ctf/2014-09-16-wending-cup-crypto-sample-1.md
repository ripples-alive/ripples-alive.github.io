---
title: 中软吉大•问鼎杯决赛样题密码类（一）
author: Ripples
layout: post
permalink: /%e4%b8%ad%e8%bd%af%e5%90%89%e5%a4%a7%e2%80%a2%e9%97%ae%e9%bc%8e%e6%9d%af%e5%86%b3%e8%b5%9b%e6%a0%b7%e9%a2%98%e5%af%86%e7%a0%81%e7%b1%bb%ef%bc%88%e4%b8%80%ef%bc%89/
views:
  - 617
categories:
  - CTF
tags:
  - decryption
  - WriteUp
  - 中软吉大•问鼎杯
---
<p style="text-indent: 2em;">
  密码类样题一共两道题，这是第一题，题目如下：
</p>

<pre class="brush:plain;toolbar:false">1、有一天，小明攻入一个服务器中发现一个可疑的字符串，&nbsp;aHR0cDovLyU3MyU2NSU2MyUyRSU2OCU2NCU3NSUyRSU2NSU2NCU3NSUyRSU2MyU2RS9TZWNIb21lLw，你能帮小明解密吗？(200分)</pre>

<!--more-->

<p style="text-indent: 2em;">
  刚看到题的时候有点晕乎，这神马玩意，不过一试便会发现其实很简单，只是自己这方面的意识还是太弱。
</p>

<p style="text-indent: 2em;">
  对这个字符串做一下base64解码，得到：
</p>

<pre class="brush:plain;toolbar:false">http://%73%65%63%2E%68%64%75%2E%65%64%75%2E%63%6E/SecHome</pre>

<p style="text-indent: 2em;">
  很明显应该方向对了，但是网址还是不对，于是再做一下urldecode：
</p>

<pre class="brush:python;toolbar:false">&gt;&gt;&gt;&nbsp;urllib.unquote(&#39;http://%73%65%63%2E%68%64%75%2E%65%64%75%2E%63%6E/SecHome&#39;)
&#39;http://sec.hdu.edu.cn/SecHome&#39;</pre>

<p style="text-indent: 2em;">
  这题便得以解决～～～
</p>
