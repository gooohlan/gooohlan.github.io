---
title: Go 语言圣经练习解答 -- 第三章
date: '2019-06-10 17:36:32'
updated: '2019-09-25 14:01:25'
tags:
  - 教程
  - 学习
  - Golang
cover: 'https://cdn.gooohlan.cn/img/100-20210107231539301.jpeg'
categories:
  - - 技术
    - 后端
permalink: /articles/2019/06/10/1560159392016.html
abbrlink: 34374
---

## go语言圣经(The Go Programming Language)第三章练习题答案
### 前言
go语言圣经是一本go入门非常不错的书籍，翻译至The Go Programming Language，本文记录该书章节后练习题答案
* [中文pdf获取地址1](https://github.com/ThomasHuke/books/blob/master/gopl-zh.pdf)  [中文pdf获取地址2](https://books.studyGolang.com/download/gopl-zh.pdf)
* [英文原版获取地址](https://github.com/KeKe-Li/book/blob/master/Go/The.Go.Programming.Language.pdf)
* [中文实体书获取地址](https://https://weidian.com/item.html?itemID=2176920472) (一个还不赖的盗版书网站)
* 本文从第三章练习3.10开始，前面的请查看移步我的[CSDN](https://blog.csdn.net/q1576962841)

#### 练习 3.10： 编写一个非递归版本的comma函数，使用bytes.Buffer代替字符串链接操作。
> 解题思路:

* 参考书中的comma函数，即实现基本的为数字添加逗号分隔符
* 确定了第一个逗号位置后，每隔三个数字添加一个逗号，最后末尾会多出来一个逗号，去掉即可
* 使用bytes.Buffe而非"+"

```go
func comma(s string) string {
    var buffer bytes.Buffer
    l := len(s)

    for i := 0; i < len(s); i++ {
        buffer.WriteString(string(s[i]))
	// 取余3可以得到第一个插入逗号的位置,后面依次+3即可,末尾不加","
        if (i+1)%3 == l%3 {  if (i+1)%3 == l%3 && i != l-1 {
            buffer.WriteString(",")
        }
    }

    s = buffer.String()
    return s
}
````

#### 练习 3.11： 完善comma函数，以支持浮点数处理和一个可选的正负号的处理。
> 解题思路:

* 将整数部分分离处理处理即可，整树部分与3.10相同
* 首先判读第一个字符是否为"+/-"，如果是，将符号添加到buffer中，然后去掉原字符串的第一个字符。
* 通过小数点将字符串分隔为两个数组，下标为0的为整数部分，如果存在小数点则下标为1的为小数部分
* 处理完整数部分后判断是否存在小数部分，存在着添加到buffer中

```go
// 判断是否有正负号
// 判断是否有小数部分
func comma(s string) string {

    var buffer bytes.Buffer

    // 获取正负号
    if s[0] == '-' || s[0] == '+' {
        // 将符号添加到返回的字符串中
        buffer.WriteByte(s[0])
        s = s[1:]
    }

    // 分离整数部分与小数部位
    arr := strings.Split(s, ".")
    s = arr[0]
    l := len(s)

    // 格式整数部分
    for i := 0; i < len(s); i++ {
        buffer.WriteString(string(s[i]))
        // 取余3可以得到第一个插入逗号的位置,后面依次+3即可,末尾不加","
        if (i+1)%3 == l%3 && i != l-1 {
            buffer.WriteString(",")
        }
    }

    // 存在小数部分
    if len(arr) > 1 {
        buffer.WriteString(".")
        buffer.WriteString(arr[1])
    }

    s = buffer.String()
    return s // 末尾会多一个逗号,去掉 + "." + arr[1]
}
```

#### 练习 3.12： 编写一个函数，判断两个字符串是否是是相互打乱的，也就是说它们有着相同的字符，但是对应不同的顺序。
> 解题思路:

* 拥有相同字符那么他们长度肯定是相同的
* 每个字符都有自己的Unicode码，记录每个字符串中每个字符出现的次数
* 循环记录的数组，对比个数是否相同

```go
func isReverse(a, b string) bool {
    // 长度不一样直接返回false
    if len(a) != len(b) {
    return false
    }
    // 用于记录每个字符串出现的次数
    m := make(map[rune]int)
    n := make(map[rune]int)
    // 以字符串Unicode码作为map的Key
    for _, v := range a {
        m[v]++
    }
    for _, v := range b {
    n[v]++
    }
    // 判断相同下标值是否相同
    for i, v := range m {
        if n[i] != v {
            return false
        }
    }
    return true
}
```
#### 练习 3.13： 编写KB、MB的常量声明，然后扩展到YB。
> 1.简单粗暴法(没有解题思路)

```go
const (
    KB = 1000
    MB = KB * KB
    GB = MB * KB
    TB = GB * KB
    PB = TB * KB
    EB = PB * KB
    ZB = EB * KB
    YB = ZB * KB
)
```
> 2.结合书中例子定义KiB到YiB解决
```
KiB = 1024,       KB = 1000
MiB = 1048576,    MB = 1000000
GiB = 1073741824, GB= 1000000000
...
```

* KiB减去24就是KB
* MIB减去48576就是MB
* ...
```go
const (
    _   = 1 << (10 * iota)
    KiB  // 1024
    MiB  // 1048576
    GiB  // 1073741824
    TiB  // 1099511627776 (exceeds 1 << 32)
    PiB  // 1125899906842624
    EiB  // 1152921504606846976
    ZiB  // 1180591620717411303424 (exceeds 1 << 64)
    YiB  // 1208925819614629174706176
)
const (
    KB = 1000
    MB = MiB - MiB % (KB * KB)
    GB = GiB - GiB % (MB * KB)
    TB = TiB - TiB % (GB * KB)
    PB = PiB - PiB % (TB * KB)
    EB = EiB - EiB % (PB * KB)
    ZB = ZiB - ZiB % (EB * KB)
    YB = YiB - YiB % (ZB * KB)
)
```
感觉第一种更加简单粗暴


[返回目录](https://www.jinjianh.com/articles/2019/06/16/1560663440490.html)
