title: 为Hexo博客移动端添加搜索功能
weburl: Blog_Search_Mobile
toc: true
mathjax: false
fancybox: true
tags: [Blog, Debug, Top]
categories: 编程之法
date: 2016-11-25 22:00:59

---

之前已经为博客的PC端界面添加了搜索功能，用起来效果不错，不过美中不足的是，使用手机访问的时候并不能使用搜索功能。这几天折腾了一下，终于给移动端界面上也加上了搜索功能~这里来记录一下实现方法。

<!--more-->

之前为博客添加搜索功能的方法见：

> [为Hexo博客Yilia主题添加本地站内搜索功能](/2016/10/10/%E4%B8%BAHexo%E5%8D%9A%E5%AE%A2Yilia%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%9C%AC%E5%9C%B0%E7%AB%99%E5%86%85%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD/)

要为移动端添加搜索功能，主要就是添加搜索框及搜索结果HTML样式及添加相关JS代码。

## 添加搜索框及搜索结果区

当前主题在移动端的布局中，点击左上角的菜单按钮就会弹出一个显示友情链接、标签等信息的叠层，综合看下来将搜索框添加在这个位置较合适，最终的效果如图：

![](https://img.gaomf.cn/20161125210304.png)

这里的搜索框和PC端基本是一样的，只是样式上做了些变化，使得整体效果更和谐。找到合适的位置添加搜索框及搜索结果区的HTML代码即可：

```HTML
<form id="search-form_mobile"  class="search_mobile">
    <input type="text" id="st-search-input_mobile" name="q" results="0" class="st-default-search-input_mobile" maxlength="50" placeholder="Search..." autocomplete="off" autocorrect="off">
    <i class="fa fa-times" onclick="resetSearch_mobile()"></i>
    <p id="search_hint" class="search-hint">向右滑动结果打开链接~</p>
    <div id="local-search-result_mobile"></div>
    <p class="no-result">No results found <i class="fa fa-spinner fa-pulse"></i></p>
    <p class="loading-xml">Loading XML File... <i class="fa fa-spinner fa-pulse"></i></p>
</form>
```

布局和PC端的稍有差异，搜索结果区和搜索框放在了同一个`form`里面。CSS样式此处从略，如有兴趣可参考源文件：[search.styl](https://github.com/g199209/BlogTheme/blob/master/source/css/_partial/search.styl)

## 添加JS代码

此主题移动端的时候使用的JS文件为`mobile.js`（PC端是`pc.js`），按照PC端的做法添加相关代码即可，注意搜索框等的名字要改为对应的移动端名字。这样修改后是可以实现基本功能了，然而用起来会有各种Bug，这两天的时间就主要花在解决这些Bug上了……

### 解决触摸无响应问题
直接把代码移植过来的第一个问题就是，在手机上那个搜索框实际上是没有用的，无法点击进行输入……具体原因并不清楚，只找到了可用的解决方案。在JS文件`search`函数中添加以下代码：

``` js
// 处理移动端搜索框无法点击的问题
inputArea.addEventListener("touchstart", function(){
    inputArea.focus();
}, false);

resetButton.addEventListener("touchstart", function(){
    $resetButton.click();
}, false);
```

监听`touchstart`事件，并通过程序触发`focus`及`click`事件。

### 解决滚动异常问题
加上以上代码后可以搜索了，只是搜索结果的滚动有各种奇怪的问题……比如只有输入单个字符的时候可以滚动、会发生点击穿透导致后面的页面滚动、只能按住标题进行滚动等等……各种尝试后决定不去解决这些问题了，直接监听触摸事件，自己重写滚动响应及滑动打开的代码。

基本思路是这样的：

- 在`touchstart`事件中记录触摸开始的坐标点，并且判断是按在了哪个搜索结果条目上；
- 在`touchmove`事件中获取手指移动的方向，判断是进行垂直滚动还是水平移动搜索条目（用于打开搜索结果），之后修改对应元素的`marginTop`或`marginLeft`属性即可实现界面元素的移动；
- 最后在`touchend`事件中判断是否发生了滑动打开链接事件，如果有的话打开对应链接。

需要注意的是，要调用`preventDefault()`来阻止触摸事件的传递，以避免各种奇怪的问题。除了基本逻辑外，还做了些处理让视觉效果更好看些，具体见以下的实现代码：

``` js
TouceArea.onscroll = function(e) {
    e.preventDefault();
}

TouceArea.ontouchstart = function(e) {
    e.preventDefault();
    
    // 记录起始点坐标
    MarginOffset = parseInt(ScrollArea.style.marginTop.replace("px", ""));
    if (isNaN(MarginOffset)) {
        MarginOffset =  0;
    }
    StartY = e.touches[0].pageY ;
    StartX = e.touches[0].pageX;
    
    // Clear Lock
    ScrollVerticalLock = false;
    ScrollHorizontalLock = false;
    
    // 找出点击了哪个选项
    var i;
    if (TouceArea.children.length > 0) {
        var liArea;
        var i;
        TouchedItem = null;
        TouchedOpen = false;
        for (i = 0; i < TouceArea.children[0].childNodes.length; i++) {
            liArea = TouceArea.children[0].childNodes[i];
            if (liArea.offsetTop > StartY) {
                TouchedItem = TouceArea.children[0].childNodes[i - 1];
                break;
            }
        }
        // Deal Last One
        if (!TouchedItem) {
            if (liArea.offsetTop + liArea.clientHeight > StartY) {
                TouchedItem = TouceArea.children[0].childNodes[TouceArea.children[0].childNodes.length - 1];
            }
        }
    }

}

TouceArea.ontouchmove = function(e) {
    e.preventDefault();
    
    // 记录当前坐标偏移
    var OffsetY = e.touches[0].pageY - StartY;
    var OffsetX = e.touches[0].pageX - StartX - 20;  // 减去一个值，避免微小移动也触发事件，影响体验
    
    // 实现水平滑动
    if (ScrollHorizontalLock) {
        if (TouchedItem && OffsetX > 0) {
            TouchedItem.style.marginLeft = OffsetX + "px";
            if (OffsetX > TouchedItem.clientWidth / 2) {
                TouchedItem.style.backgroundColor = "rgba(13,118,13,0.60)";
                TouchedOpen = true;
            } else {
                TouchedItem.style.backgroundColor = "";
                TouchedOpen = false;
            }
        }
        return;
    }
    
    // 实现垂直滚动
    var NewMargin = OffsetY + MarginOffset;
    if (TouceArea.clientHeight + NewMargin < WholeView.clientHeight * 0.7) {
        NewMargin = WholeView.clientHeight * 0.7 - TouceArea.clientHeight;
    }
    if (NewMargin > 0)
        NewMargin = 0;
    ScrollArea.style.marginTop = NewMargin + "px";
    //默认情况也会响应垂直滚动，故ScrollVerticalLock的判断放到后面
    if (ScrollVerticalLock) {
        return;
    }

    // 判断滚动方向
    if (!TouchedItem) {
        ScrollVerticalLock = true;
    } else if (Math.abs(OffsetY) > 15) {
        ScrollVerticalLock = true;
    } else if (OffsetX > 0) {
        ScrollHorizontalLock = true;
    }
}

TouceArea.ontouchend = function(e) {
    e.preventDefault();
    
    // 处理拖动打开链接事件
    if (TouchedItem) {
        TouchedItem.style.marginLeft = "0px";
        TouchedItem.style.backgroundColor = "";
        if (TouchedOpen) {
            hide(); // 隐藏搜索框
            // 待搜索框完全隐藏后再打开新链接，视觉效果更好点
            window.setTimeout('window.location.href=' + '"' + TouchedItem.children[0].href + '"', 200);
        }                        
    }
}
```

完整的`mobile.js`文件在[这里](https://github.com/g199209/BlogTheme/blob/master/source/js/mobile.js)。

## 最终成品效果
实际使用下来还是比较满意的，搜索效果像这样：

![](https://img.gaomf.cn/Search_mobile.gif)




