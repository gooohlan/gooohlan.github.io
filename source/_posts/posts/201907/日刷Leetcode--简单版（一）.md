---
title: 日刷leetcode--简单版（一）
date: '2019-07-16 15:17:53'
updated: '2019-07-22 15:17:53'
tags:
  - leetcode
  - 算法
  - Golang
permalink: /leetcode1.html
cover: 'https://gooohlan.fishpi.cn/img/timg-af5a1c18.jpeg'
categories:
  - - 技术
    - 算法
abbrlink: 14124
toc_number: false
---

### 返回总目录

[日刷leetcode--简单版](/leetcode.html)

### 1.两数之后

> #### 题目描述
>
> 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
> 你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。
>
> #### 示例
>
> ```
> 给定 nums = [2, 7, 11, 15], target = 9
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]
> ```

#### 解题思路 1

* 暴力法，双循环相加结果等于 target 就返回

##### 示例代码

```
func twoSum(nums []int, target int) []int {
    for i := 0; i < len(nums); i++ {
        for j := i + 1; j < len(nums); j++ {
            if nums[i] + nums[j] ==target {
                return []int{i,j}
            }
        }
    }
    return nil
}
```

##### 运行结果

> 执行用时 :56 ms, 在所有 Go 提交中击败了 32.33%的用户
> 内存消耗 :3 MB, 在所有 Go 提交中击败了 78.76%的用户

#### 解题思路 2

* 遍历一次，求当前值的下标是否存在哈希表中，存在就返回，不存在就将当前值与下标存入哈希表

##### 示例代码

```go
func twoSum(nums []int, target int) []int {
    maps := make(map[int]int)

    for k, v := range nums {
        difference := target - v
        if _, ok := maps[difference]; ok {  // 判断是否存在指定k的值
            return []int{maps[difference], k}
        }
        maps[v] = k // 将值作为key存储到map中
        }
    return nil
}
```

##### 运行结果

> 执行用时 :4 ms, 在所有 Go 提交中击败了 89.93%的用户
> 内存消耗 :3.8 MB, 在所有 Go 提交中击败了 30.06%的用户

---

### 7.整数反转

> ##### 题目描述
>
> 给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
> 示例 1
>
> ```
> 输入: 123
> 输出: 321
> ```
>
> 示例 2
>
> ```
> 输入: -123
> 输出: -321
> ```
>
> 示例 3
>
> ```
> 输入: 120
> 输出: 21
> ```
>
> #### 注意
>
> 假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [− 231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

#### 解题思路

* 依次取模再乘以 10 即可，注意取值范围

##### 示例代码

```go
func reverse(x int) int {
    num := 0
	for x != 0 {
		num = num*10 + x%10
		x = x / 10
	}
	if num > math.MaxInt32 || num < math.MinInt32 {
		return 0
	}
	return num
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.2 MB, 在所有 Go 提交中击败了 41.71%的用户

---

### 9.回文数

> ##### 题目描述
>
> 判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。
> 示例 1:
>
> ```
> 输入: 121
> 输出: true
> ```
>
> 示例 2:
>
> ```
> 输入: -121
> 输出: false
> 解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
> ```
>
> 示例 3:
>
> ```
> 输入: 10
> 输出: false
> 解释: 从右向左读, 为 01 。因此它不是一个回文数。
> ```

#### 解题思路 1

* 从示例中可以看出，负数和 10 的倍数的数都不是回文数，所以首先排除他们
* 转换为字符串，从两头往中间判断是否相等

##### 示例代码

```go
func isPalindrome(x int) bool {
	if x < 0 || (x != 0 && x%10 == 0) {
		return false
	}
	s := strconv.Itoa(x)
	fmt.Println(s)
	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		if s[i] != s[j] {
			return false
		}
	}
	return true
}
```

##### 运行结果

> 执行用时 :20 ms, 在所有 Go 提交中击败了 81.69%的用户
> 内存消耗 :5.5 MB, 在所有 Go 提交中击败了 15.78%的用户

#### 解题思路 2

* 从示例中可以看出，负数和 10 的倍数的数都不是回文数，所以首先排除他们
* 将数字分为两部分，的后半部分反转过来，要求后半部分不得大于前半部分
* 查看数字后半部分是否与前半部分相同，相同既是回文数，比如`1221`前半部分为`12`，后半部分也为`12`，所以他是回文数
* 如果数字位数为奇数，比如`12321`就会出现前半部分为`12`，后半部分为`123`，这样就不相等了，我们将后半部分除以 10 即可。

##### 示例代码

```go
func isPalindrome(x int) bool {
	if x < 0 || (x != 0 && x%10 == 0){
		return false
	}
	var i int
	for i < x {
		i = i * 10 + x%10
		x /= 10
	}
	return x == i || x == i/10
}
```

##### 运行结果

> 执行用时 :4 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :5.1 MB, 在所有 Go 提交中击败了 65.52%的用户

---

**PS:** <font color="red">执行速度感觉是和网速有关的，如果你觉得你的程序够快，就多提交几次。上面思路二的代码一开始 40 多秒。</font>

---

### 13. 罗马数字转整数

> ##### 题目描述
>
> 罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。
>
> ```
> 字符          数值
> I             1
> V             5
> X             10
> L             50
> C             100
> D             500
> M             1000
> ```
>
> 例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。
> 通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：
>
> ```
> I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
> X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。
> C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。
> ```
>
> 给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。
>
> 示例 1:
>
> ```
> 输入: "III"
> 输出: 3
> ```
>
> 示例 2:
>
> ```
> 输入: "IV"
> 输出: 4
> ```
>
> 示例 3:
>
> ```
> 输入: "IX"
> 输出: 9
> ```
>
> 示例 4:
>
> ```
> 输入: "LVIII"
> 输出: 58
> 解释: L = 50, V= 5, III = 3.
> ```
>
> 示例 5:
>
> ```
> 输入: "MCMXCIV"
> 输出: 1994
> 解释: M = 1000, CM = 900, XC = 90, IV = 4.
> ```

#### 解题思路

* 将罗马数字对应值放入哈希表中，注意哈希表 key 类型，因为字符串通过`s[i]`的方式取出来是对应的 ASCII ，所以哈希表的 key 最好使用`byte`类型。
* 定义一个变量记录当前罗马数字，循环字符串，依次相加，判断定义的变了是否小于当前相加的罗马数字，是即减去记录值的二倍。

##### 示例代码

```go
func romanToInt(s string) int {
	roman := map[byte]int{
		'I': 1,
		'V': 5,
		'X': 10,
		'L': 50,
		'C': 100,
		'D': 500,
		'M': 1000,
	}
	var sum int
	index := 999
	for i := 0; i < len(s); i++ {
		this := roman[s[i]]
		if index < this {
			sum = sum - index*2
		}
		sum += this
		index = this
	}
	return sum
}
```

##### 运行结果

> 执行用时 :4 ms, 在所有 Go 提交中击败了 97.65%的用户
> 内存消耗 :3 MB, 在所有 Go 提交中击败了 45.05%的用户

### 14. 最长公共前缀

> ##### 题目描述
>
> 编写一个函数来查找字符串数组中的最长公共前缀。
> 如果不存在公共前缀，返回空字符串 &quot;&quot;。
> 示例 1:
>
> ```
> 输入: ["flower","flow","flight"]
> 输出: "fl"
> ```
>
> 示例 2:
>
> ```
> 输入: ["dog","racecar","car"]
> 输出: ""
> 解释: 输入不存在公共前缀。
> ```

##### 解题思路

* 使用双循环的方式，取出第一个字符串的的一个字符，依次判断是否与剩余字符串的第一个字符相等
* 不相等或者剩余字符串长度不够时返回
* 如果双循环没有返回公共前缀则表示只传入了一个字符串，直接返回即可

##### 示例代码

```go
func longestCommonPrefix(strs []string) string {
	if len(strs) == 0{
		return ""
	}
	for i := 0; i < len(strs[0]); i++ {
		for j := 1; j < len(strs); j++ {
			if i == len(strs[j]) || strs[j][i] != strs[0][i] {
				return strs[0][:i]
			}
		}
	}
	return strs[0]
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.5 MB, 在所有 Go 提交中击败了 35.75%的用户

### 20. 有效的括号

> ##### 题目描述
>
> 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
> 有效字符串需满足：
>
> 1. 左括号必须用相同类型的右括号闭合。
> 2. 左括号必须以正确的顺序闭合。
>    注意空字符串可被认为是有效字符串。
>    示例 1:
>
> ```
> 输入: "()"
> 输出: true
> ```

```
> 示例 2:
>```
> 输入: "()[]{}"
> 输出: true
>```
> 示例 3:
>```
> 输入: "(]"
> 输出: false
```

> 示例 4:
>
> ```
> 输入: "([)]"
> 输出: false
> ```
>
> 示例 5:
>
> ```
> 输入: "{[]}"
> 输出: true
> ```

##### 解题思路

* 使用栈的特效，后进先出，匹配到右括号时去查询栈最后一个是否是相对应的左括号，如果是则从栈中去除，不是则返回 false
* 最后判断剩余栈长度是否为 0
* 使用切片模仿栈操作

##### 示例代码

```go
func isValid(s string) bool {
	brackets := map[int32]int32{
		')': '(',
		'}': '{',
		']': '[',
	}
	list := []int32{}
	for _, v := range s {
		if v == '(' || v == '{' || v == '[' {
			list = append(list, v)
		}else{
			l := len(list) - 1
			if len(list) != 0 && list[l] == brackets[v] {
				list = list[:l]
			}else{
				return false
			}
		}
	}
	return len(list) == 0
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100.00%的用户
> 内存消耗 :2.1 MB, 在所有 Go 提交中击败了 33.87%的用户

### 21.合并两个有序链表

> ##### 题目描述
>
> 将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。
> 示例：
>
> ```
> 输入：1->2->4, 1->3->4
> 输出：1->1->2->3->4->4
> ```

##### 解题思路

* 判断两个链表哪一个较小，然后递归地决定下一个添加到结果里的值。
* 判断两个链表是否为空，为空直接返回

##### 示例代码

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	if l1 == nil {
		return l2
	}
	if l2 == nil {
		return l1
	}
	if l1.Val < l2.Val {
		l1.Next = mergeTwoLists(l1.Next, l2)
		return l1
	} else {
		l2.Next = mergeTwoLists(l1, l2.Next)
		return l2
	}
}
```

##### 运行结果

> 执行用时 :0 ms, 在所有 Go 提交中击败了 100%的用户
> 内存消耗 :2.6 MB, 在所有 Go 提交中击败了 35.89%的用户

### 返回总目录

[日刷leetcode--简单版](https://www.jinjianh.com/leetcode.html)
