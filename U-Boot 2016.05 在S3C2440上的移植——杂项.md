title: U-Boot 2016.05 在S3C2440上的移植——杂项
date: 2016-06-26 21:23:14
tags: [Bootloader]
categories: 工具之术

---

这里总结一些移植过程中零碎的问题。

<!--more-->

## **解决 WARNING: Caches not enabled 提示**
启动过程中会有一条提示"WARNING: Caches not enabled"，警告未开启Cache。搜索这句提示，找到如下代码：

``` C
/*
 * Default implementation of enable_caches()
 * Real implementation should be in platform code
 */
__weak void enable_caches(void)
{
	puts("WARNING: Caches not enabled\n");
}
```

向上追踪函数的调用关系，可以找到是`initr_caches()`函数调用了`enable_caches()`。然而在`/board/samsung/smdk2440/smdk2440.c`中可以看到，`board_init()`函数会开启Cache的：

``` C
int board_init(void)
{   
    // 省略
    icache_enable();
    dcache_enable();
    
    return 0;
}
```

这就涉及到初始化顺序了，找到初始化序列：

``` C
init_fnc_t init_sequence_r[] = {
	// 省略
	initr_caches,
    // 省略
	board_init,	/* Setup chipselects */
    // 省略
};
```

可以很清楚的看到，`board_init()`函数会在`initr_caches()`函数后执行。这里有两种解决方案，一种是直接忽略`enable_caches()`；另一种是将`board_init()`中的缓存使能代码移到`enable_caches()`中，这里我采用了后一种方法。注释掉`board_init()`中的那两行代码，在`board/samsung/smdk2440/smdk2440.c`文件中加入以下代码：

``` C
void enable_caches(void)
{
    icache_enable();
    dcache_enable();
}
```

因为原来那个`enable_caches()`函数是使用`__weak`定义的，此处直接定义此函数即可，会覆盖掉原代码中的定义。修改完成后，编译运行，可以看到这个警告不再出现了。

## **配置环境变量保存位置** ##
U-boot中使用`saveenv`命令可以保存设置的环境变量，存储位置可以在NOR Flash、NAND Flash或EEPROM中，此处我选择保存在NOR Flash中。具体的存储地址和空间大小由`smdk2440.h`文件中这几个宏决定：
``` C
#define CONFIG_ENV_IS_IN_FLASH
/* Use Last Sector */
#define CONFIG_ENV_ADDR         (CONFIG_SYS_FLASH_BASE + 0x1F0000)
#define CONFIG_ENV_SIZE         0x10000    /* 64KB */
/* allow to overwrite serial and ethaddr */
#define CONFIG_ENV_OVERWRITE
```

`CONFIG_ENV_IS_IN_FLASH`表明使用NOR Flash，`CONFIG_ENV_ADDR`和`CONFIG_ENV_SIZE`即是起始地址和大小，这里选择了NOR Flash的最后一个扇区。设置的时候要注意不要把其他数据（如U-Boot本身的代码）给覆盖了就行。

## **配置启动参数** ##
为了实现上电自动引导Linux内核，需要正确设置以下3个环境变量：`bootcmd`、`bootargs`、`machid`。

`machid`为U-Boot传给Linux Kernel的机器ID，内核会根据这个ID选取对应的初始化文件对开发板进行初始化，在Linux Kernel的源代码`include/generated/mach-types.h`中可以找到Mini2440开发板的机器ID为1999(0x7CF)。所以将`machid`设置为7CF即可。

`bootcmd`为使用boot或bootd命令引导系统时实际调用的命令，这个根据实际情况设定，比如：
```shell
bootcmd=nand read 30008000 100000 300000;bootm 30008000
```
当设置了`bootcmd`后，启动时就会有读秒了，超过一段时间没有按键输入自动引导系统。环境变量`bootdelay`用于指定这段延迟时间的长度，单位是秒。

`bootargs`为启动参数，具体的设置值有待之后研究Linux内核时再来分析，目前使用的设置是：

``` shell
bootargs=noinitrd console=ttySAC0,115200 root=/dev/mtdblock3 rw rootfstype=jffs2 init=/linuxrc
```


> 移植好的U-Boot见[我的Github项目](https://github.com/g199209/U-Boot_201605_S3C2440)。

