---
title: 'NCTU UBC crackme #1 解题报告'
author: Ripples
layout: post
permalink: /nctu-ubc-crackme-1-%e8%a7%a3%e9%a2%98%e6%8a%a5%e5%91%8a/
views:
  - 369
categories:
  - CTF
tags:
  - NCTU
  - reversing
  - WriteUp
---
<p style="text-indent: 2em;">
  这道题卡了我好久好久，知道今天才好不容易得以解决，题目描述如下：
</p>

<pre class="brush:plain;toolbar:false">UBC&nbsp;crackme,&nbsp;

simple&nbsp;but&nbsp;you&nbsp;have&nbsp;to&nbsp;find&nbsp;the&nbsp;critical&nbsp;"cmp"&nbsp;first...

press&nbsp;"start"&nbsp;then&nbsp;download&nbsp;it.

Hint:&nbsp;The&nbsp;checkin&nbsp;key&nbsp;is&nbsp;serial&nbsp;of&nbsp;"HelloWorld"</pre>

<!--more-->

<p style="text-indent: 2em;">
  首先拿到这道题的程序，我本能的第一反应就是在MessageBox上加断点，然后运行的时候，IDA给谈了个窗，我直接没太注意就确定了，可谁想到，就是这不经意的一下，让我掉入了深坑……
</p>

<p style="text-indent: 2em;">
  点完确定后，IDA就被挂在MessageBox上的断点停住了，一切看起来都很正常，可是，就这样找了好久，一直找不到关键代码，于是也便曾放下了好久，知道今天又拿起来做才终于知道自己怎么掉坑里的了。
</p>

<p style="text-indent: 2em;">
  仔细看下IDA给的那个提示框：
</p>

<p style="text-align: center;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/blob1.png" />
</p>

<p style="text-indent: 2em;">
  提示框是说exception发生了，是否传递给程序，这样看起来，我们是进了程序的异常处理部分，这也就难怪我们一步步的像上追踪也没有找到要找的地方。
</p>

<p style="text-indent: 2em;">
  仔细看下程序给出的提示：
</p>

<p style="text-align: center;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/blob2.png" />
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">这是要让我们在serial里输入数字，那么我们就按照要求来做，看能不能不进入异常处理部分。</span>
</p>

<p style="text-indent: 2em;">
  可是，在我们输入的是数字后，程序却没有被我们放在MessageBox上的断点中断住，直接就正常运行了，不过这至少说明，程序没有进入异常处理部分，我们的方向是对的。
</p>

<p style="text-indent: 2em;">
  接下来就要考虑如何下断点了，我想在获取文本框内容的函数上下断点，可却没找到是哪个函数，然后就想既然MessageBox没用，那应该就是直接产生的一个普通窗口，然后就想到了ShowWindow，下断点，重新运行程序，发现，程序一上来就被暂停了，应该是主窗体的显示，无视就好，然后接下来输入完成后点check it发现程序果然被我们ShowWindow上的断点暂停住了，跟踪下去很轻松就找到了关键代码：
</p>

<pre class="brush:cpp;toolbar:false">int&nbsp;__usercall&nbsp;sub_458800&lt;eax&gt;(int&nbsp;a1&lt;ebx&gt;)
{
&nbsp;&nbsp;int&nbsp;v1;&nbsp;//&nbsp;ecx@1
&nbsp;&nbsp;int&nbsp;v2;&nbsp;//&nbsp;ecx@2
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v4;&nbsp;//&nbsp;[sp-10h]&nbsp;[bp-14h]@1
&nbsp;&nbsp;int&nbsp;v5;&nbsp;//&nbsp;[sp-4h]&nbsp;[bp-8h]@4
&nbsp;&nbsp;int&nbsp;v6;&nbsp;//&nbsp;[sp+0h]&nbsp;[bp-4h]@1
&nbsp;&nbsp;int&nbsp;v7;&nbsp;//&nbsp;[sp+4h]&nbsp;[bp+0h]@1

&nbsp;&nbsp;v4&nbsp;=&nbsp;__readfsdword(0);
&nbsp;&nbsp;__writefsdword(0,&nbsp;(unsigned&nbsp;int)&v4);
&nbsp;&nbsp;sub_458760(v4,&nbsp;loc_458874,&nbsp;&v7,&nbsp;a1,&nbsp;0);
&nbsp;&nbsp;sub_4255C0(v1,&nbsp;&v6);
&nbsp;&nbsp;if&nbsp;(&nbsp;sub_407774()&nbsp;==&nbsp;dword_45B844&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;sub_44496C();
&nbsp;&nbsp;&nbsp;&nbsp;sub_4255F0(v2,&nbsp;"CRACKED");
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;sub_44496C();
&nbsp;&nbsp;}
&nbsp;&nbsp;__writefsdword(0,&nbsp;v4);
&nbsp;&nbsp;return&nbsp;sub_4037E8(loc_45887B,&nbsp;v5);
}</pre>

<p style="text-indent: 2em;">
  跟踪调试一下可以发现，sub_4255C0是用来将输入框的内容读入，v7中存的是指向读入的name字符串的指针，v6中存的是指向读入的serial字符串的指针，sub_407774是将serial字符串转为整数，转换失败是抛出异常，也就会进入我们之前掉入的那个坑里。
</p>

<p style="text-indent: 2em;">
  sub_458760中调用sub_4255C0读入了name字符串并做了一定的处理，观察会感觉<span style="text-indent: 32px;">sub_458760还是比较麻烦的，于是乎，再按照之前的老方法，在if ( sub_407774() == dword_45B844 )上加断点，直接看dword_45B844的值（填入的name为HelloWorld，题目中有说），然后输入到serial中即可。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">成功得到正确的提示框，此题得以解决，正如题目中所说，这题的关键在于找到正确的cmp，只要找到了，基本做就不成问题了。</span>
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/ubccrackme1.rar">ubccrackme1.rar</a>
</p>
