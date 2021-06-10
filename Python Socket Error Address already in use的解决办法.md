title: Python Socket Error： Address already in use的解决办法
weburl: Python Socket Error： Address already in use的解决办法
date: 2016-04-27 15:14:43
tags: [Python, Socket, Debug]
categories: 编程之法

---

之前用Python写了个简单的TCP通信程序，放在腾讯云上24小时运行。不过有个问题，有时候使用`kill -9 pid`命令结束掉python进程后，再次运行程序就会提示`Address already in use`这个错误，然而等一段时间再去运行就可以了。

造成这个问题的原因在于此时TCP连接还没有完全关闭，而Socket默认不支持地址复用。深入的原因打算等之后仔细学习TCP/IP协议的时候再来研究，目前只是要找一个解决方案。

<!--more-->

找到的解决方案也很简单，在绑定前调用`setsockopt()`函数让Socket允许地址复用即可，即以下代码：

```
MySocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
MySocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
MySocket.bind(TCPADDR)
```

第2行代码就是调用`setsockopt()`函数，其中`SOL_SOCKET`代表对Socket层进行设置，`SO_REUSEADDR`代表是否允许在bind过程中本地地址可重复使用，最后的`1`表示允许。
