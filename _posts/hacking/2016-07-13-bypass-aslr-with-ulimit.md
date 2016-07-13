---
layout: post
title: Bypass ASLR with ulimit
modified: 2016-07-13T17:22:55+08:00
categories: hacking
description: ulimit 作为限制程序资源使用的神器，对 ASLR 的绕过也能发挥奇效。
tags: [ASLR, ulimit]
---

也不知道是不是天气热了，这两天异常焦躁，于是找点乐子，便跑去看了看 pwnable.kr，然后，然后就卡住了（我可怜的脑细胞……

这次卡的题是 ascii，然后发现有个简化版 ascii_easy，然后看完还是发现不会，看到提示说 ulimit，当时就懵了，这又是神马黑科技，ulimit 不是用来限制资源的么，跟这题有什么关系……

在网上查了一圈后发现，通过 `ulimit -s unlimited` 可以禁用 ASLR，实测一下后发现，so 地址确实固定了，不过栈地址并不固定。然后看了下原理，大概就是 `ulimit -s` 将栈地址设成无限制后，系统会采取一种与平常不同的方式计算库基地址，而在 32 位模式下采取此方式时没有加入随机量，于是地址便固定了。最后，竟然还发现了一个最近的 [CVE](https://www.exploit-db.com/exploits/39669/) 讲这个问题的，感觉也是醉了:joy:。

so 地址固定后，ascii_easy 其实就很简单了，随便在 libc 里找找函数和字符串用一用就好，注意下 ascii 字符的要求就好。至于 ascii 的话，由于静态了，需要借助 vdso 来跳一下 shellcode 了。
