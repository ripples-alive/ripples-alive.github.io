---
title: pyftpdlib架设FTPS（FTP over SSL）服务器
author: Ripples
layout: post
permalink: /pyftpdlib架设ftps（ftp-over-ssl）服务器/
views:
  - 522
categories:
  - coding
tags:
  - ftp
  - pyftpdlib
  - python
  - ssl
---
<p style="text-indent: 2em;">
  由于作业需要，要求搭建一个SSL加密的安全的FTP服务器，在网上随便找了一下，因为比较偏好Python，便决定用pyftpdlib。
</p>

<p style="text-indent: 2em;">
  首先还是使用pip进行安装：
</p>

<pre class="brush:python;toolbar:false">sudo&nbsp;pip&nbsp;install&nbsp;pyftpdlib</pre>

<!--more-->

<p style="text-indent: 2em;">
  安装很顺利，没有遇到什么问题。
</p>

<p style="text-indent: 2em;">
  接下来是使用了，首先架设一个没有使用SSL的FTP服务器：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

from&nbsp;pyftpdlib.authorizers&nbsp;import&nbsp;DummyAuthorizer
from&nbsp;pyftpdlib.handlers&nbsp;import&nbsp;FTPHandler
from&nbsp;pyftpdlib.servers&nbsp;import&nbsp;FTPServer


def&nbsp;main():
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;a&nbsp;dummy&nbsp;authorizer&nbsp;for&nbsp;managing&nbsp;&#39;virtual&#39;&nbsp;users
&nbsp;&nbsp;&nbsp;&nbsp;authorizer&nbsp;=&nbsp;DummyAuthorizer()
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;new&nbsp;user&nbsp;having&nbsp;full&nbsp;r/w&nbsp;permissions
&nbsp;&nbsp;&nbsp;&nbsp;authorizer.add_user(&#39;Jayvic&#39;,&nbsp;&#39;******&#39;,&nbsp;&#39;./ftp_home&#39;,&nbsp;perm=&#39;elradfmwM&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;read-only&nbsp;anonymous&nbsp;user
&nbsp;&nbsp;&nbsp;&nbsp;authorizer.add_anonymous(&#39;./ftp_home&#39;)

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;FTP&nbsp;handler&nbsp;class
&nbsp;&nbsp;&nbsp;&nbsp;handler&nbsp;=&nbsp;FTPHandler
&nbsp;&nbsp;&nbsp;&nbsp;handler.authorizer&nbsp;=&nbsp;authorizer

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;customized&nbsp;banner&nbsp;(string&nbsp;returned&nbsp;when&nbsp;client&nbsp;connects)
&nbsp;&nbsp;&nbsp;&nbsp;handler.banner&nbsp;=&nbsp;"Welcome&nbsp;to&nbsp;Jayvic&#39;s&nbsp;FTP."

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;FTP&nbsp;server&nbsp;class&nbsp;and&nbsp;listen&nbsp;on&nbsp;127.0.0.1:21
&nbsp;&nbsp;&nbsp;&nbsp;address&nbsp;=&nbsp;(&#39;127.0.0.1&#39;,&nbsp;21)
&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;=&nbsp;FTPServer(address,&nbsp;handler)

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;set&nbsp;a&nbsp;limit&nbsp;for&nbsp;connections
&nbsp;&nbsp;&nbsp;&nbsp;server.max_cons&nbsp;=&nbsp;256
&nbsp;&nbsp;&nbsp;&nbsp;server.max_cons_per_ip&nbsp;=&nbsp;5

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;start&nbsp;ftp&nbsp;server
&nbsp;&nbsp;&nbsp;&nbsp;server.serve_forever()


if&nbsp;__name__&nbsp;==&nbsp;&#39;__main__&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;main()</pre>

<p style="text-indent: 2em;">
  由于此处我们使用的是21号端口，所以需要使用sudo来运行，否则会报如下错误：
</p>

<pre class="brush:plain;toolbar:false">Traceback&nbsp;(most&nbsp;recent&nbsp;call&nbsp;last):
&nbsp;&nbsp;File&nbsp;"./ftp_server.py",&nbsp;line&nbsp;37,&nbsp;in&nbsp;&lt;module&gt;
&nbsp;&nbsp;&nbsp;&nbsp;main()
&nbsp;&nbsp;File&nbsp;"./ftp_server.py",&nbsp;line&nbsp;26,&nbsp;in&nbsp;main
&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;=&nbsp;FTPServer(address,&nbsp;handler)
&nbsp;&nbsp;File&nbsp;"/Library/Python/2.7/site-packages/pyftpdlib/servers.py",&nbsp;line&nbsp;145,&nbsp;in&nbsp;__init__
&nbsp;&nbsp;&nbsp;&nbsp;self._af&nbsp;=&nbsp;self.bind_af_unspecified(address_or_socket)
&nbsp;&nbsp;File&nbsp;"/Library/Python/2.7/site-packages/pyftpdlib/ioloop.py",&nbsp;line&nbsp;774,&nbsp;in&nbsp;bind_af_unspecified
&nbsp;&nbsp;&nbsp;&nbsp;raise&nbsp;socket.error(err)
socket.error:&nbsp;[Errno&nbsp;13]&nbsp;Permission&nbsp;denied</pre>

<p style="text-indent: 2em;">
  如果不想使用sudo权限，可以选择换一个用户级别的端口。
</p>

<p style="text-indent: 2em;">
  确认FTP可以正常使用后，接下来，我们将代码换成支持SSL的版本：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

from&nbsp;pyftpdlib.authorizers&nbsp;import&nbsp;DummyAuthorizer
from&nbsp;pyftpdlib.handlers&nbsp;import&nbsp;TLS_FTPHandler
from&nbsp;pyftpdlib.servers&nbsp;import&nbsp;FTPServer


def&nbsp;main():
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;a&nbsp;dummy&nbsp;authorizer&nbsp;for&nbsp;managing&nbsp;&#39;virtual&#39;&nbsp;users
&nbsp;&nbsp;&nbsp;&nbsp;authorizer&nbsp;=&nbsp;DummyAuthorizer()
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;new&nbsp;user&nbsp;having&nbsp;full&nbsp;r/w&nbsp;permissions
&nbsp;&nbsp;&nbsp;&nbsp;authorizer.add_user(&#39;Jayvic&#39;,&nbsp;&#39;******&#39;,&nbsp;&#39;./ftp_home&#39;,&nbsp;perm=&#39;elradfmwM&#39;)
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;read-only&nbsp;anonymous&nbsp;user
&nbsp;&nbsp;&nbsp;&nbsp;authorizer.add_anonymous(&#39;./ftp_home&#39;)

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;TLS&nbsp;FTP&nbsp;handler&nbsp;class
&nbsp;&nbsp;&nbsp;&nbsp;handler&nbsp;=&nbsp;TLS_FTPHandler
&nbsp;&nbsp;&nbsp;&nbsp;handler.authorizer&nbsp;=&nbsp;authorizer
&nbsp;&nbsp;&nbsp;&nbsp;handler.certfile&nbsp;=&nbsp;&#39;./server.crt&#39;
&nbsp;&nbsp;&nbsp;&nbsp;handler.keyfile&nbsp;=&nbsp;&#39;./server.key&#39;

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Define&nbsp;a&nbsp;customized&nbsp;banner&nbsp;(string&nbsp;returned&nbsp;when&nbsp;client&nbsp;connects)
&nbsp;&nbsp;&nbsp;&nbsp;handler.banner&nbsp;=&nbsp;"Welcome&nbsp;to&nbsp;Jayvic&#39;s&nbsp;FTPS."

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;Instantiate&nbsp;FTP&nbsp;server&nbsp;class&nbsp;and&nbsp;listen&nbsp;on&nbsp;127.0.0.1:21
&nbsp;&nbsp;&nbsp;&nbsp;address&nbsp;=&nbsp;(&#39;127.0.0.1&#39;,&nbsp;21)
&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;=&nbsp;FTPServer(address,&nbsp;handler)

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;set&nbsp;a&nbsp;limit&nbsp;for&nbsp;connections
&nbsp;&nbsp;&nbsp;&nbsp;server.max_cons&nbsp;=&nbsp;256
&nbsp;&nbsp;&nbsp;&nbsp;server.max_cons_per_ip&nbsp;=&nbsp;5

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;start&nbsp;ftp&nbsp;server
&nbsp;&nbsp;&nbsp;&nbsp;server.serve_forever()


if&nbsp;__name__&nbsp;==&nbsp;&#39;__main__&#39;:
&nbsp;&nbsp;&nbsp;&nbsp;main()</pre>

<p style="text-indent: 2em;">
  证书的制作可见<a href="http://geekjayvic.sinaapp.com/?p=481" target="_blank">此文</a>，如果使用最后的方法生成的证书，可以不指定keyfile。
</p>

<p style="text-indent: 2em;">
  做这题的时候，脑残了一下，还把FTPS和SFTP给弄混了，竟然在架设好了FTPS的服务器后，使用SFTP命令来连接，真是脑残的不轻，想之前只有在windows下SSH的时候想传文件还是用的FTP软件开SFTP传的，这几天不用竟然就忘了。
</p>
