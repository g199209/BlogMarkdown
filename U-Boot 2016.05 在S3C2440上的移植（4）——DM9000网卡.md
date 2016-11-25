title: U-Boot 2016.05 在S3C2440上的移植（4）——DM9000网卡
date: 2016-06-25 21:02:15
tags: [Bootloader]
categories: 嵌入式

---

SMDK2410中默认使用的网卡为CS8900，而实际开发板上使用的网卡为DM9000，故需要进行替换。U-Boot中已经提供了现成的DM9000驱动程序，所以这部分移植相对比较简单，只需要更改下配置文件即可。

<!--more-->

打开`include/configs/smdk2440.h`文件，目前网卡部分的宏定义为：
``` C
#define CONFIG_CS8900       /* we have a CS8900 on-board */
#define CONFIG_CS8900_BASE  0x19000300
#define CONFIG_CS8900_BUS16 /* the Linux driver does accesses as shorts */
```

将其修改为：
``` C
#define CONFIG_DRIVER_DM9000
#define CONFIG_DM9000_NO_SROM       //not use the dm9000 eeprom
#define CONFIG_NET_RANDOM_ETHADDR   //set the ethaddr
#define CONFIG_LIB_RAND             //random_ethadd need rand function
#define CONFIG_DM9000_BASE          0x20000000
#define DM9000_IO                   CONFIG_DM9000_BASE      
#define DM9000_DATA                 (CONFIG_DM9000_BASE + 4 ) //data address
```

网卡的初始化入口位于`board/samsung/smdk2440/smdk2440.c`文件中，在其中找到`board_eth_init()`函数，仿照现有的CS8900网卡的形式添加DM9000的初始化代码即可：
``` C
int board_eth_init(bd_t *bis)
{
    int rc = 0;
#ifdef CONFIG_CS8900
    rc = cs8900_initialize(0, CONFIG_CS8900_BASE);
#endif

#ifdef CONFIG_DM9000
    rc = dm9000_initialize(bis);
#endif

    return rc;
}
```

最后，需要正确设置器件的IP地址、子网掩码等信息，相关宏定义位于`include/configs/smdk2440.h`文件中：

``` C
#define CONFIG_NETMASK      255.255.255.0
#define CONFIG_IPADDR       192.168.1.77
#define CONFIG_SERVERIP     192.168.1.11
```

这里没有指定MAC地址，会使用一个随机MAC地址，而且运行时会提示下面这个警告：

``` shell
Warning: dm9000 (eth0) using random MAC address - 4a:0a:ab:7c:96:2f
```

解决办法就是使用使用环境变量的方法指定一个MAC地址，具体操作方法见：[U-Boot中IP地址设置方法](/2016/05/30/U-Boot%E4%B8%ADIP%E5%9C%B0%E5%9D%80%E8%AE%BE%E7%BD%AE%E6%96%B9%E6%B3%95/)

上述修改完成后编译运行，使用`ping`命令检查网络是否可正常使用：

``` shell
GMF@2440 # ping 192.168.1.11
dm9000 i/o: 0x20000000, id: 0x90000a46 
DM9000: running in 16 bit mode
MAC: 4a:0a:ab:7c:96:2f
could not establish link
Using dm9000 device
host 192.168.1.11 is alive
```

这里ping的是主机虚拟机中的Ubuntu系统，可以看到连接是没有问题的。不过有一个需要注意的问题是，目前连上网线后在电脑端并不会显示已连接，而是显示：“网络电缆被拔出”，也就是和没插网线时是一样的。不过这不影响正常使用，这个问题的具体原因有待之后再来分析。

另外，此处的移植针对的是DM9000网卡，目前有些开发板使用的是DM9000A，这是两个不同的芯片，驱动程序是不兼容的，针对DM9000A还要修改DM9000的驱动才能正常使用。这个问题也留待将来再来分析解决。

> 移植好的U-Boot见[我的Github项目](https://github.com/g199209/U-Boot_201605_S3C2440)。
