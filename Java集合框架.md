title: Java集合框架
date: 2016-09-16 21:48:45
tags: [Java, Top]
categories: 编程之法

---

Java中提供了丰富的容器类用于存储数据，这些容器类可分为两大类：Collection和Map，Collection用于保存单个元素，而Map则以键值对的形式进行存储，就像一个小型数据库一样。Collection中又可分为List、Set、Queue三类，其中List是列表，Set是集合，Queue是队列。整个Java集合框架的结构图如下：

![](https://gmf.shengnengjin.cn/Java%20Collections%20Framework2.png)

<!--more-->

网上能找到的图都不是很全面，故我根据最新版的JDK 8文档绘制了此图。图中并没有包含所有的接口与类，仅仅只是最常用的部分，这些部分均位于`java.util`包中。图中蓝色的实现类就是我们实际可以直接使用的容器类了。

JDK帮助文档中还有一张各接口与存储形式的对照表，摘录如下：

|Interface|Hash Table|Resizable Array|Balanced Tree|Linked List|Hash Table + Linked List|
|---------|----------|---------------|-------------|-----------|--------------------------|
|**Set**|**`HashSet`**|-|**`TreeSet`**|-|`LinkedHashSet`|
|**List**|-|**`ArrayList`**|-|**`LinkedList`**|-|
|**Deque**|-|**`ArrayDeque`**|-|`LinkedList`|-|
|**Map**|**`HashMap`**|-|**`TreeMap`**|-|`LinkedHashMap`|

上表中加粗显示的就是最常用的实现类，一般情况下选择这几个就可以了。

接下来将各接口最常用的一些方法整理总结一下。

## **Collection接口** ##
所有的`List`、`Set`、`Queue`均支持以下方法：

|Function Name|
|--|--|--|
|`size()`|`isEmpty()`|`contains()`|
|`add()`|`remove()`|`iterator()`|
|`toArray()`|`clear()`|`forEach()`|

## **Iterator接口** ##

|Function Name|
|--|--|--|
|`hasNext()`|`next()`|`remove()`|

## **List接口** ##
在`Collection`接口的基础上添加了以下方法：

|Function Name|
|--|--|
|`get()`|`set()`|
|`add(int,E)`|`remove(int)`|
|`indexOf()`|`lastIndexOf()`|

## **ListIterator接口** ##
在`Iterator`接口的基础上添加了以下方法：

|Function Name|
|--|--|
|`add()`|`set()`|
|`hasPrevious()`|`previous()`|

## **Set接口** ##
`Set`接口没有添加新方法，与`Collection`接口中的相同。

添加到`Set`中的类一般都要覆盖实现`hashCode()`及`equals()`函数。

## **SortedSet接口** ##
在`Set`接口的基础上添加了以下方法：

|Function Name|
|--|--|--|
|`subSet()`|`headSet()`|`tailSet()`|
|`first()`|`last()`|

添加到`SortedSet`中的类需要实现`Comparable`接口。

## **Deque接口** ##
在`Collection`接口的基础上添加了以下方法：

|Function Name|
|--|--|
|`addFirst()`|`addLast()`|
|`removeFirst()`|`removeLast()`|
|`getFirst()`|`getLast()`|

## **Map接口** ##
常用函数有：

|Function Name|
|--|--|--|
|`put()`|`get()`|`remove()`|
|`size()`|`isEmpty()`|
|`containsKey()`|`containsValue()`|
|`keySet()`|`values()`|

添加到`Map`中的Key类一般都要覆盖实现`hashCode()`及`equals()`函数。

## **SortedMap接口** ##
在`Map`接口的基础上添加了以下方法：

|Function Name|
|--|--|--|
|`subMap()`|`headMap()`|`tailMap()`|
|`firstKey()`|`lastKey()`|

`SortedMap`是按照Key来进行排序的，要求Key需要实现`Comparable`接口。

## **Arrays类** ##
用于支持数组操作，常用静态函数有：

|Function Name|
|--|--|--|
|`asList()`|`binarySearch()`|`copyOf()`|
|`equals()`|`fill()`|`sort()`|

## **Collections类** ##
用于支持集合框架的通用算法，常用静态函数有：

|Function Name|
|--|--|--|
|`sort()`|`binarySearch()`|`copy()`|
|`shuffle()`|`fill()`|`reverse()`|

----------

关于各种容器类的具体介绍与使用方法，可参考官方的Tutorial：[Trail: Collections](http://docs.oracle.com/javase/tutorial/collections/index.html)

以下几个链接也可供参考：

> [Java 集合框架](http://www.runoob.com/java/java-collections.html)
> [Java集合类: Set、List、Map、Queue使用场景梳理](http://www.cnblogs.com/LittleHann/p/3690187.html)
> [Java集合类详解](http://blog.csdn.net/softwave/article/details/4166598)
