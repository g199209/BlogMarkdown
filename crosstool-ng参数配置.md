title: crosstool-ng参数配置
weburl: crosstool-ng参数配置
date: 2016-06-16 00:01:23
tags: [Linux, Compiler]
categories: 工具之术

---

本文以Samsung S3C2440处理器为例，介绍使用crosstool-ng制作交叉编译链时该如何针对特定的目标CPU进行配置。一般来说，常用的CPU架构都有至少一个现成的示例配置文件，对于S3C2440来说，一般选用`arm-unknown-linux-gnueabi`，下面就以这个示例配置为基础进行修改。

<!--more-->

**crosstool-ng版本为1.22.0**。

## **Paths and misc options**
这部分是crosstool-ng本身的配置，目的是为了提高编译效率与工作效率，具有通用性，在制作不同交叉编译链时都是一样的。

重要配置参数如下：

```
(4) Number of parallel jobs
```
将此值设置为所需线程数即可，一般设置为CPU线程数（对于支持超线程的CPU来说就是核心数 * 2）。

> 此处的多线程参数，crosstool-ng内部是将其传给`make`的`-j`参数实现的，如：`make -j4`

----------

```
[*] Debug crosstool-NG                                                    
  [*]   Save intermediate steps                                             
    [*]     gzip saved states (NEW)
```
开启逐步编译功能，如果编译过程出错，只要找到最后成功的那一步，使用如下命令即可恢复编译：
```
ct-ng <last_successful_step>+
```

关于此功能的详细用法参考[这里](http://www.crifan.com/files/doc/docbook/crosstool_ng/release/html/crosstool_ng.html#restore_from_fail_step)。此外，crosstool-ng还可以实现出错时不立刻退出，即`Interactive shell on failed commands`，在此不多做说明，可参考[这里的说明。](http://www.crifan.com/files/doc/docbook/crosstool_ng/release/html/crosstool_ng.html#error_but_not_exit)

----------
```
(${CT_TOP_DIR}/.build) Working directory
(${build}/src) Local tarballs directory
(${build}/x-tools/${CT_TARGET}) Prefix directory
```
分别是值工作路径、源码包保存路径及目标安装路径。
工作路径：编译时生成的所有文件都放在这里面，一般使用默认配置即可；
源码包保存路径：所需各模块的源码包存放路径，如果之前下载好了可以直接放到这个路径中，如果没有的话crosstool-ng会自动下载。一般将此路径改为工作目录下的`./src/`目录；
目标安装路径：生成的交叉编译链存放的路径，改为需要路径，一般也放在当前工作目录下。

## **Target options**
重要配置参数如下：

```
[*] Use the MMU
```
是否使用MMU，S3C2440有MMU。

----------

```
Endianness: (Little endian)
```
字节序，S3C2440为小端，大部分ARM处理器均为小端。

----------

```
Bitness: (32-bit)
```
CPU运算位数，S3C2440为32 bit，最新型号的ARM Cortex-A处理器为64 bit。

----------

``` shell
(arm920t) Emit assembly for CPU
```
目标CPU核心名称，对应GCC中的`-mcpu`参数。对于S3C2440来说，其ARM核为ARM920T。设置了这个参数后，`Architecture level`及`Tune for CPU`这两个选项就会自动消失。关于这个问题，参考我的另一篇博客文章：[GCC中-march、-mtune、-mcpu三个参数的设置](/2016/06/15/GCC%E4%B8%AD-march%E3%80%81-mtune%E3%80%81-mcpu%E4%B8%89%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84%E8%AE%BE%E7%BD%AE/)。

----------

```
Floating point: (software (no FPU))
()  Use specific FPU
```
与FPU相关配置，S3C2440没有FPU，如果有FPU的话，按实际情况设置即可，其中`Use specific FPU`对应GCC中的`-mfpu`参数，可以指定所用的FPU的类型。

----------

配置好的界面：
![](https://pic.gaomf.store/20160615172011.png)


## **Toolchain options**
重要配置参数如下：

```
[ ] Build Static Toolchain
```
构建静态链接的交叉编译链，这里的静态链接是指交叉编译链本身，不是它编译出来的文件。这个主要是用于需要发布交叉编译链时，如果其他主机没有所需版本的系统库，使用动态链接就会出错，这是就需要使用静态链接。不过静态链接会增大生成的可执行文件的体积，这里只考虑在本机上用，故不选择这个选项。

----------

```
(gmf20160615)  Toolchain ID string
```
可以自定义一个版本号，接在标准默认版本号的后面。

----------

```
(s3c2440) Tuple's vendor string
```
指定交叉编译链标准命名`arch-vendor-kernel-system`中的第二部分`vendor`。

----------

配置好的界面：
![](https://pic.gaomf.store/20160615220831.png)

## **Operating System**
重要配置参数如下：

```
Target OS (linux)
```
目标系统，可以选择`bare-metal`或`linux`。
编译Bootloader时严格来说是该选择`bare-metal`的，不过这样编译Bootloader和编译其他应用就要选用两套不同的交叉编译链，使用起来不是很方便。实际上使用针对`linux`的交叉编译链来编译Bootloader也是可以的，而编译Kernel时必须选择`linux`，故此处一般都选择`linux`。相关说明如下：
> You probably want to say 'y' here if you plan to use your compiler to build bootloaders. It is not yet suitable to build Linux kernels, though, because the APCI stuff relies on the target C library headers being available?!?!...

----------

```
Linux kernel version (4.3 (mainline))
```
Linux kernel版本，可以从列表中选择，也可以自定义一个tar包，自定义tar包的版本有何要求之后再来研究。

----------

```
[*] Build shared libraries
```
是否编译共享库，编译Linux应用时不需要，不过编译Bootloader和Linux kernel等东西时需要，为了保证通用性，一般选上这个。

----------

配置好的界面：
![](https://pic.gaomf.store/20160615223725.png)

## **Binary utilities**
使用默认配置，重要配置参数如下：

```
Binary format: (ELF)
```
生成的二进制文件的格式，当`Target options`中选择了MMU后，此处只能选择ELF格式；若没有MMU，此处一般选择Flat格式。

----------

```
[*] Show Linaro versions
```
选择是否使用Linaro binutils，这是另一套binutils，专为ARM架构做过优化，相关说明：

> Linaro is maintaining some advanced/more stable/experimental versions of binutils, especially for the ARM architecture. Those versions have not been blessed by the binutils comunity (nor have they been cursed either!), but they look to be pretty much stable, and even more stable than the upstream versions. YMMV...

> Linaro binutils is a release of the GNU binutils with bug fixes and enhancements for ARM platforms. GNU binutils is a collection of tools including the ld linker and as assembler.

对此目前还没有研究过，就选用主流的社区版本就行了。

## **C-library & C compiler & Debug facilities & Companion libraries & Companion tools**

主要是各种C标准库、编译器、调试工具及其它一些库的版本选择及相关配置，它们之间似乎并不能任意组合，故这里都选用默认配置。



