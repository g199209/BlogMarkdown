title: PSpice教程及帮助文档
permalink: PSpice_Tutorial_Help
toc: false
mathjax: false
fancybox: false
tags: [PSpice]
categories: 硬件之理
date: 2017-05-05 17:20:26

---

PSpice的教程在网上随便一搜就有一大堆，不过其实还是Cadence自带的帮助文档最好，这里就来总结下有哪些比较有用的PSpice帮助文档。

<!--more-->

## 基础教程

打开OrCAD Capture CIS，选择`Help`->`Learning PSpice`即可打开一个交互式的Tutorial，这个比各种教程好多了，从最简单的元器件基础内容到各种高级分析方法都介绍到了：

> Cadence OrCAD Solution's "Learning PSpice" is made available to all, for learning various concepts related to Electrical and Electronics engineering. This complete material covers several diverse topics, ranging from basic theorems to very advanced topics in the field of Electrical & Electronics Engineering and electronic design automation etc. 

## 帮助文档

打开Cadence帮助（按F1），在左侧导航栏中可以找到PSpice文件夹，这些就是PSpice相关帮助文档，可以在当前帮助浏览器中直接查看，也可通过菜单栏上的pdf按钮打开对应的pdf文件查看：

![](http://gmf.shengnengjin.cn/20170505165654.png)

下面列举几份比较有用的文档。

### SPICE帮助

> PSpice A/D Reference Guide

在导航菜单中的名字叫做`PSpice Reference Guide`，这份文档其实是对SPICE本身的介绍，包括各种基本元件、分析方法、基本函数、SPICE语法等。实际上使用SPICE分析模拟电路的情况更多一些，故在需要时可以通过其中`Analog devices`部分查阅各种基础元件的模型：

![](http://gmf.shengnengjin.cn/20170505170850.png)

> Detailed descriptions of the simulation controls and analysis specifications, start-up option definitions, and a list of device types in the analog and digital model libraries. User interface commands are provided to instruct you on each of the screen commands.

### PSpice基础文档

> PSpice User Guide
> PSpice Help

PSpice软件相关的文档，Guide相当于一份十分详尽的软件教程，而Help则是用于查阅的帮助文档。在`PSpice Help`中有一部分叫做`Index of PSpice symbol and part properties`，其中包含了所有仿真模型和仿真参数的说明，可以在这里查到每个参数的意思，还有这个元件是包含在那个库文件中的。

### PSpice高级文档

> PSpice Advanced Analysis User Guide
> PSpice Advanced Analysis Help

与上面的`PSpice基础文档`类似，只是介绍的是PSpice中的各种高级功能，比如蒙特卡洛分析、灵敏度分析、优化器等等。
