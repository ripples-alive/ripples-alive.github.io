---
title: ucore 操作系统实验 lab8
author: Ripples
layout: post
permalink: /2015/01/11/ucore-%e6%93%8d%e4%bd%9c%e7%b3%bb%e7%bb%9f%e5%ae%9e%e9%aa%8c-lab8/
views:
  - 368
categories:
  - technology
tags:
  - ucore
  - 操作系统
  - 文件系统
---
<p style="text-indent: 2em;">
  说实在的，这次的lab写的确实有点迷迷糊糊，文件那块本来就分级比较复杂，涉及到从硬件到软件的整个过程，然后配上老师那坑爹的英语PPT，真是看的头都是大的。
</p>

<p style="text-indent: 2em;">
  这个系统都没有弄的特别清楚，自然做起lab来也就有会有点吃力，lab8相对于lab7新增的东西又太多，现在根本就没有时间慢慢看，只能做一步是一步的来。
</p>

<!--more-->

<p style="text-indent: 2em;">
  然后说下这个lab，实验中要求填写的两个部分的代码，代码量相对于之前的lab，可以说是多了很多，exercise 1还好，毕竟涉及到的函数不多，主要是分块的判断，exercise 2就相对来说要麻烦许多，整个load_icode全部是空的，要将一个程序从文件中加载，然后按照格式解析，最后设置好各种乱七八糟的东西后使其运行起来。
</p>

<p style="text-indent: 2em;">
  不过其实，如果只是为了完成任务的话，这里完全可以从之前lab的load_icode函数中复制过来，然后进行修改，毕竟两个程序差别最大的地方仅仅只是lab8中需要从文件读入，那么我们就将之前lab中所有从的缓冲区binary中获取的信息相应的改为从文件中读取对应位置即可。最后，lab8中load_icode再多的一点改变就是需要给运行的程序传递参数了，那么我们可以仿照Linux下参数的传递格式将参数放在栈中，参数的格式玩了这么久的溢出，可以说是弄的相当清楚了，轻松搞定不成问题。
</p>

<p style="text-indent: 2em;">
  对比了下答案，发现我的代码还有问题，没有关文件，不过不得不说，如果不看答案，我肯定是想不到这点的，因为觉得开关文件都是在这个函数之外做的，毕竟传进来的参数是已经开好的文件。
</p>

<p style="text-indent: 2em;">
  至此，整个操作系统的实验也就算完成了，实验一路做下来，不得不承认收获确实不少，不过还是出于时间问题，有很多东西并没有弄的很清楚，challenge也还有很多没有做，如果有时间将来回来做一下其实也是不错的，不过感觉就以我这惰性估计难了……
</p>

<p style="text-indent: 2em;">
  最后，还是老规矩，附上lab8的实验报告和修改的代码。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/lab8.zip">lab8.zip</a>
</p>
