---
layout: post
title: "[译]解析JavaScript的事件循环机制"
date: 2014-03-13 17:39:17 +0800
comments: true
category: javascript
---
[原文链接](http://blog.carbonfive.com/2013/10/27/the-javascript-event-loop-explained/)

目的
====
在web浏览器的世界中，JavaScript可以说是无处不在，也正因如此，大部分人对
JavaScript的事件驱动(event-driven)模型，以及它与Ruby，Python，
Java等语言所使用的请求-响应(request-response)模型的区别或多或少都有一些
基本的了解。我将在本文中阐述JavaScript并发(concurrency)模型的一些核心
概念，包括事件循环(event-loop)和消息队列(message queue)，以便帮你有更
深入的理解。<!--more-->


本文受众
========
本文主要面对的是在客户端或服务端使用JavaScript的web开发者。如果你已经非常
熟悉事件循环机制了，敬请拍砖。如果不是，我希望你能从本文有所收获。


非阻塞I/O
=========
在JavaScript中，几乎所有的I/O都是非阻塞(non-blocking)的。包括HTTP请求，
数据库操作，以及磁盘的读写。当执行这些I/O指令时，唯一的js执行线程会
请求runtime去做这些I/O，并提供一个callback函数，然后执行线程继续
压入执行后续的其它指令。当这个I/O操作被执行完后，runtime就会压入一条
包含对应callback的消息到执行线程的消息队列中。在后续的某个时刻，这条消息
被执行线程取出来并执行其中的callback函数。

这种模型对于用户界面的开发者而言再熟悉不过了。诸如`鼠标按下`,
`点击`等事件可以在任意时刻被触发。这与在服务端应用中典型的同步，
请求-相应模型有很大的区别。

我们来看一下这两种类型的区别。考虑HTTP请求www.google.com并输
出响应到控制台。首先上Ruby代码:

```ruby
response = Faraday.get 'http://www.google.com'
puts response
puts 'Done!'
```
执行步骤很简明：

1. 执行get方法，执行线程等待响应
2. get方法接受到Google的响应，并返回给调用者，保存到response变量中
3. 输出response变量到控制台
4. 输出`Done!`到控制台

而用JavaScript(Node.js)来实现:
```javascript
request('http://www.google.com', function(error, response, body) {
    console.log(body); });
console.log('Done!');
```

有点不一样吧，并且结果有很大区别：

1. 执行request函数，传递一个匿名函数作为回调
2. 立刻输出'Done!'到控制台
3. 后续某个时间获得Google的响应数据后，执行先前传递的回调(输出response到控制台)


事件循环
========
将调用与(I/O)响应的解耦允许JavaScript执行线程去执行其它的指令，而不用等待
I/O操作的完成及回调的执行。但这些callback被存放在内存的什么位置呢？它们以什么样
的顺序执行的呢？什么导致callback被执行的呢？

JavaScript的执行线程包含一个消息队列，用来存储将被处理的消息及其关联的callback
函数。每当产生一个事件(例如鼠标点击，收到HTTP响应)，并伴随着一个回调函数，都会生成
一个消息，并被压入队列。但如果产生事件时没有相应的回调函数提供，那么就不有消息压入队
列。(整个程序作为初始消息回调压入队列)

每次事件循环中(每循环一次，称为一个tick)，都会从消息队列中取一条消息，当取到一条
消息时，相应的回调函数就会被执行。

```javascript
function init() {
    var link = document.getElementById("foo");
    link.addEventListener("click", function changeColor() {
        this.style.color = "burlywook";
    });
}
```

{% img myimg https://erinswensonhealey.mybalsamiq.com/mockups/1228688.png?key=3f4ddd52e36d26e53ce8028ea38b5b64592e6b1a %}

(从图中1开始看)回调函数的调用作为call stack上的初始帧。由于JavaScript执行引擎
是单线程的，后续的消息提取和处理就会被暂停，直到当前stack上所有的调用都返回为止。

在这个例子当中，当用户点击`foo`标签时，`onclick`事件被触发了，这时，一个消息(包含它的回调函数changeColor)被压入到了消息队列中了。当这条消息被取出时，它的
回调函数changeColor将会被执行。当changeColor返回时(或抛出了error)，该消息
处理完毕，事件循环才能取下一条消息来处理。


执行消息队列中的回调函数时产生新的消息
====================================
如果一个异步函数(如setTimeout)被调用了，那么相应的回调函数最终会在后续某个事件
循环中作为消息的回调函数而被执行。看下面这个例子：
```javascript
function f(){
	console.log("foo");
	setTimeout(g, 0);
	console.log("baz");
	h();
}

function g() {
	console.log("bar");
}

function h() {
	console.log("blix");
}

f();
```

由于setTimeout函数的非阻塞特性，它的回调函数至少在未来0微秒后执行，并且在本次
事件循环处理消息过程中不执行。在这个例子中，调用了setTimeout，并传递回调函数g
以及0微秒的超时时间。当指定的超时时间满足以后(本例中几乎瞬时)，一个包含回调函数g
的消息将会被压人消息队列。最终控制台将会依次显示：`foo`，`baz`，`blix`。最后在
下一次事件循环处理消息(tick)的时候，打印出`bar`来。如果在同一个调用帧中有两次调
用setTimeout，且超时时间相同，那么这两个包含相应回调函数的异步消息将以调用的顺序
进入消息队列中。

异步函数(如setTimeout)很容易实现任务的延迟执行，而不需要spawn出新的线程。JavaScript
的异步函数通常只有两类：I/O和timing。


Web Workers
===========
通过使用Web Workers技术可以把一些费时的操作放到worker线程中去执行，从而分解主线
程的执行压力。worker线程包含一个独立的消息队列，事件循环以及内存空间。worker线程
与主线程之间是通过消息传递的方式进行通信的。

{% img myimg https://erinswensonhealey.mybalsamiq.com/mockups/1218698.png?key=3f4ddd52e36d26e53ce8028ea38b5b64592e6b1a %}

首先看一下worker线程的代码：
```javascript
// our worker, which does some CPU-intensive operation
var reportResult = function(e) {
	pi = SomeLib.computePiToSpecifiedDecimals(e.data);
	postMessage(pi);
}

onmessage = reportResult;
```

接着看一下主线程的调用：
```javascript
// our main code, in a `<script>` tag in our HTML page;
var piWorker = new Worker("pi_calculator.js");
var logResult = function(e) {
	console.log("PI: " + e.data);
};

piWorker.addEventListener("message", logResult, false);
piWorker.postMessage(100000);
```
在这个例子中，主线程spawn出一个worker线程，并且为其注册logResult
回调函数到`message`事件上。当worker线程接收到来自主线程的消息时，
worker线程将消息及回调函数logResult绑定在一起压入自身的消息队列中。
当worker消息从其消息队列取消息时，会向主线程回馈一个消息，同样绑定
回调函数logResult。通过这种方式，开发者就可以将CPU密集型的任务代理给
worker线程来执行，而不用阻塞主线程，从而主线程就能继续处理消息和事件了。


总结
====
JavaScript的事件驱动模型与许多程序员所熟悉的请求-响应模型是有很大区别
的，但一点也不高深莫测。只用简单的消息队列以及事件循环，JavaScript开发
者就可以使用大量的异步回调来构建它们的系统，让runtime在等待外部事件的发
生的时候，处理并发指令。但这并不是唯一一种处理并发的方式，在本系列的下一
篇文章中，我将比较一下JavaScript的并发模型与Ruby的MRI技术(通过多线程
和GIL)，Ruby的EventMachine技术及Java的多线程技术的并发模型。


相关阅读
========
* [The JavaScript Event Loop: Concurrency in the Language of the Web](https://docs.google.com/presentation/d/1KtgaIvDQwMaqZ6ax3zU2oka62sF2ZQSPv1SEirD-XtY/edit?usp=sharing)
* [Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/EventLoop) @ MDN
* [An intro to the Node.js platform](http://www.aaronstannard.com/post/2011/12/14/Intro-to-NodeJS-for-NET-Developers.aspx), by Aaron Stannard
