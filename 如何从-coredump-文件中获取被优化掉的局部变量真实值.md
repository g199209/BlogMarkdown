title: 如何从 coredump 文件中获取被优化掉的局部变量真实值
toc: false
mathjax: false
fancybox: false
tags: [Debug, x86, Linux, GDB, Top]
date: 2019-12-24 20:56:28
permalink: coredump_optimized_value
categories: 编程之法

---

在 GCC  `-O3` 优化级别下，很多局部变量是会被优化掉的，此时只能通过人工分析反汇编代码来获取所需信息，而这么做的前提是保存下来的寄存器中的值是准确的。绝大部分情况下 coredump 是由于 segment fault 或 assert 触发的，segment fault 情况下 Kernel 保存下来的 registers 信息是准确的，GDB 中直接用 `info registers` 就可以看到。然而若是由 assert 触发，由于 assert 会进行多层函数调用后最终执行 `raise()`，错误现场的寄存器信息是不准确的，这时候就需要一些其他手段来解决此问题。下面用一个具体例子来说明此问题。

<!--more-->

测试程序代码：
```C++
volatile int final = 0;

int fun(int a) {
  int b = a + 100;
  final = b;
  if (b > 0) {
    assert(false);
    return 0;
  } else {
    std::cout << b;
    return 1;
  }
}

int main(int argc, char** argv) {
  int n = 0;
  while (true) {
    if (fun(rand()) == 1) {
      n++;
    }
    if (n > 100000) {
      break;
    }
  }
}
```

运行此程序肯定会发生 assert failed，我们用 gdb 来看下调用栈：

```shell
Program terminated with signal SIGABRT, Aborted.
#0  0x00007fac9f2f31f7 in raise () from /lib64/libc.so.6
gef> bt
#0  0x00007fac9f2f31f7 in raise () from /lib64/libc.so.6
#1  0x00007fac9f2f48e8 in abort () from /lib64/libc.so.6
#2  0x00007fac9f2ec266 in __assert_fail_base () from /lib64/libc.so.6
#3  0x00007fac9f2ec312 in __assert_fail () from /lib64/libc.so.6
#4  0x0000000000400d5e in fun (a=<optimized out>)
#5  main (argc=<optimized out>, argv=<optimized out>)
```

切换到 `fun()` 的栈帧：

```Sh e
gef> f 4
#4  0x0000000000400de0 in fun (a=<optimized out>)
245	    assert(false);
gef> p a
$1 = <optimized out>
gef> p b
$2 = <optimized out>
```

可以看到 `a` 与 `b` 都被优化掉了，到底是哪个值触发了 assert 就不能直接确定了。当然并不是就彻底没办法知道了，来看下 `fun()` 函数的反汇编：

```x86asm
gef> disassemble
Dump of assembler code for function main(int, char**):
   0x0000000000400d10 <+0>:	push   rbx
   0x0000000000400d11 <+1>:	mov    ebx,0x186a1
   0x0000000000400d16 <+6>:	nop    WORD PTR cs:[rax+rax*1+0x0]
   0x0000000000400d20 <+16>:	call   0x400c70 <rand@plt>
   0x0000000000400d25 <+21>:	lea    esi,[rax+0x64]
   0x0000000000400d28 <+24>:	test   esi,esi
   0x0000000000400d2a <+26>:	mov    DWORD PTR [rip+0x201570],esi        # 0x6022a0 <final>
   0x0000000000400d30 <+32>:	jg     0x400d45 <main(int, char**)+53>
   0x0000000000400d32 <+34>:	mov    edi,0x602080
   0x0000000000400d37 <+39>:	call   0x400cd0 <_ZNSolsEi@plt>
   0x0000000000400d3c <+44>:	sub    ebx,0x1
   0x0000000000400d3f <+47>:	jne    0x400d20 <main(int, char**)+16>
   0x0000000000400d41 <+49>:	xor    eax,eax
   0x0000000000400d43 <+51>:	pop    rbx
   0x0000000000400d44 <+52>:	ret
   0x0000000000400d45 <+53>:	mov    ecx,0x400fc6
   0x0000000000400d4a <+58>:	mov    edx,0xf5
   0x0000000000400d4f <+63>:	mov    esi,0x400f70
   0x0000000000400d54 <+68>:	mov    edi,0x400fc0
   0x0000000000400d59 <+73>:	call   0x400c80 <__assert_fail@plt>
```

在 `-O3` 优化下 `fun()` 直接被内联到 `main()` 里面了，不过这不影响基本分析，重点关注 `<+16>` ~ `<+32>` 这几行，这就对应 `fun()` 的前几行逻辑，`if (b > 0)` 是通过 `test` + `jg` 来实现的，`b` 的值此时就是 `%esi` 寄存器中的值。看下 gdb 分析出来的当前栈帧的寄存器值：

```x86asm
gef> info registers
rax            0x0                 0x0
rbx            0x186a1             0x186a1
rcx            0x7fac9f2f31f7      0x7fac9f2f31f7
rdx            0x6                 0x6
rsi            0x4cc5              0x4cc5
rdi            0x4cc5              0x4cc5
rbp            0x0                 0x0
rsp            0x7fff091e0410      0x7fff091e0410
r8             0x1                 0x1
r9             0xfeff092d63646b68  0xfeff092d63646b68
r10            0x8                 0x8
r11            0x206               0x206
r12            0x400daf            0x400daf
r13            0x7fff091e04f0      0x7fff091e04f0
r14            0x0                 0x0
r15            0x0                 0x0
rip            0x400d5e            0x400d5e
eflags         0x206               [ PF IF ]
cs             0x33                0x33
ss             0x2b                0x2b
ds             0x0                 0x0
es             0x0                 0x0
fs             0x0                 0x0
gs             0x0                 0x0
```

是不是其中 `%rsi` 的值就是我们需要的 `b` 了呢？非也！注意到 `<+68>` 行，在调用 `__assert_fail()` 前 `%esi` 又被重新赋值用于传递参数了，且由于 `%esi` 属于 caller save 的寄存器，在 `__assert_fail()` 内有可能会被再次改写。因此 **使用 GDB 分析 coredump 文件不同栈帧的 register 信息时，只有为数不多的几个 callee save 寄存器的值是可靠的，其他的都是不可靠的。** 那如何才能得到可靠的寄存器值呢？一般来说只有靠我们自己保存了，一个简单思路是只要在调用 `__assert_fail()` 前把所有寄存器的值保存到一个全局数组中就可以了。

在 `assert()` 前添加如下一段内联汇编代码即可实现此目的：

```C++
    __asm__ __volatile__("movq $0, %%r15;\n\t"
                         "movq %%rax, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rbx, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rcx, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rdx, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rsi, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rdi, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rbp, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%rsp, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r8, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r9, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r10, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r11, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r12, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r13, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r14, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                         "movq %%r15, (%0, %%r15, 8);\n\t"
                         "incq %%r15;\n\t"
                            :
                            : "r" (registers_data)
                            : "%r15");
```

再来看下此时的反汇编代码：

```x86asm
gef> disassemble
Dump of assembler code for function main(int, char**):
   0x0000000000400d10 <+0>:	push   r15
   0x0000000000400d12 <+2>:	push   rbx
   0x0000000000400d13 <+3>:	mov    ebx,0x186a1
   0x0000000000400d18 <+8>:	sub    rsp,0x8
   0x0000000000400d1c <+12>:	nop    DWORD PTR [rax+0x0]
   0x0000000000400d20 <+16>:	call   0x400c70 <rand@plt>
   0x0000000000400d25 <+21>:	lea    esi,[rax+0x64]
   0x0000000000400d28 <+24>:	test   esi,esi
   0x0000000000400d2a <+26>:	mov    DWORD PTR [rip+0x201570],esi        # 0x6022a0 <final>
   0x0000000000400d30 <+32>:	jg     0x400d4b <main(int, char**)+59>
   0x0000000000400d32 <+34>:	mov    edi,0x602080
   0x0000000000400d37 <+39>:	call   0x400cd0 <_ZNSolsEi@plt>
   0x0000000000400d3c <+44>:	sub    ebx,0x1
   0x0000000000400d3f <+47>:	jne    0x400d20 <main(int, char**)+16>
   0x0000000000400d41 <+49>:	add    rsp,0x8
   0x0000000000400d45 <+53>:	xor    eax,eax
   0x0000000000400d47 <+55>:	pop    rbx
   0x0000000000400d48 <+56>:	pop    r15
   0x0000000000400d4a <+58>:	ret
   0x0000000000400d4b <+59>:	mov    eax,0x6021a0
   0x0000000000400d50 <+64>:	mov    r15,0x0
   0x0000000000400d57 <+71>:	mov    QWORD PTR [rax+r15*8],rax
   0x0000000000400d5b <+75>:	inc    r15
   0x0000000000400d5e <+78>:	mov    QWORD PTR [rax+r15*8],rbx
   0x0000000000400d62 <+82>:	inc    r15
   0x0000000000400d65 <+85>:	mov    QWORD PTR [rax+r15*8],rcx
   0x0000000000400d69 <+89>:	inc    r15
   0x0000000000400d6c <+92>:	mov    QWORD PTR [rax+r15*8],rdx
   0x0000000000400d70 <+96>:	inc    r15
   0x0000000000400d73 <+99>:	mov    QWORD PTR [rax+r15*8],rsi
   0x0000000000400d77 <+103>:	inc    r15
   0x0000000000400d7a <+106>:	mov    QWORD PTR [rax+r15*8],rdi
   0x0000000000400d7e <+110>:	inc    r15
   0x0000000000400d81 <+113>:	mov    QWORD PTR [rax+r15*8],rbp
   0x0000000000400d85 <+117>:	inc    r15
   0x0000000000400d88 <+120>:	mov    QWORD PTR [rax+r15*8],rsp
   0x0000000000400d8c <+124>:	inc    r15
   0x0000000000400d8f <+127>:	mov    QWORD PTR [rax+r15*8],r8
   0x0000000000400d93 <+131>:	inc    r15
   0x0000000000400d96 <+134>:	mov    QWORD PTR [rax+r15*8],r9
   0x0000000000400d9a <+138>:	inc    r15
   0x0000000000400d9d <+141>:	mov    QWORD PTR [rax+r15*8],r10
   0x0000000000400da1 <+145>:	inc    r15
   0x0000000000400da4 <+148>:	mov    QWORD PTR [rax+r15*8],r11
   0x0000000000400da8 <+152>:	inc    r15
   0x0000000000400dab <+155>:	mov    QWORD PTR [rax+r15*8],r12
   0x0000000000400daf <+159>:	inc    r15
   0x0000000000400db2 <+162>:	mov    QWORD PTR [rax+r15*8],r13
   0x0000000000400db6 <+166>:	inc    r15
   0x0000000000400db9 <+169>:	mov    QWORD PTR [rax+r15*8],r14
   0x0000000000400dbd <+173>:	inc    r15
   0x0000000000400dc0 <+176>:	mov    QWORD PTR [rax+r15*8],r15
   0x0000000000400dc4 <+180>:	inc    r15
   0x0000000000400dc7 <+183>:	mov    ecx,0x4010c6
   0x0000000000400dcc <+188>:	mov    edx,0xf5
   0x0000000000400dd1 <+193>:	mov    esi,0x401070
   0x0000000000400dd6 <+198>:	mov    edi,0x4010c0
   0x0000000000400ddb <+203>:	call   0x400c80 <__assert_fail@plt>
```

`<+59>` ~ `<+180>` 行就是我们新加的逻辑，可以看到这段代码紧接在 `<+32>` 行之后，理论上分析的确是可以保存准确的寄存器信息。来看下实际效果：

```shell
gef> p registers_data
$1 = {0x6021a0, 0x186a1, 0x7f872ad260d4, 0x7f872ad260c8, 0x6b8b45cb, 0x7f872ad266e0, 0x0, 0x7ffd6d53a660, 0x7f872ad260c8, 0x7f872ad26140, 0x7ffd6d53a370, 0x7f872a9a38b0, 0x400e2f, 0x7ffd6d53a750, 0x0, 0xf, 0x0 <repeats 16 times>}
gef> p final
$2 = 0x6b8b45cb
```

`registers_data[4]` 与 `final` 的值完全相同，而从源代码和反汇编 `<+26>` 行可以看到，`final` 中保存的就是 `b` 的真实值。

--------------

> 参考资料：
> [Value optimized out. Reverse debugging to the rescue!](https://undo.io/resources/value-optimized-out-reverse-debugging-rescue/)