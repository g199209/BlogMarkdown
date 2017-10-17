title: C语言中typeof的使用
permalink: C_typeof
toc: false
mathjax: false
fancybox: false
tags:
  - C语言
categories:
  - 编程
date: '2017-10-07 20:48'
---

`typeof`不是C语言本身的关键词或运算符（`sizeof`是C标准定义的运算符），它是GCC的一个扩展，作用正如其字面意思，**用某种已有东西（变量、函数等）的类型去定义新的变量类型**。

<!--more-->

`typeof`通常用于宏定义中，一些示例用法如下：

```c
typeof(var)
typeof(a[0])
typeof(int *)
typeof(fun())
```

可以看到，`typeof()`中可以是任何有类型的东西，变量就是其本身的类型，函数是它返回值的类型。`typeof`一般用于声明变量，如：

```c
typeof(a) var;
```

不过，这也不是绝对的，从语法上来说，所有可以出现基本类型关键词的地方都可以使用`typeof`，比如`sizeof(typeof(a))`这样的用法，虽然这里的`typeof`是多余的，不过它是符合语法的。

再来看一些高级用法：

```c
int fun(int a);
typeof(fun) * fptr;    // int (*fptr)(int);

typeof(int *)a, b;     // int * a, * b;
typeof(int) * a, b;    // int * a, b;
```

可以看到，`typeof`还可以用来定义函数指针等，且`typeof(int *)a, b`是定义了两个指针变量。

最后指出一些需要注意的问题。`typeof()`是在编译时处理的，故**其中的表达式在运行时是不会被执行的**，比如`typeof(fun())`，`fun()`函数是不会被执行的，`typeof`只是在编译时分析得到了`fun()`的返回值而已。`typeof`还有一些局限性，其中的变量是不能包含存储类说明符的，如`static`、`extern`这类都是不行的。

----------

> 参考资料：
> [6.6 Referring to a Type with typeof](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html)
> [GCC扩展关键字typeof学习笔记](http://cstriker1407.info/blog/the-gcc-study-notes-typeof/)
