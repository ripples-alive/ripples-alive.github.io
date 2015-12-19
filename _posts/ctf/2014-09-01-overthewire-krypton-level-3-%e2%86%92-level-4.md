---
title: OverTheWire Krypton Level 3 → Level 4
author: Ripples
layout: post
permalink: /overthewire-krypton-level-3-%e2%86%92-level-4/
views:
  - 353
categories:
  - CTF
tags:
  - decryption
  - krypton
  - wargame
  - WriteUp
  - 频率分析
---
<p style="text-indent: 2em;">
  题目传送门<a href="http://overthewire.org/wargames/krypton/krypton4.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  这一关给出的是一个维吉尼亚密码，本来应该不是这么好解的，但是，由于题目README告诉了我们密钥长度为6，同时又告诉我们频率分析奏效，那么这题就变成了一道大水题。
</p>

<p style="text-indent: 2em;">
  首先我们按照长度分组，分别做频率统计，可以写个程序来实现统计，但是发现了这个<a href="http://www.richkni.co.uk/php/crypta/vignere.php" target="_blank">网站</a>，提交上去它会自动帮我们做统计。
</p>

<p style="text-indent: 2em;">
  然后假设频率最高的字母是e，这样我们惊奇的发现，竟然直接就得到了正确答案，好吧，只能说对这题无力吐槽。
</p>
