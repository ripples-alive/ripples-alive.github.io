---
title: Mac OS X中Apache开启HTTPS
author: Ripples
layout: post
permalink: /mac-os-x%e4%b8%adapache%e5%bc%80%e5%90%afhttps/
views:
  - 1231
categories:
  - technology
tags:
  - apache
  - https
  - OS X
  - ssl
---
<p style="text-indent: 2em;">
  刚倒腾了一下HTTPS，使用的是Apache服务器，由于不想开虚拟机，所以是直接在OS X中进行的设置。
</p>

<p style="text-indent: 2em;">
  要想开启HTTPS，第一步当然还是准备好证书，关于自制证书可以参见<a href="http://geekjayvic.sinaapp.com/?p=481" target="_blank">这里</a>。
</p>

<p style="text-indent: 2em;">
  证书准备好后，就要着手开始配置Apache了。
</p>

<!--more-->

<p style="text-indent: 2em;">
  首先是开启Apache的ssl模块并启用ssl和vhosts的配置文件（去掉<span style="text-indent: 32px;">/etc/apache2/</span><span style="text-indent: 32px;">httpd.conf中</span>下面三行前面的#）：
</p>

<pre class="brush:plain;toolbar:false">LoadModule&nbsp;ssl_module&nbsp;libexec/apache2/mod_ssl.so
Include&nbsp;/private/etc/apache2/extra/httpd-ssl.conf
Include&nbsp;/private/etc/apache2/extra/httpd-vhosts.conf</pre>

<p style="text-indent: 2em;">
  接下来是配置SSL的证书位置，编辑/etc/apache2/extra/httpd-ssl.conf，修改下面两行，分别指向证书文件和私钥文件：
</p>

<pre class="brush:plain;toolbar:false">SSLCertificateFile&nbsp;"/private/etc/apache2/server.crt"
SSLCertificateKeyFile&nbsp;"/private/etc/apache2/server.key"</pre>

<p style="text-indent: 2em;">
  如果说是证书和私钥在同一个文件中的，可以不用设置上面的第二行，注释掉即可。
</p>

<p style="text-indent: 2em;">
  最后是配置虚拟主机，修改/etc/apache2/extra/httpd-vhosts.conf，在文件的最后加入如下几行（其它几个默认的vhosts如果不用可以注释掉）：
</p>

<pre class="brush:plain;toolbar:false">&lt;VirtualHost&nbsp;*:443&gt;
&nbsp;&nbsp;&nbsp;&nbsp;SSLEngine&nbsp;on
&nbsp;&nbsp;&nbsp;&nbsp;SSLCipherSuite&nbsp;ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
&nbsp;&nbsp;&nbsp;&nbsp;SSLCertificateFile&nbsp;/private/etc/apache2/certs/server.crt
&nbsp;&nbsp;&nbsp;&nbsp;SSLCertificateKeyFile&nbsp;/private/etc/apache2/certs/server.key
&nbsp;&nbsp;&nbsp;&nbsp;ServerName&nbsp;localhost
&nbsp;&nbsp;&nbsp;&nbsp;DocumentRoot&nbsp;"/Users/Jayvic/Sites"
&lt;/VirtualHost&gt;</pre>

<p style="text-indent: 2em;">
  其中SSLCertificateFile和SSLCertificateKeyFile也是同上一样指向证书和私钥。
</p>

<p style="text-indent: 2em;">
  至此，配置已经完成，可以使用下面的命令来检查Apache的配置：
</p>

<pre class="brush:bash;toolbar:false">apachectl&nbsp;configtest</pre>

<p style="text-indent: 2em;">
  如果发现有没有开启的模块就开启一下即可，确认配置无误后，使用下面的命令重启Apache即可：
</p>

<pre class="brush:bash;toolbar:false">sudo&nbsp;apachectl&nbsp;restart</pre>

<p style="text-indent: 2em;">
  PS：在配置的时候感觉/private/etc与/etc有点混乱了，就查了下，发现Mac OS X下是用/private/etc，而为了兼容Unix，所以/etc通过链接指向/private/etc，所以基本可以认为使用两个是一样的。
</p>

* * *

<p style="text-indent: 2em;">
  今天偶然看到一篇nginx https的<a href="https://s.how/nginx-ssl/" target="_blank">文章</a>，感觉不错，mark一下。
</p>
