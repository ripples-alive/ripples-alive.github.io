---
title: OverTheWire Krypton Level 4 → Level 5
author: Ripples
layout: post
permalink: /overthewire-krypton-level-5/
views:
  - 294
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
  题目传送门<a href="http://overthewire.org/wargames/krypton/krypton5.html" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  这关还是维吉尼亚密码，给了我们三个加密文本，不过没有告诉我们长度，但是频率统计还是奏效。
</p>

<p style="text-indent: 2em;">
  于是乎，首先在<a href="http://www.simonsingh.net/The_Black_Chamber/vigenere_cracking_tool.html" target="_blank">这里</a>分别提交三个加密文本，可以发现最可能的长度是3或者9，3个文本都满足这一特性。
</p>

<p style="text-indent: 2em;">
  然后用程序统计频率，尝试将最高频的字母当作e来进行解密：
</p>

<!--more-->

<pre class="brush:python;toolbar:false">def&nbsp;decode(s,&nbsp;key):
&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;l&nbsp;=&nbsp;len(key)
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;k&nbsp;in&nbsp;xrange(len(s)):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&nbsp;k&nbsp;/&nbsp;l
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;j&nbsp;=&nbsp;k&nbsp;%&nbsp;l
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;+=&nbsp;chr((ord(s[i&nbsp;*&nbsp;l&nbsp;+&nbsp;j])&nbsp;-&nbsp;ord(key[j])&nbsp;+&nbsp;26)&nbsp;%&nbsp;26&nbsp;+&nbsp;ord(&#39;a&#39;))
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;ans
&nbsp;&nbsp;&nbsp;&nbsp;
def&nbsp;search(base_s,&nbsp;deep,&nbsp;sec,&nbsp;key):
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;deep&nbsp;==&nbsp;sec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;text&nbsp;=&nbsp;decode(base_s,&nbsp;key)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;text.find(&#39;the&#39;)&nbsp;&gt;=&nbsp;0:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(text[0:20],&nbsp;key)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return
&nbsp;&nbsp;&nbsp;&nbsp;s&nbsp;=&nbsp;base_s[deep::sec]
&nbsp;&nbsp;&nbsp;&nbsp;d&nbsp;=&nbsp;{}
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;c&nbsp;in&nbsp;s:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;c&nbsp;in&nbsp;d:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d[c]&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d[c]&nbsp;=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;sorted(d.items(),&nbsp;key=lambda&nbsp;d:d[1])
&nbsp;&nbsp;&nbsp;&nbsp;low&nbsp;=&nbsp;ans[len(ans)&nbsp;-&nbsp;1][1]&nbsp;-&nbsp;3
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;one&nbsp;in&nbsp;ans:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;one[1]&nbsp;&gt;=&nbsp;low:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;search(base_s,&nbsp;deep&nbsp;+&nbsp;1,&nbsp;sec,&nbsp;key&nbsp;+&nbsp;chr((ord(one[0])&nbsp;-&nbsp;ord(&#39;E&#39;)&nbsp;+&nbsp;26)&nbsp;%&nbsp;26&nbsp;+&nbsp;ord(&#39;A&#39;)))</pre>

<p style="text-indent: 2em;">
  其中我架设解密文本中一定含有the，以尽量减少人工选择量，然后low可以根据个人感觉来设定，只有出现次数不小于low的才会被考虑当作e。
</p>

<p style="text-indent: 2em;">
  首先尝试长度为3：
</p>

<pre class="brush:python;toolbar:false">&gt;&gt;&gt;&nbsp;search(s2,&nbsp;0,&nbsp;3,&nbsp;&#39;&#39;)
(&#39;wsaoesamfiwcpedqched&#39;,&nbsp;&#39;KTC&#39;)
(&#39;wsvoenamaiwxpeyqcced&#39;,&nbsp;&#39;KTH&#39;)</pre>

<p style="text-indent: 2em;">
  很显然不太对，然后尝试长度为9，由于比较长，只选择一部分：
</p>

<pre class="brush:plain;toolbar:false">(&#39;whentsumeilgotdkcges&#39;,&nbsp;&#39;KEYLECQTD&#39;)
(&#39;whentsumailgotdkcces&#39;,&nbsp;&#39;KEYLECQTH&#39;)
(&#39;whentsumlilgotdkcnes&#39;,&nbsp;&#39;KEYLECQTW&#39;)
(&#39;whentsumrilgotdkctes&#39;,&nbsp;&#39;KEYLECQTQ&#39;)
(&#39;whentspmeilgotdfcges&#39;,&nbsp;&#39;KEYLECVTD&#39;)
(&#39;whentspmailgotdfcces&#39;,&nbsp;&#39;KEYLECVTH&#39;)
(&#39;whentspmlilgotdfcnes&#39;,&nbsp;&#39;KEYLECVTW&#39;)
(&#39;whentspmrilgotdfctes&#39;,&nbsp;&#39;KEYLECVTQ&#39;)
(&#39;whenthumeilgotskcges&#39;,&nbsp;&#39;KEYLENQTD&#39;)
(&#39;whenthumailgotskcces&#39;,&nbsp;&#39;KEYLENQTH&#39;)
(&#39;whenthumlilgotskcnes&#39;,&nbsp;&#39;KEYLENQTW&#39;)
(&#39;whenthumrilgotskctes&#39;,&nbsp;&#39;KEYLENQTQ&#39;)
(&#39;whenthpmeilgotsfcges&#39;,&nbsp;&#39;KEYLENVTD&#39;)
(&#39;whenthpmailgotsfcces&#39;,&nbsp;&#39;KEYLENVTH&#39;)
(&#39;whenthpmlilgotsfcnes&#39;,&nbsp;&#39;KEYLENVTW&#39;)
(&#39;whenthpmrilgotsfctes&#39;,&nbsp;&#39;KEYLENVTQ&#39;)
(&#39;wxeyjsumeibgzjdkcgei&#39;,&nbsp;&#39;KOYAOCQTD&#39;)
(&#39;wxeyjsumaibgzjdkccei&#39;,&nbsp;&#39;KOYAOCQTH&#39;)
(&#39;wxeyjsumlibgzjdkcnei&#39;,&nbsp;&#39;KOYAOCQTW&#39;)
(&#39;wxeyjsumribgzjdkctei&#39;,&nbsp;&#39;KOYAOCQTQ&#39;)
(&#39;wxeyjspmeibgzjdfcgei&#39;,&nbsp;&#39;KOYAOCVTD&#39;)
(&#39;wxeyjspmaibgzjdfccei&#39;,&nbsp;&#39;KOYAOCVTH&#39;)
(&#39;wxeyjspmlibgzjdfcnei&#39;,&nbsp;&#39;KOYAOCVTW&#39;)</pre>

<p style="text-indent: 2em;">
  其中的whenth部分看起来比较像when the，然后看对应的key，key似乎就应该是keylength。于是解密检查，确认密钥正确。
</p>

<p style="text-indent: 2em;">
  然后用此密钥，解密即可得到下一关的password。
</p>
