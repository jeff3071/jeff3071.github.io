---
title: leetcode.111
date: 2020-02-05 21:38:00
tags: leetcode
---
# 111. Minimum Depth of Binary Tree

## 題目敘述
Given a binary tree, find its minimum depth.

The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.


## 解法

```Python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
        def minDepth(self, root: TreeNode) -> int:
            if not root:
                return 0
                
            def check(node):
                if node is None:
                    return 0
                l = check(node.left)
                r = check(node.right)
                                            
                if min(l,r) != 0:
                    return min(l,r) + 1
                else:
                    return max(l,r) + 1
            return check(root)
```

跟110很像，這次要找root的最小深度
但要注意的是在歪斜樹的情況，也就是root只有左子樹或右子樹時，用min(l,r)會取到0
為了避免這個情況，加了一個條件，先檢查min(l,r)是否為0在進行回傳

