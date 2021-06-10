title: VS2015下编译OpenCV 3.1
weburl: VS2015_OpenCV31
toc: true
mathjax: false
fancybox: false
tags: [OpenCV]
categories: 工具之术
date: 2016-12-06 21:54:42

---

本文介绍一下如何从源码编译OpenCV 3.1，使用的环境是Win10 64bit + VS 2015。

<!--more-->

## 下载源文件

1. [OpenCV 3.1 Release](https://github.com/opencv/opencv/releases)
2. [CMake GUI 3.7.1](https://cmake.org/download/)

## 运行Cmake

打开CMake GUI，选择`Browse Source...`指定源文件路径，`Browse Build...`指定目标文件路径；之后点击`Configure`，会弹出一个对话框选择`Generator`，对于VS2015来说选`Visual Studio 14 2015`或`Visual Studio 14 2015 Win64`，前者是32位，后者是64位。确定后CMake会自动下载一些依赖文件，所以需要保证良好的网络连接，不过其中`ippicv_windows_20151201.zip`文件会因为网络问题始终无法成功下载，可以从网上搜索这个文件下载下来，比如从[这里](https://pan.baidu.com/s/1o7efLdK)。

如果最终的状态是`Configuring done`，且列出了此时可用的设置，说明配置成功，再点击`Generate`即可生成VS工程，默认的配置是可以正常编译使用的。

**需要注意的是，选择Win32编译32位版本是完全正常的，不过选择Win64编译64位版本时，Debug版可以正常编译通过，而Release版则无法编译成功，显示编译器内部错误，根据网上的资料这应该是VS2015的Bug。故使用VS2015编译OpenCV 3.1时建议使用32位模式进行编译。**

## VS编译

点击`Open Project`按钮打开生成的VS工程，待其完全载入后即可编译了，需要注意的是，不要使用默认的生成解决方案（F7）进行编译，这样最终得到的`install`文件夹中的内容是不全的。正确方法是在`INSTALL`工程上点右键，选择生成，单独编译生成此工程。

正常情况下是可以成功编译生成的，最终的得到的所有所需文件都位于目标文件路径下`.\install`文件夹中，其中`.\install\include`文件夹中存放的是头文件，`.\install\x64\vc14\bin`文件夹中是dll动态链接库文件，`.\install\x64\vc14\lib`和`.\install\x64\vc14\staticlib`中都是库文件。

**不过如果想对OpenCV源码进行Debug跟踪，是不能选用`.\install`文件夹中的库文件的，因为此处缺少`.pdb`调试文件，无法正常Debug。此时需要使用`.\lib`文件夹中的库文件，这时可以正常的跟踪到源代码中，且使用profiler分析代码性能时也可以正确定位到OpenCV内部函数上。**

## 定制优化OpenCV

上面生成的文件已经是可以正常使用的了，不过自行编译源代码的一大优点就是可以根据自己的需求进行配置。

### 使用World模块

默认情况下编译结果是很多的`lib`文件及`dll`文件，使用起来不是很方便，OpenCV提供了一个`World Module`的功能，可以把生成的文件链接在一起，合成一个`dll`及`lib`文件。选取`BUILD_opencv_world`即可。不过实际测试表明，使用了World模块后编译的时间会明显变长，故是否使用根据需要决定。

### 去除不需要的功能

根据需要去掉一些不需要的功能即可，如`WITH_1394`等。

### 开启CPU指令集支持

根据使用的CPU，可以开启`ENABLE_AVX`、`ENABLE_FMA3`等矢量指令集支持功能，以提高整体性能。
