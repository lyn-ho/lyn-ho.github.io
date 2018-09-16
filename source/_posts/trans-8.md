---
title: 【译】JavaScript 工作原理：JS 引擎，runtime 和调用栈的概述
tags:
    - JavaScript
date: 2018-09-10 16:58:58
---

> 原文：[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

JavaScript 变得越来越流行，很多团队在他们技术栈的多个方面使用它：前端，后端，hybrid 应用，嵌入式设备等等。

这篇是本系列文章的第一篇，旨在深入挖掘 JavaScript 和其实际工作原理：我们认为，通过了解 JavaScript 的构建块以及它们是如何协同工作的，你将能编写更好的代码和应用程序。我们还将分享我们在构建 [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-post1-intro) 时使用的一些经验规则，SessionStack 是一个轻量级 JavaScript 应用，它稳定且性能强大以保持竞争力。

<!-- more -->

根据 [GitHut stars](http://githut.info/) 显示，就 Active Repositories 和 Total Pushes 而言 JavaScript 在 GitHub 中是最高的。在其他类别中也没有落后很多。

![GitHut](https://cdn-images-1.medium.com/max/1600/1*Zf4reZZJ9DCKsXf5CSXghg.png)

[(Check out up-to-date GitHub language stats)](https://madnight.github.io/githut/)

如果项目越发的依赖于 JavaScript，这意味着为了构建令人惊叹的软件，开发人员必须利用 JS 语言和生态所提供的所有内容，对其内部深入理解。

事实证明，有很多开发者每天都在使用 JavaScript ，但却不了解其内部原理。

## 概述

几乎所有开发人员都知道 V8 引擎的概念，并且大多数开发人员了解 JavaScript 是单线程 (single-threaded) 的或者说它使用了回调队列 (callback queue)。

在这篇文章中，我们将详细介绍这些概念并解释 JavaScript 的实际运行方式。通过了解这些细节，你可以正确利用提供的 APIs 来编写更好的非阻塞的应用。

如果你是 JavaScript 萌新，这篇文章将会帮助你理解为什么 JavaScript 与其他编程语言相比如此“奇怪”。

如果你是一位资深 JavaScript 开发人员，希望这篇文章可以为你提供一些关于你每天使用的 JavaScript Runtime 实际工作原理的新见解。

## JavaScript 引擎

Google 的 V8 引擎是一个流行的 JavaScript 引擎。V8 引擎应用与 Chrome 和 Node.js 。下面是它的一个非常简化的视图：

![V8 engine](https://cdn-images-1.medium.com/max/1600/1*OnH_DlbNAPvB9KLxUCyMsA.png)

V8 引擎包含两个主要组件：

- 内存堆 —— 进行内存分配
- 调用栈 —— 代码执行/栈帧

## 运行时 (Runtime)

几乎所有 JavaScript 开发人员都使用过浏览器中的 APIs (e.g. `setTimeout`) 。但是这些 APIs 不是由 JavaScript 引擎提供的。

所以，它们从何而来？

事实正面现实有点复杂。

![APIs](https://cdn-images-1.medium.com/max/1600/1*4lHHyfEhVB0LnQ3HlhSs8g.png)

所以，我们有 JavaScript 引擎，但是实际上还有更多。我们有一些由浏览器提供的叫做 Web APIs 的东西，比如 DOM, AJAX, setTimeout 等等。

然后，我们还有 **事件循环 (event loop)** 和 **回调队列 (callback queue)** 。

## 调用栈 (Call Stack)

JavaScript 是一个单线程编程语言，这意味着它只有一个调用栈。因此在同一时间它执行一个任务。

调用栈是一种我们程序中的基本记录的数据结构。如果进入一个函数，我们将它放在调用栈的顶层。如果返回一个函数，我们将其从调用栈的顶层弹出。这是调用栈能做的所有事情。

让我们看一个示例。看下面的代码：

```js
function multiply(x, y) {
  return x * y
}

function printSquare(x) {
  var s = multiply(x, x)
  console.log(s)
}

printSquare(5)
```

当 JavaScript 引擎开始执行代码时，调用栈是空的。之后的步骤如下：

![Call Stack](https://cdn-images-1.medium.com/max/1600/1*Yp1KOt_UJ47HChmS9y7KXw.png)

调用栈的每个入口称为 **栈帧 (Stack Frame)** 。

当抛出异常可以看到堆栈追踪是如何构造的 —— 当发生异常时，它就是调用栈的状态。看下面的代码：

```js
function foo() {
  throw new Error('SessionStack will help you resolve crashes:')
}

function bar() {
  foo()
}

function start() {
  bar()
}

start()
```

如果在 Chrome 中执行 (假设是 foo.js 中的代码) ，将会生成下面的堆栈追踪：

![Stack Trace](https://cdn-images-1.medium.com/max/1600/1*T-W_ihvl-9rG4dn18kP3Qw.png)

"**堆栈溢出 (Blowing the stack)**" —— 当达到最大调用栈大小的时候发生。这很容易发生，特别是如果你使用递归但没有全面的测试。看下面的示例代码：

```js
function foo() {
  foo()
}

foo()
```

当 JavaScript 引擎开始执行这段代码，开始调用 `foo` 函数。但是在没有终止条件的情况下 `foo` 会递归的调用自己。所以每执行一步，就会把这个相同的函数一次又一次的添加到调用栈中。看起来像下面这样：

![Blowing the stack](https://cdn-images-1.medium.com/max/1600/1*AycFMDy9tlDmNoc5LXd9-g.png)

但是，在某一时刻，调用栈中的函数调用数量超过了调用栈的实际大小，浏览器通过抛出一个错误来结束它。如下所示：

![Maximum](https://cdn-images-1.medium.com/max/1600/1*e0nEd59RPKz9coyY8FX-uw.png)

在单线程上运行代码非常简单，因为你不需处理多线程环境中出现的复杂场景 —— 例如 死锁 (deadlocks) 。

但是单线程也有很大的限制。由于 JavaScript 只有一个调用栈，如果运行很慢会发生什么？

## 并发和事件循环 (Concurrency & Event Loop)

如果在调用栈中有函数需要花费大量时间才能处理，会发生什么？例如，假设你需要在浏览器中使用 JavaScript 进行一些复杂的图像转换。

你可能会问，为什么这是一个问题？问题是，当调用栈有函数要执行，浏览器实际不会做其他任何事 —— 浏览器被阻塞了。这意味着浏览器不能渲染，不能运行任何其他代码，它被卡住了。如果你想要在 app 中拥有流畅的 UI 体验，这将是个问题。

并且这不是唯一的问题。一旦你的浏览器开始在调用栈中处理如此多的任务，它可能在相当长的时间内停止响应。大多数浏览器会弹出一个错误，询问是否终止网页。

![Unresponsive](https://cdn-images-1.medium.com/max/1600/1*WlMXK3rs_scqKTRV41au7g.jpeg)

这并不是最好的用户体验，不是吗？

那么，如何在不阻塞 UI 和不弹出无响应的情况下执行复杂的代码？解决方法是 **异步回调 (asynchronous callbacks)** 。

这将在 "How JavaScript actually works" 第2篇中详细解释："[Inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)" 。

