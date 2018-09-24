title: Atom禁用GPU启动的方法
permalink: Atom_Disable_GPU
toc: false
mathjax: false
fancybox: false
tags:
  - 工具
categories:
  - 杂七杂八
date: '2017-09-06 22:51'
---

正如在[之前的文章](/2017/08/30/Win10_Software_Font_Blur/)中提到，Atom在Win10下若不禁用GPU加速的话，界面字体会模糊，启动时可以通过附加`--disable-gpu`选项来禁用GPU，然而在打开关联文件等时候，是无法直接设置命令行参数的，此时就需要做些配置来实现这一目的。

<!--more-->

### 修改快捷方式

若要直接打开一个程序，较常用的方法就是通过桌面或开始菜单中的快捷方式，此时可以在快捷方式属性中指定命令行参数即可，如图所示：

![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170906160347.png)

### 修改右键菜单

默认情况下，Atom会自动新建一个`Open with Atom`的右键菜单项，这个功能还是很实用的，要实现此处打开的Atom也能附加命令行参数，需要通过修改注册表实现。在`regedit`中，找到`\HKEY_CLASSES_ROOT\*\shell\Atom`项，这就对应右键菜单中那个命令，其默认项为菜单项的名称，`Icon`项指向一个`exe`文件，对应缩略图图标，不要去修改它。继续展开其`command`子目录，其默认项就是调用的命令，在最后添加一个`--disable-gpu`即可，例如：`"C:\Users\g1992\AppData\Local\atom\atom.exe" "%1" --disable-gpu`。最先为Atom的目录，之后的`"%1"`应该就是系统传入的待打开的文件，最后为我们加上的附加参数。注意，`disable-gpu`必须加在`"%1"`之后，加在前面Atom无法正常打开文件。

### 修改文件关联

要修改文件关联调用的程序，也要通过修改注册表实现。系统的打开方式中列出的程序其实是由注册表中`\HKEY_CLASSES_ROOT\Applications\`目录决定的，每一个子项对应一个程序。找到`\HKEY_CLASSES_ROOT\Applications\atom.exe`项，这就对应打开方式中Atom项。`\HKEY_CLASSES_ROOT\Applications\atom.exe\shell\open`目录下的`FriendlyAppName`项就是打开方式中程序的名称，而其下的`command`目录的默认项就对应调用的命令行。同理在最后添加一个`--disable-gpu`即可。

------------

通过以上配置，基本可以实现无论通过哪种方式打开Atom均可以完全禁用GPU加速的效果了~

----------

> 参考资料：
> [修改注册表实现文件默认打开方式](http://www.360doc.com/content/13/0518/07/4299739_286250789.shtml)
