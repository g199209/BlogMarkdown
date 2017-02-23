title: VMware中收缩Linux虚拟机体积
permalink: VMware_Shrink_Linux
toc: false
mathjax: false
fancybox: false
tags: [工具, Linux]
categories: 嵌入式
date: 2017-01-03 23:10:02

---

虚拟机使用一段时间后体积会越来越大，特别是进行了大程序编译等很占空间的行为后，虚拟磁盘文件经常会占用数十G的空间。而且就算之后删除了无用文件，虚拟磁盘文件的体积也不会自动缩小。此时就需要借助VMware Tools进行磁盘空间收缩。

<!--more-->

在正确安装了VMware Tools的前提下，`root`下执行以下命令：

``` bash
vmware-toolbox-cmd disk shrink /
```

命令中最后一个参数是虚拟磁盘的挂载点，一般就是`/`。最后若出现`disk shrinking complete`即代表压缩完成，此时在Windows资源管理器中即可看到虚拟磁盘文件的体积显著缩小，基本上就与虚拟机实际已使用空间一样大（可使用`df`确认）。

----------

若输入上述命令后提示`Shrink disk is disabled for this virtual machine.`，需要检查是否存在快照（snapshot）、是否被预分配（preallocated）、是否存在不能收缩的物理硬盘等情况。我就是因为之前[添加了一块虚拟硬盘用于访问SD卡](/2016/12/28/VMware_Internal_SD/)，直接使用上述命令收缩硬盘就会报错，把这个虚拟硬盘删掉后即可正常收缩。

> 参考资料：
> [vmware 收缩硬盘大小（compat，shrink，vmware-vdiskmanager）](http://blog.csdn.net/cymm_liu/article/details/11963687)