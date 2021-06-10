title: Hexo Git部署警告"warning： LF will be replaced by CRLF"的去除方法
weburl: Hexo_Git_CRLF
toc: false
mathjax: false
fancybox: false
tags: [Blog]
categories: 工具之术
date: 2017-01-13 20:24:11

---

Windows下在使用`hexo d`命令部署博客时，会出现下面这个警告：

```no-highlight
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in index.html.
```

这个警告的意思很直接，就是Git会把`LF`替换为`CRLF`，不过这是无关紧要的，完全可以禁用此功能，这样还可以避免这个警告信息刷屏。设置方法也很简单，在MinGW窗口中输入以下命令即可：

```bash
git config --global core.autocrlf false
```

> 参考资料：
> [Windows git “warning: LF will be replaced by CRLF”, is that warning tail backward?](http://stackoverflow.com/questions/17628305/windows-git-warning-lf-will-be-replaced-by-crlf-is-that-warning-tail-backwar)