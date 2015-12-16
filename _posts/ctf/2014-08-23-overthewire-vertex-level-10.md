---
title: OverTheWire Vertex Level 10 → Level 11
author: Ripples
layout: post
views:
  - 211
categories:
  - CTF
tags:
  - reversing
  - vortex
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">这一关本身并不难，只是由于种种蛋疼的原因，也耗费了我不少时间。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">原题参见</span><a href="http://overthewire.org/wargames/vortex/vortex10.html" target="_self" style="text-indent: 32px; white-space: normal;">这里</a><span style="text-indent: 32px;">。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">题目就是它会输出20个随机数，让计算出生成这20个随机数的种子。由随机数算种子，想来应该是除了暴力对比没啥好办法了吧～～～</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">如果直接取前20个随机数，多进程对比，那只能说太靠人品了，还是规规矩矩反汇编吧，下面给出反汇编后仿写的代码：</span>
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">int&nbsp;main()
{
&nbsp;&nbsp;&nbsp;&nbsp;seed&nbsp;=&nbsp;times(&tms);
&nbsp;&nbsp;&nbsp;&nbsp;seed&nbsp;+=&nbsp;(tms.utime&nbsp;+&nbsp;tms.stime&nbsp;+&nbsp;tms.cutime&nbsp;+&nbsp;tms.cstime);
&nbsp;&nbsp;&nbsp;&nbsp;seed&nbsp;+=&nbsp;clock();
&nbsp;&nbsp;&nbsp;&nbsp;seed&nbsp;+=&nbsp;time(NULL);
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;ass&nbsp;=&nbsp;(seed&nbsp;sar&nbsp;0x1f)&nbsp;&gt;&gt;&nbsp;0x18;&nbsp;//&nbsp;sar表示算术右移
&nbsp;&nbsp;&nbsp;&nbsp;seed&nbsp;=&nbsp;0x80&nbsp;-&nbsp;(((seed&nbsp;+&nbsp;ass)&nbsp;&&nbsp;0xff)&nbsp;-&nbsp;ass);
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;real_seed&nbsp;=&nbsp;seed&nbsp;+&nbsp;time(NULL);
&nbsp;&nbsp;&nbsp;&nbsp;srand(real_seed);
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;seed;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rand();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;printf("[");
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;20;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buffer[i]&nbsp;=&nbsp;rand();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("&nbsp;%08x,",&nbsp;buffer[i]);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;printf("]");
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;alarm(30);
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&nbsp;&ans,&nbsp;4);
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ans&nbsp;==&nbsp;real_seed)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;setresuid(geteuid(),&nbsp;geteuid(),&nbsp;geteuid());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;execlp("/bin/sh",&nbsp;"sh",&nbsp;"-i",&nbsp;NULL);
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;else&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("Nope,&nbsp;try&nbsp;again");
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;"></span>从代码中可以看出，首先rand()了seed次之后，才rand()出输出的20个数，所以只有当seed<=0时，输出的才是前20次rand()的结果。
</p>

<p style="text-indent: 2em;">
  再仔细观察seed，会发现seed的范围是128-255～128+255，即-127～383。而对于time(NULL)而言，由于其精度是到秒的，那么也就是说我们只要在运行这个程序前输出time(NULL)，那么得到的基本就是这个程序中的取到time(NULL)不会错了。这样，实际上我们需要尝试的范围就只有511个数，很轻易就可以得到结果。
</p>

<p style="text-indent: 2em;">
  启动程序如下：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;unistd.h&gt;

int&nbsp;main()
{
&nbsp;&nbsp;&nbsp;&nbsp;printf("Time&nbsp;:&nbsp;%xn",&nbsp;time(NULL));
&nbsp;&nbsp;&nbsp;&nbsp;execl("/vortex/vortex10",&nbsp;"",&nbsp;NULL);
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  暴力搜索结果的程序如下：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;time.h&gt;

int&nbsp;main()
{
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;i,&nbsp;j;
&nbsp;&nbsp;&nbsp;&nbsp;unsigned&nbsp;int&nbsp;aim[20];
&nbsp;&nbsp;&nbsp;&nbsp;unsigned&nbsp;int&nbsp;tseed;
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;flag;
&nbsp;&nbsp;&nbsp;&nbsp;scanf("Time&nbsp;:&nbsp;%xn[&nbsp;",&nbsp;&tseed);
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;20;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;scanf("%x,",&nbsp;aim&nbsp;+&nbsp;i);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(i&nbsp;=&nbsp;128&nbsp;-&nbsp;255;&nbsp;i&nbsp;&lt;=&nbsp;128&nbsp;+&nbsp;255;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;srand(tseed&nbsp;+&nbsp;i);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(j&nbsp;=&nbsp;0;&nbsp;j&nbsp;&lt;&nbsp;i;&nbsp;++j)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rand();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;flag&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(j&nbsp;=&nbsp;0;&nbsp;j&nbsp;&lt;&nbsp;20;&nbsp;++j)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(rand()&nbsp;!=&nbsp;aim[j])&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;flag&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(flag&nbsp;!=&nbsp;0)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tseed&nbsp;+=&nbsp;i;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*ans&nbsp;=&nbsp;(char&nbsp;*)&tseed;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("%.4sn",&nbsp;ans);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("cat&nbsp;/etc/vortex_pass/vortex11n");
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  然后运行./search | ./startup，并把startup的输出输入给search即可得到password。
</p>
