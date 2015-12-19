---
title: ucore 操作系统实验 lab4
author: Ripples
layout: post
permalink: /ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab4/
views:
  - 402
categories:
  - technology
tags:
  - ucore
  - 内核线程
  - 操作系统
---
<p style="text-indent: 2em;">
  这个实验涉及到的信息有点多，解释了一个线程从产生到运行的全过程，不过好在需要填写的代码不多，注释也还算清楚，所以实现起来也不算困难。
</p>

<p style="text-indent: 2em;">
  然而，这次还是没能一番风顺，程序补充完成后，运行的时候却出现了莫名其妙的错误，无奈之下甚至于去对答案，但是却除了发现少了关闭中断的处理外，并没有发现什么问题。
</p>

<!--more-->

<p style="text-indent: 0em; text-align: center;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/blob.png" />
</p>

<p style="text-indent: 2em;">
  最后，在不断的尝试之后，终于发现，在将lab2中修改的代码default_init_memmap、default_alloc_pages、default_free_pages替换成标准的答案，代码就成功运行起来了。想来错误大概是出在了页的分配算法的实现，但着实没想清楚为什么会出现问题，或许等啥时候有时间了可以仔细研究下问题出在什么地方。
</p>

<p style="text-indent: 2em;">
  老规矩，还是附上lab4的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/lab4.zip">lab4.zip</a>
</p>
