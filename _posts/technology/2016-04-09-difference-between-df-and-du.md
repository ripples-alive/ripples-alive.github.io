---
layout: post
title: df 与 du 的区别
modified: 2016-04-10T17:08:22+08:00
categories: technology
description: 磁盘空间总是不足，用 df 与 du 查看却发现结果差距很大……
tags: [df, du, log]
date: 2016-04-09T18:08:33+08:00
---

最近，某台服务器总是收到阿里云的提醒说磁盘空间不足，
今天，由于管理的人在外面，于是让我上去帮忙清下，
出于好奇，我决定顺便看下到底为什么磁盘成天满，结果就发现了个『奇怪』的现象……

要知道为什么，上来肯定是 du 找一下是哪个文件大咯，
结果意外的发现，根目录总共也就 6.5G，而磁盘本身有 20G，
而用 dh 一看，可以发现磁盘占用了 19G 多，两者结果差距这么大，着实是第一次见。
思前想后也没想明白是为啥，只好求助万能的 Google，结果发现原来是个很寻常的问题。

df -> disk free，是通过文件系统之间获取的空间大小信息，
du -> dist usage，是通过计算所选文件夹中所有文件的大小的和得到结果的，
暂且不说 inode 信息等导致的空间占用（影响肯定不会有这次遇到的这么大），会导致两者结果出现偏差最常见的一个原因就是有些被『删除』了的幽灵文件。
当我们删除一个文件的时候，其实这个可能并不会立即被从文件系统中释放掉，因为可能会有进程还在使用这个文件的句柄，
只有当所有文件句柄都被释放了之后，相应的文件才会被真正从文件系统中释放掉。

<p id="read-more-anchor"/>

鉴于 Linux 上一切皆文件，lsof 这个命令简直就是神器，这里我们可以用 `lsof | grep deleted` 列出所有文件被删除了的文件句柄。
然后看了一下，发现果然服务器上有着大量被这样的句柄，
仔细一看，发现全都是 `/var/log` 下面的日志文件，
于是 ls 看了下，发现 `/var/log` 下面空空如也，
问了下才知道，原来是管理的人觉得日志太多，又没用，就直接 rm 了 :joy:。

问题找到之后，解决就好办了，把各个服务都重启下，好释放掉原来的句柄，暴力点可以直接重启机器。
不过，重启 nginx 的时候，`service nginx restart` 给了个 failed，最讨厌这种不给原因的 failed 了 :rage:。
不过好在用 `nginx -t` 跑了下之后，发现说 `/var/log/nginx/access.log` 和 `/var/log/nginx/error.log` 不存在，
虽然不理解 nginx 为什么不能自动建文件夹，不过手动 `mkdir /var/log/nginx` 一下也不是什么大事，建好后重启 nginx 就好了。

重启完成后，df 再看就发现，磁盘一下空了 12G 出来（被 DDOS 之后日志真是太残暴 :flushed:），感觉一下整个世界都好了……
