title: Linux软路由——基础配置
permalink: Linux_SoftRouter_Basic
toc: true
mathjax: false
fancybox: false
tags: [Linux, TCP/IP, Network]
categories: 工具之术
date: 2017-04-25 21:21:21

---

之前买了台二手服务器，总觉得应该让它发挥点什么作用，正好针对学校蛋疼的网络，先试试能不能把它当做一台超级智能路由器吧~:)

<!--more-->

系统：Ubuntu Server 14.04 64bit

## 设置无线AP

使用的设备是TP-Link TL-WDN4800，配合`hostapd`实现软AP功能，实现方法参考以下几篇文章：

> [linux软AP－－hostapd+dhcpd](http://www.361way.com/hostapd-soft-ap/2933.html)
> [My Wi-Fi access point revisited](https://blog.kylemanna.com/linux/wifi-hostapd/)
> [Setting up a Wireless Access Point with Ubuntu Raring Ringtail](http://rustyrazorblade.com/2013/09/setting-up-a-wireless-access-point-with-ubuntu-raring-ringtail/)

我使用的`hostapd`的配置文件如下：

```bash
interface=wlan0
driver=nl80211
ssid=TLWDN4800
macaddr_acl=0
ignore_broadcast_ssid=0

auth_algs=1
wpa=3
wpa_passphrase=xxxxxxxxxx
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

ieee80211n=1
wme_enabled=1
wmm_enabled=1
ht_capab=[HT40-][SHORT-GI-40][TX-STBC][RX-STBC1][DSSS_CCK-40][LDPC]
hw_mode=g
channel=13
```

完整的`hostapd`配置文件说明可以参考：

> [hostapd configuration file](https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf)

配置好后使用`service hostapd restart`启动服务即可。

不过以上配置应该并不是最优配置，TL-WDN4800最高传输速率450Mbps，然而之后实际测试得到的速率只有30Mbps左右。

## 配置DHCP服务器

为无线AP添加DHCP服务。

### 安装DHCP服务

```bash
apt-get install isc-dhcp-server
```

### 配置DHCP网卡

修改`/etc/default/isc-dhcp-server`文件，根据注释说明添加对应网卡即可，可以添加不止一个，此处以`wlan0`为例：

```bash
INTERFACES="wlan0"
```

### 修改DHCP配置文件

修改`/etc/dhcp/dhcpd.conf`文件，根据注释修改即可：

```bash
ddns-update-style none;

# 此处填写分配的DNS服务器地址，一般所有客户机都是一样的，故可以使用全局配置
option domain-name-servers 223.5.5.5;

# 可适当增加地址租期
default-lease-time 6000;
max-lease-time 36000;

log-facility local7;

# 最基本的子网配置方法
subnet 192.168.137.0 netmask 255.255.255.0 {
	# 起止IP地址范围
	range 192.168.137.10 192.168.137.70;
	# 子网掩码
	option subnet-mask 255.255.255.0;
	# 默认网关
	option routers 192.168.137.77
	# 广播地址
	option broadcast-address 192.168.137.255;
}
```

以上是最基本的配置方法，各种高级配置可参考注释说明。

### 配置本机IP

DHCP服务器本身显然是不能再配置为动态获取IP的，需要配置为静态IP的形式。修改`/etc/network/interfaces`文件，如：

```bash
iface wlan0 inet static
address 192.168.137.77
netmask 255.255.255.0
gateway 192.168.137.77
dns-nameservers 223.5.5.5 223.6.6.6
```

### 重启DHCP服务

```bash
service isc-dhcp-server restart
```

此时客户端配置为自动获取IP即可。

## 通过VPN拨号连接校园网

根据学校网络中心的[说明文件](https://pic.gaomf.store/%E6%B5%99%E6%B1%9F%E5%A4%A7%E5%AD%A6L2TP%20Linux%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%85%8D%E7%BD%AE%E6%89%8B%E5%86%8C.doc)配置即可。

### 配置静态IP

修改`/etc/network/interfaces`文件：

```bash
iface eth0 inet static
address 10.12.218.231
netmask 255.255.255.0
gateway 10.12.218.1
dns-nameservers 223.5.5.5 223.6.6.6
```

### 安装`xl2tpd`

```
apt-get install xl2tpd
```

### 修改配置文件

配置`/etc/xl2tpd/xl2tpd.conf`文件，在最后加入以下代码：

```bash
[lac ZJU_VPN]
lns = 10.5.1.7
redial = yes
redial timeout = 15
max redials = 5
require pap = no
require chap = yes
name = g199209@a
ppp debug = no
pppoptfile = /etc/ppp/options.xl2tpd.zju
```

创建`/etc/ppp/options.xl2tpd.zju`文件：

```bash
noauth
proxyarp
defaultroute
```

保存用户名与密码，修改`/etc/ppp/chap-secrets`文件，在最后添加：

```bash
g199209@a * 00000000 *
```

其中`00000000`替换为真实的密码。

### 启动服务与停止服务

连接：

```bash
service xl2tpd start
echo 'c ZJU_VPN' > /var/run/xl2tpd/l2tp-control
```

连接成功后，使用`ifconfig`命令可以看到`ppp0`接口：

```bash
ppp0      Link encap:Point-to-Point Protocol
          inet addr:222.205.109.222  P-t-P:10.5.1.7  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1442  Metric:1
          RX packets:262777 errors:0 dropped:0 overruns:0 frame:0
          TX packets:348384 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3
          RX bytes:123894650 (123.8 MB)  TX bytes:332214801 (332.2 MB)
```

其中的`inet addr`就是虚拟局域网IP地址。

断开连接：

```bash
echo 'd ZJU_VPN' > /var/run/xl2tpd/l2tp-control
service xl2tpd stop
```

### 修改路由表

添加两条记录即可，首先将发往VPN服务器的流量通过本地网关发送：

```bash
route add -host 10.5.1.7 gw 10.12.218.1 dev eth0
```

默认流量都通过`ppp0`发送：

```bash
route add default gw 222.205.109.222 dev ppp0
```

此处的网关就是之前看到的`ppp0`的IP地址。

需要将矛盾的路由记录删除掉，比如只要保留一条默认记录就可以了。

## 使用手机USB共享提供4G网络连接

本来想买个4G USB网卡的，然而价格都太贵……之前一直用一个闲置的Nexus手机做无线AP的，其实可以使用USB网络共享（USB Tethering）功能，把手机当成一个4G网卡用的。将手机与电脑连起来，在手机上开启USB网络共享功能，使用`lsusb`命令可以查看手机是否正常连接上了，若成功连接的话，会有显示：

```bash
Bus 001 Device 009: ID 18d1:4ee4 Google Inc. Nexus 4 (debug + tether)
```

显示`tether`的话就说明共享成功了，使用`ifconfig -a`命令可以看到一个名为`usb0`的网络设备：

```bash
usb0      Link encap:Ethernet  HWaddr aa:e3:b2:da:e7:27
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

不过可以看到，此时还没有正确分配IP地址，使用`dhcpcd usb0`命令从手机DHCP服务器上获取一下IP地址即可，再使用`ifconfig`命令即可看到`usb0`已经获得正确的IP地址了。使用`route -n`命令可以看到此时也自动添加了路由表。

## 添加路由转发

路由器最重要的功能就是路由转发啦，Linux内核本身就支持这一功能，这在之前写的[Ubuntu虚拟机中设置NAT使树莓派开发板可以联网](http://gaomf.cn/2016/11/01/Ubuntu_NAT_Raspberry/)一文中介绍了实现方法，大概就是在`/etc/sysctl.conf`文件中开启IP转发功能，之后在`iptables`中添加一条`MASQUERADE`规则即可，文末参考资料中鸟哥的文章里对此有详细介绍。

如果要使用有线网上网：

```bash
iptables -t nat -A POSTROUTING -s 192.168.137.0/24 -o ppp0 -j MASQUERADE
```

如果要使用4G网络上网：

```bash
iptables -t nat -A POSTROUTING -s 192.168.137.0/24 -o usb0 -j MASQUERADE
```

## 自动连接脚本

以上操作可以写成一个脚本文件，方便在校园网和4G网络间切换。

### 连接至校园网

```bash
#!/bin/bash

service xl2tpd start
echo 'c ZJU_VPN' > /var/run/xl2tpd/l2tp-control
echo "Create L2TP connection..."
sleep 5
VPNIP=$(ifconfig | grep P-t-P | awk '{print $2}' | awk 'BEGIN {FS=":"} {print $2}')
if [ ! -n "$VPNIP" ]; then
        echo "VPNIP Empty!"
        exit -1
fi
echo "Update Route & Iptables"
route del -host 10.5.1.7
route add -host 10.5.1.7 gw 10.12.218.1 dev eth0
route del default
route add default gw $VPNIP dev ppp0
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 192.168.137.0/24 -o ppp0 -j MASQUERADE
echo "nameserver 223.5.5.5" > /etc/resolv.conf
echo "nameserver 223.6.6.6" >> /etc/resolv.conf

```

### 连接至4G网络

```bash
#!/bin/bash

echo 'd ZJU_VPN' > /var/run/xl2tpd/l2tp-control
service xl2tpd stop
dhcpcd usb0
echo "Update Route & Iptables"
route del default
route add default gw 192.168.42.129 dev usb0
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 192.168.137.0/24 -o usb0 -j MASQUERADE
echo "nameserver 223.5.5.5" > /etc/resolv.conf
echo "nameserver 223.6.6.6" >> /etc/resolv.conf
```

----------

使用以上方法配置好后，笔记本通过无线连接，电脑即可正常上网了，此时服务器就相当于一台路由器，只是目前4G和校园网还是独立的，要更改连接需要自己手动输入命令，之后进一步研究下如何实现更高级的策略路由。

----------

> 参考资料：
> [Ubuntu下DHCP服务器的配置](http://blog.csdn.net/zuifeng503/article/details/8203849)
> [Linux 的封包过滤软件： iptables](http://cn.linux.vbird.org/linux_server/0250simple_firewall_3.php)
> [第八章、路由观念与路由器设定](http://cn.linux.vbird.org/linux_server/0230router.php)
> [linux 添加静态路由](http://blog.csdn.net/moreorless/article/details/5397427)
> [在多种系统下通过USB连接android手机上网](http://blog.sina.com.cn/s/blog_bce68edc0101de0x.html)
> [USB Tethering on Linux](https://bbs.linuxdistrocommunity.com/discussion/779/usb-tethering-on-linux)
> [linux 路由表设置 之 route 指令详解](http://blog.csdn.net/vevenlcf/article/details/48026965)