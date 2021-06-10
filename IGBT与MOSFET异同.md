title: IGBT与MOSFET比较
weburl: IGBT与MOSFET比较
date: 2016-01-11 22:47:00
tags: [Motor, HW Component]
categories: 硬件之理

---

大功率IGBT与MOSFET都属于功率器件，一般都用来当做开关管，本文对这两种器件进行些简要的比较。

<!--more-->

## **电路符号** ##
IGBT（Insulated Gate Bipolar Transistor, 绝缘闸双极晶体管）的电路符号如下：
![](https://pic.gaomf.store/CircuitIGBT_SYMBOL.png)

三个极分别称为：
G：Gate，栅极
C：Collector，集电极
E：Emitter，发射极

MOSFET（Metal-Oxide-Semiconductor Field-Effect Transistor, 金属氧化物半导体场效应管）的电路符号如下：
![](https://pic.gaomf.store/CircuitMOSFET_Symbol.png)

三个极分别称为：
G：Gate，栅极
D：Drain，漏极
S：Source，源极

## **结构** ##
IGBT与MOSFET的结构对比如下：
![](https://pic.gaomf.store/Circuit0514_WTDrenesas_FO.gif)

从结构上来说，**IGBT可以视为一个MOSFET与一个三极管的组合**，即可用下面这个等效电路来表示：
![](https://pic.gaomf.store/Circuit20160111223222.png)

## **应用特性** ##
一言以蔽之：**IGBT高压高功率性能优越，而MOSFET高频性能优越**，见下图：
![](https://pic.gaomf.store/Circuit20160111223914.png)

## **控制方法** ##
二者的驱动电路基本完全相同，可以相互替换。

## **工作原理** ##
对于MOSFET来说,仅由多子承担的电荷运输没有任何存储效应,因此,很容易实现极短的开关时间。POWER MOSFET其高频特性十分优秀，所以MOSFET可用于较高频率的场合。在低电源电压下动作时之功率损失（POWER LOSS）远低于以往之组件，但是问题是,在高压的"开"状态下的源漏电阻很高(压降高),而且随着器件的电压等级迅速增长（耐压越高导通电阻越大,除了采用COOLMOS管芯的以外)。因而其传导损耗就很高,特别在高功率应用时,很受限制。

和MOSFET有所不同,IGBT器件中少子也参与了导电,IGBT是采用MOS结构的双极器件导通电阻小(发热就少)高耐压,因而可大大降低导通压降。但另一方面,存储电荷的增强与耗散引发了开关损耗、延迟时间(存储时间)、以及在关断时还会引发集电极拖尾电流。同时存在的电流尾巴和较高的IGBT集电极到发射极电压将产生关闭开关损耗，这样就限制了IGBT的上限频率。

## **总结** ##
**MOSFET一般在较低功率应用及较高频应用（即功率<1kW及开关频率≥100kHz）中表现较好，而IGBT则在较低频及较高功率设计中表现卓越。**

## **参考资料** ##
> [IGBT or MOSFET: Choose Wisely](http://www.irf.com/technical-info/whitepaper/choosewisely.pdf)
> [IGBT (Insulated Gate Bipolar Transistor)](http://www.infineon.com/dgdl/Infineon-Description_IGBT-AN-v1.0-en.pdf?fileId=db3a30433f565836013f5ca72d4e29db)
> [IGBT、MOSFET与三极管的区别](http://www.mosigbt.com/igbtzhishi/78.html)
> [IGBT与MOSFET的区别](http://www.mosigbt.com/igbtzhishi/26.html)