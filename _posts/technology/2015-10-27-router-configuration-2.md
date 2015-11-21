---
layout: post
title: 路由折腾记 第二弹
modified: 2015-11-21T16:29:01+08:00
categories: technology
description: 在实验室配置路由，架设 OpenVPN，所有连入设备智能翻墙、广告过滤等。
tags: [router, OpenWrt]
image:
    feature:
    credit:
    creditlink:
comments:
share:
---

鉴于上次把寝室网给配的实在用起来太舒服，但其实在寝室呆的时间很短，于是想了想，决定将路由移到实验室，实验室可以直接连接使用，然后开个 VPN，这样在任何有内网的环境中都可以连回来享受同样的网络环境（寝室可以拿室友路由 VPN 连回来），并且还摆脱了对内网服务器的依赖。

鉴于在实验室直接就有外网，所以配置比在寝室还要容易，只要稍加修改即可，剩下要折腾的主要就是 VPN 了。

然后在某人推荐下，决定用 OpenVPN，虽然各个客户端都不原生支持比较蛋疼，但是据说服务器端配置比较容易，于是乎也就认了。

最后参照 OpenWrt 官方 wiki 上的 [Guide](http://wiki.openwrt.org/doc/howto/vpn.openvpn) 来搭建。

搭建过程一切顺利，VPN 轻松连上，traceroute 结果虽然和它说的不太一样，但感觉应该没问题。然而，当企图访问网络时，却跪了。

测试发现，`curl baidu.com` 会提示 `cannot resolve`，但是 `nslookup baidu.com` 却正确出了结果，返回的复旦内网域名解析服务器的结果，然后 `curl ip` 也是可以成功访问的，当时我就懵了，真是从未见过如此奇观。

我没有手动设置域名解析服务器，按理说应该都是用的默认域名解析服务器啊，怎么就 curl 不行。然后尝试手动强制指定使用 114 DNS，发现网络立刻就通了，也是醉了。然后测试了下 dig 直接指定路由为 DNS 服务器，发现解析不了了，指定路由器 5353 端口（即之前那个 ss-tunnel，dnsmasq 的上游服务器之一）解析却正常，甚是奇怪，防火墙神马的确认不会有问题，然后 dnsmasq 监听的 0.0.0.0，但就是不给解析，然而直接连在路由下面却是会给解析的。我这到底是跟 DNS 有多大仇，上次折腾也是大部分的时间花在了折腾 DNS 上，这次咋又被 DNS 给坑着了。

在网上查了一圈，找到[这个](http://lists.thekelleys.org.uk/pipermail/dnsmasq-discuss/2007q2/001323.html)，可以发现，dnsmasq 这玩意虽然监听了所有端口，但是会丢掉所有外部的 dns 请求，也是服气。然后一圈圈的找有啥配置能让其解析所有请求，半天也没找到。

<p id="read-more-anchor"/>

鉴于实在没心思继续折腾，本来都打算先让 VPN 推送 114 DNS，暂时用着，想来 DNS 污染还没那么严重，等到时候有空了再来继续折腾好。

结果晚上最后一下给倒腾挂了之后，第二天早上看 config 的时候，竟然突然发现 `/etc/config/dhcp` 中 `config dnsmasq` 段中有这么一句：

```sh
option localservice '1'
```

看名字似乎就是我要找的东西，于是改 0 后一试，果然就可以解析了，甚好。

然而，当我以为大功告成的时候，悲剧的发现，curl 域名仍旧不行，给跪……这到底是什么鬼，难到是 curl 的时候完全不认我现在默认的那些 DNS（应该是连到无线的时候，无线给推的 DNS），可是 nslookup 都认了啊！

看来最后还是要 VPN 推送个 DNS，不过既然弄好了 dnsmasq 的问题，那就可以推送路由当 DNS 了，也无伤大雅，只是多个配置而已，也不用担心 DNS 服务暴露了，因为真的从 WAN 过来没走 VPN 的，会被防火墙拦住的。

然后其实内网应该直接访问就好，没必要过 VPN，于是顺手一起把路由表推送做了。

最后推送修改如下：

```sh
uci add_list openvpn.myvpn.push='redirect-gateway def1'
#uci add_list openvpn.myvpn.push='route 10.0.0.0 255.0.0.0 net_gateway'
uci add_list openvpn.myvpn.push='route 175.186.0.0 255.255.0.0 net_gateway'
uci add_list openvpn.myvpn.push='route 172.16.0.0 255.240.0.0 net_gateway'
uci add_list openvpn.myvpn.push='route 192.168.0.0 255.255.0.0 net_gateway'
uci add_list openvpn.myvpn.push='route 192.168.10.0 255.255.255.0 vpn_gateway'
uci add_list openvpn.myvpn.push='route 192.168.11.0 255.255.255.0 vpn_gateway'
uci add_list openvpn.myvpn.push='route 192.168.16.0 255.255.255.0 vpn_gateway'
uci add_list openvpn.myvpn.push='dhcp-option DNS 192.168.10.1'
uci add_list openvpn.myvpn.push='dhcp-option DNS 114.114.114.114'
uci commit openvpn
/etc/init.d/openvpn restart
```

`175.186.0.0` 这个是学校无线的 ip 段，不知道为啥这么神奇，似乎不是 ipv4 规定中留的内网网段啊。然后 `10.0.0.0` 网段注释了是因为学校有些资源只能有线访问，无线不行，所以把内网也走 VPN 了就可以访问这些资源了。

全校内一致的体验，简直不能更爽～～～

然后鉴于某特殊情况只有 v6，于是把 v6 也允许了，改下之前 uci 命令也行，我这里就直接改 `/etc/config/openvpn`，将：

```sh
option proto 'udp'
```

换成：

```sh
option proto 'udp6'
```

客户端配置文件也对应改下就好，这样在外面的时候也可以用 v6 顺便连回内网，不过似乎在外也不太会有 v6，除了 vps 和各大高校，也想不出还有哪会有 v6 了……

PS：方便起见，可以将各个证书直接写到 `.ovpn` 配置文件中，具体格式如下：

```
<ca>
</ca>

<cert>
</cert>

<key>
</key>
```

--------------------------------------------------------------------------------

最近使用过程中，发现莫名其妙的不稳定，并且关键不是一直不稳定，一会总是发作，一会又一直无视，很是奇怪，今天终于发现，是在多个客户端同时连接时就开始出事，也是很快便发现是同一个证书默认只允许一个客户端同时在线，所以要改下配置，于是顺手把别的配置也按照[这个](http://wiki.openwrt.org/doc/howto/openvpn-streamlined-server-setup)改了，优化一下，最后配置如下：

```sh
config openvpn 'myvpn'

        option enabled '1'

        # --- Protocol ---#
        option dev 'tun'
        option dev 'tun0'
        option topology 'subnet'
        option proto 'udp6'
        option port '1194'

        #--- Routes ---#
        option server '192.168.10.0 255.255.255.0'
        option ifconfig '192.168.10.1 255.255.255.0'

        #--- Client Config ---#
        #option ccd_exclusive '1'
        option ifconfig_pool_persist '/etc/openvpn/clients/ipp.txt'
        #option client_config_dir '/etc/openvpn/clients/'

        #--- Pushed Routes ---#
        list push 'redirect-gateway def1'
        list push 'route 175.186.0.0 255.255.0.0 net_gateway'
        list push 'route 172.16.0.0 255.240.0.0 net_gateway'
        list push 'route 192.168.0.0 255.255.0.0 net_gateway'
        list push 'route 192.168.10.0 255.255.255.0 vpn_gateway'
        list push 'route 192.168.11.0 255.255.255.0 vpn_gateway'
        list push 'route 192.168.16.0 255.255.255.0 vpn_gateway'
        list push 'dhcp-option DNS 192.168.10.1'
        list push 'dhcp-option DNS 114.114.114.114'

        #--- Encryption ---#
        option cipher 'AES-256-CBC'
        option dh '/etc/openvpn/keys/dh2048.pem'
        #option pkcs12 '/etc/openvpn/keys/server.p12'
        option ca '/etc/openvpn/keys/ca.crt'
        option cert '/etc/openvpn/keys/server.crt'
        option key '/etc/openvpn/keys/server.key'
        option tls_auth '/etc/openvpn/keys/ta.key 0'

        #--- Logging ---#
        option log '/tmp/openvpn.log'
        #option log_append '/tmp/openvpn.log'
        option status '/tmp/openvpn-status.log'
        option verb '3'

        #--- Connection Options ---#
        option keepalive '10 120'
        #option comp_lzo 'yes'

        #--- Connection Reliability ---#
        option duplicate_cn '1'
        option client_to_client '1'
        option persist_key '1'
        option persist_tun '1'

        #--- Connection Speed ---#
        option sndbuf '393216'
        option rcvbuf '393216'
        option fragment '0'
        option mssfix '0'
        option tun_mtu '48000'

        #--- Pushed Buffers ---#
        list push 'sndbuf 393216'
        list push 'rcvbuf 393216'

        #--- Permissions ---#
        option user 'nobody'
        option group 'nogroup'
```

然后 `comp_lzo` 被迫关掉了，开着的时候，一访问 Gmail，服务器就崩溃，报错如下：

```
Mon Nov  2 05:11:52 2015 us=895190 ripples/::ffff:10.147.109.29 Assertion failed at lzo.c:202
```

然后去 Github 上 openvpn 源代码，[那行](https://github.com/OpenVPN/openvpn/blob/release/2.3/src/openvpn/lzo.c#L202)是一句 assert：

```c
ASSERT (buf_safe (&work, zlen));
```

感觉看到这名字真是给跪，我都是正常访问，咋就成危险数据包了……
