---
title: 0CTF Crypto300(SATBeginner) WriteUp
author: Ripples
layout: post
permalink: /0ctf-crypto300satbeginner-writeup/
views:
  - 215
categories:
  - CTF
tags:
  - 0CTF
  - crypto
  - WriteUp
---
<p style="text-indent: 2em;">
  这题放的比较迟，题目描述（Try SAT if you fail in GRE :P）也是直接，虽然没敢看GRE，但是看着描述想来这题是GRE的简化版了的样子。
</p>

<p style="text-indent: 2em;">
  这题同样是直接给出了Server端的程序，不过不得不提一下的是，似乎是题放出来过了好久才附上的，我也是醉了，难怪刚开始看这题完全不明所以。
</p>

<!--more-->

<p style="text-indent: 2em;">
  这题首先有个sha1的检测，反正暴力就好，后面才开始关卡，我们直接看关键部分程序：
</p>

<pre class="brush:python;toolbar:false">def&nbsp;SATexam(k):
	var&nbsp;=&nbsp;range(1,&nbsp;k&nbsp;+&nbsp;1)&nbsp;*&nbsp;3
	random.shuffle(var)
	for&nbsp;i&nbsp;in&nbsp;xrange(k&nbsp;//&nbsp;2):
		v&nbsp;=&nbsp;random.choice(var)
		var[v]&nbsp;=&nbsp;-var[v]
	return&nbsp;var

def&nbsp;SATjudge(sat,&nbsp;answer):
	k&nbsp;=&nbsp;len(sat)&nbsp;//&nbsp;3
	assign&nbsp;=&nbsp;[0]&nbsp;*&nbsp;k

	assert(len(sat)&nbsp;==&nbsp;k&nbsp;*&nbsp;3)
	assert(len(answer)&nbsp;*&nbsp;8&nbsp;&gt;=&nbsp;k)&nbsp;#&nbsp;at&nbsp;least&nbsp;k&nbsp;bits

	i,&nbsp;j&nbsp;=&nbsp;0,&nbsp;0
	for&nbsp;i&nbsp;in&nbsp;xrange(k):
		assign[i]&nbsp;=&nbsp;(ord(answer[i&nbsp;//&nbsp;8])&nbsp;&gt;&gt;&nbsp;(i&nbsp;%&nbsp;8))&nbsp;&&nbsp;1

	for&nbsp;i&nbsp;in&nbsp;xrange(k):
		r&nbsp;=&nbsp;0
		for&nbsp;j&nbsp;in&nbsp;xrange(3):&nbsp;#&nbsp;neither&nbsp;SAT-1&nbsp;nor&nbsp;SAT-2,&nbsp;it&#39;s&nbsp;SAT-3&nbsp;<img src="http://localhost:7000/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" />
			v&nbsp;=&nbsp;sat[i&nbsp;*&nbsp;3&nbsp;+&nbsp;j]
			r&nbsp;|=&nbsp;(v&nbsp;&lt;&nbsp;0)&nbsp;^&nbsp;(assign[abs(v)&nbsp;-&nbsp;1]&nbsp;==&nbsp;1)
		if&nbsp;not&nbsp;r:
			print&nbsp;&#39;ERR:&nbsp;%d&nbsp;%s&#39;&nbsp;%&nbsp;(i,&nbsp;str(sat[i&nbsp;*&nbsp;3:&nbsp;(i&nbsp;+&nbsp;1)&nbsp;*&nbsp;3]))
			return&nbsp;False

	return&nbsp;True</pre>

<p style="text-indent: 2em;">
  其中SATexam产生输入，就是1-n重复三遍，随机排序，然后选取部分取相反数，不过看起来取相反数的数量不是很大，然后SATjudge用来检查结果是否正确，可以认为是要我们给出一个01串，满足某个性质即可。
</p>

<p style="text-indent: 2em;">
  简单点说就是，把3n个数，3个一组分成n组后，要求每组的数至少有一个满足：这个数是负数 xor 这个数对应的01串中那一位为1。反正感觉条件很蛋疼，不太好描述就是了……
</p>

<p style="text-indent: 2em;">
  很显然，如果输入没有负数，我们直接给出一个全1串，显然是满足条件的，于是我们以这个为初始状态，将其中一些1改成0就好，不过时间复杂度简直没法算，并且想来题目对输入中负数个数的限制应该也是对这题这样调整的效率很有影响的。不过按照之前的经验看，这种题暴力应该是都可以算出来的，于是果断对输入从前往后扫，如果发现，某一位不满足了，就尝试调整结果的01串就好，然后还准备了各种调整时的优化策略，虽然到最后都还没用上就轻松过了。
</p>

<p style="text-indent: 2em;">
  比赛时的程序如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;string
import&nbsp;hashlib
import&nbsp;itertools
import&nbsp;zlib
import&nbsp;struct

import&nbsp;zio

ROUNDS&nbsp;=&nbsp;50
#&nbsp;n&nbsp;=&nbsp;0
#&nbsp;cnt&nbsp;=&nbsp;[]
#&nbsp;data&nbsp;=&nbsp;[]
#&nbsp;adj_left&nbsp;=&nbsp;&#39;&#39;
#&nbsp;ans&nbsp;=&nbsp;[]
#&nbsp;adj_right&nbsp;=&nbsp;&#39;&#39;

def&nbsp;link(i,&nbsp;j,&nbsp;sign):
&nbsp;&nbsp;&nbsp;&nbsp;adj_left[i].append(j)
&nbsp;&nbsp;&nbsp;&nbsp;adj_right[j].append((i,&nbsp;sign))
&nbsp;&nbsp;&nbsp;&nbsp;cnt[i]&nbsp;+=&nbsp;sign&nbsp;^&nbsp;ans[j]

def&nbsp;update_right(right):
&nbsp;&nbsp;&nbsp;&nbsp;ans[right]&nbsp;=&nbsp;not&nbsp;ans[right]
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i,&nbsp;s&nbsp;in&nbsp;adj_right[right]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;s&nbsp;^&nbsp;ans[right]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cnt[i]&nbsp;+=&nbsp;1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cnt[i]&nbsp;-=&nbsp;1

def&nbsp;adjust_right(left,&nbsp;right):
&nbsp;&nbsp;&nbsp;&nbsp;update_right(right)
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i,&nbsp;s&nbsp;in&nbsp;adj_right[right]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;left&nbsp;==&nbsp;i:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;not&nbsp;adjust_left(right,&nbsp;i):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;update_right(right)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;False
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True

def&nbsp;adjust_left(right,&nbsp;left):
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;cnt[left]&nbsp;&gt;&nbsp;0:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;adj_left[left]:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;right&nbsp;==&nbsp;i:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;adjust_right(left,&nbsp;i):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;True
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;False

def&nbsp;solve():
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;n
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;cnt
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;data
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;adj_left
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;ans
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;adj_right

&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;=&nbsp;len(data)&nbsp;/&nbsp;3
&nbsp;&nbsp;&nbsp;&nbsp;cnt&nbsp;=&nbsp;[0]&nbsp;*&nbsp;n
&nbsp;&nbsp;&nbsp;&nbsp;adj_left&nbsp;=&nbsp;[[]&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(n)]
&nbsp;&nbsp;&nbsp;&nbsp;ans&nbsp;=&nbsp;[True]&nbsp;*&nbsp;n
&nbsp;&nbsp;&nbsp;&nbsp;adj_right&nbsp;=&nbsp;[[]&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(n)]

&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(n&nbsp;*&nbsp;3):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;link(i&nbsp;/&nbsp;3,&nbsp;abs(data[i])&nbsp;-&nbsp;1,&nbsp;data[i]&nbsp;&lt;&nbsp;0)

&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(n):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;not&nbsp;adjust_left(-1,&nbsp;i):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;Error!!!&#39;

&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(0,&nbsp;n,&nbsp;8):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s&nbsp;=&nbsp;ans[i:i&nbsp;+&nbsp;8][::-1]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s&nbsp;=&nbsp;&#39;&#39;.join([str(int(j))&nbsp;for&nbsp;j&nbsp;in&nbsp;s])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;+=&nbsp;chr(int(s,&nbsp;2))
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;zlib.compress(result)
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;len(result),&nbsp;io.write(struct.pack(&#39;&gt;I&#39;,&nbsp;len(result))&nbsp;+&nbsp;result)

def&nbsp;challenge():
&nbsp;&nbsp;&nbsp;&nbsp;start&nbsp;=&nbsp;io.readline().strip(&#39;n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;start
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;all_chars&nbsp;=&nbsp;[chr(i)&nbsp;for&nbsp;i&nbsp;in&nbsp;xrange(256)]
&nbsp;&nbsp;&nbsp;&nbsp;for&nbsp;pad&nbsp;in&nbsp;itertools.combinations(string.printable,&nbsp;20&nbsp;-&nbsp;len(start)):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;hashlib.sha1(start&nbsp;+&nbsp;&#39;&#39;.join(pad)).digest().endswith(&#39;xFFxFFxFF&#39;):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;io.write(start&nbsp;+&nbsp;&#39;&#39;.join(pad))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;zio.HEX(start&nbsp;+&nbsp;&#39;&#39;.join(pad))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break


TARGET&nbsp;=&nbsp;(&#39;202.112.26.111&#39;,&nbsp;23333)
#&nbsp;TARGET&nbsp;=&nbsp;(&#39;127.0.0.1&#39;,&nbsp;8888)

io&nbsp;=&nbsp;zio.zio(TARGET,&nbsp;print_read=False,&nbsp;print_write=False,&nbsp;timeout=10000000)
challenge()
for&nbsp;i&nbsp;in&nbsp;xrange(1,&nbsp;ROUNDS&nbsp;+&nbsp;1):
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;&#39;=&#39;&nbsp;*&nbsp;10&nbsp;+&nbsp;&#39;Level&nbsp;&#39;&nbsp;+&nbsp;str(i)&nbsp;+&nbsp;&#39;=&#39;&nbsp;*&nbsp;10
&nbsp;&nbsp;&nbsp;&nbsp;io.read_until(&#39;Enjoy&nbsp;your&nbsp;SAT:n&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;data_len&nbsp;=&nbsp;struct.unpack(&#39;&gt;I&#39;,&nbsp;io.read(4))[0]
&nbsp;&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;zlib.decompress(io.read(data_len)).split(&#39;,&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;[int(i)&nbsp;for&nbsp;i&nbsp;in&nbsp;data]
&nbsp;&nbsp;&nbsp;&nbsp;io.read_until_timeout()
&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;len(data)
&nbsp;&nbsp;&nbsp;&nbsp;solve()
print&nbsp;&#39;Win!&nbsp;Get&nbsp;flag:&#39;
#&nbsp;s&nbsp;=&nbsp;&#39;&#39;
#&nbsp;while&nbsp;True:
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;s&nbsp;+=&nbsp;io.read(1)
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;print&nbsp;s
io.read_until(&#39;scholarship:n&#39;)
print&nbsp;io.read_until_timeout(60)</pre>

<p style="text-indent: 2em;">
  当时先是本地跑的，很轻松的就过了，结果跑线上的时候真是跑到哭出来，要传的数据太大，然后传输起来各种超时，最后把超时加到N大之后终于可以跑过了，可是在最后一组算完之后收flag却总是收不到，然后以各种姿势一次次的试，最后终于有一次一个字符一个字符的收的时候收到了，真是哭出来啊。本来还担心过计算太慢，结果真是我想太多了，计算比这网络传输快了不知道多少个数量级了都……
</p>
