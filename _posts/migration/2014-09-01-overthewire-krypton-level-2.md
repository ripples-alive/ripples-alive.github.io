---
title: OverTheWire Krypton Level 1 → Level 2
author: Ripples
layout: post
permalink: /overthewire-krypton-level-2/
views:
  - 280
categories:
  - CTF
tags:
  - decryption
  - krypton
  - wargame
---
<p style="text-indent: 2em;">
  不想说什么了，这题竟然卡了我这么久，简直就是无力吐槽。
</p>

<p style="text-indent: 2em;">
  题目<a href="http://overthewire.org/wargames/krypton/krypton2.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  这题就是给出凯撒加密的字符串让解原串。很轻松就能解得原串。
</p>

<pre class="brush:python;toolbar:false">s&nbsp;=&nbsp;&#39;OMQEMDUEQMEK&#39;
for&nbsp;i&nbsp;in&nbsp;range(26):
&nbsp;&nbsp;&nbsp;&nbsp;print(i,&nbsp;&#39;&#39;.join([chr(ord(&#39;A&#39;)&nbsp;+&nbsp;(ord(c)&nbsp;-&nbsp;ord(&#39;A&#39;)&nbsp;+&nbsp;i)&nbsp;%&nbsp;26)&nbsp;for&nbsp;c&nbsp;in&nbsp;s]))</pre>

<!--more-->

<p style="text-indent: 2em;">
  找到原串为CAESARISEASY。
</p>

<p style="text-indent: 2em;">
  结果我试密码的时候就试了easy、EASY、caesariseasy，然后都未遂，然后又一直觉得那个encrypt是有用的，要利用找到的这个原串去通过encrypt找到密码，结果就一直卡住了。。。
</p>

<p style="text-indent: 2em;">
  哎，说多了都是泪T_T。
</p>
