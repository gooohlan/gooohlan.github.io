---
title: Leetcode-数组系列3
abbrlink: 56303
date: 2021-07-13 11:15:36
updated: 2021-07-15 15:57:24
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 204.计数质数，303.区域和检索 - 数组不可变，349.两个数组的交集，350.两个数组的交集II，453.最小操作次数使数组元素相同，455.分发饼干'
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210713160004.jpg
toc_number: false
---

### 返回总目录

[日刷leetcode–简单版](/leetcode.html)

---

### 204. 计数质数

![image-20210713143946282](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210713143946.png)

{% tabs 204 %}
<!-- tab 枚举 -->

#### 解题思路

{% note info no-icon simple %}

根据质数定义可知，除了1和它本身意外不再有其他因数的自然数。对于每一个`x`，我们可以枚举`[2, x-1]`中的每个数`y`，判断`y`是否是`x`的因数。如果`y`是`x`的因数，那么`y`永远在`[2, √x]`范围内

{% endnote %}

#### 示例代码

```go
func countPrimes(n int) int {
   var count int
   for i := 2; i < n; i++ {
      if isPrime(i) {
         count++
      }
   }
   return count
}

func isPrime(x int) bool {
   for i := 2; i*i <= x; i++ {
      if x%i == 0 {
         return false
      }
   }
   return true
}
```

<!-- endtab -->
<!-- tab 埃氏筛 -->

#### 解题思路

{% note info no-icon simple %}

[厄拉多塞筛法](https://zh.wikipedia.org/wiki/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95)的定义不再详细解释，大致的意思是：如果`x`是质数，那么`2x`，`3x`，`4x`….一定不是质数。定义一个数组`isPrime`表示是否为质数，如果`isPrime[i]`是质数，就将`i`的倍数全部标记为非质数。

{% endnote%}

#### 示例代码

```go
func countPrimes(n int) int {
   var count int
   isPrime := make([]bool, n)
   for i := 0; i < n; i++ {
      isPrime[i] = true
   }
   for i := 2; i < n; i++ {
      if isPrime[i] {
         count++
         for j := i * 2; j < n; j += i {
            isPrime[j] = false
         }
      }
   }
   return count
}
```

<!-- endtab -->

<!-- tab 埃氏筛 -->

<!-- tab 线性筛 -->

#### 解题思路

{% note info no-icon simple %}

思路来自于埃氏筛，我们将已知的质数存放与一个数组中，遍历数组取出质数`x`，我们不再标记`x`的倍数，而是标记质数集合每一项与当前`i`乘积，切当`i`为质数倍数时结束当前标记，避免了重复标记

{% endnote %}

#### 示例代码

```go
func countPrimes(n int) int {
   primes := []int{}
   isPrime := make([]bool, n)
   for i := 0; i < n; i++ {
      isPrime[i] = true
   }
   for i := 2; i < n; i++ {
      if isPrime[i] {
         primes = append(primes, i)
      }
      for _, prime := range primes {
         if i*prime >= n {
            break
         }
         isPrime[i*prime] = false
         if i%prime == 0 {
            break
         }
      }
   }
   return len(primes)
}
```

<!-- endtab -->

{% endtabs %}

### 303. 区域和检索 - 数组不可变

![image-20210713165305675](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210713165305.png)

{% tabs 303%}

<!-- tab 直接计算 -->

#### 解题思路

{% note info no-icon simple %}

将原数组存放在NumArray中，在SumRange中求`i`到`j`的和即可

{% endnote %}

#### 示例代码

```go
type NumArray struct {
   sums []int
}

func Constructor(nums []int) NumArray {
   sums := make([]int, len(nums))
   for i, v := range nums {
      sums[i] = v
   }
   return NumArray{sums: sums}
}

func (this *NumArray) SumRange(left int, right int) int {
   var count int
   for i := left; i <= right; i++ {
      count += this.sums[i]
   }
   return count
}
```

<!-- endtab -->

<!-- tab 前缀和 -->

#### 解题思路

{% note info no-icon simple %}

假设数组`nums`的长度为`n`，创建长度为`n+1`的数组`sums`，对于`0 ≤ i < n`都有`sums[i+1] = sums[i] + nums[i]`，`sums[i]`表示了`nums`从下标`0`到`i-1`的前缀和，因为前缀和数组`sums`的长度为`n+1`的缘故，不需要对`i = 0`进行额外处理，所以：`sumRange(i,j)=sums[j+1]−sums[i]`

{% endnote %}

#### 示例代码

```go
type NumArray struct {
   sums []int
}

func Constructor(nums []int) NumArray {
   sums := make([]int, len(nums)+1)
   for i, num := range nums {
      sums[i+1] = sums[i] + num // sum[i+1]存储nums[i]的前缀和
   }
   return NumArray{sums: sums}
}

func (this *NumArray) SumRange(left int, right int) int {
   return this.sums[right+1] - this.sums[left]
}
```

<!-- endtab -->

{% endtabs %}

### 349. 两个数组的交集

![image-20210713173412962](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210714143501.png)

{% tabs 349 %}

<!-- tab Map -->

#### 解题思路

{% note no-icon info simple%}

创建`key`为`int`，`value`为`bool`的map，遍历`nums1`，将数值的元素为`key`值为`true`存入map，循环`nums2`，判断map中是否存在，存在则表示此数在两数组中同事存在，为了避免重复添加，将值改为`flase`，判断是改为map中存在切值为`true`。

处于节省空间考虑，建立map数组时应尽量使用较小的数组

{% endnote%}

#### 实例代码

```go
func intersection(nums1 []int, nums2 []int) []int {
   if len(nums1) > len(nums2){ // 遍历较小的数组生成map
      intersection(nums2, nums1)
   }
   var intersection []int
   mapList := make(map[int]bool)
   for _, v := range nums1 {
      mapList[v] = true
   }

   for _, v := range nums2 {
      if _, ok := mapList[v]; ok && mapList[v] {
         intersection = append(intersection, v)
         mapList[v] = false
      }
   }
   return intersection
}
```

<!-- endtab -->

<!-- tab 排序加双指针 -->

#### 解题思路

{% note no-icon info simple%}

先对两个数组排序，然后双指针循环两个数组，判断指针指向的数是否相等，若相等，则指针后移，交集中不存在则添加值交集数组中，指针所在数不相等则判断两数的大小，较小的数指针后移

{% endnote %}

#### 示例代码

```go
func intersection(nums1 []int, nums2 []int) []int {
   var intersection []int
   sort.Ints(nums1)
   sort.Ints(nums2)
   for i,j := 0,0; i<len(nums1) && j <len(nums2); {
      if nums1[i] == nums2[j] {
         if intersection == nil || intersection[len(intersection)-1] != nums1[i] {
            intersection = append(intersection, nums1[i])
         }
         i++
         j++
      }else if nums1[i] > nums2[j]{
         j++
      } else {
         i++
      }
   }
   return intersection
}
```

<!-- endtab -->

{% endtabs%}

### 350. 两个数的交集 II

![image-20210714111828350](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210714111828.png)

{% tabs 350 %}

<!-- tab Map -->

#### 解题思路

{% note no-icon info simple%}

基于349的map思路，将map的值由bool换成int，循环第一个数组时，记录出现的值的次数，循环第二个数组时判断是否存在且剩余个数大于1则添加为交集

{% endnote%}

#### 实例代码

```go
func intersect(nums1 []int, nums2 []int) []int {
   if len(nums1) > len(nums2) { // 遍历较小的数组生成map
      intersect(nums2, nums1)
   }
   var intersection []int
   mapList := make(map[int]int)
   for _, v := range nums1 {
      mapList[v] ++
   }

   for _, v := range nums2 {
      if _, ok := mapList[v]; ok && mapList[v] > 0 {
         intersection = append(intersection, v)
         mapList[v] --
      }
   }
   return intersection
}
```

<!-- endtab -->

<!-- tab 排序+双指针 -->

#### 解题思路

{% note no-icon info simple%}

因为不需要去重，所以对于两个排好序的数组，直接遍历即可，遇到相同的就添加，不相同时较小的后移

{% endnote %}

#### 示例代码

```go
func intersect(nums1 []int, nums2 []int) []int {
   var intersection []int
   sort.Ints(nums1)
   sort.Ints(nums2)
   for i, j := 0, 0; i < len(nums1) && j < len(nums2); {
      if nums1[i] == nums2[j] {
         intersection = append(intersection, nums1[i])
         i++
         j++
      } else if nums1[i] > nums2[j] {
         j++
      } else {
         i++
      }
   }
   return intersection
}
```

<!-- endtab -->

{% endtabs%}

### 453. 最小操作次数使数组元素相同

![image-20210714150911830](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210714150911.png)

{% tabs 453%}

<!-- tab 排序 -->

#### 解题思路

{% note no-icon info simple%}

先看示例

```
[1,2,3] => [2,3,3] => [3,4,3] => [4,4,4]  3
[1,3,5] => [2,4,5] => [3,5,5] => [4,5,6] => [5,7,5] => [6,7,6] => [7,7,7]  6
[1,2,5] => [2,3,5] => [3,4,5] => [4,5,5] => [5,6,5] => [6,6,6] 5
[1,2,3,4] => [2,3,4,4] => [3,4,5,4] => [4,5,6,4] => [5,6,6,5] => [6,6,7,6] => [7,7,7,7]  6
```

根据以上示例可知，假设数组是有序，每一位加上与最小位相差的数既得到最终的次数以`[1,2,5]`为例，第一位和第二位加`4`后可得`[5,6,5]`，排序可得`[5,5,6]`，重复上述步骤加`1`，得到`[6,6,6]`，共操作5次

{%endnote%}

#### 示例代码

```go
func minMoves(nums []int) int {
   sort.Ints(nums)
   var count int
   for i := len(nums) - 1; i > 0; i-- {
      count += nums[i] - nums[0]
   }
   return count
}
```

<!-- endtab -->

<!-- tab 排序优化 -->

#### 解题思路

{% note no-icon info simple %}

基于排序的思路我们不难发现最终操作次数为数组中所有数组与最小值差的和，也即是数组的和减去N个最小值

{% endnote %}

#### 示例代码

```go
func minMoves(nums []int) int {
   count, min := 0, math.MaxInt32
   for _, num := range nums {
      count += num
      if min > num {
         min = num
      }
   }
   return count - len(nums)*min
}
```

<!-- endtab -->

<!-- tab 动态规划-->

#### 解题思路

{% note no-icon info simple %}

假设数组时有序的，数组长度为3，那么要是三个数相等，可以分两步做

- 先使前两个数相等，则需要移动次数为`nums[1]-nums[0]`记为`moves`。注意：{% label 此时第三个数以静发生了变化，增大了`moves`，`新arr[2]=原arr[2]+moves`  green %}
- 此时前两个数已经相等，相等的数可以看做一个数，此时则需要让后两个数相等，让`num[1]`等于`新num[2]`，则需要移动`新nums[2] - nums[1] = 原nums[2] + moves - nums[1]`记为`diff2`，最终结果为`moves = moves + newmoves`

当数组长度大于3时，依照上面方式递推即可

{% endnote %}

#### 示例代码

```go
func minMoves(nums []int) int {
   sort.Ints(nums)
   var moves int
   for i := 1; i < len(nums); i++ {
      diff := moves + nums[i]- nums[i-1]
      nums[i] += moves // 这里比较难理解,此时相加的实际上是完成上一次操作中的改变nums[2]
      moves += diff
   }
   return moves
}
```



<!-- endtab -->

{% endtabs %}

### 455. 分发饼干

假设你是一位很棒的家长，想要给你的孩子们一些小饼干。但是，每个孩子最多只能给一块饼干。

对每个孩子`i`，都有一个胃口值`g[i]`，这是能让孩子们满足胃口的饼干的最小尺寸；并且每块饼干`j`，都有一个尺寸`s[j]` 。如果`s[j] >= g[i]`，我们可以将这个饼干`j`分配给孩子`i`，这个孩子会得到满足。你的目标是尽可能满足越多数量的孩子，并输出这个最大数值。

**示例 1：**

> **输入**: g = [1,2,3], s = [1,1]
**输出**: 1
**解释**: 
你有三个孩子和两块小饼干，3个孩子的胃口值分别是：1,2,3。
虽然你有两块小饼干，由于他们的尺寸都是1，你只能让胃口值是1的孩子满足。
所以你应该输出1。

**示例 2：**

> 输入: g = [1,2], s = [1,2,3]
> 输出: 2
> 解释: 
> 你有两个孩子和三块小饼干，2个孩子的胃口值分别是1,2。
> 你拥有的饼干数量和尺寸都足以让所有孩子满足。
> 所以你应该输出2.

**提示：**

- `1 <= g.length <= 3 * 104`
- `0 <= s.length <= 3 * 104`
- `1 <= g[i], s[j] <= 231 - 1`

#### 解题思路

{%note no-icon simple info %}

直接将两个两个数组排序，然后双指针的方式向后移，当满足`g[i] <= s[j]`的时候，将饼干`j`分配给孩子`i`，`i`和`j`同时后移为下一个孩子分配饼干；不满足时`j`后移知道满足

{% endnote %}

#### 示例代码

```go
func findContentChildren(g []int, s []int) int {
   sort.Ints(g)
   sort.Ints(s)
   var count int
   for    i,j := 0,0; i<len(g) && j < len(s); {
      if g[i] <= s[j] { // 满足即分配
         count ++
         i++
         j++
      }else{ // 寻找符合孩子胃口的饼干
         j ++
      }
   }
   return count
}
```

