title: OpenCV 3.0安装
date: 2015-12-08 00:18:12
tags: [OpenCV]
categories: 工具之术

---

主要参考以下文章：
> [【OpenCV入门教程之一】 安装OpenCV](http://blog.csdn.net/poem_qianmo/article/details/19809337)
> [64位系统vs2013配置opencv3.0](http://blog.csdn.net/desti5/article/details/39012343)
> [VS2013+openCV3.0无脑配置方法+解决警告问题【windows平台】](http://www.bubuko.com/infodetail-793518.html)

目前OpenCV的最新版本为3.0.0，正式版发布于2015年6月4日。本文记录一下安装OpenCV 3.0及新建工程的步骤。相关环境：Windows 8.1 64bit；Visual Studio 2013。

<!--more-->

## **下载OpenCV SDK** ##
在[官网](http://opencv.org/)上找到OpenCV for Windows下载下来即可，大概有300MB不到。这是一个自解压压缩包，下载完后直接双击运行即可。**因为OpenCV项目文件打包的时候，根目录就是opencv，所以我们不需要额外的新建一个名为opencv的文件夹，然后再解压**。解压后的文件有接近3G。

## **配置环境变量** ##
在PATH路径中增加以下两项即可：
```
\opencv\build\x86\vc12\bin;
\opencv\build\x64\vc12\bin;
```
其中的`vc12`对应VS2013，`vc11`对应VS2012。在之前版本的OpenCV中，还存在`vc10`文件夹对应VS2010，不过在OpenCV 3.0中，去掉了`vc10`文件夹。
这里可以只保留需要用到的VS版本，删掉其余版本以节省空间。

## **编写OpenCV的VS工程属性表** ##
VS中配置OpenCV的包含路径、库文件路径等通常是通过项目属性对话框完成的，这样操作较为繁琐，故这里采用工程属性表的方式来进行配置。工程属性表是一个后缀为`.props`的文本文件，其中记录了工程属性配置情况，新建工程后可以直接导入此文件即可。

在OpenCV根目录下新建一个`.props`文件，比如`OpenCV.props`，文件内容如下：
```
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ImportGroup Label="PropertySheets" />
  <PropertyGroup Label="UserMacros" />
  <PropertyGroup>
    <IncludePath>D:\opencv\build\include;$(IncludePath)</IncludePath>
    <LibraryPath Condition="'$(Platform)'=='Win32'">D:\opencv\build\x86\vc12\lib;$(LibraryPath)</LibraryPath>
    <LibraryPath Condition="'$(Platform)'=='X64'">D:\opencv\build\x64\vc12\lib;$(LibraryPath)</LibraryPath>
  </PropertyGroup>
  <ItemDefinitionGroup>
    <Link Condition="'$(Configuration)'=='Debug'">
      <AdditionalDependencies>opencv_ts300d.lib;opencv_world300d.lib;%(AdditionalDependencies)
      </AdditionalDependencies>
    </Link>
    <Link Condition="'$(Configuration)'=='Release'">
      <AdditionalDependencies>opencv_ts300.lib;opencv_world300.lib;%(AdditionalDependencies)
      </AdditionalDependencies>
    </Link>
  </ItemDefinitionGroup>
  <ItemGroup />
</Project>
```
注意，其中具体路径要根据之前安装OpenCV的路径来确定。

第6行用于配置包含目录；第7行与第8行分别是Win32和X64情况下的库目录；第12行与第16行分别是Debug与Release情况下的附加依赖项。

OpenCV 3.0与之前的版本不同，库文件目录下同时有两个文件夹，`/lib/`与`/staticlib/`，并不需要同时添加这两个文件夹与其中的库文件，选择一个即可。如果选择`/staticlib/`的话与之前的版本一样，有一大堆的依赖项要添加；如果选择了`/lib/`的话，就只需要加入两个依赖项。故此处选择使用`/lib/`文件夹。

## **创建新工程** ##
正常新建一个VC++空项目，然后打开属性管理器，如果没有属性管理器的话可以从`视图`->`其他窗口`->`属性管理器`中打开。打开后在工程名上点击右键，选择添加现有属性表，找到之前新建的项目属性表文件添加进来即可。

新建一个`main.cpp`文件进行测试，使用如下测试代码：
```cpp
#include <opencv2\opencv.hpp>

using namespace cv;

int main()
{
  Mat img = imread("pic.jpg");
  imshow("Picture", img);
  waitKey();

  return 0;
}
```

将`pic.jpg`图片放置于工程目录下编译运行程序，如果之前配置正确的话就可以显示出图片了。
