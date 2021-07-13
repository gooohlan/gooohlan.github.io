---
title: 日刷leetcode--简单版（三）
date: '2019-08-09 11:58:53'
updated: '2019-08-19 11:58:53'
tags:
  - leetcode
  - 算法
  - Golang
permalink: /leetcode3.html
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/timg-af5a1c18.jpeg'
categories:
  - - 技术
    - 算法
abbrlink: 43823
toc_number: false
---


### 返回总目录

[日刷leetcode–简单版](/leetcode.html)

---

### 58. 最后一个单词的长度

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-45f2c7e9.png)

#### 解题思路

* 定义一个变量统计，从前往后遍历，遇到空格归零就可以了，注意处理最后几个个字符全为空格的情况
* 定义一个变量统计，从后往前便利，虽然时间复杂度同为 O(n)，但是第二个明显快很多

##### 示例代码

```go
func lengthOfLastWord(s string) int {
        var count int
                for i:= len(s)-1; i >= 0; i-- {
                        if s[i] == 32{
                                if count == 0 {
                                        continue
                                }else{
                                        break
                                }
                        }
                        count ++
                }
        return count
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.2 MB, 在所有 Go 提交中击败了 39.13%的用户

### 66.加一

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-1081bedf.png)

##### 解题思路

* 从后面往前面循环，最后以一位加 1 即可，处理好末尾`9`与`999`

```go
func plusOne(digits []int) []int {
        c := 1 // 定义一个变量用来进位，进位归零则程序结束
        for i := len(digits) - 1; i >= 0; i-- {
                digits[i] += c
                c--
                if digits[i] == 10 {
                        digits[i] = 0
                        c = 1
                }
                if c == 0 {
                        return digits
                }
        }
        if c != 0 { // 循环完后依旧存在进位则表示遇到了999
                digits = append([]int{1}, digits...)
        }
        return digits
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.2 MB, 在所有 Go 提交中击败了 30.57%的用户

### 67.二进制求和

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-21de0dac.png)

#### 解题思路

* 判断两个字符串的大小，保证 a 为较长的一个
* 从后往前循环相加，先循环较短的字符串，再循环长字符串剩余的，定义一个变量记录是否需要进位，两个字符串相加是需要加上进位的值
* 最后判断进位值是否为 0，不为 0 则在相加后字符串最前面一位加 1

##### 示例代码

```go
func addBinary(a string, b string) string {
	la, lb := len(a), len(b)
	if la < lb {
		la, lb = lb, la
		a, b = b, a
	}
	var carry, s byte
	str := make([]byte, la+1)
	for lb > 0 {
		la--
		lb--
		s = byte(a[la]-'0') + byte(b[lb]-'0') + carry
		carry = s / 2
		s = s % 2
		str[la+1] = byte(s + '0')
	}
	for la > 0 {
		la--
		s = byte(a[la]-'0') + carry
		carry = s / 2
		s = s % 2
		str[la+1] = byte(s + '0')
	}
	if carry == 1 {
		str[la] = carry + '0'
	} else {
		str = str[la+1:]
	}
	return string(str[la:])
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.3 MB, 在所有 Go 提交中击败了 68.18%的用户

### 69. x 的平方根

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-66ad4fc1.png)

#### 解题思路 1

* 使用官方包 math.Sqrt，然后提取整数部分即可(不提供代码)

#### 解题思路 2

* 使用二分法，判断中位数的积是否大于 x，是则右边向左移，否则左边直接等于中位数
* 注意的是取中位数是要取右中位数，也就是要加１，不然会死循环

##### 示例代码 2

```go
func mySqrt(x int) int {
	l, r := 0, x/2+1
	for l < r {
		mid := (l + r + 1) / 2
		sqrt := mid * mid
		if sqrt > x {
			r = mid - 1
		} else {
			l = mid
		}
		fmt.Println(l, r, mid)
	}

	return l
}
```

##### 运行结果

> 执行用时 :8 ms, 在所有 Go 提交中击败了 29.75%的用户
> 内存消耗 :2.8 MB, 在所有 Go 提交中击败了 5.23%的用户

### 70. 爬楼梯

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-4aa9ad43.png)

#### 解题思路

* 这是一个标准的斐波拉切数列，所以就不多说

##### 示例代码

```go
func climbStairs(n int) int {
	if n < 3 {
		return n
	}

	a, b := 1, 2
	res := 0
	for i := 2; i < n; i++ {
		res = a + b
		a, b = b, res
	}
	return res
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2 MB, 在所有 Go 提交中击败了 52.61%的用户

### 83. 删除排序链表中的重复

#### 题目描述

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-191c979d.png)

#### 解题思路

* 因为是排序了的，所以就相对来说比较简单，判断是否与下一个相等，相等即后移即可，讲`Next`指向`Next.Next`

##### 示例代码

```go
func deleteDuplicates(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	carry := head

	for carry != nil && carry.Next != nil {
		if carry.Val == carry.Next.Val {
			carry.Next = carry.Next.Next
		}else{
			carry = carry.Next
		}
	}
	return head
}
```

##### 运行结果

> 执行用时 :4 ms, 在所有 Go 提交中击败了 96.46%的用户
> 内存消耗 :3.2 MB, 在所有 Go 提交中击败了 48.18%的用户
