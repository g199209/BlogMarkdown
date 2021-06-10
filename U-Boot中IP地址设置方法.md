title: U-Boot中IP地址设置方法
weburl: U-Boot中IP地址设置方法
date: 2016-05-30 23:13:35
tags: [Bootloader]
categories: 工具之术

---

U-Boot中网络IP、网关等的设置保存在环境变量中，一共有下面这几个：

|名称|含义|示例|
|---|----|----|
|`ethaddr`|MAC地址|`08:08:11:18:12:27`|
|`ipaddr`|本地IP地址|`192.168.1.7`|
|`serverip`|提供下载服务的计算机IP地址|`192.168.1.3`|
|`getewayip`|网关IP地址|`192.168.1.1`|
|`netmask`|子网掩码|`255.255.255.0`|

使用`setenv`命令进行设置，使用`printenv`命令查看目前环境变量，最后如果需要永久保存当前环境变量设置的话使用`saveenv`命令保存。

