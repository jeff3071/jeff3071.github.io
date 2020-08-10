---
title: leetcode-118
date: 2020-02-09 20:16:03
tags: leetcode
---

# 118. Pascal's Triangle
## 題目敘述

Given a non-negative integer numRows, generate the first numRows of Pascal's triangle.
<!--more-->
## 解法

```
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        ans = []
        for i in range(numRows):
            if i == 0:
                ans.append([1])
            elif i == 1:
                ans.append([1,1])
            else:
                temp = [1]
                for x in range(i-1):
                    temp.append(ans[i-1][x]+ ans[i-1][x+1])
                temp.append(1)
                ans.append(temp)
        return ans
```
滿容易理解的，先處理第一層和第二層
再用一個For迴圈去產生接下來幾層
例如第三層是[1,2,1]，而頭尾都是1，中間的2是第二層的兩個元素相加
所以如果是第i層，會有i-2個元素需要從i-1層產生

