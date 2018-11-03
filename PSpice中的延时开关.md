title: PSpice中的延时开关
permalink: PSpice_Swt
toc: false
mathjax: false
fancybox: false
tags: [PSpice]
categories: 硬件之理
date: 2017-05-05 16:20:23

---

延时开关用于仿真某一时刻闭合或断开的元件特性，这不是SPICE的基本元件，PSpice中使用了两个子电路模块来实现这一功能：初始时刻闭合，某一时刻断开的`Sw_tOpen`；初始时刻断开，某一时刻闭合的`Sw_tClose`。

<!--more-->

这两个元件原理图符号位于`eval.olb`库中，SPICE实现位于`anl_misc.lib`库中，下面对其参数及实现做一总结。

### Sw_tOpen

![](http://gmf.shengnengjin.cn/sw_topen.png)

SPICE实现：

```no-highlight
* For DC and AC analyses, the switch will be Closed.
.SUBCKT  Sw_tOpen   1  2  PARAMS: 
+ tOpen=0      ; time at which switch begins to open
+ ttran=1u     ; time required to switch states (must be realistic, not 0)
+ Rclosed=0.01 ; Closed state resistance
+ Ropen=1Meg   ; Open state resistance (Ropen/Rclosed < 1E10)
V1 3 0 pulse(1 0 {tOpen} {ttran} 1 10k 11k)
S1 1 2 3 0 Smod
.model Smod Vswitch(Ron={Rclosed} Roff={Ropen})
.ends
```

可以看到，`Sw_tOpen`是由一个压控开关和一个脉冲电压源组成的，`tOpen`前电阻为`Rclosed`，之后`ttran`时间内电阻以指数形式变化至`Ropen`，并保持`10k`秒。

|参数|含义|默认值|
|----|---|-----|
|TOPEN|断开切换时间|0|
|TTRAN|过渡时间|1u|
|RCLOSED|导通电阻|0.01|
|ROPEN|开路电阻|1Meg|

### Sw_tClose

![](http://gmf.shengnengjin.cn/sw_tclose.png)

SPICE实现：

```no-highlight
* For DC and AC analyses, the switch will be Open.
.SUBCKT  Sw_tClose  1  2  PARAMS: 
+ tClose=0     ; time at which switch begins to close
+ ttran=1u     ; time required to switch states (must be realistic, not 0)
+ Rclosed=0.01 ; Closed state resistance
+ Ropen=1Meg   ; Open state resistance (Ropen/Rclosed < 1E10)
V1 3 0 pulse(0 1 {tClose} {ttran} 1 10k 11k)
S1 1 2 3 0 Smod
.model Smod Vswitch(Ron={Rclosed} Roff={Ropen})
.ends
```

与`Sw_tOpen`基本完全相同，只是开始时是断开的，`tClose`后导通。

|参数|含义|默认值|
|----|---|-----|
|TCLOSE|导通切换时间|0|
|TTRAN|过渡时间|1u|
|RCLOSED|导通电阻|0.01|
|ROPEN|开路电阻|1Meg|

----------

需要注意的是，默认的`RCLOSED`和`ROPEN`的值不一定合适，需要根据实际电路进行修改，而且根据注释说明，这两个值之间的差距还不能太大，也就是说其实这两个元件是无法很好的模拟仿真理想开关的性质的。