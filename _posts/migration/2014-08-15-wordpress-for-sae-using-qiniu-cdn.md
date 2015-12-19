---
title: WordPress for SAE 使用七牛CDN加速
author: Ripples
layout: post
permalink: /wordpress-for-sae%e4%bd%bf%e7%94%a8%e4%b8%83%e7%89%9bcdn%e5%8a%a0%e9%80%9f/
views:
  - 2250
categories:
  - technology
tags:
  - CDN
  - SAE
  - wordpress
  - 七牛
---
<p style="text-indent: 2em;">
  今天通过了七牛的实名，于是倒腾了下，在此记录一下。
</p>

<p style="text-indent: 2em;">
  由于强迫症，想找出一个好一点的办法来解决问题，于是乎在网上搜来搜去，最后总算倒腾出个勉强满意的方案。
</p>

<p style="text-indent: 2em;">
  首先七牛的镜像工作过程是：用户在请求七牛中文件时，若七牛有则直接返回，若没有，则请求镜像目标对应文件，存入七牛中并返回，一个bucket只能镜像于一个网址。
</p>

<p style="text-indent: 2em;">
  而在SAE上使用七牛的CDN加速，关键问题在于，由于SAE不能写入的特点，所有上传的文件是传到了storage里面，也就是说，需要加速的文件同时存在于storage和博客本身目录中，使得七牛的wordpress插件不能完美适应SAE。
</p>

<!--more-->

<p style="text-indent: 2em;">
  最开始是看到网上有人号称改写的SAE版的七牛wordpress插件，结果一试之后大失所望，发现其根本就只加速了storage里面的内容，也不知道是不是我打开方式不正确。。。反正感觉就是不知道他改写了什么，除了把默认设置改了之外。。。
</p>

<p style="text-indent: 2em;">
  后来还是直接使用了原版的七牛wordpress插件，本来第一想法是，将其安装两份，一份负责加速博客目录里面的，一份负责加速storage里面的，但是这样需要两个七牛的bucket，然后又要改插件里面的配置文件名神马的，还担心每多一个插件，就降低一点速度，觉得不满意。
</p>

<p style="text-indent: 2em;">
  然后发现网上有config.php里面的常量的，define('SAE_URL', 'http://你的七牛地址/upload')，结果发现自己的里面根本没这个常量，目测版本不同，<span style="text-decoration: line-through;">话说我应该是装的官方提供的那个最新的啊，</span><span style="text-decoration: none;">然后找了下，最后发现在wp-includes里面的functions.php里面有:</span>
</p>

```php
// for SAE
$dir = 'saestor://wordpress/uploads';
$url = 'http://' . $_SERVER['HTTP_APPNAME'] . '-wordpress.stor.sinaapp.com/uploads';
```

<p style="text-indent: 2em;">
  <span style="text-decoration: none;"></span>看了下应该就是要找的地方，修改$url ＝ 七牛地址/uploads之后，果然再插入附件的时候，自动生成的链接地址是指向七牛的。但是现在还是有几个问题：
</p>

* 还是需要两个七牛的bucket；

* 所有之前写的文章中的附件地址还需要手动修改，否则不会加速<span style="text-decoration: line-through;">（虽说我博客才开，基本没有要改的）</span>；

* 今后如果不想用七牛了，那么文章中所有附件地址都得改回来；

<p style="text-indent: 2em;">
  于是乎最好的办法应该还是对php输出内容替换，继续想办法，既然七牛插件里面是通过正则表达式的替换，那么我在“静态文件域名”中填入正则表达式，使其能够同时匹配博客本地目录和storage目录，那不就可以使得地址全部替换了，可是七牛一个bucket只能镜像一个网址，那么就通过rewrite的方式，使得访问博客目录时能自动重定向到storage中，因为storage中的都是/uploads目录下，于是得到方案如下：
</p>

1. 首先在七牛中新建bucket，设置镜像为wordpress博客的主页；

1. 然后在七牛插件中设置七牛绑定的域名为刚建的bucket网址（ACCESS KEY、SECRET KEY是为了插件中的文件更新等功能，可不设）；

1. 接着设置七牛插件中的扩展名为 js|css|png|jpg|jpeg|gif|ico|<span style="color: rgb(255, 0, 0);">zip|rar</span>（附件需要什么扩展名自行添加），设置目录为 wp-content|wp-includes|<span style="color: rgb(255, 0, 0);">uploads</span>，静态文件域名设置为http://SAE二级域名(?:|-wordpress.stor).sinaapp.com；

1. 最后在config.yaml（或者通过AppConfig的rewrite）中添加：

```
- rewrite: if ( path == "/(uploads/.*)" ) goto "http://SAE二级域名-wordpress.stor.sinaapp.com/$1"；
```

<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">PS：上述SAE二级域名是指去掉.sinaapp.com之后的。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">本来至此就可以完事了，只是强迫症，于是乎决定再把avatar头像也加速了。由于对avatar头像的请求含有GET字符串，即有?，七牛对于这种请求，在访问镜像网站对应内容时，似乎是会忽略?后面的内容，导致得到的头像不对，于是乎，需要临时替换掉？，具体步骤如下：</span>
</p>

* 首先在当前主题的functions.php中加上：

```
function cdn_update_avatar($avatar) {
    $regex = '#http://(?:0|1|2|www|en).gravatar.com/([^?]*)?(.*)#';
    return preg_replace ($regex, 'http://七牛bucket网址/$1@$2', $avatar);
}
add_filter('get_avatar', 'cdn_update_avatar');
```

* 然后在config.yaml中添加：

```
- rewrite: if ( path == "/avatar/([^@]*)@(.*)" ) goto "http://gravatar.duoshuo.com/avatar/$1?$2"
```

<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">这样，在七牛的网址中，?被替换成了@，在七牛向博客发出请求后，博客在rewrite的时候会自动把@还原成?，不过如果avatar的网址中含有@目测会挂（不过应该没有吧）。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 2em;">至此，我们完成了对WordPress for SAE的七牛CDN加速，具体效果可以查看本博客。</span>
</p>

<p style="text-indent: 2em;">
  PS：由于本博客不需要缩略图，所以没有倒腾七牛的缩略图功能～～～
</p>

---

<p style="text-indent: 2em;">
  昨天帮人调的时候遇到的问题，在这里做个说明：
</p>

* 不要勾选“自动将远程图片镜像到七牛”，否则会有问题，现在还没研究这个选项是做了什么，但昨天是出了问题就是了；

* 对于除了uploads文件夹外，还有多个并行的文件夹的情况，需要首先在第三步中设置目录的时候把相应的目录也增加上去，然后第四步中rewrite也相应的增加上相应目录的rewrite（就是把第四步中给出的rewrite规则中的uploads换成需要的目录名）；

* 对于SAE使用独立域名的情况，将第三步中的静态文件域名设置修改为 `http://(?:SAE二级域名(?:|-wordpress.stor).sinaapp.com|SAE的独立域名)`；

---

<p style="text-indent: 2em;">
  刚帮人调了个问题，在这里再补充说明下：
</p>

<p style="text-indent: 2em;">
  一定要注意rewrite的顺序，如果不是很懂的可以把上面要加的rewrite写的靠前一点，避免被某些rewrite给拦截导致没生效（比如很多网站会有的重定向到index.php）。
</p>
