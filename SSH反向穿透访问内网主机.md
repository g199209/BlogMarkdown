title: SSH反向穿透访问内网主机
permalink: SSH_Forwarding
toc: false
mathjax: false
fancybox: false
tags:
  - 工具
categories:
  - 杂七杂八
date: '2017-11-04 00:16'
---

学校的网络位于无数重NAT内网中，而且还有各种VPN，所以想要从外网访问十分困难，之前试过各种方法都没成功。今天偶然看到了SSH反向穿透的方法，因为我访问内网服务器主要也是需要SSH连接功能，故此方法可以很好的满足我的需求。此处记录下配置方法。

<!--more-->

SSH反向穿透需要有一台有公网IP的服务器作为桥梁，此处将位于多重NAT网络中需要访问的主机称为Target，而将有固定IP的中转服务器称为Server。SSH反向穿透的原理是，Target主动建立与Server间的SSH连接，利用SSH的端口转发功能，将访问Server某端口的数据包转发到Target SSH端口（22端口）上，以此实现间接登陆Target的目的。

假设Server上的转发端口为`6766`，使用如下命令在Target上建立与Server间的反向隧道：

```bash
ssh -p 22 -fN -R 6766:localhost:22 userServer@Server
```
`-R`用于定义反向隧道，`-fN`用于在建立SSH连接后SSH进入后台运行。

之后需要在Server上打开`sshd`的`GatewayPorts`功能，这样才能实现只登录一次即可连接上Target。修改`/etc/ssh/sshd_config`文件，添加下面这行：

```bash
GatewayPorts clientspecified
```

重启`sshd`服务：

```bash
service ssh restart
```

此时就可以在任意一个终端上使用`ssh -p 6766 userTarget@Server`登录到Target上了，需要注意的是，此时使用的用户名、密码、秘钥都应该是Target而不是Server的，只有IP地址或者是域名是Server的。

最后一个问题是，如何保持这个SSH反向隧道的稳定存在，并且实现若Target意外重启后能自动再次建立此反向隧道。解决方法是使用`autossh`，并且把它作为一个服务自动启动。

先安装`autossh`，之后在`/etc/init.d`下建立一个名为`autossh`的文件：（以下操作以Ubuntu为例，其他发行版可能会有区别）

```bash
#! /bin/bash
/usr/bin/autossh -p 22 -M 6777 -fN -R *:6766:localhost:22 userServer@Server -i id_rsa
```

`-M`参数指定了一个监控端口，和端口转发无关，使用一个无用的端口即可；`-i`指定了一个密钥，此处用RSA密钥的方式登陆Server。

保存此文件，并添加执行权限：`chmod +x autossh`；注册服务：`update-rc.d autossh enable`；最后启动服务：`service autossh start`。

可使用`sysv-rc-conf`工具查看`autossh`服务的开机自启动情况。这样将其作为服务配置好后，就可以实现稳定的SSH反向穿透了，终于可以实现从任何地方自由访问内网主机的目的了~~

最后顺便提一下，SSH转发其实可以承载其他更多的网络服务，这个之后有空再来研究~

----------

> 参考资料：
> [如何通过SSH反向隧道，访问NAT后面的Linux服务器?](http://network.51cto.com/art/201505/477144.htm)
> [使用SSH反向隧道进行内网穿透](http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)
> [使用autossh实现反向SSH隧道](https://marshal.ohtly.com/2017/01/26/Reverse-SSH-Tunneling-with-Autossh/)
> [ubuntu service的添加和删除](http://blog.csdn.net/yuanchao99/article/details/9111269)
> [How to use systemctl in Ubuntu 14.04](https://stackoverflow.com/questions/37438630/how-to-use-systemctl-in-ubuntu-14-04)
