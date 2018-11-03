title: ARM Cortex-M内核复位启动过程分析
date: 2016-04-27 20:14:29
tags: [ARM, Bootrom, Top]
categories: 软件之道

---

ARM Cortex-M内核的复位启动过程也被称为复位序列(Reset sequence)，下面就来简要总结分析下这一过程。

<!--more-->

ARM Cortex-M内核的复位启动过程与其他大部分CPU不同，也与之前的ARM架构（ARM920T、ARM7TDMI等）不相同。大部分CPU复位后都是从`0x0000_0000`处取得第一条指令开始运行的，然而在ARM Cortex-M内核中并不是这样的。其复位序列为：

1. **从地址`0x0000_0000`处取出`MSP`的初始值；**
2. **从地址`0x0000_0004`处取出`PC`的初始值，然后从这个值对应的地址处取指。**

即下图所示过程：
![](http://gmf.shengnengjin.cn/20160427172324.png)

事实上，地址`0x0000_0004`开始存放的就是默认中断向量表（有些资料中将地址`0x0000_0000`处的MSP指针初始值也算作中断向量表的一部分，这个说法似乎不太妥当），ARM Cortex-M内核的中断向量表布局情况如下图所示：

![](http://gmf.shengnengjin.cn/20160427200250.png)

*注意：中断向量表的位置可以改变，此处是默认情况下的设置。*

**值得注意的是，在ARM Cortex-M内核中，发生异常后，并不是去执行中断向量表中对应位置处的代码，而是将对应位置处的数据存入`PC`中，然后去此地址处进行取指。简而言之，在ARM Cortex-M的中断向量表中不应该放置跳转指令，而是该放置ISR程序的入口地址。**

有了上面的分析就很好理解复位序列了，复位其实就相当于发生了一次`Reset`异常，而从图中可以看到，地址`0x0000_0004`处存放的正是`Reset`异常对应的中断处理函数入口地址。

另外还有两个细节问题需要注意：
1. `0x0000_0000`处存放的`MSP`初始值最低三位需要是0；
2. **`0x0000_0004`处存放的地址最低位必须是1。**

第一个问题是因为ARM AAPCS中对栈使用的约定是这样的：
> 5.2.1.1 
Universal stack constraints 
At all times the following basic constraints must hold: 
Stack-limit < SP <= stack-base. The stack pointer must lie within the extent of the stack. 
SP mod 4 = 0. The stack must at all times be aligned to a word boundary. 
> 5.2.1.2 
Stack constraints at a public interface 
The stack must also conform to the following constraint at a public interface: 
SP mod 8 = 0. The stack must be double-word aligned.

简而言之，规约规定，栈任何时候都必须4字节对齐，在调用入口需8字节对齐，而且`SP`的最低两位在硬件上就被置为0了。

第二个问题与`ARM`模式与`Thumb`模式有关。ARM中`PC`中的地址必须是32位对齐的，其最低两位也被硬件上置0了，故写入`PC`中的数据最低两位并不代表真实的取址地址。ARM中使用最低一位来判断这条指令是`ARM`指令还是`Thumb`指令，**若最低位为0，代表`ARM`指令；若最低位为1，代表`Thumb`指令**。在Cortex-M内核中，并不支持`ARM`模式，若强行切换到`ARM`模式会引发一个Hard Fault。

----------

最后写一段小程序来验证下以上分析。这段程序基于STM32F4系列单片机，作用是让`PA0`管脚输出高电平。这应该也是实现这一目的最精简的写法了。

```
rAHB1ENR        EQU     0x40023830
AHB1ENRValue    EQU     0x00000001
    
rMODER          EQU     0x40020000
MODERValue      EQU     0xA8000001
    
rODR            EQU     0x40020014
ODRVaule        EQU     0x00000001

    AREA RESET, DATA, READONLY
    DCD 0x00000400
    DCD Start

    AREA |.text|, CODE, READONLY
    ENTRY

Start
    LDR R0, =rAHB1ENR
    LDR R1, =AHB1ENRValue
    STR R1, [R0]
    
    LDR R0, =rMODER
    LDR R1, =MODERValue
    STR R1, [R0]
    
    LDR R0, =rODR
    LDR R1, =ODRVaule
    STR R1, [R0]

    B .  
    END
```

第11行使用`DCD`伪指令分配了4个字节的存储空间，并将其值设置为`0x0000_0400`；第12行同理，将`Start`标号处的地址放置在偏移量为4字节的位置处；第17行`Start`标号之后的部分就是程序主体，依次完成了GPIOA端口RCC时钟使能、PA0设置为输出模式、PA0置高这三个步骤。

程序在链接时会将`RESET`段放置在目标文件开头，故相当于在地址`0x0000_0000`处的数据为`0x0000_0400`，在地址`0x0000_0004`处的数据为`Start`部分的入口地址。

不过需要指出的是，实际上在STM32F4芯片中，内部Flash的地址是从`0x0800_0000`处开始的，在BOOT管脚设置为Flash启动的时候，芯片内部会自动将`0x0000_0000`~`0x000F_FFFF`区域映射至`0x0800_0000`~`0x080F_FFFF`处，此时可以视为二者是等价的。

使用Debug模式进行调试，复位后CPU寄存器的值如下所示：
![](http://gmf.shengnengjin.cn/20160427215016.png)

Flash中的数据如图：
![](http://gmf.shengnengjin.cn/20160427215242.png)

可以看到，编译器很智能的将`0x0800_0004`处的数据设置为了`0x0800_0009`，而不是`Start`标号真实的地址值，这说明了这是一条`Thumb-2`指令。复位后`PC`中的值是`0x0800_0008`，`SP`中的值是`0x0000_0400`，与预期结果完全相同。

*最后顺便提一下，上面那段简单的程序有个问题，实际上`Start`部分的程序是占用了中断向量表的空间，这在没有异常发生的时候是没有问题的，不过一旦有异常发生，显然程序执行是会出错的。*
