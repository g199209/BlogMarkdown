title: Doxygen处理条件编译
date: 2015-11-13 22:59:59
tags: [工具]
categories: 编程

---

在使用Doxygen生成文档的时候，发现有些内容没有生成。仔细研究程序源代码，发现这部分代码使用了条件编译进行控制，代码如下：
``` C
#ifdef HAL_ADC_MODULE_ENABLED
// Code
#endif /* HAL_ADC_MODULE_ENABLED */
```
而Doxygen是会对宏进行处理的，这样就会跳过这部分代码。
解决方式：在`Expert`的`Preprocessor`选项中，找到`PREDEFINED`，添加需要预定义的宏即可。

<!--more-->

截图如下：
![](http://7xnwyt.com1.z0.glb.clouddn.com/Doxygen20151113230829.png)


