title: U-Boot版本命名规则及源码下载途径
date: 2016-05-27 23:18:37
tags: [Bootloader]
categories: 点滴之间

---

U-Boot的正式发布版本（即Released Versions）每2个月更新一次，并且正常情况下都是在此月中旬的某个周一进行更新，这被称为“[U-Boot Release Cycle](http://www.denx.de/wiki/U-Boot/ReleaseCycle)”。
U-Boot所有发布版本的源码皆可以从其官方ftp上下载到，地址为：[ftp://ftp.denx.de/pub/u-boot/](ftp://ftp.denx.de/pub/u-boot/)

<!--more-->

其源代码使用Git进行管理，如果想获取最新的代码或参与贡献代码，可追踪其[Git Repository](http://git.denx.de/u-boot.git/)。

自从2008年10月之后，U-Boot正式发布版本的命名就使用了基于时间戳的版本号，版本号中包含发布年份及月份，rc后缀代表Release Candidate。几个例子如下：

> U-Boot v2009.11     - Release November 2009
> U-Boot v2009.11.1   - Release 1 in version November 2009 stable tree
> U-Boot v2010.09-rc1 - Release candidate 1 for September 2010 release

如对各发布版本的一些统计信息有兴趣，可参考其官网上的[Statistics](http://www.denx.de/wiki/U-Boot/ReleaseCycle)。

