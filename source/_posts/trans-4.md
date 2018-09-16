---
title: 【译】精通 JavaScript： 类继承和原型继承的区别？
tag:
  - JavaScript
abbrlink: 48b81a71
date: 2018-08-29 14:38:58
---

![Electric Guitar](https://cdn-images-1.medium.com/max/2000/1*rtVyaoswTo9iljufAz7Y8A.jpeg)

原文：[Master the JavaScript Interview: What’s the Difference Between Class & Prototypal Inheritance?](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

> “精通 JavaScript 面试” 是一个系列的文章，旨在帮助面试者准备他们在申请中高级职位时可能遇到的常见问题。这些是我在现实面试中经常提出的问题。如果你想从头开始，可以看 ["What is a Closure?"](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36#.6xq65f6f5) 开始。

>注意：这篇文章的例子使用的是 ES6。如果你不知从何开始， 可以参阅 ["How to Learn ES6"](https://medium.com/javascript-scene/how-to-learn-es6-47d9a1ac2620)

与其他大多数语言不同的是， JavaScript 的对象系统是基于 **原型(prototypes)** ， 而不是 **类(classes)** 。不幸的是，大多数 JavaScript 开发人员都不能深入了解 JavaScript 的对象系统，或者不能充分利用它。其他人确实理解，但是希望它表现得更像基于类继承。导致 JavaScript 中的对象系统十分混乱，这意味着 JavaScript 开发者对 **原型(prototypes)和类(classes)** 都要了解。

<!-- more -->

## 类继承和原型继承的区别是什么？

这可能是一个棘手的问题，你可能需要后续的问答来完善这个答案，因此要特别注意了解它们的差异，以及如何利用这些知识来写出更好的代码。

**类继承：类相当于蓝图--对将要创建的对象的描述。**

实例通常是使用构建函数和 `new` 关键字来创建的。ES6 中新增了 `class` 关键字可以使用。从技术上来说，像 Java 中的类这个概念在 JavaScript 中是不存在的。而是使用构造函数。ES6 中的 `class` 关键字就是构造函数的语法糖：

```js
class Foo {}
typeof Foo // 'function'
```

在 JavaScript 中，类继承的实现建立在原型继承之上，但这并不意味着它们做了同样的事情：

JavaScript 类继承是使用原型链链接子 `Constructor.prototype` 和父  `Constructor.prototype` 的委托关系。通常，`super()`构造器也会被调用。这种机制，形成了**单一继承结构**，并且创建了**面向对象设计的最紧密耦合行为**。

> *“Classes inherit from classes and create subclass relationships: hierarchical class taxonomies.”*

> **Prototypal Inheritance: A prototype is a working object instance.** Objects inherit directly from other objects.

原型继承模式下，对象实例可以由多个对象源组成，这样使继承更灵活且 [[Prototype]] 委托继承层次浅。换句话说，基于原型继承的面向对象设计不会产生层级分类这样的副作用--这是决定性的区别。

JavaScript 中的实例通常是通过构造函数，对象字面量或 `Object.create()` 来创建。

> *“A prototype is a working object instance. Objects inherit directly from other objects.”*

## 为什么这个问题很重要？

继承是代码重用机制的根本：不同对象分享代码的方式。分享代码的方式的重要性是因为如果你弄错了，会产生很多问题，特别是：

**类继承产生的 parent/child 对象分类是一个副作用。**

这些分类几乎不可能适用于所有的新实例，并且基类的广泛使用导致了**脆弱的基类问题**，这导致了修复 bug 的难度。事实上，类继承在面向对象设计中引起了许多众所周知的问题：

+ **紧耦合问题** (类继承是面向对象设计中耦合度最高的)，这导致了下一个问题

+ 脆弱基类问题

+ 不灵活的层级问题 (新实例最终会导致所有的类都是错误的)

+ 必要的重复问题 (由于层次机构不灵活，新的实例通常是通过复制而不是调整现有代码来实现)

+ 猩猩/香蕉问题 (你需要的是一个香蕉，但是得到的是一个拿着香蕉的猩猩以及整个丛林)

我在一个演讲中深入讨论过这其中的一些问题：“Classical Inheritance is Obsolete: How to Think in Prototypal OO”：

{% youtube lKCCZTUx0sI %}

解决所有问题的方法是选择对象组合而不是类继承。

> *“Favor object composition over class inheritance.”*
> *~ The Gang of Four, [“Design Patterns: Elements of Reusable Object Oriented Software”](http://www.amazon.com/gp/product/0201633612?ie=UTF8&camp=213733&creative=393185&creativeASIN=0201633612&linkCode=shr&tag=eejs-20&linkId=WMUILDJNIUXY4NSH)*

总结：

{% youtube wfMtDGfHWpA %}

## 是不是所有的继承都有问题？

人们说“优先选择对象组合而不是继承”的时候，其实是要表达“优先选择对象组合而不是类继承”(引用自 “Design Patterns” 的原文)。这是面向对象设计的常识。因为类继承有许多缺陷并会导致许多问题。

当人们谈论类继承时，人们通常会忽略 **class** 这个单词，这看起来好像所有的继承都有问题--但事实并非如此。

继承是有不同种类的，并且大部分的优秀的。

## 原型继承的三种方式

在我们深入研究其他类型的继承之前，让我们仔细观察一下类继承的含义：

{% gist fe1d8dfd036b43811c16d087d2e7fb8c class-inheritance-example.js %}

`BassAmp` 继承于 `GuitarAmp`， `ChannelStrip` 继承于 `BassAmp` 和 `GuitarAmp`。这是面向对象设计的一个错误示范。channel strip 并不是 guitar amp 的一种，而且它根本不需要 cabinet 这个属性。一个比较好的方法是创建一个新的基类给 amps 和 channel strip 继承，但是这种方法依然有局限。

最终，新的基类策略也会失效。

更好的方法是，可以使用对象组合继承你真正需要的东西：

{% gist d1997b0efa3119ec583ad6ff40da79d9 composition-example.js %}

认真看这段代码，你就会发现：通过对象组合，我们可以确切地保证对象可以按需继承。这和类继承不同。当你继承于一个类，你会继承所有的属性，即使是你不需要的。

在这一点上，你可能会问自己，“这很好，但是原型在哪里呢？”

为了理解这一点，你必须了解有三种不同的基于原型的面向对象设计。

**拼接继承 (Concatenative inheritance)**：通过复制源对象的属性直接一个对象继承另一个对象的过程。在 JavaScript 中，源对象的属性通常被称作 **mixins**。从 ES6 开始，JavaScript 使用 `Object.assign()` 来实现这个过程。在 ES6 之前，通常使用 **Underscore/Lodash** 的 `.extend()` 和 **jQuery** 的 `$.extend()` 等来实现。上面的对象组合的例子使用了连接继承。

**原型委托 (Prototype delegation)**：在 JavaScript 中，对象有自己委托的原型，这个原型也有自己委托的原型，以此类推一直到 `Object.prototype` 作为根原型，这样就形成了一个原型链。如果一个在对象中找不到的属性，可以沿着原型链一直查找。当你使用 `new` 创建实例以及 `Constructor.prototype` 连接到这个实例形成链接。你也可以使用 `Object.create()` 来实现，或者与拼接混用，从而可以把多个原型简化为单一委托，或者在对象创建后进行扩展。

**函数继承 (Functional inheritance)**：在 JavaScript 中，任何函数都可以创建对象。如果这个函数既不是构造函数也不是 `class`，那就是**工厂函数 (factory function)**。函数继承的原理是通过工厂函数创建对象并通过直接赋予属性(使用连接继承)。Douglas Crockford 创造了这个术语，但在 JavaScript 中已经广泛使用了。

现在你会意识到，拼接继承是在 JavaScript 中实现对象组合的秘诀，这使得原型委托和函数继承更加有趣。

大多数人提到 JavaScript 的面向对象设计时，首先想到的是原型委托。现在你会发现他们错过了很多。原型委托不是类继承的最佳替换，对象组合才是。

## 为什么对象组合能改避免脆弱基类问题

要搞清脆弱基类这个问题，首先你要理解这个问题是如何形成的：

1. `A` 是基类
2. `B` 继承于 `A`
3. `C` 继承于 `B`
4. `D` 继承于 `B`

`C` 调用 `super`， 会执行 `B` 中的代码，`B` 调用 `super`， 会执行 `A` 中的代码。

`A` 和 `B` 中包含了 `C` 和 `D` 不需要的特性。 `D` 是一个新实例， `C` 和 `D` 在 `A` 初始化的代码有少许不同。所以萌新开发者会去调整 `A` 的初始化代码。由于依赖于之前 `A` 中的代码被修改了，尽管 `D` 正常工作了，但是 `C` 被破坏了。

我们有不同的方式可以从 `A` 和 `B` 中得到 `C` 和 `D` 需要的属性。`C` 和 `D` 并不需要 `A` 和 `B` 的所有特性。它们只想继承一些已经定义在 `A` 和 `B` 中的属性。但是通过 `super` 来实现继承，这不是选择性的继承而是继承所有的属性。

> *“…the problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. **You wanted a banana but what you got was a gorilla holding the banana** and the entire jungle.” ~ Joe Armstrong — “Coders at Work”*

### 使用组合 (Composition)

想象下你拥有的是特性 (features) 而不是类 (classes)：

```js
feat1, feat2, feat3, feat4
```

`C` 需要 `feat1` 和 `feat3`， `D` 需要 `feat1`，`feat2`，`feat4`

```js
const C = compose(feat1, feat3)
const D = compose(feat1, feat2, feat4)
```

现在你发现，`D` 需要 `feat1` 的行为有些许不同。这并不需要改变 `feat1`，而是创造一个自定义的 `feat1` 并且使用它。不需要改变 `feat2` 和 `feat4`。

```js
const D = compose(custom1, feat2, feat4)
```

`C` 不会受到影响。

类继承无法实现这一点的原因是，当你使用类继承时，你得到的是类这个整体。

如果你为了适配新的实例，要么复制现有类的一部分（必然性重复问题），要么重构依赖于现有类的所有内容使得修改有的类适配于新的实例，这会导致脆弱基类问题。

对象组合可以避免这两个问题。

### 你自认为了解原型，但是...

如果你的所学是，构建类或者构造函数而不是原型继承。那么你学到的是使用原型来模仿类继承。了解更多 -- [Common Misconceptions About Inheritance in JavaScript](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a)

在 JavaScript 中，长久以来类继承寄生在非常丰富灵活的原生的原型继承之上， ES6 以来的 `class` 也是一样，当你使用类继承，原型继承的强大能力和灵活性都不能得到应用。事实上，你正在把自己圈在角落并且选择所有的类继承和它所带来的问题。

> *Using class inheritance in JavaScript is like driving your new Tesla Model S to the dealer and trading it in for a rusted out 1983 Ford Pinto.*

## Stamps：可组合的工厂函数

大多数情况下，通过多个工厂函数实现对象组合：工厂函数是用来创建对象的。如果工厂函数也可以组合呢？它被称作 (Stamps) -- [The Stamp Specification](https://github.com/stampit-org/stamp-specification)

## 探索 'Master the JavaScript Interview' 系列

+ [What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36)

+ [What is the Difference Between Class and Prototypal Inheritance?](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9)

+ [What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

+ [What is a Function Composition?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)

+ [What is a Function Programming?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)

+ [What is a Promise?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)

+ [Soft Skills](https://medium.com/javascript-scene/master-the-javascript-interview-soft-skills-a8a5fb02c466)




