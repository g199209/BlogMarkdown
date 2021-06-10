title: Python爬虫--使用requests获取网页
weburl: Python爬虫--使用requests获取网页
date: 2016-06-03 22:08:49
tags: [Python, Spider]
categories: 编程之法

---

`requests`包用于获取网站的内容，使用HTTP协议，基于`urllib3`实现。其官方中文文档为：[Requests: HTTP for Humans](http://cn.python-requests.org/zh_CN/latest/)

`requests`的基本使用方法很简单，这里记录一些最常用的方法，完整的介绍见其官方文档，以下介绍基于Python 3.5。

<!--more-->

 使用`requests`首先需要导入它：

``` python
import requests
```

## 发送请求
最基本的方法是`GET`请求：
``` python
url = 'http://www.zju.edu.cn/'
r = requests.get(url)
```
返回的`r`是一个`Response`类对象，包含所有返回数据，可以从这个`Response`中提取所需的信息。

除了`get()`外，常用的请求还有`post()`，用法完全相同。除此之外`requests`也支持其它各种请求方法，具体参见其文档说明。

*可通过`r.url()`获取请求的URL。*

## 读取响应内容
使用`Response`的`text`属性即可:
``` python
r = requests.get(url)
print(r.text)
```

`text`属性是针对相应内容是文本（如HTML等）的情况下使用，如果返回的数据是二进制数据（如图片等），则通过`content`属性来读取二进制比特流：
``` python
r = requests.get(url)
f = open('file', 'wb')
f.write(r.content)
```

如果返回的数据是json，`requests`中自带了一个解析器，可以直接使用`json()`函数进行解析，返回的是一个字典：
``` python
url = 'http://ip.taobao.com/service/getIpInfo.php'
payload = {'ip': '23.91.98.188'}
r = requests.get(url, params = payload)
print(r.json()['data']['country_id'])
```
上面这个例子演示了淘宝的IP查询服务。

## 附加查询参数
可以构造类似`http://www.baidu.com/s?ie=utf-8&wd=Python`这样的查询URL，其中附加在`?`之后的部分就是查询参数，可以手动构造这样一个字符串，不过`requests`中提供了更优雅的解决方案，使用一个字典作为`params`参数即可：
``` python
url = 'http://www.baidu.com/s'
payload = {'wd': 'Python', 'ie': 'utf-8'}
r = requests.get(url, params = payload)
```

## 附加表单
与附加查询参数类似，在`POST`请求中，可以附加表单信息，这一般用于实现登录或提交信息等，使用一个字典作为`data`参数即可：
``` python
payload = {'username':'username', 'password':'passwd'}
r = requests.post(url, data = payload)
```

## 读取与设置响应数据的编码
一般情况下`requests`均能从响应头部获得正确的编码，不过若头部没有相应信息，则需要手动设置，不然可能会出错。使用`Response`的`encoding`属性即可:
``` python
r = requests.get()
f.write(r.text, encoding = r.encoding)
r.encoding = 'gb2312'
r.encdoing = 'utf-8'
```

## 使用代理
使用一个字典作为`proxies`参数即可，下面这段代码演示了使用ShadowSocks作为代理的方法：
``` python
proxies = {
  'http' : 'socks5://127.0.0.1:1080',
  'https': 'socks5://127.0.0.1:1080'
}
r = requests.get(url, proxies=proxies)
```

## 获取响应状态码
使用`Response`的`status_code`属性即可:
``` python
r = requests.get(url)
print(r.status_code)
```

## 自定义请求头部
传递一个字典作为`headers`参数即可：
``` python
header = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}
r = requests.get(url, headers = header)
```

## 会话
会话可以跨请求保持某些参数，它也会在同一个Session实例发出的所有请求之间保持cookies。使用如下方法新建一个会话：
``` python
s = requests.Session()
```
会话对象`s`具有主要的Requests API的所有方法。会话一般用于连续发起一系列请求的时候使用，它会自动处理cookies的问题，十分方便。

----------

相关文章：

[Python爬虫--使用BeautifulSoup解析HTML](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8BeautifulSoup%E8%A7%A3%E6%9E%90HTML/)
[Python爬虫--使用re正则表达式解析文本](/2016/07/20/Python%E7%88%AC%E8%99%AB--%E4%BD%BF%E7%94%A8re%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%A7%A3%E6%9E%90%E6%96%87%E6%9C%AC/)


> 参考资料：
> [python requests的安装与简单运用](http://www.zhidaow.com/post/python-requests-install-and-brief-introduction)

