title: 使用Python爬虫抓取网页
weburl: 使用Python爬虫抓取网页
date: 2016-06-03 16:03:07
tags: [Python, Spider]
categories: 编程之法

---

可以使用Python实现一个基本的爬虫，用来抓取网站上的特定内容。之前写过一个自动查询成绩的小程序，只是之后好久不用也忘了当初是怎么实现的了……最近又想研究下Python爬虫，故写点文章来记录一下。

<!--more-->

使用Python实现一个爬虫的方法有很多，相关的包有`urllib`、`urllib2`、`requests`、`bs4`、`scrapy`、`pyspider`等，此处我选择了`requests` + `bs4` + `re`(正则表达式包)的解决方案。**`requests`用于获取网站数据，`bs4`及`re`配合用于解析获取到的HTML数据。**关于学习Python爬虫的技术路线，可参考知乎上的[这个回答](https://www.zhihu.com/question/20899988/answer/96904827)。

需要安装requests及BeautifulSoup4这两个依赖包，最好使用`pip`自动安装：
```
pip install requests
pip install beautifulsoup4
```

关于这几个包的具体使用参考可我的这些文章：

[Python爬虫--使用requests获取网页](/2016/06/03/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8requests%E8%8E%B7%E5%8F%96%E7%BD%91%E9%A1%B5/)
[Python爬虫--使用BeautifulSoup解析HTML](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8BeautifulSoup%E8%A7%A3%E6%9E%90HTML/)
[Python爬虫--使用re正则表达式解析文本](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8re%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%A7%A3%E6%9E%90%E6%96%87%E6%9C%AC/)
