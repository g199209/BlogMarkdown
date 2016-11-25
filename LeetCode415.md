title: LeetCode 415. Add String (Easy)
permalink: LeetCode_415
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-24 14:50:19

---

给定两个字符串表示的非负整数，求其和，结果也用字符串表示。

<!--more-->

## 原始问题

> https://leetcode.com/problems/add-strings/
>
> Given two non-negative numbers `num1` and `num2` represented as string, return the sum of `num1` and `num2`.
> 
> Note:
> 
> The length of both `num1` and `num2` is < 5100.
> Both `num1` and `num2` contains only digits `0-9`.
> Both `num1` and `num2` does not contain any leading zero.
> You **must not use any built-in BigInteger library** or **convert the inputs to integer** directly.

## 解题思路

从最低位开始逐位相加即可，注意处理进位。网上搜索到的AC代码处理进位一般都使用`/10`及`%10`的方法：

```C
c = sum / 10;
result += sum % 10 + '0';
```

此方法代码简洁，不过效率很低，因为除法和取模开销会很大，故此处使用了其他方法进行处理，具体见下面的代码。

## AC代码

### C语言

```C
#include <stdlib.h>
#include <string.h>

char* addStrings(char* num1, char* num2) {
    int digits1 = strlen(num1);
    int digits2 = strlen(num2);
    int sumDigits = digits1 > digits2 ? digits1 + 1 : digits2 + 1;
    
    char * sumStr = (char *)malloc((sumDigits + 1) * sizeof(char));
    sumStr[sumDigits] = '\0';
    
    char carryFlag = 0;
    int tmpNum = 0;
    int i;
    for (i = 1; i <= sumDigits; i++) {
        tmpNum = carryFlag;
        if (i <= digits1) {
            tmpNum += (num1[digits1 - i] - '0');
        }
        if (i <= digits2) {
            tmpNum += (num2[digits2 - i] - '0');
        }
        
        if (tmpNum > 9) {
            tmpNum -= 10;
            carryFlag = 1;
        } else {
            carryFlag = 0;
        }
        
        sumStr[sumDigits - i] = tmpNum + '0';
    }
    
    if (sumStr[0] == '0') {
        sumStr++;
    }
    
    return sumStr;
}
```

`sumStr`数组是一次性申请分配了全部所需空间，不需要动态扩容，这样做效率很高。不过这段代码有个小瑕疵，最后去除首位不需要的`0`时直接使用了`sumStr++`，这会导致一个字节的内存泄漏问题。


### C++

```cpp
class Solution {
public:
    string addStrings(string num1, string num2) {
        int digits1 = num1.length();
        int digits2 = num2.length();
        int sumDigits = digits1 > digits2 ? digits1: digits2;
    
        string sumStr = "";
        
        char carryFlag = 0;
        int tmpNum = 0;
        int i;
        for (i = 1; i <= sumDigits; i++) {
            tmpNum = carryFlag;
            if (i <= digits1) {
                tmpNum += (num1[digits1 - i] - '0');
            }
            if (i <= digits2) {
                tmpNum += (num2[digits2 - i] - '0');
            }
            
            if (tmpNum > 9) {
                tmpNum -= 10;
                carryFlag = 1;
            } else {
                carryFlag = 0;
            }
            
            sumStr += (tmpNum + '0');
        }
    
        if (carryFlag == 1) {
            sumStr += '1';
        }
        
        reverse(sumStr.begin(), sumStr.end());
        
        return sumStr;
    }
};
```

这里使用了C++风格的`string`操作，用`+=`连接新加入的字符，最后用`reverse()`函数翻转字符串。显而易见，这样做的效率没有C语言版本高。









