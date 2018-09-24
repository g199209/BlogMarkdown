title: crosstool-ng的基本使用流程
date: 2016-06-15 11:33:58
tags: [Linux, 工具]
categories: 嵌入式

---

按照[crosstool-ng的安装](/2016/06/14/crosstool-ng%E7%9A%84%E5%AE%89%E8%A3%85/)中的做法安装好crosstool-ng后就可以使用crosstool-ng了，基本使用流程在其官网上有说明：[Using a released version](http://crosstool-ng.org/#using_a_released_version)，这里做下总结。

<!--more-->

**需要注意的是，目前新版本的crosstool-ng不能以root身份运行，否则会提示以下错误：**
``` shell
[ERROR]  You must NOT be root to run crosstool-NG
```

故下面的所有操作都**不要**在root下进行。当然，如果非要用root运行的话也是可以的，见本文最后的附注。

## 创建工作目录
一般专门创建一个工作目录用于之后的编译过程：
```
mkdir ~/toolchain
cd ~/toolchain
mkdir src
mkdir x-tools
export build="$(pwd)"
```

这里创建了工作目录`~/toolchain/`及两个子文件夹`~/toolchain/src`、`~/toolchain/x-tools`，分别用于存放下载到的源码tarball及最终生成的交叉编译链。

最后新建了一个`build`变量，代表当前工作目录。

## 使用Sample示例配置
crosstool-ng中自带了很多示例配置，可用以下命令列出所有Samples及查看某个特定的示例的主要配置参数：
```
ct-ng list-samples
ct-ng show-<sample>
```

如果有合适的示例配置可用的话，可以在其基础上进行修改。**在工作目录中**执行`ct-ng <sample>`，如：
``` shell
ct-ng arm-unknown-linux-gnueabi
```

此命令会自动在当前目录下创建示例的配置文件，shell输出类似这样的：
``` shell
  LN    config
  MKDIR config.gen
  IN    config.gen/arch.in
  IN    config.gen/kernel.in
  IN    config.gen/cc.in
  IN    config.gen/binutils.in
  IN    config.gen/libc.in
  IN    config.gen/debug.in
  CONF  config/config.in
#
# configuration written to .config
#

***********************************************************

Initially reported by: Alexander BIGGA
URL: http://sourceware.org/ml/crossgcc/2008-06/msg00031.html

***********************************************************

Now configured for "arm-unknown-linux-gnueabi"
```

**网上很多教程中都说要手动复制配置文件，目前新版本的crosstool-ng不需要这样做，而且手动复制还会出错，直接使用上述命令生成示例配置即可。**

## 修改配置文件
一般来说，示例配置是不能直接使用的，需要对其进行一些修改，**在工作目录中**输入：
``` shell
ct-ng menuconfig
```
会进入下图这样的menuconfig配置界面：
![](http://gmf.shengnengjin.cn/20160615105843.png)

一般需要更改下路径，进入`Paths and misc options`，修改`Local tarballs directory` & `Prefix directory`。前者是下载的源码包存放的路径，后者是生成的交叉编译链存放的路径，将其改为之前在工作目录下新建的文件夹即可：
``` shell
(${build}/src) Local tarballs directory
(${build}/x-tools/${CT_TARGET}) Prefix directory
```

具体参数的配置方法见：[crosstool-ng参数配置](/2016/06/16/crosstool-ng%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE/)

## 编译
配置完成后使用
``` shell
ct-ng build
```
命令即可开始编译。crosstool-ng会先下载所需的tar包，之后进行编译。如果没有出现错误的话，最终会在`Prefix directory`中生成所需的交叉编译链，可使用tar进行打包。使用时将其中的`bin`目录加入`PATH`中即可。

----------

## 附：以root身份运行crosstool-ng的方法

出于安全性考虑，crosstool-ng默认不能以root身份运行，如果确定要在root下运行，需要在配置中开启这一选项：
``` shell
-- Paths and misc options --
[*] Try features marked as EXPERIMENTAL
    [*]   Allow building as root user (READ HELP!) (NEW)
```

在这一选项的HELP文档中，可以看到crosstool-ng的作者极力反对用root运行：）
``` shell
 │ CT_ALLOW_BUILD_AS_ROOT:                                                                                                  │  
  │                                                                                                                          │  
  │ You normally do *not* need to be root to build a toolchain using                                                         │  
  │ crosstool-NG. In fact, it is *VERY* dangerous to run as root, as                                                         │  
  │ crosstool-NG will, as part of the build process, remove a few                                                            │  
  │ directories. If anything goes wrong, running as root can ruin                                                            │  
  │ your host distribution.                                                                                                  │  
  │                                                                                                                          │  
  │ I can't stress it enough:  DO  NOT  RUN  AS  ROOT  !!                                                                    │  
  │                                                                                                                          │  
  │ Do not run as root, you've been warned.                                                                                  │  
  │ Do not come whining, if it nukes your host system.                                                                       │  
  │ Do not come whining, if you lose any data.                                                                               │  
  │ Do not run as root.                                                                                                      │  
  │                                                                                                                          │  
  │ Do not run as root, you've been warned.                                                                                  │  
  │ Do not come whining, if the Earth stops rotating.                                                                        │  
  │ Do not come whining, if kittens are smashed.                                                                             │  
  │ Do not run as root.                                                                                                      │  
  │                                                                                                                          │  
  │ Do not run as root, do not run as root!                                                                                  │  
  │ (ad libitum)
```

