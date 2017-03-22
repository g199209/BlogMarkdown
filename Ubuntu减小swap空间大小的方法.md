title: Ubuntu减小swap分区大小的方法
permalink: Ubuntu_Reduce_Swap
toc: true
mathjax: false
fancybox: false
tags: [Linux, 工具, Top]
categories: 杂七杂八
date: 2017-02-28 19:35:14

---

这几天在新买的服务器上装上了Ubuntu Server 14.04，然而安装系统时没注意swap分区大小设置，导致swap分区占用了很大的磁盘空间，因为服务器本身的内存就很大，所以显然不需要分配这么多swap空间，在网上搜索一番找到了减小swap空间的方法，记录于此。

<!--more-->

## 定位swap分区

首先要确定swap分区的位置才能进行之后的调整操作，可以简单的使用`free -h`命令查看当前内存及swap空间的使用情况：

```no-highlight
# free -h

             total       used       free     shared    buffers     cached
Mem:           15G       1.5G        14G       1.0M       160M       1.0G
-/+ buffers/cache:       282M        15G
Swap:         1.0G         0B       1.0G
```

可以看到swap空间有1G，不过这是收缩了一次后的结果，最初的swap空间是16G……

`free`命令只显示了swap分区的大小，为了找到swap分区所在位置，需要具体查看一下分区情况。因为Ubuntu Server 14.04安装时默认使用了GPT分区形式，所以应该使用`parted -l`而不是`fdisk -l`来查看分区情况：

```no-highlight
# parted -l

Model: ATA EEKSATA60G1702 (scsi)
Disk /dev/sda: 60.0GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name                  Flags
 1      1049kB  538MB   537MB   fat32        EFI System Partition  boot
 2      538MB   794MB   256MB   ext2
 3      794MB   60.0GB  59.2GB                                     lvm


Model: Linux device-mapper (linear) (dm)
Disk /dev/mapper/Xeon--vg-root: 58.2GB
Sector size (logical/physical): 512B/512B
Partition Table: loop

Number  Start  End     Size    File system  Flags
 1      0.00B  58.2GB  58.2GB  ext4


Model: Linux device-mapper (linear) (dm)
Disk /dev/mapper/Xeon--vg-swap_1: 1065MB
Sector size (logical/physical): 512B/512B
Partition Table: loop

Number  Start  End     Size    File system     Flags
 1      0.00B  1065MB  1065MB  linux-swap(v1)
```

从上面的输出结果中可以看到，硬盘上只有3个物理分区，前两个是系统启动引导时用的，可以忽略，主分区就只有一个，并没有单独为swap创建一个分区。Ubuntu系统在这里采用了LVM逻辑卷管理机制（Logical Volume Manager）进行磁盘空间管理，关于LVM可进一步阅读以下文章：

> [学习 Linux，101: 硬盘布局](https://www.ibm.com/developerworks/cn/linux/l-lpic1-v3-102-1/)
> [逻辑卷管理](https://www.ibm.com/developerworks/cn/linux/l-lvm2/index.html)
> [LVM Logical Volume](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Logical_Volume_Manager_Administration/lv_overview.html)

使用`pvdisplay`命令列出所有物理卷（PVs）属性：

```no-highlight
# pvdisplay

  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               Xeon-vg
  PV Size               55.16 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              14120
  Free PE               0
  Allocated PE          14120
  PV UUID               lHbbtD-GhR7-5Ie9-OQnQ-9IYE-Y0cV-HDdZtZ
```

可以看到，只有一个物理卷`/dev/sda3`，再使用`vgdisplay`命令列出卷组（VGs）属性：

```no-highlight
# vgdisplay

  --- Volume group ---
  VG Name               Xeon-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               55.16 GiB
  PE Size               4.00 MiB
  Total PE              14120
  Alloc PE / Size       14120 / 55.16 GiB
  Free  PE / Size       0 / 0   
  VG UUID               Wbc0FF-M7HR-Rzgf-4wf2-oNjB-J2KT-1Kwp0Z
```

唯一一个卷组的名称叫做`Xeon-vg`（`Xeon`是主机名），最后用`lvdisplay`命令列出所有逻辑卷（LVs）属性：

```no-highlight
# lvdisplay

  --- Logical volume ---
  LV Path                /dev/Xeon-vg/root
  LV Name                root
  VG Name                Xeon-vg
  LV UUID                TO99bO-91qT-Zqeu-SsJ0-jFMo-30Kn-oVtGqd
  LV Write Access        read/write
  LV Creation host, time Xeon, 2017-02-27 07:01:36 +0800
  LV Status              available
  # open                 1
  LV Size                54.16 GiB
  Current LE             13866
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
   
  --- Logical volume ---
  LV Path                /dev/Xeon-vg/swap_1
  LV Name                swap_1
  VG Name                Xeon-vg
  LV UUID                Ig7xvr-scZ3-Opqe-d1Ww-Z0aT-I4B1-9h2sD4
  LV Write Access        read/write
  LV Creation host, time Xeon, 2017-02-27 07:01:36 +0800
  LV Status              available
  # open                 2
  LV Size                1016.00 MiB
  Current LE             254
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
```

此处划分了两个逻辑卷，分别是`/dev/Xeon-vg/root`和`/dev/Xeon-vg/swap_1`，然而与`parted -l`的输出结果进行对比，会发现逻辑卷的名字并不相同，在`parted -l`的输出结果中使用的是`/dev/mapper/Xeon--vg-root`和`/dev/mapper/Xeon--vg-swap_1`，查看这几个文件：

```no-highlight
# ll /dev/Xeon-vg/

lrwxrwxrwx  1 root root    7  2月 27 21:09 root -> ../dm-0
lrwxrwxrwx  1 root root    7  2月 28 15:03 swap_1 -> ../dm-1
```

```no-highlight
# ll /dev/mapper/

lrwxrwxrwx  1 root root       7  2月 27 21:09 Xeon--vg-root -> ../dm-0
lrwxrwxrwx  1 root root       7  2月 28 15:03 Xeon--vg-swap_1 -> ../dm-1
```

它们都通过符号链接指向了同一个地方：

```no-highlight
# ll /dev | grep dm-

brw-rw----  1 root disk    252,   0  2月 27 21:09 dm-0
brw-rw----  1 root disk    252,   1  2月 28 15:03 dm-1
```

事实上这里用到的是Linux内核中的Device Mapper机制，设备名称`dm`就是`Device Mapper`之意。Device Mapper机制是LVM依赖的基础技术，用于实现物理设备到逻辑设备的映射，详细说明可进一步阅读以下文章：

> [Linux 内核中的 Device Mapper 机制](https://www.ibm.com/developerworks/cn/linux/l-devmapper/index.html)
> [flashcache中应用device mapper机制](http://www.lxway.net/884499696.html)
> [DOCKER基础技术：DEVICEMAPPER](http://coolshell.cn/articles/17200.html)

另一个需要注意的问题是，尽管以上几个文件是通过符号链接联系起来的，不过它们并不等价，在使用`lvm`系列命令操作逻辑卷时，应该使用`/dev/{VG}/{LV}`这个路径，直接使用`/dev/dm-*`会出错。

通过以上操作，我们已经找到swap文件对应的逻辑卷位置了，下面就来减小其大小。

## 减小Swap分区

先用`swapoff`命令关闭交换分区：

```no-highlight
# swapoff /dev/Xeon-vg/swap_1
```

检查下是否关闭成功了：

```no-highlight
# free -h

             total       used       free     shared    buffers     cached
Mem:           15G       1.4G        14G       1.0M       176M       971M
-/+ buffers/cache:     299212   16104504
Swap:            0          0          0
```

可以看到swap分区的容量已经是0了，这时就可以通过`lvreduce`命令来收缩`Swap_1`这个逻辑卷的大小：

```no-highlight
# lvreduce -L 512M /dev/Xeon-vg/swap_1 

  WARNING: Reducing active logical volume to 512.00 MiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce swap_1? [y/n]: y
  Reducing logical volume swap_1 to 512.00 MiB
  Logical volume swap_1 successfully resized
```

`-L 512M`代表收缩至512M，更多用法可用`man`查询。之后要把新的逻辑卷重新设置为swap分区并用`swapon`命令开启之：

```no-highlight
# mkswap /dev/Xeon-vg/swap_1
 
mkswap: /dev/Xeon-vg/swap_1: warning: don't erase bootbits sectors
        on whole disk. Use -f to force.
Setting up swapspace version 1, size = 524284 KiB
no label, UUID=142b1c26-9e3f-45c4-afff-b0cce3c9448e

# swapon /dev/Xeon-vg/swap_1
```

最后检查下swap分区是否真的变小了：

```no-highlight
# free -h

             total       used       free     shared    buffers     cached
Mem:           15G       1.4G        14G       1.0M       176M       971M
-/+ buffers/cache:       294M        15G
Swap:         511M         0B       511M
```

不过此时回收的空间成为了闲置的自由空间：

```no-highlight
# vgdisplay

  --- Volume group ---
  VG Name               Xeon-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               55.16 GiB
  PE Size               4.00 MiB
  Total PE              14120
  Alloc PE / Size       13994 / 54.66 GiB
  Free  PE / Size       126 / 504.00 MiB
  VG UUID               Wbc0FF-M7HR-Rzgf-4wf2-oNjB-J2KT-1Kwp0Z
```

注意其中的`Free  PE / Size`，可以看到还有504M的未分配闲置空间，下一步就是要把这部分空间扩展到`root`逻辑卷中。


## 扩展root分区

对于Ext4文件系统，内核是支持在线扩容的，直接使用`lvextend`命令即可：

```no-highlight
# lvextend -l +100%FREE /dev/Xeon-vg/root 

  Extending logical volume root to 54.66 GiB
  Logical volume root successfully resized
```

`-l +100%FREE`表示扩展所有剩余空间，需要注意的是，`-l`与`-L`是不同的，`-l`的单位是所谓的`logical extents`，也就是上面`vgdisplay`命令显示出来的`PE Size`，这就是LVM中容量调整的一个基本单位；而`-L`的单位是传统的字节，`-L`选项不能搭配`%100`之类的形式使用。命令的详细用法还是参考`man`的说明。

逻辑卷的容量和文件系统的容量并没有直接关系，故最后还需要对文件系统进行在线扩容，对于Ext文件系统可使用`resize2fs`命令：

```no-highlight
# resize2fs /dev/Xeon-vg/root 

resize2fs 1.42.9 (4-Feb-2014)
Filesystem at /dev/Xeon-vg/root is mounted on /; on-line resizing required
old_desc_blocks = 4, new_desc_blocks = 4
The filesystem on /dev/Xeon-vg/root is now 14327808 blocks long.
```

## 调整Swap分区交换策略

Linux系统并不是在内存全部使用完之后再使用swap空间的，内核会自动判断是否该使用swap空间，可以通过很多参数控制这一行为，可参考以下文章：

> [Tuning Virtual Memory](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-tunables.html)

其中最重要的一个参数叫做`swappiness`，取值在0~100之间，表示的是内核使用swap空间的频率策略，值越大内核越倾向于使用交换空间。目前内存很大而swap空间很小，故可以尽量不去使用swap空间。要实现这一目的，将`swapiness`设置为0即可，这代表内核仅在内存快要用完时使用swap空间，具体解释可参考：

> [Swappiness](https://en.wikipedia.org/wiki/Swappiness)

设置方法也很简单，使用`sysctl`命令可以临时修改：

```no-highlight
# sysctl -w vm.swappiness=0
```

要永久修改的话，在`/etc/sysctl.conf`文件中添加`vm.swappiness=0`即可：

```no-highlight
# echo vm.swappiness=0 >> /etc/sysctl.conf
```

除了用`sysctl`命令外，还可用以下方法查看当前`swappiness`的值：

```no-highlight
# cat /proc/sys/vm/swappiness
```

----------

> 参考资料：
> [linux基础命令介绍十二：磁盘与文件系统](https://segmentfault.com/a/1190000007813965)

