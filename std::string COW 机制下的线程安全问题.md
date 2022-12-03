title: std::string COW 机制下的线程安全问题
toc: true
mathjax: false
fancybox: false
tags: [C++, Concurrent, Top]
date: 2022-12-03 15:53:00
weburl: Cpp_String_COW_Thread_Safety
categories: 编程之法

---

众所周知 C++ STL 容器是不保证线程安全的，不过对于 `vector`, `list`  这类容器来说，由于其底层实现很简单直接，我们可以较为容易地分析出什么时候多线程并发操作时可以不用加锁，什么时候需要加锁 —— 一般来说纯粹的并发读操作是可以不用加锁的。

然而 `string` 是个很奇特的异类，在 5.x 之前的老版本 GCC 上，由于 `string` 的实现使用了 COW 优化，这使得 `string` 的线程安全问题变得极为玄学。这也是 GCC 5.x 后引入了新的 `string` 实现放弃了 COW 的重要原因之一。 

本文就来讨论下 `string` 在 COW 机制下线程安全方面的一些坑。

<!--more-->

## std::string COW 机制概述

`string` 的 COW 实现下，其自身的成员变量很简单，只有一个指针，指向堆上的一片内存区域。这片内存的结构是这样的：

![string_heap_struct](https://img.gaomf.cn/202211272232515.svg)

除了实际的字符串内容及相关的长度记录外，还有一个引用计数字段，这就是 COW 机制的核心字段。

当且仅当使用**拷贝构造函数**或**赋值运算符**生成一个新的 `string` 时，新旧两个 `string` 会指向同一片内存，且其上的引用计数会加一；当某个 `string` 调用**有修改字符串内容可能性**的成员函数时，会检查引用计数，若引用计数大于 1，则将此片内存 copy 一份并将原来的引用计数减一，若引用计数降低至 0 则释放这片内存，这一行为的伪代码如下：

```C++
void COW() {
  if (rc == 1) {
    return;
  }
  malloc(); // Malloc new space
  memcpy(); // Copy data, new rc = 1
  if (--rc <= 0) {  // Dec old rc
    free();
  }
}

auto Fun() {
  COW();
  // Do Real work
  // ......
}
```

`rc` 引用计数本身是个原子变量，然而整个 `COW()` 函数执行过程中是**不加锁**的。实际的 `libstdc++` 的代码中，`COW()` 函数的名称是 `_M_leak()`。

除了这种 COW 优化外，新版本的 `string` 使用的都是 SSO 优化，可以参考我之前写过的另一篇文章：[C++ 中 std::string 的 COW 及 SSO 实现](https://gaomf.cn/2017/07/26/Cpp_string_COW_SSO/)

## 多引用并发操作引发的线程安全问题

### 问题描述

若某个 `string` 的多个**引用或指针**在不同线程中同时访问了此 `string` 就会产生非预期的内存非法访问行为。考虑以下测试代码：

```C++
#include <cassert>
#include <thread>
#include <atomic>
#include <string>
#include <iostream>
#include <vector>

static std::atomic_bool g_start = false;

int main() {
    std::string s1 = "abcdefghijklmn";
    std::string s2 = s1;
    // s2[0] = s2[0];

    std::vector<std::thread> threads;
    for (int i = 0; i < 8; ++i) {
        threads.emplace_back([i, &s1](){
            while(!g_start.load());
            assert(s1[i] == 'a' + i);
        });
    }
    g_start.store(true);
    for (auto &t : threads) {
        t.join();
    }

    std::cout << s1 << std::endl;
    std::cout << s2 << std::endl;

    return 0;
}
```

这段代码使用老版本的 GCC 编译运行会直接 core 掉，一般错误会是 `double free or corruption (fasttop)`（若只有新版本的 GCC，可以通过增加编译选项 `-D_GLIBCXX_USE_CXX11_ABI=0` 强制指定不使用 C++11 ABI 来复现）；使用新版本的 GCC 正常编译则不会有任何问题。

这段代码中，`L17 ~ L20` 启动了多个线程，每个线程中持有的都是 `s1` 的一个引用，多个线程同步地去**读取** `s1` 中**不同位置**的字符，这个行为从常理上分析应该是没有数据竞争的，然而实际情况是它 core 了！

更为玄学的是，若将 `L12` 的 `s2` 去掉，或者是加上 `L13` 注释里那行看上去没有任何用处的代码，这个程序就可以正常运行了！

这一切玄学行为的根源都是 COW 机制搞的鬼。`L12` 使用拷贝构造生成 `s2` 时不会为 `s2` 真的新分配一片内存空间，而是简单的将原有 `s1` 堆上的内存引用计数加一，这样这个 `string` 的引用计数就是 2 了。多个线程使用的是引用的方式捕获 `s1`，因此不会修改引用计数；而 `L19` 调用了 `operator []()`，由于此处 `s1` 不是常引用，编译器不会选择 `opeartor []() const`，而非 `const` 版本的 `operator []()` 返回的是一个可修改的引用，故此行为在库函数看来其实是一个写操作，会触发 COW；从上一节 `COW()` 的伪代码中可以看到，多个引用同时执行 `COW()` 很大概率会 Copy 多份并将引用计数减为负数进而产生 `double free` 错误。

若没有 `L12`，`s1` 的引用计数为 1，不会触发 COW，自然也不会有问题；若加上了 `L13`，同理 `s2` 已经触发了 COW，后续 `s1` 的引用计数也为 1 了；此外还可以通过将 lambda 内捕捉的 `s1` 修改为常引用或直接按值传递来解决此问题，将 `[i, &s1]` 改为 `[i, &s1 = static_cast<const std::string&>(s1)]` 或 `[i, s1]` 即可。

### 解决方案

在实际复杂一些的程序中，很难确定一个字符串的引用计数到底是多少，因此在多个线程中使用字符串引用的做法始终会存在风险，解决方案一般有两个：

1. 既然 `string` 底层都使用 COW 了，那就不要用 `string` 的引用了，所有地方都按值传递即可；
2. 不要通过拷贝构造函数或 `operator =()` 来构造新 `string`，始终使用 `string new_str = string(old_str.data(), old_str.size())` 的方式来构造。根据指针和长度来构造时永远不会也不可能使用 COW 机制，肯定会新分配一片内存做 memcpy。

上述两种方案其实都很不优雅，优雅的方案是直接用新版 GCC 的 SSO 实现，默认情况下 5.x 以后版本的 GCC 都会使用此行为的。然而某些时候为了兼容历史遗留第三方库，需要保证 ABI 兼容，此时就只能通过这些方案来绕开 COW 机制的坑了……

### _GLIBCXX_USE_CXX11_ABI 

最后来提一下 `_GLIBCXX_USE_CXX11_ABI` 这个编译器预定义宏，此宏代表是否使用 C++ 11 新版本的 ABI，主要区别就两个：

- 本文所述的 `string` 实现由 COW 改为 SSO；
- `std::list::size()` 的实现由 `O(N)` 复杂度改为 `O(1)` 复杂度，本质就是在 `list` 结构中增加了一个表示链表长度的字段。

默认情况下 5.x 之后的 GCC 版本都会预定义此宏，即 `-D_GLIBCXX_USE_CXX11_ABI=1`，可以手动的加上 `-D_GLIBCXX_USE_CXX11_ABI=0` 来禁用此行为使用原来老的实现，这主要是在处理 ABI 兼容问题是会使用。

-------

> 参考资料：
>
> [c++再探string之eager-copy、COW和SSO方案](https://www.cnblogs.com/cthon/p/9181979.html)
