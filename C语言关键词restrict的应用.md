title: C语言关键词restrict的应用
weburl: C_restrict
toc: false
mathjax: false
fancybox: false
tags:
  - C
categories:
  - 编程之法
date: '2017-10-25 16:22'
---

`restrict`是C99标准中新增的关键词，只能用于修饰指针（函数指针除外），其含义为：**此指针是访问其指向对象的唯一初始方法**。使用此关键词的意义在于：**有助于编译器进行代码优化**。

<!--more-->

`restrict`是一个类型限定词，在C99标准(ISO/IEC 9899:1999)的"6.7.3.1 Formal definition of restrict"中给出其定义，基本语法为`xxx * restrict var`，其中`xxx`是指针指向的变量类型。需要注意的是，只有指向所谓"object types"的指针才能用`restrict`修饰，而"object types"的定义应该是除了函数指针外所有类型的指针。另外，对于函数参数来说，由于数组和指针的等价性，函数参数为数组时也可以用`restrict`修饰，此时`restrict`放在`[]`中，如：`void fun(int par[restrict])`。

上面也说到，使用`restrict`的意义在于便于编译器优化，此处使用`restrict`最常用的一个例子进行说明：

```C
int foo(int * a, int * b) {
	*a = 5;
	*b = 6;
	return *a + *b;
}

int rfoo(int * restrict a, int * restrict b) {
	*a = 5;
	*b = 6;
	return *a + *b;
}
```
<!---workaround highlight bugs*--->

对于`foo`函数，编译器是不能假设`a`和`b`指向的区域是不同的，它们有可能指向同一内存区域，故此时编译得到的汇编代码为：

```x86asm
movl	4(%esp), %eax
movl	8(%esp), %edx
movl	$5, (%eax)
movl	$6, (%edx)
movl	(%eax), %eax
addl	$6, %eax
ret
```

可以看到，在进行最后一步加法运算前，需要再读一遍`*a`的值，以保证结果的正确性，因为若`a==b`的话，此时`*a == 6`而不是`*a == 5`；此时这段程序返回的是12而不是11。

然而，我们如果能确保`a != b`，那以上代码是可以进一步优化的，`restrict`关键词就用于把这一信息提供给编译器，此时的`rfoo`函数编译结果如下：

```x86asm
movl	4(%esp), %eax
movl	$5, (%eax)
movl	8(%esp), %eax
movl	$6, (%eax)
movl	$11, %eax
ret
```

这段代码中直接返回了编译器预先计算出来的结果11，与`foo`相比减少了一次加法运算，且不需要进行`movl	(%eax), %eax`这一步骤。（虽然由于Cache的存在，这一指令也不一定会进行内存访问）

> 以上汇编代码使用GCC 4.9.3编译得到，编译参数为：`-O3 -S -std=gnu11`，结果和参考资料中给出的汇编代码稍有区别，不知为何参数传递使用的是栈而不是寄存器……

从以上例子中可以看到，若一个指针是访问某内存区域的唯一方法，那可以为其加上`restrict`限定符，这有利于编译器进行代码优化生成效率更高的程序。

----------

> 参考资料：
> [restrict type qualifier](http://en.cppreference.com/w/c/language/restrict)
> [如何理解C语言关键字restrict？](https://www.zhihu.com/question/41653775?sort=created)
> [C99中的restrict关键字](http://blog.csdn.net/lovekatherine/article/details/1891806pub)
