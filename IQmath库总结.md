title: C28x IQmath库使用
date: 2016-05-15 14:21:27
tags: [DSP]
categories: 科研

---

IQmath库是TI C28x系列DSP中使用的一个高度优化且高精度的数学库，用于使用定点算法实现浮点运算。在DSP编程中，出于性能的考虑，应尽量使用IQmath库代替ANSI C中的math库。IQmath库同时支持C和C++，此处仅讨论使用C语言的情况。

IQmath的官方使用手册为`SPRC990 C28x IQmath Library`，在ControlSUITE中可以找到这份文档的最新版本。

<!--more-->

## **使用方法** ##
1. **添加库文件`IQmath.lib`或`IQmath_f32.lib`，后者用于带FPU单元的DSP；**
2. **包含头文件`IQmathLib.h`；**
3. **修改CMD文件，添加其中与IQmath相关的部分。**

## **Q格式** ##
[Q格式](https://en.wikipedia.org/wiki/Q_%28number_format%29)是一种定点小数表示方法，具体来说，二者的转换关系为：

**定点数 = 浮点数 * 2^Q **

在C2000中，统一使用32位来表示。Q*N*格式对应数据类型为_iq*N*，表示使用*N*位来表示小数，其余32-*N*位表示整数，故Q0其实就是一般的32位整数。具体的Q1~Q30的取值范围及精度见下表：

![](http://gmf.shengnengjin.cn/20160515095003.png)

`_iq`类型代表使用`GLOBAL_Q`定义的精度，`GLOBAL_Q`在`IQmathLib.h`文件中定义，默认为：

``` C
#ifndef GLOBAL_Q
#define GLOBAL_Q 24 /* Q1 to Q29 */
#endif
```

可直接更改`IQmathLib.h`文件中的定义，也可使用如下方法覆盖此定义：

``` C
#define GLOBAL_Q 21 /* Set the Local Q value */
#include <IQmathLib.h>
```

## **常用函数** ##

函数命名惯例为：`_IQNxxx()`，其中`N`代表使用的Q格式，省略的话使用`GLOBAL_Q`中的定义；`xxx`为函数名，如`sin`、`exp`等。下面列举一些常用函数，其中的`N`大部分都可以省略使用`GLOBAL_Q`。

### 格式转换

|函数原型|说明|
|-------|----|
|**`_iqN _IQN(float F)`**|将`float`转为`_iqN`格式|
|**`float _IQNtoF(_qiN A)`**|将`_iqN`格式转为`float`|
|**`_iqN _IQtoIQN(_iq A)`**|将全局`_iq`格式转为`_iqN`格式|
|**`_iq _IQNtoIQ(_iqN A)`**|将`_iqN`格式转为全局`_iq`格式|
|**`long _IQNint(_iqN A)`**|提取整数部分|
|**`_iqN _IQNfrac(_iqN A)`**|提取小数部分|
|**`_iqN _atoIQN(char *S)`**|将字符串转为`_iqN`格式|
|**`int _IQNtoa(char *S, const *format, long x)`**|将`_iqN`格式转为字符串|

### 算数运算
对于相同`_iqN`格式的两个数之间的加减法，可以使用`+`、`-`直接进行，注意不要溢出即可；对于不同`_iqN`格式的两个数，需要转换为相同`_iqN`格式后再进行加减。

|函数原型|说明|
|-------|----|
|**`_iqN _IQNmpy(_iqN A, _iqN B)`**|基本乘法|
|**`_iqN _IQNrsmpy(_iqN A, _iqN B)`**|带凑整(rounding)和饱和(saturation)的乘法|
|**`_iqN _IQNmpyI32(_iqN A, long B)`**|`_iqN`与`long`相乘|
|**`_iqN _IQNdiv(_iqN A, _iqN B)`**|基本除法|

### 三角函数

|函数原型|说明|
|-------|----|
|**`_iqN _IQNsin(_iqN A)`**|正弦|
|**`_iqN _IQNcos(_iqN A)`**|余弦|
|**`_iqN _IQNtan(_iqN A)`**|正切|
|**`_iqN _IQNasin(_iqN A)`**|反正弦|
|**`_iqN _IQNacos(_iqN A)`**|反余弦|
|**`_iqN _IQNatan(_iqN A)`**|反正切|

### 其它常用函数

|函数原型|说明|
|-------|----|
|**`_iqN _IQNabs(_iqN A)`**|绝对值|
|**`_iqN _IQNexp(_iqN A)`**|exp|
|**`_iqN _IQNlog(_iqN A)`**|自然对数|
|**`_iqN _IQNsqrt(_iqN A)`**|开平方|
|**`_iqN _IQNisqrt(_iqN A)`**|开平方后取倒数|
|**`_iqN _IQNmag(_iqN A, _iqN B)`**|欧氏距离|

### Benchmark
完整的IQmath函数及性能评估表如下：
![](http://gmf.shengnengjin.cn/20160515101508.png)
