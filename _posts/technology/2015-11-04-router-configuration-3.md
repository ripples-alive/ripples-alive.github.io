---
layout: post
title: 路由折腾记 第三弹
modified: 2015-11-21T16:51:00+08:00
categories: technology
description: 实验室路由开放 PPTP VPN，同时优化路由表。
tags: [router, OpenWrt]
---

## 启用 PPTP

鉴于有些人 OpenVPN 还是用不太溜，然后想了想，复旦内网目测没封 PPTP，只是出外网会封，于是就决定顺手再开了个 PPTP 的 VPN。

不得不说，PPTP 这个 VPN 开起来还真是方便，按照这个[教程](http://wiki.openwrt.org/doc/howto/vpn.server.pptpd)感觉没多少配置就搞定了，路由表不用推，也没得推 `-_-#`。

不过唯一遇到的问题就是，`opkg install pptpd` 会发现找不到 `pptpd`，原来是 `OpenWrt Chaos Calmer` 已经默认将 `pptpd` 移除了，于是找到[这个](http://openwrt.jaru.eu.org/chaos_calmer/ar71xx/)地方下的，也不知道这网站靠谱不靠谱……

不过企图开起 v6 支持的时候却给跪了，按网上说的在 `/etc/ppp/options.pptpd` 中加入 `ipv6 , ` 还是连不上，`netstat  -ltp` 也发现仍旧只是监听 `0.0.0.0` 也是郁闷。然后 OpenWrt 上面也没带 socat，于是想了想，反正也可以认为用不到，就懒得搞了。

<p id="read-more-anchor"/>

## 优化路由表

强迫症又发作了，于是决定把路由表优化下，毕竟亚洲 IP 判断还是太宽了。

然后本来是打算用 [Best Route Table](https://github.com/ashi009/bestroutetb) 来做的，然后这玩意还支持自定义输出格式，简直 perfect，于是生成命令如下

```sh
bestroutetb -p custom --gateway.net=1 --gateway.vpn=0 --rule-format='iptables -t nat -A SHADOWSOCKS_WHITELIST -d %prefix/%length -j MARK --set-mark %gw'$'\n' -o mark-best-route
```

一帆风顺的装上后，看的是时候想起来个问题，iptables 和路由表不一样，不是智能匹配最范围最精确的规则，而是从上往下一个个的匹配，虽然基于 bestroutetb 生成的表的包含顺序，从上到下匹配下来，我们得到的 mark 的值确实是对的，但是这意味着每个请求都要把着一千多个规则全部跑一遍啊，并且由于相互包含，导致不能像 chnroutes 那样一旦匹配到一个就可以返回，作为个强迫症患者这样简直不能忍啊！

于是突然想起来似乎有个叫 ipset 的东西，看了一下，发现很适合搭配 chnroutes 的那个白名单表，这样匹配效率超高不解释。

[chnroutes](https://github.com/fivesheep/chnroutes) 实际上也就是从 APNIC 的 [IP 地址分配表](http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest) 中提取中国的 IPv4 部分来生成的，我们这里鉴于格式需要自己处理，于是就自己写了，不过借用下 chnroutes 的提取部分代码了。

生成代码[在此](https://gist.github.com/ripples-alive/79a30f311295500afe78)，鉴于比较懒，直接输出到 stdout，运行时做个重定向就好，生成好后，记得还要修改下之前的 ss-fireware，将白名单部分改成：

```sh
ipset destroy china_list
ipset -N china_list nethash
iptables -t nat -A SHADOWSOCKS -m set --match-set china_list dst -j RETURN
```

弄好后重启，也不知道是不是心理作用，就是感觉跑的飞快～～～
