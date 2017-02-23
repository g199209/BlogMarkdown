title: Matlab调用Origin作图
date: 2016-01-28 21:23:24
tags: [Matlab]
categories: 科研

---

Matlab作出的图普遍没有Origin作出的美观好看，而且导出为eps或emf格式后会有各种奇怪的Bug。目前普遍采用的一种方法是，将Matlab数据导出为mat文件后再导入Origin中手工作图，这种方式需要不少重复性劳动，并不是一种很完美的解决方案。
前几天偶然看到Origin提供了COM接口可供Matlab调用，于是就研究了下可否用Matlab调用Origin来生成所需的emf格式图片，最终经过一番折腾，基本完成了这个目标。

<!--more-->

之所以能用Matlab来调用Origin，这要依赖于Origin中提供的[Automation Server](http://www.originlab.com/index.aspx?go=Products/Origin/Programming/AutomationServer)服务。这个服务提供了一个COM接口来供其他程序调用，官方提供了Matlab、VB、Excel、C#、LabVIEW等诸多程序调用Origin的例子。Automation Server的详细使用方法可参考其[官方帮助文档](http://www.originlab.com/doc/COM)。

Matlab调用Origin的示例程序位于`<Install Path>\Samples\COM Server and Client\MATLAB`路径下（以Origin 2015为例，其他版本的位置可能有所不同）。一共有两个m文件，`CreatePlotInOrigin.m`及`MATLABCallOrigin.m`，前者用于实现调用Origin绘图，并将结果保存到剪贴板中，后者演示了如何创建工作表(Worksheet)，如何插入新列等操作。另外一个`CreatePlotInOrigin.opj`文件是供`CreatePlotInOrigin.m`调用的一个Origin模板文件。

根据这两个示例程序基本就可以依葫芦画瓢写出一个符合自己要求的程序了，然而这其中并没有导出emf格式图片的示例，于是开始研究其官方帮助文档……官方帮助文档很多地方都语焉不详，而且其间还经历了种种坑，比如[上篇文章](http://GaoMF.cn/2016/01/27/Origin%20Script%20Window%20%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E4%BB%A3%E7%A0%81%E7%9A%84%E6%96%B9%E6%B3%95/)这个。不过最终还是找到了正确的解决办法，就是使用Origin [X-Function](http://www.originlab.com/doc/X-Function)中的[expGraph](http://www.originlab.com/doc/X-Function/ref/expGraph)命令。

----------

最终找到的较好的解决方案是这样的：

首先，用Origin生成一个空白模板工程，其中包含了基本的Worksheet结构及Graph样式，比如示例文件中提供的这个：
![](http://7xnwyt.com1.z0.glb.clouddn.com/Matlab20160128204513.png)

这个模板工程需要保证只要向Worksheet中填入数据，Graph中就能生成所需的图，就像这样：
![](http://7xnwyt.com1.z0.glb.clouddn.com/Matlab20160128205045.png)

这里的Worksheet和Graph可以不止有一个，不过一般情况下一个就足够了。

制作好了模板文件后，在Matlab程序中只需要通过COM接口调用Origin，打开这个模板文件，然后向其中的Worksheet填入正确的数据，最后导出图片文件即可。
Matlab程序如下：
```Matlab
% 调用Origin作图并保存为emf格式的图片
% 作者 : 高明飞
% 日期 : 2016-01-27

% mdata : 需要填充到Origin Worksheet中的数据
% template : Origin模板函数名，不含后缀，需要保存在当前工作目录下，如'CreatePlotInOrigin'
% fdir : 输出图片目标文件夹，如'D:\image'
% fname : 输出图片文件名，不含后缀，如'abc'

function OriginPlot(mdata, template, fdir, fname)
% Obtain Origin COM Server object
% This will connect to an existing instance of Origin, or create a new one if none exist
originObj=actxserver('Origin.ApplicationSI');

% Clear "dirty" flag in Origin to suppress prompt for saving current project
invoke(originObj, 'IsModified', 'false');

% Load the custom template project
dir = pwd;
dir = strcat(dir, '\', template, '.opj');
invoke(originObj, 'Load', dir);

% Send this data over to the Data1 worksheet
invoke(originObj, 'PutWorksheet', 'Data1', mdata);

% Save graph
cmd = 'expGraph type:=emf overwrite := rename tr1.unit := 2 tr1.width := 10000 path:= "';
cmd = strcat(cmd, fdir, '" filename:= "', fname, '.emf";');
invoke(originObj, 'Execute', cmd);

% Release
release(originObj);
end
```
上面这段程序中要求Worksheet的名称需要为`Data1`，这是由`invoke(originObj, 'PutWorksheet', 'Data1', mdata);`这句代码确定的；
导出的图片为emf格式，图像宽度为10000像素，因为这是矢量图，所以文件体积并不大的。

以上这个版本只是个最基本的版本，不过一般使用也够用了，更多的高级功能，比如动态调整坐标轴名称，动态调整x、y轴的范围以适应不同数据等之后有空再来研究……
