---
title: 0CTF Misc300(PolyQuine) WriteUp
author: Ripples
layout: post
permalink: /0ctf-misc300polyquine-writeup/
views:
  - 271
categories:
  - CTF
tags:
  - 0CTF
  - CTF
  - Quine
  - WriteUp
---
<p style="text-indent: 2em;">
  看到这题的题目确实是让我有点不敢相信，各种以为是自己理解错了题目的意思，一开始真的很难相信，程序可以这么玩……
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">我们先看看题目描述：</span>
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">Different&nbsp;people&nbsp;see&nbsp;different&nbsp;me.
But&nbsp;I&nbsp;am&nbsp;always&nbsp;myself.
202.112.26.114:12321
Make&nbsp;the&nbsp;output&nbsp;of&nbsp;your&nbsp;program&nbsp;exactly&nbsp;the&nbsp;same&nbsp;as&nbsp;your&nbsp;source&nbsp;code.
All&nbsp;5&nbsp;correct&nbsp;required&nbsp;to&nbsp;get&nbsp;this&nbsp;flag
$python2&nbsp;--version
Python&nbsp;2.7.6
$python3&nbsp;--version
Python&nbsp;3.4.0
$gcc&nbsp;--version
gcc&nbsp;(Ubuntu&nbsp;4.8.2-19ubuntu1)&nbsp;4.8.2
$ruby&nbsp;--version
ruby&nbsp;1.9.3p484&nbsp;(2013-11-22&nbsp;revision&nbsp;43786)&nbsp;[x86_64-linux]
$perl&nbsp;--version
This&nbsp;is&nbsp;perl&nbsp;5,&nbsp;version&nbsp;18,&nbsp;subversion&nbsp;2&nbsp;(v5.18.2)&nbsp;built&nbsp;for&nbsp;x86_64-linux-gnu-thread-multi</pre>

<p style="text-indent: 2em;">
  想要一个程序同时支持python2、python3、c、ruby、perl感觉就已经够难的了，竟然还要求每个程序的输出都是自身！
</p>

<p style="text-indent: 2em;">
  然后这题还有个简化版本，只要支持其中任何三种语言就好，好吧，不得不说这看起来一点也不简单，对于某些想要自己写的队友而言……
</p>

<p style="text-indent: 2em;">
  当然，这种题其实果断应该是Google大法好，最后队友在网上找到了<a href="http://shinh.skr.jp/obf/poly_quine5.txt" target="_blank">这个</a>，同时支持C、Ruby、Python、Perl、Brainfuck，于是果断手动将Brainfuck去掉（其实本来就有个不带Brainfuck的<a href="http://shinh.skr.jp/obf/poly_quine.txt" target="_blank">版本</a>），然后交上去试了下，过了4个，但是python3过不了，然后拿python3跑下了，发现是语法问题，“<span style="text-indent: 32px;">print a%b,</span>”这句在py3里显然是过不了的，加上括号改成形式的结局就是最后会多一个空行，此时py2、py3都是一样了。
</p>

<p style="text-indent: 2em;">
  于是问题来了，怎么解决这个空行的问题呢？
</p>

<p style="text-indent: 2em;">
  我当时的想法是，首先我们发现def printf那里是单独为python写的一段代码，于是乎我们就可以在这里做文章了，它整个代码的关键是第七行最后那一个printf的调用，而这里为python定义的printf也只是在那里调用了，于是，果断在这里截取字符串和参数，使得在最后的换行不输出，从而弥补print本身带来的换行，于是这题就这么愉快的过掉了，最后得到的程序<a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/04/PolyQuine.txt" target="_blank">在此</a>，这里是用的那个<span style="text-indent: 32px;">本来就有个不带Brainfuck的版本改的，因为会短点，改完后一共420个字符</span>。
</p>

<p style="text-indent: 2em;">
  然后后来又听说了一个更好想的想法，也是对自己当时根本没往这里想醉了：既然print会多输出一个换行，那么我们用stdout的write不就好了么！
</p>

<p style="text-indent: 2em;">
  感觉这题就是很好的说明了Google的重要性，两题一共400分，要是自己写，只怕写到比赛结束也不一定能写出来了。
</p>
