title: 使用OpenMP进行多线程计算
weburl: 使用OpenMP进行多线程计算
date: 2015-12-11 14:46:43
tags: [OpenMP, Concurrent]
categories: 工具之术

---

[OpenMP](http://openmp.org/)是一套支持跨平台共享内存方式的多线程并发的编程API，关于其详细介绍，可浏览其官方主页与[Wiki页面](https://zh.wikipedia.org/wiki/OpenMP)。

多线程并行计算是一个值得深入研究的问题，不过很多时候，我们仅仅是需要实现一些很基础的多线程计算。例如程序中需要依次处理100张图片，它们之间又没有什么关联，这时如果使用多线程，可以极大的提升程序的执行效率。使用OpenMP就可以很方便的实现上述要求而不必了解多线程的底层技术。

<!--more-->

Microsoft VS编译器与GCC均支持OpenMP，需要通过编译选项来启用。
- GCC中添加`-fopenmp`编译选项
- VS中在项目属性中启用`OpenMP支持`，如下图所示：

![](https://img.gaomf.cn/C20151211153251.png)

----------

大多数情况下，需要进行多线程优化的代码段是一个for循环，此时，只要在for循环前面加上**`#pragma omp parallel for`**即可。示例代码段如下：
```C
#pragma omp parallel for
for (i = 0; i < M; i++)
{
  // Do something
}

// Another for, this loop won't be paralleled
for (j = 0; j < N; j++)
{
  // Do something else
}
```
以上代码段中的第一个for循环会自动使用多线程进行并行计算，而第二个for循环不受影响。简而言之，`#pragma omp parallel for`仅对之后的一个for循环起作用。

----------

for循环中有一些代码位于所谓的临界区段（Critical Section），这部分代码不能同时被两个线程访问，否则会发生错误。使用**`#pragma omp critical`**来指定临界区段代码，用法如下：
```C
#pragma omp parallel for
for (i = 0; i < M; i++)
{
  // Do something

#pragma omp critical
  {
    // Critical section
  }
}
```
使用`{}`将临界区段代码包围起来，然后在前面添加`#pragma omp critical`即可。

----------

只需要简单的添加这两句编译指令，OpenMP就会自动实现多线程运行了。当然，OpenMP还支持更多高级用法，此处给出的仅是最简单实用的用法。
