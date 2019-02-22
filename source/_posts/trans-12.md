---
title: 【译】JavaScript 工作原理：深入理解 WebSockets 和拥有 SSE 技术 的 HTTP/2，以及如何在二者中做出正确的选择
tags:
  - JavaScript
---

> 原文：[How JavaScript works: Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7)

欢迎阅读致力于探索 JavaScript 及其构建模块的系列文章第5篇。在识别和描述核心元素的过程中，我们还将分享我们在构建 [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-post1-intro) 时使用的一些经验规则，SessionStack 是一个轻量级 JavaScript 应用，它稳定且性能强大以保持竞争力。

如果你错过了前面的章节，你可以从这找到他们：

* [An overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

* [Memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

这一次，我们将深入了解通信协议的世界，去讨论和对比 WebSockets 和 HTTP/2 的属性和构成。我们将提供 WebSockets 和 HTTP/2 的快速比较。最后，我们分享了一些关于如何选择这两种网络协议的方法。

<!-- more -->

## 简介

如今，具有富功能以及动态 UI 的复杂 Web 应用程序被认为是理所当然的。互联网自成立以来已经走过了漫长的道路，这一点也不足为奇。

最初，互联网不是为支持这种动态和复杂的网络应用而构建的。它被设想为 HTML 页面的集合，页面之间彼此链接以形成信息载体的 “web” 概念。一切都主要围绕 HTTP 请求/响应（request/response） 范式构建。客户端加载了一个页面后将不会再发生任何事，除非用户点击并跳转到了下一页。

大约在2005年，AJAX 被引入，很多人开始探索在客户端和服务器之间建立**双向通信**的可能性。尽管如此，所有 HTTP 通信都由客户端引导，客户端需要用户交互或定期轮询以从服务器加载新数据。

## 让 HTTP “双向通信”

使服务器“主动”向客户端发送数据的技术已经存在了相当长的一段时间。例如 "Push" 和 "Comet"。

创建服务器向客户端发送数据的错觉的最常见的 hack 之一称为**长轮询(long polling)**。通过长轮询，客户端建立与服务器的 HTTP 连接，使其保持打开状态，直到发送响应为止。每当服务器具有必须发送的新数据时，它就将其作为响应发送。

让我们看看一个非常简单的长轮询代码片段如何：

{% gist c0da11eef41070fc5d73be7852720f4b long_polling.js %}

这是一个基本的自执行函数，会自动执行。它设置了 10000ms 的间隔，并且在每次异步 Ajax 调用服务器之后，回调函数再次调用 ajax。

