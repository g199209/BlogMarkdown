title: Linux下Vim的编译安装
permalink: Linux_Vim_Compile
toc: true
mathjax: false
fancybox: false
tags: [Linux, Vim]
categories: 工具之术
date: 2017-01-17 17:01:27

---

基本所有Linux发行版的软件仓库中都有现成的Vim，不过这个发布版包含的功能不全，且一般都不是最新版，故我们需要自己动手，从源码自行编译最新版的Vim。本文以Ubuntu 14.04 及 Debian 3.16为例，介绍编译最新Vim 8.0的过程。

<!--more-->

## 卸载老版本Vim

虽然不卸载老版本的Vim也可以正常编译安装新版本的，不过既然用不到了，将其卸掉可以节省空间.

```bash
apt-get remove vim vim-runtime gvim vim-tiny vim-common vim-gui-common
```

## 下载所需依赖包

首先是基本的编译系统，一般系统都是已经安装了的：

```bash
apt-get install build-essential libncurses5-dev
```

----------

其次是各种脚本语言的支持，如Python、Lua、Perl等，需要注意的是，并不是不写这些程序就不用安装了，很多Vim插件都是依赖于这些功能的，所有最好全部都装上以免之后出现奇怪的问题。

### Python

```bash
apt-get install python python3 python-dev python3-dev
```

### Ruby

```bash
apt-get install ruby ruby-dev
```

### Lua

```bash
apt-get install lua5.2 liblua5.2-dev luajit libluajit-5.1-dev
```

### Perl

```bash
apt-get install perl libperl-dev
```

### Tcl

```bash
apt-get install tcl8.5 tcl8.5-dev libtcl8.5
```

Tcl的最新版本是8.6，不过此处不能选择最新版本，要选8.5版本的。

----------

如果需要GUI支持，还需要安装GUI相关依赖包，因为我不准备使用GVim等，就无需安装这些了，如有需要可阅读参考资料中的网页。

最后还需要安装一些杂项：

```bash
apt-get install exuberant-ctags cscope
```

## 获取源代码

```bash
git clone https://github.com/vim/vim.git
```

切换到`src`文件夹中，之后所有操作都在此文件夹中完成。

```bash
cd vim/src
```

## 配置

使用`./configure --help`可查看帮助，我使用的设置是：

```bash
./configure \
    --with-compiledby="Mingfei Gao" \
    --with-features=huge \
    --enable-multibyte \
    --enable-cscope=yes \
    --enable-pythoninterp=yes \
    --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu \
    --enable-python3interp=yes \
    --with-python3-config-dir=/usr/lib/python3.4/config-3.4m-x86_64-linux-gnu \
    --enable-perlinterp=yes \
    --enable-rubyinterp=yes \
    --enable-luainterp=yes \
    --with-luajit \
    --enable-tclinterp=yes \
    --enable-gui=no \
    --enable-fail-if-missing
```

## 编译安装

```bash
make -j 4 && sudo make install
```

这一步一般不会发生什么错误，编译时间也不是很长，几分钟就可以完成了。

## 查看版本信息及清理

打开Vim后使用`:version`查看版本信息：

![](https://gmf.shengnengjin.cn/20170117170247.png-height600)

最后清理下编译中间文件：

```bash
make clean && make distclean
```

现在就可以尽情享受最新版的Vim了~



> Update 2018-03-03:
>
> 以上编译选项编译出的vim可能会缺少clipboard支持，这会导致与系统剪贴板交互存在问题。同时为简化之后重复编译升级vim的过程，可编写一个脚本文件来自动执行上述操作，详见：
>
> https://github.com/g199209/vimrc



----------


> 参考资料：
> [Debian下Vim的编译](http://www.jianshu.com/p/3e0c242310d3)
> [Linux中源码安装编译Vim](http://www.cnblogs.com/zhongcq/p/3615980.html)
> [源码编译Vim 8](https://segmentfault.com/a/1190000007005137)