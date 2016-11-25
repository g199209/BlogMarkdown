title: LeetCode 319. Bulb Switcher (Medium)
permalink: LeetCode_319
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-26 21:54:01

---


`n`盏灯，初始状态为全灭，第一次翻转编号为1的倍数的灯（即所有灯），第二次翻转编号为2的倍数的灯，以此类推，第`n`次翻转编号为`n`的倍数的灯。求最后有几盏灯亮着？

<!--more-->

## 原始问题

> https://leetcode.com/problems/bulb-switcher/
>
> There are n bulbs that are initially off. You first turn on all the bulbs. Then, you turn off every second bulb. On the third round, you toggle every third bulb (turning on if it's off or turning off if it's on). For the ith round, you toggle every i bulb. For the nth round, you only toggle the last bulb. Find how many bulbs are on after n rounds.
> 
> Example:
> 
> Given n = 3. 
> 
> At first, the three bulbs are [off, off, off].
> After first round, the three bulbs are [on, on, on].
> After second round, the three bulbs are [on, off, on].
> After third round, the three bulbs are [on, off, off]. 
> 
> So you should return 1, because there is only one bulb is on.


## 解题思路

简单分析就可以知道，问题的关键是要求解每盏灯被翻转了奇数次还是偶数次，翻转了奇数次的灯就是最后亮着的灯。最直接的做法就是从第1盏灯开始循环判断每一轮是否翻转，若翻转次数为奇数则亮灯数加1。此方法的代码见C语言程序的方法一。然而，这种方法运行时间太长，在LeetCode上会超时，故需要使用数学分析来解决这一问题。

我们要找到翻转次数为奇数的灯，可以先考虑这么一个问题，若第`m`轮翻转了第`i`盏灯，说明`i`肯定是`m`的倍数，即存在整数`n`满足`m*n=i`，这也就意味着在第`n`轮的时候这盏灯又会被再翻转一次。那什么时候会只翻转奇数次呢？仔细一想不难发现，只有`i`为完全平方数时才有可能，此时存在`m*m=i`，就会有一轮翻转没有与之对应的另一轮翻转，翻转次数就为奇数。

用数学语言来描述这个结论就是：**有且仅有完全平方数的因子个数为奇数。**所以问题简化为求小于`n`的完全平方数有几个，易知这个数就是`(int)(sqrt(n))`，这样我们就可以写出只有一行代码的算法了~见C语言程序的方法二。

## AC代码

### C语言 - 方法一

```c
int bulbSwitch(int n) {
    int times;
    int remains = 0;
    
    // 第i盏灯
    for (int i = 1; i <= n; i++) {
        times = 0;
        // 第j轮
        for (int j = 1; j <= i; j++) {
            if (i % j == 0) {
                times++;
            }
        }
        // 开关奇数次为开
        if (times & 0x01) {
            remains++;
        }
    }
    
    return remains;
}
```

此方法直接循环判断，在LeetCode上会超时。

### C语言 - 方法二

```c
int bulbSwitch(int n) {
    return sqrt(n);
}
```

## 相关题目

> [LeetCode 292. Nim Game (Easy)](/2016/10/26/LeetCode_292/)