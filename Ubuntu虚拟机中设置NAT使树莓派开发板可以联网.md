title: Ubuntu虚拟机中设置NAT使树莓派开发板可以联网
permalink: Ubuntu_NAT_Raspberry
toc: true
mathjax: false
fancybox: false
tags: [Linux, 树莓派]
categories: 嵌入式
date: 2016-11-01 21:05:07

---

树莓派3自带了WiFi，可以将其连接到无线热点上来联网，不过实验室没有无线路由器可用，只能将其用双绞线连接到电脑上，此时要实现树莓派能正常联网就需要在电脑上的Ubuntu虚拟机中设置NAT转发，这里来记录一下设置过程。

<!--more-->

## 网络拓扑结构

基本的网络结构如下所示：

![](http://7xnwyt.com1.z0.glb.clouddn.com/20161101160822.png)

其中蓝色线条表示逻辑上的网络连接关系。树莓派上的LAN接口使用双绞线与电脑连接，在虚拟机中设置桥接模式，即会产生一块虚拟网卡`eth1`桥接到已有的以太网上构成局域网。虚拟机中还有一块虚拟网卡`eth0`，这是通过VMware的NAT模式产生的，VMware中运行着一个NAT服务实现了`eth0`到`VMnet8`这两块虚拟网卡间的转发，最终实现了虚拟机联网的目的。

从图中可以看到，目前树莓派只连接到了一个局域网中，并没有和Internet建立连接，要实现联网的目的，最简单的方法就是在Ubuntu虚拟机中建立`eth1`至`eth0`的NAT转发。

## 实现方法

### 开启IPv4转发

Linux内核本身就是支持IPv4转发的，只需要开启这个功能即可。使用以下命令：

```bash
sudo echo "1" > /proc/sys/net/ipv4/ip_forward
```

修改`ip_forward`文件可以实时启用IP转发，不过这是临时的，重启后会失效，要永久启用，需要修改配置文件`/etc/sysctl.conf`。在Ubuntu 14.04中，默认有下面这两行注释：

```bash
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
```

根据说明去掉下面那行的注释即可：

```bash
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

也可以在文件末尾直接添加上述语句，效果是一样的。要让修改实时生效，使用以下命令重新加载`sysctl.conf`文件：

```bash
sudo sysctl -p
```

### 配置iptables

首先确定目前防火墙的设置不会拦截来自树莓派开发板的数据包，之后使用以下命令添加NAT转发即可：

```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.1.7/32 -o eth0 -j MASQUERADE
```

`192.168.1.7/32`是树莓派的IP地址；`eth0`是可以联网的网卡。

这个配置也是实时临时生效的，要让其永久生效，需要将其写入配置文件中。

首先，使用`iptables-save`工具将目前的`iptables`规则保存为文件：

```bash
sudo iptables-save > /etc/iptables.rules
```

之后修改`/etc/network/interfaces`文件，在文件最后加上：

```bash
pre-up iptables-restore < /etc/iptables.rules
```

`iptables-restore`工具用于载入之前保存的规则文件；`pre-up`表示建立`interface`之前执行操作，类似的还有`up`、`post-down`等，可用命令`man interfaces`查询。这里就表示在每次建立接口前加载预设规则。

### 配置网关及DNS服务器

这一步是在树莓派开发板上进行的操作，根据[之前文章](/2016/10/27/Raspberry_Pi_Static_IP/)的做法，我们为树莓派配置的是固定IP地址，故同时还要设置网关和DNS服务器，网关需要设置为Ubuntu虚拟机中`eth1`网卡的IP地址，以便将所有数据包送至Ubuntu虚拟机中。至于DNS服务器，经过测试，最好的选择是将DNS服务器设置为VMware软件生成的那个虚拟NAT网关的地址，这样VMware会自动选用主机使用的DNS服务器。

如果要临时更改DNS服务器，可修改`/etc/resolve.conf`文件：

```bash
nameserver 192.168.195.2
```

----------

通过上述方法进行设置后，树莓派就可以上网啦~此方式的实质是将Ubuntu虚拟机作为一台路由器进行网络地址转换，从而为树莓派开发板提供网络访问。




