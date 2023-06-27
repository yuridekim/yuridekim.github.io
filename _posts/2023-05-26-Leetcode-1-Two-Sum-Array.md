---
title : '[Leetcode] 1. Two Sum Array'
date : 2023-05-26 23:01:00 +0900
categories : [Algorithm, Leetcode]
tags : [algorithm, ps, leetcode] #소문자만 가능
pinned : 0
---
```
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        buff={}
        for i, n in enumerate(nums):
            d=target-n
            if n in buff:
                return [buff[n],i]
            else:
                buff[d]=i
```

Time complexity is O(n) with space complexity O(n).