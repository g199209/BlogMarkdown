title: 为树莓派设置静态IP地址的方法
weburl: Raspberry_Pi_Static_IP
toc: false
mathjax: false
fancybox: false
tags: [Raspberry Pi, Linux, Network]
categories: 工具之术
date: 2016-10-27 21:31:39

---

Raspbian系统默认是使用DHCP方式获取IP地址的，这就要求要有一个路由器。然而实验室里面并没有有线网和路由器可用……我只能配置成电脑用无线上网，然后用网线连接树莓派这样的网络结构。这就要求树莓派要配置成静态IP的形式，下面来总结一下配置方法。

<!--more-->

在网上搜索“树莓派 静态IP”可以找到很多的教程文章，然而它们大部分都不适用于现在新版本的Raspbian系统了，按那些方法进行配置**是错误的**。2015年5月后，Raspbian Jessie发布，这一版本引入了[dhcpcd](https://wiki.archlinux.org/index.php/dhcpcd)，进而更改了网络部分的配置方式，导致之前的所有教程都失效了……新版本的配置方式和相关讨论见以下链接：

> [How do I set up networking/WiFi/Static IP](http://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip)

这篇文章中提到了两种设置方法，这里对推荐使用的基于dhcpcd的新方法做一总结：

首先，**不要修改**`/etc/network/interfaces`文件；

之后打开`/etc/dhcpcd.conf`文件，在最后添加以下代码：

```
interface eth0
static ip_address=192.168.1.7/24
static routers=192.168.1.3
static domain_name_servers=192.168.175.2
```

其中`eth0`代表有线以太网接口，如要配置无线网，使用`wlan0`；`ip_address`就是IP地址，根据实际情况配置；`routers`是网关地址；`domain_name_servers`是DNS服务器的地址。

最后使用`sudo reboot`命令重启后即可生效。

*相关文章：*

> [VMware虚拟机中嵌入式Linux开发环境网络配置](/2016/06/25/VMware%E8%99%9A%E6%8B%9F%E6%9C%BA%E4%B8%AD%E5%B5%8C%E5%85%A5%E5%BC%8FLinux%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE/)



