---
title: 【译】JavaScript 工作原理：内存管理+如何处理4个常见的内存泄漏
tags:
  - JavaScript
abbrlink: 977269ff
---

> [How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

几个星期之前，我们开始了一个旨在深入挖掘 JavaScript 及其实际工作原理的系列文章，我们认为，通过了解 JavaScript 的底层构建以及它们是如何协作的，你将能够编写更好的代码和应用。

本系列的第一篇文章重点介绍了[引擎，运行时和调用栈的概述](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)。第二篇文章仔细研究了[Google V8 JavaScript 引擎的内部区块](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)并提供了一些关于如何编写更好的JavaScript代码的技巧。

在第三篇文章中，我们将讨论由于编程语言日益成熟和复杂性日益增加而被开发人员忽视的另一个重要主题 —— 内存管理。我们还将提供一些关于如何处理 JavaScript 中的内存泄漏的提示，我们在 [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=Post-3-v8-intro) 中遵守这些提示，因为我们需要确保 SessionStack 不会导致内存泄漏或者不会增加我们集成的Web应用程序的内存消耗。

<!-- more -->

## 概述 (Overview)

像C这样的语言具有底层内存管理原函数，例如 `malloc()` 和 `free()` 。开发人员使用这些原函数来明确地从操作系统分配和释放内存。

同时，JavaScript在创建事物（对象，字符串等）时分配内存，并在不再使用时“自动”释放它，这个过程称为*垃圾回收*。这种看似“自动”的释放资源的本质是混乱的根源，并给JavaScript（和其他高级语言）开发人员提供了错误的印象，他们可以选择不关心内存管理。**这是一个大错误。**

即使使用高级语言，开发人员也应该了解内存管理（或至少是基础知识）。有时自动内存管理存在问题（例如垃圾收集器中的错误或实现限制等），开发人员必须了解这些问题才能正确处理它们（或找到合适的解决方法，并尽量减少折衷和代码影响）。

## 内存生命周期 (Memory life cycle)

无论你使用何种编程语言，内存生命周期几乎都是一样的：

![Memory life cycle](https://cdn-images-1.medium.com/max/2000/1*slxXgq_TO38TgtoKpWa_jQ.png)

以下周期的每个步骤的概述：

+ **分配内存** —— 内存由操作系统分配，允许程序使用它。在低级语言（例如C）中，这是你作为开发人员应该处理的显式操作。但是，在高级语言中，语言帮你完成。

+ **使用内存** —— 这是你的程序实际使用之前分配的内存的时候。当你在代码中使用分配的变量时，进行**读写**操作。。

+ **释放内存** —— 现在是时候释放你不需要的整个内存，以便它可以空闲可用。与**分配内存**操作一样，这个操作在低级语言中是显式操作。

关于调用栈和内存堆的概念的快速概览，你可以阅读我们关于该主题的[第一篇文章](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)。

## 什么是内存 (What is memory)

在直接跳转到关于 JavaScript 内存之前，我们将简要地讨论一下内存是什么以及它是如何工作的。

在硬件级别上，计算机内存包含大量[触发器](https://en.wikipedia.org/wiki/Flip-flop_%28electronics%29)。每个触发器包含几个晶体管并且能够存储一位。单个触发器可通过唯一标识符进行寻址，因此我们可以读取和覆盖它们。因此，从概念上讲，我们可以将整个计算机内存视为我们可以读写的一个巨大的位数组。

因为作为人类，我们并不善于在位操作中完成所有的思考和算术，我们将它们组织成更大的组，它们可以用来表示数字。8位 (bits) 称为1字节 (byte) 。超出字节，有单词（有时是16，有时是32位）。

很多东西都存储在内存中：

1. 所有程序使用的所有变量和其他数据。

2. 程序的代码，包括操作系统。

编译器和操作系统协同工作，为您处理大部分内存管理，但我们建议您先了解一下底层发生了什么。

编译代码时，编译器可以检查原始数据类型并提前计算它们需要多少内存。然后将所需的数量分配给调用**栈空间**中的程序。分配这些变量的空间称为栈空间，因为在调用函数时，它们的内存会添加到现有内存之上。当它们终止时，它们将以 LIFO（后进先出）顺序被移除。例如，请考虑以下声明：

```C
int n; // 4 bytes
int x[4]; // array of 4 elements, each 4 bytes
double m; // 8 bytes
```

编译器可以立即看到代码所需 
4 + 4×4 + 8 = 28字节。

> 这就是当前整数和双精度的大小。大约20年前，整数通常是2个字节，双精度4个字节。您的代码永远不应依赖于此时基本数据类型的大小。

编译器将插入将与操作系统交互的代码，以请求堆栈上必要的字节数，以便存储变量。

在上面的示例中，编译器知道每个变量的确切内存地址。实际上，每当我们写入变量 `n` 时，它就会在内部转换为“内存地址4127963”。

请注意，如果我们在这里尝试访问 `x[4]` ，我们将访问与 `m` 相关的数据。那是因为我们正在访问数组中不存在的元素 - 它比数组中最后一个实际分配的元素 `x[3]` 多4个字节，并且可能最终读取（或覆盖）某些 `m` 的位。这几乎肯定会其余部分产生非常不利的后果。

![m memory](https://cdn-images-1.medium.com/max/1600/1*5aBou4onl1B8xlgwoGTDOg.png)

当函数调用其他函数时，每个函数在调用时都会获得自己的栈块。它将所有局部变量保存在那里，还有一个可以记住它在执行中的位置的程序计数器。当函数完成时，其内存块再次可用于其他目的。

## 动态分配 (Dynamic allocation)

不幸的是，当我们在编译时不知道变量需要多少内存时，事情就不那么容易了。假设我们想要执行以下操作：

```C
int n = readInput(); // reads input from the user
...
// create an array with "n" elements
```

这里，在编译时，编译器不知道数组需要多少内存，因为它由用户提供的值确定。

因此，它无法为堆栈上的变量分配空间。相反，我们的程序需要在运行时明确询问操作系统是否有适当的空间量。此内存是从**堆空间**分配的。静态和动态内存分配之间的差异总结在下表中：

![Differences between statically and dynamically allocated memory](https://cdn-images-1.medium.com/max/1600/1*qY-yRQWGI-DLS3zRHYHm9A.png)

要完全理解动态内存分配的工作原理，我们需要花更多时间在**指针**上，这可能与本文的主题有点过多的偏差。如果您有兴趣了解更多信息，请在评论中告诉我们，我们可以在以后的帖子中详细介绍指针。

## JavaScript 中的内存分配 (Allocation in JavaScript)

现在我们将解释（分配内存）如何在 JavaScript 中工作的第一步。

JavaScript 使开发人员免于处理内存分配的责任 - JavaScript自行完成，同时声明值。

```js
var n = 374; // allocates memory for a number
var s = 'sessionstack'; // allocates memory for a string 
var o = {
  a: 1,
  b: null
}; // allocates memory for an object and its contained values
var a = [1, null, 'str'];  // (like object) allocates memory for the
                           // array and its contained values
function f(a) {
  return a + 3;
} // allocates a function (which is a callable object)
// function expressions also allocate an object
someElement.addEventListener('click', function() {
  someElement.style.backgroundColor = 'blue';
}, false);
```

一些函数调用也会导致对象分配：

```js
var d = new Date(); // allocates a Date object
var e = document.createElement('div'); // allocates a DOM element
```

方法可以分配新的值或对象：

```js
var s1 = 'sessionstack';
var s2 = s1.substr(0, 3); // s2 is a new string
// Since strings are immutable, 
// JavaScript may decide to not allocate memory, 
// but just store the [0, 3] range.
var a1 = ['str1', 'str2'];
var a2 = ['str3', 'str4'];
var a3 = a1.concat(a2); 
// new array with 4 elements being
// the concatenation of a1 and a2 elements
```

## JavaScript 中使用内存 (Using memory in JavaScript)

基本上在 JavaScript 中使用分配的内存意味着在读取和写入。

这可以通过读取或写入变量或对象属性的值，甚至将参数传递给函数来完成。

## 释放不再引用的内存 (Release when the memory is not needed anymore)

大多数内存管理问题都出现在这个阶段。

这里最艰难的任务是弄清楚何时不再需要分配的内存。它通常要求开发人员确定程序中的哪个位置不再需要这些内存并释放它。

高级语言嵌入了一个名为垃圾收集器的软件，其工作是跟踪内存分配和使用，以便找到不再需要分配内存，在这种情况下，它将自动释放它。

不幸的是，这个过程是近似的，因为知道是否需要某个存储器的一般问题是[不可判定的](http://en.wikipedia.org/wiki/Decidability_%28logic%29)（不能通过算法解决）。

大多数垃圾收集器通过收集不能再访问的内存来工作，例如，指向它的所有变量都超出了作用域。然而，这是一个可以收集的内存空间集的低估，因为在任何一点上，内存位置可能仍然有一个在范围内指向它的变量，但它永远不会被再次访问。

## 垃圾回收机制 (Garbage collection)

由于发现某些内存是否“不再需要”这一事实是不可判定的，因此垃圾收集实现了对一般问题的解决方案的限制。本节将解释了解主要垃圾收集算法及其局限性的必要概念。

## 内存引用 (Memory references)

垃圾收集算法所依赖的主要概念是**引用**。

在内存管理的上下文中，如果前者具有对后者的访问权（可以是隐式的或显式的），则称该对象引用另一个对象。例如，JavaScript对象具有对其原型（**隐式引用**）及其属性值（**显式引用**）的引用。

在这种情况下，“对象”的概念被扩展到比常规 JavaScript 对象更广泛的东西，并且还包含函数作用域（或全局词法作用域）。

> 词法作用域定义了如何在嵌套函数中解析变量名称：内部函数包含父函数的范围，即使父函数已返回。

## 引用计数垃圾回收 (Reference-counting garbage collection)

这是最简单的垃圾回收算法。如果指向它的零引用，则该对象被视为“垃圾收集”。

看一下下面的代码：

```js
var o1 = {
  o2: {
    x: 1
  }
};
// 2 objects are created. 
// 'o2' is referenced by 'o1' object as one of its properties.
// None can be garbage-collected

var o3 = o1; // the 'o3' variable is the second thing that 
            // has a reference to the object pointed by 'o1'. 
                                                       
o1 = 1;      // now, the object that was originally in 'o1' has a         
            // single reference, embodied by the 'o3' variable

var o4 = o3.o2; // reference to 'o2' property of the object.
                // This object has now 2 references: one as
                // a property. 
                // The other as the 'o4' variable

o3 = '374'; // The object that was originally in 'o1' has now zero
            // references to it. 
            // It can be garbage-collected.
            // However, what was its 'o2' property is still
            // referenced by the 'o4' variable, so it cannot be
            // freed.

o4 = null; // what was the 'o2' property of the object originally in
           // 'o1' has zero references to it. 
           // It can be garbage collected.
```

## 循环引用产生问题 (Cycles are creating problems)

在循环引用方面存在限制。在以下示例中，创建了两个对象并相互引用，从而创建了一个循环引用。在函数调用之后它们将超出范围，因此它们实际上是无用的并且可以被释放。但是，引用计数算法认为，由于两个对象中的每一个至少被引用一次，因此两者都不能被垃圾收集。

```js
function f() {
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1 references o2
  o2.p = o1; // o2 references o1. This creates a cycle.
}

f();
```

![cycle references](https://cdn-images-1.medium.com/max/1600/1*GF3p99CQPZkX3UkgyVKSHw.png)

## 标记扫描算法 (Mark-and-sweep algorithm)

为了确定是否需要对象，该算法确定对象是否可访问。

标记和扫描算法通过以下3个步骤：

1. 根节点：通常，根节点是在代码中引用的全局变量。例如，在JavaScript中，可以充当根节点的全局变量是 `window` 对象。Node.js中的对象称为 `global` 。垃圾收集器构建了所有根的完整列表。

2. 然后算法检查所有根节点和它们的孩子，并将它们标记为活动（意思是，它们不是垃圾）。 根节点无法访问的任何内容都将被标记为垃圾。

3. 最后，垃圾收集器释放所有未标记为活动的内存块，并将该内存返回给操作系统。

![A visualization of the mark and sweep algorithm in action](https://cdn-images-1.medium.com/max/1600/1*WVtok3BV0NgU95mpxk9CNg.gif)

此算法优于前一个算法，因为“对象具有零引用”导致此对象无法访问。正如我们在循环引用中看到的那样的情况正好相反。

截至2012年，所有现代浏览器都提供了标记 - 清除垃圾收集器。在过去几年中，在JavaScript垃圾收集（生成/增量/并发/并行垃圾收集）领域所做的所有改进都是该算法的实现改进（标记和清除），但不是对垃圾收集算法本身的改进，也不是对判断一个对象是否可访问这个目标的改进。

在这篇[文章](https://en.wikipedia.org/wiki/Tracing_garbage_collection)中，您可以更详细地阅读跟踪垃圾收集，其中还包括标记和清除及其优化。

## 循环引用不再是一个问题 (Cycles are not a problem anymore)

在上面的第一个示例中，在函数调用返回后，两个对象不再被全局对象中的某个变量引用。因此，垃圾收集器将认为无法访问它们。

![cycle references 2](https://cdn-images-1.medium.com/max/1600/1*FbbOG9mcqWZtNajjDO6SaA.png)

尽管对象之间存在引用，但它们无法从根目录访问。

## 垃圾收集器的直觉行为 (Counter intuitive behavior of Garbage Collectors)

虽然垃圾收集器很方便，但它们有自己的权衡策略。其中之一是不确定性。换句话说，GC 是不可预测的。您无法确定何时会执行回收。这意味着在某些情况下，程序会使用多于实际需要的内存。在其他情况下，在特别敏感的应用中，短暂停顿可能会很明显。尽管非确定性意味着无法确定何时执行回收，但大多数 GC 实现在分配期间执行回收的常见模式。如果没有执行分配，则大多数 GC 保持空闲。请考虑以下情形：

1. 执行大量分配。

2. 其中的大多数元素（或所有元素）都被标记为无法访问（假设我们将置空指向不再需要的缓存的引用）。

3. 没有进一步的分配。

在这种情况下，大多数 GC 不会再运行任何收集。换句话说，即使有可用于收集的无法访问的引用，收集器也不会声明这些引用。这些并非严格泄漏，但仍导致高于平常的内存使用率。

## 什么是内存泄漏 (What are memory leaks)

就像内存所描述的那样，内存泄漏是应用程序过去使用但不再需要但尚未返回操作系统或可用内存池的内存块。

![memory leak](https://cdn-images-1.medium.com/max/1600/1*0B-dAUOH7NrcCDP6GhKHQw.jpeg)

编程语言支持不同的内存管理方式。但是，是否使用某段内存实际上是一个[不可判定的问题](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#Release_when_the_memory_is_not_needed_anymore)。换句话说，只有开发人员才能明确是否可以将一块内存返回给操作系统。

某些编程语言提供的功能可帮助开发人员实现内存回收。其他人希望开发人员完全明确何时未使用内存。维基百科有关于[手动](https://en.wikipedia.org/wiki/Manual_memory_management)和[自动内存管理](https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29)的好文章。

## JavaScript 四种常见的内存泄漏 (The four types of common JavaScript leaks)

### 全局变量 (Global variables)

JavaScript 以一种有趣的方式处理未声明的变量：当引用未声明的变量时，会在全局对象中创建一个新变量。在浏览器中，全局对象将是 `window`，这意味着

```js
function foo(arg) {
    bar = "some text";
}
```

等同于

```js
function foo(arg) {
    window.bar = "some text";
}
```

假设 `bar` 仅是为了 `foo` 函数中引用的变量。但是，如果不使用 `var` 来声明它，则会创建冗余的全局变量。在上述情况下，这不会造成太大伤害。你肯定可以想象一个更具破坏性的场景。

您还可以使用 `this` 意外创建全局变量：

```js
function foo() {
    this.var1 = "potential accidental global";
}
// Foo called on its own, this points to the global object (window)
// rather than being undefined.
foo();
```

> 你可以通过在JavaScript文件的开头添加 `use strict` 来避免这一切，它将开启一种更严格的解析 JavaScript 模式，以防止意外创建全局变量。

意外的全局变量肯定是个问题，但是，通常情况下，您的代码会被显式的全局变量所侵扰，而这些变量根据定义无法被垃圾收集器收集。需要特别注意用于临时存储和处理大量信息的全局变量。如果必须使用全局变量来存储数据，当您这样做时，请确保在完成后将其指定为 `null` 或重新分配它。

### 被遗忘的定时器或回调 (Timers or callbacks that are forgotten)

我们以 `setInterval` 为例，因为它经常在JavaScript中使用。

提供观察者和其他接受回调的工具的库通常会确保一旦其实例也无法访问，所有对回调的引用都将无法访问。不过，下面的代码并不是一个罕见的发现：

```js
var serverData = loadData();
setInterval(function() {
    var renderer = document.getElementById('renderer');
    if(renderer) {
        renderer.innerHTML = JSON.stringify(serverData);
    }
}, 5000); //This will be executed every ~5 seconds.
```

上面的代码段显示了在不再需要引用的节点或数据上使用定时器的后果。

`renderer` 对象可能会在某些时候被替换或删除，这会使得间隔处理程序封装的块变得冗余。如果发生这种情况，则不会收集处理程序及其依赖项，因为需要首先停止间隔（请记住，它仍处于活动状态）。这一切都归结为这样一个事实，即无法收集确实存储和处理负载数据的 `serverData`。

使用观察者时，您需要确保在完成它们之后进行显式调用以删除它们（不再需要观察者，或者对象将无法访问）。

幸运的是，大多数现代浏览器都能为您完成这项工作：即使您忘记移除侦听器，一旦观察到的对象无法访问，它们也会自动收集观察者处理程序。过去，一些浏览器无法处理这些情况（旧的IE6）。

尽管如此，一旦对象过时，它仍然符合删除观察者的最佳实践。请参阅以下示例：

```js
var element = document.getElementById('launch-button');
var counter = 0;
function onClick(event) {
   counter++;
   element.innerHtml = 'text ' + counter;
}
element.addEventListener('click', onClick);
// Do stuff
element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
// Now when element goes out of scope,
// both element and onClick will be collected even in old browsers 
// that don't handle cycles well.
```

在使节点无法访问之前，您不再需要调用 `removeEventListener` ，因为现代浏览器支持可以检测这些循环并适当处理它们的垃圾收集器。

如果你使用 jQuery API（其他库和框架也支持它），您也可以在节点过时之前删除侦听器。即使应用程序在较旧的浏览器版本下运行，该库也将确保没有内存泄漏。

### 闭包 (Closures)

JavaScript 开发的一个关键方面是闭包：一个内部函数，可以访问外部（封闭）函数的变量。由于 JavaScript 运行时的实现细节，可能会以下列方式泄漏内存：

```js
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing) // a reference to 'originalThing'
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log("message");
    }
  };
};
setInterval(replaceThing, 1000);
```

一旦调用了 `replaceThing` ， `theThing` 将获得一个新对象，该对象由一个大数组和一个新的闭包 (`someMethod`) 组成。然而， `originalThing` 由一个由 `unused` 变量（从之前的 `replaceThing` 调用中的 `theThing` 变量）保存的闭包引用。需要记住的是，**一旦为同一父作用域中的闭包创建了闭包作用域，就会共享作用域**。

在个例子中， `someMethod` 创建的作用域与 `unused` 共享。 `unused` 包含一个关于 originalThing 的引用。即使 `unused` 从未被引用过， `someMethod` 也可以通过 `replaceThing` 作用域之外的 `theThing` 来使用它（例如全局的某个地方）。由于 `someMethod` 与 `unused` `共享闭包范围，unused` 指向 `originalThing` 的引用强制它保持活动状态（两个闭包之间的整个共享范围）。这阻止了它们的垃圾收集。

在上面的例子中，为闭包 `someMethod` 创建的作用域与 `unused` 共享，而 `unused` 又引用 `originalThing` 。 可以通过 `replaceThing` 的外部作用域 theThing 来使用`someMethod`，尽管 unused 从来没有被使用过。事实上未引用的 originalThing 依然保持活跃，因为 `someMethod` 的闭包与 `unused` 共享作用域。

所有这些都可能导致相当大的内存泄漏。当上述代码片段反复运行时，您可能会看到内存使用量激增。当垃圾收集器运行时，它的大小不会缩小。创建了一个闭包链（在这种情况下，它的根是 `theThing` 变量），每个闭包作用域都包含对大数组的间接引用。

这个问题是由Meteor团队发现的，他们有[一篇很棒的文章](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156)，详细描述了这个问题。

### 超出 DOM 的引用 (Out of DOM references)

在某些情况下，开发人员在数据结构中存储 DOM 节点。假设您要快速更新表中多行的内容。如果存储对字典或数组中每个 DOM 行的引用，则会有两个对同一 DOM 元素的引用：一个在DOM树中，另一个在字典中。如果您决定删除这些行，则需要记住使两个引用都无法访问。

```js
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image')
};
function doStuff() {
    elements.image.src = 'http://example.com/image_name.png';
}
function removeImage() {
    // The image is a direct child of the body element.
    document.body.removeChild(document.getElementById('image'));
    // At this point, we still have a reference to #button in the
    //global elements object. In other words, the button element is
    //still in memory and cannot be collected by the GC.
}
```

在引用 DOM 树内的内部或叶子节点时，还需要考虑其他因素。如果在代码中保留对表格单元格（<td>标记）的引用并决定从DOM中删除该表但保留对该特定单元格的引用，则可能会出现严重的内存泄漏。您可能认为垃圾收集器会释放除该单元之外的所有内容。然而，情况并非如此。由于单元格是表的子节点，并且子节点保持对其父节点的引用，**因此对表格单元格的单个引用将使整个表格保留在内存中**。

## 资源 (Resources)

- [http://www-bcf.usc.edu/~dkempe/CS104/08-29.pdf](http://www-bcf.usc.edu/~dkempe/CS104/08-29.pdf)

- [4 Types of Memory Leaks in JavaScript and How to Get Rid Of Them](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156?gi=39e62e0299c2)

- [An interesting kind of JavaScript memory leak](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)





