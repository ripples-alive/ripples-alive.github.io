---
title: NCTU BCG crackme 解题报告
author: Ripples
layout: post
permalink: /nctu-bcg-crackme-解题报告/
views:
  - 387
categories:
  - CTF
tags:
  - NCTU
  - reversing
  - WriteUp
  - 脱壳
---
<p style="text-indent: 2em;">
  今天又来试着做NCTU，于是乎挑了这道题，题目描述如下：
</p>

<pre class="brush:plain;toolbar:false">BCG&#39;s&nbsp;full&nbsp;name&nbsp;is&nbsp;BeGiNnEr&#39;S&nbsp;CrAcKiNg&nbsp;Group....

Do&nbsp;you&nbsp;have&nbsp;the&nbsp;ability&nbsp;to&nbsp;crack&nbsp;this&nbsp;program?....&nbsp;

press&nbsp;"start"&nbsp;then&nbsp;download&nbsp;it!

Hint1:&nbsp;The&nbsp;checkin&nbsp;key&nbsp;is&nbsp;the&nbsp;"content"...

Hint2:&nbsp;key&nbsp;length&nbsp;is&nbsp;the&nbsp;asked&nbsp;length,&nbsp;and&nbsp;all&nbsp;character&nbsp;is&nbsp;same&nbsp;as&nbsp;the&nbsp;first&nbsp;one.</pre>

<!--more-->

<p style="text-indent: 2em;">
  这道题做完之后确实的说，也是挺简单的，但由于自己还是太弱，做的时候就没有这么顺利了。
</p>

<p style="text-indent: 2em;">
  首先打开程序，会发现上面有着各种说明，似乎这题是BCG申请加入时的题，程序上写的那些限制我也没太弄明白，不过这里既然只是解题，就不管了。
</p>

<p style="text-indent: 2em;">
  IDA反汇编程序会发现，程序只有一个很短的start，感觉应该是自解压的程序，于是乎先打开程序，再将OD附加到该程序上便开始调试。然而，尝试了半天之后，还是没有什么收获，一直没能看到什么关键代码。
</p>

<p style="text-indent: 2em;">
  实在无奈之下，用PEiD查了下，发现是有个aspack壳，于是在网上找了个脱壳软件将壳脱掉，再用IDA查看，顿时感觉就是柳暗花明。
</p>

<p style="text-indent: 2em;">
  程序很明确，一眼就看到关键程序，用IDA反编译得：
</p>

<pre class="brush:cpp;toolbar:false">int&nbsp;__stdcall&nbsp;DialogFunc(HWND&nbsp;hDlg,&nbsp;int&nbsp;a2,&nbsp;int&nbsp;a3,&nbsp;int&nbsp;a4)
{
&nbsp;&nbsp;int&nbsp;result;&nbsp;//&nbsp;eax@4
&nbsp;&nbsp;int&nbsp;v5;&nbsp;//&nbsp;eax@14

&nbsp;&nbsp;if&nbsp;(&nbsp;a2&nbsp;==&nbsp;273&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;a3&nbsp;==&nbsp;1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;MessageBoxA(0,&nbsp;"中,&nbsp;"OfficialCrackme",&nbsp;0x1000u);
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;a3&nbsp;==&nbsp;2&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;EndDialog(hDlg,&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;a3&nbsp;!=&nbsp;3&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;a2&nbsp;!=&nbsp;272&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;a2&nbsp;!=&nbsp;16&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;EndDialog(hDlg,&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;SetDlgItemTextA(hDlg,&nbsp;10,&nbsp;String);
&nbsp;&nbsp;}
&nbsp;&nbsp;hFile&nbsp;=&nbsp;CreateFileA("[BCG].Key",&nbsp;0xC0000000u,&nbsp;3u,&nbsp;0,&nbsp;3u,&nbsp;(DWORD)"€",&nbsp;0);
&nbsp;&nbsp;if&nbsp;(&nbsp;hFile&nbsp;==&nbsp;(HANDLE)-1&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;goto&nbsp;LABEL_22;
&nbsp;&nbsp;if&nbsp;(&nbsp;!ReadFile(hFile,&nbsp;(LPVOID)&String2,&nbsp;0xAu,&nbsp;&NumberOfBytesRead,&nbsp;0)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;goto&nbsp;LABEL_22;
&nbsp;&nbsp;if&nbsp;(&nbsp;!ReadFile(hFile,&nbsp;String1,&nbsp;0xAu,&nbsp;&NumberOfBytesRead,&nbsp;0)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;goto&nbsp;LABEL_22;
&nbsp;&nbsp;CloseHandle(hFile);
&nbsp;&nbsp;v5&nbsp;=&nbsp;0;
&nbsp;&nbsp;do
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;*(&String2&nbsp;+&nbsp;v5)&nbsp;^=&nbsp;0x58u;
&nbsp;&nbsp;&nbsp;&nbsp;++v5;
&nbsp;&nbsp;}
&nbsp;&nbsp;while&nbsp;(&nbsp;*(&String2&nbsp;+&nbsp;v5)&nbsp;);
&nbsp;&nbsp;if&nbsp;(&nbsp;!lstrcmpA(String1,&nbsp;&String2)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;MessageBoxA(0,&nbsp;"注册验证成功,恭喜,&nbsp;"OfficialCrackme",&nbsp;0x1000u);
&nbsp;&nbsp;else
LABEL_22:
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;MessageBoxA(0,&nbsp;"革,&nbsp;"OfficialCrackme",&nbsp;0x1000u);
&nbsp;&nbsp;return&nbsp;result;
}</pre>

<p style="text-indent: 2em;">
  程序流程很清楚，是从[BCG].Key中读出两个长度为0xA的字符串，然后其中一个字符串与0x<a></a>58异或之后的结果正好要为另一个字符串，那我们可以取'0' xor 'h'，于是在文件中写入hhhhhhhhhh0000000000，再尝试运行程序。可是，结果却与我们的预期大相径庭，还是告诉我们“革命尚未成功”。
</p>

<p style="text-indent: 2em;">
  下断点再调试程序发现，String2和String1在内存空间上正好相距10，也就是说String2的后面没有字符串结尾标志 ，这样导致异或的时候String2和String1都被异或的，这还不止，在lstrcmpA的时候，更是认为String1是String2的字串了，那么看来，这两个字符串肯定不会相等了。
</p>

<p style="text-indent: 2em;">
  为了解决这个问题，第一想法就是在文件中写入 ，然String2并不能真正读入10个字符或者说让第10个字符为 ，但这样且不说是否可行，但至少是相当麻烦的，那再仔细想想会发现，如果我们输入的String2中有字符X（即0x<a></a>58），那么异或的时候就会产生 ，这样在lstrcmpA的时候，两个字符串就可以相等了。
</p>

<p style="text-indent: 2em;">
  那么，我们直接在文件中写入20个X，运行程序便成功弹出正确的提示框，得以解决。
</p>

<p style="text-indent: 2em;">
  PS：chekckin的时候需要注意，是提交10个X即可，不要提交20个，这问题坑了我半天……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/bcgcrackme.rar">bcgcrackme.rar</a>
</p>
