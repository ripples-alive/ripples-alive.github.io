---
title: SSCTF Crack类 第一题
author: Ripples
layout: post
permalink: /ssctf-crack%e7%b1%bb-%e7%ac%ac%e4%b8%80%e9%a2%98/
views:
  - 361
categories:
  - CTF
tags:
  - CTF
  - reversing
  - SSCTF
  - WriteUp
---
<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">Crack类的题目也没神马题目描述了，</span>传送门<a href="http://ctf.sobug.com/crackme/b4dc971ef90cb6ae/index.php" target="_blank">在此</a>。
</p>

<p style="text-indent: 2em;">
  今天发现IDA原来还有反编译功能，顿时觉得自己之前似乎SB了。然后试用了下，感觉效果还不错。
</p>

<p style="text-indent: 2em;">
  这题整个程序很简单，就是读入name和pass，然后通过下面的子程序（IDA生成的）来judge：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">bool&nbsp;__cdecl&nbsp;sub_401000(const&nbsp;char&nbsp;*username,&nbsp;const&nbsp;char&nbsp;*password)
{
&nbsp;&nbsp;const&nbsp;char&nbsp;*v2;&nbsp;//&nbsp;esi@1
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;len_user_plus;&nbsp;//&nbsp;kr04_4@1
&nbsp;&nbsp;int&nbsp;len_user;&nbsp;//&nbsp;ebp@1
&nbsp;&nbsp;bool&nbsp;result;&nbsp;//&nbsp;al@2
&nbsp;&nbsp;int&nbsp;i;&nbsp;//&nbsp;eax@3
&nbsp;&nbsp;int&nbsp;*str_con;&nbsp;//&nbsp;edi@4
&nbsp;&nbsp;const&nbsp;char&nbsp;v8;&nbsp;//&nbsp;bl@5
&nbsp;&nbsp;const&nbsp;char&nbsp;*v9;&nbsp;//&nbsp;edi@6
&nbsp;&nbsp;int&nbsp;j;&nbsp;//&nbsp;ecx@6
&nbsp;&nbsp;bool&nbsp;v11;&nbsp;//&nbsp;zf@6

&nbsp;&nbsp;v2&nbsp;=&nbsp;username;
&nbsp;&nbsp;len_user_plus&nbsp;=&nbsp;strlen(username)&nbsp;+&nbsp;1;
&nbsp;&nbsp;len_user&nbsp;=&nbsp;len_user_plus&nbsp;-&nbsp;1;
&nbsp;&nbsp;if&nbsp;(&nbsp;len_user_plus&nbsp;-&nbsp;1&nbsp;==&nbsp;strlen(password)&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;len_user&nbsp;&gt;&nbsp;0&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;str_con&nbsp;=&nbsp;&global_str;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;do
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v8&nbsp;=&nbsp;*(_BYTE&nbsp;*)str_con&nbsp;^&nbsp;password[i];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;++str_con;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;password[i++]&nbsp;=&nbsp;v8;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;i&nbsp;&lt;&nbsp;len_user&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;v9&nbsp;=&nbsp;password;
&nbsp;&nbsp;&nbsp;&nbsp;j&nbsp;=&nbsp;len_user_plus&nbsp;-&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;v11&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;do
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;!j&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v11&nbsp;=&nbsp;*v2++&nbsp;==&nbsp;*v9++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--j;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(&nbsp;v11&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;v11;
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0;
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;result;
}</pre>

<p style="text-indent: 2em;">
  程序很明确，就是将username与一个全局的常量进行xor，然后再将结果与password对比，相等则正确。
</p>

<pre class="brush:plain;toolbar:false">00408030&nbsp;&nbsp;01&nbsp;00&nbsp;00&nbsp;00&nbsp;02&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;04&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408040&nbsp;&nbsp;01&nbsp;00&nbsp;00&nbsp;00&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;01&nbsp;00&nbsp;00&nbsp;00&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408050&nbsp;&nbsp;01&nbsp;00&nbsp;00&nbsp;00&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;06&nbsp;00&nbsp;00&nbsp;00&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408060&nbsp;&nbsp;04&nbsp;00&nbsp;00&nbsp;00&nbsp;08&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408070&nbsp;&nbsp;01&nbsp;00&nbsp;00&nbsp;00&nbsp;02&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;04&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408080&nbsp;&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;07&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
00408090&nbsp;&nbsp;02&nbsp;00&nbsp;00&nbsp;00&nbsp;03&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;02&nbsp;00&nbsp;00&nbsp;00&nbsp;04&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
004080A0&nbsp;&nbsp;08&nbsp;00&nbsp;00&nbsp;00&nbsp;02&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;05&nbsp;00&nbsp;00&nbsp;00&nbsp;06&nbsp;00&nbsp;00&nbsp;00&nbsp;&nbsp;................
004080B0&nbsp;&nbsp;04&nbsp;00&nbsp;00&nbsp;00&nbsp;74&nbsp;72&nbsp;79&nbsp;20&nbsp;&nbsp;61&nbsp;67&nbsp;61&nbsp;69&nbsp;6E&nbsp;21&nbsp;00&nbsp;00&nbsp;&nbsp;....try&nbsp;again!..</pre>

<p style="text-indent: 2em;">
  注意这个字符串是每个dword只去某位的一个byte，所以相当于是01、02、03、04、01、05、01、05……
</p>

<p style="text-indent: 2em;">
  我们可以将username和password分别输入为0、1来测试，会发现程序提示我们正确。
</p>

<p style="text-indent: 2em;">
  于是，按照要求，使用用户名当username，算出password，计算MD5就是答案了。
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/11/CrackMe1.rar">CrackMe1.rar</a>
</p>
