---
title: 【译】精通 JavaScript： 什么是函数组合（Function Composition）？
tags:
  - JavaScript
abbrlink: 8e5d7b3b
date: 2018-09-04 17:08:58
---

> 原文：[Master the JavaScript Interview: What is Function Composition?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

![Google Datacenter Pipes — Jorge Jorquera — (CC-BY-NC-ND-2.0)](https://cdn-images-1.medium.com/max/2000/1*4trMikhKRHhSUlI2D6h_mA.jpeg)

> “精通 JavaScript 面试” 是一个系列的文章，旨在帮助面试者准备他们在申请中高级职位时可能遇到的常见问题。这些是我在现实面试中经常提出的问题。

函数式编程已经成为了 JavaScript 世界的一个热门话题。仅仅在几年前，甚至只有很少 JavaScript 开发者知道函数式编程，但是在过去的3年中我看到了大量使用函数式编程思维构建的应用。

函数组合是将两个或多个函数组合以产生新函数的过程。将函数组合在一起就像是将一系列管道拼凑在一起，以便我们的数据流过。

<!-- more -->

简而言之，函数 `f` 和 `g` 的组合可以定义为 `f(g(x))` ，它从内到外，从右到左进行求值。换句话说，求值顺序是：

1. `x`

2. `g`

3. `f`

让我们在代码中进一步观察这个概念。想象一下，你希望将用户的全名转换为 URL slugs ，以便为每个用户提供一个 profile 页面。为了实现这一点，你需要完成一系列的步骤：

1. 将姓名以空格拆分到一个数组中

2. 将名字映射为小写

3. 用破折号 `-` 链接数组中的名字

4. 编码为 URI component

下面是一个简单的实现：

{% gist abc20e52b727ad246be4afe8dcf21f16 toSlug.js %}

还不错，但如果我告诉你让它更具可读性呢？

想象一下，每个操作都对应一个可组合的函数。它可以写成这样：

{% gist ad816466c9540132b59561b56cb7720f nesting-composition.js %}

这看起来比我们的第一次尝试更难阅读，但先忍一下，我们就要解决了。

为了实现这一点，我们使用可组合形式的常用工具函数，例如 `split()` `join()` 和 `map()` 。下面是它们的实现：

{% gist a51511a34c803c758422ed4e5fcd2efe composables.js %}

除了 `toLowerCase()` 之外，这些函数都可以从 Lodash/fp 获得。你可以像这样导入它们：

```js
import { curry, map, join, split } from 'lodash/fp'
```

或者像这样：

```js
const curry = require('lodash/fp/curry')
const map = require('lodash/fp/map')
//...
```

在这我偷懒了。注意这个 curry 不是真正的技术上的柯里化函数，真正的柯里化函数总是产生一元函数。这里的 curry 只是一个简单的偏函数应用。参考 [ “What’s the Difference Between Curry and Partial Application?”](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8) ,但是为了这次演示的目的，将它作为真正的柯里化函数。

回头看我们的 `toSlug()` 的实现，有一些东西真的困扰了我：

{% gist ad816466c9540132b59561b56cb7720f nesting-composition.js %}

在我看来它似乎有很深的嵌套，读起来有点混乱。我们可以使用一个自动组合这些函数的函数来展平嵌套，这意味着它将从一个函数获得输出并自动传入到下一个函数的输入，直到输出最终值。

想想看，好像数组中有一个函数可以做到差不多的事情。这个函数就是 `reduce()` ，它拿到一个值的列表并对这每个值应用一个函数，累计得出一个结果。这些值可以是函数。但是为了和上面的行为组合相匹配，我们需要 `reduce()` 从右向左递减而不是从左到右递减。

好事情是有一个 `reduceRight()` 函数做到了我们需要的事情：

{% gist fce3416828f389618623fcfcef387756 compose.js %}

和 `.reduce()` 一样， `.reduceRight()` 方法也有一个 reducer 函数和初始值 `x` 。我们从右向左迭代数组中的函数，依次将每个函数应用后的值累计到最终值 `v` 。

使用 compose ， 我们可以不使用嵌套来重写上面的组合函数：

{% gist e9c02710a62e68683b6aa38d7729408f using-compose.js %}

当然， `compose()` 在 lodash/fp 中也有：

```js
import { compose } from 'lodash/fp'
```

或者：

```js
const compose = require('lodash/fp/compose')
```

当以数学形式从内到外的角度思考时， compose 是很棒的，但是如果你想从左到右顺序的角度来思考，该如何去做？

通常还有另外一种方式称作 `pipe()` 。在 Lodash 中称作 `flow()` 。

{% gist 8287dd95e9d7be38e9c33a948dfc63e2 pipe.js %}

注意，`pipe()` 和 `compose()` 的实现完全相同，除了使用了 `.reduce()` 而不是 `.reduceRight()` ，即是从左到右缩减而不是从右到左。

让我们看看用 `pipe()` 实现的 `toSlug()` 函数：

{% gist c0ce31c7534a5b693c32547877aac2ac using-pipe.js %}

这对我来说更易阅读。

硬核函数式程序员使用函数组合定义他们的整个应用程序。我经常使用它来消除对临时变量的需求。仔细观察 `pipe()` 版本的 `toSlug()` 函数，你可能会注意到一些特别的事情。

在命令式编程中，当您对某个变量执行转换时，您将在转换的每个步骤中找到对该变量的引用。上面 `pipe()` 的实现是以 **无参(points-free)** 风格的方式编写，这意味着它根本不识别它运行的参数。

我经常在单元测试和 Redux state reducers 之类的实现中使用 pipes 来消除对中间变量的需求，这些中间变量仅存于一个操作和下一个操作之间的瞬间值。

这听起来可能很奇怪，但是随着你使用它，你会发现在函数式编程中，你是在和相当抽象广义的函数打交道，其中事物的名称并不重要。名称只是妨碍。你可能开始将变量视为不必要的样板。

也就是说，我认为无参风格可能被过度使用。它可能变得过于密集，难以理解，但是如果你感到困惑，这里有一个小小的建议，你可以进入流程来跟踪发生了什么：

{% gist 6f354876134011f0fbcb1355ce9da6f4 trace.js %}

下面如何使用它：

{% gist 16fd8336b32ab0ec71e6c4be61ce355f using-trace.js %}

`trace()` 只是更通用的 `tap()` 的一种特殊形式，它允许你为流经管道的每个值执行一些操作。明白了吗？ Pipe ？ Tap ？ 你可以这么编写 `tap()` ：

{% gist 459b2bc590b89c2e0bea27ebac385e7f tap.js %}

现在你知道为什么 `trace()` 是 `tap()` 的一个特例：

{% gist 4981bbb10a15deab96ecbf34da3c9f34 tapped-trace.js %}

你应该开始了解函数式编程是什么了，以及 **偏函数应用(partial application)** 和 **柯里化(currying)** 是如何协同 **函数组合(function composition)** 来帮助你编写更易读且更少样板的程序。

**探索 'Master the JavaScript Interview' 系列**

+ [What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36)

+ [What is the Difference Between Class and Prototypal Inheritance?](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

+ [What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

+ [What is a Function Composition?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

+ [What is a Function Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)

+ [What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

+ [Soft Skills](https://medium.com/javascript-scene/master-the-javascript-interview-soft-skills-a8a5fb02c466)




