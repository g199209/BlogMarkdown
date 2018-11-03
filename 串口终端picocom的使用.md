title: 串口终端picocom的使用
date: 2016-06-22 20:50:23
tags: [Linux]
categories: 工具之术

---

Linux下的串口终端程序主要有这几个：minicom、kermit、picocom。其中用的最多的应该是minicom，不过picocom最为简单易用，从其名字pico-com上也可以看出，这是一个比mini-com更精简的串口终端，不过其功能足够满足大多数时候的需求。

使用时，指定波特率和串口设备文件即可打开终端：
```bash
sudo picocom -b 115200 /dev/ttyUSB0
```

<!--more-->

一般情况下，USB转串口的名字都是 `ttyUSB*`，需要注意的是，**必须使用`sudo`命令或以root的身份运行picocom**，这是由于`ttyUSB*`设备的权限一般都是660，普通用户没有读写权限。

成功打开后会显示一些基本信息，并进入正常终端模式：
```no-highlight
picocom v1.7

port is        : /dev/ttyUSB0
flowcontrol    : none
baudrate is    : 115200
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,

Terminal ready

```

如要退出，先按`Ctrl + A`进入转义模式，再按`Ctrl + Q`即可正常退出。
