---
title: Leetcode-数组系列6
updated: '2022-03-03 18:16'
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: '算法,LeetCode,数组,Golang'
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

### 682. 棒球比赛
<p>你现在是一场采用特殊赛制棒球比赛的记录员。这场比赛由若干回合组成，过去几回合的得分可能会影响以后几回合的得分。</p>

<p>比赛开始时，记录是空白的。你会得到一个记录操作的字符串列表 <code>ops</code>，其中 <code>ops[i]</code> 是你需要记录的第 <code>i</code> 项操作，<code>ops</code> 遵循下述规则：</p>

<ol>
	<li>整数 <code>x</code> - 表示本回合新获得分数 <code>x</code></li>
	<li><code>"+"</code> - 表示本回合新获得的得分是前两次得分的总和。题目数据保证记录此操作时前面总是存在两个有效的分数。</li>
	<li><code>"D"</code> - 表示本回合新获得的得分是前一次得分的两倍。题目数据保证记录此操作时前面总是存在一个有效的分数。</li>
	<li><code>"C"</code> - 表示前一次得分无效，将其从记录中移除。题目数据保证记录此操作时前面总是存在一个有效的分数。</li>
</ol>

<p>请你返回记录中所有得分的总和。</p>

<p><strong>示例 1：</strong></p>

> <strong>输入：</strong>ops = ["5","2","C","D","+"]
<strong>输出：</strong>30
<strong>解释：</strong>
"5" - 记录加 5 ，记录现在是 [5]
"2" - 记录加 2 ，记录现在是 [5, 2]
"C" - 使前一次得分的记录无效并将其移除，记录现在是 [5].
"D" - 记录加 2 * 5 = 10 ，记录现在是 [5, 10].
"+" - 记录加 5 + 10 = 15 ，记录现在是 [5, 10, 15].
所有得分的总和 5 + 10 + 15 = 30

<p><strong>示例 2：</strong></p>


> <strong>输入：</strong>ops = ["5","-2","4","C","D","9","+","+"]
<strong>输出：</strong>27
<strong>解释：</strong>
"5" - 记录加 5 ，记录现在是 [5]
"-2" - 记录加 -2 ，记录现在是 [5, -2]
"4" - 记录加 4 ，记录现在是 [5, -2, 4]
"C" - 使前一次得分的记录无效并将其移除，记录现在是 [5, -2]
"D" - 记录加 2 * -2 = -4 ，记录现在是 [5, -2, -4]
"9" - 记录加 9 ，记录现在是 [5, -2, -4, 9]
"+" - 记录加 -4 + 9 = 5 ，记录现在是 [5, -2, -4, 9, 5]
"+" - 记录加 9 + 5 = 14 ，记录现在是 [5, -2, -4, 9, 5, 14]
所有得分的总和 5 + -2 + -4 + 9 + 5 + 14 = 27

<p><strong>示例 3：</strong></p>

> <strong>输入：</strong>ops = ["1"]
<strong>输出：</strong>1
#### 解题思路

{% note no-icon info simple %}

题目比较简单，因为数据都是合法的，所以不需要额外验证数据，直接用数组模拟栈处理即可

{% endnote %}

#### 示例代码
```go
func calPoints(ops []string) int {
	var stack []int
	for _, op := range ops {
		switch op {
		case "+":
			stack = append(stack, stack[len(stack)-1]+stack[len(stack)-2])
		case "D":
			stack = append(stack, stack[len(stack)-1]*2)
		case "C":
			stack = stack[:len(stack)-1]
		default:
			score, _ := strconv.Atoi(op)
			stack = append(stack, score)
		}
	}
	var count int
	for _, i := range stack {
		count += i
	}
	return count
}
```

### 694.  数组的度
<p>给定一个非空且只包含非负数的整数数组&nbsp;<code>nums</code>，数组的 <strong>度</strong> 的定义是指数组里任一元素出现频数的最大值。</p>

<p>你的任务是在 <code>nums</code> 中找到与&nbsp;<code>nums</code>&nbsp;拥有相同大小的度的最短连续子数组，返回其长度。</p>

<p><strong>示例 1：</strong></p>

> <strong>输入：</strong>nums = [1,2,2,3,1]
<strong>输出：</strong>2
<strong>解释：</strong>
输入数组的度是 2 ，因为元素 1 和 2 的出现频数最大，均为 2 。
连续子数组里面拥有相同度的有如下所示：
[1, 2, 2, 3, 1], [1, 2, 2, 3], [2, 2, 3, 1], [1, 2, 2], [2, 2, 3], [2, 2]
最短连续子数组 [2, 2] 的长度为 2 ，所以返回 2 。

<p><strong>示例 2：</strong></p>

> <strong>输入：</strong>nums = [1,2,2,3,1,4,2]
> <strong>输出：</strong>6
> <strong>解释：</strong>
> 数组的度是 3 ，因为元素 2 重复出现 3 次。
> 所以 [2,2,3,1,4,2] 是最短子数组，因此返回 6 。

####  解题思路

{% note no-icon info simple %}

通过哈希记录当前数值出现的次数与首次出现和最后一次出现，取出最多出现次数的值，用最后一次出现减去第一次出现的即可得出度

{% endnote %}

#### 示例代码

```go
type entry struct {
   cnt, l, r int
}

func findShortestSubArray(nums []int) int {
   var ans, maxCnt int
   mp := map[int]entry{}
   for i, v := range nums {
      if e, ok := mp[v]; ok {
         e.cnt++
         e.r = i
         mp[v] = e
      } else {
         mp[v] = entry{1, i, i}
      }
   }
   for _, e := range mp {
      if e.cnt > maxCnt {
         maxCnt, ans = e.cnt, e.r-e.l+1
      }
      if e.cnt == maxCnt {
         ans = min(ans, e.r-e.l+1)
      }
   }
   return ans
}

func min(a, b int) int {
   if a < b {
      return a
   }
   return b
}
```

### 704. 二分查找

<p>给定一个&nbsp;<code>n</code>&nbsp;个元素有序的（升序）整型数组&nbsp;<code>nums</code> 和一个目标值&nbsp;<code>target</code> &nbsp;，写一个函数搜索&nbsp;<code>nums</code>&nbsp;中的 <code>target</code>，如果目标值存在返回下标，否则返回 <code>-1</code>。</p>

<strong>示例 1:</strong></p>


> <strong>输入:</strong> <code>nums</code> = [-1,0,3,5,9,12], <code>target</code> = 9
<strong>输出:</strong> 4
<strong>解释:</strong> 9 出现在 <code>nums</code> 中并且下标为 4

<p><strong>示例&nbsp;2:</strong></p>
> <strong>输入:</strong> <code>nums</code> = [-1,0,3,5,9,12], <code>target</code> = 2
<strong>输出:</strong> -1
<strong>解释:</strong> 2 不存在 <code>nums</code> 中因此返回 -1

#### 示例代码

```go
func search(nums []int, target int) int {
   l, r := 0, len(nums)-1
   for l <= r {
      mid := (r-l)/2 + l
      num := nums[mid]
      if num == target {
         return mid
      } else if target < num {
         r = mid - 1
      } else {
         l = mid + 1
      }
   }
   return -1
}
```
