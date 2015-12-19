---
title: ucore 操作系统实验 lab1小结（含Challenge）
author: Ripples
layout: post
permalink: /2014/10/11/ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab1%e5%b0%8f%e7%bb%93%ef%bc%88%e5%90%abchallenge%ef%bc%89/
views:
  - 1094
categories:
  - technology
tags:
  - ucore
  - 操作系统
  - 权限切换
---
<p style="text-indent: 2em;">
  最近做操作系统的实验，本来挺好的，结果被那个challenge给折磨了好久，真的想说challenge的题目描述实在让人无力吐槽。
</p>

<pre class="brush:plain;toolbar:false">扩展proj4,增加syscall功能，即增加一用户态函数（可执行一特定系统调用：获得时钟计数值），当内核初始完毕后，可从内核态返回到用户态的函数，而用户态的函数又通过系统调用得到内核态的服务。需写出详细的设计和分析报告。完成出色的可获得适当加分。
提示：规范一下&nbsp;challenge&nbsp;的流程。&nbsp;
kern_init&nbsp;调用&nbsp;switch_test，该函数如下：</pre>

<!--more-->

<pre class="brush:cpp;toolbar:false">static&nbsp;void&nbsp;switch_test(void)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;print_cur_status();&nbsp;//&nbsp;print&nbsp;当前&nbsp;cs/ss/ds&nbsp;等寄存器状态
&nbsp;&nbsp;&nbsp;&nbsp;cprintf(“+++&nbsp;switch&nbsp;to&nbsp;&nbsp;user&nbsp;&nbsp;mode&nbsp;+++n”);
&nbsp;&nbsp;&nbsp;&nbsp;switch_to_user();&nbsp;&nbsp;//&nbsp;switch&nbsp;to&nbsp;user&nbsp;mode
&nbsp;&nbsp;&nbsp;&nbsp;print_cur_status();
&nbsp;&nbsp;&nbsp;&nbsp;cprintf(“+++&nbsp;switch&nbsp;to&nbsp;kernel&nbsp;mode&nbsp;+++n”);
&nbsp;&nbsp;&nbsp;&nbsp;switch_to_kernel();&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;switch&nbsp;to&nbsp;kernel&nbsp;mode
&nbsp;&nbsp;&nbsp;&nbsp;print_cur_status();
}</pre>

<pre class="brush:plain;toolbar:false">switch_to_*&nbsp;函数建议通过&nbsp;中断处理的方式实现。主要要完成的代码是在&nbsp;trap&nbsp;里面处理&nbsp;T_SWITCH_TO*&nbsp;中断，并设置好返回的状态。</pre>

<p style="text-indent: 2em;">
  就这样的描述，反正从头到尾我也没能弄清楚他是想要让我们做什么，上来就说让我们增加syscall功能，可最后却check的是内核态与用户态的切换。
</p>

<p style="text-indent: 2em;">
  然后，最让我不能理解的还是，为什么他要让我们通过中断做一个从用户态到内核态的切换，用户态自己随便调用一个中断，就能把权限提升为内核权限，这样的操作系统真的还有安全性可言么，实在是不能理解！！！<span style="color: rgb(255, 0, 0);">（PS：后听一同学说是内核启动第一个用户程序的时候，需要从用户态切回内核态，完全初始化好后再关掉此中断即可）</span>
</p>

<p style="text-indent: 2em;">
  不过，既然让做，咱就来做吧：
</p>

<p style="text-indent: 2em;">
  首先是内核态切到用户态的函数：
</p>

<p style="text-indent: 2em;">
  题目建议通过中断处理实现，于是乎在trap.h中查找中断号，发现：
</p>

<pre class="brush:cpp;toolbar:false">#define&nbsp;T_SWITCH_TOU&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;120&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;user/kernel&nbsp;switch
#define&nbsp;T_SWITCH_TOK&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;121&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;user/kernel&nbsp;switch</pre>

<p style="text-indent: 2em;">
  然后，查看中断处理函数入口(vector.S)，会发现所有中断处理函数都是trapentry.S中的__alltraps函数：
</p>

<p style="text-indent: 2em;">
  而__alltraps就是将各种寄存器压栈保存，然后调用trap.c中的trap函数，最后恢复各寄存器并返回，在trap.h中可发现压栈保存的数据结构：
</p>

<pre class="brush:cpp;toolbar:false">/*&nbsp;registers&nbsp;as&nbsp;pushed&nbsp;by&nbsp;pushal&nbsp;*/
struct&nbsp;pushregs&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_edi;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_esi;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_ebp;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_oesp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/*&nbsp;Useless&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_ebx;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_edx;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_ecx;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;reg_eax;
};
struct&nbsp;trapframe&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;struct&nbsp;pushregs&nbsp;tf_regs;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_gs;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding0;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_fs;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding1;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_es;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding2;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_ds;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding3;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;tf_trapno;
&nbsp;&nbsp;&nbsp;&nbsp;/*&nbsp;below&nbsp;here&nbsp;defined&nbsp;by&nbsp;x86&nbsp;hardware&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;tf_err;
&nbsp;&nbsp;&nbsp;&nbsp;uintptr_t&nbsp;tf_eip;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_cs;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding4;
&nbsp;&nbsp;&nbsp;&nbsp;uint32_t&nbsp;tf_eflags;
&nbsp;&nbsp;&nbsp;&nbsp;/*&nbsp;below&nbsp;here&nbsp;only&nbsp;when&nbsp;crossing&nbsp;rings,&nbsp;such&nbsp;as&nbsp;from&nbsp;user&nbsp;to&nbsp;kernel&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;uintptr_t&nbsp;tf_esp;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_ss;
&nbsp;&nbsp;&nbsp;&nbsp;uint16_t&nbsp;tf_padding5;
}&nbsp;__attribute__((packed));</pre>

<p style="text-indent: 2em;">
  于是可发现，我们想要做特权级的切换，就是要修改cs、ds、es等段寄存器，而通过中断修改，最好的办法就是直接将保存在栈中这些数据修改，等返回时就会将寄存器调整为我们需要的值，实现权限切换。
</p>

<p style="text-indent: 2em;">
  然后需要注意的是，在由内核态切换成用户态的时候，一开始调用中断时，由于是从内核态调用的，没有权限切换，故ss、esp没有压栈，而iret返回时，是返回到用户态，故ss、esp会出栈，于是为了保证栈的正确性，需要在调用中断前将esp减8以预留空间，中断返回后，由于esp被修改，还需要手动恢复esp为正确值。
</p>

<p style="text-indent: 2em;">
  这样之后，系统特权级已经成功切换，但是由于切换到了用户态，导致IO操作没有权限，故之后的printf无法成功输出，为了能够正常输出，我们需要将eflags中的IOPL设成用户级别，即3，同样也是通过修改栈中值来达到修改的目的。
</p>

<p style="text-indent: 2em;">
  这个IOPL确实是让人无语至极，实验给的资料里完全没有提及过这个东西，但是没有设置这个，会导致在printf的时候走入自动中断，简直看不出来是什么原因导致的！
</p>

<p style="text-indent: 2em;">
  IOPL(EFLAGS寄存器的bits 12 and 13) [I/O privilege level field]：指示当前运行任务的I/O特权级(I/O privilege level)，正在运行任务的当前特权级(CPL)必须小于或等于I/O特权级才能允许访问I/O地址空间。这个域只能在CPL为0时才能通过POPF以及IRET指令修改。
</p>

<p style="text-indent: 2em;">
  接下来是从用户态切换到内核态，此时由于调用中断时，是从用户态调用的，故会将ss、esp压栈，但是iret返回时，是返回到内核态，故ss、esp不会出栈，所以此时需要手动出栈，而ss由于维持了中断内部的ss，即已经是kernel，无需修改，故直接出栈esp即可。最后，为了能够让用户态的时候可以调用此中断，需要在idt的初始化函数中将该中断的DPL改成USER级别。
</p>

<p style="text-indent: 2em;">
  最后，再实现一下syscall，获得时钟计数值，syscall number采取255，于是便可以得到：
</p>

<pre class="brush:cpp;toolbar:false">static&nbsp;int&nbsp;get_ticks(void)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;asm("mov&nbsp;$0xff,&nbsp;%eax");
&nbsp;&nbsp;&nbsp;&nbsp;asm("int&nbsp;$0x80");
}</pre>

<p style="text-indent: 2em;">
  而对于中断内部的处理代码就更简单了，直接在trap_dispatch的switch里面增加一个分支返回ticks即可。
</p>

<p style="text-indent: 2em;">
  最后在lab1_switch_test里面的kernel切到user后，再切回kernel前调用此函数进行测试，发现可正确获取ticks。
</p>

<p style="text-indent: 2em;">
  最后，附上整个lab1的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/10/lab1_result.zip">lab1_result.zip</a>
</p>
