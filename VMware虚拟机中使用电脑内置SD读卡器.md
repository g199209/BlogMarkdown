title: VMware虚拟机中使用电脑内置SD读卡器
permalink: VMware_Internal_SD
toc: false
mathjax: false
fancybox: false
tags: [工具, Linux]
categories: 嵌入式
date: 2016-12-28 15:19:19

---

为树莓派SD卡烧写系统时，需要先在虚拟机中识别到SD卡。笔记本电脑有一个内置的SD读卡器，不过将SD卡插入后，虚拟机中是无法直接找到SD卡设备的，需要我们手动添加一下。

<!--more-->

如果使用的是外置USB读卡器，连接上电脑后VMware就会自动发现新的可移动设备，此时选择连接到虚拟机即可正常使用。不过笔记本内置的SD读卡器应该是直接连到PCIe总线上的，并不是一个USB设备，插入SD卡后，相关驱动使其在系统中表现为一块硬盘，这一点可以打开磁盘管理器确认：

![](http://gmf.shengnengjin.cn/20161228145141.png)

此时，要想在虚拟机中访问SD卡，最好的方式就是**添加一块虚拟硬盘，以硬盘的方式去访问SD卡**，具体步骤可参考以下文章：

> [How to Use SD Card Reader in VMPlayer and VMWorkstation](http://www.htpcguides.com/how-to-use-sd-card-reader-in-vmplayer-and-vmworkstation/)

其中虚拟硬盘类型选择`IDE`、`SCSI`或`SATA`都是可以正常使用的，重点在于必须要选择使用完整的磁盘，且设备要选择正确：

![](http://gmf.shengnengjin.cn/20161228150035.png)

设备名称在主机Windows系统下的磁盘管理器中可以看到。需要注意的是，**如果是在打开了VMware后再插入SD卡，VMware是无法识别到新硬盘设备的，此时重启VMware后即可正常识别**。另外，在移除SD卡后需要同时移除这个虚拟硬盘，否则系统无法正常启动。

设置完成后打开Ubuntu虚拟机，使用`sudo fdisk -l`命令即可列出所有的硬盘设备，如果添加正确的话就可以看到名为`/dev/sd*`的SD卡了。

另外值得一提的是，在我的系统环境下，无论添加的是`IDE`还是`SATA`类型的虚拟硬盘，其名称均为`/dev/sd*`，并不会存在`/dev/hd*`。只是选择`IDE`设备的话，新添加的SD卡会出现在前面，即`/dev/sda`；选择`SATA`或`SCSI`设备的话，SD卡会在已有硬盘后面，即`/dev/sdb`。






