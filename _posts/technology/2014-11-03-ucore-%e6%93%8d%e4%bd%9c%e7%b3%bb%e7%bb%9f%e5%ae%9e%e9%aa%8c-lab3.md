---
title: ucore 操作系统实验 lab3（含Challenge）
author: Ripples
layout: post
permalink: /2014/11/03/ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab3/
views:
  - 382
categories:
  - technology
tags:
  - ucore
  - 内存管理
  - 操作系统
---
<p dir="ltr" style="text-indent: 2em;">
  做完lab3之后还是得说，lab3也挺水的了，基本上也就是按照注释来一步步的写，需要用的函数都在注释中给了，基本没有自己需要找得函数或者常量定义。
</p>

<p dir="ltr" style="text-indent: 2em;">
  lab3主要是涉及到了完整的虚实地址映射，完成虚拟内存管理器，虽然有和硬盘交换的部分，但该部分的代码已经给出，不用自己完成。
</p>

<!--more-->

<p dir="ltr" style="text-indent: 2em;">
  lab3虽说不难，但自己脑抽了一下，于是就被自己给坑了。在完成_fifo_swap_out_victim函数的时候，其中需要将链表指针转化为其对应的Page的指针，当时就记得在前面的lab中见到是有个宏可以把对一个数据结构中的一个成员的指针转换成对该数据结构的指针，但是不记得在哪里了，于是乎就死命的找啊找。最后好不容易灵光一闪，想起了负责lab2中page和list entry转化的函数，才好不容易通过其找到想找的东西。
</p>

<p dir="ltr" style="text-indent: 2em;">
  然后几经波折的使用好后，竟发现，这不就是直接用le2page就好了，只不过把第二个参数换成pra_page_link即可。
</p>

<p dir="ltr" style="text-indent: 2em;">
  至于chanllege，看了下感觉不难，不过暂时太忙，就先放着了，有时间了再说吧……
</p>

<p dir="ltr" style="text-indent: 2em;">
  最后，还是附上lab3的实验报告和修改的代码。
</p>

* * *

<p style="text-indent: 2em;">
  这次的challenge确实很水不错，不过没想到竟然有一个大坑，它ucore的程序是有点问题的，在进行页换入的时候，ucore并没有将换入的页的pra_vaddr设置为对应的线性地址。而如果不做这个设置的话，之前的FIFO算法本来是会有问题的，但是由于它FIFO中实现的check里面，是5个虚拟页在4个物理页上进行替换，故而无论怎么check也不会遇到程序崩溃的情况。
</p>

<p style="text-indent: 2em;">
  最后鉴于改ucore本身的代码不太好，就在实现的_ec_map_swappable中对page的pra_vaddr进行了一下设置，以保证程序能够正确运行不崩溃。
</p>

<p style="text-indent: 2em;">
  这个问题实在是太坑了，本来程序老早就写对了，却没想到最后被这个坑给搞的把整个页置换debug了一遍，花了好长时间，本来就是因为时间不够才没做之前challenge，这次以为很快能搞定才做的，没想到却成了这番结果……
</p>

<p dir="ltr" style="text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" style="line-height: 16px; text-indent: 2em;" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/lab3.zip" style="line-height: 16px; text-indent: 2em;">lab3.zip</a>
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/lab3（含challenge）.zip">lab3（含challenge）.zip</a>
</p>
