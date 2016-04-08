---
title: ucore 操作系统实验 lab6
author: Ripples
layout: post
permalink: /ucore-操作系统实验-lab6/
views:
  - 455
categories:
  - technology
tags:
  - ucore
  - 操作系统
  - 进程调度
---
<p style="text-indent: 2em;">
  好不容易挤出时间，总算是把lab6完成了，这个lab倒也不算难，不过坑倒是有不少。
</p>

<p style="text-indent: 2em;">
  首先是有个小坑，在初始化proc的时候，要将proc->run_link初始化为一个空链表，而不是NULL，虽然初始化为NULL可以过stride算法，但不知道为什么RR算法的时候，他非要assert是空链表才干……
</p>

<!--more-->

<p style="text-indent: 2em;">
  不过前面那个坑都算是小的了，一查就知道了，后面的都是注释在坑。
</p>

<p style="text-indent: 2em;">
  首先是stride_dequeue里面，不知道是注释故意的，还是忘记了，没有写出要把rq的proc_num减下来，虽说这样对lab6的运行没有什么影响，但总归错了还是不好的。
</p>

<p style="text-indent: 2em;">
  然后不得不提的就是trap.c中时间中断的处理，注释里写着just add ONE or TWO lines of code，于是毫不犹豫的按其要求调用run_timer_list，结果在最后测试priority就惨死了。因为lab5遗留下来的部分，每个100 ticks都会将当前进程的need_resched设成1，放弃CPU，直接就影响了最后运行时间的测量，都不想说什么了。
</p>

<p style="text-indent: 2em;">
  最后可能需要注意的就是，可能在lab6要求修改的代码全部改好之后，还是会过不了，看下priority.c的代码便会发现，很可能是精度导致的，我们只需要将<span style="text-indent: 32px;">priority.c中的MAX_TIME改大，使其运行足够长的时间即可。</span>
</p>

<p style="text-indent: 2em;">
  然后还是老规矩，附上lab6的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/12/lab6.zip">lab6.zip</a>
</p>
