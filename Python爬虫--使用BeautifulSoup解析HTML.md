title: Python爬虫--使用BeautifulSoup解析HTML
weburl: Python爬虫--使用BeautifulSoup解析HTML
date: 2016-07-20 15:40:19
tags: [Python, Spider]
categories: 编程之法

---

`bs4`用于解析获取到的HTML文件，其官方中文文档为：[Beautiful Soup 中文文档](https://www.crummy.com/software/BeautifulSoup/bs3/documentation.zh.html)，此处列举的只是最基本最常用的一些用法。

<!--more-->

使用BeautifulSoup首先要导入它：

``` python
from bs4 import BeautifulSoup
```

## 创建 BeautifulSoup 对象 ##

``` python
soup = BeautifulSoup(html)
```
其中`html`是一段HTML文本字符串。

上述代码使用Python内置的解析器进行解析，也可指定使用其它解析器，如：

``` python
soup = BeautifulSoup(html, "html5lib")
```

## 输出文档 ##

``` python
print(soup.prettify)
```
输出排版好HTML文档。

## 获取标签 ##
直接使用标签对应的属性即可，如：

``` python
soup.title
soup.head
soup.a
soup.p
```

返回的是第一个tag对象，使用`name`属性获取此标签的类型：

``` python
soup.p.name
```

返回一个字符串，对应标签名称。

使用`.string`属性获取此标签包含的文本内容：

``` python
soup.title.string
```

获取到的字符串不包括标签本身，另外注意需要仅当此标签不包含子标签时才能使用`.sting`属性。如果此标签包含其他子节点，可以使用`.contents`属性获取所有子节点：

``` python
soup.head.contents
```

返回对象是一个List。如需获取父节点，使用`.parent`属性即可，除此之外，还有些用于获取兄弟节点的`.next_siblings`等方法，此处从略。

如果要获取标签的某个属性，直接使用类似List的语法即可：

``` python
soup.p['class']
```

上述程序返回`<p></p>`标签的`class`属性。

## 搜索 ##
对于爬虫来说，BeautifulSoup的搜索功能是最有用的，最常用的函数是：

**find_all(name, \*\*kwargs )**

`find_all()`方法搜索当前标签中的所有子节点，并判断是否符合过滤器的条件。返回包含所有符合条件的tag对象的List。

`name`参数可以传入很多东西，最常用的是传入目标标签名：

``` python
soup.find_all('b')
```

上述代码将放回所有的`<b></b>`标签Tag。

`kwargs`参数可以传入任意数量的属性，搜索时会匹配对应的标签属性，如：

``` python
soup.find_all('a', id='link')
```

上述代码将会搜索包含`id="link"`属性的`<a></a>`标签。

注意，如果标签属性的名称和关键词等冲突，可在其后面加一个`_`，如：

``` python
soup.find_all('div', class_='list')
```

另外，`find_all()`方法返回的是一个List，如果只需要简单的返回第一个结果，可使用`find()`函数，用法完全相同。

----------

一般使用Beautiful Soup筛选出所需的标签后，再配合正则表达式就可以准确提取出所需的内容了。

相关文章：

[Python爬虫--使用requests获取网页](/2016/06/03/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8requests%E8%8E%B7%E5%8F%96%E7%BD%91%E9%A1%B5/)
[Python爬虫--使用re正则表达式解析文本](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8re%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%A7%A3%E6%9E%90%E6%96%87%E6%9C%AC/)

> 参考资料：
> [Python爬虫利器二之Beautiful Soup的用法](http://cuiqingcai.com/1319.html)