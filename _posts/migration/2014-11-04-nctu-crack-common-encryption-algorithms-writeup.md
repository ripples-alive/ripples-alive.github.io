---
title: NCTU Crack Common encryption algorithms 解题报告
author: Ripples
layout: post
permalink: /nctu-crack-common-encryption-algorithms-%e8%a7%a3%e9%a2%98%e6%8a%a5%e5%91%8a/
views:
  - 338
categories:
  - CTF
tags:
  - NCTU
  - reversing
  - WriteUp
---
<p style="text-indent: 2em;">
  助教给我们找了这么个网站，据说题目很水……
</p>

<p style="text-indent: 2em;">
  第一题Simple "crackMe"&#8230;short and easy实在是太水，于是直接上第二题了，传送门<a href="http://wargame.cs.nctu.edu.tw/wargame/problem/2/29" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  题目描述：
</p>

<!--more-->

<pre class="brush:plain;toolbar:false">&#39;lovetc&#39;&nbsp;forgot&nbsp;his&nbsp;own&nbsp;password,

maybe&nbsp;you&nbsp;can&nbsp;reverse&nbsp;it&nbsp;and&nbsp;find&nbsp;the&nbsp;encryption&nbsp;algorithm....

press&nbsp;"start"&nbsp;then&nbsp;download&nbsp;it.

Hint:&nbsp;The&nbsp;checkin&nbsp;key&nbsp;is&nbsp;his&nbsp;password....</pre>

<p style="text-indent: 2em;">
  这是个图形化程序，首先IDA走起，然后找到消息分发函数：
</p>

<pre class="brush:cpp;toolbar:false">int&nbsp;__stdcall&nbsp;DialogFunc(HWND&nbsp;hDlg,&nbsp;int&nbsp;a2,&nbsp;int&nbsp;a3,&nbsp;int&nbsp;a4)
{
&nbsp;&nbsp;HICON&nbsp;v4;&nbsp;//&nbsp;eax@2
&nbsp;&nbsp;UINT&nbsp;v5;&nbsp;//&nbsp;eax@9
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v7;&nbsp;//&nbsp;[sp-4h]&nbsp;[bp-8h]@11

&nbsp;&nbsp;switch&nbsp;(&nbsp;a2&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;272:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v4&nbsp;=&nbsp;LoadIconA(hInstance,&nbsp;(LPCSTR)0x1F4);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SendMessageA(hDlg,&nbsp;0x80u,&nbsp;0,&nbsp;(LPARAM)v4);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;16:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EndDialog(hDlg,&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;273:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;switch&nbsp;(&nbsp;a3&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;300:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MessageBoxA(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hDlg,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+=================================+&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;Keygen-me&nbsp;N&nbsp;Created&nbsp;on&nbsp;27/08/2003&nbsp;&nbsp;|n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;+=================================+&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nTry&nbsp;to&nbsp;keygen&nbsp;this&nbsp;program,&nbsp;and&nbsp;send&nbsp;your&nbsp;solution&nbsp;tonwww.crackmes.de,&nbsp;for&nbsp;more&nbsp;informations&nbsp;contact&nbsp;me&nbsp;at&nbsp;n#eminence&nbsp;channel&nbsp;on&nbsp;eFnet.n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enjoy&nbsp;Crypto.....n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(C)2003&nbsp;BytePtr&nbsp;[e!]&nbsp;&nbsp;n",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"AbOut",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;900:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v5&nbsp;=&nbsp;GetDlgItemTextA(hDlg,&nbsp;100,&nbsp;String1,&nbsp;300);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;!v5&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;MessageBoxA(0,&nbsp;"Your&nbsp;name&nbsp;please&nbsp;!!!",&nbsp;"oooH&nbsp;input&nbsp;Error",&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v7&nbsp;=&nbsp;v5;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;!GetDlgItemTextA(hDlg,&nbsp;200,&nbsp;String,&nbsp;300)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;MessageBoxA(0,&nbsp;"Where&nbsp;is&nbsp;Da&nbsp;serial&nbsp;DuDe&nbsp;?",&nbsp;"oooH&nbsp;input&nbsp;Error",&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lstrcatA(String1,&nbsp;"BytePtr&nbsp;[e!]");
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sub_401000((int)String1,&nbsp;v7,&nbsp;(int)&unk_4056A8);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sub_401B79();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;lstrcmpA(String,&nbsp;byte_4079D0)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MessageBoxA(0,&nbsp;"hmmm&nbsp;not&nbsp;like&nbsp;this&nbsp;&nbsp;DuDe&nbsp;Try&nbsp;again....",&nbsp;"Fatal&nbsp;Error",&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MessageBoxA(0,&nbsp;"Good&nbsp;serial",&nbsp;"Good&nbsp;Work",&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;case&nbsp;400:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EndDialog(hDlg,&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;0;
}</pre>

<p style="text-indent: 2em;">
  很显然，关键应该就在sub_401000了，可是，当我们打开这个函数一看的时候，简直是长的不忍直视，虽说是好像没再调用些别的神马函数，耐着性子一步步来肯定可以搞出答案，但肯定不是个明智的选择。
</p>

<p style="text-indent: 2em;">
  然后再仔细观察程序会发现，<span style="text-indent: 32px;">sub_401000只对输入框中输入的Name做了处理，最后是直接拿处理得到的字符串和byte_4079D0，也就是输入的Serial做对比，那么显然我们可以debug让这个程序跑起来，输入正确的Name（</span><span class="s1">lovetc</span>），让程序停在lstrcmpA前，然后我们便可以很轻松的得到Serial。
</p>

<p style="text-indent: 2em;">
  于是，此题得以解决~~~
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/keygenme.rar">keygenme.rar</a>
</p>
