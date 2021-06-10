title: 使用 perf 进行性能分析时如何获取准确的调用栈
weburl: perf_stack_traceback
toc: false
mathjax: false
fancybox: false
tags: [Linux, x86, Debug]
categories: 工具之术
date: 2019-10-30 23:09:37

----------

`perf` 是 Linux 下重要的性能分析工具，`perf` 可以通过采样获取很多性能指标，其中最常用的是获取 CPU Cycles，即程序各部分代码运行所需的时间，进而确定性能瓶颈在哪。不过在实际使用过程中发现，简单的使用`perf record -g` 获取到的调用栈是有问题的，存在大量 `[Unknown]` 函数，从 `perf report` 的结果来看这些部分对应地址大部分都是非法地址，且生成的火焰图中存在很多明显与代码矛盾的调用关系。

<!--more-->

最初怀疑是优化级别的问题，然而尝试使用 `Og` 或 `O0` 优化依然存在此问题，仔细阅读 `perf record` 的手册后发现，`perf` 同时支持 3 种栈回溯方式：`fp`, `dwarf`, `lbr`，可以通过 `--call-graph` 参数指定，而 `-g` 就相当于 `--call-graph fp`.

### 栈回溯方式

 `fp` 就是 Frame Pointer，即 x86 中的 `EBP` 寄存器，`fp` 指向当前栈帧栈底地址，此地址保存着上一栈帧的 `EBP` 值，具体可参考[此文章](https://www.cs.rutgers.edu/~pxk/419/notes/frames.html)的介绍，根据 `fp` 就可以逐级回溯调用栈。然而这一特性是会被优化掉的，而且这还是 GCC 的默认行为，在不手动指定 ` -fno-omit-frame-pointer` 时默认都会进行此优化，此时 `EBP` 被当作一般的通用寄存器使用，以此为依据进行栈回溯显然是错误的。不过尝试指定 `-fno-omit-frame-pointer` 后依然没法获取到正确的调用栈，根据 GCC 手册的[说明](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)，指定了此选项后也并不保证所有函数调用都会使用 `fp`...... 看来只有放弃使用 `fp` 进行回溯了。

`dwarf` 是一种调试文件格式，GCC 编译时附加的 `-g` 参数生成的就是 `dwarf` 格式的调试信息，其中包括了栈回溯所需的全部信息，使用 `libunwind` 即可展开这些信息。`dwarf` 的进一步介绍可参考 ["关于DWARF"](http://cwndmiao.github.io/programming%20tools/2013/11/26/Dwarf/)，值得一提的是，GDB 进行栈回溯时使用的正是 `dwarf` 调试信息。实际测试表明使用 `dwarf` 可以很好的获取到准确的调用栈。

最后 `perf` 还支持通过 `lbr` 获取调用栈，`lbr` 即 Last Branch Records，是较新的 Intel CPU 中提供的一组硬件寄存器，其作用是记录之前若干次分支跳转的地址，主要目的就是用来支持 `perf` 这类性能分析工具，其详细说明可参考 ["An introduction to last branch records"](https://lwn.net/Articles/680985/) & ["Advanced usage of last branch records"](https://lwn.net/Articles/680996/)。此方法是性能与准确性最高的手段，然而它存在一个很大的局限性，由于硬件 Ring Buffer 寄存器的大小是有限的，`lbr` 能记录的栈深度也是有限的，具体值取决于特定 CPU 实现，一般就是 32 层，若超过此限制会得到错误的调用栈。

### 测试

实际测试下以上 3 种栈回溯方式得到的结果，测试程序是一个调用深度为 50 的简单程序，从 `f0()` 依次调用至 `f50()`。

**`--call-graph fp`**：

![](https://pic.gaomf.store/perf_test_fp.svg)

**`--call-graph lbr`**：

![](https://pic.gaomf.store/perf_test_lbr.svg)

**`--call-graph dwarf`**：

![](https://pic.gaomf.store/perf_test_dwarf.svg)



可以看到，的确只有 `dwarf` 获取到了正确的调用栈。

### 总结

|         | 优点        | 缺点                                                         |
| ------- | ----------- | ------------------------------------------------------------ |
| `fp`    | None        | 1. 默认 `fp` 被优化掉了根本不可用。                          |
| `lbr`   | 1. 高效准确 | 1. 需要较新的 Intel CPU 才有此功能；2. 能记录的调用栈深度有限。 |
| `dwarf` | 1. 准确     | 1. 开销相对较大；2. 需要编译时附加了调试信息。               |

----------------------

> 参考资料：
>
> [perf Examples](http://www.brendangregg.com/perf.html)

