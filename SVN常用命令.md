title: SVN常用命令
permalink: SVN
toc: true
mathjax: false
fancybox: false
tags: [工具, Linux]
categories: 杂七杂八
date: 2017-03-30 16:12:08

---

本文总结一下常用的SVN命令，以便之后查阅。

<!--more-->

### 检出

``` bash
svn checkout [URL] --username [username]
svn co [URL] --username [username]
```

其中`[URL]`是服务器上的路径，如`svn://xxxx.com/repo`，此命令会把服务器上的内容在**当前**本地目录中检出，`co`是`checkout`的简写。使用`--username`参数附加用户名，之后会提示输入密码。

### 更新

``` bash
svn update [PATH]
svn up [PATH]
```

使用远程版本库中的文件更新本地版本库及工作副本，可以使用`[PATH]`指定更新哪个路径，如不指定的话，递归的更新**当前目录及其子目录**。会输出更新条目的状态：

- A : 已将一个文件添加到工作副本中
- U : 已更新工作副本中的一个文件
- D : 已从工作副本中删除一个文件
- R : 已替换工作副本中的一个文件
- G : 已成功合并了一个文件
- C : 一个文件已合并了,但必须手动解决冲突

----------

``` bash
svn list [-v] [PATH]
```

上述`svn update`命令会直接更新当前工作副本，若需要先确定一下哪些文件有更改了，可使用`svn list -v`命令，会列出远程版本库`[PATH]`下各文件的版本号，修改人，修改日期等信息，若没有`-v`参数则只有文件名。

### 查看状态

``` bash
svn status [-v] [PATH]
svn st [-v] [PATH]
```

查看工作副本的状态，如省略`[PATH]`的话，显示**当前路径**的状态。与版本库中相同则不显示，`?`代表不在版本库中，`D`代表删除，`M`代表有修改，`A`代表添加，`C`代表有冲突。如果附加上`[-v]`参数，会列出所有文件，第一列表示状态，第二列是版本号，第三和第四列是此文件最后一次修改的版本号及修改人。

----------

``` bash
svn info
```

查看SVN仓库的相关信息，如URL、上次修改时间等。


### 添加、删除及提交

``` bash
svn add [PATH]
```

第一次在工作目录中创建并编辑新文件后，需要使用此命令将此文件添加到版本库中，`[PATH]`可以是一个文件也可以是一个目录，可配合`*`通配符使用。

----------

``` bash
svn delete [PATH]
svn rm [PATH]
```

如果需要将版本库中的某个文件或目录删除，使用此命令，直接用`rm`删除那个文件是无效的，下次`update`的时候又会回来的。

----------

``` bash
svn commit -m ""
svn ci -m ""
```

修改、添加、删除文件后，仅仅是影响了本地副本，还没有将其提交到版本库中，使用此命令提交更改。注意，附加参数`-m`是这次提交的说明，这是必须的，可以为空`""`，但不能没有。另外，SVN与Git不同，不能只更新本地版本库而不将其上传到远程版本库上。

### 撤销更改

``` bash
svn revert [-R] [filename]
```

对本地副本进行了某些修改，但还没有使用`svn ci`提交之前，若想丢弃这些更改，可以使用此命令。`filename`可以是一个目录，此时需加上`-R`参数递归的进行处理。

### 查看日志

``` bash
svn log [-v] [-l [N]]
```

列出提交日志，使用`-l [N]`参数指定只列出最近`N`次的提交记录，否则列出全部记录；`-v`参数会列出每次提交的具体条目细节。

### 比较差异

``` bash
svn diff [-r m[:n]] [PATH]
```

比较两个版本间的差异，使用`-r m:n`参数比较版本`m`和版本`n`之间的差异，使用`-r m`参数比较当前工作副本的文件和版本`m`之间的差异，不附加此参数比较当前工作副本文件和版本库中最新版本间的差异。

SVN默认的diff是没有颜色的，看起来很不方便，可以通过安装`colordiff`来解决这一问题：

> [在ubuntu下，给 svn diff 一点颜色](http://www.cnblogs.com/pylemon/archive/2012/06/29/2569940.html)

### 回滚

``` bash
svn merge [-r [n]] [PATH]
```

实际使用时，先用`svn log`命令找出需要回滚到的版本，必要时使用`svn diff`进行比较确认；之后使用`svn merge -r n`命令将版本`n`合并到当前副本，此处的`[PATH]`可以是文件也可以是目录，若省略依然代表当前目录；最后，如果要确认这次回滚，还要使用`svn commit`将其重新提交到版本库中。


### 加锁、解锁

``` bash
svn lock -m "" [--force] [PATH]
svn unlock [PATH]
```

----------

> 参考资料：
> [使用命令行 Subversion 访问项目源文件](https://www.open.collab.net/scdocs/ddUsingSVN_command-line.html.zh-cn)
> [SVN常用命令](http://blog.csdn.net/ithomer/article/details/6187464)
> [SVN：取消对代码的修改](http://blog.csdn.net/rangf/article/details/7408230)