title: Linux下让进程在后台可靠运行的方法
permalink: Linux_Background_Run
toc: false
mathjax: false
fancybox: false
tags: [Linux]
categories: 工具之术
date: 2017-01-10 21:05:52

---

在登录到Linux服务器后运行某程序，之后断开连接，那之前运行的程序就会被中止掉。这是由于新进程默认都是当前进程的子进程，断开连接关闭当前终端就会把它的所有子进程都结束掉。不过很多时候我们需要让程序稳定的一直运行下去，这时候就需要使用一些方法来处理此问题了。IBM有一篇很好的文章深入探讨了此问题，本文就是对它的简化总结：

> [Linux 技巧：让进程在后台可靠运行的几种方法](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)

<!--more-->

下文使用`python test.py`作为示例命令。

### nohup &
`nohup`用于让提交的命令忽略`hangup`信号：

```no-highlight
# nohup python test.py
```

一般同时加上`&`把命令放到后台运行

```no-highlight
# nohup python test.py &
```

### setsid
`setsid`的作用是在一个新的`session`中运行命令：

```no-highlight
# setsid python test.py
```

提交的新进程的父进程会是`init`，即`PPID=1`。

### ( &)

```no-highlight
# (python test.py &)
```

提交的新进程的父进程也是`init`。

### disown
`disown`用于让某个已经在运行的程序忽略`hangup`信号，先将此任务放入后台中运行，之后使用如下命令即可：

```no-highlight
# disown -h %jobspec
```

其中`jobspec`是任务的作业号。

### screen
`screen`用于运行很多需要放到后台中稳定运行的命令时使用，用法如下：

```no-highlight
# screen -dmS newScreen
# screen -list
# screen -r newScreen
# python test.py
```

详细说明见IBM的原始文章。另外，在Ubuntu下，默认是没有安装`screen`的，使用命令`apt-get install screen`进行安装即可。