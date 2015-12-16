---
title: async学习笔记
author: Ripples
layout: post
views:
  - 387
categories:
  - coding
tags:
  - async
  - javascript
---
<p style="text-indent: 2em;">
  异步库对于js来说，重要性不用多说，这次刚开始选的时候，按照大家的意见，选了async，于是便把async学了下，本来打算把整个都看了再发出来，所以一直留着了，不过现在想想，还是直接发了算了，并不是所有的都会去用，反正后面用到了没有的再去看，然后加上来。
</p>

<p style="text-indent: 2em;">
  下面就按照函数一个个的列了：
</p>

<!--more-->

<p style="text-indent: 2em;">
</p>

<pre class="brush:js;toolbar:false">async.nextTick(function())</pre>

<p style="text-indent: 2em;">
  与nodejs的process.nextTick一样
</p>



<p style="text-indent: 2em;">
  <strong>然后接下来的函数都满足下面几点：<br /></strong>
</p>

<ul class=" list-paddingleft-2" style="list-style-type: disc;">
  <li>
    <p style="text-indent: 0em;">
      并行的可以不调用callback，串行的必须调用callback来触发后续任务的启动；
    </p>
  </li>

  <li>
    <p style="text-indent: 0em;">
      串行执行发生错误时，未启动任务忽略，并行执行发生错误时，未启动任务继续启动；
    </p>
  </li>

  <li>
    <p style="text-indent: 0em;">
      并行度有限的并行相当于是在并行外层串行，故而也必须调用callback来触发后续任务的启动，执行发生错误时，未启动任务忽略，已启动任务继续执行；
    </p>
  </li>

  <li>
    <p style="text-indent: 0em;">
      错误发生时，会立即执行callback，此时传递的results可能是不完整的，callback执行完后，其它任务仍会继续修改results；
    </p>

    <p style="text-indent: 2em;">
    </p>
  </li>
</ul>

<pre class="brush:js;toolbar:false">async.each(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(err));</pre>

<p style="text-indent: 2em;">
  对集合中元素依次调用callback
</p>

<p style="text-indent: 2em;">
  同步，并行执行
</p>

<pre class="brush:js;toolbar:false">async.eachSeries(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(err));</pre>

<p style="text-indent: 2em;">
  对集合中元素依次调用callback
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.eachLimit(arr,&nbsp;limit,&nbsp;function(item,&nbsp;callback),&nbsp;callback(err));</pre>

<p style="text-indent: 2em;">
  对集合中元素依次调用callback
</p>

<p style="text-indent: 2em;">
  同步，并行执行（并行度有限），发生错误时，未启动任务忽略，已启动任务继续执行
</p>



<pre class="brush:js;toolbar:false">async.every(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(result));</pre>

<p style="text-indent: 2em;">
  判断是否每个元素都满足条件
</p>

<p style="text-indent: 2em;">
  同步，并行执行
</p>

<pre class="brush:js;toolbar:false">async.some(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(result));</pre>

<p style="text-indent: 2em;">
  判断是否有元素满足条件
</p>

<p style="text-indent: 2em;">
  同步，并行执行
</p>



<pre class="brush:js;toolbar:false">async.concat(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(err,&nbsp;results))</pre>

<p style="text-indent: 2em;">
  对集合中元素依次调用callback，并将结果合并为一个数组
</p>

<p style="text-indent: 2em;">
  同步，并行执行
</p>

<pre class="brush:js;toolbar:false">async.concatSeries(arr,&nbsp;function(item,&nbsp;callback),&nbsp;callback(err,&nbsp;results))</pre>

<p style="text-indent: 2em;">
  对集合中元素依次调用callback，并将结果合并为一个数组
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>



<pre class="brush:js;toolbar:false">async.detect(arr,&nbsp;function(item,callback),&nbsp;function(result))</pre>

<p style="text-indent: 2em;">
  用于取得集合中满足条件的第一个元素
</p>

<p style="text-indent: 2em;">
  同步，并行执行
</p>

<pre class="brush:js;toolbar:false">async.detectSeries(arr,&nbsp;function(item,callback),&nbsp;function(result))</pre>

<p style="text-indent: 2em;">
  用于取得集合中满足条件的第一个元素
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>



<pre class="brush:js;toolbar:false">async.whilst(function(),&nbsp;function(callback),&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  相当于while，第一个参数是判断条件，第二个参数是要做的异步工作
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.doWhilst(function(callback),&nbsp;function(),&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  相当于do..while，第一个参数是要做的异步工作，第二个参数是判断条件
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.until(function(),&nbsp;function(callback),&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  与while相反，判断条件为true时跳出循环，第一个参数是判断条件，第二个参数是要做的异步工作
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.doUntil(function(callback),&nbsp;function(),&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  与do..while相反，判断条件为true时跳出循环，，第一个参数是要做的异步工作，第二个参数是判断条件
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.forever(function(callback),&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  无限循环，直至出错
</p>

<p style="text-indent: 2em;">
  同步，串行执行
</p>



<pre class="brush:js;toolbar:false">async.waterfall([function(result,&nbsp;...,&nbsp;callback)],&nbsp;callback(err,&nbsp;result))</pre>

<p style="text-indent: 2em;">
  按顺序依次执行一组函数，每个函数产生的值，都将传给下一个
</p>

<p style="text-indent: 2em;">
  异步，串行执行
</p>

<pre class="brush:js;toolbar:false">async.series([function(callback)],&nbsp;callback(err,&nbsp;results))</pre>

<p style="text-indent: 2em;">
  依次执行一组函数
</p>

<p style="text-indent: 2em;">
  异步，串行执行，支持关联数组（即对象）格式
</p>

<pre class="brush:js;toolbar:false">async.parallel([function(callback)],&nbsp;callback(err,&nbsp;results))</pre>

<p style="text-indent: 2em;">
  依次执行一组函数
</p>

<p style="text-indent: 2em;">
  异步，并行执行，支持关联数组（即对象）格式
</p>



<p style="text-indent: 2em;">
  <strong>接下来的函数不再是只会调用一次最后的callback，而是分别调用callback，故而任何任务出错不影响其它任务的执行：</strong>
</p>



<p style="text-indent: 2em;">
  对集合中元素依次调用callback
</p>

<p style="text-indent: 2em;">
  异步，串行执行
</p>

<pre class="brush:js;toolbar:false">queue&nbsp;=&nbsp;async.queue(function(task,&nbsp;callback),&nbsp;concurrency)</pre>

<p style="text-indent: 2em;">
  定义queue
</p>

<pre class="brush:js;toolbar:false">queue.saturated&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：如果某次push操作后，任务数将达到或超过worker数量时，将调用该函数
</p>

<pre class="brush:js;toolbar:false">queue.empty&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：当最后一个任务交给worker时，将调用该函数
</p>

<pre class="brush:js;toolbar:false">queue.drain&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：当所有任务都执行完以后，将调用该函数
</p>

<pre class="brush:js;toolbar:false">queue.push(task,&nbsp;callback(err))
queue.push([task],&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  加入任务
</p>



<p style="text-indent: 2em;">
  分组执行，对集合中每组元素依次调用callback
</p>

<p style="text-indent: 2em;">
  异步，串行执行
</p>

<pre class="brush:js;toolbar:false">cargo&nbsp;=&nbsp;async.cargo(function(tasks,&nbsp;callback),&nbsp;concurrency)</pre>

<p style="text-indent: 2em;">
  定义cargo
</p>

<pre class="brush:js;toolbar:false">cargo.saturated&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：如果某次push操作后，任务数将达到或超过worker数量时，将调用该函数
</p>

<pre class="brush:js;toolbar:false">cargo.empty&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：当最后一个任务交给worker时，将调用该函数
</p>

<pre class="brush:js;toolbar:false">cargo.drain&nbsp;=&nbsp;function()</pre>

<p style="text-indent: 2em;">
  监听：当所有任务都执行完以后，将调用该函数
</p>

<pre class="brush:js;toolbar:false">cargo.push(task,&nbsp;callback(err))
cargo.push([task],&nbsp;callback(err))</pre>

<p style="text-indent: 2em;">
  加入任务
</p>

<p style="text-indent: 2em;">
</p>

<p style="text-indent: 2em;">
  不过最近又看了下promise后，真的是觉得我更喜欢promise的方式，promise感觉要比async灵活，复杂逻辑比async写出来要清楚，异常直接找链上下一个handler，避免写一堆把异常向上传的重复代码……
</p>
