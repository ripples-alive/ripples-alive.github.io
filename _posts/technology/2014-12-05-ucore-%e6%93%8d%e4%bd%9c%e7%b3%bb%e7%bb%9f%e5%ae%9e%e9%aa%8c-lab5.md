---
title: ucore 操作系统实验 lab5
author: Ripples
layout: post
views:
  - 355
categories:
  - technology
tags:
  - ucore
  - 子进程
  - 操作系统
---
<p style="text-indent: 2em;">
  不得不说，这次的lab绝对是已经做过的几个lab中最麻烦的一个，为了做lab5，我们需要做的除了将lab5的代码补全，还需要修改lab1-lab4中的部分代码。
</p>

<p style="text-indent: 2em;">
  然后，还有个坑爹的事情就是，GitHub里面自带的答案make grade竟然只能得91分（满分150），我也是给亮瞎了，然后这题错了其实又不太好调，然后还听有同学说它同一段代码，曾经能得满分，后来好像是重装了还是怎么滴，就突然成no output了。
</p>

<!--more-->

<p style="text-indent: 2em;">
  做的时候，还有个比较大的坑就是set_links这个函数中竟然将proc插入到了proc_list中，然后还把nr_process加了1，需要在do_fork中删除之前写的这两个操作，真不知道写的人是怎么想的，这两个步骤是怎么会想到加在set_links里面的，还好被同学提醒了，不然真不知道得被这个坑多久。
</p>

<p style="text-indent: 2em;">
  最后需要注意的大概就是check的时候需要把lab1中加的print_ticks功能，不然可能会导致make grade的时候个别需要运行时间比较长的check过不了。
</p>

<p style="text-indent: 2em;">
  还是老规矩，附上lab5的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/12/lab51.zip">lab5.zip</a>
</p>
