title: Windows防火墙允许ping请求设置方法
permalink: Windows_Ping
toc: false
mathjax: false
fancybox: false
tags: [Network]
categories: 工具之术
date: 2016-10-27 19:20:41

---

Windows系统出于安全性考虑，在开启了防火墙时，默认情况下是不会响应来自其他主机的ping请求的。这不利于开发过程中检测网络连通性，故这时就需要开启防火墙中的ICMP回显功能。

<!--more-->

以Windows 10为例，打开防火墙高级设置界面，在入站规则中允许ICMP回显即可，如图：

![](http://gmf.shengnengjin.cn/20161027191757.png)