---
title: 【译】JavaScript 工程师需要知道的十道面试题
---
> 原文：[10 Interview Questions Every JavaScript Developer Should Know](https://medium.com/javascript-scene/10-interview-questions-every-javascript-developer-should-know-6fa6bdf5ad95)

## 你能说出两个对 JavaScript 应用程序开发人员很重要的编程范式吗？

JavaScript 是一个多范式语言，通过 __OOP__ （面向对象编程）和 __函数式__ 编程使其支持 __命令式/过程式__ 编程。JavaScript 支持以原型链继承方式的 OOP。

### 加分点：

+ 原型继承 （或者： 原型， OLOO）

+ 函数式编程 （或者： 闭包，头等函数，lambdas）

### 减分点：

+ 不知道编程范式是什么，没有提到基于原型的面向对象或函数式编程

<!-- more -->

### 补充学习：

+ [The Two Pillars of JavaScript Part 1](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3) -- 基于原型的面向对象
+ [The Two Pillars of JavaScript Part 2](https://medium.com/javascript-scene/the-two-pillars-of-javascript-pt-2-functional-programming-a63aa53a41a4) -- 函数式编程

## 什么是函数式编程？

函数式编程通过组合数学函数来生成程序，并且避免共享状态和可变数据。Lisp （1958年指定） 是最早支持函数式编程的语言之一，并且深受 lambda 演算的启发。至今 Lisp 以及很多衍生语言仍在普遍使用。

函数式编程是 JavaScript 的一个基本概念（JavaScript 的两大支柱之一）。ES5 JavaScript 添加了一些常用工具方法。

### 加分点：

+ 纯函数/函数纯度

+ 避免副作用

+ 简单函数组合
  
+ 函数式语言的例子：Lisp，ML，Haskell，Erlang，Clojure，Elm，F Sharp，OCaml 等等。。。
  
+ 提及支持函数式编程的特性：头等函数，高阶函数，函数作为参数/值。

### 减分点：

+ 没有提及纯函数/避免副作用
  
+ 不能举出函数式编程语言的例子
  
+ 不知道 JavaScript 作为函数式编程特性

### 补充学习：

+ [The Two Pillars of JavaScript Part 2](https://medium.com/javascript-scene/the-two-pillars-of-javascript-pt-2-functional-programming-a63aa53a41a4) -- 函数式编程
  
+ [The Dao of Immutability](https://medium.com/javascript-scene/the-dao-of-immutability-9f91a70c88cd)
  
+ [Composing Software](https://medium.com/javascript-scene/composing-software-an-introduction-27b72500d6ea)
  
+ [The Haskell School of Music](http://haskell.cs.yale.edu/wp-content/uploads/2015/03/HSoM.pdf)

## 类继承和原型继承之间有什么区别？

**类继承：** 实例继承自类（如蓝图--类的描述），并创建子类关系：层级分类。实例通常对构造函数使用‘new’关键字进行实例化。类继承可以使用 ES6 中的 ‘class’ 关键字声明。

**原型继承：** 实例直接继承于其他对象。实例通常使用工厂函数或 ‘Object.create()’ 函数 进行实例化。实例可以由许多不同的对象组成，也可以方便的选择继承对象。

>In JavaScript, prototypal inheritance is simpler &
>more flexible than class inheritance.

### 加分点：

+ 类：创建紧耦合或层级/分类
  
+ 原型：提及拼接继承，原型委托，函数继承，对象组合。

### 减分点：

+ 没有提及原型继承和组合优于类继承

### 补充学习：

+ [The Two Pillars of JavaScript Part 1 — Prototypal OO](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3)
  
+ [Common Misconceptions About Inheritance in JavaScript](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a)
  
{% vimeo 69255635 %}

## 函数式编程和面向对象编程的优缺点是什么？

**OOP 优点：** 容易理解对象的基本概念，并且易于解释方法调用的含义。OOP 倾向于使用命令式风格而不是声明式风格，它类似于计算机遵循的直接指令集。

**OOP 缺点：** 通常依赖于共享状态。对象和行为通常绑定在同一个实体上，这个实体可以由任意数量的函数随机访问，可能导致诸如竞争条件的不期望的行为。

**FP 优点：** 使用函数范式，程序员避免了共享状态和副作用，从而消除由多个函数竞争统一资源所导致的错误。由于具有像 point-free style 的特性，相比于 OOP 函数式倾向于从根本上简化和易于重新组合以获得更通用的可复用代码。

FP 也倾向于声明式和指称式，它没有详细说明每一步的操作，而是集中精力在 **做什么** ，让底层函数去处理 **如何做** 。这为重构和性能优化留下了巨大的空间，甚至只需修改很少的代码就可以用更高效的算法替换整个算法。

利用纯函数的计算可以很容易的扩展到跨多处理器或跨分布式计算集群，而不必担心线程资源冲突，条件竞争等。。。

**FP 缺点：** 过度利用 FP 特性（如 point-free style和大型组合）可能会降低可读性，因为生成的代码通常更抽象，更简洁，不具体。

与函数式编程相比，更多人熟悉 OO 和命令式编程，因此即使是函数式编程中的常见习惯也会让新团队成员感到困惑。

FP 比 OOP 有更陡峭的学习曲线，因为 OOP 的流行使得 OOP 的学习材料更易于交流，而 FP 更具学术性和正式性。FP 的概念经常被写成来自 lambda 演算，代数和范畴学所使用的习语和符号，所有这些都需要在这些领域的知识基础。

### 加分点：

+ 提及共享状态的问题，不同事件竞争同一资源等等。。。

+ 意识到 FP 能够从根本上简化很多应用

+ 意识到不同的学习曲线

+ 阐明副作用以及它如何影响程序的可维护性

+ 意识到高函数式代码基础可能具有陡峭的学习曲线

+ 意识到与同等的 FP 代码基础相比，高 OOP 代码基础会难以修改并且非常脆弱。

+ 意识到不可变性对程序状态历史的获取和可塑性产生一个极大提高，允许轻松添加诸如无限撤销/重做，倒带/重放，时间旅行调试等功能。不可变性可以在任意编程范式中实现，但共享状态对象使不可变性在 OOP 中的实现变得复杂。

### 减分点：

+ 无法列出任一编程范式的缺点--任何一种范式的人都会遇到一些限制。

### 补充学习：

+ [The Two Pillars of JavaScript Part 1](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3) -- 基于原型的面向对象

+ [The Two Pillars of JavaScript Part 2](https://medium.com/javascript-scene/the-two-pillars-of-javascript-pt-2-functional-programming-a63aa53a41a4) -- 函数式编程

## 什么时候类继承是一个合适的选择

答案是从不，或者几乎从不。一定不能有超过一层的继承。多层类继承是反模式的。多年来我一直在发出这个挑战，我听过的答案都属于几个常见的误解之一。更常见的是，挑战遇见沉默。

>“If a feature is sometimes useful
>and sometimes dangerous
>and if there is a better option
>then always use the better option.”
>~ Douglas Crockford

### 加分点:

+ 很少,几乎从不,从不.

+ 单一层级有时是可以接受的,一个基于类的框架,例如 React.Component

+ 偏爱对象组合高于类继承.

### 补充学习:

+ [The Two Pillars of JavaScript Part 1](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3) -- 基于原型的面向对象

+ [JS Objects — Inherited a Mess](http://davidwalsh.name/javascript-objects)

## 什么时候原型继承是一个合适的选择

原型继承有不止一个种类

+ **委托** (i.e.,原型链)

+ **拼接** (i.e.,mixins, ’Object.assign()‘)

+ **函数化** (不要和函数式编程混淆.用于为私有状态/封装创建闭包的函数)

每种类型的原型继承都有自己的一组用例，但它们在组合方式同样有用，它创建了 **has-a** 或 **uses-a** 或 **can-do** 的关系而不是由类继承创建的 **is-a** 的关系.

### 加分点:

+ 在模块或函数式编程没有提供明显解决方案的情况下

+ 当你需要从多个资源组合对象

+ 任何你需要继承的时候

### 减分点:

+ 不知道何时使用原型

+ 没有意识到 mixins 或 ’Object.assign()‘.

### 补充学习:

+ [“Programming JavaScript Applications”: Prototypes section](http://chimera.labs.oreilly.com/books/1234000000262/ch03.html#chcsrdou100015eilvj6l9inj)

## “优先使用对象组合高于类继承”是什么意思?

这是  [“Design Patterns: Elements of Reusable Object-Oriented Software”](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612) 的引用.这意味着代码复用应该是通过将较小的函数组合到新的对象中来实现,而不是通过类的继承来创建新的分类.

换句话说,要使用 **can-do**, **has-a**,或 **use-a** 关系而不是 **is-a** 关系.

### 加分点:

+ 避免类继承

+ 避免脆弱的基类问题

+ 避免紧耦合

+ 避免严格分类（强制 is-a 关系最终会在新的使用方案中出错）

+ 避免大猩猩香蕉问题（“你想要的是一个香蕉，得到的是拿着香蕉的大猩猩和整个丛林”）

+ 使代码更灵活

### 减分点：

+ 没有提到上面的任何问题

+ 未能阐明组合和类继承的区别，或组合的优点

### 补充学习：

{% youtube wfMtDGfHWpA %}


{% blockquote Eric Elliott https://medium.com/p/77f8911c2fee Introducing the Stamp Specification %}
Move Over, 'class':
Composable Factory Functions Are Here
{% endblockquote %}


## 什么是双向数据绑定和单向数据流，他们有何不同？

双向数据绑定意味着 UI 和数据模型的动态绑定，当 UI 发生变化，数据模型也会发生变化，反之亦然。

单向数据流意味着数据模型作为数据来源。改变 UI 会触发消息通知把用户意图发送给数据模型（类似 React 中的 “store”）。只有数据模型才能改变应用状态。结果是数据总是向单一方向流动，能够使代码易于理解。

单向数据流是确定的，而双向绑定可能产生副作用使其难以跟踪和理解。

### 加分点：

+ Reac 是单向数据流的新典范，所以提及 Reac 是一个好的信号。 + + + Cycle.js 是另外一个流行的单向数据流实现。

+ Angular 是一种使用双向绑定的流行框架

### 减分点：

+ 不知道两者的意思。无法阐明两者的区别

### 补充学习：

{% youtube XxVg_s8xAms %}

## 一体化和微服务架构的优缺点

一体化架构意味着你的应用代码是紧密结合的，所有组件协同工作，共享内存空间和资源。

微服务架构意味着你的应用程序由许多较小的独立应用程序组成，这些应用程序能够在自己的内存空间中运行，并且它们彼此相互独立，甚至可以在不同机器下运行。

**一体化架构优点：** 一体化架构的主要优势是大多数应用程序有大量的交叉点，例如日志，限速以及审查跟踪和 DOS 保护等安全功能。

当一切都在同一个应用程序中运行时，这样就很容易讲个组件串联起来。

还有性能优势，因为共享内存比进程间通信 （IPC） 更快。

**一体化架构缺点：** 一体化架构通常是紧耦合的，应用版本迭代杂糅，使得独立扩展和代码可维护性的隔离服务困难。

一体化结构也难以理解，因为当你查看特定服务或控制器时，可能存在一些不明显的依赖，副作用以及不可思议的问题。

**微服务架构优点：** 微服务架构通常有更好的组织结构，因为每个部分有自己特定的功能，且不需要关心其他部分的功能。为不同应用服务进行重构和重新配置的解藕服务也更简单。（例如同时提供给网页客户端和开放 API 的服务）

有于微服务架构的组织方式，它还有性能优势，因为可以隔离热服务并且将其扩展为独立于应用程序的其余部分。

**微服务架构缺点：** 当你构建一个新的微服务架构时，你可能会发现很多在设计时没有预料到的跨组件问题。一体化结构可以轻松地通过共享方式或者中间件来处理跨组件问题。

在微服务架构中，隔离组件之间通信的开销，封装的组件在另外一个服务层可以让所有路由通过。

最后，一体化架构也需要外部服务来处理组间通信，但一体化架构可以把这个这个工作成本延迟到项目更加成熟的时候。

微服务经常部署在它们自己的虚拟机或容器里，导致 VM 竞争的激增。这些任务经常通过容器管理工具自动完成。

### 加分点：

+ 倾向于微服务架构，尽管微服务架构初始成本更高。意识到从长远看微服务性能好易扩展。

+ 关于微服务架构和一体化架构的实用性。在构建程序方面，微服务在代码级别相互隔离，一体化在开始时就容易捆绑在一起。微服务的高成本可以延迟到更加成熟的时候。

### 减分点：

+ 不知道一体化和微服务之间的差别

+ 对微服务架构额外高成本不清楚或一些不切实际的想法

+ 对微服务架构， IPC （夸进程通信）和网络通信的额外成本不清楚

+ 对微服务架构过于消极，不能明确表达如何通过结偶一体化架构来轻松分割成微服务

+ 低估了独立可扩展的微服务架构的优势

## 什么是异步编程，为什么它在 JavaScript 中很重要？

同步编程意味着，除了条件语句和函数调用，代码从上到下依次执行，例如网络请求和磁盘 I/O 这样长时间的任务会造成阻塞。

异步编程意味着引擎运行在时间循环上。当需要阻塞操作时，请求开始后，代码继续运行不需要等待请求结果。当响应准备好，触发中断，导致事件处理运行，控制器在其后执行。通过这种方式，单线程就可以处理并发操作了。

用户交互本质上是异步的，需要花费大部分时间等待用户输入中断事件循环触发事件回调。

Node 默认时异步的，这意味着服务方式大致相同，循环等待网络请求，在处理第一个请求的同时接收更多到来的请求。

异步在 JavaScript 中时非常重要的，因为他非常适合用户交互，并且有利于提高服务器的性能。

### 加分点：

+ 知道什么是阻塞，以及性能的意义

+ 知道事件回调，以及为什么对用户界面交互很重要

### 减分点：

+ 不熟悉异步或同步

+ 无法表达用户交互和异步对性能的影响

