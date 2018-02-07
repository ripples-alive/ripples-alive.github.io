---
layout: post
title: 路由折腾记 第四弹
modified: 2017-09-04T23:45:26+08:00
categories: technology
description: PPTP VPN 固定 client IP，安装 ss-server……
tags: [router, OpenWrt]
---

## PPTP VPN 固定 client IP

为了能够对一台通过 PPTP 连上了的机器做端口映射，无疑是希望 PPTP 连上来后分配的 IP 是固定的。
而在之前的配置中，我们的 client IP 配置是全局的，不论通过哪个账号连上来，都是一起分配的 IP。

于是再查了一下现在的[官方文档](https://wiki.openwrt.org/doc/howto/vpn.server.pptpd)，
可以看到 remoteip 选项是可以为每个用户名密码单独配置的。
但是配置后发现还是无效的，在仔细看文档可以发现：

```
There are bugs in BARRIER BREAKER (14.07, r42625) init script. Modify /etc/init.d/pptpd to clean up temporary pptp.conf and chap-secrets. Original init script does not enable multiple simultaneous clients with fixed remote IP's. Following script and modified configuration file enables it:
```

<p id="read-more-anchor"/>

那么我们其实需要把 `/etc/init.d/pptpd` 给换掉，但是换掉后我们每个用户名都**必须**单独配置 remoteip，这样当用户多的时候还是有点烦的。
于是修改了下，相当于合并了下原始的脚本和修改后脚本的优点，最终得到 `/etc/init.d/pptpd` 如下：

```bash
#!/bin/sh /etc/rc.common
# Copyright (C) 2006 OpenWrt.org

START=60
BIN=/usr/sbin/pptpd
DEFAULT=/etc/default/$BIN
RUN_D=/var/run
PID_F=$RUN_D/$BIN.pid
CONFIG=/var/etc/pptpd.conf
CHAP_SECRETS=/var/etc/chap-secrets

setup_login() {
    local section="$1"
    config_get username "$section" username
    config_get password "$section" password
    config_get remoteip "$section" remoteip
    [ -n "$username" ] || return 0
    [ -n "$password" ] || return 0
    [ -n "$remoteip" ] || remoteip='*'

    echo "$username pptp-server $password $remoteip" >> $CHAP_SECRETS
}

setup_config() {
    local section="$1"

    config_get enabled "$section" enabled
    [ "$enabled" -eq 0 ] && return 1

    mkdir -p /var/etc
    cp /etc/pptpd.conf $CONFIG

    config_get localip "$section" localip
    config_get remoteip "$section" remoteip
    [ -n "$localip" ] && echo "localip  $localip" >> $CONFIG
    [ -n "$remoteip" ] && echo "remoteip $remoteip" >> $CONFIG
    return 0
}

start_pptpd() {
    [ -f $DEFAULT ] && . $DEFAULT
    mkdir -p $RUN_D
    for m in arc4 sha1_generic slhc crc-ccitt ppp_generic ppp_async ppp_mppe; do
        insmod $m >/dev/null 2>&1
    done
    ln -sfn $CHAP_SECRETS /etc/ppp/chap-secrets
    service_start $BIN $OPTIONS -c $CONFIG
}

start() {
    config_load pptpd
    setup_config pptpd || return
    config_foreach setup_login login
    start_pptpd
}

stop() {
    service_stop $BIN
    rm -rf $CHAP_SECRETS $CONFIG /etc/ppp/chap-secrets
}
```

## 安装 ss-server

鉴于 PPTP 被万恶的苹果去掉了（我的安全我自己有数，不用你们系统关心！！！），openwrt 在 iPhone 上跑又麻烦连接又慢，还是想搭个 ss。
（好像 socks 也可以加密码，不过先不管那么多了）

默认情况下，openwrt 上的 shadowsocks-libev 是去掉了 ss-server，认为路由器一般没有这个需求。

于是出于不太想去些奇奇怪怪的网站下编译好的，还是自己编译一下。
编译也很简单，首先在 [OpenWrt](https://wiki.openwrt.org/doc/howto/obtain.firmware.sdk) 官网下载 SDK，
然后按照 [openwrt-shadowsocks](https://github.com/shadowsocks/openwrt-shadowsocks) 给出的命令编译即可。
编译好后，在 bin 目录下会有编译生成的 ipk 文件，包括所有的依赖，分在 packages 和 base 中。
观察下来 packages 中的是 opkg 库里面有的，base 里面是 opkg 库里面没有的，传上去安装即可。
其中 libsodium 有的点特殊，opkg 库里的版本旧了点，要装自己编译出来的。
最后附上编译出来所有 [ipk](/attachments/2017/09/02/ss-packages.tar.gz)。

## 路由无限重置

在路由上测试 ss-server 的时候，突然被人拔了电源，再插上发现路由神奇的被重置。
然后在已经绝望的情况下又随便改了点东西，重启发现又被重置，原来是断电就会自动重置，WTF。

不禁想起了之前 DEF CON 的时候路由突然重置的事情，不过这次更惨罢了，感觉最近一定是运气爆表了。

痛定思痛，仔细查了下资料，尤其是[关于文件系统的](https://wiki.openwrt.org/doc/techref/filesystems)。
对于整个 `root file system` 而言，其实质是把一个挂载到 `/rom` 的只读 SquashFS 文件系统和挂载到 `/overlay` 的可写 jffs2 分区合在了一起，对用户透明的映射了根目录 `/`。

然后看一下发现 `/overlay` 在路由上还是挂载了的，检查下会发现之前以为丢失了的配置文件都还安然无恙的存在在 `/overlay` 里面。
再看一下 dmesg 的结果，发现报错：

![jffs2 error](/images/2017/09/04/jffs2.png)

那么此刻问题已经很明显了，`/rom` 与 `/overlay` 在整合的时候出现了问题，导致 fallback 到了 ramoverlay，即直接使用内存。
在没有闪存的情况下，自然配置无法保存，也就体现为每次断电都会重置。

那么此时我们需要做的是排除掉报错，上面的报错信息都与 `/overlay/work/work` 有关，于是查看下此文件：

![jffs2 error](/images/2017/09/04/work.png)

而 `/var/etc/chap-secrets` 文件是之前 PPTP VPN 生成的临时文件，而 `/overlay/work` 目录下面只有这一个文件。
感觉上来说，这个文件夹应该是每次运行的时候的一个工作目录，应该最开始是空的，而由于这个非预期的破损的软连接文件的存在，导致了工作目录的创建出现报错。
于是我们直接删掉这个文件，重启路由可以发现恢复如初。

很幸运这次没有在出现问题后直接刷机，才使得最后能够直接恢复。

在查资料的过程中，还看到个很有意思的情况：

> In particular, you need to be careful what packages you update. While OPKG is more than happy to install an updated package on JFFS2, it's unable to remove the original package from SquashFS; the end result is that you slowly start using more and more space until the JFFS2 partition is filled. The opkg util really has no idea how much space is available on the JFFS2 partition since it's compressed, and so it will blindly keep going until the opkg system crashes – at that point you have so little space you probably can't even use opkg to remove anything.

由于文件系统是压缩存储的，导致无法直接判断空间是否足够，于是可能到处出现神奇的 bug。
由于之前折腾的时候，路由也报过空间不足的错误，所以曾一度怀疑这次问题与这个特性有关。
