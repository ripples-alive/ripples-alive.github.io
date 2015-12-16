---
title: Defcon qualifier 2015 knockedupd(reverse1) WriteUp
author: Ripples
layout: post
views:
  - 319
categories:
  - CTF
tags:
  - CTF
  - DEFCON
  - knockd
  - reversing
  - WriteUp
---
<p style="text-indent: 2em;">
  这题作为逆向题的第一题，难道自然不会太高，但其实如果没想到点，硬生生的去逆的话，也是会足够蛋疼的……
</p>

<p style="text-indent: 2em;">
  和其他逆向题一样，这题题目描述还是也是看不出啥，然后就不得不说很佩服某逆向机的搜索能力了，当我们还在逆的时候，他就直接在GitHub上找到了这个<a href="https://github.com/jvinet/knock" target="_blank">东西</a>。
</p>

<!--more-->

<p style="text-indent: 2em;">
  仔细看下会发现，这个里面的服务端和我们这里的服务端是一样一样滴，只是配置文件会不同，<span style="text-indent: 32px;">那么我们所要做的就是找到对应的端口，按顺序knock一下，就可以开放目标端口了。</span>
</p>

<p style="text-indent: 2em;">
  于是乎动态调试，可以拉出pcap_compile的参数：
</p>

<pre class="brush:plain;toolbar:false">((udp&nbsp;dst&nbsp;port&nbsp;13102&nbsp;or&nbsp;18264&nbsp;or&nbsp;18282)&nbsp;or&nbsp;(udp&nbsp;dst&nbsp;port&nbsp;31717&nbsp;or&nbsp;35314&nbsp;or&nbsp;39979&nbsp;or&nbsp;15148&nbsp;or&nbsp;14661))</pre>

<p style="text-indent: 2em;">
  我们可以通过猜测或者暴力的方式来弄，这里完全可以猜测要依次以udp的方式knock 13102、18264、18282三个端口或者31717、35314、39979、15148、14661这五个端口，其实前三个就是这题的，而后5个是后面一道题的。
</p>

<p style="text-indent: 2em;">
  这里knock，我们可以用刚那个GitHub程序中的客户端，编译后运行：
</p>

<pre class="brush:bash;toolbar:false">./knock&nbsp;52.74.114.1&nbsp;13102:udp&nbsp;18264:udp&nbsp;18282:udp&nbsp;--delay&nbsp;5</pre>

<p style="text-indent: 2em;">
  于是就轻松完成了knock，不过这里的&#8211;delay如果不加的话是会跪的，刚开始由于没加，怎么也knock不开……
</p>

<p style="text-indent: 2em;">
  然后我们看规则，可以找到：
</p>

<pre class="brush:bash;toolbar:false;">/bin/bash&nbsp;-c&nbsp;"/sbin/iptables&nbsp;-I&nbsp;KNOCKEDUPD&nbsp;1&nbsp;-s&nbsp;%IP%&nbsp;-p&nbsp;tcp&nbsp;--dport&nbsp;10785&nbsp;-j&nbsp;ACCEPT"
/bin/bash&nbsp;-c&nbsp;"&nbsp;sbin/iptables&nbsp;-I&nbsp;KNOCKEDUPD&nbsp;1&nbsp;58&nbsp;-s&nbsp;%IP%&nbsp;-p&nbsp;tcp&nbsp;--dport&nbsp;9889&nbsp;-j&nbsp;ACCEPT"</pre>

<p style="text-indent: 2em;">
  那么，knock完之后，这题开放的是10785端口，后面一题开放的是9889端口，对于这题而言，knock完之后，直接nc上去就可以拿到flag了。
</p>
