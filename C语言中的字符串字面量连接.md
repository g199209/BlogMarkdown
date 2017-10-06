title: C语言中的字符串字面量连接
permalink: string_literal_concatenate
toc: false
mathjax: false
fancybox: false
tags:
  - C语言
categories:
  - 编程
date: '2017-10-05 15:27'
---

字符串字面量(string literal)就是程序代码中出现的`"`包围的字符串，比如`"hello"`, `"I Love C Language!"`这类的。在C语言中，有一个奇技淫巧：**两个相邻的字符串字面量会自动被合并连接为一个**。这里的相邻可以是直接连在一起，也可以是间隔着若干个空白字符。需要指出的是，这个特性是C语言标准所要求的，并不是某个编译器的扩展功能。

一个例子：

```C
printf("Hello" " World""!"    "\n");
```

以上代码完全等同于：

```C
printf("Hello World!\n");
```

----------

> 参考资料：
> [How does concatenation of two string literals work?](https://stackoverflow.com/questions/12120944/how-does-concatenation-of-two-string-literals-work)
