title: STM8 SWIM接口
weburl: STM8 SWIM接口
date: 2015-12-06 13:15:23
tags: [STM8]
categories: 硬件之理

---

STM8系列单片机通过SWIM接口进行程序下载与Debug，这是一个单线接口，仅有一条数据线，即SWIM。SWIM引脚为开漏(OD)结构，以此实现双向通信。关于SWIM协议的详细说明，可参考ST的文档"[STM8 SWIM communication protocol and debug module](http://www.st.com/st-web-ui/static/active/cn/resource/technical/document/user_manual/CD00173911.pdf)"，此文档介绍了SWIM协议的通信过程与物理层时序等内容，如果需要自行设计制作SWIM下载器需要参考此文档。

当然，如果仅是为了给STM8单片机下载程序，并不需要了解SWIM协议的具体实现方法，仅需要在板子上设计好下载接口，然后和[ST-Link](http://www.st.com/web/catalog/tools/FM146/CL1984/SC724/SS1677/PF251168?sc=internet/evalboard/product/251168.jsp)仿真器连接起来即可。

<!--more-->

ST-Link/V2中SWIM接口的定义如下：
![](https://img.gaomf.cn/CircuitST-LINK-V2-Interface_SWIM_200.jpg)

其中的VDD可以不接，从STM8L-Discovery开发板上提供的ST-Link上来看，VDD直接通过一个10k的电阻接地了：
![](https://img.gaomf.cn/Circuit20151206133333.png-height2)

另外，从中可以看到，仿真器部分已经在SWIM上加上了上拉电阻，故目标板上并不需要加上拉电阻，直接和单片机SWIM口连接即可。
