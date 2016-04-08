---
title: shadowsocks 配置优化折腾小记
author: Ripples
layout: post
permalink: /shadowsocks配置优化折腾小记/
views:
  - 1608
categories:
  - technology
tags:
  - hybla
  - shadowsocks
---
<p style="text-indent: 2em;">
  这两天给******开了台vps，试用投票下来，最后决定是用Linode，然后vps的话，显然很大的一个用途就是应付gfw了，我用shadowsocks，于是乎折腾了一下优化，这里记录一下。
</p>

<p style="text-indent: 2em;">
  为了优化，我们拥塞控制算法最好是采用hybla，而内核不自带，所以我们需要自己来编译（Digital Ocean可以不用编译，只要加载一下就好）。
</p>

<!--more-->

<ol class=" list-paddingleft-2">
  <li>
    <p>
      查看机器内核版本：
    </p>

    <pre class="brush:bash;toolbar:false;">uname&nbsp;-r</pre>

    <p>
      我的机器显示为：
    </p>

    <pre class="brush:bash;toolbar:false;">4.0.4-x86_64-linode57</pre>
  </li>

  <li>
    <p>
      在<a href="https://www.kernel.org/pub/linux/kernel">https://www.kernel.org/pub/linux/kernel</a>下载相同版本的源码：
    </p>

    <pre class="brush:bash;toolbar:false;">mkdir&nbsp;kernel
cd&nbsp;kernel
wget&nbsp;https://www.kernel.org/pub/linux/kernel/v3.0/linux-3.11.6.tar.gz&nbsp;
tar&nbsp;xzvf&nbsp;linux-3.11.6.tar.gz</pre>
  </li>

  <li>
    <p>
      安装以下内核编译工具，不然会编译失败：
    </p>

    <pre class="brush:bash;toolbar:false;">CentOS&nbsp;and&nbsp;Fedora
yum&nbsp;update&nbsp;&&&nbsp;yum&nbsp;install&nbsp;-y&nbsp;ncurses-devel&nbsp;make&nbsp;gcc

Ubuntu&nbsp;and&nbsp;Debian
sudo&nbsp;apt-get&nbsp;install&nbsp;-y&nbsp;build-essential&nbsp;libncurses5-dev</pre>
  </li>

  <li>
    <p>
      配置内核编译文件，导出官方的配置文件再修改增加<code>hybla htcp</code>模块：
    </p>

    <pre class="brush:bash;toolbar:false;">cd&nbsp;linux-3.11.6
zcat&nbsp;/proc/config.gz&nbsp;&gt;&nbsp;.config</pre>

    <p>
      编辑<code>.config</code>文件，查找<code>CONFIG_TCP_CONG_CUBIC=y</code>，要编译<code>hybla</code>模块在下面一行增加<code>CONFIG_TCP_CONG_HYBLA=y</code>，要编译<code>htcp</code>模块在下面一行增加<code>CONFIG_TCP_CONG_HTCP=y</code>，两个都要的话，都添加在下面，然后编译：
    </p>

    <pre class="brush:bash;toolbar:false;">make</pre>

    <p>
      耐心等待编译内核完成，单核编译大约需15分钟
    </p>
  </li>

  <li>
    <p>
      修改编译模块的<code>Makefile</code>
    </p>

    <pre class="brush:bash;toolbar:false;">cd&nbsp;net/ipv4/
mv&nbsp;Makefile&nbsp;Makefile.old
vim&nbsp;Makefile</pre>

    <p>
      需要<code>hybla</code>的话修改<code>Makefile</code>改成：
    </p>

    <pre>#&nbsp;Makefile&nbsp;for&nbsp;tcp_hybla.ko
obj-m&nbsp;:=&nbsp;tcp_hybla.o
KDIR&nbsp;:=&nbsp;../..
PWD&nbsp;:=&nbsp;$(shell&nbsp;pwd)
default:
$(MAKE)&nbsp;-C&nbsp;$(KDIR)&nbsp;SUBDIRS=$(PWD)&nbsp;modules</pre>

    <p>
      需要<code>htcp</code>的替换下关键词就好
    </p>
  </li>

  <li>
    <p>
      开始编译模块
    </p>

    <pre class="brush:bash;toolbar:false;">cd&nbsp;../..
make&nbsp;modules</pre>
  </li>

  <li>
    <p>
      测试加载模块
    </p>

    <pre class="brush:bash;toolbar:false;">cd&nbsp;net/ipv4
insmod&nbsp;./tcp_hybla.ko
sysctl&nbsp;net.ipv4.tcp_available_congestion_control</pre>

    <p>
      如果加载成功，会显示：
    </p>

    <pre class="brush:bash;toolbar:false;">net.ipv4.tcp_available_congestion_control&nbsp;=&nbsp;cubic&nbsp;reno&nbsp;hybla</pre>
  </li>

  <li>
    <p>
      设置开机自动加载模块
    </p>

    <p>
      首先将我们需要开机自动加载的模块复制到<code>/lib/modules/4.0.4-x86_64-linode57/kernel/net/ipv4</code>
    </p>

    <pre class="brush:bash;toolbar:false;">sudo&nbsp;mkdir&nbsp;-p&nbsp;/lib/modules/4.0.4-x86_64-linode57/kernel/net/ipv4
sudo&nbsp;cp&nbsp;-a&nbsp;~/kernel/linux-4.0.4/net/ipv4/tcp_hybla.ko&nbsp;/lib/modules/4.0.4-x86_64-linode57/kernel/net/ipv4</pre>

    <p>
      然后由于我是Ubuntu，所以直接修改<code>/etc/modules</code>，再最后加入<code>tcp_hybla</code>即可。
    </p>
  </li>
</ol>

至此，我们就已经将hybla算法添加好了，接下来开始修改配置：

<ol class=" list-paddingleft-2">
  <li>
    <p>
      首先修改<code>/etc/security/limits.conf</code>，在最后加入：
    </p>

    <pre>*&nbsp;soft&nbsp;nofile&nbsp;51200
*&nbsp;hard&nbsp;nofile&nbsp;51200</pre>
  </li>

  <li>
    <p>
      然后执行：
    </p>

    <pre class="brush:bash;toolbar:false;">ulimit&nbsp;-n&nbsp;51200
#&nbsp;其实我执行的：ulimit&nbsp;-n&nbsp;unlimited</pre>
  </li>

  <li>
    <p>
      然后修改<code>/etc/sysctl.conf</code>，在最后加入：
    </p>

    <pre>#&nbsp;max&nbsp;open&nbsp;files
fs.file-max&nbsp;=&nbsp;51200
#&nbsp;max&nbsp;read&nbsp;buffer
net.core.rmem_max&nbsp;=&nbsp;67108864
#&nbsp;max&nbsp;write&nbsp;buffer
net.core.wmem_max&nbsp;=&nbsp;67108864
#&nbsp;default&nbsp;read&nbsp;buffer
net.core.rmem_default&nbsp;=&nbsp;65536
#&nbsp;default&nbsp;write&nbsp;buffer
net.core.wmem_default&nbsp;=&nbsp;65536
#&nbsp;max&nbsp;processor&nbsp;input&nbsp;queue
net.core.netdev_max_backlog&nbsp;=&nbsp;4096
#&nbsp;max&nbsp;backlog
net.core.somaxconn&nbsp;=&nbsp;4096

#&nbsp;resist&nbsp;SYN&nbsp;flood&nbsp;attacks
net.ipv4.tcp_syncookies&nbsp;=&nbsp;1
#&nbsp;reuse&nbsp;timewait&nbsp;sockets&nbsp;when&nbsp;safe
net.ipv4.tcp_tw_reuse&nbsp;=&nbsp;1
#&nbsp;turn&nbsp;off&nbsp;fast&nbsp;timewait&nbsp;sockets&nbsp;recycling
net.ipv4.tcp_tw_recycle&nbsp;=&nbsp;0
#&nbsp;short&nbsp;FIN&nbsp;timeout
net.ipv4.tcp_fin_timeout&nbsp;=&nbsp;30
#&nbsp;short&nbsp;keepalive&nbsp;time
net.ipv4.tcp_keepalive_time&nbsp;=&nbsp;1200
#&nbsp;outbound&nbsp;port&nbsp;range
net.ipv4.ip_local_port_range&nbsp;=&nbsp;10000&nbsp;65000
#&nbsp;max&nbsp;SYN&nbsp;backlog
net.ipv4.tcp_max_syn_backlog&nbsp;=&nbsp;4096
#&nbsp;max&nbsp;timewait&nbsp;sockets&nbsp;held&nbsp;by&nbsp;system&nbsp;simultaneously
net.ipv4.tcp_max_tw_buckets&nbsp;=&nbsp;5000
#&nbsp;turn&nbsp;on&nbsp;TCP&nbsp;Fast&nbsp;Open&nbsp;on&nbsp;both&nbsp;client&nbsp;and&nbsp;server&nbsp;side
net.ipv4.tcp_fastopen&nbsp;=&nbsp;3
#&nbsp;TCP&nbsp;receive&nbsp;buffer
net.ipv4.tcp_rmem&nbsp;=&nbsp;4096&nbsp;87380&nbsp;67108864
#&nbsp;TCP&nbsp;write&nbsp;buffer
net.ipv4.tcp_wmem&nbsp;=&nbsp;4096&nbsp;65536&nbsp;67108864
#&nbsp;turn&nbsp;on&nbsp;path&nbsp;MTU&nbsp;discovery
net.ipv4.tcp_mtu_probing&nbsp;=&nbsp;1

#&nbsp;for&nbsp;high-latency&nbsp;network
net.ipv4.tcp_congestion_control&nbsp;=&nbsp;hybla

#&nbsp;for&nbsp;low-latency&nbsp;network,&nbsp;use&nbsp;cubic&nbsp;instead
#&nbsp;net.ipv4.tcp_congestion_control&nbsp;=&nbsp;cubic</pre>

    <p>
      然后执行：
    </p>

    <pre class="brush:bash;toolbar:false;">sudo&nbsp;sysctl&nbsp;-p</pre>

    <p>
      是配置生效
    </p>
  </li>

  <li>
    <p>
      最后设置好shadowsocks的配置文件，启动即可：
    </p>

    <pre class="brush:js;toolbar:false;">{
&nbsp;&nbsp;&nbsp;&nbsp;"server":&nbsp;"::",
&nbsp;&nbsp;&nbsp;&nbsp;"server_port":&nbsp;8888,
&nbsp;&nbsp;&nbsp;&nbsp;"password":&nbsp;"your&nbsp;password",
&nbsp;&nbsp;&nbsp;&nbsp;"timeout":&nbsp;300,
&nbsp;&nbsp;&nbsp;&nbsp;"method":&nbsp;"rc4-md5",
&nbsp;&nbsp;&nbsp;&nbsp;"fast_open":&nbsp;true
}</pre>

    <pre class="brush:bash;toolbar:false;">sudo&nbsp;ssserver&nbsp;-c&nbsp;/etc/shadowsocks.json&nbsp;-d&nbsp;start&nbsp;--user&nbsp;nobody</pre>

    <p>
    </p>
  </li>
</ol>

<p style="text-indent: 2em;">
  参考文章：<a href="http://www.777s.me/linode-hybla-htcp.html" target="_blank">http://www.777s.me/linode-hybla-htcp.html</a>
</p>
