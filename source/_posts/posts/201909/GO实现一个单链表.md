---
title: GO实现一个单链表
date: '2019-09-26 17:08:18'
updated: '2019-09-26 17:08:18'
tags:
  - Golang
  - 算法
categories:
  - - 技术
    - 算法
cover: 'https://goohlan.fishpi.cn/img/20180309.jpg'
permalink: /articles/2019/09/26/1569488898577.html
abbrlink: 1460
---
# 不多BB，直接上代码，有关链表定义，请自行百度

```go
package main

import "fmt"

type ListNode struct {
	Val  interface{}
	Next *ListNode
}

// 初始化
func New() *ListNode {
	return &ListNode{nil, nil}
}

// 遍历输出
func (head *ListNode) Traverse() {
	point := head
	fmt.Println("--------start----------")
	for nil != point {
		fmt.Println(point.Val)
		point = point.Next
	}
	fmt.Println("--------end----------")
}

// 插入
func (head *ListNode) Insert(val int) {
	p := head
	for p.Next != nil {
		p = p.Next // 位移至尾节点
	}
	s := &ListNode{Val: val}
	p.Next = s
	if p.Val == nil { // 插入时发现首节点为空时前移
		p.Val = p.Next.Val
		p.Next = p.Next.Next
	}
}



func main() {
	linkedList := New()
	linkedList.Insert(1)
	linkedList.Insert(2)
	linkedList.Traverse()
	// --------start----------
	// 1
	// 2
	// --------end----------
}
```
