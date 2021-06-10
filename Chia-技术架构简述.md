title: Chia 技术架构简述
weburl: Chia_Architecture_Brief
toc: true
mathjax: true
fancybox: false
tags: [Blockchain, P2P, Cryptography, Consensus, Storage, Top]
date: 2021-05-16 17:33:00
categories: 编程之法

---

Chia（起亚） 是最近极为火热的数字货币项目，对应的货币叫做 Chia Coin，简称 XCH。其核心算法为 PoST（Proof of Space and Time），以替代比特币中的 PoW（Proof of Work）。

使用 PoSpace 空间证明而非 PoW 工作量证明是 Chia 项目宣称的最大优点。据他们的开发者宣称，PoW 耗费太多能源了，不环保，我们来搞点更环保的东西吧，不用 PoW 了，改用 PoSpace，即谁有的硬盘空间多谁的投票权就更大。因此他们还把通常称为白皮书（White Paper）的文档改名叫做绿皮书（Green Paper）。

然而仔细想想这哪里环保了，把一堆硬盘搞来塞满毫无意义的数据比比特币矿机还要更邪恶吧……Chia 的官网上还可笑的宣称硬盘更不容易被垄断，因此个人还有小玩家可以更好的入场，简直是更荒谬的说法，哪个个人会去囤积一堆存不了有用数据的硬盘？

IPFS 好歹还可以存一些实际有用的数据，看起来还真能促进下社会发展，至于 Chia 简直是除了圈钱和泡沫看不到任何其他意义。不过抛开实际意义，由于最近也研究了下 Chia 的文档和代码，就单纯的来和大家分享下 Chia 的技术实现吧。

<!--more-->

## 软件架构

Chia 的整体架构图如下：

![](https://img.gaomf.cn/chia-network-architecture.png-height)

整个系统中主要有 3 种类型的参与者：
- Farmer，农民
- TimeLord，时间领主
- Full Node，全能节点（中文翻译不确定）

### Farmer

绝大部分参与者都是农民 Farmer，如何成为农民也很简单，直接去下载个打包好的客户端运行就好了。

至于为什么叫 Farmer 呢？当然是为了凸显 Chia 的*绿色环保*喽，我们不是在浪费能源的挖矿，我们是在*环保*的种田！

Farmer 的工作也很简单，基本就是两步：
1. Plotting，播种
2. Farming，摸奖

Farmer 生成的 Plots file（P 盘文件） 可能会分布在很多台机器上，因此需要在这些机器上都部署上用来支持摸奖的服务，这个服务就被称为 Harvester 收割机。Farmer 接收到来自 TimeLord 的 Challenge（质询） 后，会将此 Challenge 转发到所连接的所有 Harvester 上。

#### Plotting

Plotting 的目的是在磁盘上生成一大堆 Plots file，根据其实现代码，这一过程可分为 4 步：

1.  Phase 1, forward propagation. 计算出所需的 7 张表及 f 函数集合，这一步实际上就已经生成了 PoSpace 所需的所有数据了，只是生成的临时文件还太大，需要后续来压缩下。f 函数的计算过程中会用到 Plot ID，Plot ID 直接决定了 Plots file 的内容；
2.  Phase 2, backprogagation. 主要目的在于消除表中的无意义项（Dead entries），以减少磁盘空间占用；
3.  Phase 3, compresses. 进一步压缩生成的临时文件 1，生成临时文件 2，临时文件 2 其实就是最终的 Plots file 了；这一步做了重排序，会使得生成的表中各项的顺序发生变化，不是按 PoSpace 要求的某种顺序。
4.  Phase 4, write checkpoint table. 写入检查点表，其意义在于加快查找表的过程。

最后，若最终路径（`--final_dir`）与临时文件 2 的路径（`--tmp2_dir`）是一样的，简单的把临时文件 2 做个 Rename 重命名即可；否则做一次数据拷贝生成最终 Plots file。

此处之所以要 Copy 或 Rename 一下而不是直接把临时文件 2 作为最终文件的主要原因是为了分离临时文件及最终文件的存储位置。

根据 Chia 的设计，Plots file 越多越好，因此显然要把它们存放在廉价的大容量存储系统，如本地机械盘或云端的低价存储中。然而此类系统通常随机读写能力不佳，甚至是直接不支持随机写入。可在生成临时文件 2 时是需要随机写入的，且写入的 IOPS 对生成文件的速度有显著影响，因此在通常实践中，会把临时文件写到 SSD 上。

Plotting 的逻辑是由 `DiskPlotter` 这个类来实现的。

#### Farming

Farming 的过程就是对一系列 Challenges 的证明响应，每一轮证明过程都是以一个 256 bit 的 Challenge 为输入，输出是一个 PoSpace 结构，其中包括 Plots file 的公钥，Pool 的公钥， Proof 结果等。其中最重要的就是 Proof 结果。

生成一个区块的过程中会产生 64 次 Challenges，这一过程在客户端 Farming 的界面中可以看到：

![](https://img.gaomf.cn/20210510-170238%402x.png-height)

每一个这样的点被称为一个 Signage Point。

为减少 IO 次数及所需网络带宽，目前的实现中采用了一种类似于预筛选的方法，先用较少次数的读计算出一个 Quality 值，并根据特定算法评估此 Quality 对于当前 Signage Point 来说够不够好，如果够好的话再去获取完整 Proof 结果。获取 Proof & Quality 的过程是由 Harvester 完成的，评估 Quality 质量则是 Farmer 的工作。

#### Harvester

Harvester 负责管理某台机器上的所有 Plots file，并接收来自 Farmer 的 Challenge，返回每个 Plots file 的 PoSpace 及 PoSpace Quality。

这部分代码是由 `DiskProver` 类实现的。

Challenge 的高 $k$ bit 表示 f7 要满足的一些性质，通过对 C1, C2, C3 表的一通查询最终可以确定 Table 7 中有几项满足要求，可能有若干项满足要求，也可能一项都没有，平均期望是存在 1 项满足要求。

这里有几项满足要求就意味着这个 Plots file 中存在多少个最终的 Proof 证明结果。

之后的查找过程示意如下：

![](https://img.gaomf.cn/chia_plots_file_table.png)

不过要注意的是，从 Table 7 中的一项表项只能找到 Table 6 中对应的**一项**，这样依次找下去就可以得到 Table 1 中的 32 项，每项是由 2 个 $k$ bit 的整数构成的，因此最终结果就有 $k*64$ bit。对这 64 个数进行重新排序（排序规则是由 PoST 算法决定的），最终就可以生成一个长长的字符串，这就是 PoSpace 的证明结果。

至于 Quality 是怎么来的呢？Challenge 最低 5 bit 的含义是 Table 6 ～ Table 2 在生成 Quality 时应该选择左边的值还是右边的值，按此规则进行选取后 Table 7 中的一项就会对应得到 Table 1 中的一项，即 2 个整数。将 Challenge 与这两个整数简单的二进制连起来，并计算 SHA-256，得到的结果就是 Quality 值。

### Timelord

Timelord 一般被翻译为时间领主，它负责向 Farmer 发起质询（Challenge）并计算 VDF，计算完成后打包成新的区块，实际上整个 Chia 链中的区块都是由 TimeLord 计算生成的。TimeLord 最终决定了哪个 Farmer 的某个 Plots file 赢得了当前区块，即摸中奖了可以获得 XCH 奖励。

那如何保证 TimeLord 是公平的而不是邪恶的始终选取自己的 plots file 呢？这就是由 Chia 的 PoST 算法决定的了。离 Challenge 越近（越优）的 PoSpace 会使得 TimeLord 计算 VDF 的速度越快。系统中不止有一个 TimeLord，而是有很多 TimeLord 在互相竞争，哪个 TimeLoad 先计算完成 VDF 成功打包区块，那整个链就会沿此区块继续延伸，其他在计算同一高度区块的 TimeLord 就会失败。

此过程与传统 Bitcoin 的运行模式基本一模一样，可以猜想，对于分支情况的处理也应该和比特币基本相同。然而一个显著区别是，Bitcoin 奖励的是矿工，即最终成功生成新区块的参与者，而在 Chia 中，TimeLord 是没有任何奖励的，完全是自愿劳动:) 被奖励的是 Farmer。

那无偿劳动为啥有人来干呢？根据某个 Chia 核心开发人员的说法是，当 TimeLord 好处多多，大家都会争着来干的，最显著的好处是，自己部署一个 TimeLord 与自己的 Harvester 离得近网络延迟小，避免自己由于网络延迟太大而成为炮灰。即由于网络延迟导致自己的 PoSpace 很久之后才被送到某个遥远的 TimeLord 上，导致根本没有机会被打包到区块中，即使自己的 PoSpace 比其他人更优。

TimeLord 是 CPU 密集型任务，目前的开源实现强制要求运行平台支持 AVX512-IFMA 指令集。如果某个 TimeLord 的运行速度能压倒性的快于其他 TimeLord，那它理论上是可以凭借算力而非磁盘空间来控制整个链的，因此按照 Chia 开发者的说法，要把运行得最快的 TimeLord 算法开源出来，而且使得 ASIC 的运算速度没法超过通用 CPU，这样才能避免邪恶 TimeLord 的出现。

### Full Node

Full Node 的作用是广播中转各种消息，创建区块，保存和维护历史区块，与系统的其他参与者通信等。不同参与者之间的通信就是靠 Full Node 来完成的。

Full Node 间的一致性使用的是与比特币一样的 Gossip 协议。

## 一些重要算法

算法部分没有仔细研究，此处更多的是给出一些深入研究的链接。

### PoST 算法

Chia 最重要的算法当然要数 PoST 算法了，PoST 算法是由两部分构成的，PoSpace + VDF。

[PoSpace 的文档](https://www.chia.net/assets/Chia_Proof_of_Space_Construction_v1.1.pdf)

[VDF 的文档](https://eprint.iacr.org/2018/712.pdf)

为什么只有 PoSpace 是不够的还需要 VDF 呢？因为整个区块链网络是个 P2P 网络，产生一个 Challenge 后需要去收集所有 Farmer 的 Proof，区块链设计的核心哲学就是没有邪恶的中心节点，那怎么确定哪个 Proof 是最优的呢？如果这个判断进行得很快，比如简单的比比差值，那所有的 TimeLoad 都可以马上宣称某个 Proof 为最优，此时区块如何增长就完全不可控了，所以 PoT 也是必不可少的。需要通过计算 VDF 的过程让全网能够就哪个 Proof 是最优的达成共识。

### 一致性协议

一致性协议中主要介绍的是链的延伸过程，在 Chia 的绿皮书中对此有说明，不过目前有一份更新的 Google Doc:

[Chia Consensus Algorithm](https://docs.google.com/document/d/1tmRIb7lgi4QfKkNaxuKOBHRmwbVlGL4f7EsBDr_5xZE/edit)

### 签名算法

所有区块链技术的最底层基石都是密码学，特别是各种数字签名技术。Chia 中签名用的算法是 BLS12-381。

BLS 算法是 2003 年由斯坦福大学的 Dan **B**oneh，Ben **L**ynn 以及 Hovav **S**hacham 提出的一种基于 ECC 的数字签名算法，和 ECDSA 的用处是一样的。该方案是一个基于双线性映射且具有唯一确定性的签名方案。BLS的主要思想是待签名的消息散列到一个椭圆曲线上的一个点，并利用双线性映射 e 函数的交换性质，在不泄露私钥的情况下，验证签名。BLS的算法在签名合并，多签，m/n 多签有丰富的应用。

而 BLS12-381 则一种具体的 BLS 签名算法，此算法由 Sean Bowe 于 2017 年提出，最早被用于一个叫 Zcash 的数字货币项目中，现在不少其他区块链项目也用了此算法。

在 Chia 的实现中需要用到不止一对密钥，比如钱包的密钥，Farmer 用的农民密钥等。这些密钥不是独立的，而是由一个主私钥通过私钥派生算法得到的，对于 BLS12-381 算法来说怎么生成这些密钥可以参考这个：

[EIP-2333: BLS12-381 Key Generation](https://eips.ethereum.org/EIPS/eip-2333)

至于主私钥怎么来的呢，第一次启动 Chia 客户端时会创建一个由 24 个单词组成的助记词，这些助记词就是用来生成主私钥的。

Plotting 时会生成一个随机主私钥，通过它可以派生出一个本地私钥，这个本地私钥又可以导出一个本地公钥，最终，本地公钥与农民公钥（Farmer Public Key）融合，生成了绘图公钥（Plot Public Key），最后矿池公钥（Pool Public Key）和绘图公钥（Plot Public Key）会被组合到一起，并进行一次哈希，哈希的结果被称为绘图 ID（Plot ID）。

上述提及的绘图 ID，随机主私钥，农民公钥与矿池公钥均会被记录到 Plots file 的 Header 中。

生成区块时，需要用与 Plot file 匹配的矿池私钥（Pool Private Key）进行一次签名。

## 开源项目代码结构

Chia 的业务逻辑，网络，一致性算法等是用 Python 写的，即 [chia-blockchain](https://github.com/Chia-Network/chia-blockchain) 这个项目。这个项目也被视为 Chia 的主项目在 GitHub 上获得了最多的 Star。最终各平台上能运行的完整的程序也是在这个项目中发布 Release 版本的。

至于 GUI 部分是基于 Electron 开发的，对应项目为 [chia-blockchain-gui](https://github.com/Chia-Network/chia-blockchain-gui)。

核心的 PoST 算法则是 C++ 写的，分为两个项目：
- [chiapos](https://github.com/Chia-Network/chiapos)，PoSpace 相关代码，包括 Plots file 的生成及验证；
- [chiavdf](https://github.com/Chia-Network/chiavdf)，TimeLord 上运行的 VDF 算法。

Chia 中使用的 BLS12-381 数字签名算法的实现为：[bls-signatures](https://github.com/Chia-Network/bls-signatures)

此外 Chia 还开发了一个叫 Chialisp 的智能合约语言，相关项目有：
- [clvm](https://github.com/Chia-Network/clvm)，用 Python 写的 Chialisp 虚拟机；
- [clvm-rs](https://github.com/Chia-Network/clvm_rs)，用 Rust 写的 Chialisp 虚拟机；
- [clvm\_tools](https://github.com/Chia-Network/clvm_tools)，一些支持工具。



> 参考资料：
>
> [硬盘危机——Chia 挖矿背后的原理与技术细节（一）](https://dgideas.net/2021/hard-drive-crisis-the-principles-and-technical-details-behind-chia-mining-i/)
>
> [Chia Green Paper](https://www.chia.net/assets/ChiaGreenPaper.pdf)
>
> [Chia挖矿：深入浅出聊P盘（绘图 Plots）](https://zhuanlan.zhihu.com/p/366400067)
