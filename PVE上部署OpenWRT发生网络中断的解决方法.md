title: PVE上部署OpenWRT发生网络中断的解决方法
permalink: PVE_OpenWRT_Network_Broken
toc: false
mathjax: false
fancybox: false
tags:
  - Network
  - Virtualization

categories: 工具之术
date: 2019-07-28 11:27:09

---


在旧笔记本上使用Proxmox搭建了一个OpenWRT软路由，正常使用都很稳定，然而当PC使用百度网盘，迅雷等工具进行全速率下载时偶尔会出现网络中断问题，此时Proxmox宿主机的网络会全部断掉，即PVE自己的Web管理界面也无法登录。查看终端，此时会不断打印`Detected Hardware Unit Hang`的错误提示。

<!--more-->

Google一下这个错误提示，还是有不少类似问题的：

> [Proxmox: enp0s31f6: Detected Hardware Unit Hang](https://jhartman.pl/2018/08/06/proxmox-enp0s31f6-detected-hardware-unit-hang/)
>
> [解决FreeNAS under KVM使用Virtio网卡导致宿主机网卡Hang的问题](https://ovear.info/post/356)
>
> [e1000e Reset adapter unexpectedly / Detected Hardware Unit Hang](https://serverfault.com/questions/616485/e1000e-reset-adapter-unexpectedly-detected-hardware-unit-hang)
>
> [How to fix “eth0: Detected Hardware Unit Hang” in Debian 9?](https://superuser.com/questions/1270723/how-to-fix-eth0-detected-hardware-unit-hang-in-debian-9)
>
> [Proxmox Node freezes](https://forum.proxmox.com/threads/proxmox-node-freezes.44618/)

基本所有文章都提到此问题与`TCP checksum offload`特性有关，解决方案就是关掉`checksum offload`。具体方法是使用`ethtool`工具：

```bash
ethtool -K enp0s25 tx off rx off
```

如果要重启后永久生效的话将此命令写入`/etc/network/if-up.d/ethtool2`文件中并为此文件加上`x`权限即可：

```bash
#!/bin/sh
ethtool -K enp0s25 tx off rx off
```

------

除此之外上述第2篇文章的情况和我遇到的很像，里面提到这与`Virtio`虚拟化有很大关系，而我使用的也正是`Vritio`，根据作者的说法，更应该在OpenWRT而不是Proxmox中关闭`checksum offload`。然而实际试了下却发现一个蛋疼的问题，OpenWRT中是无法把`tx checksum offload`给关掉的……

此外作者还提到，将网卡的虚拟化方式从`Virtio`改为`E1000`也可以解决此问题，不过会有CPU占用率上升的副作用。

--------

综合以上几种方法，我最后采用的解决办法是：禁用Proxmox宿主机上的`TCP checksum offload`，并将OpenWRT使用的网卡虚拟化方式改为`E1000`。实际测试下来没有再发生网卡hang的问题，满速率下载（250Mbps左右）时CPU占用率50%左右，比之前使用`Virtio`时CPU占用率要高10%左右，还是可以接受的。

-----

问题算是解决了，最后顺带去进一步学习了下相关的知识，首先是`TCP checksum offload`，此技术的作用是将计算TCP  checksum的工作由CPU软件实现改为由NIC设备（即网卡等）硬件实现，以此达到节约CPU资源的目的。

> [Checksum Offloading](https://www2.cs.duke.edu/ari/trapeze/freenix/node7.html)
>
> [TCP checksum offload](https://www.ibm.com/support/knowledgecenter/en/ssw_aix_71/performance/tcp_checksum_offload.html)
>
> [UDP的checksum计算与硬件Offload](https://blog.csdn.net/sinat_20184565/article/details/82979778)

另外就是`Virtio`与`E1000`，这是两种不同的网络虚拟化技术，`Virtio`是半虚拟化而`E1000`是全虚拟化。对于全虚拟化方案来说，虚拟机是完全感知不到自己是运行在一个虚拟环境中的；而半虚拟化则是虚拟机知道自己就是运行在一个虚拟环境中，此时IO驱动就可以做一些针对性的修改优化，以此降低虚拟化层进行转换带来的开销及性能损失。显而易见，半虚拟化技术的隔离度是没有全虚拟化好的，而且要是虚拟机驱动有问题会导致宿主机也出问题。这就是为什么在使用`Virtio`时，OpenWRT网络出现问题会导致整个Proxmox的网络都不能用了的原因。除了这两种虚拟化方式外，还有些更为先进的虚拟化技术，如`SR-IVO`等，有兴趣的话可以看看下面这篇文章的总结：

> [KVM虚拟化网络优化技术总结](https://blog.51cto.com/xiaoli110/1558984)

