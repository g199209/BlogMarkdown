title: Python爬虫--使用re正则表达式解析文本
weburl: Python爬虫--使用re正则表达式解析文本
date: 2016-07-20 20:15:09
tags: [Python, Spider, Regular Expression]
categories: 编程之法

---

进行文本匹配等操作最合适的工具就是正则表达式了，Python中的正则表达式模块叫做`re`，在此总结下此模块最基本的用法。关于正则表达式本身的模式字符串的构成，参考我的另一篇文章：[正则表达式总结](/2016/07/20/正则表达式总结/)

<!--more-->

使用正则表达式首先要导入`re`模块：

``` python
import re 
```

## Match对象 ##
Match对象代表一次匹配的结果，包含了很多关于此次匹配的信息，其中最重要的就是获取匹配到的字符串，一般使用`group()`方法：

**`group()`等价于`group(0)`，返回整个匹配的子串；**
**`group(num)`返回第`num`个组对应的字符串，从1开始。**

``` python
ss = 'Hello World!'
matchObj = re.search(r'(\w+) (\w+)(.)', ss)

print(matchObj.group(0))
print(matchObj.group(1))
print(matchObj.group(3))
```

输出结果为：
```
Hello World!
Hello
!
```

## 匹配 ##
**使用`re.match()`方法**，默认从字符串起始位置开始匹配一个模式，此方法一般用于检查字符串是否符合某一规则，函数基本语法：

``` python
re.match(pattern, string)
```
匹配成功返回一个Match对象，否则返回None。

## 搜索 ##
**使用`re.search()`方法**，扫描整个字符串并返回**第一个**成功的匹配。匹配与搜索的区别在于：`re.match()`只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而`re.search()`匹配整个字符串，直到找到一个匹配。函数基本语法与`re.match()`相同：

``` python
re.search(pattern, string)
```
匹配成功返回一个Match对象，否则返回None。

## 替换 ##
**使用`re.sub()`方法**，先根据指定模式进行搜索，再用指定字符串进行替换，函数基本语法：

``` python
re.sub(pattern, repl, string)
```
`repl`是需要替换的字符串，返回替换后的字符串，如果模式没有发现，字符将被没有改变地返回。

一个示例：

```python
OldStr = 'Phone Number :##137-123-4567##'

NewStr = re.sub(r'\D', "", OldStr)
print(NewStr)
```

输出结果：
```
1371234567
```

这段程序可以去除原字符串中的所有非数字。

----------

相关文章：

[Python爬虫--使用requests获取网页](/2016/06/03/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8requests%E8%8E%B7%E5%8F%96%E7%BD%91%E9%A1%B5/)
[Python爬虫--使用BeautifulSoup解析HTML](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8BeautifulSoup%E8%A7%A3%E6%9E%90HTML/)


> 参考资料：
> [Python正则表达式](http://www.runoob.com/python/python-reg-expressions.html)