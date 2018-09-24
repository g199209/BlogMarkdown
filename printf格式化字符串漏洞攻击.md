title: printf格式化字符串漏洞攻击
permalink: printf_format_string_attack
toc: false
mathjax: false
fancybox: false
tags:
  - C语言
  - Top
categories:
  - 嵌入式
date: '2017-09-20 21:58'
---

前几天去Intel面试时，遇到了一个问题：`printf("%s", s)`与`printf(s)`有何区别？面试官还提示我从安全的角度回答这个问题，然而当时并没有想出答案来……:( 回来后仔细研究了下这个问题，才发现`pritnf(s)`这种写法是存在严重安全漏洞的，这被称为printf格式化字符串漏洞攻击。

<!--more-->

`printf`函数支持不定参数的原理在此不多说，可参考`stdarg.h`头文件的相关介绍。核心就是，函数使用栈传递参数，且根据cdecl函数调用约定，压栈顺序是从最后一个参数开始逆序进行的，而`printf`函数的第一个参数是确定的字符串，故可以从这个字符串中判断出栈中还有多少个数据，每个数据的类型是什么，进而依次取出各个数据。

正常情况下，格式化字符串中格式化字符`%`是和之后的参数一一对应的，然而我们可以考虑一种情况，若格式化字符`%`比参数多会怎么样呢，答案是，会取到栈中其他的数据。这就是`printf(s)`这种写法有问题的原因了，这里使用`s`而不是一个字符串字面常量，而`s`传入什么内容其实是不可控的，若传入字符中存在`%`，就会输出栈中其他一些内容。要是`s`还是可以由外部输入的，那就可以通过巧妙的构造`s`的形式来实现**访问栈中本来没有权限访问的内容**，这就是所谓的格式化字符串漏洞攻击。

下面来看一个实际的例子：

```c
int main() {
	int security;
	char str[100];
	scanf("%d", &security);
	scanf("%s", str);
	printf(str);

	return 0;
}
```

这段程序中由用户输入一个整数，存放在`security`这个变量中，之后再输入一个字符串`str`，并使用`printf(str)`这样的形式将其打印出来。针对这个例子，我们可以构造一个特定的输入字符串将`security`的内容给打印出来。

使用GCC 4.9.3编译以上程序，优化等级`-O2`，得到的程序用`objdump`反汇编一下，可以找出对应的汇编代码为：

```x86asm
00408350 <_main>:
  408350:	55                   	push   %ebp
  408351:	89 e5                	mov    %esp,%ebp
  408353:	53                   	push   %ebx
  408354:	83 e4 f0             	and    $0xfffffff0,%esp
  408357:	83 c4 80             	add    $0xffffff80,%esp
  40835a:	e8 31 95 ff ff       	call   401890 <___main>
  40835f:	8d 44 24 18          	lea    0x18(%esp),%eax
  408363:	8d 5c 24 1c          	lea    0x1c(%esp),%ebx
  408367:	c7 04 24 c0 a0 40 00 	movl   $0x40a0c0,(%esp)
  40836e:	89 44 24 04          	mov    %eax,0x4(%esp)
  408372:	e8 f1 fb ff ff       	call   407f68 <_scanf>
  408377:	89 5c 24 04          	mov    %ebx,0x4(%esp)
  40837b:	c7 04 24 c3 a0 40 00 	movl   $0x40a0c3,(%esp)
  408382:	e8 e1 fb ff ff       	call   407f68 <_scanf>
  408387:	89 1c 24             	mov    %ebx,(%esp)
  40838a:	e8 e9 fb ff ff       	call   407f78 <_printf>
  40838f:	31 c0                	xor    %eax,%eax
  408391:	8b 5d fc             	mov    -0x4(%ebp),%ebx
  408394:	c9                   	leave  
  408395:	c3                   	ret    
```

`408350`~`40835a`部分为GCC自动生成的`main`函数入口处代码，此处不用管它；`40835f`~`408372`为第一个`scanf`函数，读入`security`；`408377`~`408382`为第二个`scanf`函数，读入`str`；`408387`~`40838a`调用`printf`输出`str`。根据反汇编结果，我们可以画出`408372`行时栈的结构：

![](http://gmf.shengnengjin.cn/scanf_stack.svg)

`scanf`函数反向压栈，故`%esp`指针指向的元素`$0x40a0c0`应该就是字符串字面量`"%d"`的首地址，而`%esp+0x04`就是栈中的下一个元素，这就是`security`变量的地址，从汇编代码中可以看到，这个地址是一个`%esp`的偏移量，`%esp+0x18`。由此，我们可以分析出`security`在栈中的位置，而之后`%esp`指针没有改变，故我们可以构造以下这个特殊的字符串：`%d%d%d%d%d_____security_is:%d`，这个字符串会连续取出栈中的前6个`int`，第6个`int`的地址就是`%esp+0x18`，这正是`security`变量的地址。

实际运行程序测试一下：

![](http://gmf.shengnengjin.cn/20170920214800.png)

可以看到，我们成功读到了`security`的内容。

为了预防这个问题，我们需要确保在使用`printf`函数时第一个参数必须是一个字符串字面量。其实，以上写法编译器也会产生warning：format string is not a string literal (potentially insecure).

最后，其实配合一个比较少用的`printf`参数`%n`还可以利用此漏洞实现对栈中内容的修改，在此就不展开了，可参考第三篇参考资料。

----------

> 参考资料：
> [printf(string) vs. printf(“%s”, string)](https://stackoverflow.com/questions/13692044/printfstring-vs-printfs-string)
> [What is the underlying difference between printf(s) and printf(“%s”, s)?](https://stackoverflow.com/questions/39415536/what-is-the-underlying-difference-between-printfs-and-printfs-s)
> [格式化字符串漏洞简介](https://paper.seebug.org/papers/Archive/drops2/%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%BC%8F%E6%B4%9E%E7%AE%80%E4%BB%8B.html)
