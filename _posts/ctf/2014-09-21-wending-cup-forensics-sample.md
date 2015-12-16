---
title: 中软吉大•问鼎杯决赛样题取证类
author: Ripples
layout: post
views:
  - 1034
categories:
  - CTF
tags:
  - WriteUp
  - 中软吉大•问鼎杯
  - 信息隐藏
  - 取证
---
<p style="text-indent: 2em;">
  取证类样题一共两道题，题目如下：
</p>

<p style="text-indent: 2em;">
  1、从一个磁盘镜像（DD格式）中恢复一个有关火星登陆探索车的JPEG文件，并递交其MD5值。<a href="http://sec.hdu.edu.cn/files/5.1.Image.rar" style="box-sizing: border-box; margin: 0px; padding: 0px; border: 0px; font-family: arial, helvetica, sans-serif; line-height: 24px; vertical-align: baseline; text-decoration: underline; white-space: normal; color: red;"><span style="box-sizing: border-box; color: red; font-family: arial, helvetica, sans-serif;">文件下载</span></a>（200分）
</p>

<p style="text-indent: 2em;">
  2、图片里的奥秘。<a href="http://sec.hdu.edu.cn/files/5.2.key.jpg" style="box-sizing: border-box; margin: 0px; padding: 0px; border: 0px; font-family: arial, helvetica, sans-serif; line-height: 24px; vertical-align: baseline; text-decoration: underline; white-space: normal; color: red;"><span style="box-sizing: border-box; color: red; font-family: arial, helvetica, sans-serif;">文件下载</span></a>（200分）
</p>

<!--more-->

<p style="text-indent: 2em;">
  本来对于取证是完全不会的，不过不得不说这两道题确实是太简单。
</p>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  第一题是要从中找出JPEG，个人不知道怎么直接还原，但是观察会发现，里面的内容是没有压缩的，故可以直接从文件中找出JPEG的部分。
</p>

<p style="text-indent: 2em;">
  首先，找到JPEG的格式，可参见<a href="http://blog.csdn.net/lpt19832003/article/details/1713718" target="_blank">这里</a>。前四个字节是0xffd8ffe0，第7-11个字节是0x4A46494600（JFIF），结束是0xffd9。
</p>

<p style="text-indent: 2em;">
  如果我们直接找到这个开头，然后找其后第一个结尾，部分图片是对的，但有部分确实错误的，因为在APPn段中，也可以存储一份微缩图像，微缩图像也是同样的开头和结尾。特殊判断处理一下即可：
</p>

<pre class="brush:python;toolbar:false">import&nbsp;re

fp&nbsp;=&nbsp;open(&#39;Image.raw&#39;,&nbsp;&#39;rb&#39;)
content&nbsp;=&nbsp;fp.read()
fp.close()

cnt&nbsp;=&nbsp;0
start&nbsp;=&nbsp;0
while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;pic_start&nbsp;=&nbsp;content.find(&#39;xffxd8xffxe0&#39;,&nbsp;start)
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;pic_start&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break
&nbsp;&nbsp;&nbsp;&nbsp;start&nbsp;=&nbsp;pic_start&nbsp;+&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;first_start&nbsp;=&nbsp;content.find(&#39;xffxd8xffxe0&#39;,&nbsp;start)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;first_end&nbsp;=&nbsp;content.find(&#39;xffxd9&#39;,&nbsp;start)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;first_start&nbsp;!=&nbsp;-1&nbsp;and&nbsp;first_start&nbsp;&lt;&nbsp;first_end:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;start&nbsp;=&nbsp;first_end&nbsp;+&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break
&nbsp;&nbsp;&nbsp;&nbsp;pic_end&nbsp;=&nbsp;first_end
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;cnt&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;fp&nbsp;=&nbsp;open(&#39;%d.jpg&#39;&nbsp;%&nbsp;cnt,&nbsp;&#39;wb&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;fp.write(content[pic_start:pic_end&nbsp;+&nbsp;2])
&nbsp;&nbsp;&nbsp;&nbsp;fp.close()
&nbsp;&nbsp;&nbsp;&nbsp;start&nbsp;=&nbsp;pic_end&nbsp;+&nbsp;1</pre>

<p style="text-indent: 2em;">
  于是得到<a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/1.jpg" target="_blank">此图</a>应该为题目中所需图片，计算MD5即可。
</p>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  第二题就更水了，看起来似乎挺高端的样子，但其实，只要直接把文件当文本文件查看，便会发现其中直接写着flag，这种信息隐藏程度真是让人无话可说。
</p>

<p style="text-indent: 2em;">
  以下是xxd的部分结果：
</p>

<pre class="brush:plain;toolbar:false">0000140:&nbsp;0000&nbsp;0001&nbsp;0000&nbsp;0048&nbsp;0000&nbsp;0001&nbsp;ffd8&nbsp;ffe0&nbsp;&nbsp;.......H........
0000150:&nbsp;0010&nbsp;4a46&nbsp;4946&nbsp;0001&nbsp;0200&nbsp;0048&nbsp;0048&nbsp;0000&nbsp;&nbsp;..JFIF.....H.H..
0000160:&nbsp;ffed&nbsp;000c&nbsp;4164&nbsp;6f62&nbsp;655f&nbsp;434d&nbsp;0001&nbsp;ffee&nbsp;&nbsp;....Adobe_CM....
0000170:&nbsp;000e&nbsp;4164&nbsp;6f62&nbsp;6500&nbsp;6480&nbsp;0000&nbsp;0001&nbsp;ffdb&nbsp;&nbsp;..Adobe.d.......
0000180:&nbsp;0084&nbsp;000c&nbsp;0808&nbsp;0809&nbsp;080c&nbsp;0909&nbsp;0c11&nbsp;0b0a&nbsp;&nbsp;................
0000190:&nbsp;666c&nbsp;6167&nbsp;3d73&nbsp;6563&nbsp;2e68&nbsp;6475&nbsp;2e65&nbsp;6475&nbsp;&nbsp;flag=sec.hdu.edu
00001a0:&nbsp;2e63&nbsp;6e0c&nbsp;0c0c&nbsp;110c&nbsp;0c0c&nbsp;0c0c&nbsp;0c0c&nbsp;0c0c&nbsp;&nbsp;.cn.............</pre>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  最后，今天把初赛做了，已经彻底和决赛说再见了，之前规则明明写着有25道题，结果突然变成了100道题，然后毫不知情的情况下就慢悠悠的做，到最后十来分钟，发现怎么还有第26题，还当系统出了问题，最后却发现外面规则写着有100道题，已经无话可说了，到最后也就做了30题左右的，这就是直接不看题蒙期望也是25分，这还怎么玩。太伤心了，本来还说决赛能好好玩玩的，没想到竟是这样收场，最近真是诸事不顺……
</p>

<p style="text-indent: 2em;">
  一个人跑去帮同学做了下初赛，竟然都把他们送进了决赛，对这比赛无语了……
</p>

<p style="line-height: 16px;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" width="16" height="16" border="0" hspace="0" vspace="0" title="" style="width: 16px; height: 16px;" /><a href="http://pan.baidu.com/s/1eQqxHFc" target="_blank" textvalue="5.1.Image.rar">5.1.Image.rar</a>
</p>

<p style="line-height: 16px;">
  <a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/5.2.key_.jpg"><img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_default.png" style="line-height: 16px; white-space: normal;" /></a><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/5.2.key_.jpg" style="line-height: 16px; white-space: normal;">5.2.key.jpg</a>
</p>
