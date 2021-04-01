title: "[译] 实例学习 js 函数式编程（1）"
permalink: "functional-programming-in-js-with-practical-examples-part-1"
tags: [js, fp]
date: 2017-01-12 21:00:00
---

> 原文链接：<https://medium.com/@rajaraodv/functional-programming-in-js-with-practical-examples-part-1-87c2b0dbc276#d6qbf6y5s>

函数式编程(FP) 可以改进你的编程方式。 但是它很难学，而且很多文章和教程并没有过多地深入细节，如 单子 (Monads) , 加强版函子 (Applicative) 等，并没有也没有提供实例来帮助我们在日常中使用强大的函数式编程技术。这便是我想要写一篇能让使用用函数式编程技术变得更简单的文章。

> 请注意：这篇文章的重点不是为什么 某某特性 是必须的(WHY)，而是 某某特性 是什么(WHAT)。

** 在第一章中，你会通过一些实例了解到一些函数式编程基础概念，如函数柯里化 (Currying)，纯函数，"Fantasy-land"规范，函子 ("Functors")，单子 ("Monads")，"Maybe Monads" (可空函子)和 "Either Monads" **


## 函数式编程

函数式编程是一种通过简单组合一系列函数来编写程序的方式。

本质上, FP 让我们把几乎所有东西包装成函数的形式, 编写许多小的可复用的函数, 并简单地一个接一个地调用它们，就像这样：(func1.func2.func3) 或者 compose 的形式: func1(func2(func3()))。

但是为了真正地把程序写成这种形式，函数们需要遵循一些规则并客服一些挑战，比如下面这些：

## 函数式编程的挑战

如果万物都能通过函数的组合完成，那么：

1. 我们如何处理分支结构(if-else condition)? (提示: "Either" Monad)
2. 我们如何处理空指针异常？（提示: "Maybe" Monad）
3. 我们怎样确保函数是真正可复用的并且能在任何地方使用？（提示: 纯函数(Pure functions), 引用透明性(referential transparency)
4. 怎样确定我们传入的数据没有被函数改变，所以我们可以在其他地方复用这些数据？（提示：纯函数，不变性(immutability)）
5. 如果一个函数接收多个参数但是链接 (chaining) 只能接收一个参数，我们怎样能使它成为一个 chain 的一部分? (提示：柯里化 和 高阶函数 ("higher-order functions") )
6. 等等


## 函数式编程的解决方案

为了解决这些挑战，完全的函数式编程语言比如 Haskell 原生提供了几个来自数学的工具和概念 比如 "Monads", "Functors" 等。

然而 JavaScript并没有原生提供这些工具，幸运的是它拥有足够的 FP 特性能让人民编写库。


## Fantasy-Land 规范 和 FP 库

提供 Functors, Monads 等的库, 需要实现遵循某个规范的函数/类 来提供类似 Haskll 的函数性。

[Fantasy-Land](https://github.com/fantasyland/fantasy-land) 是其中一个突出的规范，它解释了 每个 js 函数/类 的行为。

![](https://cdn-images-1.medium.com/max/1600/1*fwhU3xa-92GSWXCfUg2PKg.png)

上图展示了所有的规范和他们的依赖。规范本质上式一种规则，类似于 Java 中的 接口。从 JavaScript 的角度来看，你可以把规范理解为：一些类或者构造函数 实现了某些方法比如(map, of, chain 等)，并遵循这个规范。

例如：

一个 JS 类是一个 "Functor" 只要它实现了 "map" 方法。并且这个 map 方法正如规范中规定地那样运行(ps: 这只是一个简化的例子，实际中会有更多规则)。

类似的，一个 JS 类只要遵循规范实现了 "map" 和 "ap" 方法，它就是一个 "Apply Functor"

类似的，一个 JS 类只要实现了"Functor", "Apply", "Applicative", "Chain" 和 "Monad" 它自身（由于 chain 的依赖）的规范，它就是一个 "Monad"(也叫 Monad Functor)。

注意: 依赖可能看起来像是继承但这并不是必须的。 例如: Monad 实现了 Applicative 和 Chain 规范(除了其它的)。

## Fantasy-Land Spec
