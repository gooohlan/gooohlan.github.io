---
title: LeetCode-数组系列2
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 566.重塑矩阵，605.种花问题，628.三个数的最大乘积，643.子数组最大平均数I，661.图片平滑器，665.非递减数列'
abbrlink: 51727
date: 2021-01-22 00:17:09
updated: 2021-01-24 01:22:00
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/1611417706506.jpg
toc_number: false
---
### 返回总目录

[日刷leetcode–简单版](/leetcode.html)

---

# 566. 重塑矩阵

![image-20210122002325449](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210122002325449.png)

{% tabs %}
<!-- tab 方法一 -->

## 解题思路

{% note info no-icon simple %}
创建固定的行，依次循环原数组，同时放入结果数组中，每一行存满后，就添加新的列继续存储
{% endnote %}

## 示例代码

```go
func matrixReshape(nums [][]int, r int, c int) [][]int {
   x, y := len(nums), len(nums[0])
   if r*c != x*y {
      return nums
   }
   i, j := 0, 0
   arr := make([][]int, r)
   for _, row := range nums {
      for _, v := range row {
         arr[i] = append(arr[i], v)
         j++
         if j == c {
            j = 0
            i++
         }
      }
   }
   return arr
}
```
<!-- endtab -->

<!-- tab 方法二 -->

## 解题思路

{% note info no-icon simple %}

将二维数组通过列行遍历改变为一维数组，我们会惊讶的发现原数组的下标`nums[i][j]`变成了一维数组`newNums[n*i+j]`,其中`n`为原二维数组的列数。二维数组的下标`i`和`j`可以分别表示了在第`i`行，第`j`列，由此我们可将一维数组`newNums[i]`变回二维数组`nums[i/c][i%c]`,其中`c`为二维数组的列数。

```go
nums := [][]int{
[]int{1, 2, 3},
[]int{4, 5, 6},
}
newNums := []int{1,2,3,4,5,6}
```

值`5`在两个数组中分别表示为`nums[1][1]`和`nums[4]`，`1(i)*3(n)+1(j) = 4(i)`，`[1(4/3)][1(4%3)]`符合上述推导，故二维数组的转换都可基于一维数组实现

{% endnote %}

## 示例代码

```go
func matrixReshape2(nums [][]int, r int, c int) [][]int {
	x, y := len(nums), len(nums[0])
	if r*c != x*y {
		return nums
	}
	newNums := make([][]int, r)
	for i := 0; i < r; i++ {
		newNums[i] = make([]int, c)
	}
	for i := 0; i < r*c; i++ {
		// 两个二维数组都是基于同一个一维数组变化而来,他们在一维数组中的值是相等的
		newNums[i/c][i%c] = nums[i/y][i%y]
	}
	return newNums
}
```
<!-- endtab -->
{% endtabs %}



# 605. 种花问题

![605.种花问题](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210123141345851.png)

## 解题思路

{% note info no-icon simple %}
数组由`0`和`1`组成，循环一次数组，并分别处理即可

- 当前元素为`1`表示有花，根据规则，下一次有花必定是两格后，直接跳到`i+2`处即可
- 当前元素为`0`表示没有话，因为遇到花就会跳两个，所以前一个位置必定不是花，我们只需要判断下一格是否为`1`即可，不为`1`或者当前格子已经是最后一格，即可种，执行`n--`，然后此格变成了`1`，继续往后跳两格；如果是`1`，则不可种，两格后也不可种，直接跳三格即可

循环结束后查看`n`是否为`0`，为`0`则表示可种`n`朵花，返回`true`，反之返回`false`
{% endnote %}

## 示例代码

```go
func canPlaceFlowers(flowerbed []int, n int) bool {
   for i := 0; i < len(flowerbed) && n > 0; {
      if flowerbed[i] == 1 { // 当前格子已经有花,我们两格后见
         i += 2
      } else if i == len(flowerbed)-1 || flowerbed[i+1] == 0 { // 当前格子不是花,下一个格子也不是花或着当前格子已经是最后一个
         n--
         i += 2
      } else { // 当前格子不是花,但是下一格是
         i += 3
      }
   }
   return n == 0
}
```

# 628. 三个数的最大乘积

![628. 三个数的最大乘积](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210123145818480.png)

{% tabs %}

<!-- tab 方法一 -->

## 解题思路

{% note info no-icon simple %}

首先将数组排序，再根据数组内的数是正负数进行处理

- 数组中的数全部为正数或者全部为负数时，最大乘机为最大的三个数
- 数组中包含正数与负数，则最大乘积有可能是最大三个正数的乘积，也有可能是两个最小负数与最大正数的乘积

{% endnote %}

## 示例代码

```go
func maximumProduct(nums []int) int {
   sort.Ints(nums)
   l := len(nums)
   return max(nums[0]*nums[1]*nums[l-1], nums[l-1]*nums[l-2]*nums[l-3])
}

func max(a,b int) int {
   if a>b {
      return a
   }
   return b
}
```

<!-- endtab -->



<!-- tab 方法二 -->

## 解题思路

{% note info no-icon simple %}

由方法一得知，我们所需的只是最大的3个数以及最小的两个数而已，直接循环一次找出这5个数即可，不用排序。

{% endnote %}

## 示例代码

```go
func maximumProduct2(nums []int) int {
   min1, min2 := math.MaxInt64, math.MinInt64
   max1, max2, max3 := math.MinInt64, math.MinInt64, math.MinInt64
   for _, v := range nums {
      if v < min1 {
         min1, min2 = v, min1
      } else if v < min2 {
         min2 = v
      }

      if v > max1 {
         max1, max2, max3 = v, max1, max2
      } else if v > max2 {
         max2, max3 = v, max2
      } else if v > max3 {
         max3 = v
      }
   }
   return max(min1*min2*max1, max1*max2*max3)
}
```

<!-- endtab -->

{% endtabs %}

# 643. 子数组最大平均数I

![643](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210123181732046.png)

## 解题思路

{% note info no-icon simple %}

使用窗口滑动的方式：先求出`0-k`的和`sum`，再往后依次遍历，求出最大的后除以K即可

{% endnote %}

## 示例代码

```go
func findMaxAverage(nums []int, k int) float64 {
   var maxSum, sum int
   for i := 0; i < k; i++ {
      sum += nums[i]
   }
   maxSum = sum
   for i := k; i < len(nums); i++ {
      sum += nums[i] - nums[i-k]
      if sum > maxSum {
         maxSum = sum
      }
   }
   return float64(maxSum) / float64(k)
}
```

# 661. 图片平滑器

![image-20210123211333512](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210123211333512.png)

## 解题思路

{% note info no-icon simple %}

循环遍历找到每个格子，找所有 9 个包括它自身在内的紧邻的格子，去除无用的格子，将他们的和存在当前位置中，同时记录邻居个数，最后用和除以个数即可。

{% endnote %}

## 示例代码

```go
func imageSmoother(M [][]int) [][]int {
   var x, y = len(M), len(M[0])
   newM := make([][]int, x)
   for i := 0; i < x; i++ {
      newM[i] = make([]int, y)
   }
   for i := 0; i < x; i++ {
      for j := 0; j < y; j++ {
         var count int
         // 寻找邻居格子
         for ii := i-1; ii <= i+1; ii++ {
            for jj := j-1; jj <= j+1; jj++ {
               if ii >=0 && ii < x && jj >=0 && jj < y { // 去除超出原数组范围的数据
                  newM[i][j] += M[ii][jj]
                  count ++
               }
            }
         }
         newM[i][j] /= count
      }
   }
   return newM
}
```

# 665. 非递减数列

![665. 非递减数列](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210124001206690.png)

## 解题思路

{% note info no-icon simple %}

这道题给了我们一个数组，说我们最多有1次修改某个数字的机会，
问能不能将数组变为非递减数组。题目中给的例子太少，不能覆盖所有情况，我们再来看下面三个例子：
	4，2，3
	-1，4，2，3
	2，3，3，2，4
我们通过分析上面三个例子可以发现，当我们发现后面的数字小于前面的数字产生冲突后，
[1]有时候需要修改前面较大的数字(比如前两个例子需要修改4)，
[2]有时候却要修改后面较小的那个数字(比如前第三个例子需要修改2)，
那么有什么内在规律吗？是有的，判断当前数字(`nums[i]`)跟再前面一个数(`nums[i-2]`)的大小有关系，
首先如果再前面的数(`nums[i-2]`)不存在，比如例子1，4前面没有数字了，我们直接修改前面的数字(`nums[i-1]`)为当前的数字(`nums[i]`)2即可。
而当再前面的数字(`nums[i-2]`)存在，并且小于当前数(`nums[i]`)时，比如例子2，-1小于2，我们还是需要修改前面的数字(`nums[i-2]`)4为当前数字(`nums[i]`)2；
如果再前面的数(`nums[i-2]`)大于当前数(`nums[i]`)，比如例子3，3大于2，我们需要修改当前数(`nums[i]`)2为前面的数3(`nums[i-1]`)。

参考：https://leetcode-cn.com/problems/non-decreasing-array/comments/59727

{% endnote %}

## 示例代码

```go
func checkPossibility(nums []int) bool {
   if len(nums) == 0 {
      return true
   }
   var count int
   for i := 1; i < len(nums) && count < 2; i++ {
      if nums[i-1] <= nums[i] {
         continue
      }
      count++
      if i >= 2 && nums[i-2] > nums[i] {
         nums[i] = nums[i-1]
      } else {
         nums[i-1] = nums[i]
      }
   }
   return count <= 1
}
```
