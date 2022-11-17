---
title: Leetcode-数组系列5
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: '算法,LeetCode,数组,Golang'
toc_number: false
abbrlink: 55663
date: 2021-08-01 20:32:33
updated: 2021-08-29 16:17:10
cover: https://cdn.gooohlan.cn/img/20210829163649.png
description: 'LeetCode 115.最小栈，594.最长和谐子序列，202.快乐数，598.范围求和 II，599.两个列表的最小索引总和，645.错误的集合'
---

### 115. 最小栈

设计一个支持`push` ，`pop`，`top`操作，并能在常数时间内检索到最小元素的栈。

- `push(x)` —— 将元素 x 推入栈中。
- `pop()` —— 删除栈顶的元素。
- `top()` —— 获取栈顶元素。
- `getMin()` —— 检索栈中的最小元素。

**示例:**

> **输入：**
> ["MinStack","push","push","push","getMin","pop","top","getMin"]
> [[],[-2],[0],[-3],[],[],[],[]]
>
> **输出：**
> [null,null,null,null,-3,null,0,-2]
>
> **解释：**
> MinStack minStack = new MinStack();
> minStack.push(-2);
> minStack.push(0);
> minStack.push(-3);
> minStack.getMin();   --> 返回 -3.
> minStack.pop();
> minStack.top();      --> 返回 0.
> minStack.getMin();   --> 返回 -2.

**提示：**

pop、top 和 getMin 操作总是在 非空栈 上调用。

#### 解题思路

{% note no-icon info simple %}

因为要输入栈中最小值，所以我们额外定义一个辅助栈，用于入栈时记录当前数此时栈中最小值，出栈时一起出即可。

{% endnote %}

#### 示例代码

```go
func min(a, b int) int {
   if a > b {
      return b
   }
   return a
}

type MinStack struct {
   stack    []int
   minStack []int
}

/** initialize your data structure here. */
func Constructor() MinStack {
   return MinStack{
      stack:    []int{},
      minStack: []int{math.MaxInt64},
   }
}

func (this *MinStack) Push(val int) {
   this.stack = append(this.stack, val)
   this.minStack = append(this.minStack, min(val, this.minStack[len(this.minStack)-1]))
}

func (this *MinStack) Pop() {
   this.stack = this.stack[:len(this.stack)-1]
   this.minStack = this.minStack[:len(this.minStack)-1]
}

func (this *MinStack) Top() int {
   return this.stack[len(this.stack)-1]
}

func (this *MinStack) GetMin() int {
   return this.minStack[len(this.minStack)-1]
}
```

### 594. 最长和谐子序列

和谐数组是指一个数组里元素的最大值和最小值之间的差别 **正好是 1** 。

现在，给你一个整数数组 nums ，请你在所有可能的子序列中找到最长的和谐子序列的长度。

数组的子序列是一个由数组派生出来的序列，它可以通过删除一些元素或不删除元素、且不改变其余元素的顺序而得到。

**示例 1：**

>  输入：nums = [1,3,2,2,5,2,3,7]
> 输出：5
> 解释：最长的和谐子序列是 [3,2,2,2,3]

**示例 2：**

>  输入：nums = [1,2,3,4]
> 输出：2

**示例 3：**

> 输入：nums = [1,1,1,1]
> 输出：0

#### 解题思路

{% note no-icon info simple %}

扫描数组，将扫描到元素`x`时，将`x`存入哈希表，然后获取哈希表中`x-1,x,x+1`出现的次数`a,b,c`，此时`a+b`与`b+c`分别为`x-1,x`与`x, x+1`组成的和谐子序列长度，我们取出其中最大的一个即为最长和谐子序列的长度。

{% endnote %}

#### 示例代码

```go
func findLHS(nums []int) int {
   mapList := make(map[int]int)
   var res int
   for _, num := range nums {
      mapList[num] ++
      if _, ok := mapList[num-1]; ok && res < (mapList[num-1] + mapList[num]){
         res = mapList[num-1] + mapList[num]
      }
      if _, ok := mapList[num+1]; ok && res < (mapList[num+1] + mapList[num]){
         res = mapList[num+1] + mapList[num]
      }
   }
   return res
}
```

### 202. 快乐数

编写一个算法来判断一个数 n 是不是快乐数。

「快乐数」定义为：

- 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
- 然后重复这个过程直到这个数变为 1，也可能是 无限循环 但始终变不到 1。

- 如果 可以变为  1，那么这个数就是快乐数。

如果 n 是快乐数就返回 true ；不是，则返回 false 。

**示例 1：**

> **输入：**19
> **输出：**true
> **解释：**
> 12 + 92 = 82
> 82 + 22 = 68
> 62 + 82 = 100
> 12 + 02 + 02 = 1

**示例 2：**

> **输入：**n = 2
> **输出：**false

{% tabs 202%}

<!-- tab 哈希 -->

#### 解题思路

{% note info no-icon simple %}

对数进行列举发现数总共会出现3种情况：

1. 最终会得到 11。
2. 最终会进入循环。
3. 值会越来越大，最后接近无穷大

第三种情况是最复杂，继续变大我们无法处理，经过实际测试我们发现：

| 位数 | 最大数        | 下一步得到的数 |
| ---- | ------------- | -------------- |
| 1    | 9             | 81             |
| 2    | 99            | 162            |
| 3    | 999           | 243            |
| 4    | 9999          | 324            |
| 13   | 9999999999999 | 1053           |

经过上表可得，3位数最大可得243，因为4位数和4位数以上的数字最后都会降到3位，这意味着这个数要么在243中循环，要么变成1。由此可将算法分为两个部分：

1. 计算n的下一位数
2. 判断该数结果是否出现过，存在过这表示进入循环，不是快乐数

{% endnote %}

#### 示例代码

```go
func isHappy(n int) bool {
   m := make(map[int]bool)
   for ; n != 1 && !m[n]; n, m[n] = step(n), true {
   }
   return n == 1
}

func step(n int) int {
   sum := 0
   for n > 0 {
      sum += (n%10) * (n%10)
      n /= 10
   }
   return sum
}
```

<!-- endtab -->

<!-- tab 快慢指针 -->

#### 解题思路

{% note info no-icon simple %}

哈希解法中的反复调用`step`函数可以看作一个链表中的`getNext`，由此我们得到一个隐形链表，虽然我们没有实际的链表节点和指针，但数据还是形成了一个链表数据结构。

由哈希初分析解法可知，最终结果只有两种，得到1和无限循环。循环在链表中既表示为闭环。采用快慢指针去遍历链表，慢指针每次前进一步，快指针前进两步，如果快指针会追上慢指针，则此链表闭环(不是快乐数)，快指针循环到链表尾部(得到1)，着不闭环(快乐数)

{% endnote %}

#### 示例代码

```go
func isHappyList(n int) bool {
   slow, fast := n, step(n)
   for ;fast != 1 && slow != fast; {
      slow = step(slow)
      fast = step(step(fast))
   }
   return fast == 1
}
```

<!-- endtab -->

{% endtabs %}

### 598. 范围求和 II

<p>给定一个初始元素全部为&nbsp;<strong>0</strong>，大小为 m*n 的矩阵&nbsp;<strong>M&nbsp;</strong>以及在&nbsp;<strong>M&nbsp;</strong>上的一系列更新操作。</p>

<p>操作用二维数组表示，其中的每个操作用一个含有两个<strong>正整数&nbsp;a</strong> 和 <strong>b</strong> 的数组表示，含义是将所有符合&nbsp;<strong>0 &lt;= i &lt; a</strong> 以及 <strong>0 &lt;= j &lt; b</strong> 的元素&nbsp;<strong>M[i][j]&nbsp;</strong>的值都<strong>增加 1</strong>。</p>

<p>在执行给定的一系列操作后，你需要返回矩阵中含有最大整数的元素个数。</p>

<p><strong>示例 1:</strong></p>

<strong>输入:</strong>

> m = 3, n = 3
> operations = [[2,2],[3,3]]
> <strong>输出:</strong> 4
> <strong>解释:</strong>
> 初始状态, M =
> [[0, 0, 0],
>  [0, 0, 0],
>  [0, 0, 0]]
>
> 执行完操作 [2,2] 后, M =
> [[1, 1, 0],
>  [1, 1, 0],
>  [0, 0, 0]]
>
> 执行完操作 [3,3] 后, M =
> [[2, 2, 1],
>  [2, 2, 1],
>  [1, 1, 1]]
>
> M 中最大的整数是 2, 而且 M 中有4个值为2的元素。因此返回 4。

<p><strong>注意:</strong></p>

<ol>
   <li>m 和 n 的范围是&nbsp;[1,40000]。</li>
   <li>a 的范围是 [1,m]，b 的范围是 [1,n]。</li>
   <li>操作数目不超过 10000。</li>
</ol>

{% tabs 598 %}

<!-- tab 暴力法 -->

#### 解题思路

{% note info simple no-icon %}

由于所有的变化都是从`[0,0]`开始，所以[0,0]肯定是最大的，循环求出所有与其相等的即可。{% label 注意：Leetcode提交时会提示内存不足 red %}

{% endnote %}

#### 示例代码

```go
func maxCount(m int, n int, ops [][]int) int {
   arr := make([][]int, m)
   for i := range arr {
      arr[i] = make([]int, n)
   }
   for _, op := range ops {
      for i := 0; i < op[0]; i++ {
         for j := 0; j < op[1]; j++ {
            arr[i][j]++
         }
      }
   }
   max := arr[0][0]
   var count int
   for _, ints := range arr {
      for _, v := range ints {
         if v == max {
            count ++
         }
      }
   }
   return count
}
```

<!-- endtab -->

<!-- tab 重叠区域 -->

#### 解题思路

{% note info simple no-icon %}

由题意可知，每次增加都是从左上角开始，到`[i,j]`，重叠的范围为ops中最小的`(x,y)`，有了范围，最大的元素数目就为`x*y`

{% endnote %}

#### 示例代码

```go
func maxCount(m int, n int, ops [][]int) int {
   for _, op := range ops {
      if m > op[0] {
         m = op[0]
      }
      if n > op[1] {
         n = op[1]
      }
   }
   return m*n
}
```

<!-- endtab -->

{% endtabs %}

### 599. 两个列表的最小索引总和

<p>假设Andy和Doris想在晚餐时选择一家餐厅，并且他们都有一个表示最喜爱餐厅的列表，每个餐厅的名字用字符串表示。</p>


<p>你需要帮助他们用<strong>最少的索引和</strong>找出他们<strong>共同喜爱的餐厅</strong>。 如果答案不止一个，则输出所有答案并且不考虑顺序。 你可以假设总是存在一个答案。</p>


<p><strong>示例 1:</strong></p>

>  <strong>输入:</strong>
>
> [&quot;Shogun&quot;, &quot;Tapioca Express&quot;, &quot;Burger King&quot;, &quot;KFC&quot;]
> [&quot;Piatti&quot;, &quot;The Grill at Torrey Pines&quot;, &quot;Hungry Hunter Steakhouse&quot;, &quot;Shogun&quot;]
> <strong>输出:</strong> [&quot;Shogun&quot;]
> <strong>解释:</strong> 他们唯一共同喜爱的餐厅是&ldquo;Shogun&rdquo;。
> </pre>


<p><strong>示例 2:</strong></p>

> <strong>输入:</strong>
>
> [&quot;Shogun&quot;, &quot;Tapioca Express&quot;, &quot;Burger King&quot;, &quot;KFC&quot;]
> [&quot;KFC&quot;, &quot;Shogun&quot;, &quot;Burger King&quot;]
> <strong>输出:</strong> [&quot;Shogun&quot;]
> <strong>解释:</strong> 他们共同喜爱且具有最小索引和的餐厅是&ldquo;Shogun&rdquo;，它有最小的索引和1(0+1)。

#### 解体思路

{% note no-icon simple info %}

利用哈希表，遍历`list1`，以餐厅名称为`key`，下标为`value`，存入map，再遍历第二个数组，map中存在表示为同时喜欢餐厅，此时将两个下标相加即可得到索引，如果和比之前记录的最小值要小，那么清空返回结果列表，并将此结果添加进去，如果相等则追加

{% endnote %}

#### 示例代码

```go
func findRestaurant(list1 []string, list2 []string) []string {
   var (
      res      = []string{}
      mapList  = make(map[string]int)
      minIndex = math.MaxInt32
   )
   for i, s := range list1 {
      mapList[s] = i
   }
   for i, s := range list2 {
      if v, ok := mapList[s]; ok {
         sum := v + i
         if sum == minIndex {
            res = append(res, s)
         }
         if sum < minIndex {
            minIndex = sum
            res = []string{s}

         }
      }
   }
   return res
}
```

### 645. 错误的集合

<p>集合 <code>s</code> 包含从 <code>1</code> 到 <code>n</code> 的整数。不幸的是，因为数据错误，导致集合里面某一个数字复制了成了集合里面的另外一个数字的值，导致集合 <strong>丢失了一个数字</strong> 并且 <strong>有一个数字重复</strong> 。</p>

<p>给定一个数组 <code>nums</code> 代表了集合 <code>S</code> 发生错误后的结果。</p>

<p>请你找出重复出现的整数，再找到丢失的整数，将它们以数组的形式返回。</p>

<p><strong>示例 1：</strong></p>

> <strong>输入：</strong>nums = [1,2,2,4]
> <strong>输出：</strong>[2,3]


<p><strong>示例 2：</strong></p>

> <strong>输入：</strong>nums = [1,1]
> <strong>输出：</strong>[1,2]

{% tabs 645 %}

<!-- tab 排序 -->

#### 解题思路

{% note no-icon simple info %}

先排序，比较每对相邻的元素，即可找到错误的集合。如果相邻的两个元素相等，则该元素为重复的数字；丢失的数字分为两种情况：

- 如果丢失的数字大于 11且小于 n，两个数的差等于 2，他们之间的数即为丢失的数字；
- 如果丢失的数字是 1 或 n，则需要额外处理。

由于要寻找丢失的数字，所以我们需要记录上一个数字，用来计算两个数字的差，如果丢失的是1，将上一个数字记录为0即可。如果最后一个数字不是n，那么丢失的就是n

{% endnote %}

#### 示例代码

```go
func findRestaurant(list1 []string, list2 []string) []string {
   var (
      res      = []string{}
      mapList  = make(map[string]int)
      minIndex = math.MaxInt32
   )
   for i, s := range list1 {
      mapList[s] = i
   }
   for i, s := range list2 {
      if v, ok := mapList[s]; ok {
         sum := v + i
         if sum == minIndex {
            res = append(res, s)
         }
         if sum < minIndex {
            minIndex = sum
            res = []string{s}

         }
      }
   }
   return res
}
```

<!-- endtab -->

<!-- tab 哈希 -->

#### 解题思路

{% note no-icon simple info %}

遍历数组，用哈希表记录数字出现的次数，再遍历1到n，出现2次的就是重复的，出现0次的就是丢失的

{% endnote %}

#### 示例代码

```go
func findErrorNums(nums []int) []int {
   res := make([]int, 2)
   m := make(map[int]int, 0)
   for _, num := range nums {
      m[num] ++
   }
   for i := 1; i <= len(nums); i++ {
      if v := m[i]; v == 2 {
         res[0] = i
      }else if v == 0 {
         res[1] = i
      }
   }
   return res
}
```

<!-- endtab -->

{% endtabs %}
