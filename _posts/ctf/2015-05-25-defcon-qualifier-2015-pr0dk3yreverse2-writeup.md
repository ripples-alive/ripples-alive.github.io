---
title: Defcon qualifier 2015 pr0dk3y(reverse2) WriteUp
author: Ripples
layout: post
views:
  - 384
categories:
  - CTF
tags:
  - CTF
  - DEFCON
  - reversing
  - WriteUp
---
<p style="text-indent: 2em;">
  作为逆向题，题目自然木有神马描述，但看一眼就知道，这题是要算个序列号，但是比较蛋疼的是，这题的大多数对单个参数的限制条件都是不等于什么，而做等于判断的各个条件又都是些涉及到好几个参数的检验，所以很是麻烦。
</p>

<p style="text-indent: 2em;">
  程序首先读入一个序列号的字符串，然后会转换为5个dword，一个word来进行一些check，然后check的大致流程如下：
</p>

<!--more-->

<pre class="brush:cpp;toolbar:false">__int64&nbsp;__fastcall&nbsp;check_key(char&nbsp;*a1)
{
&nbsp;&nbsp;__int64&nbsp;result;&nbsp;//&nbsp;rax@3
&nbsp;&nbsp;__int16&nbsp;v2;&nbsp;//&nbsp;[sp+1Ah]&nbsp;[bp-16h]@1
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v3;&nbsp;//&nbsp;[sp+1Ch]&nbsp;[bp-14h]@1
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v4;&nbsp;//&nbsp;[sp+20h]&nbsp;[bp-10h]@1
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v5;&nbsp;//&nbsp;[sp+24h]&nbsp;[bp-Ch]@1
&nbsp;&nbsp;int&nbsp;v6;&nbsp;//&nbsp;[sp+28h]&nbsp;[bp-8h]@1
&nbsp;&nbsp;unsigned&nbsp;int&nbsp;v7;&nbsp;//&nbsp;[sp+2Ch]&nbsp;[bp-4h]@1

&nbsp;&nbsp;v3&nbsp;=&nbsp;*(_DWORD&nbsp;*)a1;
&nbsp;&nbsp;v4&nbsp;=&nbsp;*((_DWORD&nbsp;*)a1&nbsp;+&nbsp;1);
&nbsp;&nbsp;v5&nbsp;=&nbsp;*((_DWORD&nbsp;*)a1&nbsp;+&nbsp;2);
&nbsp;&nbsp;v6&nbsp;=&nbsp;*((_DWORD&nbsp;*)a1&nbsp;+&nbsp;3);
&nbsp;&nbsp;v7&nbsp;=&nbsp;*((_DWORD&nbsp;*)a1&nbsp;+&nbsp;4);
&nbsp;&nbsp;v2&nbsp;=&nbsp;*((_WORD&nbsp;*)a1&nbsp;+&nbsp;10);
&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)sub_E10(v2)&nbsp;||&nbsp;(unsigned&nbsp;int)sub_DA2(v2)&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;}
&nbsp;&nbsp;else&nbsp;if&nbsp;(&nbsp;(sub_F0F(a1,&nbsp;20)&nbsp;&&nbsp;0x7FFF)&nbsp;==&nbsp;v2&nbsp;)
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)sub_E40(v3,&nbsp;v4,&nbsp;v5)&nbsp;==&nbsp;v6&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)sub_1566(v7)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)no_equal_adj_4(v3)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)no_equal_adj_8(v3)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)is_prime_larger(v3)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)some_bits_mod7_0(v4)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)some_bits_saf(v5)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(&nbsp;(unsigned&nbsp;int)odd_v4_v5_mul_v3(v4,&nbsp;v5,&nbsp;v3)&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;(unsigned&nbsp;__int16)calc_jiaoyan(v4,&nbsp;v5)&nbsp;==&nbsp;v2;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;else
&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;0LL;
&nbsp;&nbsp;}
&nbsp;&nbsp;return&nbsp;result;
}</pre>

<p style="text-indent: 2em;">
  这里的check很多，为了我们能够高效的找到key，这里我们需要选择一个合适的搜索顺序，尽可能的将强的条件放在外面判断。
</p>

<ol class=" list-paddingleft-2" style="list-style-type: decimal;">
  <li>
    <p style="text-indent: 2em;">
      首先先是some_bits_mod7_0和some_bits_saf分别是对v4、v5的偶数位的判断，我们暴力穷举会发现两者分别有9363和5042种可能；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      然后在odd_v4_v5_mul_v3这里面，对v4、v5的奇数位做了很弱的限制，但是有个v4_odd * v5_odd = v3 + 1的限制，也就是说，这里我们通过枚举v4、v5的奇数位，我们可以计算出v3，然后就可以将v3带到前面的各个对v3单独限制的地方，很好的做一个筛选；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      接下来，看calc_jiaoyan，由于v4、v5我们已经枚举过了，根据v4、v5我们可以计算出对应的v2，然后带入前面对v2的单独校验中检查；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      然后sub_1566是对v7单独的校验，这个校验可以确定v7总共有171955可能；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      此时，我们仅仅剩下了一个判断(sub_F0F(a1, 20) & 0x7FFF) == v2，这句判断实际是用v3-v7计算出一个校验和，与v2进行比较，当我们进行到这一步之前，我们可以秒出很多的结果，但是，当我们加上了这个判断之后，我跑了很久也没能跑出结果，很是无奈，不过想想也是，本来其实v4、v5枚举完后量还好，但是和v7的这近20W一乘起来，也确实恐怖；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      最后，仔细再观察一下最后那个校验和的判断，我们会发现，v2其实隐式的被限制必须要小于0x7FFF，于是我们完全可以在v4、v5算出v2后，先进行这个判断，然后再来和v7联合起来校验，这样其实我们搜到一个解的工作量直接就相当于是减少了10W倍这个级别的，尤其是当v2大于<span style="text-indent: 32px;">0x7FFF的量</span>比较大的时候。因为在这里，其实可行解的量是巨大的，最后一个校验也没有很严，我们现在一次判断就可以排除10W多组不可能的情况。
    </p>
  </li>
</ol>

<p style="text-indent: 2em;">
  总之，程序写好之后，很快便可以跑出刷屏的解，随便选个就好，然后再按照规则，用算出来的6个数，反向去计算序列号即可，这一步是队友做的，我这里就没有了……
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/05/pr0dk3y.zip">pr0dk3y.zip</a>
</p>
