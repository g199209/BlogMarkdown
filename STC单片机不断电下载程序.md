title: STC单片机不断电下载程序
date: 2015-12-10 16:51:44
tags: [MCS-51]
categories: 嵌入式

---

在使用STC-ISP为STC单片机下载程序的过程中，需要手动对单片机进行复位，之后才能正确下载程序。这样不是很方便，我们可以采用一些方法来简化这一过程。

基本思路：在程序中加入一段监控代码，监测UART接收到的数据，当接收到STC-ISP程序发送的下载开始的数据后，软件复位单片机从系统ISP监控程序区启动。

<!--more-->

首先要获取STC-ISP软件发送的数据，最简单的方法是通过虚拟串口软件配合串口助手即可看到所需的数据，结果如图所示：
![](http://gmf.shengnengjin.cn/51MCU20151210163116.png)
从中可以看到，STC-ISP会持续发送`0x7F`，直至单片机给出正确的响应，故程序中只需要接收到若干个连续的`0x7F`后进行软件复位即可。

软件复位通过写IAP_CONTR寄存器实现，其定义如下：
![](http://gmf.shengnengjin.cn/51MCU20151210164210.png)
将第5、6两位置1即可实现软件复位且从系统ISP监控程序区启动的目的。

下面给出具体实现代码：
``` C
void UART_ISR_Handle() interrupt 4 using 3
{
  static char isp_cnt = 0;
  
  /* 串口中断处理 */
  if(RI)
  {
    /* 此处必须要手动清标志位 */
    RI=0;
 	
    /* ISP Reset */
    if(SBUF==0x7F)
      isp_cnt++;
    else
      isp_cnt = 0;
    if(isp_cnt > 50)
      IAP_CONTR |= 0x60;	
  }
}
```

**需要注意的是，STC-ISP软件中的最低波特率与最高波特率必须设置为相同的，且等于程序中配置好的波特率，不能使用自适应波特率。**比如下图中就将波特率固定为115200bps。
![](http://gmf.shengnengjin.cn/51MCU20151210165012.png)
