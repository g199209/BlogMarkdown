title: OpenSSH使用多个私钥（SSH Key）配置
date: 2016-06-17 21:40:07
tags: [Linux]
categories: 工具之术

---

在使用SSH时，有时候需要针对不同网站使用不同私钥，最简单的方法就是在`.ssh`目录（一般为`~/.ssh/`）下创建一个配置文件`config`，其内容示例如下：

```
Host github.com
IdentityFile ~/.ssh/id_rsa_github

Host git.oschina.net
IdentityFile ~/.ssh/id_rsa_oscchain
```

这样在登录github.com时使用`id_rsa_github`，而登录git.oschina.net时使用`id_rsa_oscchain`。如果登陆一个没有在`config`文件中出现的地址，则会使用默认的`id_rsa`文件。
