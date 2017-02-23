title: C标准库学习笔记(1)——time、ctype、stdarg、assert
date: 2016-08-02 21:24:39
tags: [C语言, Top]
categories: 编程

---

这一系列文章是对P.J. Plauger所著的《C标准库》（The Standard C Library）一书的学习笔记，此书是关于C标准库的经典著作，讲述了每一个库函数的使用方法和实现细节。C语言标准库是最顶尖程序员的智慧结晶，要深入理解其实现细节自然也是很困难的……这里记录一下阅读这本书的收获，跳过了其中不常用或太复杂的部分（`stddef.h`、`float.h`、`limits.h`、`errno.h`、`setjmp.h`、`signal.h`、`locale.h`、`math.h`）。

<!--more-->

**需要注意的是，目前实际使用的Glibc或uClibc中的代码比书中描述的要更为复杂，而且实现机理也不尽相同，不过基本大同小异，所以此处主要是学习思想而不是具体的代码。**

相关文章：
[C语言标准库总结](/2016/04/18/C%E8%AF%AD%E8%A8%80%E6%A0%87%E5%87%86%E5%BA%93%E6%80%BB%E7%BB%93/)


## time.h
日期和时间操作。**需要特别注意的是，书中使用的`time_t`时间戳标准是从1900年1月1日午夜开始的，这与目前广泛使用的UNIX时间戳不一样，也和Glibc的实现不一样，书中是通过`_TBIAS`这个宏定义偏置量来解决这个问题的，为了简单起见，此处对此进行了改写，忽略了偏置问题，直接将其修改为与UNIX时间戳一样。**

### 使用方法
通常使用`time(NULL)`获取一个`time_t`类型的UNIX时间戳，这一般是一个32位整数(signed int)，指的是从1970年1月1日午夜至今的秒数，大约可以表示到2038年。如果要获取更精确的时间，可使用`clock()`函数。

其余函数用于在几种不同数据结构间进行转换，根据需要选取即可，其中`tm`类型的定义一般是这样的：

``` C
struct tm {
    int tm_sec;     /* [0, 60], 1 leap second */
    int tm_min;     /* [0, 59] */
    int tm_hour;    /* [0, 23] */
    int tm_mday;    /* [1, 31] */
    int tm_mon;     /* [0, 11] */
    int tm_year;    /* Years since 1900 */
    int tm_wday;    /* [0, 6], Sunday, Monday... */
    int tm_yday;    /* [0, 365], days since January 1th */
    int tm_isdst;   /* 夏令时标志，无效则为0 */
}
```

需要注意的是，以上只是`time_t`的最小实现，实际Glibc 2.23版本的代码中除了上述成员外还添加了其它字段。**`tm_year`是从1900年开始的，并不是和UNIX时间戳相同的1970年。**

### 实现方法

`time()`和`clock()`函数是依赖于具体实现的，此处不作分析。

`difftime()`函数返回两个时间戳之间的差值，考虑到`time_t`可能会被定义为无符号整数，故需要先比较二者的大小：

``` C
double difftime(time_t t1, time_t t0) {
    return (t0 <= t1 ? (double)(t1 - t0) : -(double)(t0 - t1));
}
```

`tm`与`time_t`间的转换函数是`<time.h>`中的重点，这里主要来看一下`gmtime()`和`mktime()`的实现方法。下列代码在书中给出的代码基础上进行了些改写，主要是做了些精简，没有考虑夏令时等问题。虽然以下两段代码比Glibc中的实现要简单得多，不过经测试完全可以正常使用。

``` C
static const short lmos[] = { 0, 31, 60, 91, 121, 152, 182, 213, 244, 274, 305, 335 };
static const short mos[] = { 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 };
#define MONTAB(year) ((year & 0x03) == 2 ? lmos : mos)

struct tm * gmtime(time_t * timer) {
  static struct tm ts;
  int year;
  int days;
  int secs;

  secs = *timer;
  days = secs / 86400;          // 获取天数
  ts.tm_wday = (days + 4) % 7;  // 1970年1月1日是星期四

  int dDay;
  /* days / 365 先求出year的初步估计，因为闰年的存在不一定准确（可能会多1年） */
  /* (year + 1) / 4 求出因闰年多出来的天数 */
  /* days与year年初的天数比较，若days小于它，说明year估计有误，需要减去1年 */
  for (year = days / 365; days < (dDay = (year + 1) / 4 + 365 * year);)
    year--;
  days -= dDay;             // 将days变成1年中的天数
  ts.tm_year = year + 70;   // tm_year是从1900年开始的
  ts.tm_yday = days;        // 总天数减去年初的天数

  /* 从最后一个月开始，逐步向前寻找正确的月份，pm[mon]得到月初的天数 */
  int mon;
  const short * pm = MONTAB(year);
  int tmp = (year & 0x03) == 2;
  for (mon = 11; days < pm[mon]; mon--);
  ts.tm_mon = mon;
  ts.tm_mday = days - pm[mon] + 1;

  /* 根据secs依次求出小时、分钟和秒 */
  secs %= 86400;
  ts.tm_hour = secs / 3600;
  secs %= 3600;
  ts.tm_min = secs / 60;
  ts.tm_sec = secs % 60;
  
  return &ts;
}
```

``` C
static const short lmos[] = { 0, 31, 60, 91, 121, 152, 182, 213, 244, 274, 305, 335 };
static const short mos[] = { 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334 };
#define MONTAB(year) ((year & 0x03) == 2 ? lmos : mos)

time_t mktime(struct tm * timeptr) {
  int year, days, secs;
  // 检查参数有效性，不使用tm_yday & tm_wday
#ifndef NDEBUG
  if (timeptr->tm_hour < 0 || timeptr->tm_hour > 23)
    return -1;
  if (timeptr->tm_mday < 1 || timeptr->tm_mday > 31)
    return -1;
  if (timeptr->tm_min < 0 || timeptr->tm_min > 59)
    return -1;
  if (timeptr->tm_mon < 0 || timeptr->tm_mon > 11)
    return -1;
  if (timeptr->tm_sec < 0 || timeptr->tm_sec > 60)
    return -1;
  if (timeptr->tm_year < 70 || timeptr->tm_year > 138)
    return -1;
#endif

  year = timeptr->tm_year - 70;
  days = (year + 1) / 4 + 365 * year;
  days += MONTAB(year)[timeptr->tm_mon] + timeptr->tm_mday - 1;

  secs = 3600 * timeptr->tm_hour+ 60 * timeptr->tm_min + timeptr->tm_sec;

  return (86400 * days + secs);
}
```

需要指出的是，上面这个`mktime()`函数没有考虑时区的问题，而**标准的`mktime()`函数实现的是将由`tm`表示的地方时转换为`time_t`表示的GMT时间**，所以二者并不等价。

在1970~2038年这个范围内，闰年规律符合简单的4年一闰，所以可以用`(year & 0x03) == 2`来进行闰年判断。

其余`asctime()`、`ctime()`等函数用于返回格式化的时间字符串，其原理和`sprintf()`等函数大同小异，在此不作分析。

## ctype.h
包含字符测试及大小写转换函数。

### 使用方法
提供了若干`isxxxx()`函数用于判断字符类型，并提供了大小写转换函数，具体函数列表见：[C语言标准库总结](/2016/04/18/C%E8%AF%AD%E8%A8%80%E6%A0%87%E5%87%86%E5%BA%93%E6%80%BB%E7%BB%93/)

需要说明的是，字符集的具体定义和区域设置有关，不过常用的就是英文的情况，因为这些函数也无法处理中文编码。另外，函数接受的参数是一个`int`类型的整数，不过只有`unsigned char`类型所能表示的值加上`EOF`宏定义的值（一般为-1）是有效的，传入其它值的行为是未定义的。

### 实现方法
出于效率考虑，标准库中的实现方法是基于转换表的，这里不列举具体使用的转换表了，仅描述一下设计思路。

首先将整个字符集合划分为若干个合理设计的子集，如数字（`0`~`9`）、小写字母（`a`~`z`）、大写字母（`A`~`Z`）等，每一类用一个比特位来表示，这样就可以得到如下宏定义：

``` C
#define _XD  0x01  /* '0'-'9', 'A'-'F', 'a'-'f' */
#define _UP  0x02  /* 'A'-'Z' */
#define _SP  0x04  /* space */
// ......
```

任何一个字符都属于某一子集（或某几个子集）中，这样就可以根据以上宏定义得到这个字符的编码了，将全体字符编码构成一个数组，这就是所谓的转换表，书中这个数组的名字叫做`_Ctype`。这样一来，要判断某个字符是否属于某个子集就很简单了，只要检查这个字符在转换表中对应值的特定位是否被置位了就可以了，比如检查一个字符是否是大写字母：

``` C
int isupper(int c) {
    return (_Ctype[c] & _UP);
}
```

----------

大小写间的转换也是基于转换表的，这个转换表相当于在原始ACSII表的基础上将大写字母替换为小写字母（或相反）得到的。

关于区域编码的问题此处从略。

## stdarg.h
用于处理可变参数。

### 使用方法
可变参数函数的定义类似这样：

``` C
#include <stdarg.h>

void fun(int parmN,...) {
    va_list ap;
    va_start(ap, parmN);
    //......
    int a = va_arg(ap, int);
    double b = va_arg(ap, double);
    //......
    va_end(ap);
}
```

必须要有**至少一个**固定参数，习惯上把最后一个固定参数叫做`parmN`。在函数中先调用`va_start()`初始化`va_list`，之后就可以通过`va_arg()`依次获取各参数，最后调用`va_end()`即可。**需要注意的是，在可变参数中，应用的是“加宽”原则，也就是`float`会被扩展成`double`，`char`、`short`等会被扩展成`int`，也就是说，函数中只该使用以下这些表达式：**

``` C
va_arg(ap, double);
va_arg(ap, int);
va_arg(ap, unsigned int);
```

### 实现方法

``` C
typedef char * va_list;

#define va_start(ap, A)  (void)((ap) = (char *)&A)
#define va_end(ap)       (void)(0)
#define va_arg(ap, T)    (*(T *))((ap += sizeof(T)) - sizeof(T))
```

这里给出的代码是简化版代码，没有考虑存储空隙及对齐问题，仅用来说明基本原理。

## assert.h ##
提供断言。

### 使用方法
在需要使用断言的地方加入`assert(x)`即可，`x`是一个`int`，若`x`为零断言成立，此时程序会向标准错误流输出一条包含出错行号等的错误信息并调用`abort()`函数终止程序的运行。`assert(x)`返回`void`。

一般只有在程序调试时才需要终止程序运行，发布时应该去掉这个功能，为实现这一目的，可通过定义`NDEBUG`这个宏来实现，一般使用编译器预定义。

### 实现方法
为了对`NDEBUG`作出正确回应，头文件的基本结构如下：

``` C
#undef assert    /* remove existing definition */
#ifdef NDEBUG
#define assert (test) ((void) 0)  /* passive form */
#else
#define assert (test) ...         /* active form */
#endif
```

其中`active form`的定义如下：
``` C
void _Assert(char *);
#define _STR(x) _VAL(x)
#define _VAL(x) #x
#define assert(test)  ((test) ? (void) 0 : _Assert(__FILE__":"_STR(__LINE__)" "#test))
```

`_Assert()`是一个隐藏库函数，用于调用`<stdio.h>`中的其它库函数输出错误信息并调用`abort()`函数，这个很简单，没有什么问题，上述代码的关键在于后面几行宏定义上。

----------

`__FILE__`及`__LINE__`这两个宏是由编译器定义的，代表当前文件名及当前代码行号，`__FILE__`是一个字符串，而`__LINE__`是一个十进制整数。

`_STR()`及`_VAL()`这两个宏神奇的实现了将一个**整数常量**转换为**字符串字面量**的功能，二者缺一不可，也就是说，下面这个写法是**错误的**：

``` C
#define _STR(x) #x
```

使用这个写法的话，`_STR(__LINE__)`得到的是`"__LINE__"`而不是期望的结果。
另外，`_STR()`及`_VAL()`这两个宏就是一般的宏，其名字没有任何特殊之处，改成`aaa()`一类的依然可以正常使用。至于这两个宏到底为何能起到这样的作用，想了很久也没有想清楚……只有留待将来哪时候能不能理解清楚了。

最后一句宏定义还涉及到一个字符串常量拼接操作，这也是我之前不知道的……要是想把几个字符串常量拼接起来，直接将其写在一起就可以了，中间**不需要也不能用**`+`一类的符号进行连接。

最后还有个神奇的`#`符号，书中叫做**字符串创建操作符**，这个符号可以起到把大多数信息转成字符串字面量的作用，最重要的是，这个符号似乎**只能用于宏定义中**，如下面这段程序是完全正确的：

``` C
#define MYSTR(x) #x

char * tmpstr = MYSTR(100)"\n";
```

然而改成这样就是错误的了：

``` C
char * tmpstr = #100"\n";
```

考虑到预处理命令都是以`#`开头的，`#`本身应该也算是一个预处理指令，所以并不能直接用于程序中。

