title: Nor Flash中的启动扇区（Boot Sector, Boot Block）
weburl: Nor Flash中的启动扇区（Boot Sector, Boot Block）
date: 2016-06-23 01:04:50
tags: [Flash]
categories: 点滴之间

---

在Nor Flash中，有个启动扇区（Boot Sector，有时也被称为启动块）的概念，这个概念应该是只针对Nor Flash的，因为Nand Flash无法直接寻址，故Nand Flash中是没有Boot Sector的。

一个Nor Flash一般被分为若干块（Block）**或**若干扇区（Sector），这应该只是不同厂家用的名字不同，并不存在一个Block中包含若干Sector的说法。**这些扇区的大小一般并不相同，通常情况下会配置为大量大容量扇区加上少量小容量扇区的结构，这些小扇区就被称之为启动扇区。**

<!--more-->

这样设计是考虑到小容量扇区的擦写速度比较快，而且一般可以对其进行写保护，故可用其存放Bootloader及其他一些用户配置数据。实际上可以有多个小扇区，这些小扇区中具体哪一个是启动扇区，还是都是启动扇区，这一问题并没有明确的说法，也无需纠结于此。

根据启动扇区的地址不同，可分为两大类：Top Boot & Bottom Boot。**Top Boot是指Boot Sector位于最高地址处；Bottom Boot是指Boot Sector位于最低地址处，**一般同一型号的Nor Flash都会同时提供这两个版本，除了地址分配的差异外，其它的主要参数二者应该完全相同。

一张比较清晰的对比图如下：
![](https://img.gaomf.cn/ZnJvbT1jc2RuJnVybD13WndwbUwyQUROMElUTTFFek53Z0RNdklUWnNsbVp3VjNMbjlHYmk5Q2RsNW1MNGxtYjFGbWJwaDJZdWNXYnBkMmJzSjJMdm9EYzBSSGE.jpg)

下面以AMD公司的Am29LV160D为例来具体看看这个问题。

Am29LV160D可配置为2M * 8-Bit或1M * 16-Bit，这个是通过一个引脚电平来确定的，下面仅讨论1M * 16-Bit的配置。在此配置下，Am29LV160D被分为35个扇区：

- 1个8Kword扇区
- 2个4Kwork扇区
- 1个16Kword扇区
- 31个32Kword扇区

这些扇区中，前面那4个扇区就是Boot Sector。AMD公司同时提供了Top Boot和Bottom Boot两种芯片，见下图：
![](https://img.gaomf.cn/20160623005443.png)

对于Top Boot的器件来说，其扇区分配如下：
![](https://img.gaomf.cn/20160623005627.png)
可以看到，其中小容量扇区被安排到了最高地址处，即最后几个扇区。

对于Bottom Boot的器件来说，其扇区分配如下：
![](https://img.gaomf.cn/20160623005756.png)
可以看到，其中小容量扇区被安排到了最低地址处，即最前几个扇区。

----------

不过实际上，像U-Boot这类Bootloader的体积一般都会超过100KByte的，此时显然无法只用启动扇区保存，所以说这时候启动扇区就仅仅只是个名字，使用时也无需特别关注的。不过Top Boot和Bottom Boot器件的器件ID等可能会有所区别，使用时需要注意一下。

