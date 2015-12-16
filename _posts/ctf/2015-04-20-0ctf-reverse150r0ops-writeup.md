---
title: 0CTF Reverse150(r0ops) WriteUp
author: Ripples
layout: post
views:
  - 267
categories:
  - CTF
tags:
  - 0CTF
  - reversing
  - ROP
  - WriteUp
---
<p style="text-indent: 2em;">
  这几次比赛下来，真的是深刻的让我感受到了自己逆向水平的不足，这次的0ctf也更是好不容易在第二天下午才发现这题怎么做，才总算没有在逆向题上无功而返。
</p>

<p style="text-indent: 2em;">
  这道题说实在的，确实是挺简单的，不过很麻烦倒是真的，唯独的一个问题就是不能用Hex-rays来看，反编译得到的代码是有问题的。
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">signed&nbsp;__int64&nbsp;__usercall&nbsp;handle@&lt;rax&gt;(__int64&nbsp;a1@&lt;rbp&gt;)
{
&nbsp;&nbsp;__int64&nbsp;*src;&nbsp;//&nbsp;rsi@1
&nbsp;&nbsp;__int64&nbsp;*dst;&nbsp;//&nbsp;rdi@1
&nbsp;&nbsp;signed&nbsp;__int64&nbsp;i;&nbsp;//&nbsp;rcx@1

&nbsp;&nbsp;*(_DWORD&nbsp;*)(a1&nbsp;-&nbsp;4)&nbsp;=&nbsp;accept(3,&nbsp;0LL,&nbsp;0LL);
&nbsp;&nbsp;recv(*(_DWORD&nbsp;*)(a1&nbsp;-&nbsp;4),&nbsp;save_flag,&nbsp;0x1000uLL,&nbsp;0);
&nbsp;&nbsp;close(*(_DWORD&nbsp;*)(a1&nbsp;-&nbsp;4));
&nbsp;&nbsp;src&nbsp;=&nbsp;qword_E0B00A0;
&nbsp;&nbsp;dst&nbsp;=&nbsp;qword_E0AF0A0;
&nbsp;&nbsp;for&nbsp;(&nbsp;i&nbsp;=&nbsp;0x200LL;&nbsp;i;&nbsp;--i&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;*dst&nbsp;=&nbsp;*src;
&nbsp;&nbsp;&nbsp;&nbsp;++src;
&nbsp;&nbsp;&nbsp;&nbsp;++dst;
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;0xE0AF8A0LL;
}</pre>

<p style="text-indent: 2em;">
  处理部分反编译代码如上，我们会发现非常奇怪的是，这里做了一个字符串固定字符串的复制之后接返回了，什么事都没做啊，遇到这种情况当然得看看汇编代码，但不知道自己眼睛怎么长的，最开始一直没有看到问题……
</p>

<p style="text-indent: 2em;">
  然后，我还尝试通过strings往回找flag相关部分，发现确实是有输出flag的地方，可完全不知道从哪里跳过来的，然后发现了一段乱七八糟的代码片段，每个片段都很短，如下：
</p>

<pre class="brush:plain;toolbar:false">.text:000000000DEAD1E4&nbsp;jmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;short&nbsp;loc_DEAD1E8
.text:000000000DEAD1E6&nbsp;;&nbsp;---------------------------------------------------------------------------
.text:000000000DEAD1E6&nbsp;jp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;short&nbsp;loc_DEAD181
.text:000000000DEAD1E8
.text:000000000DEAD1E8&nbsp;loc_DEAD1E8:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;CODE&nbsp;XREF:&nbsp;.text:000000000DEAD1E4
.text:000000000DEAD1E8&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax
.text:000000000DEAD1E9&nbsp;retn</pre>

<p style="text-indent: 2em;">
  看起来似乎000000000DEAD1E6那里有2个无意义的字节，问题肯定就在这些代码片段里了，然后还发现，程序复制的那个字符串里中间有些部分看起来是一些指向这些代码片段的指针，但还是没有什么解题的想法。
</p>

<p style="text-indent: 2em;">
  最后，某次突然在这段汇编代码的最后部分发现了问题：
</p>

<pre class="brush:plain;toolbar:false">.text:000000000DEAD40E&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;offset&nbsp;save_flag
.text:000000000DEAD413&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdi,&nbsp;rax
.text:000000000DEAD416&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;offset&nbsp;unk_E0B20C0
.text:000000000DEAD41B&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;rax
.text:000000000DEAD41E&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax,&nbsp;(offset&nbsp;qword_E0AF0A0+800h)
.text:000000000DEAD423&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rax
.text:000000000DEAD426&nbsp;retn
.text:000000000DEAD426&nbsp;handle&nbsp;end</pre>

<p style="text-indent: 2em;">
  仔细一看，这里000000000DEAD423这一行竟然把rsp给改了，我勒个去，这是直接换栈了，那这下就啥都说的通了，它复制的那个字符串里面都是他提前设定好的地址，然后那些代码片段最后就通过ret来返回到特定的地方，一句话说就是以ROP的方式在运行程序，只能说感觉我也是醉了。
</p>

<p style="text-indent: 2em;">
  知道方法后，做起来也是麻烦，没有什么好的办法，只能一点点的手动把代码扒出来。
</p>

<p style="text-indent: 2em;">
  首先，先是把那个字符串中看起来是需要的部分拎出来：
</p>

<pre class="brush:python;toolbar:false;">arr&nbsp;=&nbsp;[233492980,&nbsp;8,&nbsp;233493105,&nbsp;1384820248253759637,&nbsp;233492771,&nbsp;233492717,&nbsp;233492996,&nbsp;233493095,&nbsp;233492728,&nbsp;233492739,&nbsp;233492717,&nbsp;233493114,&nbsp;233493006,&nbsp;233492728,&nbsp;233492972,&nbsp;51966,&nbsp;233492801,&nbsp;233492717,&nbsp;233492996,&nbsp;233493124,&nbsp;233492728,&nbsp;233492717,&nbsp;233493114,&nbsp;233493006,&nbsp;233492728,&nbsp;233492972,&nbsp;48879,&nbsp;233492781,&nbsp;233492717,&nbsp;233492996,&nbsp;233493124,&nbsp;233492728,&nbsp;233493192,&nbsp;1,&nbsp;233493134,&nbsp;13337,&nbsp;233492964,&nbsp;0,&nbsp;233492972,&nbsp;0,&nbsp;233492988,&nbsp;472,&nbsp;233492891,&nbsp;233492717,&nbsp;233493143,&nbsp;233493182,&nbsp;233492728,&nbsp;233492717,&nbsp;233493172,&nbsp;233493006,&nbsp;233492728,&nbsp;233492972,&nbsp;1,&nbsp;233492851,&nbsp;233492717,&nbsp;233492996,&nbsp;233493182,&nbsp;233492728,&nbsp;233492717,&nbsp;233493172,&nbsp;233493006,&nbsp;233492728,&nbsp;233492972,&nbsp;1,&nbsp;233492988,&nbsp;104,&nbsp;233492906,&nbsp;233492717,&nbsp;233493201,&nbsp;233493006,&nbsp;233492728,&nbsp;233492717,&nbsp;233493085,&nbsp;233493026,&nbsp;233492728,&nbsp;233492801,&nbsp;233492717,&nbsp;233492996,&nbsp;233493211,&nbsp;233492728,&nbsp;233492717,&nbsp;233493085,&nbsp;233493006,&nbsp;233492728,&nbsp;233492717,&nbsp;233493085,&nbsp;233493026,&nbsp;233492728,&nbsp;233492801,&nbsp;233492717,&nbsp;233492996,&nbsp;233493095,&nbsp;233492728,&nbsp;233492717,&nbsp;233493143,&nbsp;233493006,&nbsp;233492728,&nbsp;233492881,&nbsp;233492717,&nbsp;233492996,&nbsp;233493153,&nbsp;233492728,&nbsp;233492717,&nbsp;233493143,&nbsp;233493006,&nbsp;233492728,&nbsp;233492972,&nbsp;0,&nbsp;233492988,&nbsp;18446744073709551072L,&nbsp;233492906,&nbsp;233492717,&nbsp;233493201,&nbsp;233493006,&nbsp;233492728,&nbsp;233492717,&nbsp;233493114,&nbsp;233493026,&nbsp;233492728,&nbsp;233492988,&nbsp;32,&nbsp;233492906,&nbsp;233492988,&nbsp;18446744073709550648L,&nbsp;233492951,&nbsp;233493308]&nbsp;#,&nbsp;233493423,&nbsp;14950972352827054927L,&nbsp;13643839583502281931L]</pre>

<p style="text-indent: 2em;">
  然后一点点的得到地址与代码的对应关系表：
</p>

<pre class="brush:python;toolbar:false">d&nbsp;=&nbsp;{}
d[0x000000000DEAD1F4]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rcx&#39;
d[0x000000000DEAD271]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r9&#39;
d[0x000000000DEAD123]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rdi]&#39;
d[0x000000000DEAD0ED]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8&#39;
d[0x000000000DEAD204]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax&#39;
d[0x000000000DEAD267]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r8,&nbsp;[rsi]&#39;
d[0x000000000DEAD0F8]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8&#39;
d[0x000000000DEAD103]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&#39;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdi,&nbsp;8&#39;
d[0xdead27a]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r9&#39;
d[0xdead20e]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]&#39;
d[0xdead1ec]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx&#39;
d[0xdead141]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;imul&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx&#39;
d[0xdead284]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r9,&nbsp;[rsi]&#39;
d[0xdead12d]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx&#39;
d[0xdead2c8]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r12&#39;
d[0xdead28e]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r10&#39;
d[0xdead1e4]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax&#39;
d[0xdead1fc]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdx&#39;
d[0xdead19b]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbxnjnz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out1nadd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdxnout1:&#39;
d[0xdead297]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r10&#39;
d[0xdead2be]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r11,&nbsp;[rsi]&#39;
d[0xdead2b4]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r11&#39;
d[0xdead173]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;and&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx&#39;
d[0xdead1aa]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbxnjz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out2nadd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdxnout2:&#39;
d[0xdead2d1]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r12&#39;
d[0xdead25d]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r8&#39;
d[0xdead222]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx,&nbsp;[rsi]&#39;
d[0xdead2db]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r12,&nbsp;[rsi]&#39;
d[0xdead191]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;shr&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;1&#39;
d[0xdead2a1]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r10,&nbsp;[rsi]&#39;
d[0xdead1d7]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;loop&nbsp;&nbsp;&nbsp;&nbsp;success,&nbsp;out3nsuccess:nadd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdxnout3:&#39;
d[0xdead33c]&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;&nbsp;&nbsp;&#39;get&nbsp;flag&#39;

[d[one]&nbsp;for&nbsp;one&nbsp;in&nbsp;arr&nbsp;if&nbsp;one&nbsp;&gt;&nbsp;0xdead000&nbsp;and&nbsp;one&nbsp;&lt;&nbsp;0xdeaf000]</pre>

<p style="text-indent: 2em;">
  然后按这个表替换之后得到代码如下：
</p>

<pre class="brush:plain;toolbar:false">pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rcx
pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r9
mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rdi]
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r8,&nbsp;[rsi]
sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdi,&nbsp;8
...
mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r9
mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx,&nbsp;[rsi]
sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdx
cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
jz&nbsp;next3
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
next3:
pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdx
loop&nbsp;&nbsp;&nbsp;&nbsp;success1,&nbsp;fail2
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
get&nbsp;flag</pre>

<p style="text-indent: 2em;">
  我们发现里面从来没有做过任何修改栈上值的操作，以及唯一对rsp的修改就是add rsp，也就是相当于把代码跳转回去，就是实现循环的作用了。
</p>

<p style="text-indent: 2em;">
  于是乎，为了看的更清楚，我们给每个语句编上地址，使我们可以知道每次add rsp是跳转到哪里，以及每次pop出来的值是多少：
</p>

<pre class="brush:python;toolbar:false">[8,&nbsp;1384820248253759637,&nbsp;51966,&nbsp;48879,&nbsp;1,&nbsp;13337,&nbsp;0,&nbsp;0,&nbsp;472,&nbsp;1,&nbsp;1,&nbsp;104,&nbsp;0,&nbsp;18446744073709551072L,&nbsp;32,&nbsp;18446744073709550648L]

i&nbsp;=&nbsp;0
for&nbsp;one&nbsp;in&nbsp;s:
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;one.find(&#39;pop&#39;)&nbsp;!=&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;one.replace(&#39;pop&nbsp;&nbsp;&#39;,&nbsp;&#39;mov_s&#39;)&nbsp;+&nbsp;&#39;,&nbsp;&#39;&nbsp;+&nbsp;hex(t[i])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;one</pre>

<p style="text-indent: 2em;">
  我们这里会将pop替换成mov_s指令，当做mov指令看就好，加个_s是为了区分下而已：
</p>

<pre class="brush:plain;toolbar:false">dst&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;0xE0AF0A0
src&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;0xE0B00A0
buffer&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;=&nbsp;0xE0B20C0
save_flag&nbsp;&nbsp;&nbsp;=&nbsp;input

rdi&nbsp;=&nbsp;save_flag
rsi&nbsp;=&nbsp;buffer

0x008:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rcx,&nbsp;0x8
0x018:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;r9,&nbsp;0x1337deadbeef0095
0x028:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rdi]
0x030:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x038:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x040:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r8,&nbsp;[rsi]
0x048:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x050:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rdi,&nbsp;8
0x058:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x060:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r9
0x068:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x070:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
//
0x078:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0xcafe
0x088:&nbsp;&nbsp;imul&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
0x090:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x098:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x0a0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r9,&nbsp;[rsi]
0x0a8:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x0b0:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x0b8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r9
0x0c0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x0c8:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x0d0:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0xbeef
0x0e0:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
0x0e8:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x0f0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x0f8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r9,&nbsp;[rsi]
0x100:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x108:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;r12,&nbsp;0x1
0x118:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;r10,&nbsp;0x3419
0x128:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rax,&nbsp;0x0
0x138:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0x0
0x148:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rdx,&nbsp;0x1d8
0x158:&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
jnz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out1
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
out1:
0x160:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x168:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r10
0x170:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r11,&nbsp;[rsi]
0x178:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x180:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x188:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r11
0x190:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x198:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x1a0:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0x1
0x1b0:&nbsp;&nbsp;and&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
0x1b8:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x1c0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x1c8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r11,&nbsp;[rsi]
0x1d0:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x1d8:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x1e0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r11
0x1e8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x1f0:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x1f8:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0x1
0x208:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rdx,&nbsp;0x68
0x218:&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
jz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out2
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
out2:
0x220:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x228:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r12
0x230:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x238:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x240:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x248:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r8
0x250:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx,&nbsp;[rsi]
0x258:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x260:&nbsp;&nbsp;imul&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
0x268:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x270:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x278:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r12,&nbsp;[rsi]
0x280:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x288:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x290:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r8
0x298:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x2a0:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2a8:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2b0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r8
0x2b8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx,&nbsp;[rsi]
0x2c0:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2c8:&nbsp;&nbsp;imul&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
0x2d0:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2d8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x2e0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r8,&nbsp;[rsi]
0x2e8:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2f0:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x2f8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r10
0x300:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x308:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x310:&nbsp;&nbsp;shr&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;1
0x318:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x320:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;rax
0x328:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r10,&nbsp;[rsi]
0x330:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x338:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x340:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r10
0x348:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x350:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x358:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rbx,&nbsp;0x0
0x368:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rdx,&nbsp;0xfffffffffffffde0L
0x378:&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
jz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out2
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
out2:
0x380:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x388:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r12
0x390:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;[rsi]
0x398:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x3a0:&nbsp;&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x3a8:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[rsi],&nbsp;r9
0x3b0:&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rbx,&nbsp;[rsi]
0x3b8:&nbsp;&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsi,&nbsp;8
0x3c0:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rdx,&nbsp;0x20
0x3d0:&nbsp;&nbsp;cmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rax,&nbsp;rbx
jz&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out2
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
out2:
0x3d8:&nbsp;&nbsp;mov_s&nbsp;&nbsp;&nbsp;rdx,&nbsp;0xfffffffffffffc38L
0x3e8:&nbsp;&nbsp;loop&nbsp;&nbsp;&nbsp;&nbsp;success,&nbsp;out3
success:
add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rsp,&nbsp;rdx
out3:
0x3f0:&nbsp;&nbsp;get&nbsp;flag</pre>

<p style="text-indent: 2em;">
  然后这里看跳转的规则是，用那句add esp前面的标号，加上这句指令给rsp增加的值，得到的地址，就是下一条指令了。
</p>

<p style="text-indent: 2em;">
  于是乎，慢慢看下这段程序，会发现就是一个快速幂（PS：这里最后几句代码可能对应的有点问题，但不太影响整体逻辑的理解），就是求输入8个字节（作为一个长整数看）的0x3419次方，与一个目标值比较，然后会这样比较9轮（好吧，我承认这一部分是动态调出来了，看出快速幂后就没管这段程序了），而目标值的初始值是0x1337DEADBEEF0095（初始值不算一轮），下一轮的值为前一轮的值乘以0xcafe后加0xbeef。
</p>

<p style="text-indent: 2em;">
  果断计算出每轮目标值，然后用WolframAlpha算出应该的输入（比如<a href="http://www.wolframalpha.com/input/?i=x+%5E+13337+mod+%282+%5E+64%29+%3D+2820389213912491205" target="_blank" textvalue="第一轮的输入">第一轮的输入</a>），最后得flag的程序如下：
</p>

<pre class="brush:python;toolbar:false">import&nbsp;zio

#&nbsp;aim&nbsp;=&nbsp;0x1337DEADBEEF0095
#&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(9):
#&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;aim&nbsp;=&nbsp;aim&nbsp;*&nbsp;0xcafe&nbsp;+&nbsp;0xbeef
#&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;aim&nbsp;%&nbsp;(2&nbsp;**&nbsp;64)
#
#&nbsp;WolframAlpha

ans&nbsp;=&nbsp;[15397851891508532645,
5882479468299745861,
6203462900357968133,
11756062250229433221,
6010465035347615365,
10697597399494305925,
14617956564689458309,
9036388475871214725,
17260895165766855813]

payload&nbsp;=&nbsp;&#39;&#39;
for&nbsp;i&nbsp;in&nbsp;ans:
&nbsp;&nbsp;&nbsp;&nbsp;payload&nbsp;+=&nbsp;zio.l64(i)

io&nbsp;=&nbsp;zio.zio((&#39;127.0.0.1&#39;,&nbsp;0x3419))
io.write(payload)</pre>

<p style="text-indent: 2em;">
  这题150分搞成这样，也是有点无语了，知道方法后，最后的一点苦力工作当时都不想干了，都是交给队友看的了~~~
</p>
