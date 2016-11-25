title: crosstool-ng的安装
date: 2016-06-14 15:36:34
tags: [Linux, 工具]
categories: 嵌入式

---

crosstool-ng用于制作交叉编译器，其本身作为Linux下的一个软件，也需要进行编译安装。下面简要记录下安装过程。

<!--more-->

开发环境：
Ubuntu 16.04 64bit
gcc v5.3.1
make v4.1

## 下载
在crosstool-ng的首页[crosstool-ng.org](http://crosstool-ng.org/)上可以下载到其源代码，目前最新版本是[crosstool-ng-1.22.0](http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.22.0.tar.bz2)。

## 配置
解压下载到的tar包，切换到`crosstool-ng`文件夹中执行
``` shell
./configure
```

这一步的目的是生成`Makefile`文件，使用`configure`的默认配置即可。

在Ubuntu环境下，因为安装的开发工具不全，可能会提示缺少一些工具，这时候用`apt-get`安装缺少的工具即可。不过有几个提示不是那么直接：

错误提示：
``` shell
configure: error: missing required tool: makeinfo
```
解决方法：
``` shell
apt-get install texinfo
```

----------

错误提示：
``` shell
configure: error: could not find GNU awk
```
解决方法：
``` shell
apt-get install gawk
```

----------

错误提示：
``` shell
configure: error: could not find GNU libtool >= 1.5.26
```
解决方法：
``` shell
apt-get install libtool libtool-bin
```
这里仅安装`libtool`是不够的，还需要安装`libtool-bin`。

----------

错误提示：
``` shell
configure: error: could not find curses header, required for the kconfig frontends
```
解决方法：
``` shell
apt-get install libncurses5-dev
```
这里缺少的是`ncurses`这个库，对应的Ubuntu下的包是`libncurses5-dev`。

## 编译
配置完生成`Makefile`后直接用`make`进行编译即可，这一步在我使用的开发环境下没有发生任何错误。可能出现的错误及解决方法可参考[这里的说明](http://www.crifan.com/files/doc/docbook/crosstool_ng/release/html/crosstool_ng.html#crosstoool_ng_install_self_common_errors)。

## 安装
编译完后使用`make install`安装即可，默认安装在`/usr/local/bin`、`/usr/local/lib`等目录下，如需更改安装目录，可在`./configure`配置时进行设置，具体可使用`./configure -h`命令查看帮助文档中的说明。一般来说，`/usr/local/bin`目录已经在环境变量`PATH`中了，故不需要再手动添加。

安装完成后使用`ct-ng version`命令进行测试，如果能正确输出如下版本信息说明安装正确：
``` shell
This is crosstool-NG version crosstool-ng-1.22.0

Copyright (C) 2008  Yann E. MORIN <yann.morin.1998@free.fr>
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
```

