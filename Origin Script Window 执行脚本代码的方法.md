title: Origin Script Window 执行脚本代码的方法
weburl: Origin Script Window 执行脚本代码的方法
date: 2016-01-27 17:09:38
tags: []
categories: 工具之术

---

在[Origin](http://www.originlab.com/)中，有一个Script Window可用于执行LabTalk和X-Function脚本程序，可通过菜单栏上的`Window`->`Script Window`打开。**然而这个Script Window有一个大坑，就是它诡异的执行代码的方式。**

<!--more-->

这个Script Window是像这样的：
![](https://img.gaomf.cn/Software20160127165933.png)

界面上并没有一个执行按钮，按正常思路应该是输完一行代码后敲一下回车就会执行这行代码了，**然而，这个Script Window却不是这样的，其运行代码的方式极为诡异……**

在Script Window中，**输完一行代码后按回车键的作用是正常的换行**，要想执行代码，需要**把光标输入点移动到这一行之前的任意位置处（即不要是行尾即可），然后再按回车键，这样就会执行这一行代码了**。另一个更坑的问题是，这样也仅仅只是执行这一行代码，并不是执行所有代码……**要执行某一行代码就需要把光标移动到这一行中间按回车才行……**
