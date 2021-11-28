title: TLA+ 形式化验证入门指南
toc: false
mathjax: false
fancybox: false
tags: [Concurrent, TLA+]
date: 2021-08-01 11:54:03
weburl: TLA_Tutorial
categories: 软件之道

---

形式化验证（Formal Verification）指一类使用数理逻辑方法来证明软件设计是正确的技术，据称是由 Edsger Dijkstra 于 1972 年最早提出，此方法一直是一种比较小众冷门的技术。形式化验证技术想要解决的核心问题是：软件总是可能存在 Bug 的，而测试始终无法涵盖所有可能性，特别是对于并发系统及分布式系统来说，就算单元测试达到了 100% 分支覆盖率，也不能肯定的说这个系统在线程安全，一致性等方面不会出问题。那如何更好的来验证我们的程序是否符合预期呢？形式化验证就旨在使用严谨的数学证明方法来证明某一算法是正确的。这样我们就可以拍着胸脯说，我的算法肯定是正确的，都证明过了:)

<!--more-->

听上去是不是很牛逼啊，感觉我们马上就要能写出 bug free 的程序来了呢～然而理想很丰满，现实很骨感，实际问题远远不会是这么简单的，要是形式化验证真这么好用那它就不至于至今还这么小众了，事实上形式化验证存在着很多局限性与不 work 的时候的，这个后面再来细说。

关于形式化方法的实际应用及其强大之处可以进一步读读下面这篇布道文：

[Don’t Test, Verify —— 哪个故事真正符合你对形式化验证的想象？](https://secbit.io/blog/2018/10/24/formal-verification-background/)

当初也是因为偶然看了此文章知道了形式化验证这个东西，后面也陆续去深入了解学习了下，最近也用它解决了一些实际工作中的问题。本文就打算分享下入门学习的一些心得体会。

## TLA+ & PlusCal 简介及一些基本概念

进行形式化验证的具体工具有很多，目前实际软件开发中最为常用的是由 [Leslie Lamport](https://en.wikipedia.org/wiki/Leslie_Lamport "Leslie Lamport") 开发的 TLA+，这是一种用于形式化验证的语言，主要用于验证并行及分布式系统的正确性。

由于 TLA+ 写的代码并不是用来实际运行的，故一般将其代码称为模型（Model）而非程序（Program）。

TLA+ 是基于数理逻辑而非经典的软件开发思想设计出来的，故其代码与其他编程语言有着显著区别，其中的基本元素是集合，逻辑运算，映射等东西，来个例子感受下：

```tla+
Next == \/ \E b \in Ballots : Phase1a(b) \/ Phase2a(b)
        \/ \E a \in Acceptors : Phase1b(a) \/ Phase2b(a) 
```

这段代码看上去完全不像在编程，实际上写 TLA+ 代码的确也不是在编程而是在用数理逻辑定义一些东西。

这学习曲线对于大部分码农来说实在是太过于陡峭了，Programer 并不是数学家，Lamport 大神也知道这一点，于是他又搞了个叫 PlusCal 的东西出来。PlusCal 是一种类似 C/Pascal 的高级语言，其目的同样不是为了生成机器代码来运行，而是依靠 TLA+ 解释器来生成对应的 TLA+ 模型代码。

来一段实际的 PlusCal 代码感受下：

```pluscal
(* --algorithm EuclidAlg {
variables alice_account = 10, bob_account = 10, money = 5;

A: alice_account := alice_account - money;
B: bob_account := bob_account + money;

} *)
```

这看上去就很像经典编程语言了，因此对于程序员来说，可以使用 PlusCal 来快速进行形式化验证。不过 PlusCal 毕竟是 TLA+ 的上层高级语言，其能实现的功能只是 TLA+ 的一个子集，不过一般来说此问题不大，这个子集对于简单应用来说足够用了。

有了代码后如何运行 TLA+ 或 PlusCal 模型呢，Lamport 为此开发了一个 IDE，即 [TLA Toolbox](https://lamport.azurewebsites.net/tla/toolbox.html). 然而此 IDE UI 界面并不是很好用，更建议使用 VSCode 中的 [TLA 插件](https://marketplace.visualstudio.com/items?itemName=alygin.vscode-tlaplus) 来进行开发。

## 入门学习路径及资料

入门学习建议从下面这个教程开始：

> [Learn TLA+](https://learntla.com/)

此教程完全从实用角度出发，立足点是如何用 PlusCal 来解决日常编程中需要关注的并发，一致性等问题，因此十分简单易学，也比较短，看完后基本就能实际上手做些事情了。

在实际写 PlusCal 代码的时候需要参考下其语法手册，PlusCal 有两种语法风格，类似 Pascal 的 P-Syntax 及类似 C 语言的 C-Syntax，语法手册分别如下：

> [A PlusCal User’s Manual C-Syntax Version 1.8](https://lamport.azurewebsites.net/tla/c-manual.pdf)
> [A PlusCal User’s Manual P-Syntax Version 1.8](https://lamport.azurewebsites.net/tla/p-manual.pdf)

网上的例子中使用 P-Syntax 的居多，不过我个人更喜欢 C-Syntax 一些。

如果看完上述简单教程后还想进一步系统的学习一下，那建议从 Lamport 的 TLA+ 项目主页开始：

> [The TLA+ Home Page](https://lamport.azurewebsites.net/tla/tla.html)

此外 Lamport 还有一本系统的讲形式化验证的书：

> [Specifying Systems](http://lamport.azurewebsites.net/tla/book.html)

观千剑而后识器，看看其他人是如何写代码的对于入门来说也是很有用的，下面这两个 Github 项目中收集整理了很多 TLA+ 模型，如果想要提高水平可以仔细学习揣摩下：

> [TLA+ Examples](https://github.com/tlaplus/Examples)
> [Dr. TLA+ Series](https://github.com/tlaplus/DrTLAPlus)

## 一些关键点及 TLA+ 的局限性

### 定义什么是正确

形式化验证是用来验证算法是正确的，那什么叫“正确”呢？**如何定义“正确”是形式化验证中最重要的问题之一**。比较符合程序员习惯的方法是在 PlusCal 中加入 `assert` 来检查是否满足某些条件。不过更好的方法是使用不变量（Invariants）检查，如何正确的定义算法中需要检查的 Invariant 是十分重要的，如果检查条件的定义本身就是不完备的，那形式化验证的结果自然也是不完备的。

### 合理定义原子操作

PlusCal 中使用 Label 来定义原子操作，一个 Label 下若干条语句会被视为是一个原子操作，如果把本来不是原子操作的行为错误的定义为了原子操作，那最终得到的结果显然就会是不完备的。

如果把本来可以视为一个原子操作的行为定义为若干条原子操作，则会让验证的计算量大幅增加，导致验证所需时间变长。PlusCal 翻译成 TLA+ 后验证原理是穷举不同进程间执行时序的所有可能性，若原子操作或分支过多，会造成解空间的急剧膨胀。

### 局限性

1. TLA+ 并不是直接去验证算法的实现，而是验证算法实现抽象出来的 PlusCal 或 TLA+ 模型，这一步抽象的正确性只能由人工自行保证，没有任何方法可以证明二者是等价的。事实上二者绝大多数时候也是不等价的，比如程序中的数字都是会溢出的，而数学模型中的数字则不会。妄图对程序所有实现细节都去建模验证的尝试也是不可行的，因为这会导致验证的解空间变得极为巨大，基本上都是没有实际意义的。估计这也就是 Lamport 为什么不设计一个 C / Java 等语言直接翻译为 PlusCal 模型解释器的原因。
2. 如前文所述，算法正确性的定义也是需要人工完成的，这一步某些时候也是比较困难的，精确的定义什么是正确的本身就很有挑战性。
3. 需要人工建模也带来了软件开发过程中成倍的工作量增加，特别是在软件需要快速迭代开发时形式化验证方法基本是不可用的，因此实际上形式化验证手段一般也只用于一些变化很小，且开发周期很长的项目中。
