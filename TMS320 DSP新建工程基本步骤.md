title: C2000 DSP新建工程基本步骤
date: 2015-11-28 12:52:05
tags: [DSP, 工具]
categories: 嵌入式

---

本文以TMS320F28031为例，介绍如何新建一个完整的CCS工程文件。其他型号的C2000系列微处理器的操作方法大同小异。

<!--more-->

## **准备工作** ##
需要安装好[CCS](http://www.ti.com.cn/tool/cn/ccstudio)及[controlSUITE](http://www.ti.com.cn/tool/cn/controlsuite)。下文使用的CCS版本为v6.0.0，controlSUITE版本为v3.3.3。

## **新建工程** ##
打开CCS，菜单栏上找到`Project`->`New CCS Project...`。在弹出的对话框中选好DSP的型号及仿真器的型号，填入工程名称及工程位置，其余选项保持默认值即可。
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128135049.png)
点击`Finish`后在Project Explorer中可以看到新建的工程
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128135556.png)


## **添加CMD文件** ##
新建好工程后自动加入了一个CMD文件，`28031_RAM_lnk.cmd`，此CMD文件用于配置DSP从RAM启动。一般情况下，程序调试过程中在RAM中进行测试，正式发布时肯定要从Flash中启动，故需要为这两种情况分别添加CMD文件。

首先，**将工程目录中已有的那个CMD文件删掉**；之后在菜单栏上找到`Project`->`Properties`进入工程属性设置界面。默认情况下，CCS已经为我们建立了两个Configuration，分别为Debug和Release，所以可以将Debug用于调试，程序下载至RAM中；Release用于发布，程序下载至Flash中。
首先配置Debug，Configuration选择Debug，下方的Linker command file选择`****_RAM_lnk.cmd`，`****`是使用的DSP型号，如图所示：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128143648.png)
之后配置Release，Configuration选择Release，下方的Linker command file选择`F****.cmd`，`****`是使用的DSP型号，如图所示：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128143850.png)
配置完成后，选择Debug模式进行编译时`F****.cmd`文件会被排除掉；同理，选择Release模式进行编译时`****_RAM_lnk.cmd`文件会被排除掉。效果见下图：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128144431.png)

----------

上面添加的CMD文件用于分配RAM、Flash等区域的存储空间，除此之外，还需要添加一个CMD文件用于为外设寄存器结构体分配正确的空间。这个CMD文件的名字为`****_Headers_nonBIOS.cmd`或`****_Headers_BIOS.cmd`。文件开头的注释对此文件的用途做了简要的说明：
> FILE:    F2802x_Headers_nonBIOS.cmd
> TITLE:   F2802x Peripheral registers linker command file 
> DESCRIPTION: 
> This file is for use in Non-BIOS applications.
> Linker command file to place the peripheral structures used within the F2802x headerfiles into the correct memory mapped locations.
> This version of the file includes the PieVectorTable structure.
> For BIOS applications, please use the F2802x_Headers_BIOS.cmd file which does not include the PieVectorTable structure.

从中可以看到`****_Headers_nonBIOS.cmd`与`****_Headers_BIOS.cmd`的区别在于是否包含PIE中断向量表，一般的，在有RTOS的情况下选用`****_Headers_BIOS.cmd`，其余情况下均选择`****_Headers_nonBIOS.cmd`。

关于此文件的详细说明，可以参考TI的Application Report "[Programming TMS320x28xx and 28xxx Peripherals in C/C++](http://www.ti.com/lit/an/spraa85d/spraa85d.pdf)"。

此文件可以从controlSUITE中找到，位于`\controlSUITE\device_support\f****\v***\DSP****_headers\cmd`路径下，其中`v***`代表固件库的版本，一般选用最新版。以本文使用的28031为例，具体路径就是`\controlSUITE\device_support\f2803x\v130\DSP2803x_headers\cmd`。

需要注意的是，在添加文件的时候一般选择`Copy files`而不是`Link to files`，如图所示：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128145950.png)

## **添加寄存器头文件** ##
将`\controlSUITE\device_support\f****\v***\DSP****_headers\`中的`source\`与`include\`文件夹复制到工程目录下，将`include\`文件夹添加到包含路径中。**注意不要选择绝对路径**，而是使用`Workspace...`选择相对路径，如下图所示：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128152726.png)

刚刚添加的`include\`文件夹中包含的内容是对所有外设寄存器的结构体定义，这种方式TI称之为“Bit Field and Register-File Structure Approach”，与传统的“Traditional #define Approach”有所区别，使用起来会更为方便。关于这两种方式的具体说明与比较，TI在其应用报告"[Programming TMS320x28xx and 28xxx Peripherals in C/C++](http://www.ti.com/lit/an/spraa85d/spraa85d.pdf)"中有详细的论述。

**需要修改**`DSP****_Device.h`**文件以指定使用的DSP型号**，以下例子中选择了`DSP28_28031PN`.
```C
#define   TARGET   1
//---------------------------------------------------------------------------
// User To Select Target Device:


#define   DSP28_28030PAG   0
#define   DSP28_28030PN    0

#define   DSP28_28031PAG   0
#define   DSP28_28031PN    TARGET

#define   DSP28_28032PAG   0
#define   DSP28_28032PN    0

#define   DSP28_28033PAG   0
#define   DSP28_28033PN    0

#define   DSP28_28034PAG   0
#define   DSP28_28034PN    0

#define   DSP28_28035PAG   0
#define   DSP28_28035PN    0
```

`source\`文件夹下只有一个文件，就是`DSP****_GlobalVariableDefs.c`，这个文件中实例化了所有的外设结构体，并且通过通过`#pragma DATA_SECTION()`将其放置在地址映射中的正确位置，以此实现对外设寄存器的正确访问。TI文档中对此文件的说明如下：
> - Declarations for the variables that are used to access the peripheral registers.
> - Data section #pragma assignments that are used by the linker to place the variables in the proper locations in memory.

## **添加标准外设库** ##
使用标准外设库可以更方便的控制外设而不需要具体了解寄存器的配置方法，这在进行初始化的过程中尤为有用，标准外设库的使用及controlSUITE中例程的说明文档位于`\controlSUITE\device_support\f****\v***\doc\`文件夹中。需要注意的是，并不是所有型号的C2000 DSP均提供了标准外设库，目前仅有2802x，2807x等几个系列提供了标准外设库文件。

如需使用标准外设库文件，将库文件（即driverlib.lib）添加至工程中，并且包含`\controlSUITE\device_support\f****\v***\DSP****_common\include\`目录即可。

## **添加TI提供的一些例程文件** ##
将`\controlSUITE\device_support\f****\v***\DSP****_common\`中的`source\`、`include\`与`lib\`文件夹复制到工程目录下，将`include\`文件夹添加到包含路径中。
这些文件的具体作用参考`\controlSUITE\device_support\f****\v***\doc\`中的文档，可根据实际去除不需要的文件。在本项目中，最终的目录结构如图所示：
![](http://7xnwyt.com1.z0.glb.clouddn.com/DSP20151128190022.png)

## **设置程序入口点** ##
根据TI文档的说明，依次找到菜单栏的`Project`->`Properties`->`C2000 Linker`->`Symbol Management`，在`Program Entry Point -e`中输入程序入口点即可。TI文档中对此设置的说明如下：
> Defines a global symbol that specifies the primary entry point for the output module. For the DSP2803x examples, this is the symbol “code_start”. This symbol is defined in the DSP2803x_common\source\DSP2803x_CodeStartBranch.asm file. When you load the code in Code Composer Studio, the debugger will set the PC to the address of this symbol. If you do not define a entry point using the -e option, then the linker will use _c_int00 by default.
