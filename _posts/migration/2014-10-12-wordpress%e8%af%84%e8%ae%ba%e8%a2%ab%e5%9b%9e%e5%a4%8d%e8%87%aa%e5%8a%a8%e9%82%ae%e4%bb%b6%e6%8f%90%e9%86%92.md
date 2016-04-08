---
title: WordPress评论被回复自动邮件提醒
author: Ripples
layout: post
permalink: /wordpress评论被回复自动邮件提醒/
views:
  - 377
categories:
  - technology
tags:
  - wordpress
  - 评论
  - 邮件提醒
---
<p style="text-indent: 2em;">
  最近愈发觉得，评论被回复时候能有邮件提醒很重要，于是乎，今天就倒腾了下。
</p>

<p style="text-indent: 2em;">
  在网上找了下，相关的资料不少，于是乎最后选了个网上的代码，稍加修改使用。
</p>

<p style="text-indent: 2em;">
  将下面代码放在主题的functions.php中即可：
</p>

<!--more-->

<pre class="brush:php;toolbar:false">function&nbsp;comment_mail_notify($comment_id)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;$admin_notify&nbsp;=&nbsp;&#39;1&#39;;&nbsp;//&nbsp;admin&nbsp;要不要收回复通知&nbsp;(&nbsp;&#39;1&#39;=要&nbsp;;&nbsp;&#39;0&#39;=不要&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;$admin_email&nbsp;=&nbsp;get_bloginfo&nbsp;(&#39;admin_email&#39;);&nbsp;//&nbsp;$admin_email&nbsp;可改为你指定的&nbsp;e-mail.
&nbsp;&nbsp;&nbsp;&nbsp;$comment&nbsp;=&nbsp;get_comment($comment_id);
&nbsp;&nbsp;&nbsp;&nbsp;$comment_author_email&nbsp;=&nbsp;trim($comment-&gt;comment_author_email);
&nbsp;&nbsp;&nbsp;&nbsp;$parent_id&nbsp;=&nbsp;$comment-&gt;comment_parent&nbsp;?&nbsp;$comment-&gt;comment_parent&nbsp;:&nbsp;&#39;&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;global&nbsp;$wpdb;
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;if&nbsp;($wpdb-&gt;query("Describe&nbsp;{$wpdb-&gt;comments}&nbsp;comment_mail_notify")&nbsp;==&nbsp;&#39;&#39;)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$wpdb-&gt;query("ALTER&nbsp;TABLE&nbsp;{$wpdb-&gt;comments}&nbsp;ADD&nbsp;COLUMN&nbsp;comment_mail_notify&nbsp;TINYINT&nbsp;NOT&nbsp;NULL&nbsp;DEFAULT&nbsp;0;");
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(($comment_author_email&nbsp;!=&nbsp;$admin_email&nbsp;&&&nbsp;isset($_POST[&#39;comment_mail_notify&#39;]))&nbsp;||
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;($comment_author_email&nbsp;==&nbsp;$admin_email&nbsp;&&&nbsp;$admin_notify&nbsp;==&nbsp;&#39;1&#39;))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$wpdb-&gt;query("UPDATE&nbsp;{$wpdb-&gt;comments}&nbsp;SET&nbsp;comment_mail_notify=&#39;1&#39;&nbsp;WHERE&nbsp;comment_ID=&#39;$comment_id&#39;");
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;$notify&nbsp;=&nbsp;$parent_id&nbsp;?&nbsp;get_comment($parent_id)-&gt;comment_mail_notify&nbsp;:&nbsp;&#39;0&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;$spam_confirmed&nbsp;=&nbsp;$comment-&gt;comment_approved;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;($parent_id&nbsp;!=&nbsp;&#39;&#39;&nbsp;&&&nbsp;$spam_confirmed&nbsp;!=&nbsp;&#39;spam&#39;&nbsp;&&&nbsp;$notify&nbsp;==&nbsp;&#39;1&#39;)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$wp_email&nbsp;=&nbsp;&#39;no-reply@&#39;&nbsp;.&nbsp;preg_replace(&#39;#^www.#&#39;,&nbsp;&#39;&#39;,&nbsp;strtolower($_SERVER[&#39;SERVER_NAME&#39;]));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;e-mail&nbsp;发出点,&nbsp;no-reply&nbsp;可改为可用的&nbsp;e-mail.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$to&nbsp;=&nbsp;trim(get_comment($parent_id)-&gt;comment_author_email);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$subject&nbsp;=&nbsp;&#39;您在&nbsp;[&#39;&nbsp;.&nbsp;html_entity_decode(get_option("blogname"),&nbsp;ENT_QUOTES)&nbsp;.&nbsp;&#39;]&nbsp;的留言有了回复&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$message&nbsp;=&nbsp;&#39;&lt;div&nbsp;style="background-color:#eef2fa;&nbsp;border:1px&nbsp;solid&nbsp;#d8e3e8;&nbsp;color:#111;&nbsp;padding:0&nbsp;15px;&nbsp;-moz-border-radius:5px;&nbsp;-webkit-border-radius:5px;&nbsp;-khtml-border-radius:5px;"&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;&#39;&nbsp;.&nbsp;trim(get_comment($parent_id)-&gt;comment_author)&nbsp;.&nbsp;&#39;,&nbsp;您好!&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;您曾在《&#39;&nbsp;.&nbsp;get_the_title($comment-&gt;comment_post_ID)&nbsp;.&nbsp;&#39;》的留言:&lt;br&nbsp;/&gt;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;trim(get_comment($parent_id)-&gt;comment_content)&nbsp;.&nbsp;&#39;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;&#39;&nbsp;.&nbsp;trim($comment-&gt;comment_author)&nbsp;.&nbsp;&#39;&nbsp;给您的回复:&lt;br&nbsp;/&gt;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;trim($comment-&gt;comment_content)&nbsp;.&nbsp;&#39;&lt;br&nbsp;/&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;您可以点击&lt;a&nbsp;href="&#39;&nbsp;.&nbsp;htmlspecialchars(get_comment_link($parent_id))&nbsp;.&nbsp;&#39;"&gt;查看回复的完整內容&lt;/a&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;还要再度光临&nbsp;&lt;a&nbsp;href="&#39;&nbsp;.&nbsp;get_option(&#39;home&#39;)&nbsp;.&nbsp;&#39;"&gt;&#39;&nbsp;.&nbsp;get_option(&#39;blogname&#39;)&nbsp;.&nbsp;&#39;&lt;/a&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;(此邮件由系统自动发送，请勿回复)&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/div&gt;&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$from&nbsp;=&nbsp;"From:&nbsp;""&nbsp;.&nbsp;html_entity_decode(get_option(&#39;blogname&#39;),&nbsp;ENT_QUOTES)&nbsp;.&nbsp;""&nbsp;&lt;$wp_email&gt;";
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$headers&nbsp;=&nbsp;"$fromnContent-Type:&nbsp;text/html;&nbsp;charset="&nbsp;.&nbsp;get_option(&#39;blog_charset&#39;)&nbsp;.&nbsp;"n";
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wp_mail(&nbsp;$to,&nbsp;$subject,&nbsp;$message,&nbsp;$headers&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
add_action(&#39;comment_post&#39;,&nbsp;&#39;comment_mail_notify&#39;);

/*&nbsp;自动加勾选栏&nbsp;*/
function&nbsp;add_checkbox()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;echo&nbsp;&#39;&lt;input&nbsp;type="checkbox"&nbsp;name="comment_mail_notify"&nbsp;id="comment_mail_notify"&nbsp;value="comment_mail_notify"&nbsp;checked="checked"&nbsp;/&gt;&lt;label&nbsp;for="comment_mail_notify"&gt;有人回复时邮件通知我&lt;/label&gt;&#39;;
}
add_action(&#39;comment_form&#39;,&nbsp;&#39;add_checkbox&#39;);</pre>

<p style="text-indent: 2em;">
  就是给评论comment_post加了个钩子，这样每次评论的时候就可以检测是否应该发送邮件而自动发送。
</p>

<p style="text-indent: 2em;">
  最后再给加上一个勾选框来让用户决定是否收到通知。
</p>

* * *

<p style="text-indent: 2em;">
  前段时间，邮件突然就收不到了，不知道是出了什么问题，无奈之下，就又折腾了一下，为了稳定起见，注册了一个sina邮箱，然后用SAE自带的MAIL服务来发送邮件，于是发送邮件部分代码修改如下：
</p>

<pre class="brush:php;toolbar:false;">&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;($parent_id&nbsp;!=&nbsp;&#39;&#39;&nbsp;&&&nbsp;$spam_confirmed&nbsp;!=&nbsp;&#39;spam&#39;&nbsp;&&&nbsp;$notify&nbsp;==&nbsp;&#39;1&#39;)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$from&nbsp;=&nbsp;&#39;geekjayvic@sina.cn&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$to&nbsp;=&nbsp;trim(get_comment($parent_id)-&gt;comment_author_email);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$subject&nbsp;=&nbsp;&#39;您在&nbsp;[&#39;&nbsp;.&nbsp;html_entity_decode(get_option("blogname"),&nbsp;ENT_QUOTES)&nbsp;.&nbsp;&#39;]&nbsp;的留言有了回复&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$message&nbsp;=&nbsp;&#39;&lt;div&nbsp;style="background-color:#eef2fa;&nbsp;border:1px&nbsp;solid&nbsp;#d8e3e8;&nbsp;color:#111;&nbsp;padding:0&nbsp;15px;&nbsp;-moz-border-radius:5px;&nbsp;-webkit-border-radius:5px;&nbsp;-khtml-border-radius:5px;"&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;&#39;&nbsp;.&nbsp;trim(get_comment($parent_id)-&gt;comment_author)&nbsp;.&nbsp;&#39;,&nbsp;您好!&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;您曾在《&#39;&nbsp;.&nbsp;get_the_title($comment-&gt;comment_post_ID)&nbsp;.&nbsp;&#39;》的留言:&lt;br&nbsp;/&gt;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;trim(get_comment($parent_id)-&gt;comment_content)&nbsp;.&nbsp;&#39;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;&#39;&nbsp;.&nbsp;trim($comment-&gt;comment_author)&nbsp;.&nbsp;&#39;&nbsp;给您的回复:&lt;br&nbsp;/&gt;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;trim($comment-&gt;comment_content)&nbsp;.&nbsp;&#39;&lt;br&nbsp;/&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;您可以点击&lt;a&nbsp;href="&#39;&nbsp;.&nbsp;htmlspecialchars(get_comment_link($parent_id))&nbsp;.&nbsp;&#39;"&gt;查看回复的完整內容&lt;/a&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;还要再度光临&nbsp;&lt;a&nbsp;href="&#39;&nbsp;.&nbsp;get_option(&#39;home&#39;)&nbsp;.&nbsp;&#39;"&gt;&#39;&nbsp;.&nbsp;get_option(&#39;blogname&#39;)&nbsp;.&nbsp;&#39;&lt;/a&gt;&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;p&gt;(此邮件由系统自动发送，请勿回复)&lt;/p&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/div&gt;&#39;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$options&nbsp;=&nbsp;array(&#39;from&#39;&nbsp;=&gt;&nbsp;$from,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;to&#39;&nbsp;=&gt;&nbsp;$to,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;smtp_host&#39;&nbsp;=&gt;&nbsp;&#39;smtp.sina.cn&#39;,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;smtp_username&#39;&nbsp;=&gt;&nbsp;$from,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;smtp_password&#39;&nbsp;=&gt;&nbsp;&#39;password&#39;,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;subject&#39;&nbsp;=&gt;&nbsp;$subject,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;content&#39;&nbsp;=&gt;&nbsp;$message,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;content_type&#39;&nbsp;=&gt;&nbsp;&#39;HTML&#39;,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;charset&#39;&nbsp;=&gt;&nbsp;get_option(&#39;blog_charset&#39;),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#39;nickname&#39;&nbsp;=&gt;&nbsp;html_entity_decode(get_option(&#39;blogname&#39;),&nbsp;ENT_QUOTES)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$mail&nbsp;=&nbsp;new&nbsp;SaeMail();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$mail-&gt;setOpt($options);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ret&nbsp;=&nbsp;$mail-&gt;send();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;($ret&nbsp;===&nbsp;false)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sae_debug($mail-&gt;errno()&nbsp;.&nbsp;&#39;&nbsp;=&gt;&nbsp;&#39;&nbsp;.&nbsp;$mail-&gt;errmsg());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
