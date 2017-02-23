title: diff及patch的使用
permalink: Diff_Patch
toc: true
mathjax: false
fancybox: false
tags: [Linux, 工具]
categories: 嵌入式
date: 2016-10-19 00:42:01

---

Linux下，`diff`命令用于比较两个文件的差异，也可用于创建补丁文件，而`patch`命令用于打补丁，也就是应用`diff`命令生成的补丁文件更新现有文件。下面简要总结下这两个命令的使用方法。

<!--more-->

## diff

### 基本用法
```bash
diff oldfile newfile
```

注意：`diff`命令是**按行**比较两个文本文件的，所以一般只用它比较两个基本相同的文本文件，如同一份源代码的不同版本。

### 输出格式
`diff`一共有3种输出格式：

- 正常格式（normal diff）
- 上下文格式（context diff）
- 合并格式（unified diff）

一般使用合并格式，这种格式比较适合阅读，加上`-u`参数即可：

```bash
diff -u f1 f2
```

关于`diff`的输出格式详细说明可参考：

> [读懂diff](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html)

### 创建补丁文件
所谓的补丁文件其实就是`diff`的输出，将其重定向至文件即可，生成的补丁文件习惯上使用`.patch`后缀。

常用参数如下：

```bash
diff -ruaN oldfile newfile > patchfile.patch
```

其中：

- -r: 递归遍历子目录
- -u: 使用合并格式输出
- -a: 将所有文件视为文本文件
- -N: 将未出现的文件视为空文件（比较目录时有用）

`oldfile`及`newfile`可以是一个目录，配合`-r`参数使用就会递归比较这两个目录。

### 示例

下面用一个例子来说明。

有以下文件目录：

```no-highlight
.
├── new
│   ├── 1.txt
│   └── 2.txt
└── old
    ├── 1.txt
    └── 2.txt
```

`old`文件夹中存放的是老版本，`new`中是新版本。使用以下命令生成补丁文件：

```bash
diff -ruaN old new > update.patch
```

生成的`update.patch`文件如下：

```diff
diff -aruN old/1.txt new/1.txt
--- old/1.txt   2016-10-19 00:12:51.572014429 +0800
+++ new/1.txt   2016-10-15 12:10:18.581633573 +0800
@@ -1,3 +1,3 @@
-old Line 1
-old Line 2
-old Line 3
+new Line 1
+new Line 2
+new Line 3
diff -aruN old/2.txt new/2.txt
--- old/2.txt   2016-10-19 00:12:51.572014429 +0800
+++ new/2.txt   2016-10-15 12:11:03.847494158 +0800
@@ -1,4 +1,4 @@
-Old Line 1
-Old Line 2
-Old Line 3
+New Line 1
+New Line 2
+New Line 3
```

----------

需要注意的是，使用如下命令也可以生成补丁文件：

```diff
diff -aruN ./old ./new > update2.patch
```

但是生成的补丁文件是不一样的：

```diff
diff -aruN ./old/1.txt ./new/1.txt
--- ./old/1.txt 2016-10-19 00:12:51.572014429 +0800
+++ ./new/1.txt 2016-10-15 12:10:18.581633573 +0800
@@ -1,3 +1,3 @@
-old Line 1
-old Line 2
-old Line 3
+new Line 1
+new Line 2
+new Line 3
diff -aruN ./old/2.txt ./new/2.txt
--- ./old/2.txt 2016-10-19 00:12:51.572014429 +0800
+++ ./new/2.txt 2016-10-15 12:11:03.847494158 +0800
@@ -1,4 +1,4 @@
-Old Line 1
-Old Line 2
-Old Line 3
+New Line 1
+New Line 2
+New Line 3
```

差异在于**文件路径不同**，这会导致之后使用`patch`打补丁时所需的命令有所区别，下文再讨论这一问题。

## patch

### 应用补丁
```bash
patch < patchfile
```

一般不需要指定需要打补丁的文件，这个可以从`patchfile`中自动推导出来，就是文件中`---`部分的`oldfile`，`patch`命令会自动将`newfile`中的变更应用于`oldfile`上。

### 回滚补丁
```bash
patch -R < patchfile
```

回滚补丁的意思是在对`oldfile`应用了补丁后，又需要取消这个补丁，即回滚到`oldfile`原始的状态。

### 移去路径级别

上面的用法中均是假定`patchfile`中的路径是正确的，然而很多时候这个路径会包含不需要的目录级别，这时就需要用`-pNum`参数来移去多余的路径级别：

```bash
patch -pNum < patchfile
```

这里的`Num`是一个数字，如`-p1`、`-p2`等。p级别告诉`patch`命令忽略掉路径名的前几个部分以正确的识别文件，总的来说，**从路径最开始删除路径分隔符（`/`）及其之前的所有字符，每次加1，直到剩下的部分存在于当前工作目录中，最后得到的就是p级别**。 具体使用方法见下文的示例。

### 示例

接着上文`diff`命令中的示例进行，此时我们已经有了补丁文件`update.patch`，下面来应用这个补丁，使用以下命令即可：

```bash
patch < update.patch
```

这样既可将`old`文件夹中的内容更新为`new`文件夹中的内容。

如果需要回滚补丁，即恢复`old`文件夹中的原始内容，使用以下命令：

```bash
patch -R < update.patch
```

----------

不过针对`update2.patch`文件，如果直接使用上述命令打补丁会报错：

```bash
can't find file to patch at input line 4
Perhaps you should have used the -p or --strip option?
The text leading up to this was:
--------------------------
|diff -aruN ./old/1.txt ./new/1.txt
|--- ./old/1.txt        2016-10-19 00:12:51.572014429 +0800
|+++ ./new/1.txt        2016-10-15 12:10:18.581633573 +0800
--------------------------
File to patch: 
```

此时可以手动输入对应文件的路径`./old/1.txt`，然而这并不是一个好方法，正确的方法是使用`-pNum`参数：

```bash
patch -p1 < update2.patch
```

`-p1`表示去掉第一层路径分隔符，即`./`，此时得到的`old`文件夹位于当前路径中，于是就可以正确的打补丁了。