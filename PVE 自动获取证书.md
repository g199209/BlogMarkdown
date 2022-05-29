title: PVE 自动获取证书
weburl: pve_auto_refresh_cert
toc: false
mathjax: false
fancybox: false
tags: []
categories: 工具之术
date: 2022-05-29 20:29:00

-----------

PVE 强制使用 https 登陆，无法切换到 http 模式。其内置证书是自签发证书，每次登录的时候浏览器都会提示警告信息。要把这个警告消掉的两个必要条件是：通过域名访问且配置了这个域名对应的 SSL 证书。通过域名访问简单，买个域名配置下 DNS 即可。SSL 证书本身也很简单，就是免费证书有效期都不长，经常要去做重复的操作很麻烦。可喜的是，PVE 内置了一个叫做 ACME 的模块，通过它我们可以一键申请并部署新的 SSL 证书。

<!--more-->

操作很简单，直接截图备忘吧。

- 创建 ACM 账号及阿里云 DNS 插件，这个只用做一次就好。

![image-20220529202901076](https://img.gaomf.cn/202205292029233.png?600x)

![image-20220529203013638](https://img.gaomf.cn/202205292030741.png?600x)

- 更新证书，每次证书到期了要点一下，Let's Encrypt 的证书有效期就短短 3 个月。

![image-20220529203219866](https://img.gaomf.cn/202205292032980.png?600x)

这一过程是自动的，等脚本跑完就 OK 了。