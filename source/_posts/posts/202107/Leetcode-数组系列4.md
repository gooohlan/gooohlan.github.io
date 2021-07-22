---
title: Leetcode-数组系列4
abbrlink: 6574
date: 2021-07-22 18:49:40
updated: 2021-07-22 20:20:45
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 465.岛屿的周长，495.提莫攻击'
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210722202914.png
toc_number: false

---

### 463. 岛屿的周长

给定一个 row x col 的二维网格地图 grid ，其中：grid[i][j] = 1 表示陆地， grid[i][j] = 0 表示水域。

网格中的格子**水平和垂直**方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。

岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。



**示例 1：**

![img](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210722195626.png)

> 输入：grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]]
输出：16
解释：它的周长是上面图片中的 16 个黄色的边


**示例 2：**

> 输入：grid = [[1]]
> 输出：4

**示例 3：**

> 输入：grid = [[1,0]]
> 输出：4

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

> 输入: [1,4], 2
> 输出: 4
> 原因: 第 1 秒初，提莫开始对艾希进行攻击并使其立即中毒。中毒状态会维持 2 秒钟，直到第 2 秒末结束。
> 第 4 秒初，提莫再次攻击艾希，使得艾希获得另外 2 秒中毒时间。
> 所以最终输出 4 秒。

**示例2:**

> 输入: [1,2], 2
> 输出: 3
> 原因: 第 1 秒初，提莫开始对艾希进行攻击并使其立即中毒。中毒状态会维持 2 秒钟，直到第 2 秒末结束。
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
