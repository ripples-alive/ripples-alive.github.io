---
title: 中软吉大•问鼎杯决赛破解类（一）
author: Ripples
layout: post
permalink: /中软吉大•问鼎杯决赛破解类（一）/
views:
  - 493
categories:
  - CTF
tags:
  - krypton
  - WriteUp
  - 中软吉大•问鼎杯
---
<p style="text-indent: 2em;">
  这题我拿到手的是一个<a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/2.1xcode.zip" target="_self">压缩文件</a>。
</p>

<p style="text-indent: 2em;">
  首先，解压这个文件我们得到一个文本文档，由于太长，此处就不贴出来了。
</p>

<p style="text-indent: 2em;">
  看到这么长的一个文件，最后还有一个=号，让我们不禁就会往base64上想，那么我们首先就用base64对其进行解码看看：
</p>

<!--more-->

<pre class="brush:bash;toolbar:false">base64&nbsp;-D&nbsp;2.1xcode.txt&nbsp;&gt;&nbsp;tmp</pre>

<p style="text-indent: 2em;">
  这个时候，如果电脑智能的话，可以自己识别出这个文件的类型的，如果电脑没有识别出来，我们可以直接用编辑器查看这个文件，我们会发现文件的开头有PNG字样，那么这应该就是个png图像文件了，改下后缀就好了。
</p>

<p style="text-indent: 2em;">
  得到图像如下：
</p>

<p style="text-indent: 2em;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/1.png" title="1.png" />
</p>

<p style="text-indent: 2em;">
  如此看下来的话，这题其实也不难，也是属于可以秒破的级别。
</p>

* * *

<p style="text-indent: 2em;">
  昨天刚听学长说，这题在扫描这张二维码之后好像给的是个很简单的密码还是什么的来着的，反正也很简单就是了，不过据说题目坑了，一直拿到了答案却不知道是答案，知道最后主办方给提示才搞定。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/2.1xcode.zip">2.1xcode.zip</a>
</p>
