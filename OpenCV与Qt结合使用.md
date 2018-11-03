title: OpenCV与Qt结合使用
permalink: OpenCV_and_Qt
toc: true
mathjax: false
fancybox: false
tags: [OpenCV, Qt]
categories: 编程之法
date: 2016-12-10 22:32:18

---

OpenCV本身能生成的GUI界面极为简陋，故一般使用MFC或Qt等框架来搭建GUI界面，并将OpenCV嵌入进去。因为我之前也用过Qt，故此处选择了Qt作为GUI框架，二者结合的主要问题在于图片的显示上，本文就以最新的OpenCV3及Qt5为例介绍一下实现方法。

<!--more-->

## 基本环境的搭建

首先保证独立的Qt及OpenCV程序都能在VS上正常的编译运行起来。

### 相关版本
Windows 10 64-bit
Visual Studio 2015
OpenCV 3.1
Qt 5.7

### Qt @ VS
> 参考之前的文章：[VS2015搭建Qt 5.7开发环境](/2016/12/09/VS2015_Qt5_7/)

注意Qt要选32-bit，以便和OpenCV兼容。

### OpenCV @ VS
> 参考之前的文章：[VS2015下编译OpenCV 3.1](/2016/12/06/VS2015_OpenCV31/)

需要注意的是，CMake配置时有一个`WITH_QT`选项，这个选项是指OpenCV的`highgui`模块本身是否使用Qt，似乎可以不选这个选项的。如果选择了`WITH_QT`，点击`Configure`后会出现新的Qt相关路径的选项，如果Qt安装正常的话，这里的路径是会自动生成的：

![](http://gmf.shengnengjin.cn/20161209223450.png)

点击`Generate`后会出现很多如下warning：

``` no-highlight
CMake Warning (dev) in modules/highgui/CMakeLists.txt:
  Policy CMP0020 is not set: Automatically link Qt executables to qtmain
  target on Windows.  Run "cmake --help-policy CMP0020" for policy details.
  Use the cmake_policy command to set the policy and suppress this warning.
This warning is for project developers.  Use -Wno-dev to suppress it.
```

相关解释可以参考[这里](https://cmake.org/cmake/help/v3.0/policy/CMP0020.html)，直接忽略这个警告即可。

另外，在编译OpenCV时也会出现很多和Qt有关的Warning，不过没有Error，可以忽略。

选择了`WITH_QT`后，`highgui`本身的窗口界面会变成基于Qt的，比原来要好看一些，而且自动集成了一些图片缩放、保存等功能：

![](http://gmf.shengnengjin.cn/20161209224409.png-width600)

不过这还是满足不了我们自由设计GUI界面的需求，接下来就要讨论如何在Qt项目中使用OpenCV。

## 数据结构转换算法

OpenCV中使用`Mat`保存图片，而Qt中则使用`QImage`或`QPixmap`，故二者结合的核心就在于这两个类之间的转换。网上已经有人实现了转换代码：

> [Converting Between cv::Mat and QImage or QPixmap](https://asmaloney.com/2013/11/code/converting-between-cvmat-and-qimage-or-qpixmap/)

经实际测试，上述代码基本是可以正常使用的，只是有一些小Bug需要修正下，后文会具体说明。这里将其中`Mat`->`QImage`的核心代码摘录备份如下：

```
QImage cvMatToQImage( const cv::Mat &inMat )
{
  switch ( inMat.type() )
  {
     // 8-bit, 4 channel
     case CV_8UC4:
     {
        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_ARGB32 );

        return image;
     }

     // 8-bit, 3 channel
     case CV_8UC3:
     {
        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_RGB888 );

        return image.rgbSwapped();
     }

     // 8-bit, 1 channel
     case CV_8UC1:
     {
        static QVector<QRgb>  sColorTable( 256 );

        // only create our color table the first time
        // 注意：下面这种写法是错的，参考后文的说明
        if ( sColorTable.isEmpty() )
        {
           for ( int i = 0; i < 256; ++i )
           {
              sColorTable[i] = qRgb( i, i, i );
           }
        }

        QImage image( inMat.data,
                      inMat.cols, inMat.rows,
                      static_cast<int>(inMat.step),
                      QImage::Format_Indexed8 );

        image.setColorTable( sColorTable );

        return image;
     }

     default:
        // qWarning() << "cvMatToQImage() - cv::Mat image type not handled in switch:" << inMat.type();
        break;
  }

  return QImage();
}
```

**上述代码在处理`CV_8UC1`类型时存在Bug**，`sColorTable.isEmpty()`这个判断永远都是不会为真的，即使是在第一次执行时。解决方法有以下几种：
1.直接去掉这个`if`判断，此做法的缺点是每次都会生成颜色表，效率不高；
2.使用`sColorTable`中元素的值来判断是否初始化过了；
3.一般使用时会将这个方法封装在一个类中，此时可以将`sColorTable`设置为类变量，在构造函数中初始化即可。

以上这种方法相当于调用构造函数生成了一个新的QImage，然后复制图像数据过去。另一种思路可参考以下链接：

> [详解 QT 框架中快速应用OpenCV 基于图片 上篇](http://mobile.51cto.com/symbian-271260.htm)


## Qt中图片的显示

最简单的办法是使用`QLabel`，更高效一些的方法是使用`QWidget`。使用`QLabel`时只需调用`setPixmap()`方法设置图片即可，`QPixmap`与`QImage`间的转换可用`QPixmap::fromImage()`函数完成。另外，使用`QLabel`的`setScaledContents()`方法可以设置图像自动缩放。

给出一个完整的例子：

```cpp
Mat inputimg = imread("pic.jpg");
QImage img = cvMatToQImage(inputimg);
ui.label->setScaledContents(true);
ui.label->setPixmap(QPixmap::fromImage(img));
```
