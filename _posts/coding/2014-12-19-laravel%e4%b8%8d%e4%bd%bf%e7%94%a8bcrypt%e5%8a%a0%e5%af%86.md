---
title: Laravel不使用Bcrypt加密
author: Ripples
layout: post
views:
  - 466
categories:
  - coding
tags:
  - extension
  - hash
  - laravel
---
<p style="text-indent: 2em;">
  在laravel中，默认的Auth使用的密码加密方式是Bcrypt散列密码，然后很多时候，我们想使用自己定义的加密方式，出于作业要求，我这次是要使用SHA256加密，当然，我们完全可以不使用Auth来自己实现，不过这里为了方便，我还是想直接使用Auth。
</p>

<p style="text-indent: 2em;">
  既然要使用Auth，那么我们就要对Auth模块进行扩展，在laravel的官方文档上，感觉讲的并不是很清楚，各种麻烦，于是最后自己查着资料研究了半天，得到了如下解决方案：
</p>

<!--more-->

<p style="text-indent: 2em;">
  首先是官网上说的最详细的，也是最麻烦的方法，我们可以写一个自己的实现了UserProviderInterface接口的UserProvider类，然后通过Auth::extend来注册我们的驱动，最后到app/config/auth.php修改配置即可，具体可以看<a href="http://v4.golaravel.com/docs/4.2/extending#authentication" target="_blank">官网</a>，这个方法还是说的比较详细的。
</p>

<p style="text-indent: 2em;">
  然后第二个方案是我正在采用的方案，通过研究laravel的框架代码发现，在AuthManager的createEloquentProvider中，我们可以发现如下代码：
</p>

<pre class="brush:php;toolbar:false">/**
	&nbsp;*&nbsp;Create&nbsp;an&nbsp;instance&nbsp;of&nbsp;the&nbsp;Eloquent&nbsp;user&nbsp;provider.
	&nbsp;*
	&nbsp;*&nbsp;@return&nbsp;IlluminateAuthEloquentUserProvider
	&nbsp;*/
	protected&nbsp;function&nbsp;createEloquentProvider()
	{
		$model&nbsp;=&nbsp;$this-&gt;app[&#39;config&#39;][&#39;auth.model&#39;];

		return&nbsp;new&nbsp;EloquentUserProvider($this-&gt;app[&#39;hash&#39;],&nbsp;$model);
	}</pre>

<p style="text-indent: 2em;">
  很显然，就是在这里选择了hash算法，那么我们将最后这句话的参数替换成我们想要的hash算法就可以了。
</p>

<p style="text-indent: 2em;">
  这里的app['hash']里面是一个BcryptHasher，于是我们可以仿照BcryptHasher写出一个Sha256Hasher：
</p>

<pre class="brush:php;toolbar:false">use&nbsp;IlluminateHashingHasherInterface;

class&nbsp;Sha256Hasher&nbsp;implements&nbsp;HasherInterface&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;Hash&nbsp;the&nbsp;given&nbsp;value.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;string&nbsp;&nbsp;$value
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;array&nbsp;&nbsp;&nbsp;$options
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@return&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@throws&nbsp;RuntimeException
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;function&nbsp;make($value,&nbsp;array&nbsp;$options&nbsp;=&nbsp;array())
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$hash&nbsp;=&nbsp;hash(&#39;sha256&#39;,&nbsp;$value);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$hash;
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;Check&nbsp;the&nbsp;given&nbsp;plain&nbsp;value&nbsp;against&nbsp;a&nbsp;hash.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;string&nbsp;&nbsp;$value
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;string&nbsp;&nbsp;$hashedValue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;array&nbsp;&nbsp;&nbsp;$options
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@return&nbsp;bool
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;function&nbsp;check($value,&nbsp;$hashedValue,&nbsp;array&nbsp;$options&nbsp;=&nbsp;array())
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;$this-&gt;make($value)&nbsp;===&nbsp;$hashedValue;
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;Check&nbsp;if&nbsp;the&nbsp;given&nbsp;hash&nbsp;has&nbsp;been&nbsp;hashed&nbsp;using&nbsp;the&nbsp;given&nbsp;options.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;string&nbsp;&nbsp;$hashedValue
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@param&nbsp;&nbsp;array&nbsp;&nbsp;&nbsp;$options
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*&nbsp;@return&nbsp;bool
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*/
&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;function&nbsp;needsRehash($hashedValue,&nbsp;array&nbsp;$options&nbsp;=&nbsp;array())
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;false;
&nbsp;&nbsp;&nbsp;&nbsp;}

}</pre>

<p style="text-indent: 2em;">
  然后我们还是采用extend的方式添加自己的驱动：
</p>

<pre class="brush:php;toolbar:false">Auth::extend(&#39;sha256&#39;,&nbsp;function()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;new&nbsp;IlluminateAuthEloquentUserProvider(new&nbsp;ExtensionsHashingSha256Hasher(),&nbsp;&#39;SecurityModelsUser&#39;);
});</pre>

<p style="text-indent: 2em;">
  或者也可以不用Eloquent，直接使用database：
</p>

<pre class="brush:php;toolbar:false">Auth::extend(&#39;sha256&#39;,&nbsp;function()&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;$connection&nbsp;=&nbsp;DB::connection();

&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;When&nbsp;using&nbsp;the&nbsp;basic&nbsp;database&nbsp;user&nbsp;provider,&nbsp;we&nbsp;need&nbsp;to&nbsp;inject&nbsp;the&nbsp;table&nbsp;we
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;want&nbsp;to&nbsp;use,&nbsp;since&nbsp;this&nbsp;is&nbsp;not&nbsp;an&nbsp;Eloquent&nbsp;model&nbsp;we&nbsp;will&nbsp;have&nbsp;no&nbsp;way&nbsp;to
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;know&nbsp;without&nbsp;telling&nbsp;the&nbsp;provider,&nbsp;so&nbsp;we&#39;ll&nbsp;inject&nbsp;the&nbsp;config&nbsp;value.
&nbsp;&nbsp;&nbsp;&nbsp;$table&nbsp;=&nbsp;Config::get(&#39;auth.table&#39;);

&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;new&nbsp;IlluminateAuthDatabaseUserProvider($connection,&nbsp;new&nbsp;ExtensionsHashingSha256Hasher(),&nbsp;$table);
});</pre>

<p style="text-indent: 2em;">
  最后，我们还是在<span style="text-indent: 32px;">app/config/auth.php修改配置为sha256即可。</span>
</p>

<p style="text-indent: 2em;">
  第三种方案应该算是官方文档中的基于IoC的扩展，和第二个有点类似，都是要实现一个<span style="text-indent: 32px;">Sha256Hasher，不过接下来要做的是就完全不一样了。</span>
</p>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">首先再添加一个<span style="text-indent: 32px;">Sha256HashServiceProvider</span>，继承IlluminateSupportServiceProvider，以将<span style="text-indent: 32px;">Sha256Hasher注册为hash组件，也就是将上面所提到的<span style="text-indent: 32px;">app['hash']直接给替换掉，<span style="text-indent: 32px;">Sha256HashServiceProvider可以仿照HashServiceProvider来实现，代码如下：</span></span></span></span>
</p>

<pre class="brush:php;toolbar:false">use&nbsp;IlluminateSupportServiceProvider;

class&nbsp;Sha256HashServiceProvider&nbsp;extends&nbsp;ServiceProvider&nbsp;{

	/**
	&nbsp;*&nbsp;Indicates&nbsp;if&nbsp;loading&nbsp;of&nbsp;the&nbsp;provider&nbsp;is&nbsp;deferred.
	&nbsp;*
	&nbsp;*&nbsp;@var&nbsp;bool
	&nbsp;*/
	protected&nbsp;$defer&nbsp;=&nbsp;true;

	/**
	&nbsp;*&nbsp;Register&nbsp;the&nbsp;service&nbsp;provider.
	&nbsp;*
	&nbsp;*&nbsp;@return&nbsp;void
	&nbsp;*/
	public&nbsp;function&nbsp;register()
	{
		$this-&gt;app-&gt;bindShared(&#39;hash&#39;,&nbsp;function()&nbsp;{&nbsp;return&nbsp;new&nbsp;Sha256Hasher;&nbsp;});
	}

	/**
	&nbsp;*&nbsp;Get&nbsp;the&nbsp;services&nbsp;provided&nbsp;by&nbsp;the&nbsp;provider.
	&nbsp;*
	&nbsp;*&nbsp;@return&nbsp;array
	&nbsp;*/
	public&nbsp;function&nbsp;provides()
	{
		return&nbsp;array(&#39;hash&#39;);
	}

}</pre>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;">最后，我们修改app/config/app.php中的providers，将其中下面一行删除：</span>
</p>

<pre class="brush:php;toolbar:false">&#39;IlluminateHashingHashServiceProvider&#39;,</pre>

<p style="text-indent: 2em;">
  <span style="text-indent: 32px;"></span>替换成自己刚实现的Sha256HashServiceProvider，即完成了散列方式的切换，这种方法意味着，laravel中所有使用到app['hash']来进行<span style="text-indent: 32px;">散列</span>的全都会被替换成我们自己的<span style="text-indent: 32px;">散列</span>方式，包括使用Hash::的时候。
</p>
