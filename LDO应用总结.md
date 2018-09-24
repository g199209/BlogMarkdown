title: LDO应用总结
permalink: LDO
toc: true
mathjax: true
fancybox: true
tags: [Top, 元器件]
categories: 硬件
date: 2017-06-12 23:33:16

---

LOD(Low Dropout Regulator, 低压差线性稳压器)是目前最常用的线性稳压元件，本文对其特性进行一总结。

<!--more-->

## 基本原理

LDO的本质可以视为一个带反馈的可变电阻，其降压的基本原理就是电阻分压。其基本原理框图如下：

![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612164119.png)

其中虚线框内的调整元件(Pass Element)可以有很多实现方式，比如NPN、PNP、NMOS、PMOS、达灵顿管等。目前比较先进的LDO使用的都是MOS，这是由于MOS管可以获得比三极管更低的导通电阻，进而获得更低的压差(Drop Voltage)。

> 最常用的1117 LDO使用的调整元件就是NPN三极管：
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612165733.png)
> 
> 而TI的TPS737xx(130mV @ 1A)使用的则是NMOS：
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612165956.png)

误差比较器也是LDO的一个核心器件，它将输出反馈电压分压与`REFERENCE`基准源提供的基准电压进行比较，之后通过改变调整元件的栅极电压（或基极电流），进而改变其电阻，以实现将输出电压稳定在某一个特定值处的目的。

`REFERENCE`基准源一般都是使用带隙基准电路([Bandgap voltage reference](https://en.wikipedia.org/wiki/Bandgap_voltage_reference))来实现的，所以其电压一般为1.25V左右。

## 特性

LDO在稳态工作时的特性与一个电阻**完全相同**，也就是说，可以直接用一个电阻来替代**稳态工作时的LDO**。LDO名称中的Low Dropout低压差是与传统三端串联稳压元件（如78xx系列）相比来说的，这类串联稳压元件之所以有一个最小压差，本质原因就是**其调整元件的电阻不能降为0**，这也就是上面的原理框图中将调整元件电阻拆分为两部分——$R\_\{DS(on)\}$及一个可变电阻的原因。下面就来具体分析下此问题，需要注意的是，**下面的分析仅在调整元件为MOS时成立，对于1117这类调整元件为三极管的LDO是不适用的**。

首先来看一下LDO的工作区域：

![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612192337.png)

图中`Saturation Line`饱和线右下方的区域即为LDO可能的工作区域，这张图其实就是MOS管的输出特性曲线，正常情况下LDO是工作在MOS的恒流区的，图中A、B、C三点即是三个可能的工作点。当输出电流发生改变时，误差放大器就会通过调节$V\_\{GS\}$保证$V\_\{DS\}$不变，就像A->B点的变化过程。任意一点与原点连线斜率的倒数就是这一工作点下LDO的等效电阻，可以看到，由于MOS管的$I\_\{DS\}$会饱和，**LDO的工作区域中存在最小的导通电阻，即饱和线上对应的电阻**，而饱和线近似为一条直线，故这个最小电阻近似为一固定值，即$R\_\{DS(on)\}$。

一般数据手册中给出的最小压降是LDO在最大工作电流时对应的最小压降，即上图饱和线左上角那一点。从图中可以很容易看出：**当工作电流降低时，对应的最小压降成比例的降低，也就是说LDO的最小导通电阻基本为一常量，最小压降与工作电流存在线性关系**。

> 几个实际LDO芯片的最小压降特性：
>
> TPS737xx:
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612194425.png)
> ISL80510:
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612194600.png)
> ADM7172:
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612195024.png)

不过需要注意的是，以上特性仅在导通元件为MOS的情况下存在，对于导通元件为三极管时，以上特性不成立。比如对于BL1117来说，其最小压降为：1.3V@1A; 1.23V@0.1A。输出电流的减小并没有让最小压降显著减小。

**当LDO工作于饱和区时（即实际压差小于最小压差时），LDO特性相当于一个固定电阻，失去了稳压调节能力。**

## 核心参数指标

### 极限参数

各种极限参数，比较重要的是最高输入电压，也就是器件的最高耐压；除此之外还有器件的极限温度，需要保证晶元温度不能超过此值。

### 输出电压

LDO有固定输出和可变输出两种，固定输出反馈回路集成在器件内部，其电压是厂家出厂前调整过的，精度较高，数据手册中一般会给出典型输出电压及其最大最小范围。输出电压的精度与很多因素有关，如温度、噪声等，详细分析可以参考文末第二篇参考资料。

### 最大输出电流

需要注意的是，设计过程中不仅要考虑最大输出电流，而且要考虑散热，就算电流小于最大输出电流，然而散热不能保证，导致器件温度超过其极限温度时，LDO一样会损坏的。

### 最小压差(Dropout Voltage)

本文前面一节分析过这一问题，最小压差与输出电流有关，除此之外，最小压差还可能会与输入电压有关，如：

> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612203009.png)

需要指出的是，当器件工作于最小压差的临界状态时，数据手册中的很多指标都无法保证，比如PSRR会下降等。

### 线性调整率(Line Regulation)

其定义为，在某个固定负载电流条件下，当输入电压变化时，对应输出电压的变化量，即：

$$Line\\ Regulation=\\frac\{\\Delta V\_\{out\}\}\{\\Delta V\_\{in\}\}$$

最小输入电压是指满足最小压差情况下的最小输入电压。线性调整率是一个DC直流参数，反映了直流情况下输入电压对输出电压的影响，其值越小越好。

### 负载调整率(Load Regulation)

与线性调整率类似，其定义为，在某个固定输入电压条件下，当输出电流变化时，对应输出电压的变化量，即：

$$Load\\ Regulation=\\frac\{\\Delta V\_\{out\}\}\{\\Delta I\_\{out\}\}$$

这也是一个DC直流参数，反映了直流情况下输出电流对输入电压的影响，其值也是越小越好。

### 电源抑制比(Power Supply Rejection Ratio, PSRR)

表征LDO性能好坏的核心参数，其定义也是输入电压对输出电压的影响，与线性调整率类似，一般用分贝表示：

$$PSRR=20\\log(\\frac\{ v\_\{in\} \} \{ v\_\{out\} \} )$$

与线性调整率不同的是，**PSRR是一个频域交流参数，其表征的是，LDO对于输入电压不同频率噪声（纹波）的动态抑制能力。**

> TPS737xx的PSRR和频率的关系：
>
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612211608.png)

其值越大代表对输入纹波的抑制效果越好，故PSRR又称Ripple Rejection Ratio。**LDO在10k~1M这个范围内的PSRR尤为重要，这是因为LDO一般是接在开关电源后的，而开关电源的开关频率一般是上述范围，这也导致开关电源的输出纹波频率集中在上述范围内。**

PSRR与频率的大致关系如下：

![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612213257.png)

图中区域1的PSRR主要与带隙基准滤波器有关，不深入分析；区域2的PSRR主要取决于**误差放大器的闭环增益特性**，这个区域曲线的形状与运放的频率响应特性基本相同，极小值点就是放大器增益降低为单位增益时的频率；达到区域3后，误差放大器对于抑制PSRR帮助已经不大了，此时的PSRR主要取决于**输出滤波电容**，也就是说高频部分（100k以上）的电源纹波抑制只有靠电容来完成了，LDO的响应速度是没有这么快的，这与电源完整性中的结论异曲同工~。

除了频率外，LDO的PSRR还与很多因素有关，如输出电流、压差、负载电容等，在此不详细分析了，可以参考文末第二篇参考资料。

### 瞬态响应

包括负载电流与输入电压突变时的瞬态响应，一般数据手册中会给出实际实验测试得到的波形图。**瞬态响应与PSRR有密切关系，因为通过傅里叶变换，阶跃信号也可以分解成很多正弦信号的叠加**。

> TPS73733瞬态响应：
> 
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612220138.png)

### 接地电流(Ground Current)

即GND脚上流出的电流，如果只给出一个值的话，一般是输出电流为0时的接地电流。接地电流与输出电流关系最大，如：

> TPS737xx的接地电流与输出电流的关系：
> 
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612215751.png)

### 噪声

LDO的噪声主要来自带隙基准电路及误差放大器，手册中一般会给出其噪声电压频谱密度，如：

> TPS737xx系列：
> 
> ![](http://gmf.shengnengjin.cn/TIM%E6%88%AA%E5%9B%BE20170612220514.png)

## 稳定性

LDO内部有误差放大器，自然就存在稳定性问题，其稳定性主要取决于**外部负载电容及其ESR**，数据手册中一般会对负载电容的容值范围进行说明，也可能会给出确保系统稳定性要求的ESR。对此问题的详细分析可参考以下技术资料：

> [ESR, Stability, and the LDO Regulator](http://www.ti.com/lit/an/slva115/slva115.pdf)
> [LDO Regulator Stability Using Ceramic Output Capacitors](http://www.ti.com/lit/an/snva167a/snva167a.pdf)
> [Understanding the stable range of equivalent series resistance of an LDO regulator](http://www.ti.com/lit/an/slyt187/slyt187.pdf)
> [Stability analysis of low-dropout linear regulators with a PMOS pass element](http://lens.unifi.it/ew/dwl.php?dwl=bm90ZXMvTERPX1N0YWJpbGl0eV9hbmFseXNpcy5wZGY=&mtyp=application/pdf.)

## 新技术

PSRR是LDO的重要参数，为了提高PSRR，研究人员设计出了各种新型电路结构，比如在以下文章中，作者将输入端电压也加入控制回路中，这相当于引入了前馈，作者将其称为“Feed‐Forward Ripple Cancellation Technique”：

> [Low Drop-Out (LDO) Linear Regulators: Design Considerations and Trends for High Power Supply Rejection (PSR)](http://sites.ieee.org/scv-sscs/files/2010/02/LDO-IEEE_SSCS_Chapter.pdf)

----------

> 参考资料：
> [Understanding Low Drop Out (LDO) Regulators](http://www.ti.com/lit/ml/slup239/slup239.pdf)
> [Understand Low-Dropout Regulator (LDO) Concepts to Achieve Optimal Designs](http://www.analog.com/media/en/analog-dialogue/volume-48/number-4/articles/understand-ldo-concepts.pdf)
> [Understanding Linear Regulators and Their Key Performance Parameters](http://www.intersil.com/content/dam/Intersil/whitepapers/linear-regulator/understanding-ldos.pdf)
> [Understanding power supply ripple rejection in linear regulators](http://www.ti.com/lit/an/slyt202/slyt202.pdf)
