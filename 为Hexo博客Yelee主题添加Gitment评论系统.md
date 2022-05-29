title: 为Hexo博客Yelee主题添加Gitment评论系统
weburl: Hexo_Yelee_Gitment
toc: false
mathjax: false
fancybox: false
tags: [Blog, Serverless]
categories: 工具之术
date: 2018-11-04 21:36:31

------

本来博客使用的是多说作为评论系统，前两年多说停止服务了换成了友言，用了没多久友言又要求备案不能用了……后面由于工作繁忙也就没管这个了。前段时间发现Gitment这个基于Github Issue的评论系统不错，这两天终于有空把它给加上了。

<!--more-->

我使用的主题是基于[Yelee](https://github.com/MOxFIVE/hexo-theme-yelee)做了些修改得到的，Yelee又是基于[Yilia](https://github.com/litten/hexo-theme-yilia)的，添加Gitment的过程可以参考这篇文章：

> [Hexo 添加 Gitment 评论](https://sogrey.github.io/article/Hexo-%E6%B7%BB%E5%8A%A0-Gitment-%E8%AF%84%E8%AE%BA/)

Yiila主题也添加了Gitment支持，其[Commit](https://github.com/litten/hexo-theme-yilia/commit/af58957e14a00b3da03e4026c56d34cdf7eda9b4)也是很有参考价值的。

与以上教程有区别的是，无需安装Gitment npm插件，添加修改的代码我也改了下，有兴趣的话可以看这个[Commit](https://github.com/g199209/BlogTheme/commit/bc591586bd737f0f24a08c54f36f6e10372050c6)。

其中Gitment的CSS & JS文件改为了本地压缩后的版本，评论框的显示效果也调整了下。

终于评论系统又可以用啦~

> Update:
>
> 2021-06-12: Gitment 的 Github OAuth 是依赖于外部服务器的，目前公共的挂得差不多了，需要自己搭一个，参考下文：
>
> [gitment修复[object ProgressEvent]](https://sherry0429.github.io/2019/02/12/gitment%E4%BF%AE%E5%A4%8D/)

------------------

Update at 2022-05-21:

gitment 需要一个中间服务器的原因在于 Github OAuth Response 是不支持 CORS 的，因此若直接由浏览器发起请求会被拒绝。Github 的文档中也明确指出由前端直接发起 OAuth 请求的方式是不被支持的。

> [Why does Gitment send a request to gh-oauth.imsun.net?](https://github.com/imsun/gitment#why-does-gitment-send-a-request-to-gh-oauthimsunnet)
>
> [Authorizing OAuth Apps](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)

所以需要一个代理服务，即上文提到的 [gh-oauth-server](https://github.com/imsun/gh-oauth-server)。之前是找了台云服务器来做这个事，然而对于这种一天就没几次请求的业务来说，单独用一台服务器来部署实在是太浪费了，更好的方法是使用 Serverless 服务来做这个事。

因此又对 Gitment 评论系统做了些升级改造：

- 使用阿里云[函数计算 FC](https://www.aliyun.com/product/fc) 及 [API 网关](https://www.aliyun.com/product/apigateway) 搭建 Serverless 服务，运行 gh-oauth-server。使用 FC 提供的 Node Express 模板，都不需要修改 gh-oauth-server 的代码就可以直接运行了；API Gateway 也基本是跟着文档在界面上配置下就可以正常调用 FC 函数了。二者都有一定量的免费额度，足够个人博客用了。
- 原始的 gh-oauth-server 作为一个通用开源服务，起到的只是个单纯代理的作用，此时 Gitment 的 Client Secret 是以明文形式直接编码在前端 js 文件中的，这其实是违背 OAuth 安全建议的做法。然而若 gh-oauth-server 不是作为个公开的代理服务，而是给某个特定网站用的，那完全可以把 Client Secret 写在后端 gh-oauth-server 里面，这样就安全多了。
- 原始的 Gitment 作者是用的是 Github OAuth 应用，这存在[权限过高的问题](https://v2ex.com/t/535608)，对于访客来说可能不是那么令人放心。因此更好的做法是使用 Github 应用来代替 Github OAuth 应用，Github 应用可以只开放对某个 repo 的 issue 权限。创建 Github 应用的步骤和创建 Github OAuth 的步骤基本一致，使用方法也差不多，直接把 Gitment 配置中的 Client ID 配置为 Github 应用的 Client ID 即可。实际尝试下来此时 Gitment 完全可以正常工作。

