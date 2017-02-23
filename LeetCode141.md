title: LeetCode 141. Linked List Cycle (Easy)
permalink: LeetCode_141
toc: true
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-10-24 17:41:59

---

给定一个链表，确定其中是否有环。

<!--more-->

## 原始问题

> https://leetcode.com/problems/linked-list-cycle/
>
> Given a linked list, determine if it has a cycle in it.
> 
> Follow up:
> Can you solve it without using extra space?


## 解题思路

最直接的想法是用一个集合将之前出现过的节点保存起来，之后判断新节点是否在已有节点的集合中即可，不过这种方法效率较低，而且需要很大的额外空间，并不是好方法。

这个问题比较优秀的解法是，引入两个指针，一个`slow`指针一次前进一步，另一个`fast`指针一次前进两步，这两个指针相遇即是存在环的充要条件。

## AC代码

### C语言
```C
bool hasCycle(struct ListNode *head) {
	struct ListNode * slow, * fast;
	slow = head;
	fast = head;
	
	if (head == NULL) {
	    return false;
	}
	
	while (fast->next != NULL && fast->next->next != NULL) {
		slow = slow->next;
		fast = fast->next->next;
		if (fast == slow)
			return true;
	}
	
	return false;
}
```

### Java

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow, fast;
        if (head == null) {
            return false;
        }
        
        slow = head;
        fast = head;
        
        do {
            try {
                slow = slow.next;
                fast = fast.next.next;
            } catch(Exception e) {
                return false;
            }
        } while(slow != fast);
        
        return true;
    }
}
```

这里用了一种比较特殊的写法，使用了`try...catch...`语句块进行异常捕捉，当`fast.next.next`遇到`null`时即会发生异常，此时意味着到达了链表结尾，链表中不存在环，故返回`false`。

## 相关题目

> [LeetCode 142. Linked List Cycle II (Medium)](/2016/10/24/LeetCode_142/)
> > [LeetCode 160. Intersection of Two Linked Lists (Easy)](/2016/10/25/LeetCode_160/)