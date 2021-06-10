title: C语言生成随机数
weburl: C语言生成随机数
date: 2015-11-16 20:22:19
toc: false
tags: [C, Number Theory]
categories: 编程之法

---

在C语言中，使用`rand()`函数生成随机数，`srand()`函数初始化随机数种子。

`rand()`函数原型为`int rand(void)`，在头文件`stdlib.h`中声明。此函数返回一个0~`RAND_MAX` 间的伪随机整数。

<!--more-->

`RAND_MAX`是头文件`stdlib.h`中定义的一个宏，其值在不同平台下有所区别。根据C语言标准，`RAND_MAX`最小为32767。在Microsoft VS提供的标准库文件中，`RAND_MAX`被定义为32767。

`srand()`函数原型为`void srand(unsigned int seed)`，同样在头文件`stdlib.h`中声明。此函数将`seed`设置为伪随机数生成的初始种子。确定了初始种子后，每次调用`rand()`函数时，就会使用递推公式给出一个伪随机数。所以显而易见，若`seed`相同，之后得到的伪随机数序列也会完全相同。
如果不显式调用`srand()`函数，在第一次调用`rand()`函数之前，会自动将`seed`设置为1。

通常做法是使用当前时间来作为`seed`，即：
``` C
srand((unsigned int)time(NULL));
```

其中，`time()`函数的原型为`time_t time(time_t *timer)`，在头文件`time.h`中声明。此处给出的函数原型是基于Microsoft VS提供的标准库文件的，在其它平台下可能会有所区别。关于此函数的参数与返回值，可参考MSDN中的说明：
> The time function returns the number of seconds elapsed since midnight (00:00:00), January 1, 1970, Coordinated Universal Time (UTC), according to the system clock. The return value is stored in the location given by timer. This parameter may be NULL, in which case the return value is not stored.
> 
> time is a wrapper for _time64 and time_t is, by default, equivalent to __time64_t. If you need to force the compiler to interpret time_t as the old 32-bit time_t, you can define _USE_32BIT_TIME_T. This is not recommended because your application may fail after January 18, 2038; the use of this macro is not allowed on 64-bit platforms.

简而言之，使用`time(NULL)`会返回从1970年1月1日0:0:0至今的秒数，每次运行程序时此值都不相同，故可以用此值作为`seed`。
需要注意的是，`time()`的最小分辨率为1s，所以在1s内多次调用此函数的返回值是相同的。

--------------------------------

直接调用`rand()`函数仅能得到0~`RAND_MAX`间的整数，如需得到其它范围的整数或小数，则需要在此基础上再进行一些处理。

### **生成任意范围的整数**

代码如下：
``` C
int random_int(int minNum, int maxNum)
{
  int delt = maxNum - minNum;
  if (delt <= RAND_MAX)
    return rand() % (delt + 1) + minNum;
  else
    return rand()*delt / RAND_MAX + minNum;
}
```
这段代码是网上广为流传的代码，然而，从理论上仔细进行分析，这个方法并不能严格保证生成的伪随机数符合均匀分布。故此方法仅适合于粗略的生成一些随机数。

### **生成0~1区间的小数**

代码如下：
``` C
float random_float()
{
  return (float)rand()/(RAND_MAX + 1);
}
```

### 数学基础

前面提到，`rand()`函数是使用某个递推公式来生成一个伪随机数序列的，事实上，这里使用的算法是[线性同余法](http://en.wikipedia.org/wiki/Linear_congruential_generator) （LCG，Linear Congruential Generator）。

LCG的基本公式为：
> x(n+1) = (A * x(n) + C) % M

所谓初始种子`seed`即为x(0)的值。

不同编译器附带的标准库文件中A、C、M这三个参数的取值并不相同，具体可参考 [Wikipedia](http://en.wikipedia.org/wiki/Linear_congruential_generator) 中的说明
对于Microsoft VS来说，在`rand.c`文件中可以找到如下代码：
``` C
int __cdecl rand (void)
{
  _ptiddata ptd = _getptd();

  return( ((ptd->_holdrand = ptd->_holdrand * 214013L
      + 2531011L) >> 16) & 0x7fff );
}

void __cdecl srand (unsigned int seed)
{
  _getptd()->_holdrand = (unsigned long)seed;
}
```
可以看到，A = 214013, B = 2531011, m = 2^32。

线性同余法的优势在计算速度快，内存消耗少。但是生成的随机数质量不高，因为其相邻的随机数并不独立，序列关联性较大。
除了线性同余法外，还有多种生成伪随机数的方法，如生成均匀分布伪随机数使用的梅森旋转算法、Multiply-With-Carry算法，生成高斯分布伪随机数使用的中心极限定理法、Box-Muller算法等。具体可以参考 [这篇文章](http://blog.skyoung.org/2013/08/27/generate-random-number/) 的说明。

> C++中生成随机数可以使用更好的方法，详见：[C++生成随机数](/2017/03/22/C++_Random/)



