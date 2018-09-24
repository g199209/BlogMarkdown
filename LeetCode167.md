title: LeetCode 167. Two Sum II - Input array is sorted (Medium)
permalink: LeetCode_167
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-25 17:14:57

---

给定一个已经排序的整数集合及一个目标数，从集合中找出两个数，使其和等于目标数。

<!--more-->

## 原始问题

> https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/
>
> Given an array of integers that is already sorted in ascending order, find two numbers such that they add up to a specific target number.
> 
>  The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.
> 
> You may assume that each input would have exactly one solution.
> 
> Input: numbers={2, 7, 11, 15}, target=9
> Output: index1=1, index2=2

## 解题思路

要充分利用数组已排序的性质，使用双指针进行操作，若两数和小于目标数，增加左侧指针；若大于目标数，减小右侧指针。具体见代码。

## AC代码
### C++

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left, right, tmp;
        vector<int> result;
        left = 0;
        right = numbers.size() - 1;
        
        while (left < right) {
            tmp = numbers[left] + numbers[right];
            if (tmp == target) {
                result.push_back(left + 1);
                result.push_back(right + 1);
                return result;
            } else if (tmp < target) {
                left++;
            } else {
                right--;
            }
        }
        
        return result;
    }
};
```

## 相关题目

> [LeetCode 001. Two Sum (Easy)](/2016/10/25/LeetCode_001/)
