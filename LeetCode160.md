title: LeetCode 160. Intersection of Two Linked Lists (Easy)
permalink: LeetCode_160
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-25 00:12:17

---

两个链表，可能会有交叉，返回交叉点的元素，若没有交叉则返回`null`。

<!--more-->

## 原始问题

> https://leetcode.com/problems/intersection-of-two-linked-lists/
>
> Write a program to find the node at which the intersection of two singly linked lists begins.
> For example, the following two linked lists:
```
A:          a1 → a2
                   ↘
                     c1 → c2 → c3
                   ↗            
B:     b1 → b2 → b3
```
> begin to intersect at node c1.
> 
> Notes:
> 
> - If the two linked lists have no intersection at all, return `null`.
> - The linked lists must retain their original structure after the function returns.
> - You may assume there are no cycles anywhere in the entire linked structure.
> - Your code should preferably run in O(n) time and use only O(1) memory.



## 解题思路
若将`b1`节点连接到`c3`节点后面，此问题就转换为了[LeetCode 142. Linked List Cycle II](/2016/10/24/LeetCode_142/)问题，可用相同方法求解，最后断开`c3 -> b1`的连接即可。[LeetCode题解](https://siddontang.gitbooks.io/leetcode-solution/content/linked_list/linked_list_cycle.html)中就是这样做的，不过对此还可以进行优化。

开始时直接设置`slow`及`fast`指针进行移动，`fast`指针在移动过程中会遇到`null`，对此进行处理即可。第一次遇到`null`时进行连接，若第二次遇到`null`则说明两个链表没有交叉。其余步骤与LeetCode 142中的做法完全一样，具体见代码。


## AC代码

### C语言

```C
struct ListNode *getIntersectionNode(struct ListNode *headA, struct ListNode *headB) {
    struct ListNode * slow, * fast, *p, *tail;
    
    if (!headA || !headB) {
        return NULL;
    }
    
    slow = headA;
    fast = headA;
    p = headA;
    tail = NULL;
    
    do{
        if (!fast->next) {
            if (tail) {
                tail->next = NULL;
                return NULL;
            }
            fast->next = headB;
            tail = fast;
        }
        if (!fast->next->next) {
            if (tail) {
                tail->next = NULL;
                return NULL;
            }
            fast->next->next = headB;
            tail = fast->next;
        }
        
        slow = slow->next;
        fast = fast->next->next;
    }while(slow != fast);
    
    while (p != slow) {
        p = p->next;
        slow = slow->next;
    }
    
    tail->next = NULL;
    
    return p;
}
```
## 相关题目

> [LeetCode 141. Linked List Cycle (Easy)](/2016/10/24/LeetCode_141/)
> [LeetCode 142. Linked List Cycle II (Medium)](/2016/10/24/LeetCode_142/)