---
title: 【译】精通 JavaScript： 什么是纯函数（Pure Function）？
tags:
    - JavaScript
date: 2018-09-03 17:13:58
---

> 原文：[Master the JavaScript Interview: What is Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

![Image: Pure - carnagenyc (CC BY 2.0)](https://cdn-images-1.medium.com/max/2000/1*gF8oCkYNvktBbAAG-nxYrg.jpeg)

纯函数对于包括函数式编程，可靠并发和 React+Redux apps 等用途至关重要。但是 "pure function" 是什么意思呢？

<!-- more -->

我们可以在一个免费的课程中学到这个问题的答案 -- [“Learn JavaScript with Eric Elliott”](http://ericelliottjs.com/product/lifetime-access-pass/) :

{% vimeo 160326750 %}

在我们理解纯函数之前，仔细研究函数可能时一个好主意。可以从不同方式去观察它们，让我们更容易理解函数式编程 (functional programming) 。

## 什么是函数 (Function) ？

函数是接收输入 (**参数**) 并产生输出 (**返回值**) 的过程。函数有以下用途：

+ **映射 (Mapping)** ：基于输入产生一些输出。函数是输入值到输出值的映射。

+ **过程 (Procedures)** ：调用函数来执行一个序列的步骤。这个序列被称为过程，并且这种编程风格被称为**过程编程 (procedural programming)** 。

+ **I/O** ：存在一些函数需要和系统的其他部分通信，例如屏幕，存储，系统日志，或者网络。

## 映射 (Mapping)

纯函数都是关于映射的。函数将参数映射成返回值，意味着对于每一组输入都存在一个输出。函数会接收输入并且返回相应的输出。

`Math.max()` 接收多个数值作为参数并且返回最大的数值：

```js
Math.max(2, 8, 5) // 8
```

在这个例子中，2，8 和 5 作为参数。传入函数中的值。

`Math.max()` 函数接收任意数量的数值作为参数并且返回参数中最大的值。在这个例子中，传入的最大的数值是 8 ，它就是我们要返回的值。

函数在计算和数学中非常重要。它们帮助我们正确地使用数据。优秀的程序员会使用有意义的函数名，当我们看代码时，通过函数名就知道这个函数是做什么的。

数学也有函数，它们和 JavaScript 的函数类似。你可能在代数中看到过函数。像下面这样：

*f(x) = 2x*

这以为了我们定义了一个叫做 f 的函数并且有参数 x 然后用 2 乘以 x 。

我们可以使用一个值代替 x 来运用这个函数：

*f(2)*

在代数中，这和写 4 的意义完全相同。

所以任何地方出现的 *f(2)* 都可以用 4 代替。

现在我们转换这个函数为 JavaScript 的方式：

```js
const double = x => x * 2
```

你可以使用 `console.log()` 函数输出来检验：

```js
console.log( double(5) ) // 10
```

还记得我说过可以用 `4` 来替换 `f(2)` ，这种情况下， JavaScript 引擎使用 `10` 替换了 `double(5)` 。

所以 `console.log(double(5))` 和 `console.log(10)` 相同。

因为 `double()` 是纯函数所以这是正确的，但是如果 `double()` 有副作用，例如保存数据到磁盘或者记录到控制台，在不改变函数含义的情况下你不能简单地用 `10` 替换 `double(5)` 。

如果你想引用透明，你需要使用纯函数。

## 纯函数 (Pure Functions)

**纯函数**是：

+ 给予相同的输入，总是返回相同的输出。

+ 不产生副作用

> *A dead giveaway that a function is impure is if it makes sense to call it without using its return value. For pure functions, that’s a noop.*

我建议你爱上纯函数。意思是如果使用纯函数可以实现程序要求，你应该选择纯函数而非其他。纯函数接收一些输入然后返回基于这些输入的输出。它们是构建程序的最简单可复用模块。也许计算机科学的最重要的设计原则是 KISS (Keep It Simple, Stupid) 。我更喜欢 Keep It Stupid Simple 。纯函数是傻瓜简单的最有可能的方式。

纯函数具有很多有益属性，并且是构建**函数式编程**的基础。纯函数是完全独立于外部状态的，因此，它们免疫于于共享可变状态相关这一类型的 bugs 。纯函数的独立性使其成为处理跨多个 CPU 及跨整个分布式计算机集群的理想选择，这使它们成为很多科学种类和资源密集型计算任务的必要条件。

纯函数也是极其独立的 -- 代码易于转换，重构，重组，使你的程序更灵活且更适应未来的改变。

### 共享状态 (Shared State) 的问题

几年前，我当时正在开发一款 app ，允许用户从数据库中搜索音乐家并将这个音乐家的音乐列表加载到网络播放器。在那个时间 Google Instant 出现，当你键入搜索查询会即时显示搜索结果。AJAX 驱动的自动完成突然风靡一时。

唯一的问题是用户输入速度比 API 自动完成搜索的返回值要快，这会导致奇怪的问题。这会触发竞争条件 (race condition) ，较新的结果会被过时的结果覆盖。

为什么会发生这个？因为每个 AJAX 请求成功后都可以直接显示在用户的建议列表上。最慢的 AJAX 请求总是盲目地替换结果显示给用户，即使被替换的结果可能是较新的。

为了解决这个问题，我创建了一个建议管理器 -- 管理查询建议状态的唯一数据来源。已知当前正在等待的 AJAX 请求，当用户新键入后，这个等待的 AJAX 请求将会取消并且发起一个新请求，所以同一时间只有一个响应处理函数能够触发 UI 更新。

任何类型的异步或同步操作都可能导致类似的竞争条件。如果输出取决于不可控事件的序列 (例如网络，设备延迟，用户输入，随机性等) ,则会发生条件竞争。事实上，如果你在使用共享状态并且这个状态依赖于根据不确定因素而变化的序列，则无论出于何种意图和目的，输出都是不可预测的，这意味着无法正确测试和完全理解。正如 Martin Odersky (Scala 创造者) 所说：

> non-determinism = parallel processing + mutable state

程序确定性通常是计算机应用中的理想属性。也许你认为 JS 在单线程中运行，因此不受并行处理问题的影响，但正如 AJAX 示例所示，单线程 JS 引擎并不意味着没有并发性。相反的，JavaScript 中存在很多并发源 (API I/O, 事件监听, web workers, iframes, timeouts) 向程序中引入不确定性。将这些与共享状态结合，你会得到一堆 bugs 。

纯函数可以帮助你避免这些 bugs 。

### 接收同样的输入，总是返回同样的输出

使用 `double()` 函数，你可以用结果替换函数调用，并且程序会认为是同一件事， `double(5)` 和 `10` 在程序应用中是同一个意思，无论上下午如何，无论调用的次数和时间。

但并不是所有的函数都是和结果一直相同。某些函数还依赖于传入参数之外的信息来生成结果。

思考这个例子：

```js
Math.random()   // 0.4619094094074556
Math.random()   // 0.602651887966867
Math.random()   // 0.9958664270880297
```

即使我们没有传入任何参数到上面的函数调用中，它们生成了不同的输出，这意味着 `Math.random()` 不是纯函数。

`Math.random()` 函数在每次调用后会生成一个新的在 0 和 1 之间的随机数，所以很明显你不能在不改变程序含义的情况下用 `0.4619094094074556` 替换 `Math.random()` 函数。

如果替换了每次都会产生相同的结果。当我们向计算机询问一个随机数时，通常意味着我们想要不同于上一次的结果。所有侧面都是相同数字的骰子有什么意思？

有时我们会向计算机询问当前时间。我们不会详细介绍时间函数的工作原理。现在，复制下面的代码：

```js
const time = () => new Date().toLocaleTimeString()
time() // => "2:20:34 PM"
```

如果用当前时间替换 `time()` 函数调用会发生什么？

它将会一直输出相同的时间：那个替换函数调用的时间。换句话说，每天只能生成一个正确的输出，并且只有在替换函数调用的时间的确切时刻运行程序才会生成。

所以很明显， `time()` 和 `double()` 不同。

只有给定相同的输入，一定生成相同的输出的函数才是纯函数。你可能记得袋鼠中的这个规则，相同的输入值总是映射到相同的输出值。然而，不同的输入值可以映射到相同的输出值。例如，下面这个函数是纯函数：

```js
const highpass = (cutoff, value) => value >= cutoff
```

相同的输入值总是映射到相同的输出值：

```js
highpass(5, 5) // => true
highpass(5, 5) // => true
highpass(5, 5) // => true
```

不同的输入值可以映射到相同的输出值：

```js
highpass(5, 123) // true
highpass(5, 6) // true
highpass(5, 18) // true

highpass(5, 1) // false
highpass(5, 3) // false
highpass(5, 4) // false
```

纯函数一定不可以依赖任何外部可变状态，以为这样函数将不再具有确定性和引用透明。

## 纯函数不会产生副作用 (No Side Effects)

纯函数不会产生副作用，意思是它不能改变任何外部状态。

## 不可变性 (Immutability)

JavaScript 的对象参数是引用，这意味着如果函数改变了一个对象或数组参数的属性，也会改变在函数外部的状态。纯函数一定不会改变外部状态。

思考这个可变的，非纯函数 `addToCart()`

{% gist 27d689731f703fa0e095c0dc524728c6 impure-add-to-cart.js %}

它的工作原理是传入 cart，item 和 quantity (购物车，商品种类和商品数量) 。函数会将 item 添加到 cart 并返回 cart。

这个函数的问题是我们刚刚改变了一些共享状态。其他函数可能依赖于调用函数之前状态的 cart ，现在我们已经改变了共享状态，如果我们改变调用函数的顺序，我们不得不担心它会对程序逻辑产生什么影响。重构代码可能导致产生 bugs ，这可能会搞砸订单，导致客户生气。

现在思考这个版本：

{% gist 56ef48bc044251fe44203703ed6902d7 pure-add-to-cart.js %}

在这个示例中，因为对象中嵌套了一个数组，所以需要深克隆。这比你一般处理的状态更复杂。大多数情况下，你可以将其分解成更小的块。

例如， Redux 允许你组合 reducers 而不是在每个 reducer 中处理整个 app 状态。结果是，每次你想要更新其中的一小部分时，不必创建整个 app 状态的深克隆。相反的，你可以使用 non-destructive 数组方法或 `Object.assign()` 来更新 app 状态的一小部分。

## 探索 'Master the JavaScript Interview' 系列

+ [What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36)

+ [What is the Difference Between Class and Prototypal Inheritance?](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

+ [What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

+ [What is a Function Composition?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

+ [What is a Function Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)

+ [What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

+ [Soft Skills](https://medium.com/javascript-scene/master-the-javascript-interview-soft-skills-a8a5fb02c466)




