---
layout: post
title: 路由折腾记 第一弹
modified: 2015-11-21T09:12:29+08:00
categories: technology
description: 配置寝室路由，通过内网服务器接通外网，同时配置智能翻墙、广告过滤等。
tags: [router, OpenWrt]
image:
    feature:
    credit:
    creditlink:
comments:
share:
date: 2015-10-20
---

## 准备工作

最近电信到期了，想想也是不想续了，效果各种渣不说，还要限制出国网速，丫的在逗我？

想了想，决定一次性搞到位，接回实验室电脑上网的同时，把翻墙问题也解决了算了。本来以为轻松就能搞定，不过没想到还是弄了一天多才搞定。于是还是记录下，以备以后再用。

这次配置，主要是麻烦在了没有一个基础网络，所以从 DNS 解析开始，都得折腾个彻底。

首先，不解释的刷了 OpenWrt（好吧，我也是第一次玩路由，只是听说这个好，也适合折腾），我是买的个 WNDR4300，然后在官网下的[固件](http://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/nand/openwrt-15.05-ar71xx-nand-wndr4300-ubi-factory.img)，其中 ar71xx 是主控，nand 是 flash 型号。

刷完固件，接下来就是装软件，在 sourceforge 上找到[这个](http://openwrt-dist.sourceforge.net)，由于路由器没有接入网络，我是直接在所给源的页面直接下载了 scp 到路由器，然后通过 `opkg install package-name.ipk` 安装。

然后其中那个 shadowsocks-libev-spec 是在是用不习惯，因为需求比较复杂，shadowsocks-libev-spec 不适合灵活的修改，所以最后还是用的 shadowsocks-libev。

配置的时候，网上各种资料参考来参考去，最后大概还是参考这个[教程](https://softwaredownload.gitbooks.io/openwrt-fanqiang/content/)来做的。

<p id="read-more-anchor"/>

## 自启动脚本

为方便后面说明，这里先贴出最后的自启动脚本 `/etc/init.d/ss`，然后再来一个个解释：

```sh
#!/bin/sh /etc/rc.common

# file: /etc/init.d/ss

START=95

SERVICE_USE_PID=1
SERVICE_WRITE_PID=1
SERVICE_DAEMONIZE=1

start() {
    echo "conf-dir=/etc/dnsmasq.d" >> /etc/dnsmasq.conf
    /etc/init.d/dnsmasq restart

    service_start /usr/bin/ss-local -c /root/etc/out.json -l 1080 -u
    service_start /root/bin/ss-in -c /root/etc/in.json -u
    service_start /root/bin/ss-out -c /root/etc/out.json -u
    service_start /root/bin/ss-in-tunnel -c /root/etc/in.json -l 5300 -L 114.114.114.114:53 -u
    service_start /root/bin/ss-out-tunnel -c /root/etc/out.json -l 5353 -L 8.8.8.8:53 -u

    /root/bin/ss-firewall
}

stop() {
    sed -i '/conf-dir=\/etc\/dnsmasq.d/d' /etc/dnsmasq.conf
    /etc/init.d/dnsmasq restart

    service_stop /usr/bin/ss-local
    service_stop /root/bin/ss-in
    service_stop /root/bin/ss-out
    service_stop /root/bin/ss-in-tunnel
    service_stop /root/bin/ss-out-tunnel

    /etc/init.d/firewall restart
}
```

记住要 disable 安装 shadowsocks-libev 时自动产生的默认自启动项，因为用不到：

```sh
/etc/init.d/shadowsocks disable
/etc/init.d/shadowsocks stop
/etc/init.d/ss enable
/etc/init.d/ss start
```

## DNS 配置

首先使用 ss-tunnel 进行 DNS 查询

```sh
service_start /root/bin/ss-in-tunnel -c /root/etc/in.json -l 5300 -L 114.114.114.114:53 -u
service_start /root/bin/ss-out-tunnel -c /root/etc/out.json -l 5353 -L 8.8.8.8:53 -u
```

其中配置文件类似如下：

```json
{
    "server": "server ip",
    "server_port": 11111,
    "local_address": "0.0.0.0",
    "local_port": 22222,
    "password": "password",
    "timeout": 150,
    "method": "rc4-md5",
    "fast_open": true
}
```

其中 in 的 server 就直接跑在内网服务器上，out 的 server 可以在内网服务器上通过 ss-tunnel 转发翻墙服务器的，也可以直接走 v6 连到翻墙服务器（如果有 v6 的话）。

然后 ss-in-tunnel 和 ss-out-tunnel 都是 ss-tunnel 的软链接，因为 service_start 无法启动两个参数不同的同名程序。

这两个程序启动好后，使用 dig 命令指定端口就可以发现可以查询了。

```sh
dig baidu.com @192.168.1.1 -p 5300
dig google.com @192.168.1.1 -p 5353
```

然后，配置 dnsmasq，监听 53 端口，根据域名转发到不动端口，实现查询。

首先是在 `/etc/dnsmasq.conf` 中增加：

```sh
#no-resolv
#no-pull
#log-queries
conf-dir=/etc/dnsmasq.d
```

然后 `mkdir /etc/dnsmasq.d`，在其中放入 [accelerated-domains.china.conf](https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/accelerated-domains.china.conf) 和 [bogus-nxdomain.china.conf](https://raw.githubusercontent.com/felixonmars/dnsmasq-china-list/master/bogus-nxdomain.china.conf)，其中 accelerated-domains.china.conf 需要修改，将设定的 DNS 服务器改为 `127.0.0.1#5300`。

然后增加 `gfwlist.conf`，写入：

```sh
server=/#/127.0.0.1#5353
```

此时，所有 DNS 请求默认翻墙，对于白名单中的不翻墙解析。然后 bogus nxdomain 是防止不存在域名的解析被劫持。

然后还可以按照[这个](https://softwaredownload.gitbooks.io/openwrt-fanqiang/content/ebook/03.6.html)加入广告过滤，以及网上找到的额外的一些过滤列表：

```sh
#ADblock
bogus-nxdomain=218.30.64.194
#Taobao
address=/s8.taobao.com/127.0.0.1
address=/s.click.taobao.com/127.0.0.1
#Baidu
address=/pos.baidu.com/127.0.0.1
address=/cpro.baidustatic.com/127.0.0.1
address=/cpro.baidu.com/127.0.0.1
address=/cb.baidu.com/127.0.0.1
address=/cbjs.baidu.com/127.0.0.1
address=/drmcmm.baidu.com/127.0.0.1
address=/duiwai.baidu.com/127.0.0.1
address=/eiv.baidu.com/127.0.0.1
address=/spcode.baidu.com/127.0.0.1
address=/baidutv.baidu.com/127.0.0.1
address=/bar.baidu.com/127.0.0.1
#popup
address=/100tjs.com/127.0.0.1
address=/114zhaofang.com/127.0.0.1
address=/116b.com/127.0.0.1
address=/234y.com/127.0.0.1
address=/arpg2.com/127.0.0.1
address=/game3896.com/127.0.0.1
address=/gelo.tw/127.0.0.1
address=/resmkt.dipan.com/127.0.0.1
address=/web.7k7k.com/127.0.0.1
address=/xp3366.com/127.0.0.1
address=/xp9365.com/127.0.0.1
#Union
address=/.allyes.com/127.0.0.1
address=/.cnxad.com/127.0.0.1
address=/.cnxad.net/127.0.0.1
address=/.unionsky.cn/127.0.0.1
address=/.unionsky2.cn/127.0.0.1
address=/.keyrun.cn/127.0.0.1
address=/.keytui.com/127.0.0.1
address=/.100fenlm.cn/127.0.0.1
address=/.chanet.com.cn/127.0.0.1
address=/.doubleclick.net/127.0.0.1
address=/p.tanx.com/127.0.0.1
#Other
address=/atm.youku.com/127.0.0.1
address=/hudong.pl.youku.com/127.0.0.1
```

配置好后记得要 `/etc/init.d/dnsmasq restart` 才会生效。

## 网络请求转发

接下来就要接通网络了，即启动脚本中的

```sh
service_start /root/bin/ss-in -c /root/etc/in.json -u
service_start /root/bin/ss-out -c /root/etc/out.json -u
```

ss-in 和 ss-out 都是 ss-server 的软链接。

然后新建脚本 `/root/bin/ss-firewall` 来设置 iptables 转发规则：

```sh
#!/bin/sh

# file: /root/bin/ss-firewall

# create a new chain named SHADOWSOCKS
iptables -t nat -N SHADOWSOCKS
iptables -t nat -N SHADOWSOCKS_WHITELIST

# Ignore your shadowsocks server's addresses
# It's very IMPORTANT, just be careful.
iptables -t nat -A SHADOWSOCKS -d 10.111.111.111 -j RETURN

# Ignore LANs IP address
iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN

# Check whitelist
iptables -t nat -A SHADOWSOCKS -j SHADOWSOCKS_WHITELIST
iptables -t nat -A SHADOWSOCKS -m mark --mark 1 -p tcp -j REDIRECT --to-ports 22222

# Anything else should be redirected to shadowsocks's local port
iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 23333
# Apply the rules
iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS

# Ignore Asia IP address
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 1.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 14.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 27.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 36.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 39.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 42.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 49.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 58.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 59.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 60.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 61.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 101.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 103.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 106.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 110.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 111.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 112.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 113.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 114.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 115.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 116.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 117.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 118.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 119.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 120.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 121.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 122.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 123.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 124.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 125.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 126.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 169.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 175.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 180.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 182.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 183.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 202.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 203.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 210.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 211.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 218.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 219.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 220.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 221.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 222.0.0.0/8 -j MARK --set-mark 1
iptables -t nat -A SHADOWSOCKS_WHITELIST -d 223.0.0.0/8 -j MARK --set-mark 1
```

整个逻辑写的比较清楚，很容易看懂，也是在那个教程给的上面稍加修改，即所有 nat 的请求过来（路由下面接的设备访问外网肯定得走 nat），内网 IP 不用代理，于是直接 RETURN，外网 IP 默认转发到 23333 端口翻墙，白名单中的 IP 转发到 22222 端口不翻墙访问。

然后还自启动了个 ss-local，在路由上开了 socks 代理，用于强制要求翻墙时使用。

## 定时任务

至此，整个网络已经通了，然后 `crontab -e`，设置下监控，每 30 分钟检查下，以防刮掉。

```sh
*/30 * * * * isfound=$(ps | grep "ss-local" | grep -v "grep"); if [ -z "$isfound" ]; then echo "$(date): restart ss..." >> /tmp/log/ss-monitor.log && /etc/init.d/ss restart; fi
```

## 接通 v6

最后使用的时候，发现 v6 又出了一点问题，由于 v6 没有配置 nat，于是乎本来是没有 v6 的，所以为了有 v6，就在一个 lan 口又插了一根接入内网的网线来提供 v6，那么此时走 v6 的时候，实际上是不通过路由访问。

本来想的挺好，但是问题就在于在访问 Google 等网站时，容易优先使用 v6，而 v6 上现在是有部分墙的（壮哉我 gfw)，所以导致经常抽风间歇性访问不了 Google。而之前用电信的时候，也这样做没出问题在于，在本机用 ss 翻墙时，用的 ShadowsocksX，然后是使用的域名来智能翻墙的，所以 Google 一定是会翻墙的，就不存在 v6 被墙的问题了。

于是想了想，没有什么太好的办法，就采取了 VLAN 的方式，发出多个 Wi-Fi，一个没有 v6，一个有 v6，但是可能会出现上面的 Google 抽风的问题，根据情况连接了。

VLAN 配置的话，首先在 Network 的 Switch 里面新建一个 VLAN：

![vlan configure](/images/2015/11/21/vlan.png)

其中 Port 5 是 WAN 口，Port 1 是接入那根网线的 LAN 口，VLAN 3 是新建的 VLAN，将 Port 1 单独划出，VLAN 1 中把 Port 1 去掉。

然后在 Network 的 Interface 中新建一个 LAN，此 LAN 桥接 VLAN 3 与需要 v6 的 Wi-Fi，原来的 LAN 只桥接 VLAN 1 和不需要 v6 的 Wi-Fi 即可。

折腾完成后，不得不说用起来真是爽 `*\(^o^)/*`，电脑上所有翻墙全关，一样享受美好新世界～～～

最后附上折腾中用的一些文件：[配置文件](/attachments/2015/11/21/conf.zip)、[ipk](/attachments/2015/11/21/software.zip)

----------------------------------------

2016-04-15 UPDATE: 之前配置好之后，路由本身是不翻墙的，这样在路由上面做些操作的时候会比较坑，可以加上一句 `iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS`，就可以把为路由器本身的请求也翻墙了。关于 iptables 的使用，可以参见[这篇文章](/articles/2016/04/14/iptables-port-forward/)。
