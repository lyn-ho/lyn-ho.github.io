---
title: 【译】JavaScript 工作原理：事件循环和 Async 编程的兴起和5种更好地使用 async/await 进行编码的方法
tags:
  - JavaScript
abbrlink: f7970ef
---

> 原文：[How JavaScript works: Event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

欢迎阅读致力于探索 JavaScript 及其构建模块的系列文章第4篇。在识别和描述核心元素的过程中，我们还将分享我们在构建 [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-post1-intro) 时使用的一些经验规则，SessionStack 是一个轻量级 JavaScript 应用，它稳定且性能强大以保持竞争力。

你错过了前三章吗？你可以在这里找到它们：

1. [An overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

2. [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

3. [Memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

这次我们的第一篇将通过回顾在单线程环境中编程的缺点以及如何克服它们来构建令人惊叹的 JavaScript UI 的文章。按照传统，在文章的最后，我们将分享有关如何使用 async/await 编写更清晰代码的5个技巧。

<!-- more -->

## 为什么单线程是一个限制？

在我们发布的[第一篇文章](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)中，我们思考了*在调用堆栈中进行函数调用时需要花费大量时间进行处理的问题*。

想象一下，例如一个在浏览器中运行的复杂图像变换算法。

虽然调用堆栈有函数需要执行，但浏览器无法执行任何其他操作 - 它被阻塞了。这意味着浏览器无法渲染，它无法运行任何其他代码，它只是卡住了。这就是问题所在 - 你的应用 UI 不再高效且令人满意。

你的应用**卡住了**。

在某些情况下，这可能不是一个至关重要的问题。但是这有一个更大的问题。一旦你的浏览器开始在调用堆栈中处理太多任务，它可能会在很长一段时间内停止响应。此时，许多浏览器会通过弹出错误来进行操作，询问是否应该终止页面：

这很丑陋，完全破坏了你的用户体验：

![page unresponsive](https://cdn-images-1.medium.com/max/1600/1*MCt4ZC0dMVhJsgo1u6lpYw.jpeg)

## JavaScript 程序的构建模块

你可能将 JavaScript 应用程序编写在单个 .js 文件中，但你的程序几乎肯定包含几个块，其中只有一个块*现在*将执行，其余块将在*稍后*执行。最常见的块单元是函数。

大多数刚接触JavaScript的开发人员似乎都有这样的问题，*之后*的代码并不一定会在*现在*的代码执行之后执行。换句话说，根据定义，现在无法完成的任务将异步完成，这意味着你不会像下意识地预期或希望的那样具有上述阻塞行为。

我们来看看下面的例子：

{% gist c0da11eef41070fc5d73be7852720f4b sample1.js %}

你可能知道标准Ajax请求不会同步完成，这意味着在代码执行时，`ajax()` 函数在没有任何返回值之前，是不会赋值给 `response` 变量的。

“等待”异步函数返回其结果的一种简单方法是使用一个名为 **callback** 的函数：

{% gist c0da11eef41070fc5d73be7852720f4b sample2.js %}

请注意：你实际上可以执行**同步** Ajax 请求。但永远不要这么做。如果你发出同步 Ajax 请求，你的 JavaScript 应用程序的 UI 将被阻塞 - 用户将无法单击，输入数据，导航或滚动。这将阻止任何用户交互。这是一个可怕的做法。

这就是它的样子，但请不要这样做 - 不要破坏 web 应用：

{% gist c0da11eef41070fc5d73be7852720f4b sample3.js %}

我们使用 Ajax 请求作为示例。你可以异步执行任何代码块。

这可以使用 `setTimeout(callback, milliseconds)` 函数来完成。 `setTimeout` 函数的作用是设置稍后发生的事件（超时）。让我们来看看：

{% gist c0da11eef41070fc5d73be7852720f4b sample4.js %}

控制台中的输出如下：

```console
first
third
second
```

## 解析事件循环

我们将从一个奇怪的声明开始 - 尽管允许异步 JavaScript 代码（如我们刚刚讨论过的 `setTimeout`），直到 ES6， JavaScript 本身实际上从来没有任何直接的内置异步概念。JavaScript引擎从未做过任何超出在任何特定时刻执行一个程序块的事情。

有关JavaScript引擎如何工作的更多详细信息（特别是 Google 的 V8），请查看我们之前关于该[主题的文章之一](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)。

那么，谁告诉 JS 引擎执行你的程序块呢？实际上，JS 引擎不是独立运行的 - 它在宿主环境中运行，对于大多数开发人员来说，它是典型的 Web 浏览器或 Node.js 。实际上，现在，JavaScript 已经嵌入到各种设备中，从机器人到灯泡。每个设备代表 JS Engine 的不同类型的宿主环境。

所有环境中的共同点是一个称为**事件循环**的内置机制，它每次调用 JS 引擎时都会处理程序的多个块的执行。

这意味着 JS Engine 只是任意 JS 代码的按需执行环境。调度事件的周围环境（JS 代码执行）。

因此，例如，当你的 JavaScript 程序发出Ajax请求以从服务器获取某些数据时，你在函数中设置 `response` 代码（`callback`），并且 JS 引擎告诉宿主环境：“嘿，我现在要暂停执行，但是当你完成该网络请求，并且你有一些数据时，请调用此函数。”

然后浏览器设置对网络响应的监听，当它有某些东西返回给你时，它将通过将回调函数插入事件循环队列等待执行。

我们来看下面的图表：

![event loop](https://cdn-images-1.medium.com/max/1600/1*FA9NGxNB6-v1oI2qGEtlRQ.png)

你可以在我们之前的[文章](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)中阅读有关内存堆和调用栈的更多信息。

什么是这些 Web API？实质上，它们是你无法访问的线程，你可以调用它们。它们是浏览器并行启动的一部分。如果你是 `Node.js` 开发人员，这些就是一些 `C++` API。

那么事件循环究竟是什么呢？

![event loop](https://cdn-images-1.medium.com/max/1600/1*KGBiAxjeD9JT2j6KDo0zUg.png)

**事件循环只有一个简单的工作 - 监视调用栈和回调队列。如果调用栈为空，它将从队列中获取第一个事件，并将其推送到调用堆栈，然后运行它。**

这种迭代在事件循环中称为 **tick** 。每个事件只是一个回调函数。

{% gist c0da11eef41070fc5d73be7852720f4b sample5.js %}

让我们“执行”这段代码，看看会发生什么：

1. 清空的状态。清空的浏览器控制台，以及空的调用栈。

![s1](https://cdn-images-1.medium.com/max/1600/1*9fbOuFXJHwhqa6ToCc_v2A.png)

2. 将 `console.log('Hi')` 添加到调用栈。

![s2](https://cdn-images-1.medium.com/max/1600/1*dvrghQCVQIZOfNC27Jrtlw.png)

1. 执行 `console.log('Hi')` 。

![s3](https://cdn-images-1.medium.com/max/1600/1*yn9Y4PXNP8XTz6mtCAzDZQ.png)

4. 将 `console.log('Hi')` 从调用栈移除。

![s4](https://cdn-images-1.medium.com/max/1600/1*iBedryNbqtixYTKviPC1tA.png)

5. 将 `setTimeout(function cb1() { ... })` 添加的调用栈。

![s5](https://cdn-images-1.medium.com/max/1600/1*HIn-BxIP38X6mF_65snMKg.png)

6. 执行 `setTimeout(function cb1() { ... })` 。浏览器创建了一个定时器(Web API 的一部分)，并且开始倒计时。

![s6](https://cdn-images-1.medium.com/max/1600/1*vd3X2O_qRfqaEpW4AfZM4w.png)

7. `setTimeout(function cb1() { ... })` 自身完成执行并从调用栈中移除。

![s7](https://cdn-images-1.medium.com/max/1600/1*_nYLhoZPKD_HPhpJtQeErA.png)

8. 将 `console.log('Bye')` 添加到调用栈。

![s8](https://cdn-images-1.medium.com/max/1600/1*1NAeDnEv6DWFewX_C-L8mg.png)

9. 执行 `console.log('Bye')` 。

![s9](https://cdn-images-1.medium.com/max/1600/1*UwtM7DmK1BmlBOUUYEopGQ.png)

10. 将 `console.log('Bye')` 从调用栈移除。

![s10](https://cdn-images-1.medium.com/max/1600/1*-vHNuJsJVXvqq5dLHPt7cQ.png)

11. 在至少 5,000 毫秒后， 定时器结束并且将 `cb1` 压入回调队列。

![s11](https://cdn-images-1.medium.com/max/1600/1*eOj6NVwGI2N78onh6CuCbA.png)

12. 事件循环从回调队列获取 `cb1` 并且将它压入调用栈。

![s12](https://cdn-images-1.medium.com/max/1600/1*jQMQ9BEKPycs2wFC233aNg.png)

13. 执行 `cb1` 并将 `console.log('cb1')` 添加到调用栈。

![s13](https://cdn-images-1.medium.com/max/1600/1*hpyVeL1zsaeHaqS7mU4Qfw.png)

14. 执行 `console.log('cb1')` 。

![s14](https://cdn-images-1.medium.com/max/1600/1*lvOtCg75ObmUTOxIS6anEQ.png)

15. 将 `console.log('cb1')` 从调用栈移除。

![s15](https://cdn-images-1.medium.com/max/1600/1*Jyyot22aRkKMF3LN1bgE-w.png)

16. `cb1` 已经从调用栈移除。

![s16](https://cdn-images-1.medium.com/max/1600/1*t2Btfb_tBbBxTvyVgKX0Qg.png)

快速回顾一下：

![recap](https://cdn-images-1.medium.com/max/1600/1*TozSrkk92l8ho6d8JxqF_w.gif)


值得注意的是，ES6 指定了事件循环应该如何工作，这意味着技术上它在 JS 引擎的职责范围内，而不再仅仅扮演宿主环境角色。

这种变化的一个主要原因是在 ES6 中引入了 Promise，因为后者需要对事件循环队列的调度操作进行直接，细粒度的控制（稍后我们将更详细地讨论它们）。

## setTimeout(...) 是如何工作的

值得注意的是， `setTimeout(...)` 不会自动将回调放在事件循环队列中。它设置了一个计时器。当计时器到期时，环境会将你的回调放入事件循环中，以便将来的某个 tick 会选择它并执行它。看看这段代码：

{% gist c0da11eef41070fc5d73be7852720f4b sample6.js %}

这并不意味着 `myCallback` 将在 1,000 毫秒内执行，而是在 1,000 毫秒内， `myCallback` 将被添加到队列中。但是，队列中可能还有其他先前添加的事件 - 你的回调必须等待。

有很多文章或教程在介绍异步代码的时候都会从 `setTimeout(callback, 0)` 开始。那么，现在你知道事件循环的作用以及 setTimeout 的工作原理：调用 setTimeout 并将0作为第二个参数，只需将回调推迟到调用堆栈为空之前。

看一下下面的代码：

{% gist c0da11eef41070fc5d73be7852720f4b sample7.js %}

虽然等待时间设置为 0 毫秒，但浏览器控制台中的结果如下：

```console
Hi
Bye
callback
```

## ES6 中的作业(Jobs)是什么？

ES6中引入了一个名为“作业队列(Job Queue)”的新概念。它在事件循环队列顶层。在处理 Promises 的异步行为时，你最有可能遇到它（我们也将讨论它们）。

我们现在只讨论这个概念，以便在我们讨论 Promise 的异步行为时，你将了解这些操作是如何进行调度和处理的。

想象一下：作业队列是一个附加到事件循环队列中每个 tick 结束的队列。。在事件循环队列的一个 tick 期间可能会发生某些异步操作，这不会导致把一整个新事件添加到事件循环队列中，而是会在当前 tick 的作业队列的末尾添加一项(也就是作业)。

这意味着你可以添加另一个稍后要执行的函数，并且你可以放心，它将在这之后其他任何操作之前立即执行。

作业还可以使更多作业添加到同一队列的末尾。从理论上讲，作业“循环”（一个不断添加其他作业的作业等）可能会无限循环，从而使程序缺乏继续下一个事件循环 tick 所需的必要资源。从概念上讲，这类似于在代码中长时间运行或无限循环（如 `while(true)` ..）。

作业有点像 `setTimeout(callback, 0)` 的 “hack”，但是实现的方式是它们引入了更加明确且有保证的顺序：稍后执行，但是要尽快执行。

## 回调

如你所知，回调是目前在 JavaScript 程序中表达和管理异步的最常用方法。实际上，回调是 JavaScript 语言中最基本的异步模式。无数的 JS 程序，甚至是非常复杂的程序，都是使用回调作为异步的基础。

回调不是没有缺点。许多开发人员正试图找到更好的异步模式。但是，如果你不明白实际情况，那么就不可能有效地使用任何抽象。

在下一章中，我们将深入探讨这些抽象中的几个，以说明为什么更复杂的异步模式（将在后续帖子中讨论）是必要的，甚至是推荐的。

## 嵌套回调

看下面的代码：

{% gist c0da11eef41070fc5d73be7852720f4b sample8.js %}

我们有一组三个嵌套在一起的函数，每个函数代表异步系列中的一个步骤。

这种代码通常被称为“回调地狱”。但是“回调地狱(callback hell)”实际上几乎与嵌套/缩进无关。这是一个比这更深刻的问题。

首先，我们正在等待 “click” 事件，然后我们正在等待计时器触发，然后我们等待 Ajax 响应返回，此时它可能会再次重复。

乍一看，这段代码似乎可以自然地将其异步映射到同步步骤，如：

{% gist c0da11eef41070fc5d73be7852720f4b sample9.js %}

然后：

{% gist c0da11eef41070fc5d73be7852720f4b sample10.js %}

再然后：

{% gist c0da11eef41070fc5d73be7852720f4b sample11.js %}

最终：

{% gist c0da11eef41070fc5d73be7852720f4b sample12.js %}

所以，这种表达异步代码的同步方式似乎更自然，不是吗？一定有这样的方式吧？

## Promises

看一下下面的代码：

{% gist c0da11eef41070fc5d73be7852720f4b sample13.js %}

这非常简单：将 `x` 和 `y` 的值相加并将其打印到控制台。但是，如果 `x` 或 `y` 的值缺失并且仍有待确定怎么办？比如说，我们需要从服务器中检索 `x` 和 `y` 的值，然后才能在表达式中使用它们。假设我们有一个函数 `loadX` 和 `loadY` ，分别从服务器加载 `x` 和 `y` 的值。然后，假设我们有一个函数 `sum` ，一旦加载了 `x` 和 `y` 的值就对它们求和。

它可能看起来像这样（非常难看，不是吗）：

{% gist c0da11eef41070fc5d73be7852720f4b sample14.js %}

这里有一些非常重要的东西 - 在那个代码片段中，我们将 `x` 和 `y` 视为**未来值**，并且我们表达了一个操作 `sum(...)` （来自外部）并不关心 `x` 或 `y` 或两者是否立即可用。

当然，这种基于回调的粗略方法还有很多不足之处。这只是了解推理*未来值*的好处的第一个小步骤，而不必担心它们何时可用。

## Promise 值

让我们简单地看一下我们如何使用 Promises 表达 `x + y` 示例：

{% gist c0da11eef41070fc5d73be7852720f4b sample15.js %}

这代码段中有两层 Promise。

`fetchX()` 和 `fetchY()` 被直接调用，它们返回的值（ promises！）被传递给 `sum(...)`。这些 promises 代表的值可能在*现在*或是*将来*准备好，但每个 promise 的自身规范都是相同的。我们以一种与时间无关的方式来解释 `x` 和 `y` 的值。它们在一段时间内是*未来值*。

第二层 promise 是 `sum(...)` 创建（通过 `Promise.all([...])` ）并返回， 等待调用 `then(...)` 。 当 `sum(...)` 操作完成，sum 的*未来值*准备好用于打印输出。我们将等待 `x` 和 `y` *未来值* 的逻辑隐藏在 `sum(...)` 之中。

**注意**：在 `sum(...)` 里面，调用 `Promise.all([...])` 创建了一个 promise (等待 `promiseX` 和 `promiseY` 完成)。链式调用 `.then(...)` 创建另一个 promise ，返回 `values[0] + values[1]` 并立即执行完成（加运算的结果）。在调用 `sum(...)` 的链尾调用 `then(...)` —— 代码片段的末尾 —— 实际在第二个 promise 返回后执行，而不是第一个通过 `Promise.all([...])` 创建的 promise 之后。此外，尽管我们没有在第二个 `then(...)` 后面再进行链式调用，但是它也创建了一个 promise，我们可以去观察或是使用它。关于 Promise 的链式调用会在后面详细地解释。

使用 Promises，`then(...)` 实际上有两个函数参数，第一个用于 fulfillment （如前所示），第二个 rejection ：

{% gist c0da11eef41070fc5d73be7852720f4b sample16.js %}

如果在获取 `x` 或 `y` 时出现问题，或者在加法运算过程中失败了，则 `sum(...)` 返回的 promise 将被拒绝（reject），并且将 promise 的值传入 `then(...)` 的第二个回调函数。

因为 Promises 封装了时间依赖状态 —— 等待内部的值已完成或是已失败  —— 从外面看，Promise 本身是与时间无关的，因此无论下面的时间或结果如何，Promise 都可以以可预测的方式组合（composed or combined）。

而且，一旦 Promise resolved ，它就会永远保持这种状态 —— 它在那时变成一个*不可变值* —— 然后可以根据需要多次*观察*。

实际上 promises 链是非常有用的：

{% gist c0da11eef41070fc5d73be7852720f4b sample17.js %}

调用 `delay(2000)` 会创建一个在 2000ms 完成的 promise，然后我们返回第一个 `then(...)` 的成功回调，这会导致第二个 `then(...)` 的 promise 要再等待 2000ms 执行。

**注意：**因为 Promise 一旦完成就是外部不可变的，现在可以安全地将该值传递给任何一方，因为它知道它不会被意外或恶意地修改。这对于在多个地方监听 Promise 的解决方案来说，尤其正确。一方不可能影响到另一方所监听到的结果。不变性可能听起来像一个学术主题，但它实际上是 Promise 设计中最基本和最重要的方面之一，不应该被忽略。

## 使用或不使用 Promise ？

关于 Promises 的一个重要细节是确定某些值是否是真正的 Promise 。换句话说，它是一个表现得像 Promise 的值吗？

我们知道 Promises 是由 `new Promise(...)` 语句构建，并且你认为 `p instanceof Promise` 为真，但是，并不一定。

主要是因为你可以从另一个浏览器 window （例如 iframe）接收 Promise 值，该窗口具有自己的 Promise 类，与当前 window 或 frame 中的 Promise 不同，并且该检查将无法识别 Promise 实例。

此外，库或框架可以选择使用自己的 Promises ，而不是使用原生 ES6 Promise 的实现。事实上，你很可能在没有 Promise 的旧浏览器中通过库使用 Promises 。

## 吞噬异常

如果在任何一个创建 Promise 或是对其结果观察的过程中，抛出了一个 JavaScript 异常错误，比如说 `TypeError` 或是 `ReferenceError` ，那么这个异常会被捕获，然后它就会把 Promise 的状态变成已失败。

例如：

{% gist c0da11eef41070fc5d73be7852720f4b sample18.js %}

如果一个 Promise 已经结束了，但是在监听结果(在 `then(…)` 里的回调函数)的时候发生了 JS 异常会怎么样呢？即使这个错误没有丢失，你可能也会对它的处理方式有点惊讶。除非你深入的挖掘一下：

{% gist c0da11eef41070fc5d73be7852720f4b sample19.js %}

看起来 `foo.bar()` 的异常确实被吞噬了。但事实并非如此。有一些更深层次的错误，我们没有监听到。调用 `p.then(…)` 它自己会返回另一个 promise，而这个 promise 会因为 `TypeError` 的异常变为已失败状态。

## 处理未捕获的异常

还有许多人会说其他更好的方法。

最常见的就是给 Promise 加一个 `done(…)` ，用来标志 Promise 链的结束。 `done(…)` 不会创建或返回一个 Promise，所以传给 `done(...)` 的回调显然不会将问题报告给一个不存在的 Promise。

在未捕获异常的情况下，这可能才是你期望的：在 `done(...)` 已失败的处理函数里的任何异常都会抛出一个全局的未捕获异常（通常是在开发者的控制台）。

{% gist c0da11eef41070fc5d73be7852720f4b sample20.js %}

## ES8 中发生了什么？ Async/await

JavaScript ES8 推出了 `async/await` ，使得可以更容易的使用 Promises 。我们将简要介绍 `async/await` 提供的能力以及如何利用它们编写异步代码。

那么，让我们看看 `async/await` 是如何工作的。

你可以使用 `async` 函数声明定义异步函数。这个函数返回 [AsyncFunction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 对象。`AsyncFunction` 对象是指执行的代码中包含异步函数。

当调用一个 `async` 函数，它返回一个 `Promise` 。当 `async` 函数返回一个值，这个值不是 `Promise` ，而是自动创建一个 `Promise` 并且使用函数返回值作为 resolved 状态。当 `async` 函数抛出一个异常，这个抛出的值作为 `Promise` rejected 状态。

一个 `async` 函数可以包含一个 `await` 表达式，它会暂停函数的执行并等待传递的 `Promise` 的完成，然后恢复异步函数的执行并返回的 resolved 值。

你可以将 JavaScript 中的 `Promise` 视为 Java 的 `Future` 或 C＃ 的 Task 。

> `async/await` 的目的是简化使用 promises 的写法。

我们来看看下面的例子：

{% gist c0da11eef41070fc5d73be7852720f4b sample21.js %}

类似地，抛出异常的函数等同于返回已被拒绝的 promise 的函数：

{% gist c0da11eef41070fc5d73be7852720f4b sample22.js %}

`await` 关键字只能在 `async` 函数中使用，并允许你同步等待 Promise 。如果我们在 `async` 函数之外使用 promise ，我们仍然必须使用 `then` 回调：

{% gist c0da11eef41070fc5d73be7852720f4b sample23.js %}

你还可以使用“ `async` 函数表达式”定义 async 函数。 async 函数表达式与 async 函数声明非常相似并且具有几乎相同的语法。 async 函数表达式和 async 函数语句之间的主要区别是函数名称，可以在 async 函数表达式中省略该函数名称以创建匿名函数。 async 函数表达式可以用作IIFE（立即执行函数），它在定义后立即执行。

它看起来像这样：

{% gist c0da11eef41070fc5d73be7852720f4b sample24.js %}

更重要的是，所有主流浏览器都支持 async/await ：

![If this compatibility is not what you are after, there are also several JS transpilers like Babel and TypeScript.](https://cdn-images-1.medium.com/max/1600/0*z-A-JIe5OWFtgyd2.)

最后，重要的是不要盲目选择“最新”的方法编写异步代码。理解异步 JavaScript 的内部结构，了解它为何如此重要以及深入了解所选方法的内部结构至关重要。每种方法都有其优点和缺点，就像编程中的其他一切一样。

## 5 个编写高度可维护，非脆弱的异步代码的技巧

1. **Clean code**：使用 async/await 能够让你少写代码。每一次你使用 async/await 你都能跳过一些不必要的步骤：写一个 .then，创建一个匿名函数来处理响应，在回调中命名响应，比如：

{% gist c0da11eef41070fc5d73be7852720f4b sample25.js %}

对比:

{% gist c0da11eef41070fc5d73be7852720f4b sample26.js %}

2. **Error handling** ：Async/await 使得我们可以使用相同的代码结构处理同步或者异步的错误 —— 著名的 try/catch 语句。让我们看看用 Promises 是怎么实现的：

{% gist c0da11eef41070fc5d73be7852720f4b sample27.js %}

对比:

{% gist c0da11eef41070fc5d73be7852720f4b sample28.js %}

3. **Conditionals**：使用 `async/await` 来写条件语句要简单得多：

{% gist c0da11eef41070fc5d73be7852720f4b sample29.js %}

对比：

{% gist c0da11eef41070fc5d73be7852720f4b sample30.js %}

4. **Stack Frames**：和 `async/await` 不同的是，根据 promise 链返回的错误堆栈信息，并不能发现哪出错了。来看看下面的代码：

{% gist c0da11eef41070fc5d73be7852720f4b sample31.js %}

对比：

{% gist c0da11eef41070fc5d73be7852720f4b sample32.js %}

5. **Debugging**：如果你使用过 promises，你知道调试它们是一场噩梦。比如，你在 `.then` 里面打了一个断点，并且使用类似 “stop-over” 这样的 debug 快捷方式，调试器不会移动到下一个 .then，因为它只会对同步代码生效。而通过 `async/await` 你就可以逐步的调试 await 调用了，它就像是一个同步函数一样。

编写**异步 JavaScript 代码**不仅对应用程序本身很重要，对于库也很重要。

## 资源

* [You Don't Know JS: Async & Performance | Chapter 2: Callbacks](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch2.md)
* [You Don't Know JS: Async & Performance | Chapter 3: Promises](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md)
* [Await and Async Explained with Diagrams and Examples](https://nikgrozev.com/2017/10/01/async-await/)







