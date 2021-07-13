---
title: Leetcode-数组系列3
abbrlink: 56303
date: 2021-07-13 11:15:36
updated: 2021-07-13 11:15:36
tags:
  - Golang
  - leetcode
  - 学习
categories:
  - - 技术
    - 算法
keywords: 算法,LeetCode,数组,Golang
description: 'LeetCode 204.计数质数，'
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

{% note info no-icon simple %}

#### 解题思路

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
{% note info no-icon simple %}

#### 解题思路

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

思路来自于埃氏筛，我们将已知的质数存放与一个数组中，遍历数组取出质数`x`，我们不再标记`x`的倍数，而是标记质数集合每一项与当前`i`乘积，切当`i`为质数倍数时结束当前标记，避免了重复标记

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
