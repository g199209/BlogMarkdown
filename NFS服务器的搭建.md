title: NFS服务器的搭建
date: 2016-05-30 23:19:59
tags: [工具, Linux]
categories: 嵌入式

---

[NFS](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)是Network File System的缩写，用于在Linux系统间实现磁盘文件共享。严格来说NFS是一种文件系统（类似Ext2、FAT32等），而不是一种传输协议（如FTP、HTTP等），它依赖于[RPC协议](https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)进行数据传输。

下面以Ubuntu 14.04为例，介绍NFS的搭建方法。

<!--more-->

## **安装** ##
``` bash
apt-get install nfs-kernel-server
```
`apt-get`会自动安装所有依赖包。

## **配置** ##
NFS的配置文件为`/etc/exports`，编辑此文件，格式为：
``` bash
共享路径      允许IP段(参数)
# 示例如下：
/root/nfs    *(rw,sync,no_root_squash,no_subtree_check)
```
上述参数设置也是最常用的设置，全部可用参数及其含义可参考[这篇文章](http://blog.csdn.net/yusiguyuan/article/details/9494385)。另外，使用`man exports`命令可以获得关于此文件更详细的说明。

## **运行** ##

### 开启NFS
``` bash
/etc/init.d/nfs-kernel-server start
```

### 停止NFS
``` bash
/etc/init.d/nfs-kernel-server stop
```

### 重启NFS
``` bash
/etc/init.d/nfs-kernel-server restart
```

### 测试运行情况
``` bash
showmount -e
```
如果NFS已启用，此命令会显示出共享目录。

### 查看挂载点情况
``` bash
showmount -a
```
显示已与客户端连上的目录信息


### 挂载
``` bash
mount -t nfs NFS服务器IP:共享目录 本地挂载点目录
```

----------

注：
> 目前较新版本的NFS已不依赖于portmap，而使用rpcbind代替，网上相关文章中与portmap相关的部分已不再需要。

参考：
> [Linux下nfs搭建](http://lxw66.blog.51cto.com/5547576/1308679)

