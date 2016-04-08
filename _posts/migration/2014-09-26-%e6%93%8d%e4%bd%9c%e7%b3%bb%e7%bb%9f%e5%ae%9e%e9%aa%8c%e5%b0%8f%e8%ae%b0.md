---
title: 操作系统实验小记
author: Ripples
layout: post
permalink: /操作系统实验小记/
views:
  - 404
categories:
  - technology
tags:
  - ucore
  - 操作系统
---
<p style="text-indent: 2em;">
  这学期上操作系统，不得不说，这个实验绝对是上大学以来做过的最难的实验了。
</p>

<pre class="brush:plain;toolbar:false">MIT的Frans&nbsp;Kaashoek等在2006年参考PDP-11上的UNIX&nbsp;Version&nbsp;6写了一个可在X86上跑的操作系统xv6（基于MIT&nbsp;License），用于学生学习操作系统。我们可以站在他们的肩膀上，基于xv6的设计，尝试着一步一步完成一个从“空空&nbsp;如也”到“五脏俱全”的“麻雀”操作系统—ucore，此“麻雀”包含虚存管理、进程管理、处理器调度、同步互斥、进程间通信、文件系统等主要内核功能，总的内核代码量（C+asm）不会超过5K行。充分体现了“小而全”的指导思想。</pre>

<!--more-->

<p style="text-indent: 2em;">
  说句实在话，本来没做之前想的是，一个课程实验能有多难，毕竟总得照顾到所有的学生吧，结果一开始做，就彻底无话可说了。
</p>

<p style="text-indent: 2em;">
  诺大一个程序，加之本来对于很多底层的编程就不熟悉，还有那以前向来不看的makefile，看起来真的是非一般的累，虽说实验配了大量的资料，但资料茫茫多，又还有点乱，真是不忍直视。
</p>

<p style="text-indent: 2em;">
  花了好长的时间，才算勉强开好了头，把实验1的前几问琢磨清楚了（找BIOS还真是废了好大劲），然后心血来潮，决定做点奇奇怪怪的事。
</p>

<p style="text-indent: 2em;">
  首先写了个hello.c：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;

int&nbsp;main(void)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;printf("hello&nbsp;world!n");
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  然后静态编译：
</p>

<pre class="brush:bash;toolbar:false">gcc&nbsp;-o&nbsp;hello&nbsp;hello.c&nbsp;-static&nbsp;-m32&nbsp;-g</pre>

<p style="text-indent: 2em;">
  接着，用hello替换掉lab1中编译生成的bin/kernel，最后重新make debug，本来指望看能不能成功输出hello world的，结果谁知道，一运行，竟然直接qemu直接花屏了，好不震惊。
</p>

<p style="text-indent: 2em;">
  最后，终于确定，是在bootmain中将hello程序的各个段载入内存的时候导致的花屏，然后突然想到原来看汇编的，有一块内存区域对应于console的频幕输出的，大概就只能是这块区域被覆盖了。
</p>

<p style="text-indent: 2em;">
  在使用VGA方式时，其文本方式和低分辨率图形方式占用了0xB8000-0xBFFFF的空间，然后程序的地址是0x<a></a>8048000之后的空间，只是，由于bootmain中，对所有载入地址，做了个&0xffffff的操作，故变成了从0x<a></a>48000开始。
</p>

<p style="text-indent: 2em;">
  然后查看程序段头：
</p>

<pre class="brush:plain;toolbar:false">&nbsp;&nbsp;[&nbsp;6]&nbsp;.text&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PROGBITS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;08048300&nbsp;000300&nbsp;07f513&nbsp;00&nbsp;&nbsp;AX&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;0&nbsp;16</pre>

<p style="text-indent: 2em;">
  发现text段是从0x<a></a>08048300开始的0x07f513个字节，这样，计算后会发现，0xB8000正好对应于文件的0x<a></a>70000的位置，还在.text段中，同时，通过对qemu进行debug，我们也会发现花屏后0xB8000出的数据值正好与文件0x<a></a>70000处的数据完全相同，验证了我们的想法。
</p>
