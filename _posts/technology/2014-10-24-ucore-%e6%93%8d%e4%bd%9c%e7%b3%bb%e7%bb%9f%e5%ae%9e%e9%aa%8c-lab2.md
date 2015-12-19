---
title: ucore 操作系统实验 lab2
author: Ripples
layout: post
permalink: /ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab2/
views:
  - 415
categories:
  - technology
tags:
  - ucore
  - 内存管理
  - 操作系统
  - 段页式
---
<p style="text-indent: 2em;">
  说句实在话，感觉lab2不算难，只要认真看给的资料应该都能完成。
</p>

<p style="text-indent: 2em;">
  lab2主要是涉及到了虚拟地址、线性地址、物理地址之间的转换。其实lab1中就用分段的段选择子虚拟地址和线性地址的转换，只是所有段选择子对应的基地址都是0，故而转换了和没转换看起来一样。
</p>

<p style="text-indent: 2em;">
  在lab2中，一切都有所改变了。最终的目标是实现：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">VirtualAddress=LinearAddress=0xC0000000+PhysicalAddress</pre>

<p style="text-indent: 2em;">
  为了实现这个目标，最开始会临时修改段基址，使得：
</p>

<pre class="brush:plain;toolbar:false">VirtualAddress=0xC0000000+LinearAddress=0xC0000000+PhysicalAddress</pre>

<p style="text-indent: 2em;">
  而在启用了分页机制，又还没来得及恢复段基址前，甚至会出现：
</p>

<pre class="brush:plain;toolbar:false">VirtualAddress=0xC0000000+LinearAddress=0xC0000000+0xC0000000+PhysicalAddress</pre>

<p style="text-indent: 2em;">
  所以，整个启动段页式内存管理的过程涉及到各种地址之间的换算，只要弄明白了这一点，完成lab2的题目就压力不大了。
</p>

<p style="text-indent: 2em;">
  稍微有点麻烦的反倒不是后面几题，而是第一题，需要特别仔细，因为看起来似乎本来的程序就已经实现了要求，而事实上本来的程序对于链表的顺序没有维护，故只要加入对链表顺序的维护即可。
</p>

<p style="text-indent: 2em;">
  至于challenge，不得不说是相当麻烦，没有个几百行代码搞不定，最近实在是太忙了，没有时间，于是乎就pass了～～～
</p>

<p style="text-indent: 2em;">
  最后，还是附上lab2的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/10/lab2.zip">lab2.zip</a>
</p>
