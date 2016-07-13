---
layout: post
title: 加壳 ELF 逆向漫谈
modified: 2016-07-13T19:41:41+08:00
categories: hacking
description: 闲谈一些加壳 ELF 逆向技巧
tags: [ELF, reverse]
---

这些日子由于实验室的原因，看了几个 apk 病毒的 native so，摸索了半天终究还算有点收获。

刚看这些 so 的时候，拖进 IDA 各种报错也就算了，readelf、010 都报错，section 信息各种不正常，根本就是连个程序入口点都找不到。程序载完 so 直接结束的也有，根本没有 dump 的机会，反调也是一应俱全，只要上调试器，肯定挂不解释。

不过想来，既然程序能正常运行，那么它破坏的一定是那些不会影响运行的字段，关键的运行相关的信息，该是怎么样，还必须得是什么样。于是在网上搜集了下资料，开着 010 对比着看了下，精炼起来大概就下面几点：

<p id="read-more-anchor"/>

1. section 信息与运行无关，所以可以随便改甚至删除；
1. 在 section 和 segment 中都存在一个 dynamic 段，两者为同一个，且对程序运行而言至关重要，segment 中的 dynamic 不可随便破坏，是我们分析程序的关键；
1. so 的 entry 字段若为 0，则会被无视，否则会在 so 加载时执行；
1. 通过 `readelf -a` 分析 dynamic 的结果，我们可以找到 init 函数和 init array 的地址（这其中的代码是在程序 entry 之前执行的，基本上壳都少不了要在这里面做手脚），找到 init 之后，至少我们可以下好断点挂上调试器跟踪了；

其中 dynamic 不可随便破坏不代表完全不可改，有一个 so 就通过修改了 dynamic 段导致 `readelf -a` 拿不到任何有用的信息，直接 `readelf -a` 会得到：

```
...
There are no relocations in this file.
...
```

010 查看一下 dynamic 段会发现 p_offset_FROM_FILE_BEGIN 为 0：

![segment info](/images/2016/07/13/segment.png)

显然这个字段绝对不应该为 0，为 0 导致 readelf 读不到正确的 dynamic 段信息，而运行时既然能读到，那说明 dynamic 段还是被正常映射了。对比观察一下会发现，其实 dynamic 段跟着前面的那个 Loadable Segment 一起映射了，于是这里的 p_offset_FROM_FILE_BEGIN 可以算出来是 `0x41490 - 0x00042490 + 0x00042C34`，修正后 readelf 即可正常工作。
