title: 使用JLink为2440 NOR Flash下载程序
date: 2015-10-31 14:30:38
tags: [JTAG]
categories: 工具之术

---

以Mini2440开发板为例，通过Jlink将程序下载至NOR Flash中运行。因为CPU可对NOR Flash直接寻址，程序可在NOR Flash中直接运行，故将裸机程序下载至NOR Flash中调试运行较为简单。

使用J-Link Commander和J-Flash ARM均可实现程序下载，不过J-Flash ARM使用起来更为简单直观。

下面总结一下使用J-Flash ARM下载程序的方法。 **需要先安装好J-Link驱动，连接好JTAG，并且给目标板上电。2440 OM\[1:0\]配置为从NOR Flash启动。 **


<!--more-->

## 新建工程(Optional)
`File` -> `New project`

## 检查连接情况(Optional)
`Target` -> `Connect`

如果连接正常的话，LOG窗口中会有类似下面这样的信息：
```no-highlight
Connecting ...
 - Connecting via USB to J-Link device 0
 - J-Link firmware: V1.20 (J-Link ARM V8 compiled Sep  2 2011 17:54:36)
 - JTAG speed: 5 kHz (Fixed)
 - Initializing CPU core (Init sequence) ...
   - Initialized successfully
 - JTAG speed: 8000 kHz (Auto)
 - J-Link found 1 JTAG device. Core ID: 0x0032409D (ARM9)
 - Reading CFI info ...
   - CFI info read successfully
 - Connected successfully
```
其中`Core ID: 0x0032409D (ARM9)`就是检测到的CPU核心。

## 设置工程(Optional)
`Options` -> `Project settings`

实际测试表明，不进行设置全部使用默认值（大部分选项的默认值均是Auto）可以成功下载程序，不过也有可能会发生一些 [问题](http://m.blog.csdn.net/blog/zhoujiaxq/7750933) 。另外，默认情况下不使用目标RAM，下载速度会很慢。所以还是建议进行下设置。

`General`一般不需要更改。

`Target Interface`可保持默认值，也可修改下JTAG speed。

`CPU`中，`Core`可以选择`Auto`，也可按实际情况选为`ARM9`。
`Use traget RAM (faster)`建议选上，否则下载速度会很慢。RAM Address根据实际情况填写，对于S3C2440来说，可以使用内部4K的SRAM，其地址为0x40000000。
在使用了目标RAM后，有时候下载会提示错误：`PC of target system has unexpected value after programming`,若出现此错误，可参考 [这篇文章](http://www.crifan.com/resolved_j-flash_arm_nor_flash_programming_error_pc_of_target_system_has_unexpected_value_after_programming/) 的做法，将`Use following init sequence:`中Reset的Delay时间改为2ms。
配置完成后的设置如图：
![](https://gmf.shengnengjin.cn/arm20151031162234.png)

`Flash`中，可不勾选`Automatically detect flash memory`，改为手动指定Flash型号。
点击`Select flash device`后，选择正确的Falsh型号即可。Mini2440开发板上使用的NOR Flash为AM29LV160DB。
如果没有实际使用的型号，可以选择一个兼容型号，然后去掉`Check manufacturer flash Id`和`Check product flash Id`即可。
配置完成后的设置如图：
![](https://gmf.shengnengjin.cn/arm20151031163621.png)

`Production`保持默认。

全部设置完成后，工程设置如图：
![](https://gmf.shengnengjin.cn/arm20151031163909.png)

可将设置文件保存为`.jfalsh`文件，下次直接打开即可。

## 测试下载速度(Optional)
`Target` -> `Test` -> `Test speed`

可以通过测试下载速度来检查之前的配置是否正确，如果能正常下载测试数据，说明配置正确。

如果测试通过的话，会自动弹出测试结果：
![](https://gmf.shengnengjin.cn/arm20151031153629.png)

## 选择程序文件
`File` -> `Open data file`

一般选择`.bin`格式的程序文件，不支持`.axf`文件，可通过`fromelf.exe`将`.axf`文件转为`.bin`文件。

## 下载编程
`Target` -> `Program`
之后选择确定擦除和复写编程区域。

如果下载成功的话，LOG窗口中会有类似下面这样的信息：
```no-highlight
Programming target (1324 bytes, 1 range) ...
 - RAM tested O.K.
 - Erasing affected sectors ...
    - Erasing sector 0
    - Erase operation completed successfully
 - Target programmed successfully - Completed after 2.146 sec
```













