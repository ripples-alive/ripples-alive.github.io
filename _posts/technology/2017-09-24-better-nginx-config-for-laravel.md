---
layout: post
title: 请给你的 Laravel 一份更好的 Nginx 配置（转载）
modified: 2017-09-24T16:22:39+08:00
categories: technology
description:
tags: [laravel, nginx]
date: 2017-09-24T16:22:39+08:00
---

* ToC
{:toc}

Laravel 是当今最流行的 PHP 框架之一，如所有 PHP 项目一样，通常情况下它都需要运行在 LAMP 或 LNMP 环境之下。如何配置 LNMP 使之为 Laravel 工作可以说是每一个 Laravist 的必修技能。

本文将就 LNMP 环境下如何为 Laravel 的配置 Nginx 进行一次比较详细的探究，通过本文你可以更清晰的认识这份你每天都在使用的配置，理解其中的原理，知晓某些配置的好与不好。在文章的后半段，还会为你推荐一种更为安全，更加适合 Laravel 的配置方案。

<p id="read-more-anchor"/>

## 广为流传的配置

```nginx
root  /path/to/your/laravel/public;
index  index.php index.html index.htm;

location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
    try_files $uri /index.php =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/var/run/php7.0-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

> 这段代码引用自：[How to Install Laravel with an Nginx Web Server on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-laravel-with-an-nginx-web-server-on-ubuntu-14-04)

在大部分的中文教程里面，你都可以看到这份配置，国人对知识的『薪火相传』由此可见一斑。在此我们应先感谢社区前人的辛勤搬运，然后，让我们来详细解读这份配置吧。

## 配置解读

在上面链接的原文中，作者解释了这份配置是如何一步一步写出来的，表达清晰浅显易懂。作者的行文逻辑是：我需要什么，于是我就增加了什么，其中并没有详尽的解释每句配置的作用和意义。鉴于这点美中不足，接下来让我们一行一行的来解读这份配置。

### Step 1

```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

通过 `try_files` 来判断请求是否是静态资源（实际存在的资源），是则返回文件，不是便将请求转发到 `/index.php` 下，并带上 `query_string` 的内容。

### Step 2

请求被转发到 `/index.php` 后，进入 `~ \.php$` 的配置继续执行

```nginx
location ~ \.php$ {
    ...
}
```

#### Step 2.1

```nginx
try_files $uri /index.php =404;
```

再一次 `try_files`, 当被请求的 php 文件不存在时，将请求转发到 `/index.php`

Laravel 是一个单入口框架，标准的实现中，我们不会允许用户直接访问其他的 php 文件，所以我们不需要也没有理由对不存在的 php 文件访问做兼容处理，因此这句配置是不应该有的。

#### Step 2.2

```nginx
fastcgi_split_path_info ^(.+\.php)(/.+)$;
```

> fastcgi_split_path_info
> Defines a regular expression that captures a value for the `$fastcgi_path_info` variable. The regular expression should have two captures: the first becomes a value of the `$fastcgi_script_name` variable, the second becomes a value of the `$fastcgi_path_info` variable.

详细参阅：[官网文档](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_split_path_info)

它的作用是把形如 `/index.php/arg1/arg2` 这样的 `$fastcgi_script_name` 拆成两个参数

* $fastcgi_script_name : /index.php
* $fastcgi_path_info : /arg1/arg2

在 Step 1 中，我们将请求转到了 `/index.php`，并没有 path_info，所以这条配置对于 Laravel 没有意义。

#### Step 2.3

```nginx
fastcgi_pass unix:/var/run/php7.0-fpm.sock;
```

没什么需要说的，必须有的配置

#### Step 2.4

```nginx
fastcgi_index index.php;
```

当 URI 以 `/` 结束的时候，使用 `index.php` 作为默认执行脚本。

然而以 `/` 结束的请求根本就不会进入这个 location 里，所以这句也是废话。

#### Step 2.5

```
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
include fastcgi_params;
```

这两句等同于

```
include  fastcgi.conf;
```

所以使用下面这句就好了

### Step 3

请求被转发到 php-fpm 上，php-fpm 根据传入的参数处理请求。

### 小结

其实这一份配置是一份较通用的 Nginx + FPM 配置，不仅是 Laravel，它还可以应对大部分的 PHP 框架和系统，堪称『以不变应万变』的典范。

当然，当我们只想要适配 Laravel 的时候，这份配置就显得有些臃肿了，并且过多的兼容处理也增加了潜在的不安全因素。比如在开启 `cgi.fix_pathinfo` 的情况下，这样的配置会将你的服务暴露在脚本小子的攻击范围之下。

## 配置简化

```nginx
root  /path/to/your/laravel/public;
index  index.php index.html index.htm;

location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
    fastcgi_pass unix:/var/run/php7.0-fpm.sock;
    include  fastcgi.conf;
}
```

## 新问题：让人崩溃的 PHP-FPM 日志和 APM 服务支持

简化之后的配置清爽了很多，各项功能正常，这样是否就完美了呢？

如果你不需要 PHP-FPM 日志，也没有使用 APM 服务，这配置足够了。但如果有，你应该就会遇到下面的问题：

```
127.0.0.1 -  10/Nov/2016:13:19:52 +0800 "GET /index.php" 304 /current/public/index.php 391.240 2048 2.56%
127.0.0.1 -  10/Nov/2016:13:22:15 +0800 "POST /index.php" 200 /current/public/index.php 333.085 2048 9.01%
127.0.0.1 -  10/Nov/2016:13:22:16 +0800 "POST /index.php" 200 /current/public/index.php 313.295 2048 3.19%
127.0.0.1 -  10/Nov/2016:14:01:48 +0800 "GET /index.php" 200 /current/public/index.php 9.712 2048 102.97%
```

所有的请求，无论请求的路由是什么，日志里面记录的都是 `/index.php`。

嵌入到 fpm 中的 APM 探针也会陷入同样的问题，你会发现 APM 收集到的数据请求全部是 `/index.php`，根本没有办法区分真正的路由，报错了也做不到快速的定位问题。

原因就在于请求进入 fpm 之前，被转发到了 `/index.php?$query_string` 上，这相当于经历了一次 Nginx 内部的资源重定向。当 fpm 接收到请求时，原始的路由信息已经被 `/index.php` 取代了，真实的路由信息只在 CGI 参数 `REQUEST_URI` 中还保留着，这也是 Laravel 还可以正常处理请求的原因。但是 php-fpm 并不使用这个参数来作为请求资源路径（the request URI），所以就有了上述问题。

## SCRIPT_FILENAME 与 SCRIPT_NAME

通过各种尝试之后，我逐步将问题定位到 SCRIPT_FILENAME 与 SCRIPT_NAME 这两个参数上：

### SCRIPT_FILENAME

> The absolute pathname of the currently executing script.

### SCRIPT_NAME

> Contains the current script’s path. This is useful for pages which need to point to themselves. The **FILE** constant contains the full path and filename of the current (i.e. included) file.

PHP-FPM 通过 `SCRIPT_FILENAME` 来找到真正需要执行的文件，`SCRIPT_NAME` 只用来标记当前脚本的 path 信息， PHP-FPM （日志）中的 `%r: the request URI` 参数其实就是 `SCRIPT_NAME` 的值。

因此只要将 `SCRIPT_FILENAME` 指向 Laravel 的入口文件 ‘/index.php’, 而 `SCRIPT_NAME` 保持请求原本的 URI path 便可以达到想要的结果。

想要详细了解 CGI 请参阅：[rfc3875](http://www.faqs.org/rfcs/rfc3875.html)

## 最终配置

```nginx
root   /path/to/your/laravel/public;

location / {
    include  fastcgi_params;
    fastcgi_pass unix:/var/run/php7.0-fpm.sock;
    fastcgi_param  SCRIPT_FILENAME  $document_root/index.php;
}

location ~* \.(ico|css|js|gif|jpg|jpeg|png)(\?.*)?$ {
    # something you want
}

location ~* \.(html|htm|txt)$ {
    # something you want
}
```

这份配置的特点在于把 Laravel 当成一个独立的纯粹的单入口应用来处理，而不是考虑各种 php 环境的通性。它直接将所有非静态资源的请求直接转发到驱动 Laravel 的 PHP-FPM 中，而静态资源使用下面的一条规则，直接取得文件（具体原理见参考文章第一篇）。

## 继续优化的方向

到目前为止，本文的讨论的内容只涉及到了 Nginx 的配置，其实在这个环境体系中还有一个关键的环节 —— PHP-FPM。通常（至少我接触到的）来说，人们只会在一个环境上配置一个 PHP-FPM, 然后所有的项目都使用这个 fpm 提供的服务，PHP-FPM 本身也被设计成可以接受这样的方式。

但是这种把 fpm 当成黑盒的使用方式其实是并不可取的，关于这一点 [A better way to run PHP-FPM](https://ma.ttias.be/a-better-way-to-run-php-fpm/) 中有非常详细的说明。至少在生产环境中，对不同的项目做 fpm 隔离是十分必要的。

如果系统部署基于一个项目一个 fpm 服务的方式，那么还可以通过设置 fpm 的 chroot 获得更好的安全隔离效果。

## 总结

其实这份『最终配置』并不是我自己拍脑袋想出来的，在 Rails、Django 等其他语言框架中，Nginx 大都是这类似的配置方式。现如今无论是何种语言，其流行的框架几乎都已经采用了单一入口模式，这一模式在代码复用和项目可维护性上的优势，古老的多入口模式远不能望其项背。

一个真正的『独立的纯粹的单入口应用』，对于其前方的 Nginx 来说应该是一个黑盒子，他们之间只能通过唯一的入口通道来进行数据交换，而不是让 Nginx 来决定访问系统的哪一个部分。Laravel 就是这样的一匹单入口应用的骏马，你为什么不给它配一个好一点的鞍呢？

虽然现今的 php 框架早已经实现了单入口，但是在 php 环境搭建这个环节上，广为流传的还是各种通用的配置。将就用，俨然已是现今 phper 们的普遍追求，只是偶尔感慨起来，还是会觉得有些许无奈。

---

参考文章：

* [http://homeway.me/2015/05/22/nginx-rewrite-conf/]()
* [http://huoding.com/2013/10/23/290]()
* [https://www.digitalocean.com/community/tutorials/how-to-install-laravel-with-an-nginx-web-server-on-ubuntu-14-04]()
* [https://ma.ttias.be/a-better-way-to-run-php-fpm]()

**原文：<https://yii.im/posts/the-right-way-to-set-nginx-for-laravel/>**
