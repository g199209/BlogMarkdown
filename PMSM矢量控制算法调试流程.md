title: PMSM矢量控制算法调试流程
weburl: PMSM_Vector_Control
toc: true
mathjax: false
fancybox: true
tags: [DSP, Motor, Top]
categories: 科研之路
date: 2016-11-14 14:47:04

---

[矢量控制](https://zh.wikipedia.org/wiki/%E5%90%91%E9%87%8F%E6%8E%A7%E5%88%B6)又称磁场导向控制（Field Oriented Control, FOC），这是永磁同步电机（PMSM）的主要控制方法，与BLDC的简单控制策略相比，矢量控制要更为复杂，故需要一套系统的调试方法。TI提供了一个用于支持各种电机控制算法的[DMC库](http://processors.wiki.ti.com/index.php/TMS320C2000_Motor_Control_Primer)，其中包含很多矢量控制中用得到的功能模块，与之配套的还有一份调试指南：

> [Sensored Field Oriented Control of 3-Phase Permanent Magnet Synchronous Motors](http://www.ti.com/lit/an/sprabq2/sprabq2.pdf)

本文就以此为基础，结合实际调试经验，介绍一下矢量控制的基本调试流程。硬件平台基于TI C2000系列DSP，使用DMC库，不过基本方法也适用于其他各种平台。

<!--more-->


## 调试基本功能模块
矢量控制中，需要获取转子位置、相电流及转速，输出是三相逆变器的占空比信号，故首先要配置好所需的硬件功能模块，并进行测试验证，这是之后所有工作的基础。

### 配置PWM输出
矢量控制中，一般需要六路三组PWM输出，配置为两两互补导通的形式，在大部分MCU中可使用定时器模块来实现PWM输出，不过TI C2000系列提供了专门的[ePWM模块](/2015/12/24/C2000%20ePWM%E6%A8%A1%E5%9D%97/)来实现这一功能。按照寄存器定义配置好模块后，需要验证配置的正确性。

断开电机连接，依次将U、V、W三相的占空比设置为0、100%、50%，使用万用表测量对应端口的电压，0占空比时输出电压应接近0V，100%占空比时接近母线电压，50%占空比时为母线电压的一半。若测量结果符合预期，说明配置正确。

### 配置电流采样
矢量控制中需要同时获得三相电流，一般采集其中两相，根据基尔霍夫定律即可推出第三相的电流。电流采集有各种方法，可以参考：

> [三相逆变器电流采样方案总结](http://gitcafepage.gaomingfei.xyz/2016/01/01/%E4%B8%89%E7%9B%B8%E9%80%86%E5%8F%98%E5%99%A8%E7%94%B5%E6%B5%81%E9%87%87%E6%A0%B7%E6%96%B9%E6%A1%88%E6%80%BB%E7%BB%93/)

无论使用哪种方法，最后都是使用AD进行采样的，根据采样电路的结构可以推导出电流采样值与AD结果寄存器值之间的关系。所以先要配置好AD模块，一般情况下，可以使用PWM信号来触发AD采样，具体触发时机取决于使用的采样方案。配置好寄存器后，需要验证电流采样的正确性。

首先断开电机连接，使用仿真器连续读取AD采样结果寄存器的值，此时的采样值即为电流零点。观察电流零点的稳定性，一般来说，如果电流采样的稳定性较好，AD结果寄存器中只会有最后一两位在波动。若电流零点波动得较严重，说明采样稳定性很差，此时需要在程序中增加滤波算法。

之后接上电机，给U相输出一个很小的占空比，V、W两相占空比设置为0。具体占空比的值取决于母线电压及绕组电阻，可以预先估计一下，保证电流在安全范围内，一般可以取为1A左右。此时再用仿真器读取计算出的U、V、W三相电流，根据正电压产生正电流的电动机原则，**U相电流应该是正的，V、W两相电流应该是负的，且V、W两相电流应基本相同。若正负号不对，需要进行调整。**

如有条件的话，可以使用电流探头或钳型电流表等仪器测量真实的电流值，将真实值与采样值进行对比，以此评估采样的正确性及精度。如果无法测量电流的真实值，可以改变PWM占空比，保证增大占空比电流也会增大，减小占空比电流也随之减小即可。

### 配置角度及转速采样
角度和转速都来源于旋转编码器，一般MCU定时器中均有正交解码功能，用于解码来自编码器的信号。在C2000 系列中，有独立的eQEP模块用于获取旋转编码器的输出信息。在TI DMC库中，提供了一个`QEP`模块，此模块可以将`QPOSCNT`寄存器中的计数值转换为电角度及机械角度，使用标幺值表示。若不使用此模块，自行编写程序计算出电角度、机械角度及转速也可以。

转速的计算也有多种方法，可参考：

> [旋转编码器速度测量方法](/2016/10/13/Encoder_Velocity_Measure/)

这里配置好后只需要初步验证一下：用手旋转电机，角度采样值会发生变化即可，详细的验证放到后面步骤中进行。

## 编写变换程序
矢量控制的核心其实就在Clark与Park变换上，通过这两个变换实现了直轴与交轴的解耦。TI DMC库提供了现成的变换模块，即`CLARKE`、`PARK`及`IPARK`，可直接使用。如果是自行编写的程序，需要预先通过仿真等方法确定程序的正确性。

## 调试SVPWM模块
其实矢量控制也并不一定要使用SVPWM（空间矢量调制）方法，也可使用其他方法（如滞环控制等）进行电流控制，不过SVPWM是最优策略，也是主流做法。TI DMC库中使用`SVGEN`模块支持SVPWM算法，也可自行编写相关程序。SVPWM的输入是`U_alpha`及`U_beta`，输出是PWM占空比，上述步骤中已经确定了PWM输出的正确性，现在再加上SVPWM算法进行验证。系统框图如下：

![](https://img.gaomf.cn/20161114000147.png)

其中`Ds`、`Qs`、`Angle`均是调试变量，使用仿真器进行Debug时可以实时更改变量值。将`Qs`固定为0，`Ds`设置为一个较小的电压值，保证输出电流在安全范围内。之后将`Angle`由0开始，每次增加30°左右，此时电机应该是会旋转的，且每次旋转的角度应该是相同的，**记录下这个旋转方向，这就是此系统固有的正方向。**此时还可以验证电机的极对数，若`Angle`重复增加`N`个周期后电机回到起始点（可用记号笔进行标注），电机的极对数即为`N`。

最后还需要验证角度采样的正确性。这里设置的`Angle`即是这个电机真实的电角度，对比`Angle`的设定值与角度的采样值，二者的绝对值一般是不一样的，这是正常的，不过每次的变化量应该是相同的，包括大小与方向，也就是说，**两个变量间应该存在一个固定的相位差。若二者的变化趋势相反，说明编码器A、B相接反了**，可以在硬件上交换接线来解决，C2000系列中也可以通过配置`eQEP`模块来交换`A`、`B`两相。**若二者每次的变化量不一样，说明电机的极对数搞错了**，需要仔细检查程序。

## 调试电流PI控制器
SVPWM模块调试正常后就可以加上电流PI控制器了，系统框图如下：

![](https://img.gaomf.cn/20161114002838.png)

其中`IqRef`、`IdRef`及`Angle`是调试变量。从图中可以看到，这里使用了两个PI控制器，需要对其参数进行粗略的整定。

首先将`Angle`及`IqRef`设置为0，`IdRef`设置为一个安全的电流值，PI控制器参数均设置为0，此时应该是没有电流的。之后将`Id`PI控制器的`Kp`值设置为一个合适的值，具体值根据输入输出的数量级来确定，不要设置得太大，若都使用标幺值的话，可以设置为1。此时重新运行程序，观察`Id`的实际值，`Id`的值应该不等于0，不过与`IdRef`之间存在一个静差，逐步增加`Ki`，直到静差满足要求为止。将`Iq`PI控制器参数值设置为相同的即可。

之后与调试SVPWM模块时相同，逐步改变`Angle`的值，此时电机应该也是会动的。可以在程序中让`Angle`自动增加，不过增加的速度不要太快，此时电机应该会正向旋转起来。若让`Angle`自动减小，则电机会反向旋转。**正转与反转应该是对称且相同的。注意，此时设置的是`IdRef`，`IqRef`需要始终保持为0。**

记录下旋转时角度采样值与`Angle`值，并绘制曲线进行观察，**二者应该是频率相同的三角波，且有一个固定相位差**。与此同时，相电流的采样值应该是比较接近正弦的波形。如下图所示：

![](https://img.gaomf.cn/20161114130432.png)

调试PI控制器参数时也可参考相电流曲线，若曲线发生畸变，不是图中那样的正弦波形，需要降低比例及积分作用。

最后，检查采样计算出的转速值，**正转时转速应该是正的，反转时是负的**。根据`Angle`的变化周期可以计算出此时的实际转速，将实际转速与转速采样值进行对比，二者应该基本相同。如果有条件的话，可以进一步使用转速计进行验证。如果此处的转速符号或大小有误，需要检查程序进行修改。

## 调试电流闭环
上面几步调试过程中使用的`Angle`值均是由程序计算生成的，现在需要加入实际角度采样值，完成完整的电流闭环部分。

上电开始运行时，如果没有霍尔信号或绝对值编码器，是很难获取转子绝对位置的，故一般的做法是先进行预定位操作。具体方法是：将`Angle`及`IqRef`设置为0，`IdRef`设置为合适值，此时电机会自动旋转到零点位置处，此过程中将角度累加寄存器（C2000中为`QPOSCNT`）持续置零即可，这样即完成了初始的预定位操作。

之后按上一步调试电流PI控制器中的方法，自动改变`Angle`让电机旋转起来，再观察`Angle`与角度采样值，二者应该是严格相同的。确定了角度采样值的正确性后，修改程序，使用角度采样值替代`Angle`，这样就完成了完整的电流闭环程序。此时已经不再需要使用`Angle`了，初始预定位时，只要在程序中一直将角度值清零即可。

最后验证电流闭环的正确性，在完成预定位后，依次验证`Id`与`Iq`。验证`Id`时将`IqRef`设置为0，`IdRef`设置为一个合适的正值，此时电机是不会旋转的，用手转动电机也是可以转动的，只是不同于自由转动状态，此时旋转电机时会感到阻力较大，有一个力始终在维持电机处于当前位置。验证`Iq`时同理将`IdRef`设置为0，`IqRef`设置为一个合适的值即可。**需要注意的是，因为启动电流明显大于稳态电流，如果`IqRef`的值设置得过小，电机无法旋转起来，而增大`IqRef`，使电机可以旋转起来后，电机会一直加速到最高转速。**为保证安全，需要对输出电压占空比进行限幅。

**将`IqRef`设置为正值时电机应该正转，设置为负值时电机应该反转，且正反转速应该是相同的。**此时还可以进一步观察转速采样结果，如果和实际情况符合的话就可以进行下一步了。

## 调试速度闭环
最后一步是加上速度闭环，此时的系统结构框图如下：

![](https://img.gaomf.cn/FOC.png)

这就是矢量控制算法完整的控制系统结构框图。对于PMSM而言，除了在弱磁控制等情况下，`IdRef`一般是固定为0的。不过对于异步电机而言，因为要产生绕组电流，`IdRef`并不是零。`IqRef`连接至速度环PI控制器的输出上，一般会对`IqRef`的范围进行限幅，以保证电流在安全范围内。

之前已经验证了速度采样的正确性，故这里只需要整定PI控制器参数即可。方法按照通用步骤，先设置一个`Kp`，再慢慢增大`Ki`，根据设定值与实际值的曲线进一步调节参数即可。整定完成后，系统应该是完全可控的，电机会按照给定速度旋转，且正转反转应该都是没有问题的。

----------

至此，整个矢量控制的基本流程已经完全打通，接下来就可以根据具体应用场景，进一步调节参数及优化上述控制策略了。










