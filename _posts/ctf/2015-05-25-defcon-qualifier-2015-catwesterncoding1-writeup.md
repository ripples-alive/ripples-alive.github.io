---
title: Defcon qualifier 2015 catwestern(Coding1) WriteUp
author: Ripples
layout: post
permalink: /defcon-qualifier-2015-catwesterncoding1-writeup/
views:
  - 270
categories:
  - CTF
tags:
  - asm
  - CTF
  - DEFCON
  - WriteUp
---
<p style="text-align: left; text-indent: 2em;">
  这题作为这次defcon预选赛中唯一的一道coding题，还是1分，很显然是送分题了，直接连上服务器可以得到如下返回：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false;">$&nbsp;nc&nbsp;catwestern_631d7907670909fc4df2defc13f2057c.quals.shallweplayaga.me&nbsp;9999
****Initial&nbsp;Register&nbsp;State****
rax=0xfcf7659c7a4ad096
rbx=0x1df0e8dfe8f70b53
rcx=0x55004165472b9655
rdx=0x1aa98e77006adf1
rsi=0x949a482579724b11
rdi=0x1e671d7b7ef9430
r8=0x3251192496cee6a6
r9=0x278d01e964b0efc8
r10=0x1c5c8cca5112ad12
r11=0x75a01cef4514d4f5
r12=0xe109fd4392125cc7
r13=0xe5e33405335ba0ff
r14=0x633e16d0ec94137
r15=0xb80a585e0cd42415
****Send&nbsp;Solution&nbsp;In&nbsp;The&nbsp;Same&nbsp;Format****
About&nbsp;to&nbsp;send&nbsp;74&nbsp;bytes:&nbsp;
hŒråRI‡ÔA]HÿÊIÇ¢éNhIÿÊHÿÃHÎIÇ^…6H¤Ã
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MÃI÷ëH)ðHÆQØ8eHÿÀIÁÕH5Œm&#39;Ã^C</pre>

<p style="text-align: left; text-indent: 2em;">
  即这里是给出了一些寄存器的初始状态，让我们执行下面给的机器指令，那显然，相比于写个解释器模拟执行，我们写个程序直接能运行这段指令是最好的。这里如果用汇编写可能会方便点吧，不过鉴于好久没写过汇编的了，用C++写了个内嵌汇编的：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;zio
import&nbsp;pwn
import&nbsp;commands

TARGET&nbsp;=&nbsp;(&#39;catwestern_631d7907670909fc4df2defc13f2057c.quals.shallweplayaga.me&#39;,&nbsp;9999)


io&nbsp;=&nbsp;zio.zio(TARGET)
io.readline()
regs&nbsp;=&nbsp;io.read_until([&#39;****Send&nbsp;Solution&nbsp;In&nbsp;The&nbsp;Same&nbsp;Format****&#39;]).split(&#39;n&#39;)[:-1]

while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;io.read_until(&#39;bytes:&nbsp;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;ops&nbsp;=&nbsp;io.read_until_timeout()
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;ops[-1]&nbsp;!=&nbsp;&#39;xc3&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;"Error,&nbsp;not&nbsp;end&nbsp;with&nbsp;xc3."
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(1)
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;len(ops)

&nbsp;&nbsp;&nbsp;&nbsp;regs&nbsp;=&nbsp;[x.split(&#39;=&#39;)&nbsp;for&nbsp;x&nbsp;in&nbsp;regs]
&nbsp;&nbsp;&nbsp;&nbsp;ops&nbsp;=&nbsp;&#39;x48xb8&#39;&nbsp;+&nbsp;zio.l64(int(regs[0][1],&nbsp;16))&nbsp;+&nbsp;ops

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;headers
&nbsp;&nbsp;&nbsp;&nbsp;cpp&nbsp;=&nbsp;open(&#39;run.cpp&#39;,&nbsp;&#39;w&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;#include&nbsp;&lt;stdio.h&gt;

&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;namespace&nbsp;std;

&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;main()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*ops&nbsp;=&nbsp;"%s";
&nbsp;&nbsp;&nbsp;&nbsp;&#39;&#39;&#39;&nbsp;%&nbsp;&#39;&#39;.join([&#39;\x%02x&#39;&nbsp;%&nbsp;ord(x)&nbsp;for&nbsp;x&nbsp;in&nbsp;ops]))

&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;x&nbsp;in&nbsp;regs:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;if&nbsp;int(x[1],&nbsp;16)&nbsp;&gt;&nbsp;int(&#39;0xffffffff&#39;,&nbsp;16):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;x[1]&nbsp;=&nbsp;str(int(x[1],&nbsp;16)&nbsp;&&nbsp;0xffffffff)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;__asm__("mov&nbsp;$%s,&nbsp;%%%s");n&#39;&nbsp;%&nbsp;(x[1],&nbsp;x[0]))

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;pwn.o
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;codes&nbsp;=&nbsp;pwn.disasm(ops).split(&#39;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;for&nbsp;x&nbsp;in&nbsp;codes:
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;__asm__("%s");n&#39;&nbsp;%&nbsp;x.split(&#39;&nbsp;&#39;&nbsp;*&nbsp;6)[-1].strip())
&nbsp;&nbsp;&nbsp;&nbsp;cpp.write("((void&nbsp;(*)())ops)();n")

&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;x&nbsp;in&nbsp;regs:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;__asm__("push&nbsp;%%%s");n&#39;&nbsp;%&nbsp;x[0])

&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&nbsp;6
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;x&nbsp;in&nbsp;regs:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;cpp.write(&#39;printf("%%%d$p\n");n&#39;&nbsp;%&nbsp;i)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;printf("0x%%%d$llx\n");n&#39;&nbsp;%&nbsp;i)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;cpp.write(&#39;printf("0x%%%d$016llx\n");n&#39;&nbsp;%&nbsp;i)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;+=&nbsp;1

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;footers
&nbsp;&nbsp;&nbsp;&nbsp;cpp.write(&#39;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&#39;&#39;&#39;)

&nbsp;&nbsp;&nbsp;&nbsp;cpp.close()

&nbsp;&nbsp;&nbsp;&nbsp;commands.getstatusoutput(&#39;g++&nbsp;-z&nbsp;execstack&nbsp;-fno-stack-protector&nbsp;-Wall&nbsp;-o&nbsp;./run&nbsp;./run.cpp&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;status,&nbsp;output&nbsp;=&nbsp;commands.getstatusoutput(&#39;./run&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;output&nbsp;=&nbsp;output.split(&#39;n&#39;)[::-1]

&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&nbsp;0
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;x&nbsp;in&nbsp;regs:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;output[i]&nbsp;=&nbsp;x[0]&nbsp;+&nbsp;&#39;=&#39;&nbsp;+&nbsp;output[i]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;+=&nbsp;1

&nbsp;&nbsp;&nbsp;&nbsp;io.writelines(output)
&nbsp;&nbsp;&nbsp;&nbsp;io.readline()
#&nbsp;print&nbsp;io.readline()
#&nbsp;print&nbsp;io.read_until_timeout()</pre>

<p style="text-align: left; text-indent: 2em;">
  这里中间产生的C++代码类似：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;cstdio&gt;

using&nbsp;namespace&nbsp;std;

int&nbsp;main()
{
char&nbsp;*ops&nbsp;=&nbsp;"……";
__asm__("mov&nbsp;$0x365a6103367515ed,&nbsp;%rax");
__asm__("mov&nbsp;$0x3cb1a6fff5b038b6,&nbsp;%rbx");
__asm__("mov&nbsp;$0x2a5e4d432e3810e7,&nbsp;%rcx");
__asm__("mov&nbsp;$0x3fc79069d112d35e,&nbsp;%rdx");
__asm__("mov&nbsp;$0xe9b6862f68ddc08b,&nbsp;%rsi");
__asm__("mov&nbsp;$0xc9bb5638f34a4ed2,&nbsp;%rdi");
__asm__("mov&nbsp;$0x1eea6b2bcfa19f55,&nbsp;%r8");
__asm__("mov&nbsp;$0xb48d0fe2a6217fcd,&nbsp;%r9");
__asm__("mov&nbsp;$0x7ff063bcf81b25d7,&nbsp;%r10");
__asm__("mov&nbsp;$0x7ecfd1750b003532,&nbsp;%r11");
__asm__("mov&nbsp;$0x2e45b91b25474403,&nbsp;%r12");
__asm__("mov&nbsp;$0xb9661518b82c711,&nbsp;%r13");
__asm__("mov&nbsp;$0x3f397cafd35489db,&nbsp;%r14");
__asm__("mov&nbsp;$0x1d0dc0438a2a9b1f,&nbsp;%r15");
((void&nbsp;(*)())ops)();
__asm__("push&nbsp;%rax");
__asm__("push&nbsp;%rbx");
__asm__("push&nbsp;%rcx");
__asm__("push&nbsp;%rdx");
__asm__("push&nbsp;%rsi");
__asm__("push&nbsp;%rdi");
__asm__("push&nbsp;%r8");
__asm__("push&nbsp;%r9");
__asm__("push&nbsp;%r10");
__asm__("push&nbsp;%r11");
__asm__("push&nbsp;%r12");
__asm__("push&nbsp;%r13");
__asm__("push&nbsp;%r14");
__asm__("push&nbsp;%r15");
printf("%6$pn");
printf("%7$pn");
printf("%8$pn");
printf("%9$pn");
printf("%10$pn");
printf("%11$pn");
printf("%12$pn");
printf("%13$pn");
printf("%14$pn");
printf("%15$pn");
printf("%16$pn");
printf("%17$pn");
printf("%18$pn");
printf("%19$pn");

&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-align: left; text-indent: 2em;">
  即先将寄存器初始设置好，然后运行所给指令，然后将所有寄存器压栈，通过printf输出。然后这里要注意的是，所给指令最后一句几乎可以肯定说是ret，所以可以很方便的直接call上去就好，但是由于编译出来call的时候，是先将地址移到rax中，然后再call，所以说我们需要在所给指令前面加上一段指令重新设置rax的值，也即上面Python代码中的：
</p>

<pre class="brush:python;toolbar:false">ops&nbsp;=&nbsp;&#39;x48xb8&#39;&nbsp;+&nbsp;zio.l64(int(regs[0][1],&nbsp;16))&nbsp;+&nbsp;ops</pre>

<p style="text-align: left; text-indent: 2em;">
  然后这题比较蛋疼的是，最开始主办方题目弄错了点啥，搞得一直过不了，各种无语，最关键的是，当时题目有问题的情况下有队过了，所以无奈啊……
</p>
