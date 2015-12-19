---
layout: post
title: Bypass ASLR with execve
modified: 2015-12-19T16:44:42+08:00
categories: hacking
description: 一直以为 execve 会重新加载 so，所以 execve 之前读取的地址是没有任何意义的，但是今天却发现事情并不总像想象中那样。
tags: [ASLR, execve]
---

昨天做作业的时候，本来是很简单的溢出题吧，老师的意思本来就是直接跳到 shellcode 上执行就好。
但是仔细一看却发现，所给虚拟机上是开了 ASLR 的，而程序是以命令行参数形式进行的输入，
那么也就是说，我们必须要在程序开始运行前就确定下来缓冲区的地址，显然这在 ASLR 的防护下是不现实的。
当然，毕竟是 32 位的系统，暴力肯定是可以跑出来的，不过这题显然不会是要这么麻烦，那么肯定是有什么地方出了问题。

想到作业是给了参考答案的，然后果断打开答案看看，发现程序就是直接读取了自身的栈地址后，execve 到目标程序上，
按照之前的印象，这样显然是错的，execve 过去后，栈地址应该是会被 ASLR 重新随机的，
然后抱着怀疑的态度，运行了参考答案好几次，却神奇的发现每次都成功拿到了 shell，于是当时就开始怀疑起自己的印象来了……

最后，几经波折，找到了这么[一篇 blackhat 上的文章](https://www.blackhat.com/presentations/bh-europe-09/Fritsch/Blackhat-Europe-2009-Fritsch-Bypassing-aslr-whitepaper.pdf)，才解开了疑惑。
文章中主要是具体讲了 ASLR 的部分可预测性，与我们这里关系不太大，我们这里主要是因为其中提到的一点：

<p id="read-more-anchor"/>

```
If an attacker manages to call execve(2) with his desired target process within one jiffy (i.e. 4ms) after his process was launched, then the output of the prf function will be the same since neither pid p nor jiffies j changed. The secret s did not change either and would most likely also not change if re-keying was in fact every second. Thus the target process will use the very same randomization.
```

也就是说，在 one jiffy (i.e. 4ms) 的时间内，ASLR 出来的值是固定的，那么我们只要是在 4ms 内 execve 到新程序上的，那么我们之前读取到的栈地址就是正确的地址。
感觉不得不说，这简直是神器啊，就读个地址就可以 execve 了，4ms 简直就是绰绰有余，破 ASLR 不解释啊。

然后测试了一下，在所给虚拟机上，把攻击程序 sleep 一会之后再 execve 就会发现果然拿不到 shell 了。
然后又试了下在自己的 kali 上情况如何，结果似乎发现不 sleep 好像也是不行了，每一次 execve 之后都重新随机了，果然是已经修复了么。
于是到 GitHub 上 Linux 源代码里找了下 commit history，大概看到下面这几个提交就是跟文章中提到的有问题的 get_random_int 有关的：

* https://github.com/torvalds/linux/commit/8a0a9bd4db63bc45e3017bedeafbd88d0eb84d02#diff-8a7983688893b4a00f280f89e6e6c559

* https://github.com/torvalds/linux/commit/26a9a418237c0b06528941bca693c49c8d97edbe#diff-8a7983688893b4a00f280f89e6e6c559

* https://github.com/torvalds/linux/commit/61875f30daf60305712e25b209ef41ced2635bad#diff-8a7983688893b4a00f280f89e6e6c559

可以发现大概从 v2.6.30-rc5 开始这个问题就已经修掉了，get_random_int 中增加了墒，不存在说一个 jiffy 内返回的值都是一样的情况了。
