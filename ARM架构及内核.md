title: ARM架构及内核
permalink: ARM_Architecture_Core
toc: true
mathjax: false
fancybox: false
tags: [ARM]
categories: 嵌入式
date: 2016-10-20 15:28:56

---

一般的ARM芯片都会涉及到以下一些概念：芯片型号、ARM内核名称、指令集名称等等，这些概念间既有联系也有区别。本文简要梳理下这些不同的概念，并选择一些我接触过的芯片进行下总结。一个比较完整的ARM产品线列表可参考Wikipedia页面：

> [List of ARM microarchitectures](https://en.wikipedia.org/wiki/List_of_ARM_microarchitectures)

<!--more-->

## 基本概念

### ARM产品家族

即ARM Family，就是ARM公司不同的产品系列，这个分类主要是从市场及销售角度进行的划分。1985年，ARM公司研发出未量产的第一代产品`ARM1`，之后陆续推出了`ARM2`、`ARM3`、`ARM6`三个系列的产品，不过这些产品基本都是没人知道的。直到1997年，ARM推出`ARM7T`系列产品，这才标志着ARM大规模商用推广的开始，2000年左右推出的`ARM9T`及`ARM9E`系列更成为了ARM公司里程碑式的经典产品。之后的ARM还推出了`ARM10E`及`ARM11`这两个系列。

不过估计是觉得这样的命名太混乱了，不利于市场推广，在2005年左右，ARM将其新推出的所有CPU分为3大家族，这就是现在广为人知的`Cortex-A`、`Cortex-M`及`Cortex-R`系列，到目前为止10多年来都沿用了这一名字没有变过。其中`Cortex-A`定位于对计算性能要求较高的`Application`领域；`Cortex-M`定位于低功耗、低成本的微控制器`Microcontroller`领域；`Cortex-R`定位于对安全、实时性要求较高的`Realtime`领域。

不知将来ARM的新产品会不会换个名字呢？毕竟现在的市场局势和10年前相比有了很大的变化，而且ARM还被软银收购了……我们拭目以待。

### ARM架构名称

即ARM Architecture，就是指一套处理器体系结构及其对应的指令集等，这是从技术层面进行的划分，架构的升级就意味着产品的更新与迭代。架构与处理器家族系列间并不存在一一对应的关系，比如`ARM7T`与`ARM9T`系列使用的就是相同的`ARMv4T`架构，而`Cortex-A`系列中包含了`ARMv7-A`及`ARMv8-A`两种架构。

除了基础架构外，ARM还会用后缀表示功能增强的衍生架构，比如`E`后缀代表DSP指令集增强（`ARMv5TE`）、`J`后缀代表Jazelle加速技术（`ARMv5TEJ`）等。

目前最新的架构为`ARMv8-A`。关于ARM架构的详细信息，可参考Wikipedia页面：

> [ARM architecture](https://en.wikipedia.org/wiki/ARM_architecture)

### ARM内核名称

即ARM Core，就是实际实现出来的内核名称。ARM内核包括由ARM自己设计的公版内核及由其他架构授权合作伙伴设计的第三方内核。ARM自己设计的内核包括`ARM920T`、`Cortex-M3`、`Cortex-M4`、`Cortex-A5`、`Cortex-A57`等诸多型号；第三方内核中，最有名的就是高通的`Snapdragon`系列，包括`Scorpion`、`Krait`、`Kryo`三种型号的内核，还有苹果自己设计的`Ax`系列，目前包括`Swift`、`Cyclone`、`Typhoon`、`Twister`、`Hurricane`等型号的内核。

### 产品名称

就是最终的SOC芯片型号，一个SOC芯片只有CPU Core是不够的，还需要很多其他外设，整合到一起最终形成一个芯片。所以很多不同型号的芯片其实是共用一个CPU Core设计的，比如`STM32F4`系列，包含相当多的芯片型号，不过它们的CPU内核都是一样的——基于`ARMv7E-M`架构的`Cortex-M4`内核。

## 部分ARM芯片总结

这里仅选取我接触过的和一些近期比较有名的手机芯片进行总结，按照ARM架构和ARM内核进行分类排序。

|芯片型号或系列|厂商|AMR内核|ARM架构|
|-------------|---|-------|-------|
|S3C2410|Samsung|ARM920T|ARMv4T|
|STM32F0|ST|Cortex-M0|ARMv6-M|
|STM32F1|ST|Cortex-M3|ARMv7-M|
|STM32F4|ST|Cortex-M4|ARMv7E-M|
|K60|Freescale|Cortex-M4|ARMv7E-M|
|STM32F7|ST|Cortex-M7|ARMv7E-M|
|H3|Allwinner|Cortex-A7|ARMv7-A|
|BCM2836|Broadcom|4*Cortex-A7|ARMv7-A|
|S5PV210|Samsung|Cortex-A8|ARMv7-A|
|AM3359|TI|Cortex-A8|ARMv7-A|
|i.MX6DL|Freescale|2*Cortex-A9|ARMv7-A|
|BCM2837|Broadcom|4*Cortex-A53|ARMv8-A|
|MT6797|MTK|2\*Cortex-A72 + 8\*Cortex-A53|ARMv8-A|
|Kirin 950|Huawei|4\*Cortex-A72 + 4\*Cortex-A53|ARMv8-A|
|Exynos 8890|Samsung|4\*Mongoose + 4\*Cortex-A53|ARMv8-A|
|Snapdragon 820|Qualcomm|4\*Kryo|ARMv8-A|
|A10|Apple|4\*Hurricane|ARMv8-A|

