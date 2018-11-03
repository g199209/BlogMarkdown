title: memcpy中的内存读写优化问题
permalink: Memcpy_Optimization
toc: false
mathjax: false
fancybox: false
tags: [Performance, Top]
categories: 软件之道
date: 2017-08-15 23:00:00

---

`memcpy`作为一个很简单的库函数，实现了内存的拷贝。不过这个函数功能虽然简单，要实现一个高效的`memcpy`函数还是很有难度的，这里对其优化问题做一简单讨论。

<!--more-->

### 基本实现

最简单的`memcpy`函数实现如下：

```c
void * memcpy1(void * dest, const void * src, size_t n) {
    char * psrc, * pdest;
    psrc = (char *)src; pdest = (char *)dest; 
    for (size_t i = 0; i < n; i++) {
        *pdest = *psrc;
        pdest++; psrc++;
    }
    return dest;
}
```

`memcpy1`函数对应的汇编代码如下：（使用gcc 4.8.4 -O1优化级别得到，-O2优化级别得到的代码差不多，-O3优化级别得到的代码会复杂很多，极为难以理解……）

```x86asm
memcpy1:
	movq		%rdi, %rax
	testq		%rdx, %rdx
	je			.L2
	movl		$0, %ecx
.L3:
	movzbl		(%rsi, %rcx), %r8d
	movb		%r8b, (%rax, %rcx)
	addq		$1, %rcx
	cmpq		%rdx, %rcx
	jne			.L3
.L2:
	ret
```

从中可以看到，在`.L3`主循环中，每次使用零扩展传送（`MOVZ`）从源地址读取一个字节的内容，然后将其写入目的地址。

### 内存读写优化

`memcpy1`实现的功能是完全正确的，然而，这段代码效率也是很低效的，其中一个重要原因就是没考虑到实际内存读写过程。目前大部分主流CPU架构中，数据总线的位宽是确定的（大部分是32位或64位），在进行数据传输时，只能以此为基本单位进行传输，且每次读写数据的地址也最好是按照总线位宽对齐的（可能会有其他更严格的对其要求，如x86平台上SSE寄存器要求128位对齐），如果需要的数据不满足此要求时，读写效率会降低，如会进行多次读写等。

根据以上分析，我们可以每次都以数据总线位宽为单位进行读写，以此优化上述代码：

```c
void * memcpy2(void * dest, const void * src, size_t n) {
    long long * psrc, * pdest;
    psrc = (long long *)src; pdest = (long long *)dest;
    n /= 8;
    for (size_t i = 0; i < n; i++) {
        *pdest = *psrc;
        pdest++; psrc++;
    }
    return dest;
}
```

需要注意的是，以上代码假设`n`是8的整数倍。对应的汇编代码如下：

```x86asm
memcpy2:
	movq		%rdi, %rax
	testq		%rdx, %rdx
	je			.L2
	movl		$0, %ecx
.L3:
	movq		(%rsi, %rcx, 8), %r8
	movq		%r8, (%rax, %rcx, 8)
	addq		$1, %rcx
	cmpq		%rdx, %rcx
	jne			.L3
.L2:
	ret
```

对比两段反汇编代码可以看到，理论上说后一段代码的执行速度会是前一段的8倍，实际测试表明后一段的确快很多，数据长度不同时加速比不一样，不过没有达到8倍，其原因应该和Cache有关，现代主流CPU中均有Cache，所以第一段代码也不是每次都会去读取内存的，之前读到的数据会存在于Cache中，后续访问的速度会比读取内存快很多。

`memcpy2`为了说明主要问题，假设`n`是8的整数倍，然而对于实际使用的函数而言，显然不能做这个假设，此时的处理方法是最后来复制`n % 8`个字节，代码如下：

```c
void * memcpy3(void * dest, const void * src, size_t n) {
	long long * psrc, *pdest;
	psrc = (long long *)src; pdest = (long long *)dest;
	n /= 8;
	size_t m = n % 8;
	for (size_t i = 0; i < n; i++) {
		*pdest = *psrc;
		pdest++; psrc++;
	}
	char * psrc2, *pdest2;
	psrc2 = (char *)psrc; pdest2 = (char *)pdest;
	for (size_t i = 0; i < m; i++) {
		*pdest = *psrc;
		pdest++; psrc++;
	}
	return dest;
}
```

此时的汇编代码类似`memcpy1`与`memcpy2`的混合体，此处从略。

另一个问题是，之前也提到，内存的读写最好以对齐的方式进行，不过我实际测试了下对于`memcpy3`来说，`src`与`dest`是不是对齐的影响并不大，特别是数据量比较大的时候。这个应该也与Cache机制有关。当然，这是对于x86架构来说的，对于ARM架构而言，地址是强制要求对齐的，好像不对齐的话会出现错误。

### 更多优化

实际的标准库`memcpy`函数的实现要复杂的多，主要还要考虑以下一些问题：

- 使用SIMD指令集，如SSE2，SSSE3，AVX2等；
- 考虑缓存预取问题（Cache Prefetch）；
- 考虑地址对齐问题；

相关讨论可以参考这个知乎帖子：

> [怎样写出一个更快的 memset/memcpy？](https://www.zhihu.com/question/35172305)

关于预取（Prefetch）技术的优化技巧，可参考ARM公司的这篇文章：

> [Using Block Prefetch for Optimized Memory Performance](http://files.rsdn.ru/23380/AMD_block_prefetch_paper.pdf)

最后，实际最新版的Glibc 2.26中`memcpy`的代码是用汇编写的，对于x86_64架构使用了SSSE3指令集优化，代码长达3000余行……列在最后以表达下对技术大佬的膜拜……

> [memcpy-ssse3.S](https://sourceware.org/git/gitweb.cgi?p=glibc.git;a=blob;f=sysdeps/x86_64/multiarch/memcpy-ssse3.S;h=f3ea52a46cc808abca920621120a3eff69250137;hb=refs/heads/release/2.26/master)





