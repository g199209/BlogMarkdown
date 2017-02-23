title: LeetCode 345. Reverse Vowels of a String (Easy)
permalink: LeetCode_345
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-25 20:36:38

---

反转字符串中的所有元音字母。

<!--more-->

## 原始问题

> https://leetcode.com/problems/reverse-vowels-of-a-string/
>
> Write a function that takes a string as input and reverse only the vowels of a string.
> 
> Example 1:
> Given s = "hello", return "holle".
> 
> Example 2:
> Given s = "leetcode", return "leotcede".
> 
> Note:
> The vowels does not include the letter "y".


## 解题思路

双指针，一个`left`从头部开始找，另一个`right`从尾部开始找，若`left`及`right`均为元音，交换二者。截止条件为`left >= right`。

## AC代码

### C语言

```c
#define checkVowel(c) (c=='a'||c=='e'||c=='i'||c=='o'||c=='u'||c=='A'||c=='E'||c=='I'||c=='O'||c=='U')

char * reverseVowels(char* s) {
	char * left, *right;
	char tmpc;

	// Initialize pointer
	left = s;
	right = s;

	while (*right != '\0') {
		right++;
	}

	while (1) {
		while (!checkVowel(*left)) {
		    left++;
		}
		
		while (!checkVowel(*right)) {
		    right--;
		}
		
		if (left < right) {
		    tmpc = *left;
		    *left = *right;
		    *right = tmpc;
		    left++;
		    right--;
		} else {
		    return s;
		}
	}

	return s;
}
```

这里判断是否为元音字母使用了宏定义，以此提高运行效率。

## 相关题目

> [LeetCode 344. Reverse String (Easy)](/2016/10/25/LeetCode_344/)
