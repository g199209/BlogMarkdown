title: OSI模型总结
date: 2015-11-23 22:52:05
tags: [Network, Top]
categories: 科研

---

OSI模型是最为常用的网络分层模型，其全称为“[开放式系统互联通信参考模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B)”，Open System Interconnection Reference Model。 此模型由国际标准化组织(ISO)提出，是一个试图使各种计算机在世界范围内互连为网络的标准框架，对应标准为ISO/IEC 7498-1。
此模型已成为计算机间和网络间进行通信的主要结构模型，目前使用的大多数网络通信协议的结构都是基于OSI模型的。 

OSI模型中将网络通信的工作分为七层，这就是通常说的OSI七层模型，具体层次划分见下图及下表：

<!--more-->

![](http://7xnwyt.com1.z0.glb.clouddn.com/Web20160304200750.jpg)

| 层                       | 数据单元                   | 典型设备        | 功能 |
| ------------------------ | ------------------------- | -------------- | --- |
|应用层 |数据                  |计算机：应用程序  |直接和应用程序连接并提供常见的网络应用服务|
|表示层|数据                  |计算机：编码方式  |将数据按照网络能理解的方案进行格式化|
|会话层    |数据                  |计算机：建立会话  |负责在网络中的两节点之间建立、维持和终止通信|
|传输层  |数据段            |计算机：进程和端口|提供端到端的交换数据的机制，检查分组编号与次序|
|网络层    |数据包 |网络：路由器     |将网络地址转化为对应的物理地址，并且决定如何将数据从发送方传到接收方|
|数据链路层|数据帧         |网络：交换机、网桥|控制物理层和网络层之间的通讯|
|物理层    |比特                   |网络：集线器、网线|产生并检测电压，以便发送和接收携带数据的信号，提供为建立、维护和拆除物理链路所需要的机械的、电气的、功能的和规程的特性|

OSI七层模型的每一层都具有清晰的特征。基本来说，第七至第四层处理数据源和数据目的地之间的端到端通信，而第三至第一层处理网络设备间的通信。另外，OSI模型的七层也可以划分为两组：主机层（7、6、5、4层）和媒介层（3、2、1层）。主机层处理应用程序问题，并且通常只应用在软件上，最高层，即应用层是与终端用户最接近的；媒介层是处理数据传输的，物理层和数据链路层应用在硬件和软件上，最底层，即物理层是与物理网络媒介最接近的，并且负责在媒介上发送数据。 

下面从最底层开始，具体说明各层的作用：

## **物理层(Physical Layer)**
物理层位于OSI参考模型的最底层，它直接面向原始比特流的传输。为了实现原始比特流的物理传输，物理层必须解决好包括传输介质、信道类型、数据与信号之间的转换、信号传输中的衰减和噪声等在内的一系列问题。另外，物理层标准要给出关于物理接口的机械、电器功能和规程特性，以便于不同的制造厂家既能够根据公认的标准各自独立地制造设备，又能使各个厂家的产品互相兼容。
简而言之，物理层规定了通信设备的机械的、电气的、功能的和过程的特性，用以建立、维护和拆除物理链路连接。
具体地讲，机械特性规定了网络连接时所需接插件的规格尺寸、引脚数量和排列情况等；电气特性规定了在物理连接上传输比特流时线路上信号电平的大小、阻抗匹配、传输速率、距离限制等；功能特性是指对各个信号先分配确切的信号含义，即定义了DTE和DCE之间各个线路的功能；规程特性定义了利用信号线进行bit流传输的一组 操作规程，是指在物理连接的建立、维护、交换信息是，DTE和DCE双放在各电路上的动作系列。
在这一层，数据的单位称为比特（bit）。
**属于物理层定义的典型规范包括：[EIA-RS-232](https://zh.wikipedia.org/wiki/RS-232)、[EIA-RS-422](https://zh.wikipedia.org/wiki/EIA-422)、[V.35](http://baike.baidu.com/view/14873387.htm)、[RJ-45](https://zh.wikipedia.org/wiki/8P8C)等。**

## **数据链路层(Data Link Layer)** 
数据链路层涉及相邻节点时间的可靠数据传输，数据链路层通过加强物理层传输原始比特的功能，使之对网络层表现为一条无错线路。为了能够实现相邻节点之间无差错的数据传输，数据链路层在数据传输过程中提供了确认、差错控制和流量控制等机制。在数据链路层的设备有二层交换机和网桥，我们可以把数据链路层看做是承上启下的一层。
数据链路层在不可靠的物理介质上提供了可靠的传输。在这一层，数据的单位称为帧（frame）。
**数据链路层典型协议包括：[IEEE 802.11](https://zh.wikipedia.org/wiki/IEEE_802.11)、[PPP](https://zh.wikipedia.org/wiki/%E7%82%B9%E5%AF%B9%E7%82%B9%E5%8D%8F%E8%AE%AE)、[L2TP](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%BA%8C%E5%B1%82%E9%9A%A7%E9%81%93%E5%8D%8F%E8%AE%AE)、[STP](https://zh.wikipedia.org/wiki/%E7%94%9F%E6%88%90%E6%A0%91%E5%8D%8F%E8%AE%AE)、[ATM](https://zh.wikipedia.org/wiki/%E5%BC%82%E6%AD%A5%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%BC%8F)、
[Ethernet](https://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E7%BD%91)、[GPRS](https://zh.wikipedia.org/wiki/GPRS)、[HSPA](https://zh.wikipedia.org/wiki/%E9%AB%98%E9%80%9F%E5%B0%81%E5%8C%85%E5%AD%98%E5%8F%96)、[PPPoE](https://zh.wikipedia.org/wiki/PPPoE)等。**

## **网络层(Network Layer)**
网络中的两台计算机进行通信时，中间可能要经过许多中间节点甚至不同的通信子网。网络层的任务就是在通信子网中选择一条合适的路径，使发送端传输层所传下来的数据能够通过所选择的路径到达目的端。网络层提供了路由及其相关的功能，可以将众多的数据链路结合成一个互连的网络，这是通过设备的逻辑寻址来实现的。
有关路由的一切事情都在这第一层处理，地址解析和路由是网络层的重要目的，网络层还可以实现拥塞控制、网际互连等功能。在这一层，数据的单位称为数据包（packet）。
**网络层的典型协议包括：[IP](https://zh.wikipedia.org/wiki/%E7%BD%91%E9%99%85%E5%8D%8F%E8%AE%AE)、[IPsec](https://zh.wikipedia.org/wiki/IPsec)、[RIP](https://zh.wikipedia.org/wiki/%E8%B7%AF%E7%94%B1%E4%BF%A1%E6%81%AF%E5%8D%8F%E8%AE%AE)、[OSPF](https://zh.wikipedia.org/wiki/%E5%BC%80%E6%94%BE%E5%BC%8F%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84%E4%BC%98%E5%85%88)、[BGP](https://zh.wikipedia.org/wiki/%E8%BE%B9%E7%95%8C%E7%BD%91%E5%85%B3%E5%8D%8F%E8%AE%AE)、[IGMP](https://zh.wikipedia.org/wiki/%E5%9B%A0%E7%89%B9%E7%BD%91%E7%BB%84%E7%AE%A1%E7%90%86%E5%8D%8F%E8%AE%AE)等。**

## **传输层(Transport Layer)**
传输层是OSI参考模型中唯一负责端到端节点间数据传输和控制功能的一层。传输层也是具有承上启下功能的一层，它下面的三层主要面向网络通信，以确保信息被准确有效地传输，它上面的三层则面向用户主机，为用户提供各种服务，传输层通过弥补网络层服务质量的不足，为会话层提供端到端的可靠数据传输服务。
传输层负责获取全部信息，因此，它必须跟踪数据单元碎片、乱序到达的数据包和其它在传输过程中可能发生的危险。传输层为上层提供端到端的透明的、可靠的数据传输服务。所为透明的传输是指在通信过程中传输层对上层屏蔽了通信传输系统的具体细节。
在这一层，数据的单也称作数据包（packets）。不过，在谈论TCP等具体的协议时又有特殊的叫法，TCP的数据单元称为段（segments），而UDP协议的数据单元称为报文（datagrams）。
**传输层的典型协议包括：[TCP](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)、[UDP](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%8A%A5%E5%8D%8F%E8%AE%AE)、[SPX](https://zh.wikipedia.org/wiki/%E5%BA%8F%E5%88%97%E5%88%86%E7%B5%84%E4%BA%A4%E6%8F%9B)、[PPTP](https://zh.wikipedia.org/wiki/%E9%BB%9E%E5%B0%8D%E9%BB%9E%E9%9A%A7%E9%81%93%E5%8D%94%E8%AD%B0)、[TLS/SSL](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0)、[SCTP](https://zh.wikipedia.org/wiki/%E6%B5%81%E6%8E%A7%E5%88%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)、[DCCP](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)等。**

## **会话层(Session Layer)**
会话层的主要功能是在两个节点之间建立、维护和释放面向用户的连接，并对会话进行管理和控制，保证会话数据可靠传输。会话层是建立在传输层之上，由于利用传输层提供的服务，使得两个会话实体之间不考虑他们之间相隔多远，使用了什么样的通信子网等网络通信细节，从而进行透明的、可靠的数据传输。
会话层不参与具体的传输，它提供包括访问验证和会话管理在内的建立和维护应用之间通信的机制，如服务器验证用户登录便是由会话层完成的。在会话层及以上的高层次中，数据传送的单位不再另外命名。
**会话层没有协议，应用层的HTTP、RPC、SDP、RTCP等协议有类似的功能。**

## **表示层(Presentation Layer)**
表示层的主要作用是为通信双方的应用层实体提供共同的表达手段，使双方能正确地理解所传送的信息。表示层为应用层提供了各种编码和数据转换功能。这些功能可以确保发自某个系统的应用层信息可以被另一个系统的应用层解读出来。
这一层主要解决拥护信息的语法表示问题。它将欲交换的数据从适合于某一用户的抽象语法，转换为适合于OSI系统内部使用的传送语法，即提供格式化的表示和转换数据服务。数据的压缩和解压缩、加密和解密等工作都由表示层负责。
**表示层没有协议，应用层的HTTP、FTP、Telnet等协议有类似的功能，传输层的TLS/SSL也有类似功能。**

## **应用层(Application Layer)**
应用层是OSI参考模型中最靠近用户的一层，负责为用户的应用程序提供网络服务。与OSI参考模型其它层不同的是，它不为任何其他OSI层提供服务，而只是为OSI模型以外的应用程序提供服务，具体来说就是为操作系统或网络应用程序提供访问网络服务的接口。
**应用层的协议最为丰富，典型协议包括：[HTTP](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)、[POP3](https://zh.wikipedia.org/wiki/%E9%83%B5%E5%B1%80%E5%8D%94%E5%AE%9A)、[SMTP](https://zh.wikipedia.org/wiki/%E7%AE%80%E5%8D%95%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)、[IMAP](https://zh.wikipedia.org/wiki/IMAP)、[DNS](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)、[DHCP](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E4%B8%BB%E6%9C%BA%E8%AE%BE%E7%BD%AE%E5%8D%8F%E8%AE%AE)、[FTP](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)、
[Telnet](https://zh.wikipedia.org/wiki/Telnet)、[SSH](https://zh.wikipedia.org/wiki/Secure_Shell)、[NNTP](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E6%96%B0%E8%81%9E%E5%82%B3%E8%BC%B8%E5%8D%94%E8%AD%B0)、[XMPP](https://zh.wikipedia.org/wiki/XMPP)、[SIP](https://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D%E5%8F%91%E8%B5%B7%E5%8D%8F%E8%AE%AE)、[RPC](https://zh.wikipedia.org/wiki/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8)、[RTCP](https://zh.wikipedia.org/wiki/%E5%AE%9E%E6%97%B6%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)、[SDP](https://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D%E6%8F%8F%E8%BF%B0%E5%8D%8F%E8%AE%AE)、
[MMS](https://zh.wikipedia.org/wiki/MMS_(%E5%8D%8F%E8%AE%AE)、[SOAP](https://zh.wikipedia.org/wiki/SOAP)、[SSDP](https://zh.wikipedia.org/wiki/%E7%AE%80%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E5%8D%8F%E8%AE%AE)等。
关于[HTTPS](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)协议，有些文章将其算作应用层的一个协议，不过严格地讲，HTTPS并不是一个单独的协议，而是对工作在一加密连接（TLS或SSL）上的常规HTTP协议的称呼。**

----------

最后，对OSI七层模型结构做一个概括性的总结：

**应用层：触及到应用程序的网络业务
表示层：数据表达
会话层：主机间通信
传输层：端到端的可靠连接
网络层：逻辑寻址和最佳路径
数据链路层：访问介质
物理层：数据二进制的传输**

知乎上有个通俗的例子可供参考：
[生动形象，切中要点的讲解osi七层模型和两主机传输过程。](http://www.zhihu.com/question/24002080)

----------

*参考资料：*
> [网络工作中应该知道的OSI七层模型 - 阿布的博客](http://www.abuve.com/article/14/)
> [OSI网络结构的七层模型 TCP/IP层次模型 - cutepig's blog](http://www.cnblogs.com/cutepig/archive/2007/10/11/921427.html)
> [OSI七层模型笔记 - 简书](http://www.jianshu.com/p/ef5703f65369)
> [OSI七层与TCP/IP五层网络架构详解 -  服务器运维与架构](http://www.ha97.com/3215.html)








