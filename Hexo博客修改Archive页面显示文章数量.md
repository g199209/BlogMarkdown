title: Hexo博客修改Archive页面显示文章数量
date: 2016-03-03 11:52:50
tags: [Web]
categories: 杂七杂八

---

之前配置的[Swiftype](https://swiftype.com/)站内搜索功能很不稳定，经常因为网络问题无法返回搜索结果，所以要找写过的某篇文章就不太方便。为解决这个问题，有一个方法是在[Archive](http://gaomf.cn/archives)页面上不分页，然后就可以用浏览器自带的搜索功能来搜索标题了。

默认情况下，Hexo无法对主页、Archive页面、标签页面每页显示文章数量进行单独设置，所以需要安装`hexo-generator-archive`插件来实现这个功能。

<!--more-->

使用如下命令安装：

```
npm install hexo-generator-archive --save

```

安装好后修改`_config.yml`中的相关配置为：

```
# Pagination
## Set per_page to 0 to disable pagination
index_generator:
  per_page: 6

archive_generator:
  per_page: 0

tag_generator:
  per_page: 0
```

参考资料：
> [Hexo程序archive页面数量设置](http://www.yuzhewo.com/2015/11/21/Hexo%E7%A8%8B%E5%BA%8Farchive%E9%A1%B5%E9%9D%A2%E6%95%B0%E9%87%8F%E8%AE%BE%E7%BD%AE/)
