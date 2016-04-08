---
title: CodeIgniter 上传文件类型限制的问题
author: Ripples
layout: post
permalink: /codeigniter上传文件类型限制的问题/
views:
  - 321
categories:
  - coding
tags:
  - CodeIgniter
  - mime
  - 上传
---
<p style="text-indent: 2em;">
  这段时间在做学校的微信公众平台，用的是CI框架，在上传文件的时候出现了一点问题。
</p>

<p style="text-indent: 2em;">
  需要上传的文件是一些音频，然后看了下CI的mimes.php，里面有的音频格式很少，比如需要上传的wma就没有。于是乎，我手动在mimes.php中添加：
</p>

<pre class="brush:php;toolbar:false">&#39;wma&#39;&nbsp;&nbsp;&nbsp;=&gt;&nbsp;&nbsp;&#39;audio/x-ms-wma&#39;,</pre>

<p style="text-indent: 2em;">
  其中的mime是根据$_FILES输出得到的，但是在网上下了个wma的铃声来上传还是告诉我类型不允许。
</p>

<p style="text-indent: 2em;">
  试了好半天都不行，网上也搜不到任何有用的信息，无奈之下，去查看CI的do_upload()的源代码，输出调试，发现，CI在做类型验证的时候，通过了下面的代码获取了mime类型：
</p>

<!--more-->

<pre class="brush:php;toolbar:false">/*&nbsp;Fileinfo&nbsp;extension&nbsp;-&nbsp;most&nbsp;reliable&nbsp;method
&nbsp;*
&nbsp;*&nbsp;Unfortunately,&nbsp;prior&nbsp;to&nbsp;PHP&nbsp;5.3&nbsp;-&nbsp;it&#39;s&nbsp;only&nbsp;available&nbsp;as&nbsp;a&nbsp;PECL&nbsp;extension&nbsp;and&nbsp;the
&nbsp;*&nbsp;more&nbsp;convenient&nbsp;FILEINFO_MIME_TYPE&nbsp;flag&nbsp;doesn&#39;t&nbsp;exist.
&nbsp;*/
if&nbsp;(function_exists(&#39;finfo_file&#39;))
{
&nbsp;&nbsp;&nbsp;&nbsp;$finfo&nbsp;=&nbsp;finfo_open(FILEINFO_MIME);
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(is_resource($finfo))&nbsp;//&nbsp;It&nbsp;is&nbsp;possible&nbsp;that&nbsp;a&nbsp;FALSE&nbsp;value&nbsp;is&nbsp;returned,&nbsp;if&nbsp;there&nbsp;is&nbsp;no&nbsp;magic&nbsp;MIME&nbsp;database&nbsp;file&nbsp;found&nbsp;on&nbsp;the&nbsp;system
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$mime&nbsp;=&nbsp;@finfo_file($finfo,&nbsp;$file[&#39;tmp_name&#39;]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;finfo_close($finfo);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/*&nbsp;According&nbsp;to&nbsp;the&nbsp;comments&nbsp;section&nbsp;of&nbsp;the&nbsp;PHP&nbsp;manual&nbsp;page,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;it&nbsp;is&nbsp;possible&nbsp;that&nbsp;this&nbsp;function&nbsp;returns&nbsp;an&nbsp;empty&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;for&nbsp;some&nbsp;files&nbsp;(e.g.&nbsp;if&nbsp;they&nbsp;don&#39;t&nbsp;exist&nbsp;in&nbsp;the&nbsp;magic&nbsp;MIME&nbsp;database)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(is_string($mime)&nbsp;&&&nbsp;preg_match($regexp,&nbsp;$mime,&nbsp;$matches))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$this-&gt;file_type&nbsp;=&nbsp;$matches[1];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

<p style="text-indent: 2em;">
  然而，通过上面的代码获取到的mime类型不是audio/x-ms-wma，而是video/x-ms-asf，然后上网查了下，在<a href="http://support.microsoft.com/kb/284094/zh-cn" target="_blank">这里</a>似乎看到一点相关的东西，asf、wma、wmv文件格式一样，具体也不了解，也不知道这里类型判断错误是文件本身的原因还是本来就是区别不出来……
</p>

<p style="text-indent: 2em;">
  反正，既然找到了原因，将<span style="text-indent: 32px;">mimes.php中添加的内容改为：</span>
</p>

<pre class="brush:php;toolbar:false">&#39;wma&#39;&nbsp;&nbsp;&nbsp;=&gt;&nbsp;&nbsp;array(&#39;audio/x-ms-wma&#39;,&nbsp;&#39;video/x-ms-wmv&#39;,&nbsp;&#39;video/x-ms-asf&#39;),</pre>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;"></span>虽然说问题算是解决了，但还是得说，给CI这mime类型跪了，敢不敢不要让人感觉这么碎蛋！
</p>
