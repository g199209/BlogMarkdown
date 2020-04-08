title: Cadence Allegro 调整界面字体字号
permalink: Allegro_Font
toc: false
mathjax: false
fancybox: false
tags: [EDA]
categories: 工具之术
date: 2017-05-14 10:54:01

---

在使用高分屏时，会发现系统设置的dpi缩放对Allegro无效，这就导致对话框中的文本会显得很小，极不方便使用。不过研究发现，Allegro中是有设置可以调整字体和字号的。

<!--more-->

打开`Setup`->`User Preference...`，在左侧`Categories`中展开`Ui`->`Fonts`。右侧的设置中，`fontsize`代表字号，这是个负数，值越小字号越大，默认值是`-12`；`fontface`是字体；`fontweight`是用来控制加粗的，`500`代表粗体，`300`是不加粗字体，默认的`400`介于二者之间。


![](https://pic.gaomf.store/20170514103643.png)

确定保存后重启程序即可生效。

最后附上官方帮助文档中的说明以供参考：

> Changing Fonts
> 
> The layout editor lets you customize the look of the graphical user interface by changing the size and type of the fonts in the console, status, and Options windows, and in the Find window pane. This can be convenient if you find it difficult to read information presented in the default size and type.
> 
> To change fonts in the user interface:
> 
> 1. Exit the layout editor, if you have it running.
> 
> 2. Set the font variables in your environment file.
> These variables can also be set in the System dialog box in the Control Panel.
> `fontSize` = -12 
> where -12 represents the default font size. A larger negative number (for example -20) makes the font larger. Do not use positive numbers in this value.
> `fontFace` = helvetica
> where helvetica represents the default font type. Fonts available to you depend on your platform and any user-installed fonts. The value is always a font name.
> `fontWeight` = 500
> where 500 represents bolded type. Change the value to 300 to produce unbolded type.
> 
> 3. Restart the tool.
> 
> 4. Resize the window, if necessary, to display all information in the larger font size.
> 
> You can also change font variables in the User Preferences Editor dialog box by running the enved command. Note that you must restart the tool to see the change.
