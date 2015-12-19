---
title: Mac OS X python Tab补全
author: Ripples
layout: post
permalink: /2015/06/23/mac-os-x-python-tab%e8%a1%a5%e5%85%a8/
views:
  - 343
categories:
  - technology
tags:
  - python
  - 自动补全
---
<p style="text-indent: 2em;">
  今天偶然把困扰好久的Python命令行给解决了，简直不能更爽，于是特此记录一下。
</p>

<!--more-->

<p style="text-indent: 2em;">
  很早之前就曾经尝试过给Python命令行加上Tab自动补全，可是网上的办法试了半天也没有作用，一直没搞清楚是什么情况，以为是Mac太奇葩，于是就一直搁置，今天心血来潮，又搜了一下，搜到了<a href="https://stackoverflow.com/questions/7116038/python-tab-completion-mac-osx-10-7-lion/7116997#7116997?newreg=864b9e5d394e46e490874b562efb9fc4" target="_blank">这个</a>，说是升到Lion之后才跪的，好吧，我买电脑的时候就已经是Lion了吧，真悲催……
</p>

<p style="text-indent: 2em;">
  于是乎，赶快按照里面给的方案加上了几句话，果断就生效了，终于不用再羡慕用node命令行时补全的爽了~~~不得不说stack overflow大法好，果然都是我没有用<span style="text-indent: 32px;">stack overflow的习惯惹的祸<img src="http://img.baidu.com/hi/jx2/j_0012.gif" />。</span>
</p>

<p style="text-indent: 2em;">
  然后这次还顺手把历史记录一起给加上了，这样新开python的时候可以上箭头找到上一次运行时的命令了，最后的<code>.pystartup</code>文件如下：
</p>

<pre class="brush:python;toolbar:false">#&nbsp;Add&nbsp;auto-completion&nbsp;and&nbsp;a&nbsp;stored&nbsp;history&nbsp;file&nbsp;of&nbsp;commands&nbsp;to&nbsp;your&nbsp;Python
#&nbsp;interactive&nbsp;interpreter.&nbsp;Requires&nbsp;Python&nbsp;2.0+,&nbsp;readline.&nbsp;Autocomplete&nbsp;is
#&nbsp;bound&nbsp;to&nbsp;the&nbsp;Esc&nbsp;key&nbsp;by&nbsp;default&nbsp;(you&nbsp;can&nbsp;change&nbsp;it&nbsp;-&nbsp;see&nbsp;readline&nbsp;docs).
#
#&nbsp;Store&nbsp;the&nbsp;file&nbsp;in&nbsp;~/.pystartup,&nbsp;and&nbsp;set&nbsp;an&nbsp;environment&nbsp;variable&nbsp;to&nbsp;point
#&nbsp;to&nbsp;it:&nbsp;&nbsp;"export&nbsp;PYTHONSTARTUP=~/.pystartup"&nbsp;in&nbsp;bash.

import&nbsp;atexit
import&nbsp;os
import&nbsp;readline
import&nbsp;rlcompleter

#&nbsp;for&nbsp;Mac&nbsp;OS&nbsp;X
if&nbsp;&#39;libedit&#39;&nbsp;in&nbsp;readline.__doc__:
&nbsp;&nbsp;&nbsp;&nbsp;readline.parse_and_bind("bind&nbsp;^I&nbsp;rl_complete")
else:
&nbsp;&nbsp;&nbsp;&nbsp;readline.parse_and_bind("tab:&nbsp;complete")

historyPath&nbsp;=&nbsp;os.path.expanduser("~/.pyhistory")

def&nbsp;save_history(historyPath=historyPath):
&nbsp;&nbsp;&nbsp;&nbsp;import&nbsp;readline
&nbsp;&nbsp;&nbsp;&nbsp;readline.write_history_file(historyPath)

if&nbsp;os.path.exists(historyPath):
&nbsp;&nbsp;&nbsp;&nbsp;readline.read_history_file(historyPath)

atexit.register(save_history)
del&nbsp;os,&nbsp;atexit,&nbsp;readline,&nbsp;rlcompleter,&nbsp;save_history,&nbsp;historyPath</pre>

<p style="text-indent: 2em;">
  当然，不要忘了在shell启动脚本（我是<code>.zshrc</code>）中加上：
</p>

<pre class="brush:bash;toolbar:false">export&nbsp;PYTHONSTARTUP=~/.pystartup</pre>
