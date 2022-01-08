---
title: 日刷leetcode--简单版（六）
date: '2019-10-24 16:32:05'
updated: '2020-04-13 16:25:40'
tags:
  - Golang
  - leetcode
  - 算法
categories:
  - - 技术
    - 算法
cover: 'https://cdn.inkdp.cn/img/20190222.jpg'
permalink: /leetcode6.html
abbrlink: 20297
---

### 返回总目录

[日刷leetcode–简单版](https://inkdp.cn/leetcode.html)

---

### 167. 两数之和 II - 输入有序数组

#### 题目描述
![image.png](https://cdn.inkdp.cn/img/image-56cb3cb1.png)
#### 解题思路
* 定义双指针，分别在头部与尾部
* 判断两个的和是否与`targent`相等，相等级返回，比sum大则尾指针前移，反之头指针后移
##### 示例代码
```go
func twoSum(numbers []int, target int) []int {
	l, r := 0, len(numbers)-1
	for l < r {
		sum := numbers[l] + numbers[r]
		if sum == target {
			return []int{l + 1, r + 1}
		}
		if sum < target {
			l++
		} else {
			r--
		}
	}
	return []int{-1, -1}
}
```
##### 运行结果
> 执行用时 :4 ms, 在所有 Golang 提交中击败了97.30%的用户
> 内存消耗 :3 MB, 在所有 Golang 提交中击败了68.38%的用户

### 168. Excel表列名称
#### 题目描述：
![image.png](https://cdn.inkdp.cn/img/image-6f5a1e22.png)
#### 解题思路
* 可以看做一个10进制转26进制问题，进制转换原理可查看-> [理解进制转换的原理](https://zhuanlan.zhihu.com/p/75006709)
##### 复杂度分析

* 时间复杂度：O(1)
* 空间复杂度：O(1)
##### 示例代码
```
func convertToTitle(n int) string{
	var str string
	for n > 0 {
		n -- // 减去一个,因为A是对应的是1,而不是0
		str = string('A' + int32(n%26)) + str
		n/=26
	}
	return str
}
```
