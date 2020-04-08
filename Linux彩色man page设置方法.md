title: Linux彩色man page设置方法
permalink: Linux_Colourful_Man
toc: false
mathjax: false
fancybox: false
tags: [Linux]
categories: 工具之术
date: 2017-01-13 19:08:36

---

Ubuntu中默认的`man`帮助页面是黑白的，如果将其改成彩色的会更方便阅读。实现方法主要有三种：使用`most`、使用`terminfo`、配置`.bashrc`文件。本文将介绍最简单的第三种方法。

<!--more-->

Linux下`man page`的显示默认是通过`less`来完成的，故在`.bashrc`文件中添加`less`的相关设置参数即可使`man page`变成彩色的:

```bash
# colourful man page
export LESS_TERMCAP_mb=$'\E[01;34m'
export LESS_TERMCAP_md=$'\E[01;34m'
export LESS_TERMCAP_me=$'\E[0m'
export LESS_TERMCAP_us=$'\E[01;32m'
export LESS_TERMCAP_ue=$'\E[0m'
export LESS_TERMCAP_so=$'\E[01;33;44m'
export LESS_TERMCAP_se=$'\E[0m'
```

更改完`.bashrc`文件后要用`source .bashrc`命令重新载入一下配置，之后重启终端才会生效，以上配置显示效果如下：

![](https://pic.gaomf.store/20170113183033.png-width600)

其中`LESS_TERMCAP_xx`的含义如下：

|`termcap`|含义|
|--------|----|
|`mb`|start blink|
|`md`|start bold|
|`me`|turn off bold, blink and underline|
|`us`|start underline|
|`ue`|stop underline|
|`so`|start standout|
|`se`|stop standout|

对照上面实际的`man page`页面可以看到：`md`对应蓝色部分；`us`对应绿色部分；`so`对应底部黄色状态栏。

至于具体颜色设置方法，参考以下页面：

> [ANSI escape code - Colors](https://en.wikipedia.org/wiki/ANSI_escape_code#Colors)

简而言之，在`'\E[0x;3y;4zm'`中：`x`代表是否加粗，`1`为加粗，`2`为正常；`y`和`z`分别代表文字前景色和背景色，使用默认值的话可省略，颜色列表如下：

<table><tr><th>Intensity</th><th>0</th><th>1</th><th>2</th><th>3</th><th>4</th><th>5</th><th>6</th><th>7</th></tr><tr><th>Normal</th><td style="background: black;color:white">Black</td><td style="background:maroon;color:white">Red</td><td style="background: green;color:white">Green</td><td style="background: olive;color:white">Yellow</td><td style="background: navy;color:white">Blue</td><td style="background: purple;color:white">Magenta</td><td style="background: teal;color:white">Cyan</td><td style="background: silver;color:black">White</td></tr><tr><th>Bright</th><td style="background: gray;color:white">Black</td><td style="background: red;color:black">Red</td><td style="background: lime;color:black">Green</td><td style="background: yellow;color:black">Yellow</td><td style="background: blue;color:white">Blue</td><td style="background: fuchsia;color:black">Magenta</td><td style="background: cyan;color:black">Cyan</td><td style="background: white;color:black">White</td></tr></table>