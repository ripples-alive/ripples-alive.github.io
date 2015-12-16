---
title: WordPress 上文章添加转载申明
author: Ripples
layout: post
views:
  - 332
categories:
  - technology
tags:
  - wordpress
  - 转载声明
---
<p style="text-indent: 2em;">
  在网上看别人文章的时候，发现转载声明，于是思前想后，觉得自己也可以做一个～～～
</p>

<p style="text-indent: 2em;">
  我是使用的wordpress，本来以为应该很简单的，结果最开始竟然没搞定，所以决定记录一下。
</p>

<p style="text-indent: 2em;">
  首先最直接的想法是修改single.php模版，给页面直接给加上信息，但是这样并没有给RSS中文章加上转载声明。
</p>

<p style="text-indent: 2em;">
  于是乎，决定改成给the_content过滤器增加一个钩子，直接在文章最后增加信息，结果，由于我的摘要是用the_content('Read More →')，直接导致在文章列表页面每篇文章的后面也都被加上了转载声明。
</p>

<p style="text-indent: 2em;">
  然后又改成在模版里增加add_action，但是这样还是有和最开始一样的问题，RSS中没有增加声明。
</p>

<!--more-->

<p style="text-indent: 2em;">
  接着研究了半天的the_content（在wp-includes/post-template.php中），发现the_content过滤器里面的的参数确实是只有一个$content，但是，却发现声明了全局变量，其中有一个叫$more，通过这个变量的值，可以判断出来调用the_content的时候，到底是要输出全文还是摘要。于是乎，最终将以下程序加到主题的functions.php中即可：
</p>

<pre class="brush:php;toolbar:false">function&nbsp;add_copyright($content)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;$more;
&nbsp;&nbsp;&nbsp;&nbsp;$title&nbsp;=&nbsp;get_the_title();
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;($more&nbsp;&&&nbsp;(is_single()&nbsp;||&nbsp;is_feed())&nbsp;&&&nbsp;(mb_strrchr($title,&nbsp;&#39;（&#39;,&nbsp;&#39;utf-8&#39;)&nbsp;.&nbsp;&#39;（转载）&#39;&nbsp;!==&nbsp;$title))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$content&nbsp;.=&nbsp;&#39;&lt;br/&gt;&lt;p&nbsp;style="text-indent:&nbsp;2em;"&gt;原文连接：&lt;a&nbsp;href="&#39;&nbsp;.&nbsp;get_permalink()&nbsp;.&nbsp;&#39;"&gt;&#39;&nbsp;.&nbsp;$title&nbsp;.&nbsp;&#39;&lt;/a&gt;，转载请注明出处。&lt;/p&gt;&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$content;
}
add_filter(&#39;the_content&#39;,&nbsp;&#39;add_copyright&#39;);</pre>

<p style="text-indent: 2em;">
  其中mb_strrchr处的判断，是为了防止给转载的别人的文章也加上了自己的转载声明（转载的文章标题要以"（转载）"结尾）。
</p>

<p style="text-indent: 2em;">
  然后用mb_strrchr的时候遇到个奇怪的问题，就是，如果最后参数不加'utf-8'的时候，返回的字符串是从找到的字符到原串结尾，但是加上之后，就变成从原串开头到找到的字符了。。。只能不明觉厉。。。
</p>

<p style="text-indent: 2em;">
  PS：之前忘记加上is_single() || is_feed()的判断，导致在留言版块中也出现了转载声明，现已改正～～～
</p>
