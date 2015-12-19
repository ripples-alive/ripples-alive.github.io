---
title: OverTheWire Vertex Level 7 → Level 8
author: Ripples
layout: post
permalink: /overthewire-vertex-level-7/
views:
  - 617
categories:
  - CTF
tags:
  - buffer overflow
  - vortex
  - wargame
  - WriteUp
---
<p style="text-indent: 2em;">
  之前为了比赛，在网上做题，后来看到了OverTheWire，感觉不错，就试了下，既然开博客了，就写点题解吧，也方便自己以后回顾，现在先补一篇，剩下的有时间慢慢补。
</p>

<p style="text-indent: 2em;">
  原题参见<a href="http://overthewire.org/wargames/vortex/vortex7.html" target="_blank">这里</a>。
</p>

<!--more-->

```c
int main(int argc, char **argv)
{
    char buf[58];
    u_int32_t hi;
    if((hi = crc32(0, argv[1], strlen(argv[1]))) == 0xe1ca95ee) {
        strcpy(buf, argv[1]);
    } else {
        printf("0x%08xn", hi);
    }
}
```

<p style="text-indent: 2em;">
  题目很明确，就是要对上面的代码进行溢出，得到shell。
</p>

<p style="text-indent: 2em;">
  代码很简洁，先对输入的参数进行crc32，要求得到的值为0xe1ca95ee，然后一句没有长度检查的strcpy很显然就是注入点，strcpy后直接就结束程序了，所以应该是通过溢出覆盖函数返回地址了。
</p>

<p style="text-indent: 2em;">
  那么，首先要解决的问题就是那个crc32了，上网搜了下（题目的Reading Material中也给了<a href="http://www.woodmann.com/fravia/crctut1.htm" target="_blank">讲解</a>），CRC就是循环冗余码，好吧，没想到这学期计原刚听过的竟然在这里遇到了。循环冗余码很明显是个容易制造冲突的算法，谁叫别个本来就不是专门设计来防恶意篡改的<span style="text-decoration: line-through;">（正所谓防君子不防小人）</span>。
</p>

<p style="text-indent: 2em;">
  关于如何解决CRC算法，在<a href="http://www.woodmann.com/fravia/crctut1.htm" target="_blank" style="white-space: normal;">讲解</a>中已经说明的很清楚，大致总结一下就是：对于只知道CRC结果要产生字符串的时候，就是要解决如何在一个已知字符串后面添加一些字符来把其CRC值改变成我们需要的值，因为CRC就是做了个二进制除法（用异或代替了普通的加减法），所以我们只需要能控制被除数末尾的和除数位数相同的位数，就肯定可以随意控制余数，也就是CRC值了，所以解决CRC32只需要在后面增加4个字节（32位）就可以了。确定了增加的位数之后，按照CRC的算法列个方程，求解即可。
</p>

<p style="text-indent: 2em;">
  后面附件中有我写的python版本的CRC Crack，其中CRC Table也可以选择在网上搜出来直接复制，本人强迫症，所以就自己写了生成。
</p>

<p style="text-indent: 2em;">
  接下来就是制作payload了，函数返回地址位置好找，gdb一下就可以看到buf和返回地址的位置差了74个字节，接下去的问题就是将返回地址改成什么。最简单的办法，就是利用环境变量，在之前的Level 4 → Level 5中我们就研究过栈空间的分布，环境变量的位置基本就在栈底了，而由于是静态栈，栈底的位置我们很容易得到，那么，我们将shellcode放在环境变量中，避免精确计算，可以加入一些nop，最后将返回值地址指向接近栈底的位置即可。
</p>

<p style="text-indent: 2em;">
  于是方案如下：
</p>

<p style="text-indent: 2em;">
  argv[1] = "x90" * 74 + (栈底地址 + 250) + CRC矫正值
</p>

<p style="text-indent: 2em;">
  env[0] = "x90" * 500 + shellcode
</p>

<p style="text-indent: 2em;">
  源代码参见附件。
</p>

<p style="text-indent: 2em;">
  到此，此题已经解决，但由于最开始脑残，总想着把shellcode直接放到argv[1]中，那这样虽然麻烦了点，但也确实是可行的，方法如下：
</p>

<p style="text-indent: 2em;">
  因为这样我们需要知道buf的准确位置，我尝试通过gdb，set follow-fork-mode child希望跟进子程序，从而获得目标程序被我的程序调用时的栈位置，结果发现在服务器上，gdb跟不进去，fork子程序会失败，于是乎写了个小程序通过同样的方法同样的参数被启动程序调用来获取ebp的值。
</p>

```c
#include <stdio.h>

int getebp()
{
    __asm__("pop %eaxn"
            "push %eax");
}

int main(int argc, char *argv[])
{
    unsigned int ebp = getebp();
    printf("%08x", ebp);
}
```

<p style="text-indent: 2em;">
  获取到ebp后，ebp + 0<a></a>x4就是返回地址的位置了，那么我们只需要将返回地址修改为ebp + 0<a></a>x8即可。
</p>

<p style="text-indent: 2em;">
  即：argv[1] =&nbsp;"x90" * 74 + (ebp + 0<a></a>x8) + shellcode + CRC矫正值。
</p>

<p style="text-indent: 2em;">
  源代码参见附件。当然，如果担心计算不精确，也可以在shellcode前面加上一些nop，增大命中范围。
</p>

<p style="text-indent: 2em;">
  PS：我尝试让argv[1] = "x90" * (74 &#8211; len(shellcode)) +&nbsp;shellcode + (ebp + 0<a></a>x8) + CRC矫正值，结果程序崩溃，没能出shell，不知道为何。
</p>

<p style="text-indent: 2em;">
  总得来说，在前面的题的经验下，这道题并不难，只是在之前完全不了解CRC的情况下，还是要花点时间研究下。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/08/vortex7.zip">vortex7.zip</a>
</p>
