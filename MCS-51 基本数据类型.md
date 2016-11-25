title: MCS-51 基本数据类型
date: 2016-03-25 15:08:44
tags: [C语言, MCS-51]
categories: 嵌入式

---

以Keil C51编译器为例，MCS-51上基本数据类型及其占用空间如下表：

<!--more-->

| 数据类型 | 占用空间 | 范围 |
| ------- | -------- | ---- |
| char | 1 Byte (8 bits) | 0 ~ 255 |
| signed char | 1 Byte (8 bits) | -128 ~ +127 |
| short | 2 Bytes (16 bits) | -32768 ~ +32767 |
| unsigned short | 2 Bytes (16 bits) | 0 ~ 65535 |
| int | 2 Bytes (16 bits) | -32768 ~ +32767 |
| unsigned int | 2 Byte (16 bits) | 0 ~ 65535 |
| long | 4 Bytes (32 bits) | -2147483648 ~ +2147483647 |
| unsigned long | 4 Bytes (32 bits) | 0 ~ 4294967295 |
| float | 4 Bytes (32 bits) | ±1.175494E-38～±3.402823E+38 |
| double | 4 Bytes (32 bits) | ±1.175494E-38～±3.402823E+38 |
| * (Pointer) | 3 Bytes (24 bits) | - |
| bit | 1 bit | 0 or 1 |
| sbit | 1 bit | 0 or 1 |
| sfr | 1 Byte (8 bits) | 0 ~ 255 |
| sfr16 | 2 Bytes (16 bits) | 0 ~ 65535 |

关于MCS-51中的指针，分为通用指针（3字节）、xdata指针（2字节）、code指针（2字节）、idata指针（1字节），详见以下文章：
> [C51:Keil c51指针变量](http://blog.csdn.net/onicolascage/article/details/46670373)
> [关于Keil C51指针的使用](http://wenku.baidu.com/view/7f42b0e19b89680203d825e7.html)

参考资料：
> [KEIL C51编译器所支持的数据类型及各其长度](http://www.baiheee.com/Documents/100623/100623155050.html) 
