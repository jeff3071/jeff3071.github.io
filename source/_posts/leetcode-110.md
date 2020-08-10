---
title: leetcode.110
date: 2020-02-05 21:15:52
tags: leetcode
---
# 110. Balanced Binary Tree
## 題目敘述
```
Given a binary tree, determine if it is height-balanced.
```
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
        def isBalanced(self, root: TreeNode) -> bool:
            if not root:
                return 1
            def check(node):
                if node is None:
                    return 0
                                            
                l = check(node.left)
                r = check(node.right)
            
                if l == -1 or r == -1 or abs(l - r) > 1:
                    return -1
                return max(l,r) + 1
            return check(root) != -1
```

利用 -1 代表非Balance的情況，並且計算每個node的深度
若左右深度相減大於 1 的話就是不平衡

最後確認return值是否為-1
