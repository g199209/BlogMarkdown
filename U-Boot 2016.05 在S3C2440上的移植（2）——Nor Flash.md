title: U-Boot 2016.05 在S3C2440上的移植（2）——NOR Flash
date: 2016-06-24 14:07:50
tags: [Bootloader]
categories: 编程之法

---

移植好基本框架能运行后就可以开始移植NOR Flash了，需要让U-Boot支持实际硬件平台中的NOR Flash。开发板上使用的NOR Flash型号为EN29LV160B，1024K * 16-bit。U-Boot中默认没有对此型号NOR Flash的支持，故需要添加相关代码。

U-Boot对NOR Flash的检测有两种方法，使用Legacy方法进行检测和使用CFI接口进行检测，下面将分别介绍这两种方法。其中使用CFI接口进行检测应该是更好的方法，优先使用此方法。

<!--more-->

## **更改NOR Flash扇区数宏定义**
无论使用哪种检测方法，都需要先修改NOR Flash扇区数的定义，这个定义在`include/configs/smdk2440.h`文件中：

``` C
#define CONFIG_SYS_MAX_FLASH_SECT   (35)
```

具体值根据实际使用的NOR Flash确定，在数据手册中都会有说明的，以EN29LV160B为例：
![](https://pic.gaomf.store/20160623105538.png)
可以算出其扇区数为35。

## **两种检测方式的切换**
NOR flash初始化函数`flash_init()`位于`drivers/mtd/cfi_flash.c`文件中，其中有这么一段代码：
``` C
if (!flash_detect_legacy(cfi_flash_bank_addr(i), i))
	flash_get_size(cfi_flash_bank_addr(i), i);
```

第一行调用`flash_detect_legacy()`函数使用Legacy方法进行检测，第二行调用`flash_get_size()`函数使用CFI方法进行检测。再来看`flash_detect_legacy()`函数的基本结构：

``` C
static int flash_detect_legacy(phys_addr_t base, int banknum)
{
	flash_info_t *info = &flash_info[banknum];

	if (board_flash_get_legacy(base, banknum, info)) {
		// 使用legacy方法进行检测
		return 1;
	}
	return 0; /* use CFI */
}
```

从这里可以看到，使用哪种检测方法由`board_flash_get_legacy()`函数的返回值确定，如果此函数返回1，使用Legacy方法；返回0则使用CFI方法。`board_flash_get_legacy()`函数是和具体硬件平台相关的，故它位于`board\samsung\smdk2440\smdk2440.c`文件中。从SMDK2410移植过来的默认版本为：
``` C
/*
 * Hardcoded flash setup:
 * Flash 0 is a non-CFI AMD AM29LV800BB flash.
 */
ulong board_flash_get_legacy(ulong base, int banknum, flash_info_t *info)
{
	info->portwidth = FLASH_CFI_16BIT;
	info->chipwidth = FLASH_CFI_BY16;
	info->interface = FLASH_CFI_X16;
	return 1;
}
```

如果要使用Legacy方法进行检测，直接使用这段代码就可以；如果要改为使用CFI方式进行检测，将返回值改为0即可。因为实际使用的EN29LV160B与AM29LV800BB的接口方式是一样的，故其他代码无需修改，不过亦可修改下函数注释，说明一下板子实际使用的NOR Flash型号等。


## **Legacy检测**

使用Legacy方法进行检测的基本步骤如下：
1. 依次使用AMD及Intel标准去读取Flash的厂商ID及器件ID；
2. 根据读取到的ID查找现有的器件列表，从而获得Flash更详细的信息。

从中可以看到，移植的关键就是要添加所使用的Flash芯片的配置文件。

EON公司的厂商宏定义已经存在，位于`include\flash.h`中，不需要更改，其定义如下：
``` C
#define EON_ALT_MANU	0x001C001C	/* EON     manuf. ID in D23..D16, D7..D0 */
```

EN29LV160B这个型号的NOR Flash不存在，需要添加相应的宏定义，可添加在EN29LV040A的宏定义下方：
``` C
/* EON */
#define EN29LV040A  0x004F
#define EN29LV160B  0x2249
```

具体宏定义的值可参考数据手册，其中有对Device ID的描述：
![](https://pic.gaomf.store/20160623100634.png)

最后找到`drivers/mtd/jedec_flash.c`文件，需要在`jedec_table`这个数组中添加此型号NOR Flash的结构体，此数组会在`jedec_flash_match()`函数中被调用，此函数代码如下：
``` C
/*-----------------------------------------------------------------------
 * match jedec ids against table. If a match is found, fill flash_info entry
 */
int jedec_flash_match(flash_info_t *info, ulong base)
{
	int ret = 0;
	int i;
	ulong mask = 0xFFFF;
	if (info->chipwidth == 1)
		mask = 0xFF;

	for (i = 0; i < ARRAY_SIZE(jedec_table); i++) {
		if ((jedec_table[i].mfr_id & mask) == (info->manufacturer_id & mask) &&
		    (jedec_table[i].dev_id & mask) == (info->device_id & mask)) {
			fill_info(info, &jedec_table[i], base);
			ret = 1;
			break;
		}
	}
	return ret;
}
```
可以看到，其中会将`jedec_table`成员的`mfr_id`、`dev_id`与读取到的`manufacturer_id`、`device_id`进行对比，若相同者调用`fill_info()`函数填充`info`结构体中的其它信息。

找到`jedec_table`的定义，这是一个数组，其中每个元素都是一个`amd_flash_info`类型的结构体：
``` C
struct amd_flash_info {
	const __u16 mfr_id;
	const __u16 dev_id;
	const char *name;
	const int DevSize;
	const int NumEraseRegions;
	const int CmdSet;
	const __u8 uaddr[4];		/* unlock addrs for 8, 16, 32, 64 */
	const ulong regions[6];
};
```

可仿照已有配置格式添加EN29LV160B的定义：
``` C
{
    .mfr_id     = (u16)EON_ALT_MANU,
    .dev_id     = EN29LV160B,
    .name       = "EON EN29LV160B",
    .uaddr      = {
        [1] = MTD_UADDR_0x0555_0x02AA /* x16 */
    },
    .DevSize    = SIZE_2MiB,
    .CmdSet     = CFI_CMDSET_AMD_LEGACY,
    .NumEraseRegions    = 4,
    .regions    = {
        ERASEINFO(0x04000, 1),
        ERASEINFO(0x02000, 2),
        ERASEINFO(0x08000, 1),
        ERASEINFO(0x10000, 31),
    }
},
```

下面对其中每一个字段的含义进行说明：

|字段|含义|说明|
|--|--|--|
|`.mfr_id`|制造商ID|EON公司的宏定义已经存在，直接使用|
|`.dev_id` |器件型号ID|之前已经添加了此宏定义|
|`.name`|器件名称字符串|主要用于显示|
|`.uaddr` |Unlock Address|发送命令时需要写入的地址，下面详细说明|
|`.DevSize` |器件容量|从已有的宏定义中选择一个正确的即可|
|`.CmdSet`|命令集|具体有哪些可能的选择还不太清楚，这里选择了AMD兼容命令集|
|`.NumEraseRegions`|扇区种类|指定有几类不同大小的扇区|
|`.regions`|扇区分配情况|下面详细说明|

`.uaddr` 这个字段是一个数组，分别用于存放8位、16位、32位访问时的地址，这里只需要写入16位访问时的地址即可。地址使用了枚举的形式，有以下一些选择：
``` C
/*
 * Unlock address sets for AMD command sets.
 * Intel command sets use the MTD_UADDR_UNNECESSARY.
 * Each identifier, except MTD_UADDR_UNNECESSARY, and
 * MTD_UADDR_NO_SUPPORT must be defined below in unlock_addrs[].
 * MTD_UADDR_NOT_SUPPORTED must be 0 so that structure
 * initialization need not require initializing all of the
 * unlock addresses for all bit widths.
 */
enum uaddr {
	MTD_UADDR_NOT_SUPPORTED = 0,	/* data width not supported */
	MTD_UADDR_0x0555_0x02AA,
	MTD_UADDR_0x0555_0x0AAA,
	MTD_UADDR_0x5555_0x2AAA,
	MTD_UADDR_0x0AAA_0x0555,
	MTD_UADDR_DONT_CARE,		/* Requires an arbitrary address */
	MTD_UADDR_UNNECESSARY,		/* Does not require any address */
};
```

从名称上就可以看出其意义，选用哪种根据数据手册确定；

`.NumEraseRegions`、`.regions`这两个参数指定了Flash芯片的扇区配置情况，其中`.NumEraseRegions`指定有几类不同大小的扇区，`.regions`指定每一类扇区的大小及个数。从`amd_flash_info`结构体的定义中可以看到，`.regions`是一个数组，且其大小默认设置为了6，如果扇区类型过多（一般不会出现），可修改此数组大小即可。


再来看一下`ERASEINFO`，这是一个宏，其宏定义为：
``` C
#define ERASEINFO(size,blocks) (size<<8)|(blocks-1)
```

其意义在于使用一个`ulong`类型的整数同时表示扇区大小及扇区个数。

`.NumEraseRegions`与`.regions`配合就可以确定每一个扇区的起始地址及器件的容量，下面就来分析下这一过程。这一部分代码位于`fill_info()`函数中，以下是`fill_info()`函数的精简改写版，仅保留了计算扇区部分最核心的代码：

``` C
static inline void fill_info(flash_info_t *info, const struct amd_flash_info *jedec_entry, ulong base)
{
	// 省略之前代码

	sect_cnt = 0;
	total_size = 0;
	for (i = 0; i < jedec_entry->NumEraseRegions; i++) {
		ulong erase_region_size = jedec_entry->regions[i] >> 8;
		ulong erase_region_count = (jedec_entry->regions[i] & 0xff) + 1;

		total_size += erase_region_size * erase_region_count;
		for (j = 0; j < erase_region_count; j++) {
			info->start[sect_cnt] = base;
			base += (erase_region_size);
			sect_cnt++;
		}
	}
	info->sector_count = sect_cnt;
	info->size = total_size;
}
```
第7~17行的循环遍历`NumEraseRegions`，依次处理每一类扇区；第8、9行获取`erase_region_size`及`erase_region_count`，这一过程就是`ERASEINFO`宏的逆过程；第12~16行依次循环处理每一个扇区，计算各扇区的起始地址，并计算总的扇区数。

分析清楚了扇区的计算方法后结构体中的`.regions`部分该如何编写就很清楚了，参考数据手册中的地址分配表即可写出所需的代码。

最后编译下载运行，可以看到已经正确识别出了Flash的容量。

## **CFI检测**
使用Legacy方法进行检测相当于要自己手动指定Flash的信息，不是很灵活，那是否可以自动检测Flash信息呢？答案是可以的，就是使用CFI方法进行检测。CFI是一种通用的接口规范，关于其详细说明可参考Cypress公司的[这份文档](http://www.spansion.com/Support/Application%20Notes/Quick_Guide_to_CFI_AN.pdf)。按上面的说明将`board_flash_get_legacy()`函数的返回值改为0即可启用CFI检测，之后不用做任何配置，直接编译运行即可，极为简单方便，而且可以自动适配不同型号的Flash。
在两块不同的板子上测试了CFI方法，结果表明，都能正确识别出Flash，这说明CFI方法具有很好的适应性。

## **测试**
要测试NOR Flash是否正常工作可使用以下方法：

使用`flinfo`命令可输出Flash的基本信息，若成功识别了Flash后，就会显示正确的信息。使用Legacy方式进行检测会输出类似这样的信息：
``` raw
GMF@2440 # flinfo

Bank # 1: EON EN29LV160B flash (16 x 16)  Size: 2 MB in 35 Sectors
  AMD Legacy command set, Manufacturer ID: 0x1C, Device ID: 0x2249
  Erase timeout: 30000 ms, write timeout: 100 ms

  Sector Start Addresses:
  00000000   RO   00004000   RO   00006000   RO   00008000   RO   00010000   RO 
  00020000   RO   00030000   RO   00040000   RO   00050000   RO   00060000   RO 
  00070000   RO   00080000        00090000        000A0000        000B0000      
  000C0000        000D0000        000E0000        000F0000        00100000      
  00110000        00120000        00130000        00140000        00150000      
  00160000        00170000        00180000        00190000        001A0000      
  001B0000        001C0000        001D0000        001E0000        001F0000      
```

使用CFI方式进行检测则会输出：
``` raw
GMF@2440 # flinfo

Bank # 1: CFI conformant flash (16 x 16)  Size: 2 MB in 35 Sectors
  AMD Standard command set, Manufacturer ID: 0x1C, Device ID: 0x2249
  Erase timeout: 16384 ms, write timeout: 1 ms

  Sector Start Addresses:
  00000000   RO   00004000   RO   00006000   RO   00008000   RO   00010000   RO 
  00020000   RO   00030000   RO   00040000   RO   00050000   RO   00060000   RO 
  00070000   RO   00080000        00090000        000A0000        000B0000      
  000C0000        000D0000        000E0000        000F0000        00100000      
  00110000        00120000        00130000        00140000        00150000      
  00160000        00170000        00180000        00190000        001A0000      
  001B0000        001C0000        001D0000        001E0000        001F0000      
```

----------

之后测试Flash的读写是否正常，可将内存中的一段数据写入Flash中再读取回来，比较二者是否相同即可。

先擦除最后一个扇区：
``` raw
GMF@2440 # erase 1:34      
Erase Flash Sectors 34-34 in Bank # 1 
. done
GMF@2440 # md.l 001F0000 20
001f0000: ffffffff ffffffff ffffffff ffffffff    ................
001f0010: ffffffff ffffffff ffffffff ffffffff    ................
001f0020: ffffffff ffffffff ffffffff ffffffff    ................
001f0030: ffffffff ffffffff ffffffff ffffffff    ................
001f0040: ffffffff ffffffff ffffffff ffffffff    ................
001f0050: ffffffff ffffffff ffffffff ffffffff    ................
001f0060: ffffffff ffffffff ffffffff ffffffff    ................
001f0070: ffffffff ffffffff ffffffff ffffffff    ................
```

读取内存开始部分的数据：
``` raw
GMF@2440 # md.l 30000000 20
30000000: 7effffdb defb5bbf dbf6fafa efbbffdf    ...~.[..........
30000010: 7f5f5e5a fcfebc5d feff7f7f d95edfff    Z^_.].........^.
30000020: def8feff fbf77d7e fbdf5e5f f3dabf6b    ....~}.._^..k...
30000030: ffde7eff debbfaf7 fbdfdfda fbbe7b7b    .~..........{{..
30000040: fffbd1bf 7fdffddf 4bffd7da febfdb5f    ...........K_...
30000050: 7f5b7aff fedffffa 5affeb7f d7df7fde    .z[........Z....
30000060: 5fff5afa 7fdeeeda fbbfebdb 5f4fddff    .Z._..........O_
30000070: fc7e7ffe 5bfdda7e ff7bdbfb b7fffbfa    ..~.~..[..{.....
```

将这些数据写入Flash的最后一个扇区，并回读写入的数据：
``` raw
GMF@2440 # cp.l 30000000 001F0000 20
Copy to Flash... 9....8....7....6....5....4....3....2....1..done
GMF@2440 # md.l 001F0000 20         
001f0000: 7effffdb defb5bbf dbf6fafa efbbffdf    ...~.[..........
001f0010: 7f5f5e5a fcfebc5d feff7f7f d95edfff    Z^_.].........^.
001f0020: def8feff fbf77d7e fbdf5e5f f3dabf6b    ....~}.._^..k...
001f0030: ffde7eff debbfaf7 fbdfdfda fbbe7b7b    .~..........{{..
001f0040: fffbd1bf 7fdffddf 4bffd7da febfdb5f    ...........K_...
001f0050: 7f5b7aff fedffffa 5affeb7f d7df7fde    .z[........Z....
001f0060: 5fff5afa 7fdeeeda fbbfebdb 5f4fddff    .Z._..........O_
001f0070: fc7e7ffe 5bfdda7e ff7bdbfb b7fffbfa    ..~.~..[..{.....
GMF@2440 # cmp.l 30000000 001F0000 20
Total of 32 word(s) were the same
```

可以看到，二者完全相同，这说明U-Boot已完全支持了目标板上的NOR Flash。

> 移植好的U-Boot见[我的Github项目](https://github.com/g199209/U-Boot_201605_S3C2440)。

