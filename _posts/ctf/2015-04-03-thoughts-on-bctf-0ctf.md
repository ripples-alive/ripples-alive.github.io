---
title: 'BCTF&#038;0CTF小记'
author: Ripples
layout: post
permalink: /bctf0ctf%e5%b0%8f%e8%ae%b0/
views:
  - 375
categories:
  - CTF
tags:
  - 0CTF
  - BCTF
  - CTF
  - 感想
---
<p style="text-align: left; text-indent: 2em;">
  由于某场比赛的原因，导致了BCTF延迟了一周，于是乎成功和0CTF连着了，连着两周的比赛，真的就是比到人都虚脱了，外加这段时间本来就忙，于是成功体会到了忙成狗的感觉，不过好在现在一切都结束了，虽然结果不尽如人意，但也还算可以接受。
</p>

<!--more-->

<p style="text-align: left; text-indent: 2em;">
  先说说BCTF，BCTF的题从一开始就给人一种蛋疼的感觉，上来就是一个450分的exploit（zhongguancun），连别的选择都木有，然后一直到结束，放出来的题满满的都是reverse、exploit和web，就连crypto的题都是同时打上了reverse的标签，虽说我挺喜欢逆向溢出的题，但BCTF的题目难度却是让我一点也高兴不起来，溢出从头到尾真的感觉就是一题都做不出来，zhongguancun那题也是自己犯二，一直算错长度，所以根本就没找到溢出点，虽说找到溢出点也真的不一定能够会做，逆向的题更是各种不会，逆向水平本来就渣，要不是最后好不容易找到个裸算法的题（redundant_code）慢慢看，不然就要一无所获了。
</p>

<p style="text-align: left; text-indent: 2em;">
  不过BCTF的时候，队友的状况真是让我十分无语，一个个的都啥动静木有，然后尤其是做web的，sqli_engine那个注入玩了不知道多久也没玩出来，到头来竟然是我这边实在做不下去，跑过去玩了下给玩出来的，这简直就是让人情何以堪，我们队从开始比到现在，第一个成功的注入竟然是我弄的……
</p>

<p style="text-align: left; text-indent: 2em;">
  也许是由于要面向国际队伍，这次BCTF的题真的就是把难度整个的提了上去，我们队则简直就是成了被针对的那种，溢出逆向都是稍微会一点的人做不出来的，而web题则是超水，但我们队却是完全不会做web，于是结果不言而喻。
</p>

<p style="text-align: left; text-indent: 2em;">
  再说0CTF，由于BCTF的悲剧，0CTF也算是憋着一口气，并且0CTF有线下赛，我们已经等了很久了，一直都想去现场赛感受一下，这次机会一定不能错过。不过可能是由于BCTF已经比得有点心力交瘁，还没有完全恢复，这次比赛说句实在话，就我个人而言比赛状况也不是很好。
</p>

<p style="text-align: left; text-indent: 2em;">
  首先比赛刚开始的时候，插入了一个腾讯的笔试，于是前几个小时更是浑浑噩噩的过去了。然后下午也明显不在状态，一个简单exploit都不知道在想什么纠结了很久，第一天我基本可以说是没有什么收获，基本都是靠队友在拿分，名次也是很不容乐观，好在第二天总算是勉强找到了一点状态，才慢慢的开始A题，看着名次一点点的往上爬，直到最后拿到决赛的入场卷。
</p>

<p style="text-align: left; text-indent: 2em;">
  这次比赛的整个过程中，不得不说，感觉整个队的比赛氛围至少正常多了，可以看到相互的讨论，各种冒泡，不管有没有做出题，至少可以发现大家都是努力在想方法，这种感觉真的很好。
</p>

<p style="text-align: left; text-indent: 2em;">
  不过这次比赛最让我郁闷的应该是那个exploit400（freenote）了，明明都已经想到了怎么堆溢出进行dword shoot，却由于把权限记错了，记成了exploit300那题的权限，以为GOT表是不能写的了，于是就完全不知道怎么做了，想想就觉得可惜。
</p>

<p style="text-align: left; text-indent: 2em;">
  但既然过都过去了，已经改变不了什么，反正也算是进了线下赛，也不算太不满意，只是接下来的比赛大概需要更认真点，一定要拿到最终决赛的入场卷。
</p>

<p style="text-align: left; text-indent: 2em;">
  至于这两场比赛的WriteUp，只有抽空慢慢写了，最近确实是太忙了，怎么也没想到一开学竟然会忙成这样，为了这两场比赛，也是耽误了不少东西，总归是要慢慢补上了的……
</p>
