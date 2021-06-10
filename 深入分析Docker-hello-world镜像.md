title: 深入分析Docker hello-world镜像
toc: false
mathjax: false
fancybox: false
tags: [Docker, Linux, C, Compiler, Runtime]
date: 2019-02-15 22:46:25
weburl: Deep_Into_Dokcer_Helloworld
categories: 软件之道

---

学习Docker时一般刚开始接触的第一个docker image就是`hello-world`，这个image运行起来的效果也很简单直接，仅仅是在屏幕上输出一段Docker的使用说明就结束了。这个镜像虽然简单，然而仔细分析下还是涉及不少底层机制的。

<!--more-->

我之所以会对这个镜像感兴趣，是发现它的大小仅仅只有1.84kB，这实在是太小了，写一个`printf("Hello Wolrd\n");`的程序编译出来大小就远超1.84kB了，所以很好奇这个镜像是如何构建出来的。

## Dockerfile

Docker的镜像构建过程是由其镜像描述文件Dockerfile决定的，所以就先找到其Dockerfile来看看。`hello-world`用于`AMD64`架构的Dockerfile可以在[Github上](https://github.com/docker-library/hello-world/blob/b715c35271f1d18832480bde75fe17b93db26414/amd64/hello-world/Dockerfile)找到，只有简单的3行：

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

第1行导入了一个名为`scratch`的东西，这并不是一个真正的image，可以把它视为是所有image的最底层虚拟镜像，类似于一个基本抽象类，Docker官方对其的说明[如下](https://hub.docker.com/_/scratch)：

> This image is most useful in the context of building base images (such as [`debian`](https://registry.hub.docker.com/_/debian/) and [`busybox`](https://registry.hub.docker.com/_/busybox/)) or super minimal images (that contain only a single binary and whatever it requires, such as [`hello-world`](https://registry.hub.docker.com/_/hello-world/)).
>
> As of Docker 1.5.0 (specifically, [`docker/docker#8827`](https://github.com/docker/docker/pull/8827)), `FROM scratch` is a no-op in the `Dockerfile`, and will not create an extra layer in your image (so a previously 2-layer image will be a 1-layer image instead).
>
> ......
>
> You can use Docker’s reserved, minimal image, `scratch`, as a starting point for building containers. Using the `scratch` “image” signals to the build process that you want the next command in the `Dockerfile` to be the first filesystem layer in your image.

后面两行的含义也很直接，把一个名为hello的程序copy到根目录下，在运行image的时候运行此程序。下面就来看下这个如此小的hello world程序是如何实现的。

## 主程序

hello.c文件的源码也在同一个[Github仓库中](https://github.com/docker-library/hello-world/blob/master/hello.c)，省略掉过长的字符串常量后很简单：

```c
#include <sys/syscall.h>

const char message[] =
	"Hello World!"
	"\n";

void _start() {
	syscall(SYS_write, 1, message, sizeof(message) - 1);
	syscall(SYS_exit, 0);
}
```

这个最简版本的Hello World和C语言教科书中第一个Hello World是有不小差别的。首先是程序入口点上，众所周知正常C/C++程序的入口点是`main()`，然而这里使用的是`_start()`。

我们的程序是运行在Linux系统上的，程序的加载与运行必然是由OS发起的，**对于Linux来说，OS层面的程序入口点就是`_start()`而不是`main()`**，一个程序要能正常运行在`main()`之前是有一些准备工作要做的，比如建立程序运行环境（初始化.bss全局变量等）；在`main()`返回之后也有些收尾工作要处理，比如调用`exit()`通知系统等。这些工作正常情况下是由语言标准库来完成的，也就是所谓的Runtime运行环境，对于C语言来说就是`crt0.o`。大部分程序的`_start()`就位于其中，在建立好运行环境后`_start()`会调用`main()`跳转到用户定义的入口点处。当`main()`返回后程序又将回到`ctr0.o`中，最终调用`exit()`通知OS回收进程资源。

这里为了缩小程序体积和简单起见，没有使用标准的`ctr0.o` Runtime，事实上这一个简单的程序也不需要什么Runtime。程序最后直接通过`syscall`函数调用了`SYS_exit`系统调用结束了自身的运行。

将字符串输出到屏幕上也没有使用标准库中的`printf()`，同样是直接调用了`SYS_write`这个系统调用，其第一个参数显式的写为了1，其实就是`STDOUT_FILENO`，Linux系统在`unistd.h`中定义了`stdin`, `stdout`, `stderr`这几个标准文件描述符。

可以看到，这样一个程序是可以不依赖于任何其他的库在Linux上独立运行的，为了实现不链接C标准库的目的，需要使用一些特殊的编译选项。从编译这个`hello-world`程序使用的[Makefile](https://github.com/docker-library/hello-world/blob/master/Makefile)中可以找到使用的编译选项为：

```makefile
CFLAGS := -static -Os -nostartfiles -fno-asynchronous-unwind-tables
```

- `-static`表示静态链接，虽然对这个程序来说无所谓动态链接还是静态链接……
- `-Os`表示为空间进行`-O2`级别的优化，专门用于减少目标文件大小；
- `-nostartfiles`是关键编译选项，此选项表示不使用标准C语言运行库（即`crt0.o`），也不链接C标准库；
- `-fno-asynchronous-unwind-tables`选项也是用于减少代码空间的，其大概含义是不产生C++异常处理机制中使用的`.eh_frame`段，关于什么是`unwind-tables`和`.eh_frame`是个比这篇文章复杂多了的问题，文末有几篇参考资料，之后有空可以深入学习下C++的底层机制……

进行了以上诸多特殊优化处理后，终于可以得到一个只有1k多的可以正常运行于Linux上的Hello World程序了。

---------

> 参考资料：
>
> [What is the use of _start() in C?](https://stackoverflow.com/questions/29694564/what-is-the-use-of-start-in-c)
>
> [When is the gcc flag -nostartfiles used?](https://stackoverflow.com/questions/43050089/when-is-the-gcc-flag-nostartfiles-used)
>
> [GCC x86 code size optimizations](https://software.intel.com/en-us/blogs/2013/01/17/x86-gcc-code-size-optimizations)
>
> [c++ 异常处理（2）](https://www.cnblogs.com/catch/p/3619379.html)