title: LeetCode 344. Reverse String (Easy)
permalink: LeetCode_344
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-25 19:29:19

---

反转一个字符串。

<!--more-->

## 原始问题

> https://leetcode.com/problems/reverse-string/
>
> Write a function that takes a string as input and returns the string reversed.
> 
> Example:
> Given s = "hello", return "olleh".


## 解题思路

直接反向复制即可。

## AC代码

### C语言

```c
char* reverseString(char* s) {
     int n = strlen(s);
     char * r = (char *)malloc((n+1) * sizeof(char));
     
     for (int i = 0;i < n;i++) {
         r[i] = s[n - i - 1];
     }
     r[n] = '\0';

     return r;
}
```

## 相关题目

> [LeetCode 345. Reverse Vowels of a String (Easy)](/2016/10/25/LeetCode_345/)
