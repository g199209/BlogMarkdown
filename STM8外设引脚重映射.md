title: STM8外设引脚重映射
date: 2015-12-21 15:26:15
tags: [STM8]
categories: 嵌入式

---

在STM8中，外设对应引脚可能会有几种不同的选择，可根据实际情况来选择合适的引脚。下面总结下STM8S与STM8L系列中进行外设引脚重映射(Remap)的方法。

<!--more-->

## STM8S ##
在STM8S中，默认情况下一个外设功能只会对应一个引脚，只有少数外设引脚可以改变位置。**进行引脚重映射通过修改"Option bytes"完成**，具体修改内容可参考器件的数据手册。需要注意的是，修改"Option bytes"并不像读写其他一般寄存器那样直接，其修改方式在数据手册中有如下说明：
> Option bytes can be modified in ICP mode (via SWIM) by accessing the EEPROM address shown in the table below.
> Option bytes can also be modified "on the fly" by the application in IAP mode, except the ROP option that can only be modified in ICP mode (via SWIM).

一般情况下可直接在IDE中设置"Option bytes"，如在IAR中：
![](http://7xnwyt.com1.z0.glb.clouddn.com/STM820151221155326.png)

## STM8L ##
在STM8L中，同样可以通过修改"Option bytes"来完成一些引脚的重映射，不过除此之外，有些外设功能本身就对应到了多个引脚上，此时**可以通过SYSCFG寄存器来选择需要使用的引脚**。

以STM8L051F3为例：
![](http://7xnwyt.com1.z0.glb.clouddn.com/STM820151221160054.png)
此图中使用"[]"括起来的功能就是只能通过上文修改"Option bytes"的方法来进行重映射的功能，而其余功能的选择则可通过修改SYSCFG寄存器来完成，如Timer 2-channel 1可以选择PB0或PC5。

SYSCFG寄存器的主要作用是进行引脚功能选择，其操作较为简单，直接根据参考手册中的说明设置所需的位即可。
如果使用库函数的话，可通过调用`SYSCFG_REMAPPinConfig()`函数完成。如下面两句代码将STM8L051中TIM2_CH1与TIM2_CH2由默认的PB0、PB2重映射至PC5、PC6。
```C
SYSCFG_REMAPPinConfig(REMAP_Pin_TIM2Channel1, ENABLE);
SYSCFG_REMAPPinConfig(REMAP_Pin_TIM2Channel2, ENABLE);
```