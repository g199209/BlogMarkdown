title: LeetCode 292. Nim Game (Easy)
permalink: LeetCode_292
toc: true
mathjax: false
fancybox: false
tags: [Online Judge]
categories: 算法之美
date: 2016-10-26 20:57:34

---

一堆石子，两个人依次从中拿1~3个，最后一个全部拿完的人获胜。给出石子数，请问先拿那个人在这种情况下是否能获胜？

<!--more-->

## 原始问题

> https://leetcode.com/problems/nim-game/
>
> You are playing the following Nim Game with your friend: There is a heap of stones on the table, each time one of you take turns to remove 1 to 3 stones. The one who removes the last stone will be the winner. You will take the first turn to remove the stones.
> 
> Both of you are very clever and have optimal strategies for the game. Write a function to determine whether you can win the game given the number of stones in the heap.
> 
> For example, if there are 4 stones in the heap, then you will never win the game: no matter 1, 2, or 3 stones you remove, the last stone will always be removed by your friend.


## 解题思路

这题可以算一个智力题，通过简单的逻辑分析可知，当且仅当石子的数量不为4的倍数时，先拿者可以获胜，具体推理过程在此就不多说了。这样就可以写出很简洁的Java版程序。

另外这题还可以用递归来解决，基本就相当于穷举所有情况，C语言版的代码就是用这种方法实现的。只是这种方法实际上是不可用的，计算时间会随`n`的增加而指数增加，当`n=50`时就需要超过10分钟才能计算出结果。

最好的通用方法是使用动态规划来解决这一问题，记录每次递归的中间结果即可。

## AC代码

### C语言

```c
bool canWinNim(int n) {
    if (n < 4)
        return true;
    
    return !(canWinNim(n -1) && canWinNim(n-2) && canWinNim(n-3));
}
```

### Java

```java
public class Solution {
    public boolean canWinNim(int n) {
        return n % 4 != 0;
    }
}
```

## 相关题目

> [LeetCode 319. Bulb Switcher (Medium)](/2016/10/26/LeetCode_319/)
