title: apt与apt-get命令
permalink: apt_apt-get
toc: false
mathjax: false
fancybox: false
tags: [Linux, 工具]
categories: 杂七杂八
date: 2016-10-25 13:49:16

---

`apt-get`是Debian及Ubuntu这类Linux发行版中最常用的软件安装命令，前几天又发现一个很相似的命令`apt`，也可以用于安装软件。去网上搜索了下这两个命令，整理如下。

<!--more-->

`apt-get`是一个传统的包管理程序，而`apt`似乎是Ubuntu Trusty之后才引入的一个新程序。具体讨论可参考下面这个链接：

> [What is the difference between apt and apt-get?](http://askubuntu.com/questions/445384/what-is-the-difference-between-apt-and-apt-get)

重点内容摘录如下：

> They are very similar command line tools available in Trusty. Apt-get and apt-cache's most commonly used commands are available in apt.
> 
> `apt-get` may be considered as lower-level and "back-end", and support other APT-based tools. `apt` is designed for end-users (human) and its output may be changed between versions.
> 
> The big news for this version is that we included a new `apt` binary that combines the most commonly used commands from `apt-get` and `apt-cache`. The commands are the same as their `apt-get`/`apt-cache` counterparts but with slightly different configuration options.

一言以蔽之，`apt`可以视为是对`apt-get`及`apt-cache`中常用命令的二次打包封装，一般来说，可以直接使用`apt`来替代`apt-get`及`apt-cache`，它们之间并没有什么大的差别，而`apt`使用起来会更方便。

`apt`支持以下命令操作：

|命令|作用|
|----|----|
|`list`|列出已安装的软件包|
|`search`|在源索引文件中搜索软件包|
|`show`|列出特定软件包的详细信息|
|`install`|安装软件包|
|`remove`|移除软件包|
|`edit-sources`|编辑源列表|
|`update`|同步源索引文件|
|`upgrade`|升级现有软件包|
|`full-upgrade`|类似`upgrade`，只是会自动删除冲突的软件包|

