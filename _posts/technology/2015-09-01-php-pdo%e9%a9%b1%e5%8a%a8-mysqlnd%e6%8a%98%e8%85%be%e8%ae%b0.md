---
title: php PDO驱动 mysqlnd折腾记
author: Ripples
layout: post
views:
  - 166
categories:
  - technology
tags:
  - casts
  - mysql
  - mysqlnd
  - PDO
---
&nbsp; &nbsp; 昨天晚上折腾的laravel的时候，又是发现本地的laravel的model自动对字段有类型识别，感到很奇怪，在想难不成是每次都直接先拉了一次表信息，那这样岂不是代价太大了，于是趁着这次有时间，好好的钻研了一番。

&nbsp;&nbsp;&nbsp;&nbsp;一路跟着laravel的流程一点点的打log，发现了下面这段代码：

<!--more-->

<pre class="brush:php;toolbar:false">public&nbsp;function&nbsp;select($query,&nbsp;$bindings&nbsp;=&nbsp;[],&nbsp;$useReadPdo&nbsp;=&nbsp;true)
{
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$this-&gt;run($query,&nbsp;$bindings,&nbsp;function&nbsp;($me,&nbsp;$query,&nbsp;$bindings)&nbsp;use&nbsp;($useReadPdo)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;($me-&gt;pretending())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;[];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;For&nbsp;select&nbsp;statements,&nbsp;we&#39;ll&nbsp;simply&nbsp;execute&nbsp;the&nbsp;query&nbsp;and&nbsp;return&nbsp;an&nbsp;array
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;of&nbsp;the&nbsp;database&nbsp;result&nbsp;set.&nbsp;Each&nbsp;element&nbsp;in&nbsp;the&nbsp;array&nbsp;will&nbsp;be&nbsp;a&nbsp;single
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;row&nbsp;from&nbsp;the&nbsp;database&nbsp;table,&nbsp;and&nbsp;will&nbsp;either&nbsp;be&nbsp;an&nbsp;array&nbsp;or&nbsp;objects.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$statement&nbsp;=&nbsp;$this-&gt;getPdoForSelect($useReadPdo)-&gt;prepare($query);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$statement-&gt;execute($me-&gt;prepareBindings($bindings));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$statement-&gt;fetchAll($me-&gt;getFetchMode());
&nbsp;&nbsp;&nbsp;&nbsp;});
}</pre>

&nbsp;&nbsp;&nbsp;&nbsp;很明显，这里直接就是操作pdo执行了，然后那句execute的结果发现就已经是识别出数据类型了的，也就是说，PDO做了这件事，不像是laravel自己做的，难不成是laravel升级之后怎么配置了下PDO（因为这次本地用的是最新的laravel）？甚感诧异之下，我又跑服务器上去运行了下，发现服务器给的结果还是没有数据类型的，当时我就**了。

&nbsp;&nbsp;&nbsp;&nbsp;想想看，我跟服务器上的php版本确实是不大相同，我是php5.5.25，服务器是5.5.9，但就一个小版本号的差距不至于吧！于是十分蛋疼的在Google上好好搜了一番，结果发现在php5.3之后，php mysql换了个mysqlnd的驱动，这个驱动本身提供了对数据类型的识别，而之前的libmysql不支持，可是我都已经php5.5了，怎么还没有，也是醉了！

&nbsp;&nbsp;&nbsp;&nbsp;为了确认问题，又在stack overflow上找了个测试mysql到底用的啥驱动的代码跑了下：

<pre class="brush:php;toolbar:false">&lt;?php
$hasMySQL&nbsp;=&nbsp;false;
$hasMySQLi&nbsp;=&nbsp;false;
$withMySQLnd&nbsp;=&nbsp;false;
$sentence=&#39;&#39;;

if&nbsp;(function_exists(&#39;mysql_connect&#39;))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;$hasMySQL&nbsp;=&nbsp;true;
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"(Deprecated)&nbsp;MySQL&nbsp;&lt;b&gt;is&nbsp;installed&lt;/b&gt;&nbsp;";
}&nbsp;else&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"(Deprecated)&nbsp;MySQL&nbsp;&lt;b&gt;is&nbsp;not&lt;/b&gt;&nbsp;installed&nbsp;";

if&nbsp;(function_exists(&#39;mysqli_connect&#39;))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;$hasMySQLi&nbsp;=&nbsp;true;
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"and&nbsp;the&nbsp;new&nbsp;(improved)&nbsp;MySQL&nbsp;&lt;b&gt;is&nbsp;installed&lt;/b&gt;.&nbsp;";
}&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"and&nbsp;the&nbsp;new&nbsp;(improved)&nbsp;MySQL&nbsp;&lt;b&gt;is&nbsp;not&nbsp;installed&lt;/b&gt;.&nbsp;";

//&nbsp;这句是关键，只有mysqlnd才提供了mysqli_fetch_all
if&nbsp;(function_exists(&#39;mysqli_fetch_all&#39;))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;$withMySQLnd&nbsp;=&nbsp;true;
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"This&nbsp;server&nbsp;is&nbsp;using&nbsp;MySQLnd&nbsp;as&nbsp;the&nbsp;driver.";
}&nbsp;else
&nbsp;&nbsp;&nbsp;&nbsp;$sentence.=&nbsp;"This&nbsp;server&nbsp;is&nbsp;using&nbsp;libmysqlclient&nbsp;as&nbsp;the&nbsp;driver.";

echo&nbsp;$sentence;</pre>

&nbsp; &nbsp; 果然就发现服务器上用的是libmysql，当时那个愤怒啊，之前蛋疼了好久的数据库为啥就不能自动识别下类型的问题，竟然是因为用了一个这么老的驱动，阿里云是闹哪样啊，为啥默认装这种老旧的东西！！！

&nbsp; &nbsp; 不过不管咋滴，先把驱动换掉再说，网上看到各种都是说mysql编译的时候改参数，不过apt-get看了下，发现有php5-mysqlnd这个玩意，于是果断直接装了，然后再运行脚本发现已经好了（apt-get大法好(>_>)，自动enable mod＋restart fpm）。

&nbsp; &nbsp; 最后想想的话，laravel的那个$casts功能其实就没有我之前觉得的那么重要了，反正类型已经识别出来了，不过，设置一下也还是有好处的，可以保证取出来的model属性被我们修改后也一定是正确的数据类型，毕竟PDO只能保证取出数据的初始数据类型是对的，并且想来bool类型也识别不了，因为mysql就没bool……
