---
title: leetcode-141
date: 2020-02-18 22:21:46
tags: leetcode
---
# 141. Linked List Cycle
## 題目敘述
Given a linked list, determine if it has a cycle in it.

To represent a cycle in the given linked list, we use an integer pos which represents the position (0-indexed) in the linked list where tail connects to. If pos is -1, then there is no cycle in the linked list.

簡單來說就是檢查LinkList是不是循環的
題目要求O(1)Space

## 解法

```Python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        slow = head
        fast = head
        try:
            while fast.next.next:
                slow = slow.next
                fast = fast.next.next

                if slow is fast:
                    return True
        except:
            return False
```
不能使用額外空間的話，就設置兩個指標，一個快，一個慢
快的一次走兩步，慢的一次走一步
走到有None時代表沒有循環
如果兩個指標相遇，代表有循環
而`while fast.next.next`有可能發生錯誤，這時用try except抓起來
並且出現錯誤代表沒有循環，這樣就解決了
