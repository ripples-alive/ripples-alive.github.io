---
layout: post
title: 挂载网络文件夹后网络故障时文件操作命令卡死
modified: 2018-01-26T16:47:01+08:00
categories: technology
description:
date: 2018-01-26T15:18:02+08:00
---

挂载 NFS 或者 Samba 的时候，经常会由于网络故障导致挂载好的链接断掉。
此时如果尝试进行 ls、cd、df 等各种命令，只要与此目录沾上边，就会卡住。
如果使用了类似 oh-my-zsh 这种配置的，只要在网络目录中，弹出命令提示符前就会直接卡住。

这个时候第一反应就是 umount 掉出问题的目录，但是如果直接执行，umount 本身也会卡住。
这时我们需要加上参数 `-l`，进行 lazy umount，即先把网络文件系统从 file system hierarchy 上 detach 掉，至于各种 reference busy 的问题押后处理。
