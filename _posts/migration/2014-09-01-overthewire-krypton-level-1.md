---
title: OverTheWire Krypton Level 0 → Level 1
author: Ripples
layout: post
permalink: /overthewire-krypton-level-1/
views:
  - 214
categories:
  - CTF
tags:
  - decryption
  - krypton
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  这一关题目<a href="http://overthewire.org/wargames/krypton/krypton1.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  不得不说，这一关也是大水题，明确告诉我们是rotation了，本来就试不了一下就OK了，然后服务器上README中竟然还写It is 'encrypted' using a simple rotation called ROT13。这下就连试都不用了，知道得到答案。
</p>

<pre class="brush:python;toolbar:false">s&nbsp;=&nbsp;&#39;YRIRY&nbsp;GJB&nbsp;CNFFJBEQ&nbsp;EBGGRA&#39;
ans&nbsp;=&nbsp;&#39;&#39;
for&nbsp;c&nbsp;in&nbsp;s:
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;c&nbsp;==&nbsp;&#39;&nbsp;&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;+=&nbsp;c
&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;+=&nbsp;chr((ord(c)&nbsp;-&nbsp;ord(&#39;A&#39;)&nbsp;+&nbsp;13)&nbsp;%&nbsp;26&nbsp;+&nbsp;ord(&#39;A&#39;))

print&nbsp;ans</pre>
