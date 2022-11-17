---
title: go结构体组合排序
toc_number: false
tags:
  - Golang
categories:
  - - 技术
    - 后端
keywords: 'golang, sort, 排序, 多字段排序, 组合排序'
abbrlink: 14370
date: 2022-05-04 20:30:34
updated: 2022-05-04 21:30:34
cover: https://cdn.gooohlan.cn/img/20220504214250.png
description:
---

&emsp;&emsp;排序是开发中不可避免的，go的标准库[sort](https://pkg.go.dev/sort)提供了这一功能，主要针对`[]int`，`[]float64`，`[]string`、以及其他自定义切片的排序。对于前三个切片基本没有可说的，而对于用户自定义切片，就需要我们自己去实现它的排序了，实现`Interface`接口再调用`sort.Sort()`

举个例子：

```go
type NewInt []int

func (n NewInt) Len() int {
   return len(n)
}

func (n NewInt) Less(i, j int) bool {
   return n[i] < n[j]
}

func (n NewInt) Swap(i, j int) {
   n[i], n[j] = n[j], n[i]
}

func main() {
   n := []int{1,3,2}
   sort.Sort(NewInt(n))
   fmt.Println(n)
}
```

这样写虽然规范，但是略微繁琐，大多时候我更希望简单一点，使用`sort.Slice`可以简化上述代码

```go
func main() {
  people := []struct {
    Name string
    Age  int
  }{
    {"Gopher", 18},
    {"Alice", 20},
    {"Vera", 24},
    {"Bob", 18},
  }
  // 按年龄降序排序
  sort.Slice(people, func(i, j int) bool {
    return people[i].Age > people[j].Age
  })
  fmt.Println("Sort by age:", people)
}
```

这样的好处是简化了许多内容，但是不易于后期维护。

&emsp;&emsp;上述方式实现了一个对结构体单字段的排序，核心在于自定义函数`Lees()`，思考一下，我们能否实现一个基于多字段的排序呢，类似于mysql的`order by`。当然是可以的，实现起来也很简单，当第一个排序索引相同时对第二个排序索引进行排序即可，示例如下。

```go
func main() {
  people := []struct {
    Name string
    Age  int
  }{
    {"Gopher", 18},
    {"Alice", 20},
    {"Vera", 24},
    {"Bob", 18},
  }
  // 按年龄降序排序,年龄相同的按姓名
  sort.Slice(people, func(i, j int) bool {
    if people[i].Age == people[j].Age {
      return people[i].Name < people[j].Name
    }
    return people[i].Age > people[j].Age
  })
  fmt.Println("Sort by age and name:", people)
}
```
