---
title: Leetcode-数组系列4
abbrlink: 6574
date: 2021-07-22 18:49:40
updated: 2021-07-27 20:57:50
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 465.岛屿的周长，495.提莫攻击，496.下一个更大元素I，500.键盘行，506.相对名次，575.分糖果'
cover: https://cdn.inkdp.cn/img/20210722202914.png
toc_number: false

---

### 463. 岛屿的周长

给定一个 row x col 的二维网格地图 grid ，其中：grid[i][j] = 1 表示陆地， grid[i][j] = 0 表示水域。

网格中的格子**水平和垂直**方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。

岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。



**示例 1：**

![img](https://cdn.inkdp.cn/img/20210722195626.png)

> **输入**：grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]]
**输出**：16
**解释**：它的周长是上面图片中的 16 个黄色的边


**示例 2：**

> **输入**：grid = [[1]]
> **输出**：4

**示例 3：**

> **输入**：grid = [[1,0]]
> **输出**：4

{% tabs 463 %}
<!-- tab 公式法 -->

#### 解题思路

{% note info no-icon simple %}

通过观察不难看出每个格子都有4条边，而两个格子相邻时会去掉两条边，所以周长应该为 `4*格子数量 - 2*响铃的边`；由于格子相邻是互相的，所以寻找相邻的格子时只寻找右下方的即可

{% endnote %}

#### 示例代码

```go
func islandPerimeter(grid [][]int) int {
   var count, edge int
   for i := 0; i < len(grid); i++ {
      for j := 0; j < len(grid[i]); j++ {
         if grid[i][j] == 0 {
            continue
         }
         count++
         if j+1 < len(grid[i]) && grid[i][j+1] == 1 {
            edge++
         }
         if i+1 < len(grid) && grid[i+1][j] == 1 {
            edge++
         }
      }
   }
   return 4*count - 2*edge
}
```

<!-- endtab -->

<!-- tab DFS -->

#### 解题思路

{% note info no-icon simple %}

因为总共就一个岛屿，所以我们从遇到的第一块土地开始，基于他递归上下左右4个点：从土地到土地不会产生周长，从土地到海洋会产生周长，从土地到边界会参数周长，将这些周长相加即可。

在上述过程中，递归会导致重复，造成重复计算，将遍历过得土地做一个特殊标记，区别于1和0表示已经访问过了。

{% endnote %}

#### 示例代码

```go
func islandPerimeter(grid [][]int) int {
   for i := 0; i < len(grid); i++ {
      for j := 0; j < len(grid[i]); j++ {
         if grid[i][j] == 1 {
            return dfs(grid, i, j)
         }
      }
   }
   return 0
}

func dfs(grid [][]int, x, y int) int {
   if x < 0 || x >= len(grid) || y < 0 || y >= len(grid[0]) {
      return 1
   }
   if grid[x][y] == 0 {
      return 1
   }
   if grid[x][y] == 2 {
      return 0
   }
   grid[x][y] = 2
   return dfs(grid, x-1, y) + dfs(grid, x+1, y) + dfs(grid, x, y-1) + dfs(grid, x, y+1)
}
```

<!-- endtab -->

{% endtabs %}

### 495. 提莫攻击

在《英雄联盟》的世界中，有一个叫 “提莫” 的英雄，他的攻击可以让敌方英雄艾希（编者注：寒冰射手）进入中毒状态。现在，给出提莫对艾希的攻击时间序列和提莫攻击的中毒持续时间，你需要输出艾希的中毒状态总时长。

你可以认为提莫在给定的时间点进行攻击，并立即使艾希处于中毒状态。



**示例1:**

> **输入**: [1,4], 2
> **输出**: 4
> **原因**: 第 1 秒初，提莫开始对艾希进行攻击并使其立即中毒。中毒状态会维持 2 秒钟，直到第 2 秒末结束。
> 第 4 秒初，提莫再次攻击艾希，使得艾希获得另外 2 秒中毒时间。
> 所以最终输出 4 秒。

**示例2:**

> **输入**: [1,2], 2
> **输出**: 3
> **原因**: 第 1 秒初，提莫开始对艾希进行攻击并使其立即中毒。中毒状态会维持 2 秒钟，直到第 2 秒末结束。
> 但是第 2 秒初，提莫再次攻击了已经处于中毒状态的艾希。
> 由于中毒状态不可叠加，提莫在第 2 秒初的这次攻击会在第 3 秒末结束。
> 所以最终输出 3 。

#### 解题思路

{% note  info no-icon simple %}

先考虑两个相邻攻击时间节点`timeSeries[i]`和`timeSeries[i-1]`的差，如果大于攻击持续时间`duration`，则第`i-1`次攻击持续

`duration`，反之则持续`timeSeries[i] - timeSeries[i-1]`，最后一次攻击始终会持续`duration`，将它们相加即可

{% endnote %}

#### 示例代码

```go
func findPoisonedDuration(timeSeries []int, duration int) int {
   if len(timeSeries) == 0 {
      return 0
   }
   var total int
   for i := 1; i < len(timeSeries); i++ {
      if timeSeries[i]-timeSeries[i-1] < duration {
         total += timeSeries[i] - timeSeries[i-1]
      } else {
         total += duration
      }
   }
   return total + duration
}
```

### 496. 下一个更大元素 I

给你两个**没有重复元素**的数组`nums1`和`nums2` ，其中`nums1`是`nums2`的子集。

请你找出`nums1`中每个元素在`nums2`中的下一个比其大的值。

nums1 中数字 x 的下一个更大元素是指 x 在 nums2 中对应位置的右边的第一个比 x 大的元素。如果不存在，对应位置输出 -1 。



**示例 1** :

> **输入**: nums1 = [4,1,2], nums2 = [1,3,4,2].
> **输出**: [-1,3,-1]
> **解释**:
>      		对于 num1 中的数字 4 ，你无法在第二个数组中找到下一个更大的数字，因此输出 -1 。
>      		对于 num1 中的数字 1 ，第二个数组中数字1右边的下一个较大数字是 3 。
>      		对于 num1 中的数字 2 ，第二个数组中没有下一个更大的数字，因此输出 -1 。

**示例 2**:

> **输入**: nums1 = [2,4], nums2 = [1,2,3,4].
> **输出**: [3,-1]
> **解释**:
>     	对于 num1 中的数字 2 ，第二个数组中的下一个较大数字是 3 。
>   	  对于 num1 中的数字 4 ，第二个数组中没有下一个更大的数字，因此输出 -1 。

**提示**：

- `1 <= nums1.length <= nums2.length <= 1000`
- `0 <= nums1[i], nums2[i] <= 104`
- `nums1`和`nums2`中所有整数 互不相同
- `nums1` 中的所有整数同样出现在 `nums2` 中

**进阶**：你可以设计一个时间复杂度为`O(nums1.length + nums2.length)`的解决方案吗？

{% tabs 496 %}
<!-- tab 暴力法 -->

#### 解题思路

{% note  info no-icon simple %}

循环`nums1`，在`nums2`中查找它的位置，并想有遍历寻找第一个比它大的元素

{% endnote %}

#### 示例代码

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
   res := make([]int, len(nums1))
   for i := 0; i < len(nums1); i++ {
      j := 0
      for ;j<len(nums2) && nums1[i] != nums2[j]; {
         j++
      }
      for ;j<len(nums2); j ++ {
         if nums1[i]< nums2[j] {
            res[i] = nums2[j]
            break
         }
      }
      if j == len(nums2){
         res[i] = -1
      }
   }
   return res
}
```
<!-- endtab -->
<!-- tab 单调栈 -->

#### 解题思路

{% note no-icon info simple %}

由于`nums1`是`nums2`的子集，整体思路就是求出`nums2`中每个元素对应的下一个更大元素，存在map中，其中`key`为`nums2`中的元素值，`value`为下一个更大元素。

正序遍历数组`nums2`，对于当前考察元素`nums2[i]`来说：

- 如果栈为空，将`nums[i]`入栈，继续向后遍历，看是否有大于`nums[i]`的
- 如果栈不为空且其值大于栈顶元素，由于正序遍历所以当前元素`nums[i]`是栈顶元素的下一个更大元素。所以栈顶元素出栈作为`key`，`nums[i]`作为`value`存入map。对于出栈后的新栈，继续上诉操作，知道`nums[i]`不再大于栈顶元素或者栈为空时入栈。

数组遍历完后，查看栈是否为空，不为空时栈类元素为没有更大的元素，值为`-1`

{% endnote %}

#### 示例代码

```go
func nextGreaterElement(nums1 []int, nums2 []int) []int  {
   stack := make([]int, 0)
   mapList := make(map[int]int)
   for _, v := range nums2 {
      if len(stack) == 0 {
         stack = append(stack, v)
         continue
      }
      i := 0
      for ; i < len(stack); i++ {
         if v > stack[i] {
            break
         }
      }
      for j := i; j < len(stack); j++ {
         mapList[stack[j]] = v
      }
      stack = stack[:i]
      stack = append(stack, v)
   }
   for _,v := range stack {
      mapList[v] = -1
   }
   for k, v := range nums1 {
      nums1[k] = mapList[v]
   }
   return nums1
}
```

<!-- endtab -->

{% endtabs%}

### 500. 键盘行

给你一个字符串数组 words ，只返回可以使用在**美式键盘**同一行的字母打印出来的单词。键盘如下图所示。

**美式键盘**中：

- 第一行由字符`"qwertyuiop"`组成。
- 第二行由字符`"asdfghjkl" 组成。
- 第三行由字符`"zxcvbnm"`组成。

![American keyboard](https://cdn.inkdp.cn/img/20210726193833.png)

**示例 1**：

> **输入：**words = ["Hello","Alaska","Dad","Peace"]
> **输出：**["Alaska","Dad"]

**示例 2**：

> **输入：**words = ["omk"]
> **输出：**[]

**示例 3：**

> **输入：**words = ["adsdf","sfd"]
> **输出：**["adsdf","sfd"]

**提示：**

- `1 <= words.length <= 20`
- `1 <= words[i].length <= 100`
- `words[i]`由英文字母（小写和大写字母）组成

{% tabs 500 %}

<!-- tab 哈希 -->

#### 解题思路

{% note no-icon simple info %}

以键盘字母为`key`，所在行数为`value`存入map。遍历words中每一个单词的字母，是否在同一行

{% endnote %}

#### 示例代码

```go
func findWords(words []string) []string {
   var (
      res []string
      keyboard map[byte]int
   )
   keyboard = map[byte]int{
      'q': 0, 'w': 0, 'e': 0, 'r': 0, 't': 0, 'y': 0, 'u': 0, 'i': 0, 'o': 0, 'p': 0,
      'a': 1, 's': 1, 'd': 1, 'f': 1, 'g': 1, 'h': 1, 'j': 1, 'k': 1, 'l': 1,
      'z': 2, 'x': 2, 'c': 2, 'v': 2, 'b': 2, 'n': 2, 'm': 2,
   }
   for _, word := range words {
      b := true
      lowercase := strings.ToLower(word)
      for i := 1; i < len(lowercase); i++ {
         if keyboard[lowercase[i-1]] != keyboard[lowercase[i]] {
            b = false
            break
         }
      }
      if b {
         res = append(res, word)
      }
   }
   return res
}
```

<!-- endtab -->

<!-- tab 标准库 -->

#### 解题思路

{% note no-icon info simple %}

利用`strings`库提供的`ContainsAny`函数，判断输入的单词是否出现在键盘的某一行中，如果一个单词在多行中出现则表示不符合标准

{% endnote %}

#### 示例代码

```go
func findWords2(words []string) []string {
   line1 := "qwertyuiopQWERTYUIOP"
   line2 := "asdfghjklASDFGHJKL"
   line3 := "zxcvbnmZXCVBNM"
   var res []string
   for _, word := range words{
      b1 := strings.ContainsAny(word, line1)
      b2 := strings.ContainsAny(word, line2)
      b3 := strings.ContainsAny(word, line3)
      if (b1 && !b2 && !b3) || (!b1 && b2 && !b3) || (!b1 && !b2 && b3){
         res = append(res, word)
      }
   }
   return res
}
```

<!-- endtab -->

{% endtabs %}

### 506. 相对名次

给出**N**名运动员的成绩，找出他们的相对名次并授予前三名对应的奖牌。前三名运动员将会被分别授予 “金牌”，“银牌” 和“ 铜牌”（"Gold Medal", "Silver Medal", "Bronze Medal"）。

(注：分数越高的选手，排名越靠前。)

**示例 1:**

> **输入:** [5, 4, 3, 2, 1]
> **输出:**  ["Gold Medal", "Silver Medal", "Bronze Medal", "4", "5"]
> **解释:**  前三名运动员的成绩为前三高的，因此将会分别被授予 “金牌”，“银牌”和“铜牌” ("Gold Medal", "Silver Medal" and "Bronze Medal").
> 余下的两名运动员，我们只需要通过他们的成绩计算将其相对名次即可。

**提示:**

1. N是一个正整数并且不会超过 10000。
2. 所有运动员的成绩都不相同。

#### 解题思路

{% note no-icon info simple %}

1. 将原位置存入Hash
2. 排序
3. 循环排序后的数组，从map中出去原位置作为下标加入返回数组中

{% endnote %}

#### 示例代码

```go
func findRelativeRanks(score []int) []string {
   mapList := make(map[int]int, len(score))
   for k, v := range score {
      mapList[v] = k
   }
   sort.Ints(score)
   res := make([]string, len(score))
   for i := 0; i < len(score); i++ {
      if len(score)-i == 1 {
         res[mapList[score[i]]] = "Gold Medal"
         continue
      }
      if len(score)-i == 2 {
         res[mapList[score[i]]] = "Silver Medal"
         continue
      }
      if len(score)-i == 3 {
         res[mapList[score[i]]] = "Bronze Medal"
         continue
      }
      res[mapList[score[i]]] = strconv.Itoa(len(score) - i)
   }

   return res
}
```

### 575. 分糖果

给定一个**偶数**长度的数组，其中不同的数字代表着不同种类的糖果，每一个数字代表一个糖果。你需要把这些糖果**平均**分给一个弟弟和一个妹妹。返回妹妹可以获得的最大糖果的种类数。

**示例 1:**

> **输入:** candies = [1,1,2,2,3,3]
> **输出:** 3
> **解析:** 一共有三种种类的糖果，每一种都有两个。
>      	最优分配方案：妹妹获得[1,2,3],弟弟也获得[1,2,3]。这样使妹妹获得糖果的种类数最多。

**示例 2 :**

> **输入:** candies = [1,1,2,3]
> **输出:** 2
> **解析:** 妹妹获得糖果[2,3],弟弟获得糖果[1,1]，妹妹有两种不同的糖果，弟弟只有一种。这样使得妹妹可以获得的糖果种类数最多。

**注意:**

1. 数组的长度为[2, 10,000]，并且确定为偶数。
2. 数组中数字的大小在范围[-100,000, 100,000]内。

#### 解题思路

{% note no-icon info simple %}

因为是平均分配，所以妹妹糖果的情况为：

- 最多：所有糖果数量的一半(此时糖果种类大于等于糖果总数的一半)
- 最少：1种(所有糖果为一类)

所以妹妹获得糖果种类为：`min(总数的一半,糖果种类)`

{% endnote %}

#### 示例代码：排序

```go
func distributeCandies(candyType []int) int {
   sort.Ints(candyType)
   count := 1
   for i := 1; i < len(candyType) && count < len(candyType)/2; i++ {
      if candyType[i] != candyType[i-1] {
         count++
      }
   }
   return count
}
```

#### 示例代码：Map

```go
func distributeCandies(candyType []int) int {
   mapList := make(map[int]struct{}, 0 )
   for _, v := range candyType {
      mapList[v] = struct{}{}
   }
   if len(mapList) < len(candyType)/2 {
      return len(mapList)
   }
   return len(candyType)/2
}
```
