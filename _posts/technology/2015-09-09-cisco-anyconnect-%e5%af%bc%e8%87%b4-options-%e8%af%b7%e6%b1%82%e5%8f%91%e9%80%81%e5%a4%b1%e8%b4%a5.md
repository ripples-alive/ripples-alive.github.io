---
title: Cisco AnyConnect 导致 OPTIONS 请求发送失败
author: Ripples
layout: post
permalink: /cisco-anyconnect-%e5%af%bc%e8%87%b4-options-%e8%af%b7%e6%b1%82%e5%8f%91%e9%80%81%e5%a4%b1%e8%b4%a5/
views:
  - 231
categories:
  - technology
tags:
  - acwebsecagent
  - Any Connect
  - CORS
  - OPTIONS
---
<p style="text-indent: 2em;">
  前两天某人突然表示他在纷印上传文件一上传就立马报失败，换了N个浏览器都屡试不爽，然后大概找了下也不知道是什么问题。于是乎今天找了个空，过去帮他查了一通。
</p>

<!--more-->

<p style="text-indent: 2em;">
  可是，一圈查下来，发现简直一切正常，但就是每次跨域前的OPTIONS请求发都没发到服务器那边去就挂彩了，开burpsuite连个请求都看不到，也是服气。十分怀疑是OPTIONS发不出去，甚至于拿curl试了下，也是果断其余所有请求都能发，就是OPTIONS失败……
</p>

<p style="text-indent: 2em;">
  于是乎，找不到原因，只能在网上搜来搜去，结果竟然在github上发现了这么一个<a href="https://github.com/fex-team/webuploader/issues/62" target="_blank" textvalue="issue">issue</a>。于是一想到前两天他才刚似乎给装过any connect，虽然好像装失败了还是咋滴，就觉得估计八九不离十了，控制台一看果然有acwebsecagent相关的错误，于是想去卸载any connect，但是根本找不到有any connect这个app，只好再搜怎么卸载acwebsecagent，不过现在问题明确，很容易就找到下面这条卸载命令：
</p>

<pre class="brush:bash;toolbar:false">sudo&nbsp;/opt/cisco/anyconnect/bin/websecurity_uninstall.sh</pre>

<p style="text-indent: 2em;">
  运行完之后再试，果然什么问题都没了，不得不说真是深深的给any connect跪了，竟然还有这种破事，网上一搜这个问题也是，除了说CORS的，还有说浏览器网速变慢的，真是不知道这玩意是闹哪样。
</p>

<p style="text-indent: 2em;">
  看来以后装真的小心，千万得把Web security的勾去掉了，不然真的容易死都不知道咋死的。
</p>
