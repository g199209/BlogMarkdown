title: SSH连接自动断开的解决方法
permalink: SSH_Broken_Pipe
toc: false
mathjax: false
fancybox: false
tags: [Linux]
categories: 杂七杂八
date: 2017-01-10 16:53:56

---

使用SSH连接远程服务器时，如果长时间不操作，SSH连接上就没有数据传输，此时连接会自动断开，常见的错误提示是：

```bash
Write failed: Broken pipe
```

这种超时断开机制估计是出于安全考虑设计的，不过这也会对正常使用造成一定影响，需要进行一些设置来避免这一问题。

<!--more-->

核心思路就是定时发送心跳包，这样就可以保证连接上始终有数据传输，就不会触发超时断开了。客户端`ssh`和服务器端`sshd`均支持此功能，只需要配置一下就可以了，以下方法选择其一即可。

### 服务器端配置
如果在服务器端进行配置的话，所有连接到此服务器的会话都会产生效果。

修改`/etc/ssh/sshd_config`文件，在其中添加一行：

```bash
ClientAliveInterval  30
```

这样服务器端每隔30s就会向客户端发送一个`keep-alive`包，以此保持连接不会断开。还可以指定发送`keep-alive`包的最大次数：

```bash
ClientAliveCountMax  60
```

若发送了60个`keep-alive`包后客户端依然没有响应，则断开SSH连接，如果不指定此参数的话会一直发送下去，也就是永远不断开连接。

### 客户端配置
如果没有服务器端的权限，也可在客户端进行配置，这样这个客户端所发起的所有会话都会产生效果。

修改`/etc/ssh/ssh_config`文件，与服务器端配置类似，添加以下两个参数即可：

```bash
ServerAliveInterval  
ServerAliveCountMax 
```

此时就是客户端定时向服务器端发送`keep-alive`包。

### 会话配置
如果不希望或不能修改配置文件，也可以在每次建立SSH连接时通过`-o`参数指定当前会话配置：

``` bash
ssh -o ServerAliveInterval=30 root@192.168.1.1
```

> 参考资料：
> [ssh连接linux服务器不断开－ "Write failed: Broken pipe"](http://www.cnblogs.com/livingintruth/p/3473627.html)

