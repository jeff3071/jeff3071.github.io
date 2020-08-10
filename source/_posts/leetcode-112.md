---
title: leetcode-112
date: 2020-02-09 19:55:00
tags: leetcode
---

# 112. Path Sum
## 題目敘述

Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.
<!--more-->
## 解法

```Python

# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def hasPathSum(self, root: TreeNode, sum: int) -> bool:
        if not root:
            return 0
        
        sum = sum - root.val
        
        if sum == 0 and root.left is None and root.right is None:
            return True
        
        return self.hasPathSum(root.left, sum) or self.hasPathSum(root.right, sum)
```
當到達一個node時，將sum減掉這個node的值，並設定當sum為0且無左右節點時就回傳為真
接著用遞迴遍歷整棵樹
因為只要找出一條Path所以使用 or
