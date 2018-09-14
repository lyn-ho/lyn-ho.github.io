---
title: 【译】JavaScript 工作原理：V8 引擎中5个优化代码的技巧
tags:
  - JavaScript
---

> 原文：[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

几个星期之前，我们开始了一个旨在深入挖掘 JavaScript 及其实际工作原理的系列文章，我们认为，通过了解 JavaScript 的底层构建以及它们是如何协作的，你将能够编写更好的代码和应用。

[第一篇文章](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)主要关注引擎，运行时和调用栈的概述。 第二篇文章将深入探讨 Google's V8 引擎内部。我们还提供了一些关于如何编写更好的 JavaScript 代码的快速提示 —— 我们 [SessionStack](https://www.sessionstack.com/) 开发团队在构建产品时  遵循的最佳实践。

## 概述 (Overview)

**JavaScript 引擎**是一个可以执行 JavaScript 代码的程序或解释器。 JavaScript 引擎可以是标准实现解释器或者是可以以某种方式实现编译 JavaScript 为字节码的即时编译器。

<!-- more -->

下面是流行 JavaScript 引擎列表：

- [**V8**](https://en.wikipedia.org/wiki/V8_%28JavaScript_engine%29) —— 开源， Google 开发， C++ 编写

- [**Rhino**](https://en.wikipedia.org/wiki/Rhino_%28JavaScript_engine%29) —— Mozilla 基金会管理，开源，全 Java 开发

- [**SpiderMonkey**](https://en.wikipedia.org/wiki/SpiderMonkey_%28JavaScript_engine%29) —— 第一个 JavaScript 引擎，当时支持 Netscape Navigator ， 现在支持 FireFox

- [**JavaScriptCore**](https://en.wikipedia.org/wiki/JavaScriptCore) —— 开源， Apple 公司 为 Safari 浏览器开发并以 Nitro 为名字推广

- [**KJS**](https://en.wikipedia.org/wiki/KJS_%28KDE%29) ——  KDE 的引擎，最初是由 Harri Porten 为 KDE 项目的 Konqueror 网络浏览器开发

- [**Chakra** (JScript9)](https://en.wikipedia.org/wiki/Chakra_%28JScript_engine%29) —— Internet Explorer

- [**Chakra** (JavaScript)](https://en.wikipedia.org/wiki/Chakra_%28JavaScript_engine%29) —— Microsoft Edge

- [**Nashorn**](https://en.wikipedia.org/wiki/Nashorn_%28JavaScript_engine%29) —— 开源，由 Oracle 的 Java 语言工具组开发，是 OpenJDK 的一部分

- [**JerryScript**](https://en.wikipedia.org/wiki/JerryScript) —— 物联网轻量级引擎

## 为什么创建了 V8 引擎

V8 引擎是由 Google 开发 C++ 编写的开源引擎。它被用于 Google Chrome 。但是，和其他引擎不一样，V8 也被用于 Node.js runtime 。

![V8](https://cdn-images-1.medium.com/max/1600/1*AKKvE3QmN_ZQmEzSj16oXg.png)

V8 最初是为了提高 Web 浏览器中 JavaScript 执行的性能。为了获得更快的速度，V8 将 JavaScript 转换为更高效的机器码，而不是使用解释器。它像 SpiderMonkey 或者 Rhino (Mozilla) 等很多现代 JavaScript 引擎一样，通过 **JIT(Just-In-Time)编译器** 将 JavaScript 代码编译成机器码。区别是 V8 不生成字节码或任何中间代码。

## V8 曾经有两个编译器

在 V8 的 v5.9 版本出来之前，它曾经拥有两个  编译器：

- full-codegen —— 一个简单且快速的编译器，可以生成简单但相对  较慢的机器码。

- Grankshaft —— 一个较复杂的即时优化编译器，可以生成  高度优化的代码

V8 引擎内部使用了多线程：

- 主线程完成你所期望的任务：获取你的代码，编译执行

- 还有一个单独的线程用于编译，以便主线程可以继续执行，而前者可以优化代码

- Profiler 线程，它将通知  运行时  哪些  方法花费大量时间，以便 Crankshaft 可以优化它们

-  少许线程来处理垃圾收集器扫描

首次执行 JavaScript 代码，V8 利用 **full-codegen** 直接将解析后的 JavaScript 不经任何转换地编译为机器码。这使它可以非常快速地  开始执行机器码。注意， V8 不使用中间字节码，因此不需要解释器。

当代码运行一段时间后， profiler 线程以及收集来足够的数据来通知运行时应该优化哪个方法。

然后， **Crankshaft** 在另一个线程开始优化。它将 JavaScript 抽象语法树转换为名为 **Hydrogen** 的高级静态单元分配表示 (SSA) ，并尝试去优化这个 Hydrogen 图。大多数优化在这个层级完成。

## 代码嵌入 (Inlining)

首次优化是尽可能的提前嵌入更多的代码。代码嵌入就是将使用函数的地方( 函数被调用的那一行代码)替换成调用函数的本体。这个简单的步骤使下面的优化更有意义。

![inlining](https://cdn-images-1.medium.com/max/1600/0*RRgTDdRfLGEhuR7U.png)

## 隐藏类 (Hidden class)

JavaScript 是一门基于原型的语言：没有通过克隆创建的类和对象。 JavaScript 也是一门动态语言，这意味着可以在对象实例化后轻易地向它添加或删除属性。

 大多数 JavaScript 解释器使用类似字典的结构 (基于[散列函数](http://en.wikipedia.org/wiki/Hash_function)) 来存储对象属性在内存中的位置。这种结构使得在 JavaScript 中检索属性的值比在 Java 或 C＃ 等非动态编程语言中的计算成本更高。在 Java 中，所有对象属性都是在编译之前由固定对象布局确定的，并且无法在运行时动态添加或删除 (C＃具有 [动态类型](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/dynamic)，这是另一个主题) 。因此，属性值（或指向这些属性的指针）可以作为连续 buffer 存储在存储器中，每个值之间具有固定偏移值。可以根据属性类型轻易确定偏移的长度，而在运行时可以更改属性类型的 JavaScript 中，这是不可能的。

由于使用字典在内存中查找对象属性的位置效率非常低，因此 V8 使用不同的方法：**隐藏类 (hidden classes)** 。隐藏类的工作方式类似于 Java 等语言中使用的固定对象布局（类），除了它们是在运行时创建的。现在，让我们看看它们实际上是什么样的：

```js
function Point(x, y) {
  this.x = x
  this.y = y
}

var p1 = new Point(1, 2)
```

一旦 `new Point(1, 2)` 被调用，V8 会生成一个 `C0` 的隐藏类。

![C0](https://cdn-images-1.medium.com/max/1600/1*pVnIrMZiB9iAz5sW28AixA.png)

到现在还没有为 `Point` 定义任何属性，所以 `C0` 是空的。

 一旦第一条语句 `this.x = x` (`Point` 函数内的) 执行后，V8 将创建一个基于 `C0` 的第二个隐藏类 `C1` 。`C1` 描述了属性值 `x` 在内存中的位置(相对于对象指针) 。在这个例子中，`x` 存储在[偏移值]()为 0 的地方，这意味着当在内存中把 `point` 对象视为一段连续的 buffer 时，它的第一偏移量对应的属性就是 `x` 。V8  还会使用 “ 类转换 (class transition)” 更新 `C0` ，如果将属性 `x` 添加到 `Point` 对象， 隐藏类就会从 `C0` 切换到 `C1` 。现在 `Point` 对象的隐藏类为 `C1` 。

![C1](https://cdn-images-1.medium.com/max/1600/1*QsVUE3snZD9abYXccg6Sgw.png)

每当一个新属性添加到对象，旧的隐藏类就会通过一个转换路径更新成一个新的隐藏类。隐藏类转换非常重要，因为它们允许在以相同方式创建的对象之间共享隐藏类。如果两个对象共享一个隐藏类，并给它们添加相同的属性，隐藏类转换能够确保这两个对象都获得新的隐藏类以及与之相关联的优化代码。

当执行语句 `this.y = y` (同样，在 `Point` 函数内部，`this.x = x` 语句之后) 时，将重复此过程。

一个新的隐藏类 `C2` 被创建了，如果属性 `y` 被添加到 Point 对象(已经包含了 `x` 属性)，同样的过程，类型转换被添加到 `C1` 上，然后隐藏类开始更新成 `C2`，并且 Point 对象的隐藏类就要更新成 `C2` 了。

![C2](https://user-gold-cdn.xitu.io/2017/11/18/15fcec13180b6f2e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

隐藏类转换是根据属性被添加到对象上的顺序而发生变化。我们看看下面这一小段代码：

```js
function Point(x, y) {
  this.x = x
  this.y = y
}

var p1 = new Point(1, 2)
p1.a = 5
p1.b = 6

var p2 = new Point(3, 4)
p2.b = 7
p2.a = 8
```

现在，你可能认为 `p1` 和 `p2` 使用了相同的隐藏类和类转换。其实不然，对于 `p1` 先添加属性 `a` 再添加属性 `b` ，但是 `p2` 先添加 `b` 后添加 `a` 。因此，`p1` 和 `p2` 以不同转换路径的结束，它们的隐藏类也不同。**在这种情况下， 最好以相同的顺序初始化动态属性，以便可以复用隐藏类。**

## 内联缓存 (Inline caching)

V8 利用另一种被称为内联缓存的技术来优化动态类型语言。内联缓存依赖于观察到：发生在相同类型的对象上的同一个方法的重复调用。关于内联缓存深入解释请看[这里](https://github.com/sq/JSIL/wiki/Optimizing-dynamic-JavaScript-with-inline-caches)。

我们将讨论内联缓存的一般概念（如果您没有时间进行上面的深入解释）。

那么它是怎样工作的？V8 会维护一个在最近的方法调用的参数的对象类型的缓存，并使用这些信息去预测将要传入参数的对象类型。如果 V8 对传递给方法的对象类型做出了很好的预测，那么它就能够绕开获取对象属性的计算过程，取而代之的是使用先前查找这个对象的隐藏类时所存储的信息。

那么隐藏类和内联缓存的概念是如何  关联的？每当在特定对象上调用方法时，V8 引擎必须执行对该对象的隐藏类的查找，以确定访问特定属性的偏移量。在将同一方法成功调用两次到同一个隐藏类之后，，V8 就会略过查找隐藏类，将这个属性的偏移值添加到对象本身的指针上。未来这个方法的所有调用，V8 引擎都会假设隐藏类没有更改，并使用先前查找中存储的偏移  值直接跳转到特定属性的内存地址。这极大的提高了 V8 的执行速度。

内联缓存也是同类型对象共享隐藏类非常重要的原因。如果创建两个相同类型且具有不同隐藏类的对象(如同我们之前的示例)，V8 将无法使用内联缓存，因为即使两个对象属于同一类型，其对应的隐藏类也会为其属性分配不同的偏移值。

![inline caching](https://cdn-images-1.medium.com/max/1600/1*iHfI6MQ-YKQvWvo51J-P0w.png)

这两个对象基本相同，但 `a` 和 `b` 属性的创建顺序不同。

## 编译成机器码 (Compilation to machine code)

经 Hydrogen graph 优化后，Crankshaft 将其降低到一个较低层次 Lithium 。大多数 Lithium 实现都是特定于体系结构的。寄存器分配发生在这个级别。

最后， Lithium 会被编译成机器码。然后，触发 OSR 一种运行时替换正在运行的栈帧的技术 (on-stack replacement) 。这我们开始编译和优化一个明显耗时的方法之前，我们可能正在运行它。V8 不会  遗弃  正在缓慢执行的代码而直接开始执行优化  后的。相反，它将转换所有的上下文环境 (堆栈，寄存器) ，以便我们可以这执行过程中切换到优化的版本。这是一项非常复杂的任务，请记住，在其他优化中，V8 原来已经内联了代码。V8 并不是唯一能够做到这一点的引擎。

V8 有一种称为去优化的保护措施可以进行相反的转换，并在发动机制造的假设不再适用的情况下恢复到非优化代码。

## 垃圾回收 (Garbage collection)

对于垃圾收集，V8 采用传统的标记和扫描方式来清理旧数据。标记阶段会阻止 JavaScript 执行。为了控制 GC 成本并使运行更稳定，V8 使用增量标记：不是遍历整个堆，尝试标记每个可能的对象，它只是遍历堆的一部分，然后恢复正常执行。下一个 GC 将从上一个堆遍历停止的位置继续。这允许在正常执行期间非常短暂的暂停。如前所述，扫描阶段由单独的线程处理。

## Ignition 和 TurboFan

随着 2017 年早些时候发布 V8 v5.9，引入了新的执行管道。这个新的管道在实际的 JavaScript 应用程序中实现了更大的性能提升和显着的内存节省。

新的执行管道建立在 V8 解释器 [Ignition](https://github.com/v8/v8/wiki/Interpreter)，和最新优化的 V8 编译器 [TurboFan](https://github.com/v8/v8/wiki/TurboFan) 之上。

您可以在[这里](https://v8project.blogspot.bg/2017/05/launching-ignition-and-turbofan.html)查看 V8 团队关于该主题的博客文章。

自从 V8 的 v5.9 版本问世以来，V8 已经不再使用 full-codegen 和 Crankshaft（自 2010 年以来为 V8 提供服务的技术）用于执行 JavaScript，因为 V8 团队一直在努力跟上新的 JavaScript 语言功能以及功能优化。

这意味着整体 V8 将具有更简单，更易维护的架构。

![Improvements on Web and Node.js benchmarks
](https://cdn-images-1.medium.com/max/1600/0*pohqKvj9psTPRlOv.png)

这些改进只是一个开始。新的 Ignition 和 TurboFan 管道为进一步优化铺平了道路，这些优化将在未来几年内提升 JavaScript 性能并缩小 V8 在 Chrome 和 Node.js 中的占用空间。

最后，这里有一些关于如何编写优化良好的 JavaScript 的技巧和窍门。您可以从上面的内容中轻松地推导出这些内容，但是，这里是为方便起见的摘要：

## 如何编写优化的 JavaScript

1. **对象属性的顺序** ：始终以相同的顺序实例化对象属性，以便可以共享隐藏类和随后优化的代码。

2. **动态属性** ：在向实例化的对象添加属性将强制更改隐藏类并减慢为先前隐藏类优化的任何方法。而是在其构造函数中分配所有对象的属性。

3. **方法** ：重复执行相同方法的代码将比仅执行一次许多不同方法的代码运行得更快（因为内联缓存）。

4. ** 数组** ：避免 keys 不是增量数字的稀疏数组。含有空元素的稀疏数组是**哈希表**。这种数组中的元素访问起来更加昂贵。另外，尽量避免预先分配大数组。最好的做法是随着你的需要慢慢的增大数组。最后，不要删除数组中的元素。它使 keys 稀疏。

5. **标记值** ：V8 使用 32 位表示对象和数字。它使用一个位来知道它是一个对象（flag = 1）还是一个称为 SMI（SMall Integer）的整数（flag = 0），因为它是 31 位的。然后，如果数值大于 31 位，V8 将对该数字进行  封装 (box) ，将其变为双精度 (double) 并创建一个新对象以将数字放入其中。尽可能使用 31 位带符号的数字，以避免对 JS 对象进行昂贵的装箱操作。尽可能使用 31 位带符号的数字，以避免对 JS 对象进行昂贵的封装操作。

## 资源 (Resources)

- [Crankshafting from the ground up](https://docs.google.com/document/u/1/d/1hOaE7vbwdLLXWj3C8hTnnkpE0qSa2P--dtDvwXXEeD0/pub)

- [Notes and resources related to V8 and thus Node.js performance](https://github.com/thlorenz/v8-perf)

- [Project: v8](https://bugs.chromium.org/p/v8/adminIntro)

- [V8 Resources](https://mrale.ph/v8/resources.html)

- [Google I/O 2012 - Breaking the JavaScript Speed Limit with V8](https://www.youtube.com/watch?v=UJPdhx5zTaw)

- [V8: an open source JavaScript engine](https://www.youtube.com/watch?v=hWhMKalEicY)
