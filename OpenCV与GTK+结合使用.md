title: OpenCV与GTK+结合使用
weburl: OpenCV_and_GTK
toc: false
mathjax: false
fancybox: false
tags: [OpenCV]
categories: 编程之法
date: 2017-06-01 22:55:41

---

之前研究过[OpenCV与Qt结合使用](/2016/12/10/OpenCV_and_Qt/)，毕设要用的Intel Joule开发板上安装Qt较为困难，不过它自带了GTK+3.0 GUI框架，于是研究了下是否可以在GTK+中显示OpenCV的图片，折腾了一下发现是可行的，记录一下方法。

<!--more-->

OpenCV默认的`imshow`似乎使用的就是GTK+框架，在GTK中显示图片的方法参考以下这篇教程：

> [GTK常用控件之图片控件(GtkImage)](http://blog.csdn.net/tennysonsky/article/details/43057081)

可以看到，GTK中的图片是使用`GdkPixbuf`结构体来保存的，而OpenCV 3.x中使用`Mat`类来保存图片，所以只要实现`Mat`到`GdkPixbuf`的转换即可在GTK中显示图片了，实现代码如下：

```C++
Mat Image;

// Mat(BGR) -> IplImage(RGB)
cvtColor(image, image, COLOR_BGR2RGB);
IplImage ipltmp = image;

// IplImage -> GdkPixbuf
GdkPixbuf * src = gdk_pixbuf_new_from_data(
	(const guchar *)ipltmp.imageData, 
	GDK_COLORSPACE_RGB, 
	0, 
	ipltmp.depth, 
	ipltmp.width, 
	ipltmp.height, 
	ipltmp.widthStep, 
	NULL, 
	NULL);
```

关于`gdk_pixbuf_new_from_data()`及`GdkPixbuf`的详细说明参见：

> [Gdk-pixbuf](http://openbooks.sourceforge.net/books/wga/graphics-gdk-pixbuf.html)

----------

> 参考资料：
> [How can I display an OpenCV IplImage in Gtk+/Gtkmm?](https://stackoverflow.com/questions/1188665/how-can-i-display-an-opencv-iplimage-in-gtk-gtkmm/1230761#1230761)
> [Converting cv::Mat to IplImage*](https://stackoverflow.com/questions/4664187/converting-cvmat-to-iplimage)