title: VMware虚拟机中嵌入式Linux开发环境网络配置
date: 2016-06-25 21:27:00
tags: [Linux, Network]
categories: 工具之术

---

嵌入式Linux开发中，主机一般在VMware上运行Ubuntu等桌面Linux系统，主机与板子之间通过网线连接，以实现文件挂载、通信等目的。此时我们希望Ubuntu系统既能正常访问互联网，又能和板子进行通信。解决方法是添加两块网卡，第一块使用NAT方式，第二块使用桥接方式桥接到有线网卡上。第一块网卡(eth0)用于上网，配置为DHCP自动获取地址；第二块网卡(eth1)用于和板子连接，配置为静态IP。这样就可以同时满足这两个需求了。

<!--more-->

需要注意的是，二者的顺序最好就按照上述顺序添加，要是将eth0与eth1反过来，会导致默认情况下无法上网的。这是由于默认情况下的路由表是这样的：

```bash
gmf@gmf:/$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.175.2   0.0.0.0         UG    0      0        0 eth0
192.168.1.0     *               255.255.255.0   U     1      0        0 eth1
192.168.175.0   *               255.255.255.0   U     1      0        0 eth0
```

`default`记录使用eth0，若eth0是通过桥接方式连接板子的，此时显然无法上网，不过这可以通过修改路由表来解决。关于`route`命令和路由表的详细说明，可参考：

> [鳥哥的 Linux 私房菜 - 5.1.2 路由修改： route](http://linux.vbird.org/linux_server/0140networkcommand.php#route)
