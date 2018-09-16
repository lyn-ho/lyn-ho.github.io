---
title: 【译】精通 JavaScript： 什么是 Promise？
tags:
  - JavaScript
abbrlink: 6a904bf2
date: 2018-08-30 15:43:58
---

> 原文：[Master the JavaScript Interview: What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

![Photo by Kabun (CC BY NC SA 2.0)](https://cdn-images-1.medium.com/max/1600/1*agGENodMcD6hhwIFdqGwrw.jpeg)

> “精通 JavaScript 面试” 是一个系列的文章，旨在帮助面试者准备他们在申请中高级职位时可能遇到的常见问题。这些是我在现实面试中经常提出的问题。

## 什么是 Promise？

Promise 是一个对象，在未来的某个时刻产生一个值：已完成 (resolved) 的值或者是未完成的原因(e.g. 网络错误)。Promise 可能是下面三种状态中的一种：fulfilled，rejected，pending 。Promise 提供回调来处理 fulfilled 返回的值或者 rejection 的原因。

<!-- more -->

Promise 会立即执行，这意味着一旦 promise 构造函数被调用，promise 就会开始执行你给它的任何任务。如果你需要 promise 懒加载，参考 [observables](https://github.com/Reactive-Extensions/RxJS) 或者 [tasks](https://github.com/rpominov/fun-task)。

## Promise 的不完全历史

早在1980年代，promise 和 futures (类似/相关的想法) 的早期实现开始出现在 MultiLisp 和 Concurrent Prolog 等语言中。“promise” 这个概念是由 Barbara Liskov 和 Liuba Shrira 在1998年提出。

我第一次听说 JavaScript 中的 promise，Node 才刚刚出现，Node 社区正在讨论异步行为的最佳方案。社区实验了一段时间 promise，但最终选择了 error-first 回调作为 Node 标准。

大概在同一时间，Dojo 通过 Deferred API 添加了 promises 。不断增长的兴趣和活跃度最终导致了新的 Promises/A 规范的形成，提高了不同 promises 之间的互操作性。

jQuery 的异步操作是围绕 promises 的重构。jQuery 的 promise 和 Dojo 的 Deferred 非常相似，并且由于 jQuery 的流行，它一度在 JavaScript 中被广泛使用。但是 [ it did not support the two channel (fulfilled/rejected) chaining behavior & exception management](https://blog.domenic.me/youre-missing-the-point-of-promises/) 人们期望在 promise 上的构建工具。

尽管有这些缺点，jQuery 让 JavaScript promise 成为主流，并且一下更好的独立的 promise 库，例如 Q ， When ， 和 Bluebird 变得非常流行。jQuery 的实现的不兼容性促使 promise 规范中的一些重要说明，它被重写并重命名为 [Promises/A+ specification](https://promisesaplus.com/) 。

ES6 使用了遵循 Promises/A+ 兼容的 Promise 作为全局变量，并且一些非常重要的 API 构建在新的 Promise 标准之上：特别是 [WHATWG Fetch](https://fetch.spec.whatwg.org/) 规范和 [Async Functions](https://tc39.github.io/ecmascript-asyncawait/) 标准(第3阶段草案)

本文描述的是与 Promises/A+ 规范兼容的 promise，着重于基于 ECMAScript 标准的 `Promise` 实现。

## Promises 是如何工作的

promise 是一个从异步函数同步返回的对象。它有3种可能的状态：

+ **Fulfilled：** 将会调用 `onFulfilled()` (e.g.， 调用 `resolve()`)

+ **Rejected：** 将会调用 `onRejected()` (e.g.， 调用 `reject()`)

+ **Pending：** 非 fulfilled 或者 rejected

如果 promise 不在 pending 状态，它就固定了(处于 fulfilled 或 rejected)。有时我们用 *resolved* 或 *settled* 表示同一件事：*not pending* 。

promise 一旦被固定，就不能重新固定。调用 `resolve()` 或 `reject()` 将没有效果。被固定的 promise 的不可变性是一个重要的特性。

原生 JavaScript promise 不暴露 promise 状态。相反，你应该把 promise 视为一个黑盒。只有创建这个 promise 的函数才知道 promise 的状态，或者说有 `resolve` 或 `reject` 的权限。

这是一个返回一个 promise 的函数，并在制定的时间延迟后 `resolve` 。

{% gist 2dbea5f98fde8a0a77eb93775aa97468 wait.js %}

调用 `wait(3000)` 会等待3000毫秒 (3秒) ，然后调用打印 `'Hello!'` 。所以与规范兼容的 promise 都定义了 `.then()` 方法，你可以传入一个函数来处理 `resolve` 或 `reject` 返回的值。

ES6 promise 的构造函数需要一个函数。这个函数需要两个参数，`resovle()` 和 `reject()` 。在上面的例子中，我们只使用了 `resolve()` ,将 `reject()` 从参数列表中删除了。我们调用 `setTimeout()` 来创建延迟，并在完成后调用 `resolve()` 。

你可以按需向 `resolve()` 或 `reject()` 中传值，这个值会作为参数传入 `.then()` 的回调函数。

当我调用 `reject()` 并且传入一个值，我总是传入一个 `Error` 对象。一般来说，我想要两种可能的解决状态：正常的或异常的 (阻碍正常状态) 。传入一个 `Error` 对象来表示。

## Promise 的重要规则

标准的 Promise 是由 [Promises/A+ specification](https://promisesaplus.com/implementations) 社区定义的。有很多符合标准的实现，包括 JavaScript ECMAScript promise。

Promise 必须遵循下面的规则：

+ Promise 或 "thenable" 需要提供标准的 `.then()` 方法。

+ pending 状态的 promise 可以转换为 fulfilled 或 rejected 状态。

+ fulfilled 或 rejected 状态的 promise 是固定的，不能转换为其他状态。

+ 一旦 promise 固定，一定有一个值 (可以是 undefined) 。这个值不可变。

在这个上下午的变更指的是恒等 (===) 。fulfilled 的值可以是一个对象，这个对象的属性是可变的。

所以的 promise 都必须实现具有下面方法签名的 `.then()` 方法：

```js
promise.then(
    onFulfilled? : Function,
    onRejected? : Function
) => Promise
```

`.then()` 方法必须符合一下规则：

+ `onFulfilled()` 和 `onRejected()` 都是可选的。

+ 如果提供的参数不是函数，必须忽略。

+ `onFulfilled()` 在 promise 转换为 fulfilled 状态时被调用，promise 的值作为这个函数第一个参数。

+ `onRejected()` 在 promise 转换为 rejected 状态后被调用，拒绝的原因作为第一个参数。原因可以是任何有效的 JavaScript 值，但是拒绝基本和异常是同义词，所以我建议使用 Error 对象。

+ `onFulfilled()` 和 `onRejected()` 都不能多次调用。

+ `.then()` 可以在同一个 promise 上多次调用。换句话说，可以使用 promise 来聚合回调。

+ `.then()` 一定会返回一个 promise， `promise2` 。

+ 如果 `onFulfilled()` 或 `onRejected()` 的返回值为 `x` ，并且 `x` 是一个 promise ， `promise2`  会被锁定 (假设和 `x` 有相同的状态和值) 。 否则， `promise2` 将会转换为 fulfilled 状态并且值为 `x` 。

+ 不论 `onFulfilled` 或 `onRejected` 跑出一个异常 `e` ， `promise2` 一定会 rejected 并且 `e` 作为原因。

+ 如果 `onFulfilled` 不是一个函数并且 `promise1` 处于 fulfilled 状态， `promise2` 一定是 fulfilled 状态并且和 `promise1` 有同样的值。

+ 如果 `onRejected` 不是一个函数并且 `promise1` 处于 rejected 状态， `promise2` 一定是 rejected 状态并且和 `promise1` 的原因相同。

## Promise 链

因为 `.then()` 总是返回一个新的 promise ，所以可以链接 promises 并精确控制错误的处理方式和位置。Promises 允许你模仿同步代码的 `try/catch` 方式。

链式调用会形成一个有序的序列，类似于同步代码。换句话说，你可以这么做：

```js
fetch(url)
    .then(process)
    .then(save)
    .catch(handleErrors)
```

假设 `fetch()` `process()` `save()` 每个函数都返回 promises ， `fetch()` 完成后 `process()` 才开始执行， `process()` 完成后 `save()` 才开始执行。任何一个 promise reject 后 `handleErrors()` 才会执行。

这是一个复杂的 promise 链 的例子，并具有多个 rejection：

{% gist 8a49bbbf15e697182a5dd9edafd765e9 promise-chaining.js %}

## Error 处理

注意 promise 同时有成功和错误的 handler ，下面的代码很常见：

```js
save().then(
    handleSuccess,
    handleError
)
```

但是 `handleSuccess()` 抛出错误应该怎么做？从 `.then()` 返回的 promise 将会 rejected ，但是没有捕捉到这个 rejection 。意味着你的 app 的这个错误被吞了。

出于这个原因，有些人认为上面的代码是反模式的，并推荐下面的代码：

```js
save()
    .then(handleSuccess)
    .catch(handleError)
```

差异很微妙但是很重要。在第一个示例中， 源自 `save()` 的错误将会被捕获， 但源自 `handleSuccess()` 的错误将会被吞掉。

![Without .catch(), an error in the success handler is uncaught.](https://cdn-images-1.medium.com/max/1600/1*5Z_vNz6xHn9mjTgvrqa2Aw.png)


在第二个示例中，`.catch()` 可以处理 `save()` 和 `handleSuccess()` 两者的 rejection 。

![With .catch(), both error sources are handled.](https://cdn-images-1.medium.com/max/1600/1*vRaV9sYpYKdxBj3Ld7KM1Q.png)

当然， `save()` 的错误可能是网络错误，但 `handleSuccess()` 的错误可能是开发人员忘记处理特定的状态码。如果你想以不同的方式处理它们，你可以选择这样处理：

```js
save()
    .then(
        handleSuccess,
        handleNetworkError
    )
    .catch(handleProgrammerError)
```

无论你喜欢哪一种方式，我都建议在所有的 promise 链都是以 `.catch()` 结尾。重复这么做是值得的：

> *I recommend ending all promise chains with a .catch().*

## 如何取消一个 Promise？

promises 萌新经常想知道的第一件事是如何取消一个 promise。这是一个想法：以 "Cancelled" 为原因 reject promise 。如果你想和 "normal" 错误以不同的处理方式，在 error handler 中添加一个处理分支。

以下是人们在取消 promise 时所犯的一些常见错误：

### 在 promise 中添加 .cancel()

添加 .cancel() 使 promise 不符合标准，而且也违反了一些 promises 的规则：只有创建 promise 的函数才能 resolve， reject，或者 cancel 这个 promise 。暴露出来会破坏 promise 的封装，并且会鼓励人们在不应该知道 promise 的地方编写操作 promise 的代码。避免意大利面式的代码以及破坏 promises 。

### 忘记清理

有一些聪明的人发现可以使用 `Promise.race()` 组委取消机制。问题是使用了创建 promise 的函数来做取消控制，只有这个函数可以清理活动内容，例如清理 timeouts 或者清理数据引用的内存空间等等。

### 忘记处理 rejected 状态下取消 promise

你知道吗当你忘记处理 promise rejection ， Chrome 控制台会抛出一个警告信息。

### 过度复杂

[withdrawn TC39 proposal](https://github.com/tc39/proposal-cancelable-promises) 提出了使用独立的消息通道实现取消 promise 。它提出了一个叫做 cancellation token 的新概念。在我看来，这个解决方案会让 promise 规范变得臃肿，它的唯一作用是分离了 rejections 和 cancellations ，这是没有必要的。

你是否希望基于 exception 或者 cancellation 来进行切换？这确实是 promise 的工作？我认为不是。

### 重新思考 Promise Cancellation

通常，在 promise 创建时传入所有需要的信息以确定如何 resolve / reject / cancel 这个promise 。这样就没有必要在 promise 上添加 `.cancel()` 函数了。你可能会有疑问，在 promise 创建的时候怎么知道是否可以取消。

> *“If I don’t yet know whether or not to cancel, how will I know what to pass in when I create the promise?”*

如果只有某种对象可以代替未来的潜在的值...等一下。

我们传入的那个代表是否取消的值可以是一个 promise 。可能是这样：

{% gist acb7f555a2f8b4ba93807105da5e11ae cancellable-wait.js %}

我们使用的默认参数默认是不取消的。`cancel` 参数是可选的。像之前一样设定一个 settimeout ，不过会保存这个 timeout 的 ID 以便于在之后清理。

我们使用 `cancel.then()` 方法来处理 cancellation 和资源清理。这个方法只会在 promise resolve 之前取消了才会执行。如果你太晚取消，你就错过了你的机会。火车已经驶离了车站。

> *注意：你可能想知道 `noop()` 函数是做什么的。单词 noop 代表了 no-op，意思是空函数什么都不做。如果没有这个函数，V8 引擎会抛出警告：`UnhandledPromiseRejectionWarning: Unhandled promise rejection` 。永远都要处理 promise rejections 是一个好主意，即使是 `noop()` 函数。*

> *Note: You may be wondering what the noop() function is for. The word noop stands for no-op, meaning a function that does nothing. Without it, V8 will throw warnings: UnhandledPromiseRejectionWarning: Unhandled promise rejection. It’s a good idea to **always handle promise rejections**, even if your handler is a noop().*

### 抽象 Promise Cancellation

这对于 `wait()` 计时器是很好的，但是我们可以进一步抽象这个想法，封装所有你需要的：

1. 默认拒绝 cancel promise -- 如果没有传入 cancel 到 promise ，我们不希望 cancel 或抛错。

2. 记住因 cancellations 而 reject 时要进行清理。

3. 记住 `onCancel` 的清理工作本身也会抛错，这个错误也需要处理。(注意上面的示例中省略了错误处理 -- 很容易忘记！)

让我们来创建一个可取消的 promise 工具，你可以使用它来包装任何 promise 。例如，处理网络请求等... 方法前面如下所示：

```js
speculation(fn: SpecFunction, shouldCancel: Promise) => Promise
```

SpecFunction 就像你传入 `Promis` 的构造函数一样，有一点不同 -- 它有一个 `onCancel()` handler：

```js
SpecFunction(resolve: Function, reject: Function, onCancel: Function) => void
```

{% gist 3dcf478645b131c3259456794861840a speculation.js %}

注意这个例子只是向你解释它是如何工作的。还有一些其他边界问题需要考虑。例如，这个版本的代码，promise 已经固定了，取消 promise 依然会调用 `handleCancel` 函数。

我实现了一个生产版本，处理了边界问题的开源库，[Speculation](https://github.com/ericelliott/speculation) .

让我们使用这个改进的抽象库来重写之前的 `wait()` 工具函数。首先安装 speculation：

```bash
npm install --save speculation
```

现在你可以导入和使用它：

{% gist 36ab3548b44a0833ddb3bb69745e18b9 wait-speculation.js %}

这简化了一些，因为你不需要关心 `noop()` ， 捕获 `onCancel()` ，函数或其他边界问题的错误。这些细节都被抽象到 `speculation()` 中了。随时在真实项目中使用它。

## 原生 JS Promise 的附加内容

原生 Promise 对象有一些你可能感兴趣的附加内容：

+ `Promise.reject()` 返回一个 rejected promise 。

+ `Promise.resolve()` 返回一个 resolved promise 。

+ `Promise.race()` 接收一个 promise 数组 (或任何迭代器) 并且返回第一个 resolved (值) 或 rejected (原因) 的 promise 。

+ `Promise.all()` 接收一个 promise 数组 (或任何迭代器) 并且当迭代器中所有 promises 都 resolved 后返回一个 promise ，或者返回第一个 rejected (原因) 的 promise。

## 总结

Promises 已称为 JavaScript 语法的重要部分，包括用于 ajax 请求的 [WHATWG Fetch](https://fetch.spec.whatwg.org/) 标准，用于让异步代码看起来像同步代码的 [Async Functions](https://tc39.github.io/ecmascript-asyncawait/) 标准。

在撰写本文时 (2017.1.23)，Async functions 还处于第3阶段，但我预测它们很快将成为 JavaScript 中异步编程的一种非常流行，非常有效的解决方案 -- 这意味着在不久的将来，学习 promises 对 JavaScript 开发人员来说将变得更加重要。

例如，如果你正在使用 Redux ，我建议你看下 [redux-saga]()：一个用于管理 Redux 中副作用的库，整个文档都依赖于异步函数。

我希望即使是有着丰富 promise 使用经验的开发人员在阅读完本文之后，能够对 promise 有更好的理解和 promise 是如何工作的，以及更好地去使用 promise 。

## 探索 'Master the JavaScript Interview' 系列

+ [What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36)

+ [What is the Difference Between Class and Prototypal Inheritance?](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

+ [What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

+ [What is a Function Composition?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

+ [What is a Function Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)

+ [What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

+ [Soft Skills](https://medium.com/javascript-scene/master-the-javascript-interview-soft-skills-a8a5fb02c466)
