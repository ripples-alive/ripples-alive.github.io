---
title: SCTF Code500 WriteUp
author: Ripples
layout: post
views:
  - 514
categories:
  - CTF
tags:
  - CTF
  - SCTF
  - 搜索
---
<p style="text-indent: 2em;">
  code500这道题其实并不难，只是由于自己太错误的执着，导致花了不少时间。
</p>

<p style="text-indent: 2em;">
  题目描述如下：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">女神Alice找回了钱钱，女神非常开心，然后就去洗澡了
这时Mallory已经逃走了！
plusplus7希望抓住Mallory，于是决定开挖掘机去追他。
刚刚坐上挖掘机，结果发现启动挖掘机需要把一些插头按照一定的顺序连接起来...
插头有两面，每个插头的两端都有颜色
如：
绿色和黑色插头&nbsp;“G:B”
当插头的两个面颜色相同，才能被接在一起
如：
对接成功&nbsp;“G:B&nbsp;=&nbsp;B:V”
对接失败&nbsp;“G:B&nbsp;*&nbsp;C:G”
插座也有两面，一面有颜色，一面无颜色以"X"表示
如：
左插座&nbsp;“X:B”
右插座&nbsp;“C:X”
插座只有两个，且不可移动
如：
“X:B&nbsp;*&nbsp;C:G&nbsp;=&nbsp;G:E&nbsp;*&nbsp;B:C&nbsp;*&nbsp;E:X”
对挖掘机的siri输入指令，“3&nbsp;3&nbsp;1”
“X:B&nbsp;=&nbsp;B:C&nbsp;=&nbsp;C:G&nbsp;=&nbsp;G:E&nbsp;=&nbsp;E:X”
那么，问题来了...
218.2.197.248:7777</pre>

<p style="text-indent: 2em;">
  第一眼看到题目的时候，确实是被那个指令给搞懵了，但是稍微恢复了下神智之后，到nc上去试，试了半天总算猜对了指令的含义：
</p>

<p style="text-indent: 2em;">
  a b c 表示将第a到b个插头移动到第c个插头的前面（这里也将插座算入，编号从0开始）
</p>

<p style="text-indent: 2em;">
  弄明白指令的含义之后，说实在的，对于该怎么做其实还是没有什么好的想法（因为操作有步数限制，要在规定步数内完成才算有效）。
</p>

<p style="text-indent: 2em;">
  但是观察nc看到的数据发现，似乎所有的字母永远都只出现2次，即都是唯一匹配的，<span style="text-indent: 2em;">然后当时脑子里很快就有了个猜测：</span>
</p>

<p style="text-indent: 2em;">
  对于两个相邻的未匹配的插头，假设编号为a &#8211; 1和a，left[i]、right[i]分别表示第i个插头的两个面的颜色，那么，一定存在b、c，使得left[a] = right[c]、right[a &#8211; 1] = left[b]，如果b、c满足b < c且[a &#8211; 1, a]与[b, c]不相交，那么我们这一步就将[b, c]这一段移动到a &#8211; 1与a中间，即指令为b c a。
</p>

<p style="text-indent: 2em;">
  这个猜想很好理解，至于这样做是不是就是最好的抉择没太想清楚怎么证明，但感觉是可行的。
</p>

<p style="text-indent: 2em;">
  然后，对于不满足这个猜想的条件的，我最开始是采取随便取一个的方式，结果花了很多时间也没能解决，经常会比要求的步数多1步，最多好像是能跑到30多轮吧，但是看了下code200的数据组数，50组，于是觉得这题也是这个数，于是就对跑不出来的特例暴搜得到答案后硬编码判断，然后就成功的跑到了68轮，感觉已经无法再爱了……
</p>

<p style="text-indent: 2em;">
  最后，无奈，重写了一下搜索的代码，加上上面的那个猜测作为A*，同时限制了一下，保证每次移动都不会将两个已经配对的插头分开，然后就很轻松就秒了，最后结果有92轮……
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;socket

def&nbsp;get_one(s):
&nbsp;&nbsp;&nbsp;&nbsp;try:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;=&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;ret.find(":Xn")&nbsp;==&nbsp;-1&nbsp;and&nbsp;ret.find("SCTF")&nbsp;==&nbsp;-1:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;+=&nbsp;s.recv(100)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;ret
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;ret[:-1]
&nbsp;&nbsp;&nbsp;&nbsp;except&nbsp;Exception,&nbsp;e:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;Fail&#39;,&nbsp;ret
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;ret

def&nbsp;check(now,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(1,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[i&nbsp;-&nbsp;1][2]&nbsp;!=&nbsp;now[i][0]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;False
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True

def&nbsp;swap(now,&nbsp;i,&nbsp;j,&nbsp;k,&nbsp;deep):
&nbsp;&nbsp;&nbsp;&nbsp;history[deep]&nbsp;=&nbsp;[i,&nbsp;j,&nbsp;k]
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;k&nbsp;&lt;&nbsp;i:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;now[:k]&nbsp;+&nbsp;now[i:j&nbsp;+&nbsp;1]&nbsp;+&nbsp;now[k:i]&nbsp;+&nbsp;now[j&nbsp;+&nbsp;1:]
&nbsp;&nbsp;&nbsp;&nbsp;elif&nbsp;k&nbsp;&gt;&nbsp;j:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;now[:i]&nbsp;+&nbsp;now[j&nbsp;+&nbsp;1:k]&nbsp;+&nbsp;now[i:j&nbsp;+&nbsp;1]&nbsp;+&nbsp;now[k:]
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;now

def&nbsp;dfs(now,&nbsp;deep,&nbsp;max_deep,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;deep&nbsp;==&nbsp;max_deep:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;check(now,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;False
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(1,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[i&nbsp;-&nbsp;1][2]&nbsp;!=&nbsp;now[i][0]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;j&nbsp;in&nbsp;xrange(1,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[i&nbsp;-&nbsp;1][2]&nbsp;==&nbsp;now[j][0]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;start&nbsp;=&nbsp;j
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;j&nbsp;in&nbsp;xrange(0,&nbsp;size&nbsp;-&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[i][0]&nbsp;==&nbsp;now[j][2]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;stop&nbsp;=&nbsp;j
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;start&nbsp;&lt;=&nbsp;stop&nbsp;and&nbsp;(start&nbsp;&gt;&nbsp;i&nbsp;or&nbsp;stop&nbsp;&lt;&nbsp;i&nbsp;-&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;dfs(swap(now,&nbsp;start,&nbsp;stop,&nbsp;i,&nbsp;deep),&nbsp;deep&nbsp;+&nbsp;1,&nbsp;max_deep,&nbsp;size)
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(1,&nbsp;size&nbsp;-&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[i&nbsp;-&nbsp;1][2]&nbsp;==&nbsp;now[i][0]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;j&nbsp;in&nbsp;xrange(i,&nbsp;size&nbsp;-&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[j][2]&nbsp;==&nbsp;now[j&nbsp;+&nbsp;1][0]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;k&nbsp;in&nbsp;xrange(1,&nbsp;i):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[k&nbsp;-&nbsp;1][2]&nbsp;!=&nbsp;now[k][0]&nbsp;and&nbsp;dfs(swap(now,&nbsp;i,&nbsp;j,&nbsp;k,&nbsp;deep),&nbsp;deep&nbsp;+&nbsp;1,&nbsp;max_deep,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;k&nbsp;in&nbsp;xrange(j&nbsp;+&nbsp;2,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;now[k&nbsp;-&nbsp;1][2]&nbsp;!=&nbsp;now[k][0]&nbsp;and&nbsp;dfs(swap(now,&nbsp;i,&nbsp;j,&nbsp;k,&nbsp;deep),&nbsp;deep&nbsp;+&nbsp;1,&nbsp;max_deep,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;False

def&nbsp;get_answer(now,&nbsp;max_step):
&nbsp;&nbsp;&nbsp;&nbsp;now&nbsp;=&nbsp;now.replace(&#39;&nbsp;&#39;,&nbsp;&#39;&#39;).replace(&#39;=&#39;,&nbsp;&#39;*&#39;).split(&#39;*&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;size&nbsp;=&nbsp;len(now)
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(1,&nbsp;max_step&nbsp;+&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;dfs(now,&nbsp;0,&nbsp;i,&nbsp;size):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;history
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;i

history&nbsp;=&nbsp;[[]&nbsp;for&nbsp;i&nbsp;in&nbsp;range(10)]

s&nbsp;=&nbsp;socket.socket(socket.AF_INET,&nbsp;socket.SOCK_STREAM)
s.connect((&#39;218.2.197.248&#39;,&nbsp;7777))
round_count&nbsp;=&nbsp;1
ret&nbsp;=&nbsp;get_one(s)[len("Let&#39;s&nbsp;start&nbsp;this&nbsp;game!n"):]
while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;step&nbsp;=&nbsp;get_answer(ret,&nbsp;7)
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(step):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.send("%d&nbsp;%d&nbsp;%d"&nbsp;%&nbsp;(history[i][0],&nbsp;history[i][1],&nbsp;history[i][2]))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;=&nbsp;get_one(s)
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;Round&#39;,&nbsp;round_count,&nbsp;&#39;finsh.n&#39;
&nbsp;&nbsp;&nbsp;&nbsp;round_count&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;ret&nbsp;=&nbsp;get_one(s)[len(&#39;Round&nbsp;pass...n&#39;):]</pre>

<p style="text-indent: 2em;">
  重写代码的时间比之前小调优化的时间少了不知道多少倍，感觉大好的时光就这么浪费了，还好最后pwn400轻松秒了，不然估计真得为这个事情后悔好久~~~
</p>

* * *

<p style="text-indent: 2em;">
  关于代码写的丑的问题，各位看官就不用吐槽了，毕竟当时大脑已经是一半浆糊了，只要能跑出来，哪还有心情管代码美不美，直接在暴力上面改的，所以DFS都没有改成BFS，强行枚举了max deep，我也是服了我自己的……
</p>

<p style="text-indent: 2em;">
  这道题最后比赛结束之后竟然还被打电话challenge了，被质疑速度，看起来似乎标答应该不是搜索的样子，也不知道有没有哪位会愿意把标准解法分享一下了~~~
</p>
