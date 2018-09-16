---
title: 【译】精通 JavaScript： 什么是函数式编程（Functional Programming）？
tags:
  - JavaScript
abbrlink: 25329c80
date: 2018-08-24 15:26:58
---

> 原文：[Master the JavaScript Interview: What is Functional Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)

![Structure Synth — Orihaus (CC BY 2.0)](https://cdn-images-1.medium.com/max/1600/1*1OxglOpkZHLITbIKEVCy2g.jpeg)

> “精通 JavaScript 面试” 是一个系列的文章，旨在帮助面试者准备他们在申请中高级职位时可能遇到的常见问题。这些是我在现实面试中经常提出的问题。

函数式编程已经成为了 JavaScript 世界的一个热门话题。仅仅在几年前，甚至只有很少 JavaScript 开发者知道函数式编程，但是在过去的3年中我看到了大量使用函数式编程思维构建的应用。

<!-- more -->

**函数式编程** （简称 FP）是通过纯函数组合来构建软件，避免**共享状态(shared state)**， **可变数据(mutable data)**，以及**副作用(side-effects)**。函数式编程是**声明式(declarative)**的而不是**命令式(imperative)** 的，应用程序状态通过纯函数而改变。与面向对象编程相比，应用程序的状态和对象的方法是共享和共存的。

函数式编程是一种**编程范式(programming paradigm)**，意思是它是一种基于一些基本的，确定的原则来思考软件构建的一种方式。编程范式的其他例子有包括面向对象编程和过程编程。

与命令式和面向对象代码相比，函数式的代码通常更简洁，可预测和易于测试。但是如果你不熟悉函数式编程以及与之相关的常见模式，它看起来更愚钝，而且对于新手来说相关文献也难以理解。

如果你使用谷歌搜索函数式编程，你很快会发现学术术语的高墙，对于初学者来说会非常令人生畏。函数式编程有一个恐怖的学习曲线。但是如果你已经使用了 JavaScript 一段时间，你很可能在真实编程中是用了很多函数式编程的概念和工具。

***Don’t let all the new words scare you away. It’s a lot easier than it sounds.***

最困难的部分是关于不熟悉的词汇的思维转变。在你掌握函数式编程的意义之前，你还需要理解下面这些定义：

+ 纯函数

+ 函数组合

+ 避免共享状态

+ 避免可变状态

+ 避免副作用

换句话说，如果你想真正理解函数式编程的含义，你必须首先理解这些核心概念。

## 纯函数 (Pure Function)

纯函数是：

+ 输入相同，输出一定相同

+ 没有副作用

纯函数具有很多在函数式编程中非常重要的属性，包括**引用透明** （你可以在不改变程序的定义下替换函数调用及返回值）。更多细节 -- [“What is a Pure Function?”](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)


## 函数组合 (Function Composition)

**函数组合**是组合两个或多个函数以产生新的函数或执行某些计算的程序。例如 `f.g` 等同于 JavaScript 中的 `f(g(x))`。理解函数组合是了解如何使用函数式编程构建软件的重要一步。补充学习 -- [“What is Function Composition?”](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

## 共享状态 (Shared State)

**共享状态** 是存在于共享作用域的任何变量，对象或内存空间，或者是在不同作用域之间传递的一个对象的属性。共享作用域可以是全局或闭包。通常，在面向对象编程中，不同作用域直接通过想另一对象添加属性来进行共享对象。

例如，在电子游戏中又一个主对象，这个主对象将游戏角色和物品作为属性。函数式编程避免了共享状态，依赖于不可变数据结构或从已有数据计算而得到的新数据。有关软件函数如何处理软件状态的更多详细信息 -- [“10 Tips for Better Redux Architecture”](https://medium.com/javascript-scene/10-tips-for-better-redux-architecture-69250425af44)

共享状态的问题在于，为了理解函数的结果，你必须要了解所有函数使用或影响每个共享变量的完整历史记录。

想象一下，你有一个需要保存的用户对象。`saveUser()` 是一个向服务器发送请求的 API。当 `saveUser()` 执行时，用户使用 `updateAvatar()` 函数更改个人资料图片并触发了另一个 `saveUser()` 函数。保存时，为了保持与服务器同步或响应其他 API 调用，服务器返回一个用户对象来替换内存中的对象。

不幸的是，第二个响应在第一个响应之前被接收，所以当第一个（现在是过时的）响应返回时，内存中的旧对象已经被覆盖了图片的对象替换了。这是一个条件竞争的例子--一个与共享状态相关的常见错误。

另一个与共享状态的常见问题是，更改函数调用顺序可能会导致级联错误，因为作用于共享状态的函数与调用时机有关：

{% gist 23f93290914912816211c9041ddd856d timing-dependency.js %}

当你避免了共享状态，函数调用的时机和顺序不会改变返回结果。使用纯函数，相同的输入，总能得到相同的输出。这使得函数调用完全独立于其他函数调用，这从根本上简化了修改和重构。一个函数的修改或函数调用的时机不会改变或破坏程序的其他部分。

{% gist a9768374c9084bb3cae6d64b5827024d no-timing-dependency.js %}

上面的例子中，我们使用 `Object.assign()` 并传入一个空对象作为第一个参数来复制 x 的属性而不是直接修改它。在这种情况下，它相当于从头创建了一个新对象。 `Object.assign()` 是 JavaScript 中复制已存在的属性来创建对象的常用模式而不是使用直接修改，我们在第一个例子中演示了直接修改赋值。

如果你仔细观察了这个例子中的 `console.log()` 语句，你应该注意到我已经提到过的事情：函数组合。回想之前提到的，函数组成如下所示： `f(g(x))` 。我们使用了 `x1()` 和 `x2()` 替换了 `f()` 和 `g()`。

如果你改变了组合的顺序，输出会发生改变。操作顺序依然是一个麻烦。 `f(g(x))` 不总是等同于 `g(f(x))`，但是函数外部的变量发生了什么是无所谓的--这个是重点。使用非纯函数，除非你知道函数使用或影响的每个变量的完整历史记录，否则无法完全理解函数的作用。

移除函数调用顺序的依赖，并且消除整个类潜在的错误。

## 不可变性 (Immutability)

**不可变(immutable)** 对象指的是一个对象在创建后无法修改。反过来说，一个 **可变(mutable)** 对象指的是一个对象在创建后可以修改。

**不可变性(Immutability)** 是函数式编程的一个核心概念，因为没有她，你的程序中的数据流是有损的。状态历史被丢弃，你的软件会蔓延奇怪的错误。关于不可变性的更多信息 -- [“The Dao of Immutability.”](https://medium.com/javascript-scene/the-dao-of-immutability-9f91a70c88cd)

在 JavaScript 中，重要的是不要将 `const` 和不可变性混淆。`const` 创建了一个变量和变量名的绑定，在创建后无法重新赋值。`const` 不是创建了一个不可变对象。你无法修改绑定引用的对象，但可以修改对象的属性，也就是说这个 `const` 创建的绑定是可变的。

不可变对象是从根本上无法修改的。你可以深度冻结一个对象是这个值无法修改。JavaScript 有一个方法可以冻结对象的一层：

{% gist e01b161efad12f9615fba55b15c684f7 frozen-objects.js %}

但是冻结的对象只是表面上的不可变。例如，下面这个对象是可变的：

{% gist 18585681ea1079101307fa269ec4716c frozen-not-immutable.js %}

如你所见，冻结对象的顶层原始属性不能修改，但任何是对象的属性（包括数组等）仍然可以修改。所以，即使冻结对象也不是不可变的，除非你遍历这个对象数并冻结每个对象属性。

在许多函数式编程语言中，有一种特殊的不可变数据结构称为 **trie 数据结构**，它们被有效地深度冻结。意味着任何层次的属性都无法修改。

Tries 使用 **结构共享(structural sharing)** 被复制过的不可变对象的所有部分是共享引用内存位置的，这样可以减少使用内存空间，并且可以显著提高某些操作的性能。

例如，你可以在对象树的根部标识进行比较。如果标识相同，就不必遍历整个树来检查差异。

JavaScript 中有几个库利用了 tries，包括 [Immutable.js](https://github.com/facebook/immutable-js) 和 [Mori](https://github.com/swannodette/mori).

我已经对两者进行了实验，并且倾向于在需要大量不可变状态的大型项目中使用 Immutable.js 。有关更多信息 -- [“10 Tips for Better Redux Architecture”](https://medium.com/javascript-scene/10-tips-for-better-redux-architecture-69250425af44)

## 副作用 (Side Effects)

副作用是除了被调用函数的返回值外还有应用程序状态的更改。副作用包括：

+ 修改了任何外部变量或对象属性（e.g., 全局变量或者在父函数作用域链的变量）

+ 控制台日志

+ 屏幕输出

+ 写入文件

+ 写入网络

+ 触发任何外部进程

+ 调用任何其他函数

函数式编程大多避免了副作用，这使得程序更易理解，也更易测试。

Haskell 和其他函数式语言经常使用 **[monads](https://en.wikipedia.org/wiki/Monad_%28functional_programming%29)** 来隔离和封装来自纯函数的副作用。

你现在需要知道的是，具有副作用的操作需要和软件的其他部分隔离。如果将副作用和程序逻辑其余部分分开，你的软件将更容易扩展，重构，调试，测试和维护。

这就是大部分前端框架鼓励用户将状态和组件渲染分开，松耦合的原因。

## 高阶函数的可重用性 (Reusability Through Higher Order Functions)

函数式编程倾向于复用一组通用的工具函数来处理数据。面向对象编程倾向于方法和数据共存在对象中。这些方法只能处理专门设计的数据，并且通常只能操作同一实例的数据。

在函数式编程中，所有类型的数据都是公平的。同一个工具函数 `map()` 可以处理对象，字符串，数字或其他任何类型的数据，因为它将函数作为处理给定类型数据的一个参数。FP 使用高阶函数作为通用技巧。

JavaScript 的 **头等函数(First Class Functions)** ,允许将函数作为数据。将函数作为变量，参数或返回值等等。。。

**高阶函数(Height Order Function)**, 将函数作为参数或返回值，或两者都有。高阶函数通常用于：

+ 抽象或隔离动作，结果，或者通过回调，promise，monads 等等来实现异步函数。

+ 创建可以处理各种类型数据的工具函数

+ 将函数作为参数或创建柯里化函数，目的是复用或函数组合

+ 输入一组函数并返回这些函数的组合

## 容器，函子，列表和流(Containers, Functors, Lists, and Streams)

函子是可以映射的。换句话说，他是一个拥有使用函数替换其中的值的接口的容器。当你看到函子时，你应该想到“可映射的”。

之前我们学习了可以处理各种数据类型的工具函数 `map()`。它提供一种映射操作作为函子 API。`map()` 使用的流程控制操作利用了这个接口。在 `Array.prototype.map()` 这种情况下，容器时数组，但是只要支持 mapping API，其他的数据结构也可以作为函子。

让我们来看看 `Array.prototype.map()`，它时如何允许你从映射工具中抽象数据类型，使 `map()` 可用于任何数据类型。我们将创建一个简单的 `double()` 映射，将传入的值双倍输出。

{% gist 3e62d5c301c70a6c29eaaafe15009405 double-mapping.js %}

如果我们想要在游戏中的点数翻倍，需要怎么做？我们所要做的就是传入 `map()` 的 `double()` 函数进行微妙的修改，并且一切正常：

{% gist 0238c8e291b2c2c5a2f33d3333c51fb2 map-custom-data-type.js %}

在函数式编程中，为了操作任意数量的不同数据类型，使用如同函子和高阶函数等的抽象的通用工具函数是一个重要概念。你可以看到类似的概念 -- [all sorts of different ways](https://github.com/fantasyland/fantasy-land)

*“A list expressed over time is a stream.”*

现在你需要知道的是，数组和函子不是容器应用中容器和值的唯一概念。例如，数组只是事物列表。随时间改变的列表是流，所以你可以使用相同类型的方法来处理传入事件的流，当你真正开始用 FP 构建软件时，你会看到很多东西。

## 声明式 vs 命令式(Declarative vs Imperative)

函数式编程是一种声明式范式，意味着程序逻辑不需要确定流程控制来表达。

**命令式(Imperative)** 程序使用特定步骤来获得需要的结果 -- **流程控制(flow control)**：**如何(how)** 去做。

**声明式(Declarative)** 程序抽象流程控制 -- **做什么(what)**。抽象做的方式。

例如，命令式映射传入一个数字数组，返回一个这个数组且其中的值都加倍：

{% gist 8b5c98dc2b324ee8cfea451762d07bfe imperative-example.js %}

这个声明式映射做了相同的事情，但是使用了 `Array.prototype.map()` 工具来抽象流程控制，这使你更清楚地表达数据流：

{% gist a3fe04fe9f099cb54492bedea6a33fc1 declarative-example.js %}

命令式代码经常使用语句。语句是执行某些操作的一段代码。常用语句包括 `for`， `if`， `switch`， `throw` 等等。

声明式代码更多依赖于表达式。表达式是一段计算一些值的代码。表达式通常组合函数调用，值和操作来计算得出结果。

下面都是表达式的例子

``` js
2 * 2
doubleMap([2, 3, 4])
Math.max(4, 3, 2)
```

通常，表达式在代码中作为变量，函数返回值或函数参数。表达式在赋值，返回或者出入之前，需要先计算出它的结果。

## 结论

函数式编程的偏好：

+ 纯函数而不是共享状态和副作用

+ 不可变性高于可变数据

+ 函数组合高于命令式的流程控制

+ 很多使用高阶函数处理各种数据类型的通用的，可复用的工具函数而不是只能处理同一对象数据的方法

+ 声明式而不是命令式 (做什么，而不是怎么做)

+ 表达式而不是语句

+ 容器 & 高阶函数优于特定多态(ad-hoc polymorphism)








