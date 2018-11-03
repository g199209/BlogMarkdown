title: LeetCode 019. Remove Nth Node From End of List (Easy)
permalink: LeetCode_019
toc: true
mathjax: false
fancybox: false
tags: [Online Judge]
categories: 算法之美
date: 2016-11-01 23:00:00

---

给出一个链表，移除其中倒数第N个元素。

<!--more-->

## 原始问题

> https://leetcode.com/problems/remove-duplicates-from-sorted-list/
>
> Given a linked list, remove the nth node from the end of list and return its head.
> 
> For example,
> Given linked list: `1->2->3->4->5`, and `n = 2`.
> After removing the second node from the end, the linked list becomes `1->2->3->5`.
> 
> Note:
> Given n will always be valid.
> Try to do this in one pass.


## 解题思路

双指针，后一个指针比前一个指针先走N步，后一个指针移动到链表末尾时前一个指针指向的就是要删除的元素。


## AC代码

### C语言

```c
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    struct ListNode * p1, * p2, * tmp;
    struct ListNode dummy;
    
    dummy.next = head;
    p1 = &dummy;
    p2 = &dummy;
    
    for (int i = 0; i < n; i++) {
        p2 = p2->next;
    }
    
    while (p2->next != NULL) {
        p1 = p1->next;
        p2 = p2->next;
    }
    
    tmp = p1->next;
    p1->next = p1->next->next;
    free(tmp);
    
    return dummy.next;
}
```

这里使用了正确的删除节点的方法，使用`free()`释放了内存空间。