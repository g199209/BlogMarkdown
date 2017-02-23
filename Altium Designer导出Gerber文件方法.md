title: Altium Designer导出Gerber文件方法
date: 2016-03-04 20:05:35
tags: [工具]
categories: 硬件

---

详细步骤参考[此文档](http://d1.amobbs.com/bbs_upload782111/files_27/ourdev_540478.pdf)与[这篇文章](http://pcbask.maihui.net/?/article/18)，此处仅简要记录一下所需的操作步骤。

<!--more-->

1. 将坐标原点设置为板子的左下角

2. 在`Drill Drawing`层添加字符串`.Legned`，这是孔位图生成的位置，一般放在板子右侧下方

3. 输出`Gerber Files`
3.1 `General`选项卡中，单位`Inches`，格式`2:5`
3.2 `Layers`选项卡中，选中所需层，一般不要选`Mirror`，不要选`Mechanical Lyaers(s) to Add to All Plots`中的层
3.3 `Drill Drawing`选项卡中，勾选`Drill Drawing Plots`，`.Legend Symbols`选`Characters`
3.4 `Apetures`选项卡中勾选`RS274X`
3.5 `Advanced`选项卡中，选取`Suppress leading zeros`、`Reference to relative origin`、`Unsorted(raster)`，其余用默认值即可

4. 输出`NC Drill Files`，单位`Inches`，格式`2:5`，选取`Suppress leading zeros`、`Reference to relative origin`、`Optimize change location commands`，其余用默认值即可

5. 打包输出文件，并不是所有输出文件都是必要的，不过出于简单考虑，可以全部一起打包即可

