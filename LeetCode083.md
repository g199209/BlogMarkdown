title: LeetCode 083. Remove Duplicates from Sorted List (Easy)
permalink: LeetCode_083
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-11-01 22:43:58

---

给出一个有序链表，去除其中的重复元素。

<!--more-->

## 原始问题

> https://leetcode.com/problems/remove-duplicates-from-sorted-list/
>
> Given a sorted linked list, delete all duplicates such that each element appear only once.
> 
> For example,
> Given `1->1->2`, return `1->2`.
> Given `1->1->2->3->3`, return `1->2->3`.


## 解题思路

因为链表是有序的，直接从头循环搜索一遍即可。

## AC代码

### C语言

```c
struct ListNode* deleteDuplicates(struct ListNode* head) {
    struct ListNode * p = head;
    struct ListNode * tmp;
    
    if(head == NULL) {
        return NULL;
    }
    
    while(p->next != NULL) {
        if (p->val == p->next->val) {
            tmp = p->next;
            p->next = p->next->next;
            free(tmp);
        } else {
            p = p->next;
        }
    }
    
    return head;
}
```

这里使用了正确的删除节点的方法，使用`free()`释放了内存空间。