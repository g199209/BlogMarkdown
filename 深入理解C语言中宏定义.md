title: 深入理解C语言中宏定义
weburl: C_Macro
toc: false
mathjax: false
fancybox: false
tags: [C, Top]
categories: 编程之法
date: '2017-10-06 20:15'

---

从本质上看，C语言中的宏定义实现的是一个文本替换的功能，似乎很简单的样子，然而这几天去看了下Linux Kernel源码中的各种宏定义，才发现一个宏定义竟然也可以有如此多的奇技淫巧……于是花了一天时间仔细研究了下宏的相关知识，此处整理总结下。

<!--more-->

关于宏，网上有一组写得极好的文章，基本上看完这几篇文章就可以对宏有一个深入的理解了：

> [宏定义黑魔法-从入门到奇技淫巧 (1) —— 基本概念](http://feng.zone/2017/05/17/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-1/)
> [宏定义黑魔法-从入门到奇技淫巧 (2) —— object-like宏的展开](http://feng.zone/2017/05/18/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-2/)
> [宏定义黑魔法-从入门到奇技淫巧 (3) —— function-like宏的展开](http://feng.zone/2017/05/20/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-3/)
> [宏定义黑魔法-从入门到奇技淫巧 (4) —— 一些宏的高级用法](http://feng.zone/2017/05/21/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-4/)
> [宏定义黑魔法-从入门到奇技淫巧 (5) —— 图灵完备](http://feng.zone/2017/05/21/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-5/)
> [宏定义黑魔法-从入门到奇技淫巧 (6) —— 宏的一些坑](http://feng.zone/2017/05/28/%E5%AE%8F%E5%AE%9A%E4%B9%89%E9%BB%91%E9%AD%94%E6%B3%95-%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7-6/)

作者的知乎上也有一份[相同的备份](https://www.zhihu.com/people/feng-yu-yao/posts)。

相同的内容此处就不再重复了，此处列出一些要点。

- 带参数的宏中可以使用两个特殊运算符，`#`(Stringification Operator)和`##`(Token Pasting Operator)，作用分别是把宏参数变为字符串字面量和连接两个token。且遇到这两个运算符时，宏参数不会展开。
- 宏的嵌套展开过程中，已展开过的宏不会重复展开。
- 宏展开后，会进一步检查是否构成了新的宏，若构成了会进一步展开。
- 宏定义中也可以使用`...`代表可变参数，用`__VA_ARGS__`获取可变参数列表。
- 宏参数会先展开，之后再进行替换，这也被称为"prescan".
- 宏基本上是图灵完备的，所以可以只靠宏实现各种东西……

宏的展开过程遵循以下流程图：

![](https://pic.gaomf.store/Macro_Expand3.svg)

这个流程图是我根据自己的理解和实验画出来的，并不确定完全正确……图中的“已展开宏记录”就是文章中说的“蓝色列表”。

使用gcc编译时，可以通过附加`-E`参数，让gcc只进行预处理，这样就可以看到各种宏实际展开出来的结果是什么了，如`gcc -E -o test.i test.c`命令对`test.c`文件进行预处理，生成`test.i`文件。

关于宏的展开流程，有一些不太明确的地方，此处用例子说明下，结果都经过gcc预处理验证过。

------------

```C
#define P 2
#define M K(P)
#define K(a) a a ## a a
#define PP 11
#define FOO(a) #a a

FOO(M)	-->  FOO(K(P))
		-->  FOO(2 PP 2)
		-->  FOO(2 11 2)
		-->  "M" 2 11 2
```

遇到`#`或`##`时，其相连的宏参数不会展开，然而这不意味着这个宏参数本身不会展开，其他部分用到这个宏参数的地方还是会展开的。

另外，`#`运算符后必须是一个宏参数，不能是其他东西，不过`##`两端则无这一要求，来看个例子：

```C
#define P(a)  #a a
#define PP(a) #a a
#define CAT(a) P ## P(P##a(a))

CAT(P)	-->  PP(PP(P))
		-->  PP("P" P)
		-->  "PP(P)" "P" P

CAT(A)	-->  PP(PA(A))
		-->  "PA(A)" PA(A)
```

可以看到，`##`两端可以是任意token，其作用就是把这两个token合成一个。另外，还可以看到，`##`是在最开始就进行处理的，所以`P(a)`这个宏是没有用到的。另外，由`##`操作符组合产生的新宏是会继续展开的，并不像某些文章说的那样会停止展开。

----------------

关于多次扫描展开的问题，有些文章中说的是展开完成后会重新扫描一遍当前字符串，若有可以继续展开的则继续展开，然而实际测试下来并不是这样的。还是来看个例子：

```C
#define BRACKET ()
#define CREATE(a, b)  a ## b
#define FOO() 123

CREATE(F, OO) BRACKET   -->  FOO BRACKET
                        -->  FOO ()

CREATE(F, OO) ()        -->  FOO ()
                        -->  123

FOO BRACKET             -->  FOO ()
```

可以看到，第1、3个例子中，展开到`FOO ()` 之后就没有继续展开下去了，这说明并没有重新扫描字符串这一步，**已经处理过的部分不会再次处理的**。而第2个例子则说明的确是会再展开合并出新的宏来的，故上面的流程图中使用了"向后扫描一个token，形成一个新的字符串"这样的说法。考虑到token是以空白为界划分的，**后面组合出来的新宏只可能是function-like的宏**，所以这样的展开方式是不存在歧义的，不会出现原来的宏被组合成其他宏的情况。

如果要继续展开上面未能展开的那两个宏，可以再封装一层：

```C
#define BRACKET ()
#define CREATE(a, b)  a ## b
#define FOO() 123
#define EXPAND(...) __VA_ARGS__

EXPAND(CREATE(F, OO) BRACKET)	-->  EXPAND(FOO())
								-->  EXPAND(123)
								-->  123
```

这里利用的原理是：宏参数会先尽可能展开后再进行替换。

------------

将宏定义的各种奇技淫巧应用得巅峰造极、神鬼莫测之作就是[The Boost Preprocessing library](http://www.boost.org/doc/libs/1_65_1/libs/preprocessor/doc/index.html)，这是Boost库的一部分，不过和其他部分完全独立，这部分包含了各种数据结构和算法等，而且只有头文件，全部都是宏定义……简直可谓是丧心病狂……Github上有人将这部分独立的代码提取出来了，有兴趣的读者可以去进一步揣摩瞻仰：[boost-preprocessor](https://github.com/imoldman/boost-preprocessor)

----------

> 参考资料：
> [代码自动生成-宏带来的奇技淫巧](http://www.cppblog.com/kevinlynx/archive/2008/03/19/44828.html)
> [《C标准库》—之<assert.h>实现](http://blog.csdn.net/jy_95/article/details/45260775)
> [C preprocessor](https://en.wikipedia.org/wiki/C_preprocessor)
