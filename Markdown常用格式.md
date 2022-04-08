title: Markdown常用格式
weburl: Markdown常用格式
date: 2015-10-30 14:34:23
tags: [Blog]
mathjax: true
categories: 工具之术

---

根据以下文章整理得到：
> [markdown简明语法](http://ibruce.info/2013/11/26/markdown/)
> [献给写作者的 Markdown 新手指南](http://www.jianshu.com/p/q81RER)
> [Markdown语法说明（详解版）](http://www.ituring.com.cn/article/504)
> [BAZINGA的博客](http://caoyudong.com/2015/07/15/MacDown/)

不同Markdown解析器的行为可能会有所区别，支持的标签也不完全相同，本文根据Hexo所附带的解析器进行讨论。


<!--more-->

## 段落
一个段落是由一个以上相连接的行句组成，而一个以上的空行则会切分出不同的段落（空行的定义是显示上看起来像是空行，便会被视为空行。比如，若某一行只包含空白和 tab，则该行也会被视为空行）。

## 标题
使用1到6个`#`，对应1至6级标题。
```markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

## 分割线
使用3个以上`_`表示。
```markdown
______________________
```
下面是一条分割线
_____________________
上面是一条分割线

## 列表
### 无序列表
使用`-`，注意，`-`后有一个空格，不能省略。
```markdown
- 列表项1
- 列表项2
- 列表项3
```
- 列表项1
- 列表项2
- 列表项3

### 嵌套列表
交替使用`-` `+`，并使用空格进行缩进。
*注：Hexo似乎只支持两级嵌套。*
```markdown
- 嵌套列表1
 + 嵌套列表11
 + 嵌套列表12
- 嵌套列表2
```
- 嵌套列表1
 + 嵌套列表11
 + 嵌套列表12
- 嵌套列表2

### 有序列表
使用`1.` `2.` `3.`，注意，之后要有一个空格。
```markdown
1. 列表项1
2. 列表项2
3. 列表项3
```
1. 列表项1
2. 列表项2
3. 列表项3

## 引用
使用`>`表示。
```markdown
> 这是一条引用。
```
> 这是一条引用。

## 超链接
使用`[显示文本](链接地址)`这样的语法即可。
```markdown
[浙江大学](http://www.zju.edu.cn)
[上海交通大学](http://www.sjtu.edu.cn)
```
[浙江大学](http://www.zju.edu.cn)
[上海交通大学](http://www.sjtu.edu.cn)
__________________________________
也可使用尖括号，会自动处理链接形式。
```markdown
<http://gaomingfei.xyz>
<g199209@163.com>
```
<http://gaomingfei.xyz>
<g199209@163.com>

## 图片
使用`![](图片链接地址)`这样的语法即可。
```markdown
![](https://img.gaomf.cn/Octocat.png)
```
![](https://img.gaomf.cn/Octocat.png?300x)

## 粗体和斜体
使用2个`*`包含一段文本代表粗体，使用1个`*`包含一段文本代表斜体。
```markdown
这是正常文本，**这是粗体文本**，*这是斜体文本*。
```
这是正常文本， **这是粗体文本**， *这是斜体文本*。

## 表格
参见以下代码段，使用`:`来确定对齐方式。
```markdown
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

## 代码
### 行内代码
使用<code>\`</code>包含代码片段即可。

### 代码段
使用<code>\`\`\`</code>包含代码片段即可，第一行的<code>\`\`\`</code>后面可以指定使用的编程语言，一般建议手动指定，常用语言列表如下：

|Language Name|Alias|Language Name|Alias|
|-------------|-----|-------------|-----|
|ARM assembler|armasm, arm|AVR assembler|avrasm|
|Awk|awk, mawk, nawk, gawk|Bash|bash, sh, zsh|
|Basic|basic|C#|cs, csharp|
|C, C++|cpp, c, cc, c++|CMake|cmake|
|CSS|css|DOS|dos, bat, cmd|
|Delphi, Pacal|delphi, pascal|Diff|diff|
|Django|django|DTS (Device Tree)|dts|
|Excel|excel|Fortran|fortran|
|Go|go|Gradle|gradle|
|Groovy|groovy|HTML, XML|xml, html, xhtml, rss|
|HTTP|http|JSON|json|
|Java|java|JavaScript|javascript, js|
|Lisp|lisp|Lua|lua|
|Makefile|makefile, mk, mak|Markdown|markdown, md|
|Matlab|matlab|PHP|php|
|Perl|perl|Processing|processing|
|Prolog|prolog|Python|python, py|
|R|r|Ruby|ruby|
|SQL|sql|Swift|swift|
|Tcl|tcl|TeX|tex|
|VB.Net|vbnet, vb|VBScript|vbscript, vbs|
|VHDL|vhdl|Verilog|verilog, v|
|Vim Script|vim|x86 Assembly|x86asm|

完整的帮助见：[CSS classes reference](http://highlightjs.readthedocs.io/en/latest/css-classes-reference.html)。如果不需要高亮，使用`no-highlight`

## 公式
需要[Mathjax](https://www.mathjax.org/)支持，详细说明可参考其[文档](http://mathjax.readthedocs.org/en/latest/)。
Hexo本身不直接支持Mathjax，不过本博客系统使用的[Yilia](https://github.com/litten/hexo-theme-yilia)主题添加了Mathjax支持，故不需要做其它配置即可直接使用。Mathjax公式常用语法可参考:

> [Mathjax与LaTex公式简介](http://3iter.com/2015/10/14/Mathjax%E4%B8%8ELaTex%E5%85%AC%E5%BC%8F%E7%AE%80%E4%BB%8B/)

使用`$`表示行内公式，使用`$$`表示整行公式，其中使用Latex格式来编辑公式，推荐一个比较好用的在线Latex公式可视化编辑器：[CodeCogs](https://www.codecogs.com/latex/eqneditor.php?lang=zh-cn)。下面是一些公式的示例效果。
____________________________________
```markdown
著名的质能守恒方程为：$E=mc^2$。其中$E$代表能量，$m$代表质量，$c$代表光速
```
著名的质能守恒方程为：$E=mc^2$。其中$E$代表能量，$m$代表质量，$c$代表光速
____________________________________

```markdown
同样很著名的麦克斯韦方程组为
$$\nabla\cdot\vec{B}=0$$
$$\nabla\cdot\vec{D}=\rho$$
$$\nabla\times\vec{H}=\vec{J} + \frac{\partial \vec{D}}{\partial t}$$
$$\nabla\times\vec{E}=- \frac{\partial \vec{B}}{\partial t}$$
此方程组由两个散度方程和两个旋度方程组成，是麦克斯韦方程组的微分形式，可转化为对应的积分形式：
$$\oint\_{S} \vec{B}\cdot\mathrm{d}\vec{S}=0$$
$$\oint\_{S} \vec{B}\cdot\mathrm{d}\vec{S}=\int\_{V}\rho \mathrm{d}V$$
$$\oint\_{l} \vec{H}\cdot \mathrm{d}\vec{l}=\int\_{S}(\vec{J}+\frac{\partial \vec{D}}{\partial t})\cdot\mathrm{d}\vec{S}$$
$$\oint\_{l} \vec{E}\cdot \mathrm{d}\vec{l}=-\int\_{S}\frac{\partial \vec{B}}{\partial t}\cdot\mathrm{d}\vec{S}$$
```
同样很著名的麦克斯韦方程组为
$$\nabla\cdot\vec{B}=0$$
$$\nabla\cdot\vec{D}=\rho$$
$$\nabla\times\vec{H}=\vec{J} + \frac{\partial \vec{D}}{\partial t}$$
$$\nabla\times\vec{E}=- \frac{\partial \vec{B}}{\partial t}$$

以上4个方程由两个散度方程和两个旋度方程组成，是麦克斯韦方程组的微分形式，可转化为对应的积分形式:

$$\oint\_{S} \vec{B}\cdot\mathrm{d}\vec{S}=0$$
$$\oint\_{S} \vec{B}\cdot\mathrm{d}\vec{S}=\int\_{V}\rho \mathrm{d}V$$
$$\oint\_{l} \vec{H}\cdot \mathrm{d}\vec{l}=\int\_{S}(\vec{J}+\frac{\partial \vec{D}}{\partial t})\cdot\mathrm{d}\vec{S}$$
$$\oint\_{l} \vec{E}\cdot \mathrm{d}\vec{l}=-\int\_{S}\frac{\partial \vec{B}}{\partial t}\cdot\mathrm{d}\vec{S}$$
____________________________________
需要注意的是，正如[这篇文章](http://hijiangtao.github.io/2014/09/08/MathJaxinHexo/)所说，在书写MathJax公式的时候有时候会出现一些问题，主要是因为Markdown会将一些标记给编译掉，所以`_`、`{}`和`\\`等符号有时会出现问题，解决方式是在前面加上\进行转义。

## HTML标签
文本中可直接用html标签，不需要额外标注这是 HTML 或是 Markdown，只要直接加标签就可以了。
只有区块元素──比如 `<div>`、`<table>`、`<pre>`、`<p>` 等标签，必需在前后加上空白，以利与内容相区分。
比如以下代码段，可以居中放置一张图片，其中url代表图片地址。

```html
<div style="text-align: center">
<img src="url"/>
</div>
```

Html标签另外一个比较有用的作用是可以方便的直接插入一些符号，避免被Markdown解释器错误的解释。



















