title: 为Hexo博客Yelee主题添加Gitment评论系统
toc: false
mathjax: false
fancybox: false
tags: [Blog]
permalink: Hexo_Yelee_Gitment
categories: 工具之术
date: 2018-11-04 21:36:31

------

本来博客使用的是多说作为评论系统，前两年多说停止服务了换成了友言，用了没多久友言又要求备案不能用了……后面由于工作繁忙也就没管这个了。前段时间发现Gitment这个基于Github Issue的评论系统不错，这两天终于有空把它给加上了。

<!--more-->

我使用的主题是基于[Yelee](https://github.com/MOxFIVE/hexo-theme-yelee)做了些修改得到的，Yelee又是基于[Yilia](https://github.com/litten/hexo-theme-yilia)的，添加Gitment的过程可以参考这篇文章：

> [Hexo 添加 Gitment 评论](https://sogrey.github.io/article/Hexo-%E6%B7%BB%E5%8A%A0-Gitment-%E8%AF%84%E8%AE%BA/)

Yiila主题也添加了Gitment支持，其[Commit](https://github.com/litten/hexo-theme-yilia/commit/af58957e14a00b3da03e4026c56d34cdf7eda9b4)也是很有参考价值的。

与以上教程有区别的是，无需安装Gitment npm插件，添加修改的代码我也改了下，有兴趣的话可以看这个[Commit](https://github.com/g199209/BlogTheme/commit/bc591586bd737f0f24a08c54f36f6e10372050c6)。

其中Gitment的CSS & JS文件改为了本地压缩后的版本，评论框的显示效果也调整了下。

终于评论系统又可以用啦，之后就该静心学学技术提高下自己的水平了……