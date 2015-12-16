---
title: BCTF warmup(Crypto50) WriteUp
author: Ripples
layout: post
views:
  - 430
categories:
  - CTF
tags:
  - BCTF
  - crypto
  - RSA
  - Wiener
  - WriteUp
---
<p style="text-indent: 2em;">
  不得不说，这道题真的感觉很坑，浪费了很多时间也无济于事，只因为完全没往正确的方向走，真的就是方向选对了就超水，也就是导致这题分值如此之低的原因，然而方向不对根本察觉不过来。
</p>

<!--more-->

<p style="text-indent: 2em;">
  题目给了一个加密的字符串和一个public key，但是对于RSA而言，我们知道，其实公钥和私钥完全就是在有哪些人应该知道这一点理解上的一个概念而已，其实完全是可以反过来的，于是我们想会不会是用这个公钥来解密这个字符串，甚至怀疑过会不会是别的加密而不是RSA，也就是这样，我们在错误的路上彻底的回不来了。直到最后比赛结束后，被指点了一下之后才总算是知道了这题怎么做。
</p>

<p style="text-indent: 2em;">
  其实这题做不出的关键还是在于，我们的心中一直是本能的记住，RSA是不可以由公钥推算私钥的，这是RSA的基础。然而，其实有些情况下，是可以有效攻击RSA的，就比如这题，这题公钥和我们平常见到的公钥完全不像，特别大，与n位数差不多了，虽然比赛的时候就觉得很奇怪，但也没多想，就是因为这一点，导致这题RSA可以被攻击了，推算出私钥，至于具体的攻击代码，github上随便一搜就能找到了，这里用Wiener攻击可以破掉。
</p>

<p style="text-indent: 2em;">
  首先我们用openssl来解析说给公钥：
</p>

<pre class="brush:bash;toolbar:false">openssl&nbsp;rsa&nbsp;-in&nbsp;warmup-c6aa398e4f3e72bc2ea2742ae528ed79.pub&nbsp;-pubin&nbsp;-text</pre>

<p style="text-indent: 2em;">
  得到结果如下：
</p>

<pre class="brush:plain;toolbar:false">Modulus&nbsp;(2050&nbsp;bit):
&nbsp;&nbsp;&nbsp;&nbsp;03:67:19:8d:6b:56:14:e9:58:13:ad:d8:f2:2a:47:
&nbsp;&nbsp;&nbsp;&nbsp;17:bc:72:be:1e:ab:d9:33:d1:b8:69:44:fd:b7:5b:
&nbsp;&nbsp;&nbsp;&nbsp;8e:d2:30:be:62:d7:d1:b6:9d:22:20:95:c1:28:c8:
&nbsp;&nbsp;&nbsp;&nbsp;6f:82:01:2e:cb:11:61:91:fd:9d:01:8a:6d:02:f8:
&nbsp;&nbsp;&nbsp;&nbsp;4d:b2:7b:c5:1a:21:30:7d:c8:6f:4b:f7:71:c6:91:
&nbsp;&nbsp;&nbsp;&nbsp;c1:43:e5:ab:e5:49:b5:bd:2d:6e:b1:a2:1f:d6:27:
&nbsp;&nbsp;&nbsp;&nbsp;0e:7e:1b:48:fe:06:11:fb:b2:e1:b0:b3:52:4e:6f:
&nbsp;&nbsp;&nbsp;&nbsp;4d:e8:b4:e4:a3:45:da:44:a1:3d:e8:25:b7:26:08:
&nbsp;&nbsp;&nbsp;&nbsp;db:6c:7c:4a:40:b7:82:66:e6:c8:7b:bf:de:f6:b4:
&nbsp;&nbsp;&nbsp;&nbsp;83:81:d4:9c:45:07:a5:8b:cd:47:b7:6d:64:b4:59:
&nbsp;&nbsp;&nbsp;&nbsp;08:b1:58:bd:7e:bc:4d:ac:b0:b1:cf:d6:c2:c1:95:
&nbsp;&nbsp;&nbsp;&nbsp;74:f4:0e:b2:ef:d0:e9:e1:0d:c7:00:5c:ad:39:bc:
&nbsp;&nbsp;&nbsp;&nbsp;af:52:b9:ea:c3:87:33:68:d6:90:31:c5:e7:24:68:
&nbsp;&nbsp;&nbsp;&nbsp;4a:44:f0:68:ef:d1:d3:dc:09:6d:9b:5d:64:11:e5:
&nbsp;&nbsp;&nbsp;&nbsp;8b:de:e4:3e:46:b9:9a:0d:04:94:b9:db:28:19:5a:
&nbsp;&nbsp;&nbsp;&nbsp;f9:01:af:f1:30:d4:a6:e2:03:da:d0:8d:a5:7f:a7:
&nbsp;&nbsp;&nbsp;&nbsp;e4:02:62:a5:ba:db:2a:32:3e:da:28:b4:46:96:ab:
&nbsp;&nbsp;&nbsp;&nbsp;30:5d
Exponent:
&nbsp;&nbsp;&nbsp;&nbsp;00:f3:95:9d:97:8e:02:eb:9f:06:de:f3:f3:35:d8:
&nbsp;&nbsp;&nbsp;&nbsp;f8:af:d7:60:99:51:dd:ac:60:b7:14:b6:c2:2a:f0:
&nbsp;&nbsp;&nbsp;&nbsp;fa:91:2f:21:0b:34:20:6b:d2:4a:96:01:c7:8d:f4:
&nbsp;&nbsp;&nbsp;&nbsp;a0:27:5f:10:7f:d3:ab:55:2d:95:05:7e:b9:34:e7:
&nbsp;&nbsp;&nbsp;&nbsp;1b:dd:cd:70:45:c2:4b:18:58:7b:8c:8f:cf:5a:dd:
&nbsp;&nbsp;&nbsp;&nbsp;4c:5d:83:f0:c7:7c:94:dc:9c:50:cb:e4:38:e2:b6:
&nbsp;&nbsp;&nbsp;&nbsp;7b:af:d3:16:33:b6:aa:f1:78:1d:90:c3:ad:6f:03:
&nbsp;&nbsp;&nbsp;&nbsp;d0:37:b3:32:18:01:b2:35:46:d4:83:e6:7e:26:06:
&nbsp;&nbsp;&nbsp;&nbsp;7f:7b:22:34:7d:db:c0:c2:d5:92:ce:81:4c:bf:5d:
&nbsp;&nbsp;&nbsp;&nbsp;fc:cc:14:14:37:f1:4e:0b:39:90:f8:80:61:e5:f0:
&nbsp;&nbsp;&nbsp;&nbsp;ba:e5:f0:1e:3f:a7:0d:b0:e9:60:5e:7c:fd:57:5e:
&nbsp;&nbsp;&nbsp;&nbsp;9c:81:ef:ee:c5:29:c3:3f:d9:03:7a:20:fd:8a:cd:
&nbsp;&nbsp;&nbsp;&nbsp;51:3a:c9:63:77:68:31:3e:63:f9:83:8a:e3:51:1c:
&nbsp;&nbsp;&nbsp;&nbsp;dd:0a:9a:2b:51:6f:21:48:c8:d4:75:a3:60:a0:63:
&nbsp;&nbsp;&nbsp;&nbsp;59:44:97:39:ee:cd:25:1a:bb:42:b0:14:57:3e:43:
&nbsp;&nbsp;&nbsp;&nbsp;9f:2f:a4:57:35:57:b2:56:99:ff:c1:1e:63:1c:e8:
&nbsp;&nbsp;&nbsp;&nbsp;ee:97:5a:86:e7:e2:72:bc:f5:f7:6a:93:45:03:48:
&nbsp;&nbsp;&nbsp;&nbsp;fe:3f
writing&nbsp;RSA&nbsp;key
-----BEGIN&nbsp;PUBLIC&nbsp;KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAQEDZxmNa1YU6VgTrdjyKkcX
vHK+HqvZM9G4aUT9t1uO0jC+YtfRtp0iIJXBKMhvggEuyxFhkf2dAYptAvhNsnvF
GiEwfchvS/dxxpHBQ+Wr5Um1vS1usaIf1icOfhtI/gYR+7LhsLNSTm9N6LTko0Xa
RKE96CW3JgjbbHxKQLeCZubIe7/e9rSDgdScRQeli81Ht21ktFkIsVi9frxNrLCx
z9bCwZV09A6y79Dp4Q3HAFytObyvUrnqw4czaNaQMcXnJGhKRPBo79HT3Altm11k
EeWL3uQ+RrmaDQSUudsoGVr5Aa/xMNSm4gPa0I2lf6fkAmKlutsqMj7aKLRGlqsw
XQKCAQEA85Wdl44C658G3vPzNdj4r9dgmVHdrGC3FLbCKvD6kS8hCzQga9JKlgHH
jfSgJ18Qf9OrVS2VBX65NOcb3c1wRcJLGFh7jI/PWt1MXYPwx3yU3JxQy+Q44rZ7
r9MWM7aq8XgdkMOtbwPQN7MyGAGyNUbUg+Z+JgZ/eyI0fdvAwtWSzoFMv138zBQU
N/FOCzmQ+IBh5fC65fAeP6cNsOlgXnz9V16cge/uxSnDP9kDeiD9is1ROsljd2gx
PmP5g4rjURzdCporUW8hSMjUdaNgoGNZRJc57s0lGrtCsBRXPkOfL6RXNVeyVpn/
wR5jHOjul1qG5+JyvPX3apNFA0j+Pw==
-----END&nbsp;PUBLIC&nbsp;KEY-----</pre>

<p style="text-indent: 2em;">
  得到n和e之后，用Wiener攻击，我们可以得到私钥d为：
</p>

<pre class="brush:plain;toolbar:false">4221909016509078129201801236879446760697885220928506696150646938237440992746683409881141451831939190609743447676525325543963362353923989076199470515758399</pre>

<p style="text-indent: 2em;">
  然后接下来就是简单用c ^ d % n来解密了，最后，写的计算脚本如下：
</p>

<pre class="brush:python;toolbar:false">#!/usr/bin/env&nbsp;python
#encoding:utf-8

import&nbsp;Wiener

c&nbsp;=&nbsp;0x1e04304936215de8e21965cfca9c245b1a8f38339875d36779c0f123c475bc24d5eef50e7d9ff5830e80c62e8083ec55f27456c80b0ab26546b9aeb8af30e82b650690a2ed7ea407dcd094ab9c9d3d25a93b2140dcebae1814610302896e67f3ae37d108cd029fae6362ea7ac1168974c1a747ec9173799e1107e7a56d783660418ebdf6898d7037cea25867093216c2c702ef3eef71f694a6063f5f0f1179c8a2afe9898ae8dec5bb393cdffa3a52a297cd96d1ea602309ecf47cd009829b44ed3100cf6194510c53c25ca7435f60ce5f4f614cdd2c63756093b848a70aade002d6bc8f316c9e5503f32d39a56193d1d92b697b48f5aa43417631846824b5e86

n_str&nbsp;=&nbsp;&#39;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;03:67:19:8d:6b:56:14:e9:58:13:ad:d8:f2:2a:47:
&nbsp;&nbsp;&nbsp;&nbsp;17:bc:72:be:1e:ab:d9:33:d1:b8:69:44:fd:b7:5b:
&nbsp;&nbsp;&nbsp;&nbsp;8e:d2:30:be:62:d7:d1:b6:9d:22:20:95:c1:28:c8:
&nbsp;&nbsp;&nbsp;&nbsp;6f:82:01:2e:cb:11:61:91:fd:9d:01:8a:6d:02:f8:
&nbsp;&nbsp;&nbsp;&nbsp;4d:b2:7b:c5:1a:21:30:7d:c8:6f:4b:f7:71:c6:91:
&nbsp;&nbsp;&nbsp;&nbsp;c1:43:e5:ab:e5:49:b5:bd:2d:6e:b1:a2:1f:d6:27:
&nbsp;&nbsp;&nbsp;&nbsp;0e:7e:1b:48:fe:06:11:fb:b2:e1:b0:b3:52:4e:6f:
&nbsp;&nbsp;&nbsp;&nbsp;4d:e8:b4:e4:a3:45:da:44:a1:3d:e8:25:b7:26:08:
&nbsp;&nbsp;&nbsp;&nbsp;db:6c:7c:4a:40:b7:82:66:e6:c8:7b:bf:de:f6:b4:
&nbsp;&nbsp;&nbsp;&nbsp;83:81:d4:9c:45:07:a5:8b:cd:47:b7:6d:64:b4:59:
&nbsp;&nbsp;&nbsp;&nbsp;08:b1:58:bd:7e:bc:4d:ac:b0:b1:cf:d6:c2:c1:95:
&nbsp;&nbsp;&nbsp;&nbsp;74:f4:0e:b2:ef:d0:e9:e1:0d:c7:00:5c:ad:39:bc:
&nbsp;&nbsp;&nbsp;&nbsp;af:52:b9:ea:c3:87:33:68:d6:90:31:c5:e7:24:68:
&nbsp;&nbsp;&nbsp;&nbsp;4a:44:f0:68:ef:d1:d3:dc:09:6d:9b:5d:64:11:e5:
&nbsp;&nbsp;&nbsp;&nbsp;8b:de:e4:3e:46:b9:9a:0d:04:94:b9:db:28:19:5a:
&nbsp;&nbsp;&nbsp;&nbsp;f9:01:af:f1:30:d4:a6:e2:03:da:d0:8d:a5:7f:a7:
&nbsp;&nbsp;&nbsp;&nbsp;e4:02:62:a5:ba:db:2a:32:3e:da:28:b4:46:96:ab:
&nbsp;&nbsp;&nbsp;&nbsp;30:5d
&#39;&#39;&#39;

e_str&nbsp;=&nbsp;&#39;&#39;&#39;
&nbsp;&nbsp;&nbsp;&nbsp;00:f3:95:9d:97:8e:02:eb:9f:06:de:f3:f3:35:d8:
&nbsp;&nbsp;&nbsp;&nbsp;f8:af:d7:60:99:51:dd:ac:60:b7:14:b6:c2:2a:f0:
&nbsp;&nbsp;&nbsp;&nbsp;fa:91:2f:21:0b:34:20:6b:d2:4a:96:01:c7:8d:f4:
&nbsp;&nbsp;&nbsp;&nbsp;a0:27:5f:10:7f:d3:ab:55:2d:95:05:7e:b9:34:e7:
&nbsp;&nbsp;&nbsp;&nbsp;1b:dd:cd:70:45:c2:4b:18:58:7b:8c:8f:cf:5a:dd:
&nbsp;&nbsp;&nbsp;&nbsp;4c:5d:83:f0:c7:7c:94:dc:9c:50:cb:e4:38:e2:b6:
&nbsp;&nbsp;&nbsp;&nbsp;7b:af:d3:16:33:b6:aa:f1:78:1d:90:c3:ad:6f:03:
&nbsp;&nbsp;&nbsp;&nbsp;d0:37:b3:32:18:01:b2:35:46:d4:83:e6:7e:26:06:
&nbsp;&nbsp;&nbsp;&nbsp;7f:7b:22:34:7d:db:c0:c2:d5:92:ce:81:4c:bf:5d:
&nbsp;&nbsp;&nbsp;&nbsp;fc:cc:14:14:37:f1:4e:0b:39:90:f8:80:61:e5:f0:
&nbsp;&nbsp;&nbsp;&nbsp;ba:e5:f0:1e:3f:a7:0d:b0:e9:60:5e:7c:fd:57:5e:
&nbsp;&nbsp;&nbsp;&nbsp;9c:81:ef:ee:c5:29:c3:3f:d9:03:7a:20:fd:8a:cd:
&nbsp;&nbsp;&nbsp;&nbsp;51:3a:c9:63:77:68:31:3e:63:f9:83:8a:e3:51:1c:
&nbsp;&nbsp;&nbsp;&nbsp;dd:0a:9a:2b:51:6f:21:48:c8:d4:75:a3:60:a0:63:
&nbsp;&nbsp;&nbsp;&nbsp;59:44:97:39:ee:cd:25:1a:bb:42:b0:14:57:3e:43:
&nbsp;&nbsp;&nbsp;&nbsp;9f:2f:a4:57:35:57:b2:56:99:ff:c1:1e:63:1c:e8:
&nbsp;&nbsp;&nbsp;&nbsp;ee:97:5a:86:e7:e2:72:bc:f5:f7:6a:93:45:03:48:
&nbsp;&nbsp;&nbsp;&nbsp;fe:3f
&#39;&#39;&#39;

n&nbsp;=&nbsp;int(n_str.replace(&#39;n&#39;,&nbsp;&#39;&#39;).replace(&#39;&nbsp;&#39;,&nbsp;&#39;&#39;).replace(&#39;:&#39;,&nbsp;&#39;&#39;),&nbsp;16)
e&nbsp;=&nbsp;int(e_str.replace(&#39;n&#39;,&nbsp;&#39;&#39;).replace(&#39;&nbsp;&#39;,&nbsp;&#39;&#39;).replace(&#39;:&#39;,&nbsp;&#39;&#39;),&nbsp;16)

attacker&nbsp;=&nbsp;Wiener.Wiener(n,&nbsp;e)
attacker.crack()

decrypted&nbsp;=&nbsp;pow(c,&nbsp;attacker.d,&nbsp;n)
print&nbsp;hex(decrypted)[2:-1].decode(&#39;hex&#39;)</pre>

<p style="text-indent: 2em;">
  这题就这样轻松的解决了，但这题也真的是给了我们不小的教训，有时候思维定势真的很恐怖，其实稍微换个思路就会是柳暗花明。
</p>
