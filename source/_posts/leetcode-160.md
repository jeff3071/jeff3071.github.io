---
title: leetcode-160
date: 2020-02-21 21:26:11
tags: leetcode
---
# 160. Intersection of Two Linked Lists
## 題目敘述

Write a program to find the node at which the intersection of two singly linked lists begins.
![](https://assets.leetcode.com/uploads/2018/12/13/160_example_1.png)

**使用O(1)的空間**
## 解法
```Python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        A = headA
        B = headB
        count = 0
        while A and B and count < 3:
            if B is A:
                return A
            A = A.next
            B = B.next
            if A is None:
                A = headB
                count = count + 1 
            if B is None:
                B = headA
                count = count + 1
        
        return None
```
假設List1是[4,1,8,4,5],List2是[5,0,1,8,4,5]
而 List1 + List2 就是 [4,1,8,4,5,5,0,1,8,4,5]
List2 + List1 就是 [5,0,1,8,4,5,4,1,8,4,5]
相加後長度會相同，且若有交叉點，最後面一定是相同的
這樣的時間複雜度會是O(m+n)

接下來要解決題目要求的空間複雜度需求
這邊使用雙指標，當List1的指標讀完List1時，跳到List2的開頭，反之亦然

最後要解決沒有交叉的情況，在有交叉的情況下，跳躍的次數最多是兩次
利用這個當作停止條件，也就是跳第三次時就代表沒有交叉
