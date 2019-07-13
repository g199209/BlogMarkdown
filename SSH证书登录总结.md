title: SSH证书登录总结
permalink: SSH_Authentication
toc: true
mathjax: false
fancybox: false
tags: [Linux]
categories: 工具之术
date: 2017-03-01 22:34:37

---

远程登录Linux服务器一般都是通过SSH方式，除此之外，像Git同步等其他一些操作也可通过SSH进行，默认情况下SSH是通过用户名和密码来进行用户身份验证的，这样密码太复杂自己输入不方便，太短了不安全，故更好的方式是通过证书进行身份验证和登录。

<!--more-->

## 证书生成

使用`ssh-keygen`工具：

```no-highlight
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/home/gmf/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/gmf/.ssh/id_rsa.
Your public key has been saved in /home/gmf/.ssh/id_rsa.pub.
```

直接使用`ssh-keygen`而不附加任何参数相当于`ssh-keygen -t rsa -b 2048`，即RSA类型2048字节的公钥/私钥对。`ssh-keygen`还可以附加很多参数，可以参考文末参考资料。

`Enter passphrase`处可以输入一个秘钥文件的密码，如果设置了密码的话之后每次访问都要输入密码。

## 服务器配置

### 修改配置文件

一般是`/etc/ssh/sshd_config`文件，只需要设置以下几项启用秘钥登录就可以了：

```no-highlight
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

> Update 2019-07-13:
>
> 在CentOS 7.4及以上版本中，没有`RSAAuthentication`这一项配置了，也不需要去配置这个，可参考：
>
> [CentOS7.4配置SSH登录密码与密钥身份验证踩坑](https://www.cnblogs.com/Leroscox/p/9627809.html)

在**保证可以用证书正常登录后**，可以禁用密码登录以提高安全性：

```no-highlight
PasswordAuthentication no
```

以上几项设置默认情况下都是注释，把开头的注释符号去掉即可。其余还有一些和安全性有关的设置，此处就省略了，可以根据文件中的注释说明进行修改，也可参考文末资料。

### 添加用户公钥

受信任的用户公钥记录在`~/.ssh/authorized_keys`文件中，服务器上每一个用户都有自己的`authorized_keys`文件，在使用SSH登录时，登录用户是哪一个就会去这个用户的用户目录中查找`~/.ssh/authorized_keys`文件，比如使用`ssh root@192.168.1.1`登录时，SSH服务器端就会使用`/root/.ssh/authorized_keys`文件中的公钥。

所以我们要将之前生成的公钥添加到需要登录的用户目录下，如允许使用多个用户身份登录，需要分别添加公钥。

添加方法有好几种，可以手动用vim打开此文件添加（如果不存在新建一个即可），也可先把之前生成的`id_rsa.pub`文件传到服务器上，使用以下命令添加：

```no-highlight
cat id_rsa.pub >> ~/.ssh/authorized_keys
```

此外，似乎还可以直接使用`ssh-copy-id`工具上传：

```no-highlight
ssh-copy-id user@host
```

需要注意的是，`~/.ssh`目录及`~/.ssh/authorized_keys`文件**必须是**700及600权限的，如果不是的话要先修改一下，否则之后ssh客户端会出于安全因素拒绝使用此证书文件。

### 重启sshd服务

```no-highlight
sudo /etc/init.d/ssh restart
```

这时就可以在客户机上使用`ssh user@host`的方式登录了，默认会使用位于**当前用户目录下的**`~/.ssh/id_rsa`文件作为私钥。不过还可以在客户端上进行下配置简化这一命令。

## 客户端配置

修改`~/.ssh/config`文件（若不存在则新建），按以下格式添加Host配置：

```
Host myserver
User <user_name>
Hostname <server_name>
Port <port>
IdentityFile <private_key_path>
```

`Port`可以省略，此时使用默认的22端口。

配置好后输入`ssh myserver`即等价于`ssh -i <private_key_path> <user_name>@<server_name> -p <port>`。这样用一条简单的命令就可以实现自动调用指定的私钥文件以特定的用户名和端口登录远程服务器的目的了，不同的服务器可以使用不同配置，十分方便灵活。

----------

> 参考资料：
> [SSH 原理和基本使用：ssh 安全配置 以及ssh key 认证登录](http://skypegnu1.blog.51cto.com/8991766/1641064)
> [配置 SSH 服务以使用证书登录 Linux 服务器](https://cnzhx.net/blog/linux-server-ssh-key-auth-configuration/)
