---
title: 中软吉大•问鼎杯决赛取证类（一）
author: Ripples
layout: post
permalink: /中软吉大•问鼎杯决赛取证类（一）/
views:
  - 559
categories:
  - CTF
tags:
  - WriteUp
  - 中软吉大•问鼎杯
  - 信息隐藏
  - 取证
---
<p style="text-indent: 2em;">
  这道题我拿到手的就是一张<a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/3.1.jpg" target="_blank">图片</a>。
</p>

<p style="text-indent: 2em;">
  按照惯例，首先直接查看下文件的内容，利用strings命令提取出可见字符：
</p>

<pre class="brush:bash;toolbar:false">strings&nbsp;-10&nbsp;3.1.jpg&nbsp;|&nbsp;less</pre>

<p style="line-height: 16px; text-indent: 2em;">
  可以发现，在最后部分，有这么一段内容：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">Q^DgC)io&nbsp;v
bsR--%|U8IL
*~}(w/mUnn
5uQERcRjx`
eg59&lt;rx~^]cK
MTk3MsTqMtTCMjHI1aOsw8C5+tfczbPE4b/Ly8m3w87KsbG+qaGj1eLKx9bQw8DBvbn6t8fV/bOju6/H6b/2z8K1xLXa0ru0ztSqyte8trfDzsqho7WxzOzPws7no6zDq9b3z6/U2tSiy/m74bz7wcvE4b/Ly8mho7C01dXLq7e9ysLHsNS8tqijrLTLtM674cy4xNrI3b371rnN4tC5oaO74cy4veHK+Lrzo6zW0Le9vau74cy4vMfCvNf3zqq++MPczsS8/qGjDQpXREZMQUd7dGhlIGludGVsbGlnZW5jZSBhYm91dCBBbWVyaWNhIFByZXNpZGVudCBOaXhvbiB2aXNpdGluZyBCZWlqaW5nfQ==
"!&+7/&)4)!"0A149;&gt;&gt;&gt;%.DIC&lt;H7=&gt;;
%&&#39;()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
]dYb5FDc&lt;
bwqJd-(E&lt;U</pre>

<p style="line-height: 16px; text-indent: 2em;">
  很明显，我们会发现期中有一段特别显眼，以两个＝号结束，这不禁又让我们想到了base64：
</p>

<pre class="brush:bash;toolbar:false">➜&nbsp;&nbsp;echo&nbsp;"MTk3MsTqMtTCMjHI1aOsw8C5+tfczbPE4b/Ly8m3w87KsbG+qaGj1eLKx9bQw8DBvbn6t8fV/bOju6/H6b/2z8K1xLXa0ru0ztSqyte8trfDzsqho7WxzOzPws7no6zDq9b3z6/U2tSiy/m74bz7wcvE4b/Ly8mho7C01dXLq7e9ysLHsNS8tqijrLTLtM674cy4xNrI3b371rnN4tC5oaO74cy4veHK+Lrzo6zW0Le9vau74cy4vMfCvNf3zqq++MPczsS8/qGjDQpXREZMQUd7dGhlIGludGVsbGlnZW5jZSBhYm91dCBBbWVyaWNhIFByZXNpZGVudCBOaXhvbiB2aXNpdGluZyBCZWlqaW5nfQ=="&nbsp;|&nbsp;base64&nbsp;-D
1972��2��21�գ������ͳ����ɷ��ʱ�����������������������µĵ�һ��Ԫ�׼����ʡ��������磬ë��ϯ��Ԣ����������ɡ�����˫����ǰԼ�����˴λ����ݽ���й����������з�������¼��Ϊ�����ļ���
WDFLAG{the&nbsp;intelligence&nbsp;about&nbsp;America&nbsp;President&nbsp;Nixon&nbsp;visiting&nbsp;Beijing}%</pre>

<p style="line-height: 16px; text-indent: 2em;">
  很明显，在解出来内容的末尾就是我们想要的flag了，于是这题也得以解决，这个信息隐藏程度只能说至少比样题要稍微高点了。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_default.png" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/3.1.jpg">3.1.jpg</a>
</p>
