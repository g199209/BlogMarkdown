title: 在U-Boot中添加自定义命令以实现自动下载程序
date: 2016-06-26 21:06:56
tags: [Bootloader]
categories: 编程之法

---

U-Boot中通过NFS下载程序是一种很普遍的方式，然而下载程序的过程并不能只用一条命令实现。以下载到NOR Flash中为例，一般需要以下几步：
1. 通过NFS将文件下载到内存中；
2. 解除NOR Flash写保护；
3. 擦除NOR Flash；
4. 写入NOR Flash。

每一步都需要手动输入命令，十分麻烦，所以我们可以在U-Boot中添加一个自定义命令`download`，以实现一键全自动下载的目的。下面就来介绍一下实现方法。

<!--more-->

## **向U-Boot中添加命令** ##
以U-Boot 2016.05为例，其绝大部分命令都位于`cmd/`文件夹中，可以选择一个简单点的文件打开看看，就可以看到命令接口的基本结构如下：

```c
static int do_mycmd(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    // Do Something
}

U_BOOT_CMD(
    mycmd, 1, 1, do_mycmd,
    "short description",
    "help"
);
```

其中`do_mycmd()`函数就是命令的执行函数，它的名字可以是任意的，只是按照U-Boot惯例一般就叫做这种形式。此函数的`cmdfp`和`flag`两个参数是由U-Boot系统传入的和命令相关的一些信息，一般用不到，重点是后两个参数。`argc`是参数个数，至少为1，表示命令本身；`argv`就是具体传入的参数字符串数组。

这个一般的函数是如何和命令关联起来的呢？靠的就是`U_BOOT_CMD`这个宏。这里不详细分析此宏的实现机理，仅从应用的角度说明一下。`U_BOOT_CMD`的命令格式为：

```c
U_BOOT_CMD(name,maxargs,rep,cmd,usage,help)
```
各参数的含义为：

|参数|含义|
|-----|------|
|`name` | 命令的名称，此处直接输入即可，不要用字符串`"xxx"`的形式|
|`maxargs` |命令的最大参数个数，至少为1，表示命令本身|
|`rep` |是否自动重复（为1的话下次直接按Enter键会重复执行此命令）|
|`cmd` |命令对应的响应函数，即之前的`do_mycmd()`函数，直接使用函数名|
|`usage` |简短的使用说明（字符串）|
|`help` |输入`help`后显示的较详细的帮助文档（字符串）|

按以上格式新建一个C源文件后，将其加入Makefile中编译即可。这个文件可以放在任何地方，不过`cmd/`文件夹中存放的是通用的命令，我们自己新加入的命令最好不要放在里面，而是放在`board/`中板子相关的文件夹里，比如`board/samsung/smdk2440/`。

如果需要灵活控制是否添加此命令，可加入条件编译，仿照U-Boot本身的做法定义以下宏：

```c
#define CONFIG_CMD_MYCMD
```

这个定义可以放在板子的头文件中，也可加入defconfig文件中。之后在Makefile文件中加入条件编译即可：

```makefile
obj-$(CONFIG_CMD_MYCMD) += mycmd.o
```

## **运行特定命令**
要实现自动下载，需要使用一个命令代替一系列命令，这就要求能够在程序中自动运行特定命令。U-Boot提供了一个方便的接口函数来实现这一目的：

```c
/*
 * Run a command using the selected parser.
 *
 * @param cmd	Command to run
 * @param flag	Execution flags (CMD_FLAG_...)
 * @return 0 on success, or != 0 on error.
 */
int run_command(const char *cmd, int flag);
```

只需调用此函数即可运行特定的命令。

## **自动下载程序** ##
最后给出完整版的自动下载程序的实现代码：

```c
#include <common.h>
#include <command.h>

static int do_download(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    int i;

    if(argc == 1)
        printf("param:\nu : U-Boot;\nl : Linux;\nf : File System.\n");

    const char * const cmd_uboot[5] = {
        "nfs 30000000 /home/gmf/nfs/u-boot.bin",
        "protect off all",
        "erase 0 +$filesize",
        "cp.b 30000000 0 $filesize",
        "reset",
    };

    const char * const cmd_linux[4] = {
        "nfs 30008000 /home/gmf/nfs/uImage",
        "nand erase 60000 300000",
        "nand write.jffs2 30008000 60000 300000",
        "bootm 30008000",
    };

    const char * const cmd_fs[4] = {
        "nfs 32000000 /home/gmf/nfs/ramdisk.gz",
        "nand erase 560000 $filesize",
        "nand write.jffs2 32000000 560000 $filesize",
        "bootd",
    };

    switch(*argv[1]) {
    case 'u':
    case 'U':
        for(i = 0; i < 5; i++)
        {
            printf("\n##########\n");
            printf(cmd_uboot[i]);
            printf("\n##########\n");
            run_command(cmd_uboot[i], 0);
        }
        break;

    case 'l':
    case 'L':
        for(i = 0; i < 4; i++)
        {
            printf("\n##########\n");
            printf(cmd_linux[i]);
            printf("\n##########\n");
            run_command(cmd_linux[i], 0);
        }
        break;

    case 'f':
    case 'F':
        for(i = 0; i < 4; i++)
        {
            printf("\n##########\n");
            printf(cmd_fs[i]);
            printf("\n##########\n");
            run_command(cmd_fs[i], 0);
        }
        break;
    }
    return 0;
}


U_BOOT_CMD(
    download,   2,  1,  do_download,
    "Download File (Uboot, Linux or FS)",
    " - Download File:\nu : U-Boot;\nl : Linux;\nf : File System.\n"
);
```

此处实现了自动下载`u-boot.bin`文件、`uImage`文件和`ramdisk.gz`文件，分别输入`download u`、`download l`和`download f`即可。命令执行序列位于`cmd_uboot`、`cmd_linux`及`cmd_fs`数组中。此代码结构很好进行扩展，如要加入新的选项，仿照目前的结构添加即可。
