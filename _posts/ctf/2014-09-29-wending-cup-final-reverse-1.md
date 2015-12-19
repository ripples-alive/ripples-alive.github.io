---
title: 中软吉大•问鼎杯决赛逆向类（一）
author: Ripples
layout: post
permalink: /%e4%b8%ad%e8%bd%af%e5%90%89%e5%a4%a7%e2%80%a2%e9%97%ae%e9%bc%8e%e6%9d%af%e5%86%b3%e8%b5%9b%e9%80%86%e5%90%91%e7%b1%bb%ef%bc%88%e4%b8%80%ef%bc%89/
views:
  - 820
categories:
  - CTF
tags:
  - reversing
  - WriteUp
  - 中软吉大•问鼎杯
---
<p style="text-indent: 2em;">
  这题我拿到手的就这个<a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/crackme.exe.zip" target="_self">程序</a>，想来逆向的题，也不需要什么题目描述了。
</p>

<p style="text-indent: 2em;">
  作为逆向的题目，第一反应当然是打开反汇编器，但是这题是个GUI程序，面对如此大一个程序，傻傻的理解所有代码显然是不划算的，那么我们要找些捷径来解决问题。
</p>

<p style="text-indent: 2em;">
  首先，我们运行这个程序，运行界面很明确就一个输入框和确定按钮，我们随便输入一点东西并点击确定，会弹出一个提示窗。那么，我们就从这个提示窗下手，因为在这个提示窗的之前就应该是判断密码是否正确的地方。
</p>

<!--more-->

<p style="text-indent: 2em;">
  想要找到这个提示窗，我们通常是给MessageBox函数下断点或者对提示窗中现在内容下内存断点。在这里，我们采取后一种办法，首先在源程序中查找字符串"Try again!"，找到这个字符串之后，再我们下断点之前，我们发现，在这个字符串的后面，紧跟着一句The key is :&#8230;.CraCKS0EAsY!，这么看来，我们应该是已经找到了我们需要的东西，连断点都不用了。
</p>

<p style="text-indent: 2em;">
  我们在输入框中输入"CraCKS0EAsY!"（PS：注意中间是个数字0，而不是字母O，我刚开始就被这个给坑了，果然还是复制粘贴好！），点击确定，程序如我们所愿，弹出成功的提示窗，此题便得以解决。
</p>

<p style="text-indent: 2em;">
  这题的难道着实很低，只是刚看的时候显得有点吓人而已。
</p>

* * *

<p style="line-height: 16px; text-indent: 2em;">
  由于有人说答案错了，于是乎又跑去看了下程序，通过下内存断点，发现了在程序相对地址0x<a></a>510的地方有点击事件的处理函数，函数内就是将输入框字符取出，并与"CraCKS0EAsY!"这个字符串做了个__mbscmp，根据比较结果分别弹出不同的内容。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  然后消息分发函数应该是在程序相对地址0x157B5处，对编号0xC的消息调用了上面的处理函数。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/09/crackme.exe.zip">crackme.exe.zip</a>
</p>
