title: C++中std::string的COW及SSO实现
weburl: Cpp_string_COW_SSO
toc: false
mathjax: false
fancybox: false
tags: [C++, Performance, Top]
categories: 软件之道
date: 2017-07-26 23:31:41

---

在牛客网上看到一题字符串拷贝相关的题目，深入挖掘了下才发现原来C++中string的实现还是有好几种优化方法的，这里简单记录一下。

<!--more-->

原始题目是这样的：

> 关于代码输出正确的结果是()（Linux g++ 环境下编译运行）
> 
```cpp
int main(int argc, char *argv[])
{
	string a="hello world";
    string b=a;
    if (a.c_str()==b.c_str())
    {
        cout<<"true"<<endl;
    }
    else cout<<"false"<<endl;
    string c=b;
    c="";
    if (a.c_str()==b.c_str())
    {
        cout<<"true"<<endl;
    }
    else cout<<"false"<<endl;
    a="";
    if (a.c_str()==b.c_str())
    {
        cout<<"true"<<endl;
    }
    else cout<<"false"<<endl;
    return 0;
 }
```

这段程序的输出结果和编译器有关，在老版本(5.x之前)的GCC上，输出是`true true false`，而在VS上输出是`false false false`。这是由于不同STL标准库对`string`的实现方法不同导致的。

简而言之，目前各种STL实现中，对`string`的实现有两种不同的优化策略，即COW(Copy On Write)和SSO(Small String Optimization)。`string`也是一个类，类的拷贝操作有两种策略——深拷贝及浅拷贝。我们自己写的类默认情况下都是浅拷贝的，可以理解为指针的复制，要实现深拷贝需要重载赋值操作符或拷贝构造函数。不过对于`string`来说，大部分情况下我们用赋值操作是想实现深拷贝的，故所有实现中`string`的拷贝均为深拷贝。

最简单的深拷贝就是直接new一个对象，然后把数据复制一遍，不过这样做效率很低，STL中对此进行了优化，基本策略就是上面提到的COW和SSO。

### COW

所谓COW就是指，复制的时候不立即申请新的空间，而是把这一过程延迟到写操作的时候，因为在这之前，二者的数据是完全相同的，无需复制。这其实是一种广泛采用的通用优化策略，它的核心思想是懒惰处理多个实体的资源请求，在多个实体之间共享某些资源，直到有实体需要对资源进行修改时，才真正为该实体分配私有的资源。

COW的实现依赖于引用计数（reference count, `rc`），初始时`rc=1`，每次赋值复制时`rc++`，当修改时，如果`rc>1`，需要申请新的空间并复制一份原来的数据，并且`rc--`，当`rc==0`时，释放原内存。

不过，实际的`string`COW实现中，对于什么是"写操作"的认定和我们的直觉是不同的，考虑以下代码：

```cpp
string a = "Hello";
string b = a;
cout << b[0] << endl;
```

以上代码显然没有修改`string b`的内容，此时似乎`a`和`b`是可以共享一块内存的，然而由于`string`的`operator[]`和`at()`会返回某个字符的引用，此时无法准确的判断程序是否修改了`string`的内容，为了保证COW实现的正确性，`string`只得统统认定`operator[]`和`at()`具有修改的“语义”。

这就导致`string`的COW实现存在诸多弊端（除了上述原因外，还有线程安全的问题，可进一步阅读文末参考资料），因此只有老版本的GCC编译器和少数一些其他编译器使用了此方式，VS、Clang++、GCC 5.x等编译器均放弃了COW策略，转为使用SSO策略。

### SSO

SSO策略中，拷贝均使用立即复制内存的方法，也就是深拷贝的基本定义，其优化在于，当字符串较短时，直接将其数据存在栈中，而不去堆中动态申请空间，这就避免了申请堆空间所需的开销。

使用以下代码来验证一下：

```cpp
int main() {
	string a = "aaaa";
	string b = "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb";

	printf("%p ------- %p\n", &a, a.c_str());
	printf("%p ------- %p\n", &b, b.c_str());

	return 0;
}
```

某次运行的输出结果为：

```no-highlight
0136F7D0 ------- 0136F7D4
0136F7AC ------- 016F67F0
```

可以看到，`a.c_str()`的地址与`a`、`b`本身的地址较为接近，他们都位于函数的栈空间中，而`b.c_str()`则离得较远，其位于堆中。

SSO是目前大部分主流STL库的实现方式，其优点就在于，对程序中经常用到的短字符串来说，运行效率较高。

----------

> 参考资料：
> [std::string的Copy-on-Write：不如想象中美好](http://www.cnblogs.com/promise6522/archive/2012/03/22/2412686.html)
> [C++ 工程实践(10)：再探std::string](http://www.cnblogs.com/Solstice/archive/2012/03/17/2403335.html)
> [Why is COW std::string optimization still enabled in GCC 5.1?](https://stackoverflow.com/questions/31228579/why-is-cow-stdstring-optimization-still-enabled-in-gcc-5-1)
> [ C++ string的COW和SSO](http://blog.csdn.net/kemaWCZ/article/details/52709747)
