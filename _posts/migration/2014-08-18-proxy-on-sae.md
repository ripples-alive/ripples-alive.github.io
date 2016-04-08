---
title: 在 SAE 上架设代理
author: Ripples
layout: post
permalink: /在sae上架设代理/
views:
  - 1068
categories:
  - technology
tags:
  - SAE
  - 代理
---
<p style="text-indent: 2em;">
  听到代理的第一反应大概是翻墙了，不过由于SAE本身是在墙内的，所以SAE上架设的代理不能翻墙～～～
</p>

<p style="text-indent: 2em;">
  那么在SAE上架设代理的作用，个人感觉大概有以下几条：
</p>

<ol class=" list-paddingleft-2" style="list-style-type: decimal;">
  <li>
    <p style="text-indent: 2em;">
      有些神奇的墙内的网站不知道为啥就是上不去（反正我上微信的客服那个网是上不去），经SAE代理就OK了；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      据说SAE线路好，所以有时候可以通过SAE代理加速；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      国外上国内某些网站的时候需要代理（神奇的天朝）；
    </p>
  </li>

  <li>
    <p style="text-indent: 2em;">
      做些奇奇怪怪的事情的时候想要隐藏IP（咳咳～）；
    </p>
  </li>
</ol>

<!--more-->

<p style="text-indent: 2em;">
  废话不多说了，反正有啥用大家伙自己看着办，不同人肯定不同需求。
</p>

<p style="text-indent: 2em;">
  想架设代理，自己写程序当然可以，不过麻烦不说，还不是每个人都有这个技术，于是乎咱偷个懒，直接用GoAgent，常年有人维护，多好～～～
</p>

<p style="text-indent: 2em;">
  我下的是GoAgent 3.0，在server/php中有个index.py，用其替换SAE上python应用中的index.wsgi即可（本来以为要改的，后来一看发现原生支持SAE，惊呆了），如果要php的应用的话，应该就是上传该文件夹下的index.php吧，不过本人没试过，反正python版的能用就行了。这样服务器端就已经架好，剩下的就是修改客户端的配置文件使用了。
</p>

<p style="text-indent: 2em;">
  修改proxy.ini中的[php]段
</p>

<pre class="brush:python;toolbar:false;">[php]    
enable = 0    
password = 123456    
crlf = 0    
validate = 0    
listen = 127.0.0.1:8088    
fetchserver = http://.com/</pre>

<p style="text-indent: 2em;">
  将enable改为1，fetchserver改为SAE服务端的网址即可。不同的GoAgent版本里面段名可能不是php，不过应该都能找到fetchserver，对应修改即可。
</p>

<p style="text-indent: 2em;">
  然后启动GoAgent，在浏览器中配置代理为上面设置的listen中的ip、端口即可。
</p>

<p style="text-indent: 2em;">
  不过要注意，代理流量消耗可不低，各位要注意自己的云豆量力而为！
</p>
