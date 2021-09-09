---
title: Leetcode-数组系列6
updated: '2021-09-07 21:45'
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: '算法,LeetCode,数组,Golang'
toc_number: false
abbrlink: 55343
date: 2021-09-07 16:33:31
cover:
description:
---

### 674. 最长连续递增序列

<p>给定一个未经排序的整数数组，找到最长且<strong> 连续递增的子序列</strong>，并返回该序列的长度。</p>

<p><strong>连续递增的子序列</strong> 可以由两个下标 <code>l</code> 和 <code>r</code>（<code>l < r</code>）确定，如果对于每个 <code>l <= i < r</code>，都有 <code>nums[i] < nums[i + 1]</code> ，那么子序列 <code>[nums[l], nums[l + 1], ..., nums[r - 1], nums[r]]</code> 就是连续递增子序列。</p>

<p> </p>

<p><strong>示例 1：</strong></p>

> <strong>输入：</strong>nums = [1,3,5,4,7]
> <strong>输出：</strong>3
> <strong>解释：</strong>最长连续递增序列是 [1,3,5], 长度为3。
> 尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为 5 和 7 在原数组里被 4 隔开。

<p><strong>示例 2：</strong></p>

> <strong>输入：</strong>nums = [2,2,2,2,2]
> <strong>输出：</strong>1
> <strong>解释：</strong>最长连续递增序列是 [2], 长度为1。

#### 解题思路

{% note no-icon info simple %}

因为求的是递增序列，所以当前元素小于等于上一个元素时，表示上一个递增子序列结束，我们更新下一个子序列开始位置，并求出此时最大子序列，依此循环即可

{% endnote %}

#### 示例代码

```go
func findLengthOfLCIS(nums []int) int {
   var ans, start int
   for k, v := range nums {
      if k > 0 && v <= nums[k-1] {
         start = k
      }
      ans = max(ans, k-start +1)
   }
   return ans
}

func max(a, b int) int {
   if a < b {
      return b
   }
   return a
}
```
