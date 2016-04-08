---
title: 通过SQL实现二维表格
author: Ripples
layout: post
permalink: /通过sql实现二维表格/
views:
  - 300
categories:
  - coding
tags:
  - mysql
  - sql
  - view
---
<p style="text-indent: 2em;">
  前天下午，为了满足PM的要求给做个View，真是写SQL写到要吐了，这大概是我至今为止写过的最恶心的SQL了，没有之一，可又有什么办法了，谁叫我们只是苦力
</p>

<p style="text-indent: 2em;">
  要求本来并不复杂，简单点说就是各种要实现类似按照日期统计每种状态的订单数量，本来按照日期和订单状态做个group by就好了的，然后得到结果格式如下：
</p>

<!--more-->

<table align="center">
  <tr class="firstRow">
    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      日期
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      状态
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      数量
    </td>
  </tr>

  <tr>
    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-31
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state1
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      a
    </td>
  </tr>

  <tr>
    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-31
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state2
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      b
    </td>
  </tr>

  <tr>
    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-30
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state1
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      c
    </td>
  </tr>

  <tr>
    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-30
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state2
    </td>

    <td width="339" valign="middle" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      d
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  可是这下PM不满意了，非说看着不爽，要改成下面的格式：
</p>

<table align="center">
  <tr class="firstRow">
    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      日期
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state1
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      state2
    </td>
  </tr>

  <tr>
    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-31
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      a
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      b
    </td>
  </tr>

  <tr>
    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      2014-12-30
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      c
    </td>

    <td width="339" valign="top" style="word-break: break-all; border-width: 1px; border-style: solid;" align="center">
      d
    </td>
  </tr>
</table>

<p style="text-indent: 2em;">
  听到这要求我也是跪了，这尼玛真是让我把SQL当Excel来玩啊！！！
</p>

<p style="text-indent: 2em;">
  蛋疼完还是得想想到底怎么去做，首先印象中SQL里似乎没有什么函数可以统计某一字段为特定值的记录数，然后大概想了一下，脑子里只有一个想法，首先将各个状态的订单都分别select出来，然后各自按日期group并统计数量，这样就会每个状态得到一个表，最后把这些表join起来就得到了想要的效果。显然，虽然这样很麻烦，但是至少是可行的。
</p>

<p style="text-indent: 2em;">
  既然没有什么好的办法，那就硬着头皮做呗，然而好不容易让SQL跑起来得到预想的效果后，准备存成view的时候却又出现了问题。由于这样实现的时候，在from中会有子查询，然后尝试用Navicat存这样的view的时候，会得到如下的错误：
</p>

<p style="text-align: center;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/G1PENYMDALYEHDS0.jpg" />
</p>

<p style="text-indent: 2em;">
  无奈之下，只得又手动将from中的select又一个个的存成view，然后最后通过将这些view join起来得到结果，这样总算是得到了一个可以使用的view。不过很显然，这样的view不仅写起来极度麻烦，而且效率很低，虽说现在数据量小，都是秒出，但是对于我这种强迫症而言，这简直就是不能接受。于是完成之后又思前想后，总算灵机一动，想到一个不错的解决方案。
</p>

<p style="text-indent: 2em;">
  首先，我们增加一个如下辅助的表：
</p>

<p style="text-align: center;">
  <img src="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2015/01/blob.png" />
</p>

<p style="text-indent: 2em;">
  然后，我们可以通过订单状态和这个表的id字段做连接，此时，当我们count(one)的时候，就会得到订单状态为1的记录数，因为count是不会将NULL的行算上的。类似的，我们需要状态为几的订单，就可以在这个辅助表中加上对应的记录即可，最终得到SQL形式如下：
</p>

<pre class="brush:sql;toolbar:false">SELECT `date`, COUNT(`one`) AS `state1`, COUNT(`two`) AS `state2`
FROM `order` JOIN `assistant` ON `order`.`state` = `assistant`.`id`
GROUP BY `date`
ORDER BY `date` DESC</pre>

<p style="text-align: left; text-indent: 2em;">
  通过这样一种方式，我们不仅极大的降低了SQL语句的复杂程度，更是使得运行效率得到了提升，毕竟和一个如此下的表做连接，代价是很低的。
</p>
