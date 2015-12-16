---
title: WordPress for SAE 使用 Memcache
author: Ripples
layout: post
views:
  - 1024
categories:
  - technology
tags:
  - memcache
  - SAE
  - wordpress
---
<p style="text-indent: 2em;">
  本人使用的是SAE应用仓库中WordPress for SAE，使用应用安装包装的，没有在线装，查看了下版本号是3.4.1。
</p>

<p style="text-indent: 2em;">
  由于介绍中明确说明“轻量的Memcache缓存模块，加快网页显示速度的同时减少资源消耗，为您节省云豆”，于是在SAE中初始化了Memcache后便也没有在意，只是最近在查看资源报表的时候，发现Memcache只有内存消耗，请求次数始终是0，越想越觉得不对劲，于是便决定认真查一查。
</p>

<!--more-->

<p style="text-indent: 2em;">
  本来之前都是大概看了眼，大家都是说官方的直接初始化了Memcache后就会自动启用了，但是这次既然决定认真查了，就专门查看了下mysql的请求次数，在footer中添加以下语句即可：
</p>

```php
<?php echo get_num_queries();?> Queries. <?php timer_stop(1);?> Seconds
```

<p style="text-indent: 2em;">
  这一查果然就查出问题了，不管我是否初始化了Memcache，mysql的查询次数根本就没有变，这明显是不对劲的（话说其实如果Wordpress开启了Memcache，SAE不初始化Memcache的时候，网站会直接挂彩）。
</p>

<p style="text-indent: 2em;">
  于是乎，我看了下其它的WordPress在SAE上的移植版（非官方的应用仓库中的）对Memcache的处理，发现是用了一个<a href="http://wordpress.org/plugins/memcached/" target="_blank" textvalue="Memcached Object Cache">Memcached Object Cache</a>插件，将一个object-cache.php放在wp-content中来启用的Memcache。我便到我的SAE上的WordPress和官方的安装文件中找了下，果然就没有发现这个文件。
</p>

<p style="text-indent: 2em;">
  那么，不解释，赶快给自己的WordPress中加上了object-cache.php，然后mysql的查询次数果断就下来了，SAE后台统计的Memcache请求次数也不是0了，消耗的云豆终于不是0了（似乎第一次因为看到扣云豆了而高兴）～～～
</p>

<p style="text-indent: 2em;">
  PS：由于SAE上Memcache的使用方法稍有区别，不需要设置server，所以object-cache.php需要修改一下，将构造函数WP_Object_Cache()中的：
</p>

```php
<?php
foreach ( $buckets as $bucket => $servers) {
    $this->mc[$bucket] = new Memcache();
    foreach ( $servers as $server  ) {
        list ( $node, $port ) = explode(':', $server);
        if ( !$port )
            $port = ini_get('memcache.default_port');
        $port = intval($port);
        if ( !$port )
            $port = 11211;
        $this->mc[$bucket]->addServer($node, $port, true, 1, 1, 15, true, array($this, 'failure_callback'));
        $this->mc[$bucket]->setCompressThreshold(20000, 0.2);
    }
}
```

<p style="text-indent: 2em;">
  修改成：
</p>

<pre class="brush:php;toolbar:false">foreach ( $buckets as $bucket =&gt; $servers) {
    $this-&gt;mc[$bucket] = memcache_init();
}</pre>

<p style="text-indent: 2em;">
  也可以在网上找到别人已经修改好的<a href="http://blog.gimhoy.com/archives/memcached.html" target="_blank">SAE移植版</a>，后面附件中也是修改好的。
</p>



<p style="text-indent: 2em;">
  本来至此，Memcache就已开启，只是由于本人使用的主题的问题，在进入后台时会出现如下错误：
</p>

<p style="text-indent: 2em;">
  <strong style="font-family: Times; font-size: medium; white-space: normal;">Fatal error</strong><span style="font-family: Times; font-size: medium;">: ThemeUpdateChecker::injectUpdate() [<a href='themeupdatechecker.injectupdate'>themeupdatechecker.injectupdate</a>]: The script tried to execute a method or access a property of an incomplete object. Please ensure that the class definition &quot;ThemeUpdate&quot; of the object you are trying to operate on was loaded _before_ unserialize() gets called or provide a __autoload() function to load the class definition in&nbsp;</span><strong style="font-family: Times; font-size: medium; white-space: normal;">wp-content/themes/clearision/func/theme-update-checker.php</strong><span style="font-family: Times; font-size: medium;">&nbsp;on line&nbsp;</span><strong style="font-family: Times; font-size: medium; white-space: normal;">182</strong>
</p>

<p style="text-indent: 2em;">
  <span style="font-family:Times;font-size:16px">检查了一下，发现是主题内置插件中：</span>
</p>

```php
<?php
public function injectUpdate($updates){
    $state = get_option($this->optionName);
    //Is there an update to insert?
    if ( !empty($state) && isset($state->update) && !empty($state->update) ){
        $updates->response[$this->theme] = $state->update->toWpFormat();
    }
    return $updates;
}
```

<p style="text-indent: 2em;">
  <span style="font-family:Times;font-size:16px"></span>其中的$state->update是个__PHP_Incomplete_Class，虽然不知道为什么是在开启Memcache后才出现，但是既然只是检查更新的，那就注释了就好，至于以后还会不会出神马奇奇怪怪的问题，就不得而知了，但愿不会吧！
</p>

<p style="line-height: 16px; text-indent: 2em;">
  <img src="http://geekjayvic.sinaapp.com/wp-content/plugins/wp-ueditor2/ueditor/dialogs/attachment/fileTypeImages/icon_rar.gif" /><a href="http://geekjayvic-wordpress.stor.sinaapp.com/uploads/2014/08/memcached.zip">memcached.zip</a>
</p>
