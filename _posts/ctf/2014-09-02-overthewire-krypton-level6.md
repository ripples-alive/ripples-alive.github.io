---
title: OverTheWire Krypton Level 5 → Level 6
author: Ripples
layout: post
views:
  - 973
categories:
  - CTF
tags:
  - decryption
  - krypton
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  题目传送门<a href="http://overthewire.org/wargames/krypton/krypton6.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  题目中英语漫漫长，就以吾等这英语水平，只能表示不明觉厉了。
</p>

<p style="text-indent: 2em;">
  最后大概总结出来，这题就是要选择明文攻击，然后HINT1也说了The 'random' generator has a limited number of bits, and is periodic，那这就好办了。
</p>

<p style="text-indent: 2em;">
  首先我们尝试明文：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA</pre>

<p style="text-indent: 2em;">
  得到对应密文：
</p>

<pre class="brush:plain;toolbar:false">EICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXY</pre>

<p style="text-indent: 2em;">
  很明显可以看出是循环的，循环节长度为30。
</p>

<p style="text-indent: 2em;">
  然后尝试明文：
</p>

<pre class="brush:plain;toolbar:false">BBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA</pre>

<p style="text-indent: 2em;">
  得到密文：
</p>

<pre class="brush:plain;toolbar:false">FJDUEHYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXYCPFUEOCKRNEICTDGYIYZKTHNSIRFXY</pre>

<p style="text-indent: 2em;">
  发现只有前6个字母变了，那么看来应该是一位只会影响对应的一位，那么我们直接枚举每一位为A-Z的情况，分别加密，然后拿我们的加密了的password来对比，即可得到password的原文了。
</p>

<p style="text-indent: 2em;">
  程序如下：
</p>

<pre class="brush:python;toolbar:false">import&nbsp;subprocess

ans&nbsp;=&nbsp;[]
for&nbsp;i&nbsp;in&nbsp;xrange(26):
&nbsp;&nbsp;&nbsp;&nbsp;fp&nbsp;=&nbsp;open(&#39;/tmp/kry/foo&#39;,&nbsp;&#39;w&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;fp.write(chr(i&nbsp;+&nbsp;ord(&#39;A&#39;))&nbsp;*&nbsp;30)
&nbsp;&nbsp;&nbsp;&nbsp;fp.close()
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;=&nbsp;subprocess.call([&#39;/krypton/krypton6/encrypt6&#39;,&nbsp;&#39;/tmp/kry/foo&#39;,&nbsp;&#39;/tmp/kry/bar&#39;])
&nbsp;&nbsp;&nbsp;&nbsp;fp&nbsp;=&nbsp;open(&#39;/tmp/kry/bar&#39;,&nbsp;&#39;r&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;ans.append(fp.read())
&nbsp;&nbsp;&nbsp;&nbsp;fp.close()
&nbsp;&nbsp;&nbsp;&nbsp;
s&nbsp;=&nbsp;&#39;PNUKLYLWRQKGKBE&#39;
plain&nbsp;=&nbsp;&#39;&#39;
for&nbsp;i&nbsp;in&nbsp;xrange(len(s)):
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;j&nbsp;in&nbsp;xrange(26):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;s[i]&nbsp;==&nbsp;ans[j][i]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;plain&nbsp;+=&nbsp;chr(j&nbsp;+&nbsp;ord(&#39;A&#39;))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
print(plain)</pre>

<p style="text-indent: 2em;">
  至此，整个krypton得以解决，不得不说，krypton感觉就是入门级的题，没有什么难度。
</p>
