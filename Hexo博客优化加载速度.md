title: Hexo博客优化加载速度
date: 2015-11-01 13:28:23
tags: Blog
categories: 工具之术

---

前几天根据 [小A同学](http://caoyudong.com/) 的方法在Github Pages上使用Hexo搭建了这个博客，然而之后发现访问速度实在是太慢了，而且有些时候还会直接无法打开。当然要想个办法来解决这个问题，于是又开始了折腾……写篇文章来记录下折腾的过程。

<!--more-->

## Github & Gitcafe 同时部署
> 目前Gitcafe已被Coding.net收购，其业务也并入Coding.net中，以下方法也适用于Coding.net

[Gitcafe](https://gitcafe.com/) 可以视为中国版的Github，Github有Github Pages， Gitcafe就有Gitcafe Pages。二者的使用方法也如出一辙，具体可以参考 [这篇文章](http://blog.devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog/)。
本来可以直接把博客全部迁移到Gitcafe上，不过在网上看到，可以同时在Github和Gitcafe上进行部署，并且让国内用户访问Gitcafe，国外用户访问Github。实现过程也不复杂，可以参考 [这篇文章](http://blog.yuanbin.me/posts/2014/05/multi-deployment-with-hexo.html) 的做法。

主要就是在建立好Gitcafe Pages后，通过修改`_config.yml`文件的配置，让每次使用`hexo d`命令进行部署时，能同时部署至Github和Gitcafe上。相关代码如下：
```
deploy:
  type: git
  repository: 
    gitcafe: git@gitcafe.com:username/username.git,gitcafe-pages
    github: git@github.com:username/username.github.io.git,master
```
Github与Gitcafe使用的SSH RSA公钥必须相同。

部署完成后，需要配置DNS多线路解析，以DNSPod为例，添加如下主机记录即可：
![](https://pic.gaomf.store/web20151101141232.png)
Github与Gitcafe均使用了官方推荐的CNAME方式进行解析。

## 优化Mathjax
### 选择性加载
加载Mathjax的过程很费时间，根据 [Yilia](https://github.com/litten/hexo-theme-yilia) 主题的默认写法，即使在网页中并没有生成公式时, 也会加载最基本 MathJax.js。为解决此问题，可根据 [这篇文章](http://lukang.me/2014/mathjax-for-hexo.html) 的做法，只有在用到公式的页面才加载 Mathjax。
修改`after-footer.ejs`文件，将原来的
```
<% if (theme.mathjax){ %>
<%- partial('mathjax') %>
<% } %>
```
替换为
```
<% if (page.mathjax){ %>
<%- partial('mathjax') %>
<% } %>
```
在需要使用公式的文章开头加上`mathjax: true`即可，比如：
```
title: Markdown常用格式
date: 2015-10-30 14:34:23
tags: Web
mathjax: true
---
```

### 替换Mathjax镜像
Mathjax的CDN服务器全部位于海外，故访问速度都很慢，可以参考 [这篇文章](http://blog.josephjctang.com/2015/03/github/) 的做法，将其替换为一个中国镜像。
找到`mathjax.ejs`文件，将
```
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```
替换为
```
<script type="text/javascript" src="http://mathjax.josephjctang.com/MathJax.js?config=TeX-MML-AM_HTMLorMML">
</script>
```

## 使用百度CDN加速静态资源库
- **Fancybox**
`after-footer.ejs`中，将
```
<% if (theme.fancybox){ %>
  <%- css('fancybox/jquery.fancybox') %>
<% } %>
```
改为
```
<% if (theme.fancybox){ %>
  <%- css('http://apps.bdimg.com/libs/fancybox/2.1.5/jquery.fancybox') %>
<% } %>
```
_____________
`main.js`中，将
```
if(yiliaConfig.fancybox === true){
  require(['/fancybox/jquery.fancybox.js'], function(pc){
```
改为
```
if(yiliaConfig.fancybox === true){
  require(['http://apps.bdimg.com/libs/fancybox/2.1.5/jquery.fancybox.js'], function(pc){
```
_______________
同时，可以直接删掉`\yilia\source\fancybox\`文件夹。

- **Lazyload**
`main.js`中，将
```
if(yiliaConfig.animate === true){
  require(['/js/jquery.lazyload.js'], function(){
```
改为
```
if(yiliaConfig.animate === true){
  require(['http://apps.bdimg.com/libs/jquery-lazyload/1.9.5/jquery.lazyload.js'], 
```
_______________
同时，可以直接删掉`\yilia\source\js\jquery.lazyload.js`文件。此文件并不是官方版本的Lazyload，主题作者Litten对它进行了一些更改，不过测试表明，使用官方的Lazyload来替代此文件没有什么问题。

## 使用七牛加速静态资源文件
- **Jquery**
~~将`http://7.url.cn/edu/jslib/comb/require-2.1.6,jquery-1.9.1.min.js`下载下来，上传到七牛上，之后修改`after-footer.ejs`，将~~
```
<%- js('http://7.url.cn/edu/jslib/comb/require-2.1.6,jquery-1.9.1.min') %>
```
~~改为七牛提供的地址。~~

> 7.url.cn的访问速度和稳定性均好于七牛，故没必要画蛇添足。

- **mobile.js & pc.js **
将`\yilia\source\js\mobile.js`和将`\yilia\source\js\pc.js`上传至七牛，之后修改`\yilia\source\js\main.js`，将
```
require(['/js/mobile.js'], function(mobile){

require(['/js/pc.js'], function(pc){
```
改为七牛提供的地址。

- **main.js**
将`\yilia\source\js\main.js`上传至七牛，之后修改`after-footer.ejs`，将
```
<%- js('js/main') %>
```
改为七牛提供的地址。

- **字体文件**
将`\yilia\source\css\fonts\`目录中全部文件上传至七牛，之后修改以下这些文件：
`_variables.styl`中，将
```
font-icon-path = "fonts/fontawesome-webfont"
```
改为七牛提供的地址。
_____________
`style.styl`中，修改以下代码段
```
@font-face {
  font-family:'FontAwesome';
  src:url("fonts/fontawesome-webfont.eot");
  src:url("fonts/fontawesome-webfont.eot?#iefix") format("embedded-opentype"),url("fonts/fontawesome-webfont.woff") format("woff"),url("fonts/fontawesome-webfont.ttf") format("truetype"),url("fonts/fontawesome-webfont.svgz#FontAwesomeRegular") format("svg"),url("fonts/fontawesome-webfont.svg#FontAwesomeRegular") format("svg");
  font-weight:normal;
  font-style:normal
}
```





