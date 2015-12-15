---
layout: post
title: scrapy 使用笔记
modified: 2015-12-15T23:45:17+08:00
categories: technology
description: 早就听闻 scrapy，前段时间入坑学了下，特此记录下使用过程的一点问题，以备将来不时之需。
tags: [scrapy, crawler]
---

前段时间时不时就会要去爬的东西，之前这种情况大多数时候都是直接 python 裸写了，反正就是临时下少量东西，所以没太大关系，
这次想了想，还是决定试用一下现成的框架，毕竟用框架很多轮子不用自己造，还不用担心 cookie、retry、多线程的事，好处多多，唯一就是有个学习成本而已。

长痛不如短痛，趁着现在不是这么急，于是投身了 scrapy 的怀抱。
很早就听闻了这个框架，作为 python 爬虫框架的代表，不得不说，用起来却是感觉是不错，不过使用中也遇到了点小问题，就此记录下，以备不时之需。

## 过滤重复 URL

作为爬虫框架的代表，scrapy 在去重上自然也是有考虑的，
在一次爬虫运行过程中，scrapy 会自动对请求的所有地址做去重处理，保证同一个 URL 不会被重复访问多次，
一方面是减小了无意义的开销，另一方面更是避免了超链接连成环导致死循环。

然而，这次要爬的某网站的设计比较奇葩：

某内容页面 URL 为 `http://domain/path/{id}`，这个 URL 本身没有什么问题，
但是这个 URL 访问之后其实是会设置一个 cookie，然后重定向到一个固定网址，然后那个网址会根据 cookie 来给出返回的内容。

这种设计不得不说真还是第一次见，当 scrapy 遇到这种设计的时候，
由于被重定向到的目标网址都是同一个 URL，而 scrapy 遇到重定向的时候实际上是重发一个新请求到新的目标地址，然后这时候去重的判断就会把重发的所有的请求全部认为是重复请求了。
这个问题本身其实并不难解决，但是由于对于重定向的请求去重的时候，scrapy 并没有给出任何日志提醒，所以导致最开始一直找不到原因。
知道原因后，其实很容易搜到，比如 stack overflow 上[这篇](http://stackoverflow.com/questions/18928253/why-is-my-scrapy-spider-not-following-the-request-callback-in-my-item-parse-func)。

解决方案也就是在构造 Request 对象时增加一个参数 `dont_filter=True`，从而对此请求禁用掉调度器中的过滤。

<p id="read-more-anchor"/>

## 返回状态码检查

默认情况下，scrapy 会认为返回状态码 2xx 是成功的 http 请求，而 301、302 会做重定向，
而我要爬的这个网站大概是做了个频率限制（抑或是它太渣？），当频率过高时会给个 302 重定向到 error 页面，而 error 页面给的是 200。
也就是说，当 error 出现的时候，scrapy 会 follow 302，然后认为请求成功了，
所以这里我们需要首先对这个请求禁掉 302 follow，然后将 302 加入成功的 http 请求状态码范围。

禁掉 302 的话，在构造 Request 的时候，meta 参数中需要增加一项 `'dont_redirect': True`，
然后设置成功请求码范围的话，在 stack overflow 上也可以搜到[这篇](http://stackoverflow.com/questions/9698372/scrapy-and-response-status-code-how-to-check-against-it)，
即在爬虫类中加入属性 `handle_httpstatus_list = [302]`。

## Ending

最后不得不说，用了框架之后，确实是很多事情上都方便了很多，
印象比较深的比如这个[自动频率限制](http://doc.scrapy.org/en/1.0/topics/autothrottle.html)确实很好使，开了之后就基本没被刚说的那个有频率限制的网站报错了。
不过由于对于框架的不熟悉，所以有时候很简单的一个问题可能会导致查起问题来很麻烦，
好在这代价大多数时候都还是可以忍受的。

虽然现在挺喜欢 scrapy，但不得不承认，遇到 js 动态生成的东西，目测就要歇菜了，大概还是得靠 phantomjs 这种东西了，
但 python 才是真爱啊，不到不得已，干嘛要用 js 去折腾自己呢！
