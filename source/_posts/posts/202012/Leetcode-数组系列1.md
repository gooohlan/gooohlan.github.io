---
title: Leetcode-数组系列1
date: '2020-12-07 15:29:30'
updated: '2020-12-07 15:29:57'
tags:
  - Golang
  - leetcode
  - 学习
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20190622.jpg'
categories:
  - - 技术
    - 算法
permalink: /leetcode_array1.html
abbrlink: 76
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 169.多数元素，217.多数元素，219. 存在重复元素 II，228. 汇总区间，268. 丢失的数字，283. 移动零'
toc_number: false
---


### 169.多数元素

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-7f596c4b.png)

#### 解题思路：

1. 利用哈希表存储每个字符出现的个数，出现次数大于`n/2`的即为多数元素
2. 根据题意，多数元素的个数大于`n/2`，每次遇到多数元素就将个数+1，否则减一，值为负数时则证明当前选取的这个数不是多数元素，则更换多数元素继续循环(摩尔投票法)
   ![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-022d2db7.png)
   上图来着，题169解解[摩尔投票法](https://leetcode-cn.com/problems/majority-element/solution/3chong-fang-fa-by-gfu-2)
3. 排序，取下标为`len(nums)/2`的值

#### 示例代码：

```go
// map解法
func majorityElement(nums []int) int {
	mapArr := make(map[int]int)
	for _, v := range nums {
		if _, ok := mapArr[v]; !ok {
			mapArr[v] = 1
		} else {
			mapArr[v]++
		}
	}

	for k, v := range mapArr {
		if v > len(nums)/2 {
			return k
		}
	}
	return 0
}


func majorityElement0(nums []int) int {
	mapArr := make(map[int]int)
	for _,v := range nums{
		mapArr[v] ++
		if mapArr[v] > len(nums)/2{
			return v
		}
	}
	return 0
}


// 题目中保证了众数出现次数多余一半,所以遇到众数+1,非众数-1的最终结果始终大于1
func majorityElement1(nums []int) int {
	count, repeat := 0, nums[0]
	for k, v := range nums {
		if v == repeat {
			count++
		} else {
			count--
			if count == 0 && k<len(nums) {
				repeat = nums[k+1]
			}
		}
	}
	return repeat
}

// 先排序直接找 len/2的数
func majorityElement2(nums []int) int{
	sort.Ints(nums)
	return nums[len(nums)/2]
}
```

### 217.多数元素

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-3291f9c1.png)

#### 解题思路：

1. 哈希表

#### 示例代码:

```go
func containsDuplicate(nums []int) bool {
	mapArr := make(map[int]bool)
	for _, v := range nums {
		if mapArr[v] {
			return true
		}
		mapArr[v] = true
	}
	return false
}
```

### 219. 存在重复元素 II

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-b7080722.png)

#### 解题思路：

1. 哈希表，不存在则存入，存在这判断两个下标之间的差值是否小于k

#### 示例代码：

```go
func containsNearbyDuplicate(nums []int, k int) bool {
	mapArr := make(map[int]int)
	for key, val := range nums {
		if v, ok := mapArr[val]; ok {
			if key-v <= k {
				return true
			}
		}
		mapArr[val] = key
	}
	return false
}
```

### 228. 汇总区间

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-071d61e0.png)

#### 解题思路：

1. 判断相邻元素差值是否为1，不分1则处于下一个区间，注意处理最后一个元素

#### 示例代码：

```go
func summaryRanges(nums []int) []string {
	var rStr []string
	if len(nums) == 0 {
		return rStr
	}
	var head, tail int
	head, tail = nums[0], nums[0]
	for i := 1; i < len(nums); i++ {
		if nums[i] != tail+1 {
			if head == tail {
				rStr = append(rStr, strconv.Itoa(head))
			} else {
				rStr = append(rStr, strconv.Itoa(head)+"->"+strconv.Itoa(tail))
			}
			head = nums[i]
			tail = nums[i]
		}else{

			tail = nums[i]
		}
	}
	if head == tail {
		rStr = append(rStr, strconv.Itoa(head))
	} else {
		rStr = append(rStr, strconv.Itoa(head)+"->"+strconv.Itoa(tail))
	}
	return rStr
}


func summaryRanges1(nums []int) []string {
	var rStr []string
	for i,j :=0,0; j<len(nums);{
		if j+1 < len(nums) && nums[j+1] == nums[j]+1{
			j++
			continue
		}
		if i==j {
			rStr = append(rStr, strconv.Itoa(nums[i]))
		}else{
			rStr = append(rStr, strconv.Itoa(nums[i]) + "->" + strconv.Itoa(nums[j]))
		}
		i = j+1
		j++
	}
	return rStr
}
```

### 268. 丢失的数字

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-89a5554f.png)

#### 解题思路：

1. 哈希表，循环一次将数组元素存下来，在循环`0-len(nums)`，不存在的就是丢失的
2. 排序后遍历，下标和数组不相等这标识此下标树缺失
3. 数学方式，通过高斯求和`n*(n+1)/2`算出所有的和，再减去所有的数组元素，剩下的就是丢失的树
4. 异或运算

#### 示例代码：

```go
// 哈希
func missingNumber1(nums []int) int {
	mapArr := make(map[int]bool)
	for _, v := range nums {
		mapArr[v] = true
	}
	for i := 1; i <= len(nums); i++ {
		if _, ok := mapArr[i]; !ok {
			return i
		}
	}
	return 0
}

// 排序

func missingNumber2(nums []int) int {
	sort.Ints(nums)
	for k, v := range nums {
		if k != v {
			return k
		}
	}
	return len(nums)
}

// 数学
func missingNumber3(nums []int) int {
	sum := 0
	for _, v := range nums {
		sum += v
	}
	return len(nums)*(len(nums)+1)/2 - sum
}

// 异或位运算
func missingNumber4(nums []int) int {
	miss := len(nums)
	for k, v := range nums {
		miss ^= k ^ v
	}
	return miss
}
```

### 283. 移动零

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-be9c99ed.png)

#### 解题思路

1. 双指针，A一直向右移动，遇到非0的与B交换即可
   ![283_2.gif](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/36d1ac5d689101cbf9947465e94753c626eab7fcb736ae2175f5d87ebc85fdf0-283_2.gif)

#### 示例代码：

```go
func moveZeroes(nums []int) {
	for i, j := 0, 0; i < len(nums); i++ {
		if nums[i] != 0 {
			nums[i], nums[j] = nums[j], nums[i]
			j++
		}
	}
	fmt.Println(nums)
}
```
