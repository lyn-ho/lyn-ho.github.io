---
title: 【译】精通 JavaScript： 什么是闭包（Closure）？
tags:
  - JavaScript
abbrlink: 3b887bb4
date: 2018-08-27 12:34:58
---

![closure](https://cdn-images-1.medium.com/max/2000/1*J-jjDviwGUfzka1HX5LG9A.jpeg)

> 原文：[Master the JavaScript Interview: What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36)

> “精通 JavaScript 面试” 是一个系列的文章，旨在帮助面试者准备他们在申请中高级职位时可能遇到的常见问题。这些是我在现实面试中经常提出的问题。

什么是闭包通常是我 JavaScript 面试的第一个和最后一个问题。坦白说，如果你不理解闭包你就不能深入掌握 JavaScript。

<!-- more -->

你别含糊其辞，但是你真的理解如何构建一个严肃的 JavaScript 程序吗？你真的明白发生了什么，或者程序是如何运行的。我有疑虑，如果不知道这个问题的答案是一个非常危险的信号。

你不仅应该知道闭包的机制，还应该知道闭包为什么重要，并且可以轻易回答闭包的几个应用场景。

在 JavaScript 中，闭包通常用于数据私有，事件处理和毁掉函数，以及 [偏函数应用(Partial Applications)和柯里化(Currying)](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.l4b6l1i3x) ,还有其他函数式编程模式。

我不在乎面试者是否知道 “closure” 这个单词或它的专业定义。我只想知道他们是否知道闭包的基本原理。如果不知道，通常代表他们没有很多构建真正 JavaScript 应用的经验。

> *If you can’t answer this question, you’re a junior developer. I don’t care how long you’ve been coding.*

> That may sound mean, but it’s not. What I mean is that most competent interviewers will ask you what a closure is, and most of the time, getting the answer wrong will cost you the job. Or if you’re lucky enough to get an offer anyway, it will cost you potentially tens of thousands of dollars per year in pay because you’ll be hired as a junior instead of a senior level developer, regardless of how long you’ve been coding.

快速准备一下：“你能说出两种闭包的通用场景？”

## 什么是闭包

闭包是绑定函数和引用其周围的状态的组合 **(词法环境 lexical environment)**。换句话说，闭包可以让内部函数访问外部函数的作用域。在 JavaScript 中，每次创建函数时，都会在创建函数时创建闭包。

使用闭包，只需在另一个函数内定义一个函数并且公开出来。通过返回或传递给另一个函数来公开这个函数。

内部函数可以访问外部函数作用域的变量，即使外部函数完成返回了。

## 使用闭包 (例子)

闭包通常用于对象数据私有化。数据私有是让我们能够面向接口编程而不是实现编程的基础。这是可以帮助我们构建强壮软件的一个重要概念。因为实现细节比接口更容易改变。

> *“Program to an interface, not an implementation.”*

> *[Design Patterns: Elements of Reusable Object Oriented Software](http://www.amazon.com/gp/product/B000SEIBB8?ie=UTF8&camp=213733&creative=393177&creativeASIN=B000SEIBB8&linkCode=shr&tag=eejs-20&linkId=CSQYBHTUP625XI4T)*

在 JavaScript 中，闭包是用于实现数据私有的主要机制。当你使用闭包实现数据私有时，被封装的数据只能在闭包函数作用域中使用。你无法绕过对象 **被授权的方法(privileged methods)** 在外部访问这些数据。任何在闭包作用域内定义的公开方法都是特权方法。例如：

{% gist 6fb9890ce1ffb14bad9f059adc4a2f0d data-privacy-example.js %}

在上面的例子中，'.get()' 方法定义在 `getSecret()` 方法的作用域内，它可以访问`getSecret()` 中的任何变量，于是它就是一个被授权的方法。在这个例子中，它可以访问参数 `secret`。

对象不是产生数据私有的唯一方法。闭包也可以用于创建有状态函数，其返回值可能受其内部状态影响，e.g.:

```js
const secret = msg => () => msg;
```

{% gist b16af817ec53ad8bbb49875aa4795e74 secret.js %}

在函数式编程中，闭包通常用于偏函数应用和柯里化。为了理解这些，我们需要一些定义：

**应用程序**：一个应用给定参数并产生一个返回值的过程。

**偏函数应用**：一个应用部分参数的函数的过程。并返回一个新函数作为参数。换句话说，应用一个具有多个参数的函数并返回一个较少参数的函数。偏函数应用先使用一个或部分参数在将要返回的函数中，等待返回的函数调用剩余的参数来完成整个应用。

偏函数应用利用闭包作用域来提前使用部分参数。你可以实现一个通用函数来赋予指定函数部分参数。如下所示：

```js
partialApply(targetFunction: Function, ...fixedArgs: Any[]) =>
  functionWithFewerParams(...remainingArgs: Any[])
```

如果你想进一步了解，请看 [Rtype: Reading Function Signatures](https://github.com/ericelliott/rtype#reading-function-signatures)

利用一个接受任意参数的函数，赋予我们指定的参数。并且返回一个新函数接受剩余的参数。

这个例子将会帮助你理解。你现在又一个 求两个数的和 的函数：

```js
const add = (a, b) => a + b;
```

现在你想要一个 10 加任意数字的函数。我们称它为 `add10()`。`add10(5)` 的结果应该是15. `partialApply()` 函数将会实现它。

```js
const add10 = partialApply(add, 10)
add10(5)
```

在这个例子中，'10‘ 作为 **固定参数(fixed parameter)** 通过闭包作用域提前赋予给 `add()`, 从而我们获得了 `add10()`。

让我们来看一下如何实现 `partialApply()`：

{% gist 543442825c8519c03e19584daab41bc9 partial-apply.js %}

如你所见，它只是简单地返回一个函数，这个函数通过闭包访问了传给 `partialApply()` 函数的 fixedArgs 参数。












