---
title: SCTF Misc100 WriteUp
author: Ripples
layout: post
permalink: /sctf-misc100-writeup/
views:
  - 253
categories:
  - CTF
tags:
  - CTF
  - reversing
  - SCTF
  - upx
  - WriteUp
---
<p style="text-indent: 2em;">
  似乎每次比赛都会有一道游戏题，这次SCTF的游戏题就是这道了，话说着实没想明白为什么这题算是Misc……
</p>

<p style="text-indent: 2em;">
  题目描述就一句话：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">简单的贪吃蛇，吃到30分它就告诉你flag！但是要怎么控制它呢?</pre>

<p style="text-indent: 2em;">
  首先将程序download下来，是一个后缀名为exe的程序，果断就给扔XP虚拟机里了，一运行就看到一个控制台窗口上一个光标跳来跳去，完全看不懂，说好的贪吃蛇呢……
</p>

<p style="text-indent: 2em;">
  然后往IDA里面扔的时候，却被自动识别为类型为ELF，当时就惊呆了……
</p>

<p style="text-indent: 2em;">
  然后放Unbuntu里跑，总算是正常看到了贪吃蛇，不过果不其然不知道怎么能够控制。
</p>

<p style="text-indent: 2em;">
  然后就只能规规矩矩看程序了，很明显程序是加了个UPX的壳（PEiD查的），于是马上aptget了一个upx，然后upx -d搞定。
</p>

<p style="text-indent: 2em;">
  看程序的时候，从start一路跟下去，会发现程序进到了如下一段代码中：
</p>

<pre class="brush:plain;toolbar:false">.text:08048C40
.text:08048C40&nbsp;;&nbsp;Segment&nbsp;type:&nbsp;Pure&nbsp;code
.text:08048C40&nbsp;;&nbsp;Segment&nbsp;permissions:&nbsp;Read/Execute
.text:08048C40&nbsp;_text&nbsp;segment&nbsp;para&nbsp;public&nbsp;&#39;CODE&#39;&nbsp;use32
.text:08048C40&nbsp;assume&nbsp;cs:_text
.text:08048C40&nbsp;;org&nbsp;8048C40h
.text:08048C40&nbsp;assume&nbsp;es:nothing,&nbsp;ss:nothing,&nbsp;ds:_data,&nbsp;fs:nothing,&nbsp;gs:nothing
.text:08048C40
.text:08048C40
.text:08048C40&nbsp;;&nbsp;Attributes:&nbsp;noreturn&nbsp;bp-based&nbsp;frame
.text:08048C40
.text:08048C40&nbsp;sub_8048C40&nbsp;proc&nbsp;near
.text:08048C40&nbsp;lea&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ecx,&nbsp;[esp+4]
.text:08048C44&nbsp;and&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;esp,&nbsp;0FFFFFFF0h
.text:08048C47&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;dword&nbsp;ptr&nbsp;[ecx-4]
.text:08048C4A&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;ebp
.text:08048C4B&nbsp;mov&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ebp,&nbsp;esp
.text:08048C4D&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;ecx
.text:08048C4E&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;esp,&nbsp;4
.text:08048C51&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_initscr
.text:08048C56&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;sub_8048EA0
.text:08048C5B&nbsp;sub&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;esp,&nbsp;8
.text:08048C5E&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;handler&nbsp;&nbsp;;&nbsp;handler
.text:08048C63&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;0Eh&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048C65&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048C6A&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048C6B&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edx
.text:08048C6C&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048E10&nbsp;;&nbsp;handler
.text:08048C71&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;38h&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048C73&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048C78&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ecx
.text:08048C79&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048C7A&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048E70&nbsp;;&nbsp;handler
.text:08048C7F&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;32h&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048C81&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048C86&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048C87&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edx
.text:08048C88&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;nullsub_1&nbsp;;&nbsp;handler
.text:08048C8D&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;5&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048C8F&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048C94&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ecx
.text:08048C95&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048C96&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048E10&nbsp;;&nbsp;handler
.text:08048C9B&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;0Ah&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048C9D&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048CA2&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048CA3&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edx
.text:08048CA4&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048E70&nbsp;;&nbsp;handler
.text:08048CA9&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;0Ch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048CAB&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048CB0&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ecx
.text:08048CB1&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048CB2&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048DE0&nbsp;;&nbsp;handler
.text:08048CB7&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;34h&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048CB9&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048CBE&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;eax
.text:08048CBF&nbsp;pop&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;edx
.text:08048CC0&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;offset&nbsp;sub_8048E40&nbsp;;&nbsp;handler
.text:08048CC5&nbsp;push&nbsp;&nbsp;&nbsp;&nbsp;36h&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;;&nbsp;sig
.text:08048CC7&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;_signal
.text:08048CCC&nbsp;add&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;esp,&nbsp;10h
.text:08048CCF&nbsp;call&nbsp;&nbsp;&nbsp;&nbsp;sub_80492C0
.text:08048CCF&nbsp;sub_8048C40&nbsp;endp
.text:08048CCF</pre>

<p style="text-indent: 2em;">
  可以看到基本都是在发信号来完成整个程序，那么程序的关键肯定就在信号处理函数上，然后再看32h，34h，36h，38h这些数字很奇怪，正好就是2、4、6、8的ASCII码，这让我们不禁想到了程序运行时屏幕上的那句话：
</p>

<pre class="brush:plain;toolbar:false">&nbsp;UP----&#39;8&#39;&nbsp;&nbsp;&nbsp;DOWN----&#39;2&#39;&nbsp;&nbsp;&nbsp;LEFT----&#39;4&#39;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RIGHT----&#39;6&#39;</pre>

<p style="text-indent: 2em;">
  然后我们尝试手动发下信号，很容易便会发现，我们的猜想是正确的，这4个信号正好对应着4个方向。
</p>

<p style="text-indent: 2em;">
  然后接下来的事就可以很简单了，写个程序，将方向键换成发这4个信号，然后玩游戏就好，玩到30之后，就会获得flag的base64编码的值，解码即可。
</p>

<p style="text-indent: 2em;">
  最后玩游戏这步是队友做的，程序如下：
</p>

<pre class="brush:cpp;toolbar:false">#include&nbsp;&lt;stdio.h&gt;
#include&nbsp;&lt;termios.h&gt;
#include&nbsp;&lt;term.h&gt;
#include&nbsp;&lt;curses.h&gt;
#include&nbsp;&lt;unistd.h&gt;
#include&nbsp;&lt;sys/types.h&gt;&nbsp;&nbsp;
#include&nbsp;&lt;stdlib.h&gt;&nbsp;&nbsp;
#include&nbsp;&lt;signal.h&gt;&nbsp;

static&nbsp;struct&nbsp;termios&nbsp;initial_settings,&nbsp;new_settings;
static&nbsp;int&nbsp;peek_character&nbsp;=&nbsp;-1;
void&nbsp;init_keyboard();
void&nbsp;close_keyboard();
int&nbsp;kbhit();
int&nbsp;readch();

int&nbsp;main(&nbsp;int&nbsp;argc,&nbsp;char&nbsp;*argv[])
{
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;ch&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;init_keyboard();
&nbsp;&nbsp;&nbsp;&nbsp;pid_t&nbsp;pid;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;pid&nbsp;=&nbsp;atoi(argv[1]);&nbsp;&nbsp;&nbsp;//先运行snake，然后获得进程pid，作为参数传进来
&nbsp;&nbsp;&nbsp;&nbsp;//printf(pid);
&nbsp;&nbsp;&nbsp;&nbsp;while(ch&nbsp;!=&nbsp;&#39;q&#39;)&nbsp;{		//w,a,s,d经典上下左右按键
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if(kbhit())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ch&nbsp;=&nbsp;readch();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;(ch)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	case&nbsp;&#39;w&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		kill(pid,&nbsp;56);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		break;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	case&nbsp;&#39;a&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		kill(pid,&nbsp;52);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		break;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	case&nbsp;&#39;s&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		kill(pid,&nbsp;50);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		break;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;	case&nbsp;&#39;d&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		kill(pid,&nbsp;54);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;		break;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;close_keyboard();
&nbsp;&nbsp;&nbsp;&nbsp;exit(0);
}

void&nbsp;init_keyboard()
{
&nbsp;&nbsp;&nbsp;&nbsp;tcgetattr(0,&initial_settings);
&nbsp;&nbsp;&nbsp;&nbsp;new_settings&nbsp;=&nbsp;initial_settings;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_lflag&nbsp;&=&nbsp;~ICANON;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_lflag&nbsp;&=&nbsp;~ECHO;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_lflag&nbsp;&=&nbsp;~ISIG;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_cc[VMIN]&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_cc[VTIME]&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;tcsetattr(0,&nbsp;TCSANOW,&nbsp;&new_settings);
}

void&nbsp;close_keyboard()
{
&nbsp;&nbsp;&nbsp;&nbsp;tcsetattr(0,&nbsp;TCSANOW,&nbsp;&initial_settings);
}

int&nbsp;kbhit()
{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;ch;
&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;nread;
&nbsp;&nbsp;&nbsp;&nbsp;if(peek_character&nbsp;!=&nbsp;-1)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_cc[VMIN]=0;
&nbsp;&nbsp;&nbsp;&nbsp;tcsetattr(0,&nbsp;TCSANOW,&nbsp;&new_settings);
&nbsp;&nbsp;&nbsp;&nbsp;nread&nbsp;=&nbsp;read(0,&ch,1);
&nbsp;&nbsp;&nbsp;&nbsp;new_settings.c_cc[VMIN]=1;
&nbsp;&nbsp;&nbsp;&nbsp;tcsetattr(0,&nbsp;TCSANOW,&nbsp;&new_settings);
	if(nread&nbsp;==&nbsp;1)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;	&nbsp;&nbsp;peek_character&nbsp;=&nbsp;ch;
&nbsp;&nbsp;&nbsp;&nbsp;	&nbsp;&nbsp;return&nbsp;1;
	}
	return&nbsp;0;
}

int&nbsp;readch()
{
&nbsp;&nbsp;&nbsp;&nbsp;char&nbsp;ch;
&nbsp;&nbsp;&nbsp;&nbsp;if(peek_character&nbsp;!=&nbsp;-1)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ch&nbsp;=&nbsp;peek_character;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;peek_character&nbsp;=&nbsp;-1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;ch;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;read(0,&ch,1);
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;ch;
}</pre>

<p style="text-indent: 2em;">
  我最开始是为了方便，想写个shell的脚本来做的，结果在网上各种搜不到，搜到的唯一一个还用不起来，我也是醉了……最后实现的是一个需要每次输入之后回车的，然后就感觉蛇跑太快了，玩不过去，于是就先放着给队友了……
</p>

<p style="text-indent: 2em;">
  其实当时还想过打补丁、改内存神马的方式来弄的，不过在Linux也不太会弄，然后就放弃了……
</p>

<p style="text-indent: 2em;">
  最后比赛结束后才想起来，可以把命令行的窗口拉大，这样游戏范围就变大了，这样就算是每次输入后回车也毫无压力玩过了，瞬间感觉自己当时好脑残……
</p>

<p style="text-indent: 2em;">
  按说这道题是可以完全靠逆向做出来，不用玩游戏的，不过自己水平不济，连程序整体都没大看明白，就更别谈那个复杂的要死的给答案的部分了。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/12/snake.zip">snake.zip</a>
</p>
