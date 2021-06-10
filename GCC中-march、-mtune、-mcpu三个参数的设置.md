title: GCC中-march、-mtune、-mcpu三个参数的设置
weburl: GCC中-march、-mtune、-mcpu三个参数的设置
date: 2016-06-15 16:52:42
tags: [Linux, Compiler, Top]
categories: 工具之术

---

在配置交叉编译链时，需要指定目标CPU的型号，根据网上广为流传的说法，需要同时指定`-march`、`-mtune`、`-mcpu`这三个参数，并且这三个参数还是不同的。在使用crosstool-ng时，就对应`CT_ARCH_ARCH`、`CT_ARCH_TUNE`、`CT_ARCH_CPU`这三个参数，针对S3C2440，网上所有文章中的设置均是：
> Architecture level = CT_ARCH_ARCH = -march = armv4t
> Emit assembly for CPU = CT_ARCH_CPU = -mcpu = arm9tdmi
> Tune for CPU = CT_ARCH_TUNE = -mtune = arm920t

<!--more-->

比如以下这些文章：
> [【整理】crosstool-ng中的Architecture level，Emit assembly for CPU，Tune for CPU对于TQ2440的S3C2440的ARM920T填写何值](http://www.crifan.com/crosstool_ng_architecture_level_emit_assembly_for_cpu_tune_for_cpu_for_tq2440_s3c2440_arm920t/)
> [第一部分：crosstool-ng 制作交叉编译工具链 for s3c2440](http://blog.csdn.net/woshidahuaidan2011/article/details/51344312)
> [用crosstool-ng建立自己的ARM交叉编译工具链 (适用于S3C6410以及其它处理器)](http://blog.csdn.net/HumorRat/article/details/5615298)

**这样设置不能说是错的，然而并不是推荐的设置方法。老版本的GCC的确是需要这样同时设置三个参数的，然而新版本的GCC（具体是哪个版本之后估计不可考了）并不需要这样设置，只需要使用`-mcpu`(`CT_ARCH_CPU`)这一个参数就可以了。**网上很多近些年来的文章还这样说其实是以讹传讹。

以GCC 5.2.0为例，其帮助文档中关于这三个参数的说明如下（ARM部分）：

-march=name

> This specifies the name of the target ARM architecture.  GCC uses this nameto determine what kind of instructions it can emit when generating assemblycode.  This option can be used in conjunction with or instead of the -mcpu=option. 
Permissible names are: armv2, armv2a, armv3, armv3m, armv4,armv4t, armv5, armv5t, armv5e, armv5te, armv6, armv6j, armv6t2, armv6z,armv6zk, armv6-m, armv7, armv7-a, armv7-r, armv7-m, armv7e-m, armv7ve,armv8-a, armv8-a+crc, iwmmxt, iwmmxt2, ep9312.

----------

-mtune=name

> This option specifies the name of the target ARM processor for which GCCshould tune the performance of the code.  For some ARM implementationsbetter performance can be obtained by using this option.
Permissible namesare: arm2, arm250, arm3, arm6, arm60, arm600, arm610, arm620, arm7, arm7m,arm7d, arm7dm, arm7di, arm7dmi, arm70, arm700, arm700i, arm710, arm710c,arm7100, arm720, arm7500, arm7500fe, arm7tdmi, arm7tdmi-s, arm710t,arm720t, arm740t, strongarm, strongarm110, strongarm1100, strongarm1110,arm8, arm810, arm9, arm9e, arm920, **arm920t**, arm922t, arm946e-s, arm966e-s,arm968e-s, arm926ej-s, arm940t, **arm9tdmi**, arm10tdmi, arm1020t, arm1026ej-s,arm10e, arm1020e, arm1022e, arm1136j-s, arm1136jf-s, mpcore, mpcorenovfp,arm1156t2-s, arm1156t2f-s, arm1176jz-s, arm1176jzf-s, cortex-a5, cortex-a7,cortex-a8, cortex-a9, cortex-a12, cortex-a15, cortex-a53, cortex-a57,cortex-a72, cortex-r4, cortex-r4f, cortex-r5, cortex-r7, cortex-m7,cortex-m4, cortex-m3, cortex-m1, cortex-m0, cortex-m0plus,cortex-m1.small-multiply, cortex-m0.small-multiply,cortex-m0plus.small-multiply, exynos-m1, marvell-pj4, xscale, iwmmxt,iwmmxt2, ep9312, fa526, fa626, fa606te, fa626te, fmp626, fa726te, xgene1.
Additionally, this option can specify that GCC should tune the performance
of the code for a big.LITTLE system.  Permissible names are:
cortex-a15.cortex-a7, cortex-a57.cortex-a53, cortex-a72.cortex-a53.

----------
-mcpu=name

> This specifies the name of the target ARM processor.  **GCC uses this name to derive the name of the target ARM architecture (as if specified by -march)and the ARM processor type for which to tune for performance (as if specified by -mtune).  Where this option is used in conjunction with -marchor -mtune, those options take precedence over the appropriate part of this option.**
**Permissible names for this option are the same as those for -mtune.**

**特别注意其中对`-mcpu`参数的说明，指定了`-mcpu`后，GCC编译器会自动推导出`-march`及`-mtune`，故不需要再指定这两个参数，只需要给出`-mcpu`即可，而且`-mcpu`的可能取值与`-mtune`完全相同**。比如S3C2440，只需要加上`-mcpu=arm920t`即可。

在crosstool-ng的新版本（比如1.22.0）中，使用menuconfig进行配置时，一旦设置了`Emit assembly for CPU`这个选项，`Architecture level`及`Tune for CPU`这两个选项就会自动消失。最初在配置的时候还以为是Bug，后面仔细研究下才发现这是crosstool-ng已根据GCC的新特性进行了升级。

