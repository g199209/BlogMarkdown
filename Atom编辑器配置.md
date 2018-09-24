title: Atom编辑器配置
permalink: Atom_Configure
toc: true
mathjax: false
fancybox: false
tags:
  - 工具
categories:
  - 杂七杂八
date: '2017-09-16 10:22'
---

之前使用过各种文本编辑器，[Notepad++](https://notepad-plus-plus.org/)、[Sublime Text](https://www.sublimetext.com)、[VIM](http://www.vim.org/)等，使用下来觉得各有千秋，都有些不太满意的地方。前段时间体验了下[Atom](https://atom.io/)，感觉很不错~作为一个所谓的"A hackable text editor for the 21st Century"，还是有很多可以折腾配置的东西，此处总结整理下。

<!--more-->

## 基本使用

Atom是Github社区推出的开源编辑器，即可以开箱即用无需配置，也可以充分的去定制折腾：

> Atom is a text editor that's modern, approachable, yet hackable to the core—a tool you can customize to do anything but also use productively without ever touching a config file.

Atom官方的使用手册见文末第一个参考资料链接。

### 下载安装

一般从其[主页](https://atom.io/)上下载编译好的二进制程序文件即可，Windows下一般使用`exe`安装包。安装过程完全不需要用户参与，也无法参与……安装包会将Atom自动安装到C盘的某个位置，且这是无法更改的。

### 命令面板（Command Palette）

Windows下，使用`ctrl-shift-p`键即可打开命令面板，在这个命令面板中可以使用模糊搜索快速查找所需的命令，而无需像传统方式那样在层层菜单中寻找。如设置界面，可从菜单中的`File`->`Setting`打开，更快的方式是在命令面板中搜索`setting`，即可看到所需命令及其快捷键：

![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170827162506.png)

### 主题与字体

在设置界面中，`Themes`一项用于设置界面及语法高亮主题，尝试下来`One`这个主题最好看，还提供了`One Light`及`One Dark`两套亮色及暗色主题。如果对自带的几个主题配色不满意的话，可以去官方的[几千个主题](https://atom.io/themes)中挑选自己喜欢的，或者干脆自己自定义一个~不过自带的这两个主题已经可以完全满足我的需求了，就不再去折腾了……

至于编辑器界面的字体与字号，可在`Editor`选项卡中配置，一般选择等宽字体，如我选择的`YaHei Consolas Hybrid`。

### 查找文件

Atom中，通过文件夹的方式来组织管理项目，界面左侧就是`Project`面板。若要查找项目中的文件，可使用`ctrl-p`快捷键；使用`ctrl-b`查找已打开的文件。

### 编辑&移动

Atom中本身就有一些光标移动的快捷键，不过更方便的是使用下文将要提到的Vim扩展插件，此处仅列举一些比Vim更好用的功能：

- 跳转到某行：`ctrl-g`，之后输入行号
- 跳转到标签：`ctrl-r`，如要在工程内搜索，使用`ctrl-shift-r`
- 将当前行（或选中的几行）上下移动：`ctrl-↑/↓`，此功能在调整代码时会很有用

Atom中还有个很强大的多光标功能，按住`ctrl`键的同时用鼠标在多个位置点击或选取，即可启用多光标编辑模式。也可以选中一个词后，按`ctrl-d`键即可选择下一个相同的词。

另外，Atom中的括号等不仅会自动配对，而且选中某一个部分后输入单边括号，编辑器会自动用此括号把选中内容包围起来。

### 书签

使用`alt-ctrl-F2`键来添加或删除书签，使用`F2`键可以调到下一个书签，`ctrl-F2`打开书签列表。

### 搜索和替换

与大部分软件一样，`ctrl-f`打开搜索替换面板，`ctrl-shift-f`在整个工程中执行搜索替换。也可以使用正则表达式，在替换时若需反向引用，使用`$1`, `$2`...

## 各种高级功能

Atom内置了很多高级编辑功能，此处列出一些常用的，详见[Atom Flight Manual](http://flight-manual.atom.io/)中的帮助说明。

### Snippets

常用代码片段，输入响应关键词后，会自动弹出提示，此时按`Tab`键即可自动输入。不同文件类型有不同的默认代码片段，各种package也会添加自己的snippets扩展，可在命令面板中输入`Snippets:Available`查看当前可用的snippets。如果要自定义snippets，可参考官方文档的说明：[Creating Your Own Snippets](http://flight-manual.atom.io/using-atom/sections/snippets/#creating-your-own-snippets)

### Git集成

Atom是Github发布的编辑器，自然内置了很好的Git支持，界面右下角即可看到当前状态，点击后即可打开Git面板。不过和大部分IDE中集成的Git一样，此处的Git只适合日常工作时commit新代码，若要查看之前的提交记录或进行一些更复杂的操作，还是使用专门的软件更好，比如[GitKraken](https://www.gitkraken.com/)。

值得一提的是，编辑器左侧使用不同颜色直接标出了此文件的修改情况，十分的直观~使用快捷键`alt-g-↑/↓`可以快速切换到下一个修改过的地方。

## 插件

Atom本身是一个很轻量级的编辑器框架，它的各种功能都是由不同插件实现的，称为Packages，这也就是配置Atom中最好玩的部分了~插件的安装有几种不同的方法，最简单的方法是，在`Settings`->`Install`中直接搜索安装即可。此处记录下我安装的插件。

### 配置备份

Atom的所有配置文件其实都位于`.atom`文件夹下，理论上可通过自行备份这个文件夹来实现配置备份，不过此处我们有一个更优雅的解决方案：`Sync-setttings`插件。此插件基于Github gist进行备份，所以还需要进行一些额外的配置，详见下面两篇文章：

> [Sync-setttings(插件-备份神器)](http://wiki.jikexueyuan.com/project/atom/sync-settings.html)
> [Atom 编辑器配置sync-setting](http://elickzhao.github.io/2016/10/Atom%20%E7%BC%96%E8%BE%91%E5%99%A8%E9%85%8D%E7%BD%AEsync-setting/)

### Vim扩展

Vim最好用的地方应该是其移动方式，在Atom中也可以通过安装`vim-mode-plus`这个插件实现基础的Vim功能，这个插件是对官方Vim插件的升级加强，功能还是很完善的。这个插件还有一个[Wiki页面](https://github.com/t9md/atom-vim-mode-plus/wiki)，里面有各种帮助文档，其中[Advanced Topic Tutorial](https://github.com/t9md/atom-vim-mode-plus/wiki/AdvancedTopicTutorial)部分介绍了各种高级编辑技巧，很值得一读。

`vim-mode-plus`插件默认绑定了很多快捷键，如果需要自定义的话，可以修改`keymap.cson`文件。比如我就根据自己的需求加入了以下一些配置：

```nohight
'atom-text-editor.vim-mode-plus':
  # 保证通用的复制粘贴快捷键可用
  'ctrl-c': 'core:copy'
  'ctrl-v': 'core:paste'

'atom-text-editor.vim-mode-plus:not(.insert-mode)':
  'H': 'vim-mode-plus:move-to-first-character-of-line'
  'L': 'vim-mode-plus:move-to-last-character-of-line'
  'ctrl-/': 'vim-mode-plus:toggle-line-comments'
  'space space': 'vim-mode-plus:toggle-fold'
```

Vim中还有个很好用的插件叫EasyMotion，可以实现快速移动定位，同样在Atom中，也可以通过安装`Jumpy`这个插件来实现这一目的。此插件默认绑定的快捷键是`shift-Enter`，可以将`vim-mode-plus`中的`f`键替换为这个插件：

```nohight
'atom-text-editor:not(.mini).vim-mode-plus:not(.insert-mode):not(.jumpy-jump-mode)':
  'f': 'jumpy:toggle'
```

### Markdown扩展

Atom本身就支持Markdown语法高亮的，再配合一些插件就可以很完美的作为一个Markdown编辑器使用了：

- `markdown-preview-enhanced` : Markdown预览增强版，提供了各种高级功能，详见其[文档](https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/)。安装好后可以把Atom自带的`markdown-preview`给禁用掉
- `Markdown-Writer` : 各种辅助增强功能，功能介绍见其[文档](https://github.com/zhuochun/md-writer/wiki/Features)

`Markdown-Writer`有一个很方便的功能就是自动生成草稿文件(draft)和发布(publish)完成的草稿文件，这需要先在其设置中配置一下，相应的`config.cson`文件如下：

```
"markdown-writer":
  fileExtension: ".md"
  frontMatter: '''
    title: <title>
    permalink:
    toc: false
    mathjax: false
    fancybox: false
    tags:
    categories:
    date: <date>

    ---



    <!--more-->

    ----------

    > 参考资料：
    > []()
  '''
  siteDraftsDir: "draft"
  siteEngine: "hexo"
  siteLocalDir: "D:\\Github\\Hexo"
  sitePostsDir: "source\\_posts"
  urlForCategories: "http://gaomf.cn/categories.json"
  urlForPosts: "http://gaomf.cn/posts.json"
  urlForTags: "http://gaomf.cn/tags.json"
```

最后三个选项是为了实现自动管理标签和分类用的，这需要另一个Hexo插件[hexo-generator-atom-markdown-writer-meta](https://github.com/timnew/hexo-generator-atom-markdown-writer-meta)配合，这个插件有些小Bug，在Hexo 3上无法正常使用，不过有人给出了解决方案，打上[这个补丁](https://github.com/timnew/hexo-generator-atom-markdown-writer-meta/pull/5/files)即可。

这样配置好后，再配合内置Terminal，就可以实现在Atom内完成博客写作的全过程了~

### minimap

Sublime Text中有一个很好用的文档缩略图功能，在滚动时屏幕右侧会显示此文档的缩略图。在Atom中也可以通过插件实现这一功能，这就是`minimap`插件，除了这个基础插件外，还有几个扩展插件用于增强其功能：

- `minimap-autohider` : 在不滚动鼠标时自动隐藏minimap缩略图；
- `minimap-bookmarks` : 在缩略图中高亮显示书签；
- `minimap-find-and-replace` : 在缩略图中高亮显示搜索结果；
- `minimap-git-diff` : 在缩略图中高亮显示当前文档修改情况；
- `minimap-highlight-selected` : 在缩略图中高亮显示选中的单词；
- `minimap-split-diff` : 在缩略图中显示`split-diff`插件的比较结果；

### Terminal集成

Atom中的terminal插件有很多，使用下来最好用的是`platformio-ide-terminal`，在Windows下默认调用的是`powershell`。安装好此插件后，点击界面左下角的加号即可新建一个terminal，其显示风格也可以自定义，用起来极为方便。

### 编程辅助

最常用的两个功能就是自动补全和代码检查，通过插件`autocomplete-plus`和`linter`实现，这两个插件都只是一个框架，针对不同语言有不同语言的扩展。针对C/C++，可安装`autocomplete-clang`和`linter-clang`，这两个插件都依赖于`clang`，所以要先装好`clang`。`clang`是LLVM的一部分，下载链接在[这里](http://releases.llvm.org/download.html)，在Windows下就是一个exe安装包，直接安装即可，不过需要注意的是，**安装时一定要选择添加到环境变量，否则无法正常使用**。安装好后，在命令行中输入`clang -v`，若有输出说明安装正确。

关于这两个插件的详细使用和配置可参考其文档。另外，`linter`的显示似乎还需要一个名为`Linter-UI-Default`的插件支持，也要一起安装上。

其他一些推荐插件还有：

- `atom-ctags` : 为当前工程生成tag索引，以便实现跳转，默认跳转快捷键为`F12`，`shift-F12`返回；
- `symbols-tree-view` : 与`atom-ctags`配合使用，以列表的形式列出tag；

最后一个待解决的问题就是调试器，要是能把GDB等集成到Atom里面就完美了，这个之后再来折腾……

> Update 2017-10-05:
> 对于Python来说，可以安装`autocomplete-python`和`linter-pylama`，Python的语法补全和错误检查比C语言用起来感觉更好~
> 关于调试器，可以使用`dbg-gdb`插件

### 其它插件

- `highlight-selected` : 高亮当前选中的单词
- `file-icons` : 添加一些文件类型的图标，更为美观
- `split-diff` : 可以对比两个文件的差异

## 杂项

使用过程中也遇到了一些零散的问题，此处记录一下。

Q: 无法通过拖拽打开文件，即将文件拖入一个已经打开的Atom窗口界面时，显示红色的禁止标志。
A: 不要使用管理员身份运行Atom即可。

Q: 界面字体发虚。
A: [Win10下软件界面显示模糊问题解决办法](/2017/08/30/Win10_Software_Font_Blur/)
   [Atom禁用GPU启动的方法](/2017/09/06/Atom_Disable_GPU/)

Q: 安装Package时网络连接错误。
A: 这是由于GFW把相关网站墙掉了的缘故……开VPN或者是系统代理即可，使用代理的话，在`Settings`->`Core`中找到`Use Proxy Settings When Calling APM`，勾选上此选项即可。

----------

最后，使用了一段时间后觉得，Atom有各种好，不过它也有个最大的缺点，就是慢，真的好慢啊…………

----------

> 参考资料：
> [Atom Flight Manual](http://flight-manual.atom.io/)
> [Atom编辑器入门到精通(一) 安装及使用基础](http://blog.csdn.net/u010494080/article/details/50372857)
> [atom在vim模式下设置快捷复制按键](https://www.urlteam.org/2017/07/atom%E5%9C%A8vim%E6%A8%A1%E5%BC%8F%E4%B8%8B%E8%AE%BE%E7%BD%AE%E5%BF%AB%E6%8D%B7%E5%A4%8D%E5%88%B6%E6%8C%89%E9%94%AE/)
> [Windows 软件系列-atom插件](https://draapho.github.io/2016/10/12/1610-WinSoft-atompack/)
