title: Vim配置及插件总结
permalink: Vim_Plugin
toc: true
mathjax: false
fancybox: false
tags: [Linux]
categories: 杂七杂八
date: 2017-02-02 16:42:48

---

Vim之所以强大很大程度上是源于其灵活的配置及丰富的插件，本文就对我的配置文件及用到的插件做一总结记录，插件用法主要给出参考链接，不做过多重复说明。

<!--more-->

## 配置文件

如果不对Vim进行配置的话，其界面和功能都不是很好用，Vim的配置文件在Linux下是`~/.vimrc`文件，不过为了方便管理和使用Git进行备份同步，一般将其放在`~/.vim/.vimrc`处，并在`~/`和`/root/`路径下建立硬链接。

Vim的配置文件中有一个被称为最强大的终极配置文件——[spf13](https://github.com/spf13/spf13-vim)，这是一系列插件和配置文件的集合，不过我没有直接使用这个，只是参考了下它`.vimrc`文件的写法，我自己使用的配置文件在[这里](https://github.com/g199209/vimrc)，其中的注释写得很详细，就不多做说明了。

## 插件管理

Vim的插件管理功能本身也是靠插件实现的，这类插件有很多：

> [vim有哪些插件管理程序？都有些什么特点？](https://www.zhihu.com/question/24294358/answer/27362814)

我选用的是使用最广泛且最简单的Vundle。

### [Vundle](https://github.com/VundleVim/Vundle.vim)

Vundle的Github页面上对其使用方法说得很清楚了，照着做就好。在`.vimrc`文件中配置好相关插件目录，常用命令就这几个：

```no-highlight
:PluginList       - lists configured plugins
:PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
:PluginSearch foo - searches for foo; append `!` to refresh local cache
:PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
```

*注：早期的Vundle使用`Bundle`的形式，不过这种用法已经被遗弃了，目前使用的是`Plugin`的形式。*

## 界面美化插件

### [Solarized](https://github.com/altercation/vim-colors-solarized)

据说最受欢迎的配色方案，使用下来感觉的确不错。在Vundle中添加：

```vim
Plugin 'altercation/vim-colors-solarized'
```

Solarized标准的背景色是深蓝色的，在其Github页面上可以看到效果示意图，不过实际使用下来我却始终无法调出这种背景色来。后面仔细研究了其配色代码，发现蓝色背景色是其针对Gvim的配色，而我是在终端中直接使用Vim的，这时候使用的配色方案背景色是深灰色。当然也可以设置为透明的，即使用终端背景色，不过其实深灰色我看着更舒服，就没有去把它改成蓝色了，只是对部分颜色进行了一些调整，最终得到了一个[定制版的Solarized主题](https://github.com/g199209/vim-colors-solarized)。

最终的显示效果如下：
  
![](http://gmf.shengnengjin.cn/20170117234815.png-width600)

### [Airline](https://github.com/vim-airline/vim-airline)

状态栏增强美化插件，使用方法可参考：

> [vim-airline配置](http://www.zygotee.com/vim/vim-airline%E9%85%8D%E7%BD%AE/)
> [VIM配置:vim-airline插件安装](http://blog.csdn.net/the_victory/article/details/50638810)
> [安装Vim插件vim-airline](http://www.jianshu.com/p/310368097c75)

vim-ariline的使用很简单，基本无需多少配置，且可以自动和诸多其他插件配合使用，十分方便。我使用`bubblegum`主题的显示效果如下：

![](http://gmf.shengnengjin.cn/20170118154146.png-width600)

### [Startify](https://github.com/mhinz/vim-startify)

为Vim添加一个开始界面，可以显示最近使用的文件等。

## 功能增强插件

### [NERDTree](https://github.com/scrooloose/nerdtree)

据说最受欢迎的Vim插件，可以在Vim中显示一个类似工程管理器的界面，用于方便的选择文件打开。一般同时配合以下两个插件一起使用：

- [nerdtree-git-plugin](https://github.com/Xuyuanp/nerdtree-git-plugin)：用于显示Git状态
- [vim-nerdtree-tabs](https://github.com/jistr/vim-nerdtree-tabs)：在多Tab切换时保持NERDTree界面

### [CtrlP](https://github.com/ctrlpvim/ctrlp.vim)

用于快速查找文件的插件。

### [Bookmarks](https://github.com/MattesGroeger/vim-bookmarks)

用于增强书签功能的插件，可在左侧边栏显示书签，而且优化了快捷键。我觉得比[Signature](https://github.com/kshenoy/vim-signature)更简单好用些，按照Github说明配置即可。

### [EasyGrep](https://github.com/dkprice/vim-easygrep)

全局搜索插件。

### [EasyMotion](https://github.com/easymotion/vim-easymotion)

快速移动的神器，方便定位到所需位置。

### [Fixtc.Vim](https://github.com/lilydjwg/fcitx.vim)

在离开或重新进入插入模式时自动记录和恢复每个缓冲区各自的输入法状态，以便在普通模式下始终是英文输入模式，切换回插入模式时恢复离开前的输入法输入模式。

## 编程相关插件

### [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)

代码补全插件，虽然效果还是比不过VS+VAssixtX，不过可以满足基本需求了。如果使用Makefile编译的话，可使用[Bear](https://github.com/rizsotto/Bear)生成索引数据库。

### [Ultisnips](https://github.com/SirVer/ultisnips)

快速插入代码片段，与YCM插件配合使用。

### [AutoPair](https://github.com/jiangmiao/auto-pairs)

自动括号配对。

### [TagList](https://github.com/vim-scripts/taglist.vim)

列出函数、宏定义等。

### [NerdCommenter](https://github.com/scrooloose/nerdcommenter)

快速注释代码。

### [ConqueGDB](https://github.com/vim-scripts/Conque-GDB)

在Vim中集成GDB的插件，只是并不是很好用……

----------


> 参考资料：
> [跟我一起学习VIM - The Life Changing Editor](http://feihu.me/blog/2014/intro-to-vim/)
> [所需即所获：像 IDE 一样使用 vim](https://github.com/yangyangwithgnu/use_vim_as_ide)