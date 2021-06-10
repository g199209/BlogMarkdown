title: Linux kernel中的min和max宏
weburl: Kernel_min_max_macro
toc: false
mathjax: false
fancybox: false
tags: [Kernel, C]
categories: 编程之法
date: '2017-10-08 16:38'

---

`min`和`max`是两个很常用的操作，一般都是用宏实现的，不过想要写出一个很完善的宏定义还是要考虑很多问题的，本文就来分析下Linux Kernel中的实现方法。文中仅考虑`min`，`max`的结构与其完全相同，只要修改下大于小于号即可。

<!--more-->

宏定义中要将整体和变量都加上括号的意义此处就不多说了，据此我们可以写出一个最基本的形式：

```c
#define min(a, b) ((a) < (b) ? (a) : (b))
```

然而这种写法是有副作用的，考虑`min(a++, b)`这样的用法，其展开后的形式为：

```C
((a++) < (b) ? (a++) : (b++))
```

当`a<b`时，`a++`会被执行两次，这显然不是我们所希望的，为了解决这一问题，我们可以使用下面这个稍显复杂的宏定义：

```C
#define min(a, b) ({ \
	typeof(a) __min1__ = (a);  \
	typeof(b) __min2__ = (b);  \
	(void)(&__min1__ == &__min2__);  \
	__min1__ < __min2__ ? __min1__ : __min2__;})
```

这里用到了GCC的一个扩展特性，形如`({ ... })`这样的代码块会被视为一条语句，其计算结果是`{ ... }`中最后一条语句的计算结果。故上述宏定义展开后的结果就是第5行返回的结果。注意，这个扩展特性不是所有编译器都有的，如果用VS编译上述代码，是无法通过编译的。
这个宏定义中，先根据`a`, `b`的类型生成了两个局部变量`__min1__`和`__min2__`，之后比较其大小，返回较小的一个，这样就保证了宏参数只会被执行一次，避免了上述副作用。另外，第4行代码其实是没有实际作用的，其意义在于，若`__min1__`和`__min2__`的类型不同，比较其地址时编译器会给出一个Warning，这样可以避免一些潜在的错误发生。

以上宏定义就是网上普遍流传的Linux Kernel中的实现方法，然而，我实际阅读了当前`4.12.7`版本的Kernel源代码，发现实际的实现方法要更复杂一些。在引入实际的实现方法前，我们先思考一下以上宏定义还存在什么漏洞。

考虑以下代码段：

```C
int __min1__ = 10;
int __min2__ = 20;
int min_val = min(__min1__--, __min2__++);
```

期望得到的结果应该是`__min1__ = 9`, `__min2__ = 21`, `min_val = 10`。然而实际情况是，`__min1__ = 10`, `__min2__ = 20`，`min_val`的值则是不确定的。造成这一结果的原因在于输入参数和宏定义内部使用的局部变量重名了，这样就会导致在宏定义的语句块内，外层同名变量的作用域被内层局部变量的作用域所屏蔽，展开后的代码就成了这样：

```C
int main_val = ({typeof(__min1__--) __min1__ = (__min1__--); /* 省略 */ })
```

这就类似于`int a = a`这样的语句，执行完后`a`的值是不确定的，且因为展开后`__min1__`成了宏体内的局部变量，`__min1__--`的自减操作对于外层变量来说也是无效的。

知道问题的原因后解决方法就很清晰了，只要避免重名就可以了，其实上述宏定义中使用`__min1__`这样的名字也是为了避免重名，然而，靠起特殊的名字这种方法不是那么的优雅，故实际新版的Linux Kernel中使用了编译器产生的唯一名称来解决这一问题：

```C
/* Indirect macros required for expanded argument pasting, eg. __LINE__. */
#define ___PASTE(a,b) a##b
#define __PASTE(a,b) ___PASTE(a,b)

#define __UNIQUE_ID(prefix) __PASTE(__PASTE(__UNIQUE_ID_, prefix), __COUNTER__)

/*
 * min()/max()/clamp() macros that also do
 * strict type-checking.. See the
 * "unnecessary" pointer comparison.
 */
#define __min(t1, t2, min1, min2, x, y) ({		\
	t1 min1 = (x);					\
	t2 min2 = (y);					\
	(void) (&min1 == &min2);			\
	min1 < min2 ? min1 : min2; })
#define min(x, y)					\
	__min(typeof(x), typeof(y),			\
	      __UNIQUE_ID(min1_), __UNIQUE_ID(min2_),	\
	      x, y)
```

可以看到这是一个多重的宏嵌套结构，主要区别就在于`__UNIQUE_ID(min1_)`上，`__UNIQUE_ID`可以生成一个唯一的名字。唯一性是由编译器提供的`__COUNTER__`宏保证的，这也是GCC的一个扩展，[GCC文档中](https://gcc.gnu.org/onlinedocs/cpp/Common-Predefined-Macros.html)对其的说明如下：

> This macro expands to sequential integral values starting from 0. In conjunction with the ## operator, this provides a convenient means to generate unique identifiers.

简而言之，`__COUNTER__`会被展开为一个从0开始的整数，且每次调用后其值都会加一，这也就保证了其唯一性。

至于`__PASTE`宏这是用来实现两个token连接的，之所以要两重宏定义和宏的嵌套展开规则有关，可以参考我之前写的[总结文章](/2017/10/06/C_Macro/)。调用`__UNIQUE_ID(min1_)`后产生的就是形如`__UNIQUE_ID_min1_0`这样的变量名，这就确保了此名称不会和传入变量的名称重复了。当然，我们还是可以通过刻意构造这样一个特殊名称来实现冲突的，只是程序是程序员自己写的，相信也没有程序员这么无聊……故我们只需要保证正常情况下不会发生冲突即可。
