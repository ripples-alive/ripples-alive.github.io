---
layout: post
title: 转发 PPTP VPN
modified: 2016-04-14T19:26:51+08:00
categories: technology
description: 通过端口转发连接 PPTP VPN
tags: [pptp, vpn, forward]
date: 2016-04-14T19:26:51+08:00
---

为了能够从公网连接 VPN 回来，最终还是把路由移到了寝室，
外网是通了，但电信毕竟上传带宽小，效果是在惨不忍睹，于是在校内的时候还是要选择内网连接。

然而我旦信息办脑子常年有洞，大张江的寝室运气不好，惨遭毒手，于是便出现这等诡异现象：
从实验室有线能连回寝室，但是，用学校无线怎么也 ping 不通寝室。
为了能够从无线连回寝室，于是便想用实验室一台台机做转发。

OpenVPN 转发的时候，直接一行 socat 转 UDP 就轻松搞定，然而想转 PPTP 的时候却蛋疼了。
PPTP 的 1723 端口转发下 TCP 也很容易，但是 ~~47 端口~~（47 是协议号，之前看错了，如有误导，非常抱歉，感谢评论指出） 还有个 GRE 协议再使用，于是也得转，
鉴于 socat 实在玩不转，也不知道能不能实现这个转发，于是最终还是回到 iptables 转发。

iptables 一直没能搞太清楚，于是趁着这次机会，又好好学习了一遍万能的 iptables，
这次在网上找到一篇觉得讲的很清楚的文章，为了防止找不到了，[转载了一份](/articles/2016/04/14/iptables-port-forward/)。

<p id="read-more-anchor"/>

按照这篇文章的说明，很轻松的完成了 TCP 的转发，然而 GRE 的转发还是不生效，
最后又在网上找来找去，终于发现，需要加载一个模块的样子：`modprobe ip_nat_pptp`。
加载完成后，再连接立刻就连上了。

于是最后得到操作脚步如下：

```sh
echo 1 > /proc/sys/net/ipv4/ip_forward
modprobe ip_nat_pptp

local_ip=xx.xx.xx.xx
remote_ip=yy.yy.yy.yy

iptables -P FORWARD DROP
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# forward socks
iptables -t nat -A PREROUTING -d $local_ip -p tcp --dport 10801 -j DNAT --to $remote_ip
iptables -A FORWARD -d $remote_ip -p tcp --dport 10801 -j ACCEPT
iptables -t nat -A POSTROUTING -d $remote_ip -p tcp --dport 10801 -j SNAT --to $local_ip

# forward PPTP
iptables -t nat -A PREROUTING -d $local_ip -p tcp --dport 1723 -j DNAT --to $remote_ip
iptables -A FORWARD -d $remote_ip -p tcp --dport 1723 -j ACCEPT
iptables -t nat -A POSTROUTING -d $remote_ip -p tcp --dport 1723 -j SNAT --to $local_ip
iptables -t nat -A PREROUTING -d $local_ip -p gre -j DNAT --to $remote_ip
iptables -A FORWARD -d $remote_ip -p gre -j ACCEPT
iptables -t nat -A POSTROUTING -d $remote_ip -p gre -j SNAT --to $local_ip
```
