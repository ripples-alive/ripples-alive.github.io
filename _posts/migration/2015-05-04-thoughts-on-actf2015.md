---
title: ACTF2015 小记
author: Ripples
layout: post
permalink: /actf2015小记/
views:
  - 768
categories:
  - CTF
tags:
  - ACTF
  - CTF
  - 感想
---
<p style="text-indent: 2em;">
  说句实在话，这次ACTF在开始之前，我真的很担心，很怕这次的结果不好，进不了决赛，毕竟想进决赛想了很久了。然后比赛刚开始的几个小时，看到放出来的几个题，我更是觉得这次要玩完，真的是怎么也没想到，最后竟然能拿到第一，连我自己都不敢相信。
</p>

<!--more-->

<p style="text-indent: 2em;">
  这次的比赛，真的是不禁让人想吐槽shenmegui，无论是从比赛题目还是管理上，无疑都是整个XCTF联赛中最烂的一次。虽然比赛开始前，lym就跟我们预告不要对这次比赛报太大的希望，结果真是果不其然，甚至是比想象中还要惨。比赛一上来先是登录不了，整个比赛推迟半小时开始，本来说第一天晚上补，结果到时间竟然把服务器关了，说忘了，然后比赛过程中更是各种人为错误一大堆，<span style="text-indent: 32px;">然后最最神奇的是，竟然有道web题说因为管理员失误导致难度降低，做出来的队回收了积分（虽然好像回收了对我们队有利，从不会做web飘过）</span>，然后hint完全乱放，都是在有队做出来之后放，这明显不公平吧……
</p>

<p style="text-indent: 2em;">
  然后再来说下比赛题目，签到题就给人与众不同的感觉，应该是见过分值最高的签到题了吧，然后也确实是比其它签到题难想，但是签到题的flag让人看着就感觉不好了，竟然直接表明这是一场Misc CTF，还好不是Web CTF，不然真的就要死无全尸了！然后这次题目真是各种神奇啊，各种信号题，然后还有什么邮箱提交解答方法的人工审核题（果断到最后也没人做），结果，就这种奇葩题目，竟然除了人工审核题和一道有前置条件的题，每题都有人过，我也是醉了。
</p>

<p style="text-indent: 2em;">
  这次能拿第一，关键还是4G那道信号题了，全场唯一过，虽然做出来的队友各种吐槽这题不难啊，还表示应该有队做到最后一步了，然后最后一步又不难，怎么会做不出，但终究还是靠着这题拿到了第一。开场也是靠信号题开的场了，本来是毫无突破口，结果队友突然就跟打了鸡血似的，把那道我看都不想看的信号题给做出来了，然后又没多久，另一个队友又把4G给过了，一下子就跑到了前面，然后我就在那个Secret FS上各种纠结，各种猜题意，直到大概7点才好不容易确定题意，然后蛋疼了半天过了第一关，发现后面个数增多后，之前的想法完全没法完，然后想出了不错的解法后，几乎整个程序推倒重写，然后各种bug，结果中途还被老板拉着扯公司的破事，简直头大。然后果断导致没能在第一天结束前搞定，于是第一天结束前爬上第一挂一个晚上的愿望就这么破灭了（因为以为只有这样才有可能在第一的位置多挂几个小时）。总而言之，第一天我是打了一圈酱油，上午各种无事可做，然后一天也没能做出一题，签到除外（话说这还是我第一次过签到）。
</p>

<p style="text-indent: 2em;">
  然后晚上真是一晚上没睡好，各种失眠，第二天早早侯着开场，然后稍微再调了下后就过掉了FS，成功爬到第一的位置，为此我还专门截了图，因为当时是估计难得再能有第二次。然后再第一天结束后，队友也是发现了Cube Game的点，我再当时又无事可做的情况下过去帮忙，倒腾了会后成功搞定，继续扩大优势。接着发现Dice Game放出后没多久就被各种队过掉，于是果断过去看，看完感觉果然很水，暴力找下随机种子就好，虽然还被Python3的问题坑了，已经各种蛋疼的小坑，最终还是在N个小时的折腾后搞定（不过刚竟然听说，这题有问题，乱发东西就可以轻松随便水过，也是醉了）。然后又是各种蛋疼的啥也不会，直到晚上换口味的猪头提示越放越多才终于搞定。
</p>

<p style="text-indent: 2em;">
  然后之后就再也没能过题了，于是乎，其实直到比赛结束前最后一秒，都还在各种担心会不会被人绝杀，因为到最后我们基本已经弃疗，脑洞题也是实在脑洞不出来，而看后面紧跟着的队，差的是感觉很可能会被做出来的题，真的就是十分担心，毕竟那个时候心态已经变了，看到了拿第一的希望，不愿意就这样失去这个绝佳的机会（大神们各种不在！！！）。还好最后是上天保佑，没有被绝杀。
</p>

<p style="text-indent: 2em;">
  这次比赛应该说是hctf那场之后，最闲的一场比赛了，反正各种不会，会的又都过掉了，除了蛋疼就只能蛋疼。
</p>

<p style="text-indent: 2em;">
  这次比赛，相比于之前的比赛，最让我感到欣慰的是，终于不再是我和某队友两人过题，变成了三人过，虽然由于几道题最后一步是我来做了，所以flag还是由我交了，但是毕竟谁做了多少大家心里都清楚，这次的比赛才更有团队的感觉。
</p>

<p style="text-indent: 2em;">
  然后，不得不说的是，web题还是一如既往的一道也没能搞定，真的就是哭出来啊！
</p>

<p style="text-indent: 2em;">
  从只是为了进决赛不得不过来挣扎一下，到现在拿到第一，真的就是说不出的感觉，可以说，如果不是因为怕出不来线，我这场很可能就不会来做了，后天和大后天就要考算法和概率论，半个学期没听课，真的是不知道怎么去救，然后还有什么英语期中作文要写，感觉自己都要疯了。
</p>

<p style="text-indent: 2em;">
  就先说这么多吧，然后至于WriteUp，投稿了<a href="http://bobao.360.cn/ctf/learning/130.html" target="_blank">360安全播报</a>，给team弄点外快，但愿别被大神喷就好。不过看到群里说放的早了也是有点无奈，通知也没说不能放啊，于是写完了就直接投了，从没弄过不清楚，也就SCTF那次赶早发过一次Code500的WriteUp（但那次官方给了时间点），再就都是悠哉悠哉慢慢写了。然后过几天考试考完了，可能会在博客里补点详细的解题过程，做个笔记。
</p>
