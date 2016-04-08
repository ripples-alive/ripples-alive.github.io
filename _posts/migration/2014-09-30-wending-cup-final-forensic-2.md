---
title: 中软吉大•问鼎杯决赛取证类（二）
author: Ripples
layout: post
permalink: /中软吉大•问鼎杯决赛取证类（二）/
views:
  - 543
categories:
  - CTF
tags:
  - WriteUp
  - 中软吉大•问鼎杯
  - 取证
---
<p style="text-indent: 2em;">
  这一题的题目由“丁同学”提供，特此表示感谢，题目如下：
</p>

<pre class="brush:plain;toolbar:false">2.（取证）从一个磁盘镜像（DD格式）中恢复一个有关于“A&nbsp;hall&nbsp;in&nbsp;DUKE&nbsp;SOLINUS&#39;S&nbsp;palace”的html文件。递交KEY为其MD5值。（500分）</pre>

<p style="text-indent: 2em;">
  这道题的文件和取证样题中的文件竟然又是一模一样，不过好在题目要取的东西不一样了，也不完全算原题了。
</p>

<!--more-->

<p style="text-indent: 2em;">
  首先，我们在文件中查找DUKE，会发现一段看起来像HTML的内容，然后向前找到其开头处：
</p>

<pre class="brush:plain;toolbar:false">0d6cff0:&nbsp;dd3d&nbsp;2f42&nbsp;f816&nbsp;80dd&nbsp;b3a0&nbsp;78b6&nbsp;9258&nbsp;1d1b&nbsp;&nbsp;.=/B......x..X..
0d6d000:&nbsp;3c21&nbsp;444f&nbsp;4354&nbsp;5950&nbsp;4520&nbsp;4854&nbsp;4d4c&nbsp;2050&nbsp;&nbsp;&lt;!DOCTYPE&nbsp;HTML&nbsp;P
0d6d010:&nbsp;5542&nbsp;4c49&nbsp;4320&nbsp;222d&nbsp;2f2f&nbsp;5733&nbsp;432f&nbsp;2f44&nbsp;&nbsp;UBLIC&nbsp;"-//W3C//D
0d6d020:&nbsp;5444&nbsp;2048&nbsp;544d&nbsp;4c20&nbsp;342e&nbsp;3020&nbsp;5472&nbsp;616e&nbsp;&nbsp;TD&nbsp;HTML&nbsp;4.0&nbsp;Tran
0d6d030:&nbsp;7369&nbsp;7469&nbsp;6f6e&nbsp;616c&nbsp;2f2f&nbsp;454e&nbsp;220a&nbsp;2022&nbsp;&nbsp;sitional//EN".&nbsp;"
0d6d040:&nbsp;6874&nbsp;7470&nbsp;3a2f&nbsp;2f77&nbsp;7777&nbsp;2e77&nbsp;332e&nbsp;6f72&nbsp;&nbsp;http://www.w3.or
0d6d050:&nbsp;672f&nbsp;5452&nbsp;2f52&nbsp;4543&nbsp;2d68&nbsp;746d&nbsp;6c34&nbsp;302f&nbsp;&nbsp;g/TR/REC-html40/
0d6d060:&nbsp;6c6f&nbsp;6f73&nbsp;652e&nbsp;6474&nbsp;6422&nbsp;3e0a&nbsp;203c&nbsp;6874&nbsp;&nbsp;loose.dtd"&gt;.&nbsp;&lt;ht</pre>

<p style="text-indent: 2em;">
  很显然，这个文件是从0x0d6d000处开始了，然后再从此处开始，查找</html>，找到其结尾处：
</p>

<pre class="brush:plain;toolbar:false">0dc4820:&nbsp;3c2f&nbsp;413e&nbsp;3c62&nbsp;723e&nbsp;0a3c&nbsp;703e&nbsp;3c69&nbsp;3e45&nbsp;&nbsp;&lt;/A&gt;&lt;br&gt;.&lt;p&gt;&lt;i&gt;E
0dc4830:&nbsp;7865&nbsp;756e&nbsp;743c&nbsp;2f69&nbsp;3e3c&nbsp;2f70&nbsp;3e0a&nbsp;3c2f&nbsp;&nbsp;xeunt&lt;/i&gt;&lt;/p&gt;.&lt;/
0dc4840:&nbsp;626f&nbsp;6479&nbsp;3e0a&nbsp;3c2f&nbsp;6874&nbsp;6d6c&nbsp;3e0a&nbsp;0a8f&nbsp;&nbsp;body&gt;.&lt;/html&gt;...
0dc4850:&nbsp;a9f4&nbsp;343b&nbsp;01b4&nbsp;32e1&nbsp;6544&nbsp;b7dd&nbsp;0e41&nbsp;cb8b&nbsp;&nbsp;..4;..2.eD...A..</pre>

<p style="text-indent: 2em;">
  于是乎，我们可以很轻松的提取出从开头到结尾的内容。
</p>

<p style="text-indent: 2em;">
  然后打开我们提取出来的文件，我们会发现其中有一段内容是乱码的，由于没法测试，我也不知道这段乱码是否应该除去。
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">联系HTML的内容，我们可以发现</span>这段乱码是从<span style="text-indent: 32px;">0x</span><span style="text-indent: 32px;">0d7ae00到0x0da93ff，具体</span>如下：
</p>

<pre class="brush:plain;toolbar:false">0d7ae00:&nbsp;ffd8&nbsp;ffe0&nbsp;0010&nbsp;4a46&nbsp;4946&nbsp;0001&nbsp;0101&nbsp;0064&nbsp;&nbsp;......JFIF.....d
……
0da9250:&nbsp;3eb9&nbsp;fce7&nbsp;f39f&nbsp;ce7f&nbsp;39f5&nbsp;cfaf&nbsp;ffd9&nbsp;540e&nbsp;&nbsp;&gt;.......9.....T.
……
0da9400:&nbsp;323e&nbsp;5b57&nbsp;6974&nbsp;6869&nbsp;6e5d&nbsp;2020&nbsp;5269&nbsp;6768&nbsp;&nbsp;2&gt;[Within]&nbsp;&nbsp;Righ</pre>

<p style="text-indent: 2em;">
  其中，0x0d7ae00到0x0da925d为一张JPG图片，其余内容暂不太清楚是什么。
</p>

<p style="text-indent: 2em;">
  鉴于没有办法测试，这题就先这样了。
</p>

<p style="text-indent: 2em;">
  PS：据题目提供者称，提取出来的文件最后</html>后面要有换行，也就是那个0x0a别落了。
</p>

<p style="text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" width="16" height="16" border="0" hspace="0" vspace="0" title="" style="line-height: 16px; white-space: normal; width: 16px; height: 16px;" /><a href="http://pan.baidu.com/s/1eQqxHFc" target="_blank" textvalue="5.1.Image.rar" style="line-height: 16px; white-space: normal;">5.1.Image.rar</a>
</p>
