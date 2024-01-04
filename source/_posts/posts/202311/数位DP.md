---
title: 数位DP
toc_number: false
abbrlink: 13990
date: 2023-11-07 11:33:46
updated: 2023-11-07 11:33:46
tags:
  - Golang
  - leetcode
categories:
  - - 技术
    - 算法
keywords: '算法,LeetCode,DP,2376. 统计特殊整数,233. 数字 1 的个数,面试题 17.06. 2出现的次数,600. 不含连续1的非负整数,902. 最大为 N 的数字组合,1067. 范围内的数字计数,1397. 找到所有好字符串'
cover: https://gooohlan.fishpi.cn/img/202311122224218.png
description:
katex: true
---

{% note no-icon info%}

站在巨人的肩膀上，此文章思路来源于[灵茶山艾府](https://leetcode.cn/u/endlesscheng)，文章原思路视频版：[数位 DP 通用模板](https://www.bilibili.com/video/BV1rS4y1s721/?vd_source=97fc90767c8a46e683abef8d36c11d39)

{% endnote %}

### 例题：2376. 统计特殊整数

![image-20231108213945185](https://gooohlan.fishpi.cn/img/202311082139219.png)

要解决这道题需要前置知识[「位运算与集合论」](https://gooohlan.cn/skill/algorithm/7087.html)，在这简述一下：

集合可以用二进制表示，二进制从右往左第 $i$ 位为 $1$ 表示 $i$ 在集合中，为 $0$ 表示 $i$ 不在集合中。例如集合 $\left\{ 0,2,3 \right\}$ 对应的二进制数为 $1101_{(2)}$

设集合对应的二进制数为 $x$，本题需要用到两个位运算操作：

- 判断元素 $d$ 是否在集合中：`x >> d &` 可以取出 $x$ 的第 $d$ 个比特位，如果是 $1$ 就说明 $d$ 在集合中
- 把元素 $d$ 添加到集合中：`x = x|(1<<d)`

### 思路：

将 $n$ 转换为字符串 $s$，定义 $f(i,mask,isLimit,isNum)$ 表示构造第 $i$ 位及其之后数位的合法方案数，其余参数的含义：

- $mask$ 表示当前选过的数字集合，换句话说，第 $i$ 位要选的数字不能再 $mask$ 中
- $isLimt$ 表示当前是否受到了 $n$ 的约束（要构造的数不能超过 $n$）。若为真，则第 $i$ 位填入的数字至多为 $s[i]$，否则可以是 $9$，如果在受到了约束的情况下填入了 $s[i]$，那么后续填入的数字任然会受到 $n$ 的约束。例如 $n=135$，那么 $i=0$ 填的是 $1$ 的话，$i=1$ 的这一位最多填 $3$，否则可以填入 $9$
- $isNum$ 表示 $i$ 前面的数位是否填了数字。若为假，则当前位可以跳过（不填数字），或者填入的数字至少为 $1$；若为真，则要填入的数字可以从 $0$ 开始。例如 $n=123$，在 $i=0$ 时跳过的话，相当于后面要构造的是一个 $99$ 以内的数字了，如果 $i=1$ 不跳过，那么相当于构造一个 $10$ 到 $99$ 的两位数，如果 $i=1$ 跳过，相当于构造的是一个 $9$ 以内的数

### 实现细节：

递归入口 `f(0, 0, true, false)`，表示：

- 从 $s[0]$ 开始枚举；
- 一开始集合中没有数字
- 一开始要受到 $n$ 的约束（第一位必须受到约束）
- 一开始没有填数字

递归中：

- 如果 $isNum$ 为假，说明前面没有填数字，那么当前也可以不填数字。一旦从这里递归下去，$isLimit$ 就可以置为 $false$ 了，这是因为 $s[0]$ 必然是大于 $0$ 的，后面就不受到 $n$ 的约束了。或者说，最高位不填数字，后面无论怎么填都比 $n$ 小。
- 如果 $isNum$ 为真，那么当前必须填一个数字，枚举填入的数字，根据 $isNum$ 和 $isLimt$ 来决定填入数字的范围

递归终点：当 $i$ 等于 $s$ 的长度时，如果 $isNum$ 为真，则表示得到了一个合法数字（因为不合法的不会继续递归下去），返回 $1$ ，否则返回 $0$。

### 答疑

**问：**$isNum$ 这个参数是否可以去掉？

**答：** 由于 $mark$ 中记录了数字，可以通过判断 $mark$ 是否为 $0$ 来判断前面是否填了数字，所以 $isNum$ 可以省略。

下面的代码保留了 $isNum$，主要是为了方便掌握模版。因为有些题目不需要 $mark$，但需要 $isNum$。

**问：**记忆化四个状态有点麻烦，能不能只记忆化 $(i,mark)$ 这两个状态？

**答：** 是可以的。比如 $n=234$，第一位填 $2$，第二位填 $3$，后面无论怎么递归，都不会再递归到第一位填 $2$，第二位填 $3$ 的情况，所以不需要记录。又比如，第一位不填，第二位也不填，后面无论怎么递归也不会再次递归到这种情况，所以也不需要记录。

根据这个例子，我们可以只记录不受到 $isLimit$ 或 $isNum$ 约束的状态 $(i,mark)$。比如 $n=234$，第一位（最高位）填的 $1$，那么继续递归，后面就可以随便填，所以状态 $(1,2)$ 就表示前面填了一个 $1$ （对应的 $mark =2$），从第二位往后随便填的方案数。

**问：**能不能只记忆化 $i$ ？

**答：** 这是不行的。想一想，我们为什么要用记忆化？如果递归到同一个状态时，计算出的结果就可能是不一样的。如果只记忆化 $i$，就可能会算出错误的结果。

也可以这样理解：记忆化搜索要求递归函数无副作用（除了修改 `memo` 数组），从而保证递归到同一个状态时，计算出的结果是一样的。

```go
func countSpecialNumbers(n int) int {
    s := strconv.Itoa(n)
    l := len(s)
    memo := make([][1 << 10]int, l)
    for i := range memo {
        for j := range memo[i] {
            memo[i][j] = -1
        }
    }
    
    var f func(int, int, bool, bool) int
    f = func(i int, mark int, isLimit bool, isNum bool) (res int) {
        if i == l {
            if isNum {
                return 1
            }
            return
        }
        
        if !isLimit && isNum {
            p := &memo[i][mark]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        if !isNum { // 跳过当前数位
            res += f(i+1, mark, false, false)
        }
        
        d := 0
        if !isNum {
            d = 1 // 前面没数字,最低填1
        }
        up := 9
        if isLimit {
            up = int(s[i] - '0')
        }
        for ; d <= up; d++ {
            if mark>>d&1 == 0 { // d没选中过
                res += f(i+1, mark|1<<d, isLimit && d == up, true)
            }
        }
        return
    }
    return f(0, 0, true, false)
}
```

### 强化训练（数位 DP）

#### 233. 数字 1 的个数

![image-20231112205906575](https://gooohlan.fishpi.cn/img/202311122059706.png)

根据上面模版定义 $f(i, cnt, isLimit, isNum)$，表示构造到从左到右第 $i$ 位，已经出现了 $cnt$ 个 $1$，在这种情况下，继续构造最终会得到的 $1$ 的个数 （你可以直接从回溯的角度理解这个过程，只不过是多了个记忆化）。

对于本题来说，由于前导零对答案无影响，$isNum$ 可以省略。

```go
func countDigitOne(n int) int {
    s := strconv.Itoa(n)
    l := len(s)
    dp := make([][]int, l)
    for i := range dp {
        dp[i] = make([]int, l)
        for j := range dp[i] {
            dp[i][j] = -1
        }
    }
    
    var f func(int, int, bool) int
    f = func(i int, cnt int, isLimit bool) int {
        if i == l {
            return cnt
        }
        var res int
        if !isLimit {
            p := &dp[i][cnt]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        
        up := 9
        if isLimit {
            up = int(s[i] - '0')
        }
        for d := 0; d <= up; d++ { // 枚举要填入的数字 d
            c := cnt
            if d == 1 { // 满足要求结果+1
                c++
            }
            res += f(i+1, c, isLimit && d == up)
        }
        return res
    }
    return f(0, 0, true)
}
```

#### 面试题 17.06. 2出现的次数

![image-20231112212159355](https://gooohlan.fishpi.cn/img/202311122121382.png)

跟上面的题一样，把 $d==1$ 改为 $d==2$ 即可

```go
func numberOf2sInRange(n int) int {
    s := strconv.Itoa(n)
    l := len(s)
    dp := make([][]int, l)
    for i := range dp {
        dp[i] = make([]int, l)
        for j := range dp[i] {
            dp[i][j] = -1
        }
    }

    var f func(int, int, bool) int
    f = func(i, cnt int, isLimit bool) (res int){
        if i == l {
            return cnt
        }

        if !isLimit {
            p := &dp[i][cnt]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res}()
        }

        up := 9
        if isLimit {
            up = int(s[i]-'0')
        }

        for d := 0; d <= up; d++ {
            c := cnt
            if d == 2 {
                c++
            }
            res += f(i+1, c, isLimit && d == up)
        }
        return
    }
    return f(0, 0, true)
}
```

#### 600. 不含连续1的非负整数

![image-20231112211048043](https://gooohlan.fishpi.cn/img/202311122110283.png)

根据上述通用模版 $f(i, pre, isLimt, isNum)$ 表示构造从左往右第 $i$ 位及其之后数位的合法方案数，其中 $pre$ 表示第 $i-1$ 位是否为 $1$，如果为真则当前位不能填 $1$。

```go

func findIntegers(n int) int {
    s := strconv.FormatInt(int64(n), 2)
    l := len(s)
    dp := make([][2]int, l)
    for i := range dp {
        dp[i] = [2]int{-1, -1}
    }
    
    var f func(int, int8, bool) int
    f = func(i int, pre int8, isLimit bool) (res int) {
        if i == l {
            return 1
        }
        
        if !isLimit {
            p := &dp[i][pre]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        up := 1
        if isLimit {
            up = int(s[i] & 1)
        }
        res = f(i+1, 0, isLimit && up == 0) // 填入0
        if pre == 0 && up == 1 {            // 前一位是0,并且这一位最高可以填1
            res += f(i+1, 1, isLimit)
        }
        return
    }
    return f(0, 0, true)
}
```

#### 902. 最大为 N 的数字组合

![image-20231112211615711](https://gooohlan.fishpi.cn/img/202311122116912.png)

因为这道题填入的数字是直接从 $digits$ 中选择，所以可以胜利第二个参数，然后只记忆化 $i$ 就可以了，因为：

1. 对于一个固定的 $i$，它受到 $isLimit$ 或 $isNum$ 的约束在整个递归过程中至多会出现一次，没必要记忆化。比如 $n=234$，当 $i=2$ 的时候，前面可以填 $11,12,13,...,23$，如果受到 $isLimt$ 的约束，就说明前面填的是 $23$。「当 $i=2$ 的时候，前面填的是 $23$」这件事情，在整个递归过程中至多会出现一次。
2. 另外，如果只记忆化 $i$，$dp$ 数组的含义就变成**在不受到 $n$ 的约束时**的合法方案数，所以要在 `!isLimit && isNum` 成立时才去记忆化。接着上面的例子，在前面填 $23$ 的时候，下一位填的数字不能超过 $4$，因此算出来的结果是不能套用到前面甜的是 $11,12,13,...$ 这些数字上面的。

```go
func atMostNGivenDigitSet(digits []string, n int) int {
    s := strconv.Itoa(n)
    l := len(s)
    dp := make([]int, l)
    for i := range dp {
        dp[i] = -1
    }
    
    var f func(int, bool, bool) int
    f = func(i int, isLimit bool, isNum bool) (res int) {
        if i == l {
            if isNum {
                return 1
            }
            return
        }
        if !isLimit && isNum { // 不受到任何约束
            p := &dp[i]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        
        if !isNum {
            // isLimit 因为当前位置没有填数字,可以跳过
            res += f(i+1, false, false)
        }
        // 根据是否收到约束,决定可以填入数字的上线
        up := byte('9')
        if isLimit {
            up = s[i]
        }
        
        // 对于一般的题目而言,如果此时 isNum 为 false, 则必须从1开始枚举,本体中没有0,所以无需处理这种情况
        for _, d := range digits {
            if d[0] > up {
                break
            }
            res += f(i+1, isLimit && up == d[0], true)
        }
        return
    }
    return f(0,true, false)
}
```

#### 1067. 范围内的数字计数

![image-20231112213310770](https://gooohlan.fishpi.cn/img/202311122133001.png)

这题思路和第一题和第二题一致，因为 $high >= low$ 所以 $high$ 中 $d$ 个数，包含了 $low$ 中 $d$ 的个数，计算出 $high$ 中的个数即可

注意，$low$ 可能是包含 $d$ 的整数，所以求 $low$ 时需要将 $low-1$ 再求，避免最终结果未统计到 $low$

```go
func digitsCount(d int, low int, high int) int {
    return count(high, d) - count(low-1, d)
}

func count(n, x int) int {
    s := strconv.Itoa(n)
    l := len(s)
    dp := make([][]int, l)
    for i := range dp {
        dp[i] = make([]int, l)
        for j := range dp[i] {
            dp[i][j] = -1
        }
    }
    
    var f func(int, int, bool) int
    f = func(i int, cnt int, isLimit bool) (res int) {
        if i == l {
            return cnt
        }
        
        if !isLimit {
            p := &dp[i][cnt]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        
        up := 9
        if isLimit {
            up = int(s[i] - '0')
        }
        
        for d := 0; d <= up; d++ {
            c := cnt
            if d == x {
                c++
            }
            res += f(i+1, c, isLimit && d == up)
        }
        return
    }
    return f(0, 0, true)
}
```

#### 1397. 找到所有好字符串

![image-20231112215256912](https://gooohlan.fishpi.cn/img/202311122152180.png)

依旧是套用上面的模版，不同的是在递归时，进入递归的时机，由`kmp算法提供`，注意几个小细节即可：

- 由于 $next$ 数组中并没有 $-1$，进入下一次递归时，当 $kmp$ 指针在字符串头部时，也会出现 $d \neq evil[nxt]$ 的情况，若直接将 $nxt+1$ 进入递归，是认为两个字符串匹配上了，实际则可能不是，这里需要将 $nxt$ 减 $1$ 后进入递归。使用多一位的 `next` 数组也可以，将循环条件改为 `for nxt > -1 && d != evil[nxt] ` 即可。
- 因为 $s2 >= s1$，所以 `s2` 不包含 `evil` 的子串肯定包含了 `s1` 不包含 `evil` 的子串部分，去掉公共部分即可，跟上题一样。
- 注意 $s2-s1$ 时，$s1$ 可能是一个好字符串，如果是，需要加回来。 

```go
const mod = 1e9 + 7

func findGoodStrings(n int, s1 string, s2 string, evil string) int {
    m := len(evil)
    dp := make([][]int, n)
    for i := range dp {
        dp[i] = make([]int, m)
        for j := range dp[i] {
            dp[i][j] = -1
        }
    }
    
    next := make([]int, m)
    for i := 1; i < m; i++ {
        j := next[i-1]
        for j > 0 && evil[i] != evil[j] {
            j = next[j-1]
        }
        if evil[i] == evil[j] {
            j++
        }
        next[i] = j
    }
    
    var f func(int, int, bool, string) int
    f = func(i int, pos int, isLimit bool, s string) (res int) {
        if pos == m {
            return
        }
        if i == n {
            return 1
        }
        
        if !isLimit {
            p := &dp[i][pos]
            if *p >= 0 {
                return *p
            }
            defer func() { *p = res }()
        }
        
        d := byte(97)
        up := byte(122)
        if isLimit {
            up = s[i]
        }
        for ; d <= up; d++ {
            nxt := pos
            for nxt > 0 && d != evil[nxt] {
                nxt = next[nxt-1]
            }
            // 此处要注意，当 nxt == 0 的时候，会存在 d != evil[nxt] 的情况
            // 若直接 nxt + 1 进入递归，是认为此时的两个字符一定是匹配上了，实际上可能并没有
            if nxt == 0 && d != evil[nxt] {
                nxt = -1
            }
            res += f(i+1, nxt+1, isLimit && d == up, s)
            res %= mod
        }
        return
    }
    res := f(0, 0, true, s2) - f(0, 0, true, s1)
    if res < 0 {
        res += mod
    }
    if strings.Index(s1, evil) == -1 {
        res += 1
    }
    return res
}
```
