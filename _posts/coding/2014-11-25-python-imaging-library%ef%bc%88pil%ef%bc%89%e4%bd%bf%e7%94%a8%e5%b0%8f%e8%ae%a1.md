---
title: Python Imaging Library（PIL）使用小计
author: Ripples
layout: post
views:
  - 353
categories:
  - coding
tags:
  - PIL
  - python
---
<p style="text-indent: 2em;">
  由于作业需要做个给图像嵌入信息的程序，需要全自动，所以便开始倒腾PIL了（之前都是手动转BMP然后按照格式把其中的像素点信息取出来修改的）。
</p>

<p style="text-indent: 2em;">
  首先是安装，我是用的Mac OS，直接使用pip安装：
</p>

<!--more-->

<pre class="brush:bash;toolbar:false">sudo&nbsp;pip&nbsp;install&nbsp;PIL&nbsp;--allow-external&nbsp;PIL&nbsp;--allow-unverified&nbsp;PIL</pre>

<p style="text-indent: 2em;">
  如果不加&#8211;allow-external PIL &#8211;allow-unverified PIL，会出现下面两个报错：
</p>

<pre class="brush:plain;toolbar:false">Some&nbsp;externally&nbsp;hosted&nbsp;files&nbsp;were&nbsp;ignored&nbsp;(use&nbsp;--allow-external&nbsp;PIL&nbsp;to&nbsp;allow).

Some&nbsp;insecure&nbsp;and&nbsp;unverifiable&nbsp;files&nbsp;were&nbsp;ignored&nbsp;(use&nbsp;--allow-unverified&nbsp;PIL&nbsp;to&nbsp;allow).</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  安装好后就是使用了~~~
</p>

<p style="text-indent: 2em;">
  从文件中读取图像：
</p>

<pre class="brush:python;toolbar:false">import&nbsp;Image
im&nbsp;=&nbsp;Image.open("test.jpg")

fp&nbsp;=&nbsp;open("test.jpg",&nbsp;"rb")
im&nbsp;=&nbsp;Image.open(fp)

import&nbsp;StringIO
im&nbsp;=&nbsp;Image.open(StringIO.StringIO(buffer))

import&nbsp;TarIO
fp&nbsp;=&nbsp;TarIO.TarIO("Imaging.tar",&nbsp;"Imaging/test/test.jpg")
im&nbsp;=&nbsp;Image.open(fp)</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  查看图片格式等信息：
</p>

<pre class="brush:python;toolbar:false">print&nbsp;im.format,&nbsp;im.size,&nbsp;im.mode</pre>

<p style="text-indent: 2em;">
  其中format为图片原格式，size为图片大小，mode为图片模式（L为灰度图，RGB为像素图……）。
</p>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  查看图片：
</p>

<pre class="brush:python;toolbar:false">im.show()</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  保存图片：
</p>

<pre class="brush:python;toolbar:false">im.save(&#39;out.jpg&#39;)
im.save(&#39;out.thumbnail&#39;,&nbsp;&#39;JPEG&#39;)</pre>



<p style="text-indent: 2em;">
  裁剪图片：
</p>

<pre class="brush:python;toolbar:false">box&nbsp;=&nbsp;(100,&nbsp;100,&nbsp;400,&nbsp;400)
region&nbsp;=&nbsp;im.crop(box)</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  处理图片的一部分：
</p>

<pre class="brush:python;toolbar:false">region&nbsp;=&nbsp;region.transpose(Image.ROTATE_180)
im.paste(region,&nbsp;box)</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  将图片分解与合成：
</p>

<pre class="brush:python;toolbar:false">r,&nbsp;g,&nbsp;b&nbsp;=&nbsp;im.split()
im&nbsp;=&nbsp;Image.merge("RGB",&nbsp;(b,&nbsp;g,&nbsp;r))</pre>

<p style="text-indent: 2em;">
  需要注意的是要将图片先转换为RGB模式才能分解。
</p>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  改变图片大小：
</p>

<pre class="brush:python;toolbar:false">out&nbsp;=&nbsp;im.resize((128,&nbsp;128))</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  图片旋转：
</p>

<pre class="brush:python;toolbar:false">out&nbsp;=&nbsp;im.rotate(45)
out&nbsp;=&nbsp;im.transpose(Image.ROTATE_90)
out&nbsp;=&nbsp;im.transpose(Image.ROTATE_180)
out&nbsp;=&nbsp;im.transpose(Image.ROTATE_270)</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  图片翻转：
</p>

<pre class="brush:python;toolbar:false">out&nbsp;=&nbsp;im.transpose(Image.FLIP_LEFT_RIGHT)
out&nbsp;=&nbsp;im.transpose(Image.FLIP_TOP_BOTTOM)</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  图片模式转换：
</p>

<pre class="brush:python;toolbar:false">out&nbsp;=&nbsp;im.convert("L")</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  接下来是两个我最需要用到的函数：
</p>

<p style="text-indent: 2em;">
  获取<span style="text-indent: 32px;">字符串形式的</span>图片中的像素信息：
</p>

<pre class="brush:python;toolbar:false">im.tostring()</pre>

<p style="text-indent: 2em;">
  使用字符串形式的<span style="text-indent: 32px;">图片中的像素信息生成一张图片：</span>
</p>

<pre class="brush:python;toolbar:false">out&nbsp;=&nbsp;Image.fromstring(&#39;RGBA&#39;,&nbsp;im.size,&nbsp;im.convert(&#39;RGBA&#39;).tostring()</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  再附个PIL的<a href="http://effbot.org/imagingbook/image.htm" target="_blank">教程</a>。
</p>
