title: C++ 中的 volatile，atomic 及 memory barrier
toc: false
mathjax: false
fancybox: false
tags: [C++, Compiler, Concurrent, x86, ARM]
date: 2020-09-11 16:46:28
weburl: Cpp_Volatile_Atomic_Memory_barrier
categories: 编程之法

---

C++ 中的 `volatile` 关键字，`std::atomic` 变量及手动插入内存屏障指令（Memory Barrier）均是为了避免内存访问过程中出现一些不符合预期的行为。这三者的作用有些相似之处，不过显然它们并不相同，本文就将对这三者的应用场景做一总结。

<!--more-->

这三者应用场景的区别可以用一张表来概括：

|                | `volatile` | Memory Barrier | `atomic` |
| -------------- | ---------- | -------------- | -------- |
| 抑制编译器重排 | Yes        | Yes            | Yes      |
| 抑制编译器优化 | Yes        | No             | Yes      |
| 抑制 CPU 乱序  | No         | Yes            | Yes      |
| 保证访问原子性 | No         | No             | Yes      |

下面来具体看一下每一条。

### 抑制编译器重排

所谓编译器重排，这里是指编译器在生成目标代码的过程中交换没有依赖关系的内存访问顺序的行为。

比如以下代码：

``` C++
*p_a = a;
b = *p_b;
```

编译器**不保证**在最终生成的汇编代码中对 `p_a` 内存的写入在对 `p_b` 内存的读取之前。

如果这个顺序是有意义的，就需要用一些手段来保证编译器不会进行错误的优化。具体来说可以通过以下三种方式来实现：

- 把对应的变量声明为 `volatile` 的，C++ 标准保证对 `volatile` 变量间的访问编译器不会进行重排，不过仅仅是  `volatile` 变量之间， `volatile` 变量和其他变量间还是有可能会重排的；
- 在需要的地方手动添加合适的 Memory Barrier 指令，Memory Barrier 指令的语义保证了编译器不会进行错误的重排操作；
- 把对应变量声明为 `atomic` 的， 与 `volatile` 类似，C++ 标准也保证 `atomic` 变量间的访问编译器不会进行重排。不过 C++ 中不存在所谓的 "atomic pointer" 这种东西，如果需要对某个确定的地址进行 atomic 操作，需要靠一些技巧性的手段来实现，比如在那个地址上进行 placement new 操作强制生成一个 `atomic` 等；

### 抑制编译器优化

此处的编译器优化特指编译器不生成其认为无意义的内存访问代码的优化行为，比如如下代码：

```C++
void f() {
  int a = 0;
  for (int i = 0; i < 1000; ++i) {
    a += i;
  }
}
```

在较高优化级别下对变量 `a` 的内存访问基本都会被优化掉，`f()` 生成的汇编代码和一个空函数基本差不多。然而如果对 `a` 循环若干次的内存访问是有意义的，则需要做一些修改来抑制编译器的此优化行为。可以把对应变量声明为 `volatile` 或 `atomic` 的来实现此目的，C++ 标准保证对 `volatile` 或 `atomic` 内存的访问肯定会发生，不会被优化掉。

不过需要注意的是，这时候手动添加内存屏障指令是没有意义的，在上述代码的 `for` 循环中加入 `mfence` 指令后，仅仅是让循环没有被优化掉，然而每次循环中对变量 `a` 的赋值依然会被优化掉，结果就是连续执行了 1000 次 `mfence`。

### 抑制 CPU 乱序

上面说到了编译器重排，那没有了编译器重排内存访问就会严格按照我们代码中的顺序执行了么？非也！现代 CPU 中的诸多特性均会影响这一行为。对于不同架构的 CPU 来说，其保证的内存存储模型是不一样的，比如 x86_64 就是所谓的 TSO（完全存储定序）模型，而很多 ARM 则是 RMO（宽松存储模型）。再加上多核间 Cache 一致性问题，多线程编程时会面临更多的挑战。

为了解决这些问题，从根本上来说只有通过插入所谓的 Memory Barrier 内存屏障指令来解决，这些指令会使得 CPU 保证特定的内存访问序及内存写入操作在多核间的可见性。然而由于不同处理器架构间的内存模型和具体 Memory Barrier 指令均不相同，需要在什么位置添加哪条指令并不具有通用性，因此 C++ 11 在此基础上做了一层抽象，引入了 `atomic` 类型及 Memory Order 的概念，有助于写出更通用的代码。从本质上看就是靠编译器来根据代码中指定的高层次 Memory Order 来自动选择是否需要插入特定处理器架构上低层次的内存屏障指令。

关于 Memory Order，内存模型，内存屏障等东西的原理和具体使用方法网上已经有很多写得不错的文章了，可以参考文末的几篇参考资料。

### 保证访问原子性

所谓访问原子性就是 Read，Write 操作是否存在中间状态，具体如何实现原子性的访问与处理器指令集有很大关系，如果处理器本身就支持某些原子操作指令，如 Atomic Store， Atomic Load，Atomic Fetch Add，Atomic Compare And Swap（CAS）等，那只需要在代码生成时选择合适的指令即可，否则需要依赖锁来实现。C++ 中提供的可移植通用方法就是 `std::atomic`，`volatile` 及 Memory Barrier 均与此完全无关。

### 总结

从上面的比较中可以看出，`volatile`，`atomic` 及 Memory Barrier 的适用范围还是比较好区分的。

- 如果需要原子性的访问支持，只能选择 `atomic`；
- 如果仅仅只是需要保证内存访问不会被编译器优化掉，优先考虑 `volatile`；
- 如果需要保证 Memory Order，也优先考虑 `atomic`，只有当不需要保证原子性，而且很明确要在哪插入内存屏障时才考虑手动插入 Memory Barrier。



> 参考资料：
>
> [内存模型与c++中的memory order](https://www.cnblogs.com/ishen/p/13200838.html)
>
> [volatile与内存屏障总结](https://zhuanlan.zhihu.com/p/43526907)
>
> [X86/GCC memory fence的一些见解](https://zhuanlan.zhihu.com/p/41872203)