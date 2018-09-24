title: Simulink仿真模式
date: 2015-12-23 16:25:37
tags: [Matlab]
categories: 科研

---

在Simulink中，一共有6种仿真模式可供选择，如图：
![](http://gmf.shengnengjin.cn/Matlab20151223135602.png)
- **Normal模式为一般正常的仿真模式**
- **Accelerator及Rapid Accelerator模式用于加快代码的执行速度**
- **SIL及PIL模式用于自动代码生成时进行测试仿真**
- **External模式用于连接外部系统实现基于客户端/服务器模式的实时系统仿真**

正常模式(Normal)不需要进行特殊设置，这是Simulink默认的仿真模式。下面简要介绍一下其他几种仿真模式。

<!--more-->

## **加速模式(Accelerator)** ##
Normal、Accelerator、Rapid Accelerator模式的比较如下：
![](http://gmf.shengnengjin.cn/Matlabaccel_perform12c.png)
简而言之，就是Normal模式执行速度最慢，不过支持的功能最多；Rapid Accelerator模式执行速度最快，不过支持的功能最少，对于使用的模块也有限制(需要模块有C代码以可以编译为可执行文件)；Accelerator模块介于二者之间。
这三种模式的具体功能比较可参阅帮助文档中"[Choosing a Simulation Mode](http://cn.mathworks.com/help/simulink/ug/choosing-a-simulation-mode.html)"小节，执行方式比较详见"[How Acceleration Modes Work](http://cn.mathworks.com/help/simulink/ug/how-the-acceleration-modes-work.html)"小节。

## **SIL与PIL模式**
这两种模式用于在使用Embedded Coder™、HDL Coder™或Simulink PLC Coder™从模型生成代码的过程中进行测试分析，属于Simulink Verification and Validation的一部分。
关于各种"In-the-Loop"测试的含义，可参阅"[有关基于模型的设计（MBD）一些概念和理解](http://www.matlabsky.com/thread-38774-1-1.html)"这篇文章，这里把SIL与PIL的相关部分摘录如下：

> 2）SIL，软件在环测试，软件在环测试，应该说是从模型在环测试引申过来的，区别只是把控制器的模型换成了由控制器模型生成的C代码编译成的S-function，SIL的目的是为了验证生成的代码和模型在功能上是否一致，或者说验证生成的代码和模型在功能上是否等效。
>
> 验证等效性，是否一定需要被控对象模型？不必要，既然验证生成的代码和模型的一致性，那只需要给生成代码和用于代码生成的模型相同的输入，比较它们在相同的输入条件下，输出是否一致即可。
> 
> 3）PIL，PIL有两个目的，一是为了等效性验证，二是为了测量模型生成的代码在目标处理器上的运行时间。有关运行时间的测量，如果你选择的处理器足够强大，或者你非常把握目标代码的运行不会超限，那么PIL的意义就要打折扣了。

## **外部模式(External)** ##
外部模式(External)是MATLAB Real Time Workshop(RTW)提供的一种仿真模式，可实现两个独立系统（宿主机与目标机）之间的通信。宿主机是指运行MATLAB和Simulink的计算机，而目标机是指运行RTW所生成的可执行程序的设备。在外部模式下，使用Simulink中的控制面板工具，可以通过网络连接控制模型的启动、停止、数据回传，调整模型运行参数等。

External模式主要用于实现数据的实时采集与处理，此处的外部设备可以也是本机上另一个进程，也可以是另一台计算机，还可以配置为诸如Rasberry Pi等嵌入式硬件平台。
