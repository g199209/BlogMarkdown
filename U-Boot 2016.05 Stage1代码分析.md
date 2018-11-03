title: U-Boot 2016.05 Stage1代码分析
date: 2016-07-15 17:14:24
tags: [Bootloader, Top]
categories: 软件之道
mathjax: false
fancybox: false

---

主流Bootloader的启动过程均可分为Stage1及Stage2两部分，一般来说，Stage1完成最基本的初始化操作、搬移代码及建立C语言运行环境；Stage2完成剩余的初始化操作。故Stage1一般使用汇编完成，Stage2一般使用C语言完成，不过这只是很粗略的情况，实际上Stage1中也会使用C语言代码。

下面以之前针对S3C2440移植好的U-Boot 2016.05为例，分析U-Boot中Stage1的代码。

<!--more-->

## **程序入口点分析** ##

要分析启动流程首先要找到程序入口点，也就是最先运行的代码位于何处，这是由链接过程决定的，故找到链接脚本文件`u-boot.lds`：

```arm
ENTRY(_start)
SECTIONS
{
 . = 0x00000000;
 . = ALIGN(4);
 .text :
 {
  *(.__image_copy_start)
  *(.vectors)
  arch/arm/cpu/arm920t/start.o (.text*)
  *(.text*)
 }
;......省略......
}
```

可以看到，位于映像文件最前面的依次是`.__image_copy_start`、`.vectors`及`arch/arm/cpu/arm920t/start.o`文件。

----------

`.__image_copy_start`位于映像文件的起始地址处，它在文件`arch/arm/lib/sections.c`中被定义：

```c
char __bss_start[0] __attribute__((section(".__bss_start")));
char __bss_end[0] __attribute__((section(".__bss_end")));
char __image_copy_start[0] __attribute__((section(".__image_copy_start")));
char __image_copy_end[0] __attribute__((section(".__image_copy_end")));
```

这个变量的用处应该是用来进行代码定位的，也就是这个变量的地址就代表映像文件的起始地址，并不占用真实的存储空间，所以此处使用了一个0长度的数组，其他变量的用处也是一样的。U-Boot作者对此的说明是：

> We need a 0-byte-size type for these symbols, and the compiler does not allow defining objects of C type 'void'. Using an empty struct is allowed by the compiler, but causes gcc versions 4.4 and below to complain about aliasing. Therefore we use the next best thing: zero-sized arrays, which are both 0-byte-size and exempt from aliasing warnings.

----------

`.vectors`是中断向量表，对于ARM920t来说，使用的是位于`arch/arm/lib/vectors.S`中的通用向量表：

```arm
.globl _start
    .section ".vectors", "ax"

_start:
    b   reset
    ldr pc, _undefined_instruction
    ldr pc, _software_interrupt
    ldr pc, _prefetch_abort
    ldr pc, _data_abort
    ldr pc, _not_used
    ldr pc, _irq
    ldr pc, _fiq

;......省略......
```

可以看到，复位后会首先执行`b reset`这条指令跳转至`reset`处执行，而`reset`标号正是位于`arch/arm/cpu/arm920t/start.S`文件中：

```arm
    .globl  reset
reset:
;   ......省略......
```

所以一般都说，代码执行开始于`start.S`文件，这也就是程序的入口点。下面就从这个文件开始分析。

## **start.S** ##

`start.S`的执行流程见下图：
![](http://gmf.shengnengjin.cn/U-Boot%20Stage1_1.svg)

`lowlevel_init`是平台相关的，故它位于`board/samsung/smdk2440/lowlevel_init.S`中。此处需要完成内存控制器的初始化工作，以便之后能正常使用SDRAM。对于S3C2440来说，就是初始化了`BWSCON`、`BANKCONn`、`REFRESH`、`BANKSIZE`、`MRSRB`这些寄存器。

`_main`是平台无关的，位于`arch/arm/lib/crt0.S`中，crt的意思就是C-runtime startup Code，此文件的目的就是建立C语言运行环境，以便之后跳转到Stage2，下面就来分析此文件。

## **crt0.S** ##

`crt0.S`文件开头的注释中说明了`_main`的执行序列，摘录如下：

> This file handles the target-independent stages of the U-Boot start-up where a C runtime environment is needed. Its entry point is _main and is branched into from the target's start.S file.
>
> _main execution sequence is:
>
> 1. Set up initial environment for calling board_init_f().
> This environment only provides a stack and a place to store the GD ('global data') structure, both located in some readily available RAM (SRAM, locked cache...). In this context, VARIABLE global data, initialized or not (BSS), are UNAVAILABLE; only CONSTANT initialized data are available. GD should be zeroed before board_init_f() is called.
>
> 2. Call board_init_f().
> This function prepares the hardware for execution from system RAM (DRAM, DDR...) As system RAM may not be available yet, , board_init_f() must use the current GD to store any data which must be passed on to later stages. These data include the relocation destination, the future stack, and the future GD location.
>
> 3. Set up intermediate environment where the stack and GD are the ones allocated by board_init_f() in system RAM, but BSS and initialized non-const data are still not available.
>
> 4. For U-Boot proper (not SPL), call relocate_code().
> This function relocates U-Boot from its current location into the relocation destination computed by board_init_f().
>
> 5. Set up final environment for calling board_init_r().
> This environment has BSS (initialized to 0), initialized non-const data (initialized to their intended value), and stack in system RAM. GD has retained values set by board_init_f().
>
> 6. For U-Boot proper (not SPL), some CPUs have some work left to do at this point regarding memory, so call c_runtime_cpu_setup.
>
> 7. Branch to board_init_r().

其程序流程图如下：

![](http://gmf.shengnengjin.cn/U-Boot%20Stage1_2.svg)

全局变量`gd`是一个`struct global_data`类型的结构体，此结构体的定义位于`include/asm-generic/global_data.h`文件中。此结构体保存了初始化阶段需要用到的各种全局变量，大约有100个左右，相关说明如下：

> The following data structure is placed in some memory which is available very early after boot (like DPRAM on MPC8xx/MPC82xx, or some locked parts of the data cache) to allow for a minimum set of global variables during system initialization (until we have set up the memory controller so that we can use RAM).
> Keep it **SMALL** and remember to set GENERATED_GBL_DATA_SIZE > sizeof(gd_t)
> Each architecture has its own private fields. For now all are private

下面具体说明每一步的作用：

### Step 1.1 **初始化SP寄存器**
```arm
ldr sp, =(CONFIG_SYS_INIT_SP_ADDR)
bic sp, sp, #7
```

`CONFIG_SYS_INIT_SP_ADDR`在板子的配置文件`include/configs/smdk2440.h`中定义：

```c
#define CONFIG_SYS_INIT_SP_ADDR	(CONFIG_SYS_SDRAM_BASE + 0x1000 - GENERATED_GBL_DATA_SIZE)
```

`GENERATED_GBL_DATA_SIZE`是`struct global_data`按16字节对齐后的大小。可以看到，这里是使用了SDRAM开头4K部分来作为临时堆栈。

### Step 1.2 **board_init_f_alloc_reserve**

```arm
mov r0, sp
bl  board_init_f_alloc_reserve
mov sp, r0
```

```c
ulong board_init_f_alloc_reserve(ulong top)
{
	/* reserve GD (rounded up to a multiple of 16 bytes) */
	top = rounddown(top-sizeof(struct global_data), 16);
	return top;
}
```

为`GD`结构体分配保留空间，16字节对齐。结构体的存储方式是从低地址向高地址生长，故此处返回的就是`gd`的基地址。

### Step 1.3 **board_init_f_init_reserve**

```arm
mov r9, r0
bl  board_init_f_init_reserve
```

简化版的函数实现：
```c
void board_init_f_init_reserve(ulong base)
{
	struct global_data *gd_ptr;
	int *ptr;

	gd_ptr = (struct global_data *)base;
	/* zero the area */
	for (ptr = (int *)gd_ptr; ptr < (int *)(gd_ptr + 1); )
		*ptr++ = 0;
}
```

将`gd`的基地址放到`r9`中保存起来，之后将刚刚分配的保留空间清零完成初始化。

### Step 2 **board_init_f**
```arm
mov r0, #0
bl  board_init_f
```

```c
void board_init_f(ulong boot_flags)
{
	gd->flags = boot_flags;
	gd->have_console = 0;

	if (initcall_run_list(init_sequence_f))
		hang();
}
```

`board_init_f()`函数主要是调用`initcall_run_list(init_sequence_f)`函数，而此函数的作用就是依次执行`init_sequence_f`中的各函数指针，去除错误处理、Debug信息输出等部分后，简化版的函数体很简单：

```c
int initcall_run_list(const init_fnc_t init_sequence[])
{
	const init_fnc_t *init_fnc_ptr;

	for (init_fnc_ptr = init_sequence; *init_fnc_ptr; ++init_fnc_ptr) {
		(*init_fnc_ptr)();
	}
	return 0;
}
```

下面来看一下`init_fnc_t init_sequence_f[]`这个数组，`init_fnc_t`是一个是用`typedef`定义的函数指针类型：
```c
typedef int (*init_fnc_t)(void);
```

`init_sequence_f[]`数组中包含的就是需要执行的初始化函数列表，这里面的函数很多，**这些函数的核心目的就是填充`gd`结构体**。其中比较重要的有下面这些(**省略了其中大部分函数**)：
```c
static init_fnc_t init_sequence_f[] = {
    board_early_init_f,
    serial_init,        /* serial communications setup */
    console_init_f,     /* stage 1 init of console */
    display_options,    /* say that we are here */
    print_cpuinfo,      /* display cpu info (and speed) */
    dram_init,          /* configure available RAM banks */
    show_dram_config,
    setup_dest_addr,
    reserve_xxxxxx,
    setup_reloc,
    // jump_to_copy,
}
```

U-Boot自带的SMDK2410配置中，PLL时钟配置是在`board_early_init_f`中完成的，不过在针对2440移植时将此部分代码移到了start.S文件中，故`board_early_init_f`中只包含了GPIO端口初始化；

`serial_init`及`console_init_f`完成串口终端的初始化，在此之后串口终端才会有输出。

`display_options`和`print_cpuinfo`就是U-Boot运行时串口最先输出的那段版本信息。

`dram_init`：对于2440来说，因为之前已经初始化好了DRAM，此处只是将DRAM大小写入`gd`中，在`show_dram_config`中将SDRAM大小信息输出至控制台。

`setup_dest_addr`中根据RAM的大小计算了重定位开始地址，将RAM顶端地址存放在`gd->relocaddr = gd->ram_top`中。

`reserve_xxxxxx`依次计算重定位每一部分所需的RAM空间，并从`gd->relocaddr`中减去这部分空间，其中`reserve_uboot`是计算重定位U-Boot本身（.text + .bss）的地址；`reserve_global_data`是计算重定位`gd`的地址。从这里可以看出，**重定位后的代码和数据位于RAM顶端**。

`setup_reloc`向`gd`中填入了重定位地址信息，并完成了`gd`本身的重定位，**`gd->reloc_off`就是在这里计算出来的，代表重定位地址偏移量**：
```c
static int setup_reloc(void) {
    gd->reloc_off = gd->relocaddr - CONFIG_SYS_TEXT_BASE;
    memcpy(gd->new_gd, (char *)gd, sizeof(gd_t));
    return 0;
}
```

对于其他平台来说，最后还有一步`jump_to_copy`，这里会实现代码跳转，也就是不会再返回此函数，不过对于ARM平台来说，是没有这一步的，执行完`init_sequence_f[]`中的函数后会返回到`board_init_f()`中，最后回到`_main`中继续执行后面的代码。

### Step 3 **设置中间变量**

```arm
    ldr sp, [r9, #GD_START_ADDR_SP] /* sp = gd->start_addr_sp */
    bic sp, sp, #7                  /* 8-byte alignment for ABI compliance */
    ldr r9, [r9, #GD_BD]            /* r9 = gd->bd */
    sub r9, r9, #GD_SIZE            /* new GD is below bd */
    adr lr, here
    ldr r0, [r9, #GD_RELOC_OFF]     /* r0 = gd->reloc_off */
    add lr, lr, r0
    ldr r0, [r9, #GD_RELOCADDR]     /* r0 = gd->relocaddr */
    b   relocate_code
here:
    bl  relocate_vectors
```

这里面用到的`gd`中的值都是在`board_early_init_f()`中设置好的，其中有几句很重要的语句：

```arm
adr lr, here
ldr r0, [r9, #GD_RELOC_OFF]     /* r0 = gd->reloc_off */
add lr, lr, r0
```

`here`就是下面那个标号，此处将`here`的地址加上`gd->reloc_off`这个偏移量后存放到`lr`寄存器中，之后调用`b   relocate_code`命令返回后就会返回到`lr`寄存器中的地址处。**这就实现了将代码拷贝到SDRAM后从新的重定位地址处接着运行的目的。**这也就是源码中这一段注释的含义：

> Set up intermediate environment (new sp and gd) and call relocate_code(addr_moni). **Trick here is that we'll return 'here' but relocated.**

设置好这些寄存器后调用`relocate_code`实现代码重定位。

### Step 4 **relocate_code**

位于`/arch/arm/lib/relocate.S`中，使用汇编完成，简化版的代码如下：

```arm
ENTRY(relocate_code)
    ldr r1, =__image_copy_start /* r1 <- SRC &__image_copy_start */
    ldr r2, =__image_copy_end   /* r2 <- SRC &__image_copy_end */

copy_loop:
    ldmia   r1!, {r10-r11}      /* copy from source address [r1]    */
    stmia   r0!, {r10-r11}      /* copy to   target address [r0]    */
    cmp r1, r2          /* until source end address [r2]    */
    blo copy_loop

    bx  lr
ENDPROC(relocate_code)
```

在这段代码中，使用`ldmia`及`stmia`指令，每次使用`r10`及`r11`两个寄存器进行复制，**实现了将`__image_copy_start`及`__image_copy_end`中的代码复制到`gd->reloc_off`处的目的**。

这里省略了`.rel.dyn`部分的重定位代码，这部分代码和以上代码差不多，只是还做了些额外处理。

最后调用`bx lr`返回，此时会进行状态切换后接着运行行下面的程序，不过此时运行的程序已是位于重定位的地址处了。

之后执行的是`bl  relocate_vectors`，这个实现的是中断向量表的重定位，具体就不展开了。在此之后还调用了`c_runtime_cpu_setup`，不过对于2440来说，这里没有做任何事情，可以跳过。

### Step 5 **清零BSS段**

```arm
    ldr r0, =__bss_start    /* this is auto-relocated! */
    ldr r1, =__bss_end      /* this is auto-relocated! */
    mov r2, #0x00000000     /* prepare zero to clear BSS */

clbss_l:
    cmp r0, r1              /* while not at end of BSS */
    strlo   r2, [r0]        /* clear 32-bit BSS word */
    addlo   r0, r0, #4      /* move to next */
    blo clbss_l
```

这一步建立C语言运行环境，实际上也就是清零BSS段，代码也很好懂，根据`__bss_start`及`__bss_end`的地址循环写入0即可。只是这有一个问题，注释中有说明：“this is auto-relocated!”，这说明`__bss_start`及`__bss_end`的地址已经是重定位后的新地址了，这是如何实现的呢？`__bss_start`及`__bss_end`的定义见本文最前面`arch/arm/lib/sections.c`中的代码，是两个长度为0的数组，估计是在`relocate_code`搬运代码过程中已经搬运了这两个0长度的代码了？这个问题还需要之后来仔细思考下。

### Step 6 **board_init_r**

```arm
bl coloured_LED_init
bl red_led_on
/* call board_init_r(gd_t *id, ulong dest_addr) */
mov r0, r9                  /* gd_t */
ldr r1, [r9, #GD_RELOCADDR] /* dest_addr */
ldr pc, =board_init_r   /* this is auto-relocated! */
```

在调用`board_init_r`前还调用了两个函数：`coloured_LED_init`及`red_led_on`，不过在SMDK2410的BSP文件中没有实现这两个函数，目前使用的是`common/board_f.c`中的空函数，相关函数还有下面这些：

```c
/************************************************************************
 * Coloured LED functionality
 ************************************************************************
 * May be supplied by boards if desired
 */
__weak void coloured_LED_init(void) {}
__weak void red_led_on(void) {}
__weak void red_led_off(void) {}
__weak void green_led_on(void) {}
__weak void green_led_off(void) {}
__weak void yellow_led_on(void) {}
__weak void yellow_led_off(void) {}
__weak void blue_led_on(void) {}
__weak void blue_led_off(void) {}
```

----------

最后来看看`board_init_r()`函数：

```c
void board_init_r(gd_t *new_gd, ulong dest_addr)
{
	if (initcall_run_list(init_sequence_r))
		hang();
	/* NOTREACHED - run_main_loop() does not return */
	hang();
}
```

`initcall_run_list`函数在上面"2.board_init_f"小节中已经分析过了，就是依次执行传入数组中的各函数，此处传入了`init_sequence_r`，这就是Stage2的初始化函数列表。

至此，Stage1结束，开始执行Stage2的初始化流程。
