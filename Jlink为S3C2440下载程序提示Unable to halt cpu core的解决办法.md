title: Jlink为S3C2440下载程序提示"Unable to halt cpu core"的解决办法
date: 2016-06-20 17:08:06
tags: [工具, ARM]
categories: 嵌入式

---

今天使用JLink为S3C2440下载U-Boot程序时发现连不上设备了，选择Connect后提示"Unable to halt cpu core. Failed to connect"，然而之前明明是正常的，于是开始寻找解决方案。

<!--more-->

首先使用J-Link Commander尝试连接，不出意外，打开程序后提示"Unable to halt cpu core"，然而尝试复位后就可以连接了：
``` shell
VTarget = 3.300V
Info: TotalIRLen = 4, IRPrint = 0x01
Info: Using DBGRQ to halt CPU
Info: Resetting TRST in order to halt CPU
Info: CP15.0.0: 0x41129200: ARM, Architecure 4T
Info: J-Link: ARM9, 920 core

****** Error: Unable to halt CPU core
Found 1 JTAG device, Total IRLen = 4:
 #0 Id: 0x0032409D, IRLen: 04, Unknown device
Found 1 JTAG device, Total IRLen = 4:
 #0 Id: 0x0032409D, IRLen: 04, Unknown device
Found ARM with core Id 0x0032409D (ARM9)
J-Link>Regs
CPU is not halted !
J-Link>r
Reset delay: 0 ms
Reset type NORMAL: Using RESET pin, halting CPU after Reset
Info: TotalIRLen = 4, IRPrint = 0x01
Info: CP15.0.0: 0x41129200: ARM, Architecure 4T
Info: CP15.0.1: 0x0D172172: ICache: 16kB (64*8*32), DCache: 16kB (64*8*32)
Info: Cache type: Separate, Write-back, Format A
J-Link>Regs
PC: (R15) = 00000000, CPSR = 600000D3 (SVC mode, ARM FIQ dis. IRQ dis.)
R0 = 33B1C948, R1 = 00000000, R2 = 33B1C058, R3 = 19000300
R4 = 33B1C948, R5 = 33B1C058, R6 = 19000300, R7 = 00000000
USR: R8 =33FEC9A0, R9 =33B1BF08, R10=DEADBEEF, R11 =000185BC, R12 =00000000
     R13=FF7CFAEF, R14=BFBFFD9E
FIQ: R8 =E7EF7AF0, R9 =BEBFBFFB, R10=FDA3FBFD, R11 =FCF82DF3, R12 =9FDFFD30
     R13=FDFFDF79, R14=FFEDE3F9, SPSR=00000010
SVC: R13=33B1BE80, R14=33F59374, SPSR=00000013
ABT: R13=FFDFFEFF, R14=4FFFEF77, SPSR=40000014
IRQ: R13=BF9E36F9, R14=69F7FFFF, SPSR=00000010
UND: R13=FBBAD9FF, R14=00000008, SPSR=700000DB
```

再回到J-Flash ARM中，设置菜单中有一项"Use following init sequence"，按照[之前的做法](/2015/10/31/%E4%BD%BF%E7%94%A8JLink%E4%B8%BA2440%20NOR%20Flash%E4%B8%8B%E8%BD%BD%E7%A8%8B%E5%BA%8F/)是设置成了"Reset & Delay 2ms"。注意上面J-Link Commander的输出信息，输入`r`进行复位后第一行就是:
```
Reset delay: 0 ms
```
复位延时是0ms。受此启发，将J-Flash ARM中的复位延时也改为0ms，此时再尝试连接，发现可以建立连接了。不过问题并没有解决，之后下载程序时会提示错误："PC of target system has unexpected value after programming"，这也就是之前为何要设置成"Reset & Delay 2ms"的原因（详见[使用JLink为2440 NOR Flash下载程序](/2015/10/31/%E4%BD%BF%E7%94%A8JLink%E4%B8%BA2440%20NOR%20Flash%E4%B8%8B%E8%BD%BD%E7%A8%8B%E5%BA%8F/)）。

面对这个矛盾的问题，我想到的解决方案是：**使用两次复位，第一次延时设置为0ms，这是为了能顺利建立连接；第二次复位延时设置为2ms，这是为了能顺利下载程序**。实际测试表明，这样设置后的确能正常工作了。虽然表面上看问题是解决了，不过这种解决方案不明不白的，还是要深入分析下问题的原因。

----------

之前屡次实验的结果是：
- 直接在程序运行时去连接 -> 无法连接
- 切换到NAND Flash启动（其中没有程序）后去连接 -> 正常连接
- 复位延时0ms后连接 -> 正常连接
- 复位延时2ms后连接 -> 无法连接

从中可以看到，只要是程序运行起来后均无法正常连接，复位延时2ms后连接可以视为此时程序已经运行起来了，而复位延时0ms后连接相当于程序还没运行就去连接。JATG的连接过程相当于中断（Halt）CPU的过程，这也就意味着问题是由于程序造成的，程序运行起来后因为某些原因导致CPU无法被Halt了。为验证此猜想，在启动代码中，关闭了中断和看门狗后就加入一个死循环指令：
```
b .
```
然后再编译下载运行，此时就可以去掉第一次复位了，连接过程很正常，而且在程序运行中打开J-Link Commander也可以直接连上了，说明JLink此时成功的Halt并连接上了CPU。**这就确定应该是程序中有些问题，导致CPU在某些时候进入了异常状态，从而无法使用JTAG连接。**

既然确定了是程序中的问题，那么是程序中的什么问题造成的呢？联想到之前测试时发现U-Boot的输出每次都不太一样，有些时候可以正常输出用户交互命令行：
```
In:    serial
Out:   serial
Err:   serial
Net:   CS8900-0
Error: CS8900-0 address not set.

SMDK2440 # 
```
然而更多的时候会卡在`Net: `处：
```
In:    serial
Out:   serial
Err:   serial
Net:   
```
这就暗示我们会不会是网卡设置出错了呢。LT2440开发板上使用的DM9000网卡连接在了Bank 4上，目前的程序没有对其访问模式做设置，然而根据网上的各种资料，其访问模式应该要设置为：
``` C
#define B4_BWSCON    (DW16 + WAIT + UBLB)
```
即加入等待和启用UB/LB。按这个进行设置后，编译下载运行一切正常。至此，问题已经完全搞清楚了，**因为对网卡的总线访问模式有误，导致CPU进入了异常状态，从而无法建立JTAG连接，修正此问题后一切正常。**