title: VMware Ubuntu虚拟机中为终端配置Shadowsocks代理
permalink: VMWare_Ubuntu_Terminal_Shadowsocks
toc: false
mathjax: false
fancybox: false
tags: [Linux]
categories: 工具之术
date: 2016-12-27 21:17:36

---

很多常用的终端命令操作是需要联网的，比如`git clone`等，然而由于墙的存在，很多像Github这样的网站访问可靠性极差，或者直接就无法访问，这时候就需要梯子了……只使用浏览器时很简单，`Chrome SwitchyOmega` + `Shadowsocks`的方案很完美，不过涉及到终端命令时这种方案就无能为力了，此时需要使用其它一些方法来解决。

<!--more-->

最终目标是：虚拟机中的Ubuntu系统可以使用运行于主机Windows系统中的Shadowsocks代理服务器上网，且终端命令一样可以使用代理。

解决方案是使用[ProxyChains](http://proxychains.sourceforge.net/)这个软件，安装过程很简单:

```bash
apt-get install proxychains
```

之后新建一个配置文件`~/.proxychains/proxychains.conf`：

``` 
strict_chain
proxy_dns 
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
localnet 127.0.0.0/255.0.0.0
quiet_mode

[ProxyList]
socks5  192.168.175.1 1080
```

前半部分是对ProxyChains的设置，来源于以下文章：

> [命令行工具下使用Shadowsocks](https://segmentfault.com/a/1190000002589135)

最后的`[ProxyList]`显然就是代理服务器设置，一般情况下都设置为本机`127.0.0.1`，然而这是针对Shadowsocks服务器运行于当前系统上使用的设置，此处主机Windows系统中已经运行了一个Shadowsocks服务器，我们可以直接使用这个服务器，只要将此处的IP地址设置为主机IP地址即可。

VMware虚拟机与宿主机之间的网络连接一般是通过NAT方式，在Ubuntu中使用`route`命令可以列出此时的路由表，从默认网关中可以推断出主机的IP地址，当然，也可以在VMware的网络设置中看到NAT转发中主机的IP地址，此处即为`192.168.175.1`。

最后，在Windows的Shadowsocks客户端中，需要选择“**允许来自局域网的连接**”，否则虚拟机是无法正常访问主机的Shadowsocks服务器的。

----------

安装配置完成后，即可通过`proxychains`来运行命令，如：

```bash
proxychains git clone https://github.com/g199209/Spider.git
proxychains curl http://www.google.com
```

在需要运行的命令前加上`proxychains`即可。

如果有大量命令均需要翻墙的话，这样输入会比较麻烦，此时可以使用`proxychain bash`命令新建一个bash，在此bash中输入的任何命令均会使用`proxychains`。

需要测试是否成功翻墙可使用`curl ip.gs`，这是一个测试当前IP地址的网站，通过返回的IP地址地理位置，可以很容易的判断出当前是否使用了Shadowsocks代理。




