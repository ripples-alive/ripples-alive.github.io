---
title: 中软吉大•问鼎杯决赛样题密码类（二）
author: Ripples
layout: post
permalink: /%e4%b8%ad%e8%bd%af%e5%90%89%e5%a4%a7%e2%80%a2%e9%97%ae%e9%bc%8e%e6%9d%af%e5%86%b3%e8%b5%9b%e6%a0%b7%e9%a2%98%e5%af%86%e7%a0%81%e7%b1%bb/
views:
  - 631
categories:
  - CTF
tags:
  - decryption
  - WriteUp
  - 中软吉大•问鼎杯
  - 频率分析
---
<p style="text-indent: 2em;">
  密码类样题一共两道题，这是第二题，题目如下：
</p>

<pre class="brush:plain;toolbar:false">2、截获到一段加密信息：
PCQ&nbsp;VMJYPD&nbsp;LBYK&nbsp;LYSO&nbsp;KBXBJXWXV&nbsp;BXV&nbsp;ZCJPO&nbsp;EYPD&nbsp;KBXBJYUXJ&nbsp;LBJOO&nbsp;KCPK.&nbsp;CP&nbsp;LBO&nbsp;LBCMKXPV&nbsp;XPV&nbsp;IYJKL&nbsp;PYDBL,QBOP&nbsp;KBO&nbsp;BXV&nbsp;OPVOV&nbsp;LBO&nbsp;LXRO,&nbsp;KBO&nbsp;JCKO&nbsp;XPV&nbsp;EYKKOV&nbsp;LBO&nbsp;DJCMPV&nbsp;ZOICJO&nbsp;BYS,&nbsp;KXUYPD：“DJOXL&nbsp;EYPD,&nbsp;ICJ&nbsp;XLBCMKXPV&nbsp;XPV&nbsp;CPO&nbsp;PYDBLK&nbsp;Y&nbsp;BXNO&nbsp;ZOOP&nbsp;JOACMPLYPD&nbsp;LC&nbsp;UCM&nbsp;LBO&nbsp;IXZROK&nbsp;CI&nbsp;FXKL&nbsp;XDOK&nbsp;XPV&nbsp;LBO&nbsp;RODOPVK&nbsp;CI&nbsp;XPAYOPL&nbsp;EYPDK.&nbsp;SXU&nbsp;Y&nbsp;SXEO&nbsp;KC&nbsp;ZCRV&nbsp;XK&nbsp;LC&nbsp;AJXNO&nbsp;X&nbsp;IXNCMJ&nbsp;CI&nbsp;UCMJ&nbsp;SXGOKLU?”&nbsp;请帮助解密。（400分）</pre>

<p style="text-indent: 2em;">
  这题一看连空格都没有去掉，就觉得应该不难了。
</p>

<p style="text-indent: 2em;">
  首先还是老规矩，频率统计：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">o&nbsp;:&nbsp;33
x&nbsp;:&nbsp;29
p&nbsp;:&nbsp;27
k&nbsp;:&nbsp;23
c&nbsp;:&nbsp;22
b&nbsp;:&nbsp;22
l&nbsp;:&nbsp;21
y&nbsp;:&nbsp;17
j&nbsp;:&nbsp;16
v&nbsp;:&nbsp;16
d&nbsp;:&nbsp;12

pv&nbsp;=&gt;&nbsp;9
lb&nbsp;=&gt;&nbsp;9
bo&nbsp;=&gt;&nbsp;8
cm&nbsp;=&gt;&nbsp;7
xp&nbsp;=&gt;&nbsp;7

ypd&nbsp;=&gt;&nbsp;6
xpv&nbsp;=&gt;&nbsp;6
lbo&nbsp;=&gt;&nbsp;5</pre>

<p style="text-indent: 2em;">
  首先猜测lbo为the，然后替换后发现有thJee，于是猜测J对应r。
</p>

<p style="text-indent: 2em;">
  紧接着，由于ypd和xpv第二个字母相同，于是两者应该是ing和and，首先尝试xpv为and、ypd为ing。
</p>

<p style="text-indent: 2em;">
  替换后发现有thiK、nightK，猜测K对应s；
</p>

<p style="text-indent: 2em;">
  发现有Iirst，猜测I对应f；
</p>

<p style="text-indent: 2em;">
  发现有anAient，猜测A对应c；
</p>

<p style="text-indent: 2em;">
  发现有dMring，猜测M对于u；
</p>

<p style="text-indent: 2em;">
  发现有thCusand，猜测C对应o；
</p>

<p style="text-indent: 2em;">
  发现有haNe Zeen，猜测N对应v，Z对应b；
</p>

<p style="text-indent: 2em;">
  发现由noQ during this tiSe开头，猜测Q对应w，S对应m；
</p>

<p style="text-indent: 2em;">
  发现有saUing：，猜测U对于y；
</p>

<p style="text-indent: 2em;">
  然后就只剩jklpqxz了。
</p>

<p style="text-indent: 2em;">
  发现有Regend，猜测R对应l；
</p>

<p style="text-indent: 2em;">
  发现有maEe，猜测E对应k；
</p>

<p style="text-indent: 2em;">
  发现有maGesty，猜测G对应j；
</p>

<p style="text-indent: 2em;">
  发现有Fast，猜测F对应p；
</p>

<p style="text-indent: 2em;">
  然后只剩shahraWad中W未知了，应该为z。
</p>

<p style="text-indent: 2em;">
  其实在中途就可以通过google搜到原文了：
</p>

<pre class="brush:plain;toolbar:false">now&nbsp;during&nbsp;this&nbsp;time&nbsp;shahrazad&nbsp;had&nbsp;borne&nbsp;king&nbsp;shahriyar&nbsp;three&nbsp;sons.&nbsp;on&nbsp;the&nbsp;thousand&nbsp;and&nbsp;first&nbsp;night,&nbsp;when&nbsp;she&nbsp;had&nbsp;ended&nbsp;the&nbsp;tale&nbsp;of&nbsp;ma’aruf,&nbsp;she&nbsp;rose&nbsp;and&nbsp;kissed&nbsp;the&nbsp;ground&nbsp;before&nbsp;him,&nbsp;saying:&nbsp;“great&nbsp;king,&nbsp;for&nbsp;a&nbsp;thousand&nbsp;and&nbsp;one&nbsp;nights&nbsp;i&nbsp;have&nbsp;been&nbsp;recounting&nbsp;to&nbsp;you&nbsp;the&nbsp;fables&nbsp;of&nbsp;past&nbsp;ages&nbsp;and&nbsp;the&nbsp;legends&nbsp;of&nbsp;ancient&nbsp;kings.&nbsp;may&nbsp;i&nbsp;make&nbsp;so&nbsp;bold&nbsp;as&nbsp;to&nbsp;crave&nbsp;a&nbsp;favour&nbsp;of&nbsp;your&nbsp;majesty?”</pre>

<p style="text-indent: 2em;">
  搜索也发现在<a href="http://cs.brynmawr.edu/Courses/ESEM/IntroductionToSecretCodes.pdf" target="_blank">这里</a>的例子中就有我们这道题。
</p>

<p style="text-indent: 2em;">
  样题果然很水，也不知道比赛的时候题目难度能不能也这么水～～～
</p>
