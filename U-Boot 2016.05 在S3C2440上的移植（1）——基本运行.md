title: U-Boot 2016.05 在S3C2440上的移植（1）——基本运行
date: 2016-06-19 17:12:00
tags: [Linux, Bootloader]
categories: 编程之法

---

相关环境：
系统版本：Ubuntu 16.04 64bit
U-boot 版本：2016.05
交叉编译链GCC版本为5.20，使用crosstool-ng自行制作。

基本运行的目标是能够顺利的编译通过，是之后移植修改的基础。

<!--more-->

## 创建2440配置文件
复制2410的板子配置文件，以便在此基础上建立2440的配置文件。
涉及到的文件可在`board/samsung/smdk2410/MAINTAINERS`文件中找到。此文件实际上是一个代码维护说明，指出了这个板子的配置包含哪些文件，由谁维护等，其内容如下：
```
SMDK2410 BOARD
M:      David Müller <d.mueller@elsoft.ch>
S:      Maintained
F:      board/samsung/smdk2410/
F:      include/configs/smdk2410.h
F:      configs/smdk2410_defconfig
```

从中可以看到，有三处需要复制：
```bash
cp -r board/samsung/smdk2410/ board/samsung/smdk2440/
mv board/samsung/smdk2440/smdk2410.c board/samsung/smdk2440/smdk2440.c

cp -r include/configs/smdk2410.h include/configs/smdk2440.h

cp -r configs/smdk2410_defconfig configs/smdk2440_defconfig
```

### MAINTAINERS文件

修改`board/samsung/smdk2440/MAINTAINERS`文件，将其中2410的部分全部改为2440：
```diff
--- board/samsung/smdk2410/MAINTAINERS
+++ board/samsung/smdk2440/MAINTAINERS
@@ -1,6 +1,6 @@
-SMDK2410 BOARD
-M:     David Müller <d.mueller@elsoft.ch>
+SMDK2440 BOARD
+M:     Mingfei Gao  <G199209@gmail.com>
 S:     Maintained
-F:     board/samsung/smdk2410/
-F:     include/configs/smdk2410.h
-F:     configs/smdk2410_defconfig
+F:     board/samsung/smdk2440/
+F:     include/configs/smdk2440.h
+F:     configs/smdk2440_defconfig

```

### Kconfig文件

同理修改`board/samsung/smdk2440/Kconfig`文件：

```diff
--- board/samsung/smdk2410/Kconfig
+++ board/samsung/smdk2440/Kconfig
@@ -1,7 +1,7 @@
-if TARGET_SMDK2410
+if TARGET_SMDK2440
 
 config SYS_BOARD
-       default "smdk2410"
+       default "smdk2440"
 
 config SYS_VENDOR
        default "samsung"
@@ -10,6 +10,6 @@
        default "s3c24x0"
 
 config SYS_CONFIG_NAME
-       default "smdk2410"
+       default "smdk2440"
 
 endif
```

### Makefile文件

同理修改`board/samsung/smdk2440/Makefile`文件：

```diff
--- board/samsung/smdk2410/Makefile
+++ board/samsung/smdk2440/Makefile
@@ -4,6 +4,9 @@
 #
 # SPDX-License-Identifier:     GPL-2.0+
 #
+# Modified by Mingfei Gao
+# 2016.06.17
+#
 
-obj-y  := smdk2410.o
+obj-y  := smdk2440.o
 obj-y  += lowlevel_init.o
```

### smdk2440.c文件

同理修改`board/samsung/smdk2440/smdk2440.c`文件：

```diff
--- board/samsung/smdk2410/smdk2410.c
+++ board/samsung/smdk2440/smdk2440.c
@@ -96,8 +96,8 @@
 
 int board_init(void)
 {
-       /* arch number of SMDK2410-Board */
-       gd->bd->bi_arch_number = MACH_TYPE_SMDK2410;
+       /* arch number of SMDK2440-Board */
+       gd->bd->bi_arch_number = MACH_TYPE_SMDK2440;
 
        /* adress of boot parameters */
        gd->bd->bi_boot_params = 0x30000100;

```

### smdk2440_defconfig文件

修改`configs/smdk2440_defconfig`文件：

```diff
--- configs/smdk2410_defconfig
+++ configs/smdk2440_defconfig
@@ -1,7 +1,7 @@
 CONFIG_ARM=y
-CONFIG_TARGET_SMDK2410=y
+CONFIG_TARGET_SMDK2440=y
 CONFIG_HUSH_PARSER=y
-CONFIG_SYS_PROMPT="SMDK2410 # "
+CONFIG_SYS_PROMPT="SMDK2440 # "
 CONFIG_CMD_USB=y
 # CONFIG_CMD_SETEXPR is not set
 CONFIG_CMD_DHCP=y

```

### arch/arm/Kconfig文件

因为前面修改了`smdk2440_defconfig`文件，将其中的配置项名称由`CONFIG_TARGET_SMDK2410`改成了`CONFIG_TARGET_SMDK2440`，故还需要修改`arch/arm/Kconfig`文件，在其中添加`SMDK2440`的板子配置才行：

```diff
--- a/arch/arm/Kconfig
+++ b/arch/arm/Kconfig
@@ -96,6 +96,10 @@ config TARGET_SMDK2410
        bool "Support smdk2410"
        select CPU_ARM920T
 
+config TARGET_SMDK2440
+       bool "Support smdk2440"
+       select CPU_ARM920T
+
 config TARGET_ASPENITE
        bool "Support aspenite"
        select CPU_ARM926EJS
@@ -854,6 +858,7 @@ source "board/phytec/pcm051/Kconfig"
 source "board/phytec/pcm052/Kconfig"
 source "board/ppcag/bg0900/Kconfig"
 source "board/samsung/smdk2410/Kconfig"
+source "board/samsung/smdk2440/Kconfig"
 source "board/sandisk/sansa_fuze_plus/Kconfig"
 source "board/schulercontrol/sc_sps_1/Kconfig"
 source "board/siemens/draco/Kconfig"
```

### mach-types.h文件

`smdk2440.c`中将`MACH_TYPE_SMDK2410`改成了`MACH_TYPE_SMDK2440`，此时需要在`arch/arm/include/asm/mach-types.h`中添加对应的代码才行：

```diff
--- a/arch/arm/include/asm/mach-types.h
+++ b/arch/arm/include/asm/mach-types.h
@@ -57,6 +57,7 @@ extern unsigned int __machine_arch_type;
 #define MACH_TYPE_IQ80321              169
 #define MACH_TYPE_KS8695               180
 #define MACH_TYPE_SMDK2410             193
+#define MACH_TYPE_SMDK2440             1999
 #define MACH_TYPE_CEIVA                200
 #define MACH_TYPE_VOICEBLUE            218
 #define MACH_TYPE_H5400                220
@@ -1648,6 +1649,19 @@ extern unsigned int __machine_arch_type;
 # define machine_is_smdk2410() (0)
 #endif
 
+#ifdef CONFIG_ARCH_SMDK2440
+# ifdef machine_arch_type
+#  undef machine_arch_type
+#  define machine_arch_type     __machine_arch_type
+# else
+#  define machine_arch_type     MACH_TYPE_SMDK2440
+# endif
+# define machine_is_smdk2440()  (machine_arch_type == MACH_TYPE_SMDK2440)
+#else
+# define machine_is_smdk2440()  (0)
+#endif
+
+
 #ifdef CONFIG_ARCH_CEIVA
 # ifdef machine_arch_type
 #  undef machine_arch_type
```

此处`MACH_TYPE`的宏定义值是根据Linux kernel中的定义值来确定的，二者最好匹配起来，不过要是不同也没有关系，在引导内核的时候也可以通过环境变量来设置这个值。

### smdk2440.h文件

修改`include/configs/smdk2440.h`文件：

```diff
--- a/include/configs/smdk2440.h
+++ b/include/configs/smdk2440.h
@@ -18,8 +18,8 @@
  * (easy to change)
  */
 #define CONFIG_S3C24X0         /* This is a SAMSUNG S3C24x0-type SoC */
-#define CONFIG_S3C2410         /* specifically a SAMSUNG S3C2410 SoC */
-#define CONFIG_SMDK2410                /* on a SAMSUNG SMDK2410 Board */
+#define CONFIG_S3C2440         /* specifically a SAMSUNG S3C2440 SoC */
+#define CONFIG_SMDK2440                /* on a SAMSUNG SMDK2440 Board */
 
 #define CONFIG_SYS_TEXT_BASE   0x0

```


### 添加交叉编译链路径

最后，添加交叉编译链的路径，根据U-Boot README说明，可以不修改Makefile文件，而是使用环境变量`CROSS_COMPILE`来指定交叉编译链，不过这里还是采用了修改Makefile文件的方法。修改顶层Makefile文件，即`Makefile`：

```diff
--- a/Makefile
+++ b/Makefile
@@ -246,6 +246,8 @@ ifeq ($(HOSTARCH),$(ARCH))
 CROSS_COMPILE ?=
 endif
 
+CROSS_COMPILE = "/home/gmf/uboot/toolchain/x-tools/arm-s3c2440-linux-gnueabi/bin/arm-s3c2440-linux-gnueabi-"
+
 KCONFIG_CONFIG ?= .config
 export KCONFIG_CONFIG

```

### 测试编译

至此，已成功创建2440的配置文件，使用以下命令进行配置和编译：
``` 
make smdk2440_defconfig
make all
```

顺利编译后就会在U-Boot根目录下生成所需的`u-boot.bin`文件。当然，此文件目前还是针对2410的，接下来需要针对2440进行修改移植。

## Stage 1代码移植
第一阶段代码一般是汇编代码，用于初始化硬件，建立C语言运行环境等。从`README`文件中可以找到U-Boot初始化流程的说明，位于`Board Initialisation Flow`部分，其中提到最初执行的启动代码为`start.S`文件，对于2440来说，就是`arch/arm/cpu/arm920t/start.S`文件，下面针对2440来修改此文件。

### 修改屏蔽中断代码

修改屏蔽中断的代码：
```diff
@@ -78,6 +78,10 @@ copyex:
 	ldr	r1, =0x3ff
 	ldr	r0, =INTSUBMSK
 	str	r1, [r0]
+# elif defined(CONFIG_S3C2440)
+	ldr	r1, =0x7fff
+	ldr	r0, =INTSUBMSK
+	str	r1, [r0]
 # endif
 
 	/* FCLK:HCLK:PCLK = 1:2:4 */
```

此处的设置参考2440数据手册中对`INTSUBMSK`寄存器的说明：
![](http://gmf.shengnengjin.cn/20160618160432.png)

### 更改时钟设置

更改时钟设置，默认`start.S`中只有时钟分频设置，PLL的设置放在Stage 2中进行。对此进行下修改，在此完成所有时钟部分的初始化设置。一般都将2400配置为：FCLK=400M；HCLK=100M；PCLK=50M，需要将原来的：

```arm
/* FCLK:HCLK:PCLK = 1:2:4 */
/* defualt FCLK is 120 MHz ! */
ldr     r0, =CLKDIVN
mov     r1, #3
str     r1, [r0]
```

改为：

```arm
/* Initialize Clock */
# if defined(CONFIG_S3C2440)
    #define MPLLCON 0x4C000004
    #define UPLLCON 0x4C000008

    #define S3C2440_MPLL_400M ((92 << 12) | (1 << 4) | (1))
    #define S3C2440_UPLL_48M  ((56 << 12) | (2 << 4) | (2))         

    /* FCLK:HCLK:PCLK = 1:4:8 */
    ldr     r0, =CLKDIVN
    mov     r1, #5
    str     r1, [r0]

    /* MMU Set Async Bus Mode */
    mrc     p15, 0, r0, c1, c0, 0
    orr     r0, r0, #0xC0000000
    mcr     p15, 0, r0, c1, c0, 0

    /* Set PLL */
    ldr     r0, =MPLLCON
    ldr     r1, =S3C2440_MPLL_400M
    str     r1, [r0]

    /* Some delay between MPLL and UPLL */
    mov     r0, #(1 << 13)
    mov     r1, #0
1:
    subs    r0, r0, #1
    cmp     r0, r1
    bne     1b

    ldr     r0, =UPLLCON
    ldr     r1, =S3C2440_UPLL_48M
    str     r1, [r0]

    /* Some delay */
    mov     r0, #(1 << 14)
    mov     r1, #0
2:
    subs    r0, r0, #1
    cmp     r0, r1
    bne     2b

# else
    /* FCLK:HCLK:PCLK = 1:2:4 */
    /* defualt FCLK is 120 MHz ! */
    ldr     r0, =CLKDIVN
    mov     r1, #3
    str     r1, [r0]
# endif
```

### 修改SDRAM设置

`start.S`文件会跳转到`board/samsung/smdk2440/lowlevel_init.S`中继续执行，下面来修改此文件。

修改SDRAM刷新周期、预充电时间，根据SDRAM的数据手册刷新要求为8192/64ms，一般HCLK设置为100M，故需要将`REFCNT`的值改为1268；Trp根据数据手册为20ns，故选择3 clk即可；另外2440中没有Tchr寄存器，需要删掉：
```diff
@@ -105,8 +105,7 @@
 /* REFRESH parameter */
 #define REFEN                  0x1     /* Refresh enable */
 #define TREFMD                 0x0     /* CBR(CAS before RAS)/Auto refresh */
-#define Trp                    0x0     /* 2clk */
+#define Trp                    0x1     /* 3clk */
 #define Trc                    0x3     /* 7clk */
-#define Tchr                   0x2     /* 3clk */
-#define REFCNT                 1113    /* period=15.6us, HCLK=60Mhz, (2048+1-15.6*60) */
+#define REFCNT                 1268    /* period=7.81us, HCLK=100Mhz, (2048+1-7.81*100) */
 /**************************************/
```

----------

Stage 1中，`start.S`还会跳转去执行`_main`，不过这个是平台无关的代码，移植时不需要考虑。下面开始移植Stage 2代码。

## Stage 2代码移植
因为在Stage 1中已经完成了PLL时钟的初始化，这里将Stage 2中的相关代码全部删掉。所在文件为`board/samsung/smdk2440/smdk2440.c`：

```diff
--- a/board/samsung/smdk2440/smdk2440.c
+++ b/board/samsung/smdk2440/smdk2440.c
@@ -16,36 +16,6 @@
 
 DECLARE_GLOBAL_DATA_PTR;
 
-#define FCLK_SPEED 1
-
-#if (FCLK_SPEED == 0)          /* Fout = 203MHz, Fin = 12MHz for Audio */
-#define M_MDIV 0xC3
-#define M_PDIV 0x4
-#define M_SDIV 0x1
-#elif (FCLK_SPEED == 1)                /* Fout = 202.8MHz */
-#define M_MDIV 0xA1
-#define M_PDIV 0x3
-#define M_SDIV 0x1
-#endif
-
-#define USB_CLOCK 1
-
-#if (USB_CLOCK == 0)
-#define U_M_MDIV       0xA1
-#define U_M_PDIV       0x3
-#define U_M_SDIV       0x1
-#elif (USB_CLOCK == 1)
-#define U_M_MDIV       0x48
-#define U_M_PDIV       0x3
-#define U_M_SDIV       0x2
-#endif
-
-static inline void pll_delay(unsigned long loops)
-{
-       __asm__ volatile ("1:\n"
-         "subs %0, %1, #1\n"
-         "bne 1b" : "=r" (loops) : "0" (loops));
-}
 
 /*
  * Miscellaneous platform dependent initialisations
@@ -53,27 +23,8 @@ static inline void pll_delay(unsigned long loops)
 
 int board_early_init_f(void)
 {
-       struct s3c24x0_clock_power * const clk_power =
-                                       s3c24x0_get_base_clock_power();
        struct s3c24x0_gpio * const gpio = s3c24x0_get_base_gpio();
 
-       /* to reduce PLL lock time, adjust the LOCKTIME register */
-       writel(0xFFFFFF, &clk_power->locktime);
-
-       /* configure MPLL */
-       -       writel((M_MDIV << 12) + (M_PDIV << 4) + M_SDIV,
-              &clk_power->mpllcon);
-
-       /* some delay between MPLL and UPLL */
-       pll_delay(4000);
-
-       /* configure UPLL */
-       writel((U_M_MDIV << 12) + (U_M_PDIV << 4) + U_M_SDIV,
-              &clk_power->upllcon);
-
-       /* some delay between MPLL and UPLL */
-       pll_delay(8000);
-
        /* set up the I/O ports */
        writel(0x007FFFFF, &gpio->gpacon);
        writel(0x00044555, &gpio->gpbcon);

```

## 编译运行
做完上面的修改后，编译UBoot：

```
make smdk2440_defconfig
make all
```

将生成的`u-boot.bin`文件通过JTAG烧写到板子的Nor Flash中，之后连上串口（Serial 0），波特率设置为115200，复位后即可看到串口有数据输出：

```no-highlight
U-Boot 2016.05-gf52ee9c-dirty (Jun 19 2016 - 16:45:57 +0800)

CPUID: 32440001
FCLK:      400 MHz
HCLK:      100 MHz
PCLK:       50 MHz
DRAM:  64 MiB
WARNING: Caches not enabled
Flash: 0 Bytes
NAND:  0 MiB
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   CS8900-0
Error: CS8900-0 address not set.

SMDK2440 # 
```

这说明U-Boot已经运行起来了，不过没有识别出Flash，而且网卡配置有误，故我们还需要继续进行修改。

> 移植好的U-Boot见[我的Github项目](https://github.com/g199209/U-Boot_201605_S3C2440)。
