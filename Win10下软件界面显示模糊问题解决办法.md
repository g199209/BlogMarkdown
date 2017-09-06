title: Win10下软件界面显示模糊问题解决办法
permalink: Win10_Software_Font_Blur
toc: false
mathjax: false
fancybox: false
tags:
  - 工具
categories:
  - 杂七杂八
date: '2017-08-30 16:20'
---

某些软件在使用过程中会出现显示显示模糊的问题，鼠标动一下或者点一下菜单显示又清晰起来。使用的系统是Win 10，笔记本显卡为GT 730M，开启了系统DPI缩放。遇到此问题的软件有：

- Visual Studio 2015
- Atom
- MarkdownPad

<!--more-->

我遇到的问题与这两个问题基本完全一样：

> [atom，vs2017、vscode 在 win10 下界面显示模糊怎么办？](https://www.zhihu.com/question/57583720)
> [Visual Studio 2015字体发虚、模糊解决方案](http://jingyan.baidu.com/article/6181c3e0ba537f152ef1532d.html)

解决方法正如其中所说，关掉GPU硬件加速即可。不过对于Atom来说，使用配置`useHardwareAcceleration: false`是**无效的**，必须在启动时附加命令行参数`--disable-gpu`才行。

此问题显然应该是某个Bug造成的，但具体是什么原因导致的并没有查到……
