title: sudo不需要输入密码的方法
toc: false
mathjax: false
fancybox: false
tags: [Linux]
date: 2018-11-17 17:20:09
permalink: Sudo_No_Passwd
categories: 工具之术

---

正常情况下，使用`sudo`命令是需要输入密码的，连续输入多条`sudo`只用输一次密码就行，不过若干分钟后又需要输入密码了。对于自己使用的本地桌面环境来说，其实是可以配置成`sudo`免输入密码的，这样可以减少一些麻烦。

<!--more-->

以`Ubuntu 18.04`为例说明设置方法，其他发行版可能会有区别。`Ubuntu Desktop`默认已经将安装系统时配置的用户加入了`admin`用户组，且`admin`用户组中的用户都是有`sudo`权限的，因此无需修改`sudo`用户组。若需要将某用户添加到`sudo`用户组中，可参考文末链接。

输入`su -`命令切换到`root`下，修改`/etc/sudoers`文件，找到：

```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL
```

修改为：

```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) NOPASSWD:ALL
```

即可。

这样就可以允许`sudo`用户组中的用户免密码执行`sudo`命令了。

--------------

> 参考资料：
>
> [免密码使用sudo和su](https://www.jianshu.com/p/5d02428f313d)