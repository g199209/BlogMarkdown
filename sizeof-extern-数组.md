title: sizeof 获取 extern 数组长度
permalink: sizeof_extern_array
toc: false
mathjax: false
fancybox: false
tags:
  - C语言
categories: 编程
date: 2018-06-30 16:52:12

---

sizeof是获取数组元素个数的常用运算符，然而前几天使用时发现，对于extern类型的数组，sizeof的使用上是有些需要考虑的问题的。

<!--more-->

假设系统中有3个文件：

`file1.c`:

```C
int array[] = {1, 2, 3};
```

`header1.h`:

```C
extern int array[];
```

`main.c`:

```C
#include "header1.h"

void fun()
{
	// This is WRONG!
	size_t elements_in_array = sizeof(array) / sizeof(int);
}
```

在`main.c`中期望通过`sizeof`运算符获取`array`中元素个数，然而这么做是错误的，编译时无法通过，错误提示类似`incomplete type not allowed`这类。

造成这一问题的原因在于，**`sizeof`是在编译时计算的，而C/C++的编译是以文件为基本单位的**。在编译`main.c`文件时，编译器是不可能知道定义在`file1.c`文件中`array`数组具体信息的，只根据`header1.h`文件中的声明是无法确定`array`的具体大小的，因此，就算某些编译器编译时不报错，得到的结果也是不正确的。

分析清楚原因后来看下解决方案，基本解决方法有4种：

1. 避免使用匿名长度的数组声明，使用宏定义预先确定数组大小；
2. 定义一个辅助变量用于保存数组大小信息，将其定义赋值放在定义`array`数组的同一个文件中；
3. 使用特殊元素表示数组结束，就像字符串结尾的`'\0'`一样，这样就可以在运行阶段动态确定数组大小；
4. 将数组的定义放到使用它的源文件中。

这几种方法都有其缺点：

1. 使用`sizeof`就是不想固定数组长度，因为使用宏定义固定数组长度不够灵活，要是想添加数组元素也要同时修改宏定义，否则尽管编译不会报错，然而运行时新添加的元素其实是无效的，这会导致将来维护时一些潜在Bug发生的可能性增加；
2. 需要一个额外的存储空间，且由于这是一个变量，每次使用数组长度时都需要访问内存，编译器也无法对数组长度作出任何假设，进而影响编译优化，理论上说这可能会导致运行时一些微小的效率损失；
3. 需要修改上层逻辑，缺乏通用性；
4. 大部分情况下，使用非`static`全局变量的原因就是多个源文件需要使用这个变量，这时显然无法做到这一点，多次重复定义链接时会出错的。

实际使用中，需要根据具体问题具体分析采用哪种方法最恰当，一般而言不经常变化的数组就使用宏定义确定其大小，会经常变化的第2种方法最常用，此时还可以用一些宏定义简化编程，以上代码可修改为：

`file1.c`:

```C
#include "header1.h"

int array[] = {1, 2, 3};

ELEMENTS_IN_DEF(array)
```

`header1.h`:

```C
#define  ELEMENTS_IN(array)            __elements_in_##array
#define  ELEMENTS_IN_DEF(array)        size_t __elements_in_##array = sizeof(array) / sizeof(array[0]);
#define  ELEMENTS_IN_DECLARE(array)    extern size_t __elements_in_##array;

extern int array[];
ELEMENTS_IN_DECLARE(array)
```

`main.c`:

```C
#include "header1.h"

void fun()
{
    size_t elements_in_array = ELEMENTS_IN(array);
}
```
> 参考资料：
>
> [comp.lang.c FAQ list · Question 1.24](http://c-faq.com/decl/extarraysize.html)
>
> [C: How to determine sizeof(array) / sizeof(struct) for external array?](https://stackoverflow.com/questions/23230114/c-how-to-determine-sizeofarray-sizeofstruct-for-external-array)
>
> [sizeof extern数组](https://blog.csdn.net/ranhui_xia/article/details/39502665)