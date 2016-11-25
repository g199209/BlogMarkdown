title: C语言标准库总结
date: 2016-04-18 12:52:05
tags: [C语言, Top]
categories: 编程

---

标准库（Standard Library）是C语言重要的一部分，不过学习C语言这么长时间，都没有细致的了解过标准库到底中包含哪些内容，这几天打算来仔细看看这部分内容。

C语言标准库有各种不同的实现，比如最著名的[glibc](https://www.gnu.org/software/libc/)， 用于嵌入式Linux的[uClibc](https://uclibc.org/)，还有ARM公司的自己的C语言标准库及精简版的MicroLib等。不同标准库的实现并不相同，而且提供的函数也不完全相同，不过有一个它们都支持的最小子集，这也就是最典型的C语言标准库。

这个C语言标准库中一共包含15个头文件，粗略的按常用程度排序列举如下：

<!--more-->

| Header File | Content |
| ----------- | --------|
| **[`<stdio.h>`](#stdio) ** | **输入和输出** |
| **[`<stdlib.h>`](#stdlib)**  | **最常用的一些系统函数** |
| **[`<string.h>`](#string)**  | **字符串处理** |
| **[`<math.h>`](#math)**  | **数学函数** |
| **[`<ctype.h>`](#ctype)**  | **字符类测试** |
| **[`<time.h>`](#time)**  | **时间和日期** |
| **[`<stdarg.h>`](#stdarg) ** | **可变参数列表** |
| **[`<signal.h>`](#signal)**  | **信号** |
| **[`<assert.h>`](#assert)**  | **断言** |
| [`<setjmp.h>`](#setjmp)  | 非局部跳转 |
| [`<errno.h>`](#errno)  | 定义错误代码 |
| [`<stddef.h>`](#stddef)  | 一些常数、类型和变量 |
| [`<locale.h>`](#locale)  | 本土化 |
| [`<float.h>`](#float)  | 浮点数运算 |
| [`<limits.h>`](#limits)  | 定义整数数据类型的取值范围 |


在这里不准备列举所有库函数的详细用法，如需查询具体的库函数用法，可以参考以下几个链接：
> [Standard C 语言标准函数库速查 (Cheat Sheet)](http://ganquan.info/standard-c/)
> [C标准库参考手册](http://wiki.jikexueyuan.com/project/c/c-standard-library.html)
> [C Standard Library Reference Tutorial](http://www.tutorialspoint.com/c_standard_library/)

本文总结的是不完整的C标准库，仅列举一些常用且最重要的部分。

关于C标准库本身的实现分析，可参考我的学习笔记：
[C标准库学习笔记(1)——time、ctype、stdarg、assert](/2016/08/02/C%E6%A0%87%E5%87%86%E5%BA%93%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%281%29%E2%80%94%E2%80%94time%E3%80%81ctype%E3%80%81stdarg%E3%80%81assert/)

<span id="stdio"></span>
## **stdio.h** ##
输入和输出。

在其中定义了以下一些常用的类型及常量：

| Name | Comment |
| ------- | --------|
| `FILE`   | 文件指针 |
| `EOF`    | End Of File，表示文件的结尾 |
| `stderr` | 标准错误流 |
| `stdin`  | 标准输入流 |
| `stdout` | 标准输出流 |

其中`stderr`、`stdin`、`stdout`为宏定义，是指向`FILE`类型的指针。

`<stdio.h>`中的函数有很多，大致可分为对标准输入输出流的操作、对文件流的操作、对标准错误流的操作、对字符串的操作这几大类。

### 标准输入输出流
其实从`stdin`与`stdout`的定义中也可以看到，标准输入输出流也就是文件，只是一般情况下已经默认定义为键盘和屏幕。这与Linux中一切皆文件的思想一脉相承。

常用的函数有以下这些：

| Name | Comment |
| ------- | --------|
|`int printf(const char * format, ...)`  | 格式化输出数据至`stdout`|
|`int scanf(const char * format, ...)`   | 由`stdin`读取格式化输入数据|
|`int putchar(int c)` | 向`stdout`输出一个字符|
|`int getchar(void)` | 由`stdin`读入一个字符|
|`int puts(const char * s)`    | 向`stdout`输出一串字符串|
|`char * gets(char * s)`    | 由`stdin`读入一串字符串|

另外，`vprintf()`函数主要用于需要自己实现一些类似`printf()`的函数时使用，关于这个函数的用处可参考[StackOverflow上的讨论](http://stackoverflow.com/questions/1485805/whats-the-difference-between-the-printf-and-vprintf-function-families-and-when)，用于文件流的`vfprintf()`与用于字符串的`vsprintf()`的用处也是相似的。

### 文件流
对文件的操作是`<stdio.h>`中的核心，其他函数均可视为对特定文件的操作，大部分函数均以`f****()`命名。

最重要的函数是以下这几个：

| Name | Comment |
| ------- | --------|
|`FILE * fopen(const char * filename, const char * mode)`  | 打开文件，失败返回`NULL`|
|`int fclose(FILE * stream)` | 关闭文件，成功返回0，失败返回`EOF`|
|`size_t fread(void * ptr, size_t size, size_t nmemb, FILE * stream)`  | 读取文件内容|
|`size_t fwrite(cosnt void * ptr, size_t size, size_t nmemb, FILE * stream)` | 写入文件内容|

只使用这4个函数就可以完成基本的文件读写操作了，其它函数可以视为是为了更方便的进行文件读写而引入的。在Linux中，文件不仅仅是指磁盘上的一个file，也有可能是一个设备等，不过都可以以统一的方式进行读写。常用的打开模式有`r`(读)、`w`(写)、`a`(附加)、`b`(二进制)等。

----------

与标准输入输出流的操作相同，对文件的操作也有以下这些函数：

| Name | Comment |
| ------- | --------|
|`int fprintf(FILE * stream, const char * format, ...)` | 格式化输出数据至文件|
|`int fscanf(FILE * stream, cosnt char * format, ...)`  | 由文件读取格式化输入数据|
|`int putc(int c, FILE * stream)`    | 向文件输出一个字符|
|`int getc(FILE * stream)`    | 由文件读入一个字符|
|`int fputc(int c, FILE * stream)`   | 向文件输出一个字符|
|`int fgetc(FILE * stream)`   | 由文件读入一个字符|
|`int fputs(const char * s, FILE * stream)`   | 向文件输出一串字符串（或比特流）|
|`char * fgets(char * s, int n, FILE * stream)`   | 由文件读入一串字符串（或比特流）|

其中`putc()`与`fputc()`、`getc()`与`fgetc()`的区别在于前者可能是使用宏定义实现的，而后者一定是函数，具体分析可以参考[这篇文章](http://www.cnblogs.com/aqxin/archive/2011/05/20/2052069.html)。

----------

用于对文件进行修改（如删除文件等）的函数有以下这些：

| Name | Comment |
| ------- | --------|
|`int remove(const char * filename)`  | 删除文件，成功返回0|
|`int rename(const char * old, const char * new)`  | 更改文件名称或位置，成功返回0|
|`FILE * tmpfile(void)` | 以wb+形式创建一个临时二进制文件|

其中`tmpfile()`创建的临时文件在调用`fclose()`关闭时会被自动删除。

----------

对文件流的定位通常使用以下这些函数：

| Name | Comment |
| ------- | --------|
|`int fseek(FILE * stream, long int offset, int fromwhere)`  | 移动文件流的读写位置，错误返回非0|
|`long int ftell(FILE * stream)`  | 取得文件流的读取位置|
|`void rewind(FILE * stream)` | 重设读取目录的位置为开头位置|
|`int feof(FILE * stream)`   | 检测文件结束符|

`whence`可设置为`SEEK_SET`、`SEEK_END`或`SEEK_CUR`。

----------

使用这两个函数处理读写文件流操作中的错误：

| Name | Comment |
| ------- | --------|
|`int ferror(FILE * stream)`   | 检查流是否有错误|
|`void clearerr(FILE * stream)` | 复位错误标志|

----------

与缓冲(Buffer)机制有关的函数常用的有以下这两个：

| Name | Comment |
| ------- | --------|
|`void setbuf(FILE * stream, char * buf)` | 把缓冲区与流相联|
|`int fflush(FILE * stream)` | 更新缓冲区，成功返回0，错误返回`EOF`|

### 其他流操作
对`stderr`的操作通过以下函数完成：

| Name | Comment |
| ------- | --------|
|`void perror(const char * s)` | 打印出错误原因信息字符串 |

此函数将上一个函数发生错误的原因输出到`stderr`，此错误原因依照全局变量`errno`的值来决定要输出的字符串，`errno`在`<errno.h>`中声明。

----------

对字符串也提供了格式化输入输出函数：

| Name | Comment |
| ------- | --------|
|`int sprintf(char * s, const char * format, ...)` | 格式化字符串复制 |
|`int sscanf(const char * s, const char * format, ...)`  | 格式化字符串输入 |

<span id="stdlib"></span>
## stdlib.h ##

最常用的一些系统函数。

在其中定义了以下一些常用的类型及常量：

| Name | Comment |
| ----------- | --------|
| `size_t`  | `sizeof`运算符产生的数据类型，一般是一个无符号整数 |
| `wchar_t`  | 一个宽字符的大小 |
| `NULL` | 空 |
| `RANDMAX` | `rand()`的最大返回值 |

下面分类整理一下其中的重要函数。

### 内存管理函数

最常用的是以下两个函数：

| Name | Comment |
| ------- | --------|
|`void * malloc(size_t size)` | 从堆上动态分配内存空间 |
|`void free(void * ptr)`   | 释放之前分配的内存空间 |

还有一些常用的内存控制函数位于`<string.h>`中。

### 数学函数

常用函数有：

| Name | Comment |
| ------- | --------|
|`int abs(int j)`   | int类型数据绝对值 |
|`long labs(long j)`  | long类型数据绝对值|
|`int rand(void)`  | 产生一个随机数|
|`void srand(unsigned int seed)` | 初始化随机数种子 |

关于`rand()`与`srand()`的用法，之前写的[这篇文章](/2015/11/16/C%E8%AF%AD%E8%A8%80%E7%94%9F%E6%88%90%E9%9A%8F%E6%9C%BA%E6%95%B0/)中进行了总结。

### 字符串转换函数

常用的有以下这3个函数：

| Name | Comment |
| ------- | --------|
|`int atoi(const char * nptr)` | 将字符串转换为整数（int）|
|`long atol(const char * nptr)` | 将字符串转换为长整数（long）|
|`double atof(const char * nptr)` | 将字符串转换为浮点型数（double）|

### 环境函数

常用的函数有：

| Name | Comment |
| ------- | --------|
|`int system(const char * string)` | 执行Shell（或命令行）命令|
|`char * getenv(const char * name)` | 获取环境变量中的内容 |
|`int exit(int stauts)`   | 结束进程 |

### 搜索和排序函数

| Name | Comment |
| ------- | --------|
|`void qsort(void * base, size_t nmemb, size_t size, int (* compar)(const void *, const void *))`   | 快速排序算法|
|`void * bsearch(const void * key, const void * base, size_t nmemb, size_t size, int (* compar)(const void *, const void *))` | 在数组进行二分法查找某一元素，要求数组预先已排好序|

----------

在`<stdlib.h>`中还有一些用于进行多字节字符处理的函数，此处没有列出。

<span id="string"></span>
## string.h ##

`<string.h>`中除了字符串处理函数，还有一些内存管理函数：

| Name | Comment |
| ------- | --------|
|`void * memset(void * dest, int c, size_t n)` | 将一段内存空间填上某值|
|`void * memcpy(void * dest, const void * src, size_t n)` | 复制一段内存内容|
|`int memcmp(const void * s1, const void * s2, size_t n)` | 比较两段内存内容|
|`void * memchr(const void * s, int c, size_t n)` | 在某一段内存范围中查找特定字节|

----------

常用的字符串操作函数有：

| Name | Comment |
| ------- | --------|
|`char * strcat(char * deat, const char * src)` | 连接两个字符串 |
|`char * strcpy(char * dest, const char * src)` | 复制字符串|
|`int strcmp(const char * s1, const char * s2)` | 比较两个字符串|
|`size_t strlen(const char * s)` | 获取一个字符串的长度|
|`char * strtok(char * s1, const char * s2)` | 分割字符串|

以下这些函数用于进行字符串查找：

| Name | Comment |
| ------- | --------|
|`char * strchr(const char * s, int c)`  | 正向查找一个字符|
|`char * strrchr(const char * s, int c)` | 反向查找一个字符|
|`char * strstr(const char * s1, const char * s2)`  | 查找一个字符串|
|`char * strpbrk(const char * s1, const char * s2)` | 查找一个字符集合|

<span id="math"></span>
## math.h
标准数学库，常用函数如下：

### 三角函数

| Name | Comment |
| ------- | --------|
|`double sin(double x)` | 正弦|
|`double cos(double x)` | 余弦|
|`double tan(double x)` | 正切|
|||
|`double asin(double x)` | 反正弦|
|`double acos(double x)` | 反余弦|
|`double atan(double x)` | 反正切|
|`double atan2(double y, double x)` | 计算y/x的反正切|

### 双曲三角函数

| Name | Comment |
| ------- | --------|
|`double sinh(double x)` | 双曲正弦|
|`double cosh(double x)` | 双曲余弦|
|`double tanh(double x)` | 双曲正切|

### 指数与对数

| Name | Comment |
| ------- | --------|
|`double exp(double x)`            | e的n次幂|
|`double pow(double x, double y)`  | x的y次幂|
|`double sqrt(double x)`           | 开根号|
|||
|`double log(double x)`   | e为底的对数|
|`double log10(double x)` | 10为底的对数|

### 取整

| Name | Comment |
| ------- | --------|
|`double ceil(double x)`  | 向上取整|
|`double floor(double x)` | 向下取整|

### 其它

| Name | Comment |
| ------- | --------|
|`double fabs(double x)` | 计算绝对值|

<span id="ctype"></span>
## ctype.h ##

包含字符测试及大小写转换函数。

### 字符测试

| Name | Comment |
| ------- | --------|
|`isalpha(c)`  | 是否为字母|
|`isupper(c)`  | 是否为大写字母|
|`islower(c)`  | 是否为小写字母|
|||
|`isdigit(c)`  | 是否为数字|
|`isxdigit(c)` | 是否为16进制数字（数字 & A~F & a~f）|
|||
|`isalnum(c)`  | 是否为字母及数字|
|||
|`ispunct(c)`  | 是否为标点符号|
|`isspace(c)`  | 是否为空白字符（空格、\r(CR)、\n(LF)、\t(TAB)、\v(VT)、\f(FF)）|
|`iscntrl(c)`  | 是否为控制字符（ASCII 0 ~ 37(0x1F) & 177(0x7F)）|
|||
|`isgraph(c)`  | 是否为可显示字符（字母 & 数字 & 标点）|
|`isprint(c)`  | 是否为可打印字符（字母 & 数字 & 标点 & 空白）|

### 大小写转换

| Name | Comment |
| ------- | --------|
|`tolower(c)` | 转换为小写|
|`toupper(c)` | 转换为大写|

<span id="time"></span>
## time.h ##

日期及时间操作。定义了`time_t`、`clock_t`及`tm`这几种类型，常用函数有：

### 获取时间及相关计算

| Name | Comment |
| ------- | --------|
|`time_t time(time_t * timer)` | 获取UNIX时间戳，一般传入`NULL`|
|`clock_t clock(void)`         | 获取CPU时钟计数|
|`double difftime(time_t time1, time_t time0)` | 计算时间差，`time1` - `time0`|
|||
|`struct tm * gmtime(const time_t * timer)`    | GMT时间|
|`struct tm * localtime(const time_t * timer)` | 地方时时间|
|`time_t mktime(struct tm * timeptr)`          | 地方时时间|

### 转换为可阅读的字符串

| Name | Comment |
| ------- | --------|
|`char * ctime(const time_t * timer)`         | 返回标准时间字符串，地方时时间，等价于`asctime(localtime())`|
|`char * asctime(const struct tm * timeptr)`  | 返回标准时间字符串|
|`size_t strftime(char *s, size_t maxsize, const char *format, const struct tm *)` | 返回自定义格式时间字符串|

<span id="stdarg"></span>
## stdarg.h ##

用于支持可变参数，定义了`va_list`这个结构体，通过以下三个宏进行操作：

| Name | Comment |
| ------- | --------|
|`void va_start(va_list ap, parmN)` | 初始化`va_list`|
|`type va_arg(va_list ap, type)`    | 从`va_list`中获取一个`type`类型的参数|
|`void va_end(va_list ap)`          | 释放`va_list`|

<span id="signal"></span>
## signal.h ##

定义了信号(Signal)处理的相关宏及函数，这与Linux中的信号机制密切相关，包含下面两个函数：

| Name | Comment |
| ------- | --------|
|`signal()`        | 设置处理特定Signal的Handler|
|`raise(int sig)`  | 产生一个Signal|

`signal()`函数原型如下：
`void (* signal(int sig, void (* handler)(int)))(int);`

<span id="assert"></span>
## assert.h ##

此头文件的唯一目的是提供`assert(int x)`这个宏，如果断言非真，程序会在标准错误流输出错误信息，并调用`abort()`函数使程序异常终止。

<span id="setjmp"></span>
## setjmp.h ##

非局部跳转，用于从一个深层次嵌套中直接返回至最外层，通过这两个宏完成：

| Name | Comment |
| ------- | --------|
|`int setjmp(jmp_buf env)`            | 设置跳转点|
|`void longjmp(jmp_buf env, int val)` | 进行跳转|

<span id="errno"></span>
## errno.h ##

声明了一个外部整形变量`errno`用于表示错误，可用`perror(const char * s)`输出错误原因，其中`s`是错误提示前缀。

标准使用方法是：在一个库函数调用之前把它设为0，然后在下一个库函数调用前测试它，任何非零值均表示错误。示例代码：

``` C
#include <errno.h>
#include <math.h>

//......
    errno = 0;
    y = sqrt(x);
    if (errno != 0)
        perror("Error");
```

<span id="stddef"></span>
## stddef.h ##
定义了一些标准定义，如`size_t`、`wchar_t`、`NULL`等，这些定义也会出现在其他的头文件里。还定义了以下这个宏：

| Name | Comment |
| ------- | --------|
|`offsetof(type, member)` | 返回结构体中某一成员相对于结构体起始地址的偏移量|

<span id="locale"></span>
## locale.h ##

国家、文化和语言规则集称为区域设置，主要影响字符串格式，通过以下函数进行设置：

| Name | Comment |
| ------- | --------|
|`setlocale()` | 设置或恢复本地化信息|

<span id="float"></span>
## float.h ##

用宏定义的方式定义了浮点数的最大值、最小值等信息。

<span id="limits"></span>
## limits.h ##

定义了基本数据类型（int、char、short等）的最大值及最小值。






