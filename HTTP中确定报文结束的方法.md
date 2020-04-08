title: HTTP中确定报文结束的方法
date: 2016-04-14 10:41:10
tags: [Socket, HTTP]
categories: 编程之法

---

HTTP中，确定报文结束有几种不同方法，较为常见的是：
- **关闭TCP连接**
- **通过`Content-Length`检测**

若不关闭TCP连接，也不在HTTP头部加上`Content-Length`字段，则无法正确确定HTTP报文是否结束，对于浏览器来说，此时就会一直处于加载状态。

<!--more-->

这几天学习Python Socket编程时就遇到了这个问题，下面是一段最简单的HTTP服务器代码，无论收到什么请求都返回一个`Hello World!`

``` python
from socket import socket, AF_INET, SOCK_STREAM

HTTPResponse ='HTTP/1.1 200 OK\r\n\r\n<html>Hello World!</html>'

WebSocket = socket(AF_INET, SOCK_STREAM)
WebSocket.bind(('localhost', 80))
WebSocket.listen(1)

HTTPSocket, addr = WebSocket.accept()  # Wait Connection

while True:
  print 'Waiting HTTP Request...'
  Request = HTTPSocket.recv(1024)
  
  print 'Send HTTP Response!'
  HTTPSocket.sendall(HTTPResponse)
```

然而，这段代码是无法正常工作的，运行之后使用浏览器访问`http://localhost/`，得到的结果是这样的：

![](https://pic.gaomf.store/20160414085934.png)

从Shell的输出结果来看，HTTP响应报文已经发送成功了，此时服务器已经在等待下一次请求了，而浏览器却始终处于`Connecting`状态中。

可以通过由服务器主动关闭TCP连接来解决这一问题，修改后的代码是这样的：

``` python
from socket import socket, AF_INET, SOCK_STREAM

HTTPResponse ='HTTP/1.1 200 OK\r\n\r\n<html>Hello World!</html>'

WebSocket = socket(AF_INET, SOCK_STREAM)
WebSocket.bind(('localhost', 80))
WebSocket.listen(1)

while True:
  print 'Waiting HTTP Request...'
  HTTPSocket, addr = WebSocket.accept()  # Wait Connection
  Request = HTTPSocket.recv(1024)
  
  print 'Send HTTP Response!'
  HTTPSocket.sendall(HTTPResponse)

  HTTPSocket.close()  # Close Connection
```

修改后就可以正常运行了：
![](https://pic.gaomf.store/20160414085607.png)

如果不关闭TCP连接，也可以通过加上`Content-Length`字段来解决这一问题，代码如下：

``` python
from socket import socket, AF_INET, SOCK_STREAM

# Add Content-Length
HTTPResponse ='HTTP/1.1 200 OK\r\nContent-Length: 25\r\n\r\n<html>Hello World!</html>'

WebSocket = socket(AF_INET, SOCK_STREAM)
WebSocket.bind(('localhost', 80))
WebSocket.listen(1)

HTTPSocket, addr = WebSocket.accept()  # Wait Connection

while True:
  print 'Waiting HTTP Request...'
  Request = HTTPSocket.recv(1024)
  
  print 'Send HTTP Response!'
  HTTPSocket.sendall(HTTPResponse)
```

这样浏览器也可以正常访问服务器，运行结果和之前关闭TCP连接完全相同。

一般来说，没有必要始终维持着一个TCP连接，所以最佳的解决方案是：使用`Content-Length`字段，并且在每次响应之后关闭TCP连接，即以下代码：

``` python
from socket import socket, AF_INET, SOCK_STREAM

# Add Content-Length
HTTPResponse ='HTTP/1.1 200 OK\r\nContent-Length: 25\r\n\r\n<html>Hello World!</html>'

WebSocket = socket(AF_INET, SOCK_STREAM)
WebSocket.bind(('localhost', 80))
WebSocket.listen(1)

while True:
  print 'Waiting HTTP Request...'
  HTTPSocket, addr = WebSocket.accept()  # Wait Connection
  Request = HTTPSocket.recv(1024)
  
  print 'Send HTTP Response!'
  HTTPSocket.sendall(HTTPResponse)

  HTTPSocket.close()  # Close Connection
```
