title: LeetCode 014. Longest Common Prefix (Easy)
permalink: LeetCode_014
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-11-01 23:35:10

---

给出一个字符串数组，返回其最长公共前缀子序列。

<!--more-->

## 原始问题

> https://leetcode.com/problems/longest-common-prefix/
>
> Write a function to find the longest common prefix string amongst an array of strings.

## 解题思路

以第一个字符串为基准，依次比较每个字符即可。

## AC代码

### C语言

```c
char* longestCommonPrefix(char** strs, int strsSize) {
    int n, i;
    char tmp;
    char * res;
    
    if (strsSize == 0)
        return "";
    
    for(n = 0;;n++) {
        tmp = strs[0][n];
        if (tmp == '\0')
            break;
        for(i = 1; i < strsSize; i++) {
            if (strs[i][n] != tmp)
                break;
        }
        if (i < strsSize)
            break;
    }
    
    res = (char *)malloc(n+1);
    
    for (i = 0; i < n; i++) {
        res[i] = strs[0][i];
    }
    
    res[n] = '\0';
    
    return res;
}
```

这里先进行比较，判断出最先几个字符为相同的字符，之后再申请内存空间，复制字符串作为结果返回。之所以要分两步处理，是因为开始的时候并不知道结果字符串的长度，若采用动态分配空间的方法效率会很低。

### C++

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        string result = "";
        int n = strs.size();
        
        if (n == 0) {
            return result;
        }
        
        for (int pos = 0; pos < strs[0].size(); pos++) {
            for (int k = 1; k < n; k++) {
                if (pos == strs[k].size() || strs[0][pos] != strs[k][pos]) {
                    return result;
                }
            }
            result += strs[0][pos];
        }
        
        return result;
    }
};
```

