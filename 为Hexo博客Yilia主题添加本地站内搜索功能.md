title: 为Hexo博客Yilia主题添加本地站内搜索功能
date: 2016-10-10 22:49:23
tags: [Web]
categories: 杂七杂八

---

博客的站内搜索功能一直是一个缺憾，最初使用了[Swiftype](https://swiftype.com/)，虽然效果不是很理想，不过也正常使用了一段时间，然而之后发现使用Swiftype会导致博客标签加载不完全，进而影响正常显示。奈何我基本不懂前端，瞎折腾了很久也没能解决这个问题，最后只得禁用了Swiftype，留下的搜索框成了一个摆设。前段时候发现Hexo博客可以使用本地站内搜索，于是又折腾了一天，然而最终还是没有做出可用的搜索界面来：(本来已经不抱什么希望了，今天偶然看见[让 Hexo 博客支持本地站内搜索](http://moxfive.xyz/2016/05/31/hexo-local-search/)这篇文章，作者使用的主题也是基于Yilia的，顿时觉得有戏，于是又是一天折腾，终于做出了个像样的站内搜索功能来了~~写篇文章记录下折腾过程。

<!-- more -->

## 生成索引文件

### 安装插件

本地站内搜索都是基于索引文件的，Hexo中可通过`hexo-generator-search`插件生成XML格式的索引文件，通过`hexo-generator-json-content`插件生成JSON格式的索引文件，此处选择了`hexo-generator-search`:

```
npm install --save hexo-generator-search
```

然后在Hexo站点根目录下的`_config.yml`中添加如下配置即可：

```
search:
  path: search.xml
  field: all
```

### 修改插件

默认生成的`search.xml`文件很大，而且包含很多冗余信息，参考[完美解决Hexo静态博客搜索问题](http://www.netcan666.com/2015/11/20/%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3Hexo%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2%E6%90%9C%E7%B4%A2%E9%97%AE%E9%A2%98/)这篇文章对其进行精简。

首先在修改`node_modules/hexo-generator-search/index.js`文件，在其中添加3个函数：

``` javascript
stripe_code = function(str) { // 去除代码
    return str.replace(/<figure class="highlight.*?<\/figure>/ig, '');
}
stripe = function (str) { // 去除html标签
    return str.replace(/(<([^>]+)>)/ig, '');
}
minify = function (str) { // 压缩成一行
    return str.trim().replace(/\n/g, ' ').replace(/\s+/g, ' ');
}
```

[Netcan](http://www.netcan666.com/)的文章中是去除代码行数，此处改成了去除所有代码，因为一般无需对代码进行搜索。

之后修改模板文件`search.ejs`，主要目的是调用上面添加的3个函数对实际内容进行精简，修改`<content type="html">`标签中的内容即可：

```
<content type="html"><%-: minify(stripe(stripe_code(post.content))) | cdata %></content>

<content type="html"><%-: minify(stripe(stripe_code(page.content))) | cdata %></content>
```

精简后，生成的`search.xml`文件体积可缩小为原来的1/3.

> Update 2018-01-21:
>
> 目前新版的`hexo-generator-search`插件模板文件位置有所改变，改为了`templates/xml.ejs`文件，修改方法不变。


## 界面结构及样式

### 添加HTML代码

搜索框沿用了之前Swiftype的搜索框，放在侧边栏的最上方，这个比较符合我的审美。参考[MOxFIVE](http://moxfive.xyz/2016/05/31/hexo-local-search/)的做法将搜索结果也放在侧边栏中，没有添加搜索重置按钮。修改`left-col.ejs`文件，在其中添加相关代码：

``` html
<div class="overlay"></div>
<div class="intrude-less">
    
<form id="search-form" class="search">  <!-- 搜索框相关 -->
    <input type="text" id="st-search-input" name="q" results="0" class="st-default-search-input" maxlength="30" placeholder="Search..." autocomplete="off" autocorrect="off">
</form>

<header id="header" class="inner">
    <a href="/" class="profilepic">
        <% if (theme.animate){ %>
        <img lazy-src="<%=theme.avatar%>" class="js-avatar">
        <%}else{%>
        <img src="<%=theme.avatar%>" class="js-avatar" style="width: 100%;height: 100%;opacity: 1;">
        <%}%>
    </a>

    <hgroup>
      <h1 class="header-author"><a href="/"><%=theme.author%></a></h1>
    </hgroup>
    

    <% if (theme.subtitle){ %>
    <p class="header-subtitle"><%=theme.subtitle%></p>
    <%}%>
    
    <div id="local-search-result"></div> <!-- 搜索结果区 -->
    <p class='no-result'>No results found </p> <!-- 无匹配时显示，注意请在 CSS 中设置默认隐藏 -->

    <!-- 以下保持不变，省略 -->
```

### 修改CSS样式

CSS样式直接使用了[MOxFIVE](http://moxfive.xyz/2016/05/31/hexo-local-search/)的样式（去掉了其中用不到的部分），在`style.styl`文件中添加以下代码：

``` css
form input.st-default-search-input {
    font-size: 12px;
    padding: 4px 9px 4px 27px;
    height: 20px;
    width: 140px;
    color: #666;
    border: 1px solid #ccc;
    -webkit-border-radius: 15px;
    -moz-border-radius: 15px;
    -ms-border-radius: 15px;
    -o-border-radius: 15px;
    border-radius: 15px;
    -webkit-box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
    -moz-box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
    box-shadow: inset 0 1px 3px 0 rgba(0,0,0,0.17);
    outline: none;
    background: #fcfcfc url(/img/search.png) no-repeat 7px 7px;
}

/*搜索结果区*/
#local-search-result {
  margin: auto -12% auto -6%;
  margin-top: 10px;
  font-size: 0.9em;
  text-align: left;
  word-break: break-all;
}

#local-search-result ul.search-result-list li:hover {
  font-weight: normal;
}

/*单条搜索结果*/
#local-search-result li {
  margin: 0.5em auto;
  border-bottom: 2px solid #d3d3d3;
}
#local-search-result .search-result-list li:hover {
  background: rgba(158,188,226,0.21);
  box-shadow: 0 0 5px rgba(0,0,0,0.2);
}

/*匹配的标题*/
#local-search-result a.search-result-title {
  line-height: 1.2;
  font-weight: bold;
  color: #708090;
}

/*搜索预览段落*/
#local-search-result p.search-result {
  margin: 0.4em auto;
  line-height: 1.2em;
  max-height: 3.6em;
  overflow: hidden;
  font-size: 0.8em;
  text-align: justify;
  color: #808080;
}

/*匹配的关键词*/
#local-search-result em.search-keyword {
  color: #f58e90;
  border-bottom: 1px dashed #f58e90;
  font-weight: bold;
  font-size: 0.85em;
}

/*无匹配搜索结果时显示*/
p.no-result {
  display: none;
  margin: 2em 0 2em 6%;
  padding-bottom: 0.5em;
  text-align: left;
  color: #808080;
  font-family: font-serif serif;
  border-bottom: 2px solid #d3d3d3;
}
```

其中`input.st-default-search-input`是Swiftype提供的搜索框样式。

### 添加侧边栏滚动条

Yilia主题的侧边栏是不会添加滚动条的，这就会造成搜索结果显示不完全，需要修改`main.styl`文件：

``` css
.left-col {
    background: #fff;
    width: 250px;
    position:fixed;
    opacity:1;
    transition:all .2s ease-in;
    height:100%;
+    overflow-y: auto;

.switch-area{
    position: relative;
    width: 100%;
    overflow: hidden;
-    min-height: 500px;
+    min-height: 200px;
    font-size: 14px;
    .switch-wrap{
        transition: transform .3s ease-in;
        position: relative;
    }
}
```

## **Javascript功能代码**

参考[MOxFIVE](http://moxfive.xyz/2016/05/31/hexo-local-search/)的代码，进行了以下修改：

1. 修改搜索框的名称以便和HTML代码配套；
2. 去除重置搜索按钮相关代码；
3. 不在新标签页中打开搜索结果；
4. 搜索结果URL链接去掉主域名，使用相对路径表示；
5. 保持搜索结果中标题大小写格式；
6. **对搜索结果相关度进行了打分，根据关键词出现的次数进行排序**

首先在`pc.js`文件中添加以下代码：

``` javascript
var search = function(){
    require(['/js/search.js'], function(){
        var inputArea = document.querySelector("#st-search-input");
        var $HideWhenSearch = $("#toc, #tocButton, .post-list, #post-nav-button a:nth-child(2)");
        var $resultArea = $("#local-search-result");

        var getSearchFile = function(){
            var search_path = "/search.xml";
            var path = search_path;
            searchFunc(path, 'st-search-input', 'local-search-result');
        }

        var getFileOnload = inputArea.getAttribute('searchonload');
        if (getFileOnload === "true") {
            getSearchFile();
        } else {
            inputArea.onfocus = function(){ getSearchFile() }
        }

        var HideTocArea = function(){
            $HideWhenSearch.css("visibility","hidden");
        }
        inputArea.oninput = function(){ HideTocArea() }
        inputArea.onkeydown = function(){ if(event.keyCode==13) return false}

        $resultArea.bind("DOMNodeRemoved DOMNodeInserted", function(e) {
            if (!$(e.target).text()) {
                $(".no-result").show(200);
            } else {
              $(".no-result").hide();
            }
        })
    })
}();
```

代码详细说明见[MOxFIVE](http://moxfive.xyz/2016/05/31/hexo-local-search/)的文章。

新建`source/js/search.js`文件：

``` javascript
// A local search script with the help of [hexo-generator-search](https://github.com/PaicHyperionDev/hexo-generator-search)
// Copyright (C) 2015 
// Joseph Pan <http://github.com/wzpan>
// Shuhao Mao <http://github.com/maoshuhao>
// Edited by MOxFIVE <http://github.com/MOxFIVE>
// Edited by Mingfei Gao <http://gaomf.cn>

var searchFunc = function(path, search_id, content_id) {
    'use strict';
    $.ajax({
        url: path,
        dataType: "xml",
        success: function( xmlResponse ) {
            // get the contents from search data
            var datas = $( "entry", xmlResponse ).map(function() {
                return {
                    title: $( "title", this ).text(),
                    content: $("content",this).text(),
                    url: $( "url" , this).text()
                };
            }).get();
            var $input = document.getElementById(search_id);
            var $resultContent = document.getElementById(content_id);
            $input.addEventListener('input', function(){
                var finalHTML='<ul class=\"search-result-list\">';
                var str = "";                
                var keywords = this.value.trim().toLowerCase().split(/[\s\-]+/);
                $resultContent.innerHTML = "";
                if (this.value.trim().length <= 0) {
                    return;
                }
                // Search result Array
                function SearchData(str, score) {
                    this.str = str;
                    this.score = score;
                }
                var SearchResultArr = new Array();
                var tmpscore;
                // perform local searching
                datas.forEach(function(data) {
                    var content_index = [];
                    var data_title = data.title.trim().toLowerCase();
                    var data_content = data.content.trim().replace(/<[^>]+>/g,"").toLowerCase();
                    var data_url = data.url.replace("http://gaomingfei.xyz","");
                    var index_title = -1;
                    var index_content = -1;
                    var first_occur = -1;
                    // only match artiles with not empty titles and contents
                    if(data_title != '' && data_content != '') {
                        tmpscore = 0;
                        keywords.forEach(function(keyword, i) {
                            index_title = data_title.indexOf(keyword);
                            index_content = data_content.indexOf(keyword);
                            
                            if (index_title >= 0) {
                                tmpscore += 30;
                            }
                            if (index_content >= 0) {
                                if (first_occur < 0) {
                                    first_occur = index_content;
                                }
                                while (index_content >= 0) {
                                    tmpscore += 1;
                                    index_content = data_content.indexOf(keyword, index_content + 1);
                                }
                            }
                        });
                    }
                    // show search results
                    if (tmpscore > 0) {
                        str = "<li><a href='"+ data_url +"' class='search-result-title'>"+ "> " + data.title +"</a>";
                        var content = data.content.trim().replace(/<[^>]+>/g,"");
                        // cut out characters
                        var start = first_occur - 6;
                        var end = first_occur + 6;
                        if(start < 0){
                            start = 0;
                        }
                        if(start == 0){
                            end = 10;
                        }
                        if(end > content.length){
                            end = content.length;
                        }
                        var match_content = content.substr(start, end); 
                        // highlight all keywords
                        keywords.forEach(function(keyword){
                            var regS = new RegExp(keyword, "gi");
                            match_content = match_content.replace(regS, "<em class=\"search-keyword\">"+keyword+"</em>");
                        })
                        str += "<p class=\"search-result\">" + match_content +"...</p>"
                        
                        SearchResultArr.push(new SearchData(str, tmpscore));
                    }
                })
                
                // Sort Search Result
                function compareScore(a, b) {
                    return b.score - a.score;
                }
                SearchResultArr.sort(compareScore);
                
                // Final HTML
                SearchResultArr.forEach(function(data){
                    finalHTML += data.str;
                })
                $resultContent.innerHTML = finalHTML;
            })
        }
    })
}
```

其中打分排序标题中出现的关键词计30分，内容中的关键词一个计1分，总分越高意味着相关度越高，排在越前面。

----------

实际效果可直接使用文章左侧边栏上的搜索框进行体验，总体来说还是很令人满意的，至此终于搞定了站内搜索问题啦~~