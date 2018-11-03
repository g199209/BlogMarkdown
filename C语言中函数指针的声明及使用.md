title: C语言中函数指针的声明及使用
date: 2016-04-21 15:58:46
tags: [C]
categories: 编程之法

---

函数指针一般的声明方式为：

``` C
Return (* pf)(Params);
```

其中`Return`代表返回值的类型，`Params`代表函数参数列表，如下面两个比较简单的声明：

``` C
void (* pf1)(int, char *);
int * (* pf2)(void);
```

<!--more-->

使用的时候可以使用以下两种等价的赋值及调用方式：

``` C
// Assign
pf = FunctionName;
pf = &FunctionName;

// Call
pf();
(*pf)();
```

更为复杂一点的问题是，如果一个函数指针作为函数的参数与函数的返回值时该如何进行声明呢？

作为函数参数的情况比较简单，像这样处理即可：

```
void Fun(void (*)(int, int), int, int);

void Fun(void (* pf)(int, int), int a, int b)
{
  return pf(a, b);
}
```

这个例子中，声明并定义了一个函数`Fun()`，这个函数接受3个参数，后两个参数为`int`，第一个参数的类型是`void (*)(int, int)`，这是一个函数指针，指向一个返回值为`void`，参数为两个`int`的函数。

----------

然而如何处理函数指针作为返回值的情况呢？标准库`<signal.h>`中的`signal()`函数就是一个这样的函数，其声明形式如下：

``` C
void (* signal(int sig, void (* handler)(int)))(int);
```

这个函数声明一眼看过去不是很好懂，下面来仔细分析一下。

如果我们要声明一个整形变量`a`，可以这样写：

``` C
int a;
```

如果要改为声明一个返回值为整形的函数`fun()`，可以视为只要将声明变量时使用的`a`改为`fun()`即可，即

``` C
int fun();
```

用同样的思路来分析声明一个返回值为函数指针的函数的声明方法，只要将函数指针声明中的变量名部分换为函数体即可。用实例分析一下：

``` C
void (* pf)(int);
```

这里面`pf`是一个函数指针，指向一个输入参数为`int`，返回值为`void`的函数。将其中的`pf`换为一个函数`fun()`，即

``` C
void (* fun())(int);
```

这样就得到了一个返回值为函数指针的函数`fun()`，它返回的函数指针其实就是上面的`pf`。

理解了这样的声明方式后再来看`signal()`函数的声明就不那么难理解了。先看最里层：

``` C
void (* handler)(int)
```

这是一个函数指针`handler`，带一个`int`参数，返回值为空。

再向外一层：

``` C
signal(int sig, void (* handler)(int))
```

这是一个函数`signal()`，其参数为一个整形和一个函数指针。那这个函数的返回值是什么呢？由上面的分析易知，也是一个函数指针，这个函数指针的原型是这样的：

``` C
void (* Returnpf)(int);
```

对比`Returnpf`和`handler`的声明，可以看到二者是完全一样的。这时候就可以利用`typedef`这个好东西来简化之前那个复杂的`signal()`函数的声明了，具体做法如下：

``` C
void (* signal(int sig, void (* handler)(int)))(int);

typedef void (* sighandler_t)(int);

sighandler_t signal(int sig, sighandler_t handler);
```

其中第1行与第5行的声明是完全等价的，使用第5行这种声明方式就显得直观多了。

----------

最后编写了两个简单的测试程序来进一步熟悉下函数指针的使用方法。

不使用`typedef`的版本：

```
#include <stdio.h>

// Function declaration
void PrintStr(char *(*)(void));
char *(* Run(int, int, int (*)(int, int)))(void);

char * Msg1(void)
{
  return "Great than 0!\r\n";
}

char * Msg2(void)
{
  return "Less than 0!\r\n";
}

void PrintStr(char *(* Fun)(void))
{
  puts(Fun());
}

int Sum(int a, int b)
{
  return a + b;
}

char *(* Run(int a, int b, int (* SumFun)(int, int)))(void)
{
  if(SumFun(a, b) >= 0)
    return Msg1;
  else
    return Msg2;
}

int main()
{
  PrintStr(Run(1,2,Sum));
  PrintStr(Run(-1,-2,Sum));
  
  return 0;
}
```

使用`typedef`的版本：

``` C
#include <stdio.h>

// Typedef
typedef int (* MySumFun)(int, int);
typedef char * (* MyCharFun)(void);

// Function declaration
void PrintStr(MyCharFun);
MyCharFun Run(int, int, MySumFun);

char * Msg1(void)
{
  return "Great than 0!\r\n";
}

char * Msg2(void)
{
  return "Less than 0!\r\n";
}

void PrintStr(MyCharFun Fun)
{
  puts(Fun());
}

int Sum(int a, int b)
{
  return a + b;
}

MyCharFun Run(int a, int b, MySumFun Fun)
{
  if(Fun(a, b) >= 0)
    return Msg1;
  else
    return Msg2;
}

int main()
{
  PrintStr(Run(1,2,Sum));
  PrintStr(Run(-1,-2,Sum));
  
  return 0;
}
```

程序运行输出结果均为：

```no-highlight
Great than 0!

Less than 0!

```

使用`typedef`的版本明显可读性要好得多。
