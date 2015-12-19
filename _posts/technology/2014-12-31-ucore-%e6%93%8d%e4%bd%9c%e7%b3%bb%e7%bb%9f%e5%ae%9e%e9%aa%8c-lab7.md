---
title: ucore 操作系统实验 lab7
author: Ripples
layout: post
permalink: /ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab7/
views:
  - 370
categories:
  - technology
tags:
  - ucore
  - 信号量
  - 操作系统
  - 管程
---
<p style="text-indent: 2em;">
  一学期没听课，看到lab7的内容的时候，当时就跪了，信号量和管程完全就没听过，无奈之下只有先去看下PPT了。
</p>

<p style="text-indent: 2em;">
  看PPT的时候看的真的是头都是大的，进程同步与互斥果然是个要命的事，不过好在最终要补全的程序并不复杂。
</p>

<!--more-->

<p style="text-indent: 2em;">
  不过比较蛋疼的是，这次的PDF也是忽悠的飞起，首先说是要改lab1-lab6的代码，可以到头来根本不用改，完全拷过来即可，然后号称make grade在填充完前面的lab之后并不能得满分，可实际是直接就满分了，于是乎为了测试最后完成lab7后哲学家问题是否正确了需要自己盯着make qemu的结果慢慢看……
</p>

<p style="text-indent: 2em;">
  这次的程序基本也没有什么坑，就是要分清楚信号量的signal和wait操作分别对应哪个函数。大概最难理解的地方就是哲学家问题的实现了，不过实际上其实就算不理解，直接翻译伪代码也可以做……
</p>

<p style="text-indent: 2em;">
  最后，还是老规矩，附上lab7的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/12/lab7.zip">lab7.zip</a>
</p>
