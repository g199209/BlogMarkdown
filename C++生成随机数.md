title: C++生成随机数
permalink: C++_Random
toc: false
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2017-03-22 15:42:55

---

之前写过一篇[C语言生成随机数](/2015/11/16/C%E8%AF%AD%E8%A8%80%E7%94%9F%E6%88%90%E9%9A%8F%E6%9C%BA%E6%95%B0/)，其中生成随机数的方法自然也可用于C++中，不过C++ 11扩展了生成随机数的方法，我们可以有更多更好的选择了。

<!--more-->

随机数的相关类声明都位于`<random>`头文件中，完整说明可参考：[random](http://www.cplusplus.com/reference/random/)，此处选择一些常用的总结一下。文末参考文献中那两篇知乎讨论很有趣，介绍了各种关于随机数的知识，很值得一读。

## 随机数生成器

一共提供了3种算法的生成器：线性同余法（LCG）、梅森旋转法、Lagged Fibonacci generator（LFG），其中梅森旋转法据说是最好的伪随机数生成算法，在C++中的具体实现类为`std::mt19937`和`std::mt19937_64`，前者是32位的，后者是64位的。

## 非确定性随机数生成设备

上面几个随机数生成器的本质都是从一个种子开始，使用某个递推方法生成伪随机数序列，其实还有种随机数生成策略就是利用物理世界中的各种噪声，以此产生所谓的真随机数。`std::random_device`就是这样一种机制，具体实现方法Linux和Windows不相同，以Linux为例就是读取`/dev/urandom`设备的输出得到的。这个设备中有一个容纳来自各种设备驱动噪声数据的熵池，具体可参考：

> [/dev/random](https://zh.wikipedia.org/wiki//dev/random)

`/dev/urandom`是`/dev/random`的非阻塞版本，会重复利用熵池，保证在熵池为空的时候也会有输出，这以降低随机数可靠性为代价提高了其生成速度。

## 生成特定分布的随机数

在C语言中，要生成特定分布的随机数并不容易，不过在C++中这就很简单了，C++中提供了各种常用分布的实现，直接调用就可以了。比如最常用的均匀分布的整数：`std::uniform_int_distribution`；均匀分布的小数：`std::uniform_real_distribution`；正态分布：`std::normal_distribution`等。

## 使用示例

最后举几个实际的例子来说明具体用法。

直接使用`random_device`的输出作为随机数：

``` C++
#include <iostream>
#include <random>

int main() {
	std::random_device rd;

	for (int i = 0; i < 100; i++) {
		std::cout << rd() << std::endl;
	}

	return 0;
}
```
此方法速度较慢，所以一般并不直接使用。

----------

以`random_device`的输出为初始种子，使用梅森旋转法生成[0,99]范围内的整数：

<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br></pre></td><td class="code"><pre><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;iostream&gt;</span></span></span><br><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;random&gt;</span></span></span><br><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;functional&gt;</span></span></span><br><span class="line"></span><br><span class="line"><span class="function"><span class="keyword">int</span> <span class="title">main</span><span class="params">()</span> </span>&#123;</span><br><span class="line">    <span class="built_in">std</span>::random_device rd;</span><br><span class="line">    <span class="built_in">std</span>::mt19937 mt(rd());</span><br><span class="line">    <span class="built_in">std</span>::uniform_int_distribution&lt;<span class="keyword">int</span>&gt; dist(<span class="number">0</span>, <span class="number">99</span>);</span><br><span class="line">    <span class="keyword">auto</span> dice = <span class="built_in">std</span>::bind(dist, mt);</span><br><span class="line"></span><br><span class="line">    <span class="keyword">for</span> (<span class="keyword">int</span> i = <span class="number">0</span>; i &lt; <span class="number">100</span>; i++) &#123;</span><br><span class="line">        <span class="built_in">std</span>::<span class="built_in">cout</span> &lt;&lt; dice() &lt;&lt; <span class="built_in">std</span>::endl;</span><br><span class="line">    &#125;</span><br><span class="line"></span><br><span class="line">    <span class="keyword">return</span> <span class="number">0</span>;</span><br><span class="line">&#125;</span><br></pre></td></tr></table></figure>

使用`std::bind()`是为了让之后调用起来更方便些。此方法就是生成某个区间内均匀分布的整数随机数最常用的方法了，注意区间两端都是闭区间。

----------

以`random_device`的输出为初始种子，使用梅森旋转法生成[0.0, 1.0)范围内的小数：

<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br></pre></td><td class="code"><pre><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;iostream&gt;</span></span></span><br><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;random&gt;</span></span></span><br><span class="line"><span class="preprocessor">#<span class="keyword">include</span> <span class="string">&lt;functional&gt;</span></span></span><br><span class="line"></span><br><span class="line"><span class="function"><span class="keyword">int</span> <span class="title">main</span><span class="params">()</span> </span>&#123;</span><br><span class="line">    <span class="built_in">std</span>::random_device rd;</span><br><span class="line">    <span class="built_in">std</span>::mt19937 mt(rd());</span><br><span class="line">    <span class="built_in">std</span>::uniform_real_distribution&lt;<span class="keyword">double</span>&gt; dist(<span class="number">0.0</span>, <span class="number">1.0</span>);</span><br><span class="line">    <span class="keyword">auto</span> dice = <span class="built_in">std</span>::bind(dist, mt);</span><br><span class="line"></span><br><span class="line">    <span class="keyword">for</span> (<span class="keyword">int</span> i = <span class="number">0</span>; i &lt; <span class="number">100</span>; i++) &#123;</span><br><span class="line">        <span class="built_in">std</span>::<span class="built_in">cout</span> &lt;&lt; dice() &lt;&lt; <span class="built_in">std</span>::endl;</span><br><span class="line">    &#125;</span><br><span class="line"></span><br><span class="line">    <span class="keyword">return</span> <span class="number">0</span>;</span><br><span class="line">&#125;</span><br></pre></td></tr></table></figure>

生成[0, 1)区间内任意小数最常用的方法，注意这里是左闭右开区间，不包含1.0的。

----------

> 参考资料：
> [C++11带来的随机数生成器](http://www.cnblogs.com/egmkang/archive/2012/09/06/2673253.html)
> [C++ 如何生成大随机数？](https://www.zhihu.com/question/24297923)
> [电脑取随机数是什么原理，是真正的随机数吗？](https://www.zhihu.com/question/20423025)