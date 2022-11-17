---
title: Go面试总结
date: '2019-09-25 17:05:55'
updated: '2020-10-26 19:04:24'
tags:
  - 面试
  - Golang
  - MySQL
  - docker
categories: 面试
cover: 'https://cdn.gooohlan.cn/img/100.jpeg'
permalink: /articles/2019/09/25/1569402355322.html
abbrlink: 20413
---

## 前言

俗话说得好，打铁还需自身硬，面对面试官的各种套路，刁钻复杂的各种问题，只有自身实力足够强硬，才能从容不迫的对答如流。
此贴意在总结自身面试中遇到的各种问题，让自己面试前能够得到复习，可以抱抱佛脚。

## Go 基础

基础部分有部分为包的一下基础东西，自行查阅文档或谷歌

1. sync
   参考[浅谈 Golang sync 包的相关使用方法](https://deepzz.com/post/Golang-sync-package-usage.html)
2. channel
3. goroutine
4. reflect
   答：当时答不上来，依稀记得公司大佬说过，反射性能不好，随口答曰，影响性能很少用它?
5. 并发通道安全
6. go支持类的重载吗？
   答：不支持
   6.1. 如果我需要的话可以实现吗？
   答：可以用接口实现（具体实现方式百度）
7. interface
8. 包名/目录名之间的关系
   答：姑且总结为一下几点：

   * import导入的是路径而不是包名
   * 一个文件夹下只能有一个package
   * 尽量让目录名与包名一致(非强制）
   * 代码中使用包时，引用的是包名称而非目录
   * 一个包所有的文件，必须位于同一个目录下
9. 字符串拼接的方式（延升问题，性能比较）
   答：使用运算符、fmt.Sprintf()、strings.Join()、buffer.WriteString()。执行效率如下图（理论上最后一个应该是最快的，不知道是不是我测试用例的原因）
   ![image.png](https://cdn.gooohlan.cn/img/image-b6c8f4f0.png)
10. GC，何时回收，如何手动回收等
11. 并非时如何防止公共变量污染问题

    答：锁或者通道

## MySQL

1. MySQL 事务隔离级别
   了解隔离级别前先了解脏读、不可重复读、幻读这三个概念

   - 脏读：一个事物读取到了另外一个事物未提交更新的数据，事物 A 更新了数据，但未提交，事物 B 读取到了这个更新数据，由于某些原因事物 A 回滚了，而此时事物 B 读取到的是事物 A 未提交的更新数据，此为脏读
   - 不可重复读：在一个事务中多次查询统一数据的到的结果不一致，事物 A 中多次读取数据'a',在此过程中事物 B 更新并提交数据'a',导致事物 A 多次读取的数据'a'不一致，此为不可重复读。可重复读与之相反，即多次读取到的都是同意数据，事物 B 更新并提交后的数据'a'读取不到。
   - 幻读：在一个事物中使用同样的查询语句查询出来的结果不一致(这里的结果不一致体现在结果集个数，而非数据内容不同，数据内容不同为不可重复读)，事物 A 中多次使用同一查询条件查询数据，在此过程中，事物 B 插入了若干条符合事物 A 中查询条件的数据，事物 A 后续查询突然多出若干条数据，此为幻读
   - 小结：不可重复读的和幻读很容易混淆，不可重复读侧**重于修改**，幻读侧重于**新增或删除**

   | 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :- | :-: | :-: | :-: |
| 读未提交(read-uncommitted) | ✓ | ✓ | ✓ |
| 不可重复读(read-committed) | × | ✓ | ✓ |
| 可重复读(repeatable-read) | × | × | ✓ |
| 串行化(serializable) | × | × | × |

2. 索引失效场景

   - 列类型是字符串，查询条件未加引号
   - 未使用索引列作为查询条件
   - 使用了比较操作符 LIKE 和 REGEXP，搜索模板的第一个字符是通配符
   - 在查询条件中使用 OR，如果想要 OR 时索引生效，需要将所有 OR 中每个列都加上索引
   - 对索引列进行运算
   - 查询条件里有不等于号
   - 查询条件里使用了函数
   - 在 JOIN 操作中（需要从多个数据表提取数据时），MySQL 只有在主键和外键的数据类型相同时才能使用索引，否则即使建立了索引也不会使用
   - 在 ORDER BY 操作中，MySQL 只有在排序条件不是一个查询条件表达式的情况下才使用索引。尽管如此，在涉及多个数据表的查询里，即使有索引可用，那些索引在加快 ORDER BY 操作方面也没什么作用
   - 如果 MySQL 估计使用全表扫描要比使用索引快
   - ...
3. MySQL存储引擎的区别与比较

   * MyISAM：有较高的插入、查询速度，但**不支持事务**
   * InnoDB：事务型数据库的首选引擎，支持事务安全表（ACID），支持**行锁定**和**外键，InnoDB是默认的MySQL引擎**
4. Delete，drop与trunkcate的区别

   * delete和truncate操作只删除表中数据，而不删除表结构；delete删除时对于自增类型的字段，值不会从1开始，truncate可以实现删除数据后，自增类型字段从1开始。drop语句将删除表的结构被依赖的约束(constraint)，触发器(trigger)，索引(index)；依赖于该报的存储过程/函数将会保留，但是会变成invalid状态。
   * 属于不同类型的操作，delete属于DML，这个操作会发放到rollback segement中，事务提交后才能生效；如果有相应的trigger，执行的时候将被触发。drop与truncate属于DDL，操作立即生效，原数据不会放到rollback segement中，不能回滚，操作是不触发trigger。
   * delete语句不影响表所占用的extent，高水线(high watermark)保持原位置不动。显然drop语句将所占用的空间全部释放，truncate语句缺省情况下把空间释放到minextents个extent，除非使用reuse storage；truncate会将高水位线复位(回到最开始)
   * 执行速度，drop > truncate > delete
   * 安全性：小心使用drop和truncate，尤其是没有备份的时候
   * 完全删除表[drop]，想保留表而删除所有数据且与事务无关[truncate]，如果和事务有关，或者想触发trigger[delete]

## 算法

1. ```go
   func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
    	return head
    }
    var prev *ListNode
    p := head
    for p != nil {
    	p.Next, prev, p = prev, p, p.Next
    }
    return prev
   }
   ```
2. time，string，regexp，goroutine

```go
package main

import (
	"fmt"
	"regexp"
	"strings"
	"time"
)

func main() {
	// 第一题
	from, _ := time.ParseInLocation("2006-01-02 15:04:05", "2019-09-27 00:00:00", time.Local)
	fmt.Println(sumDiffTime(time.Now(), from))

	// 第二题
	vars := make(map[string]string)
	vars["name"] = "小狮子"
	vars["age"] = "18"
	fmt.Println(printTemplate(vars))

	// 第三题：并发答应1-10，要求顺序输出
	gogogo()
}

type TimeDiff struct {
	Days    int64
	Hours   int64
	Minutes int64
	Ms      int64
}

// sumDiffTime 需要实现以下方法：传入两个不同的时间，计算出两个时间的间隔时间

func sumDiffTime(from time.Time, to time.Time) TimeDiff {
	diff := from.Unix() - to.Unix()
	Days := diff / 60 / 60 / 24
	diff = diff - 60*60*24*Days

	Hours := diff / 60 / 60
	diff = diff - 60*60*Hours
	Minutes := diff / 60
	diff = diff - 60*Minutes
	return TimeDiff{
		Days:    Days,
		Hours:   Hours,
		Minutes: Minutes,
		Ms:      diff,
	}
}

// 需要将str转换为指定字符串:⼩狮⼦今年18岁了,不要⽤字符串拼接
func printTemplate(vars map[string]string) string {
	tp := "{name}今年{age}岁了"
	str := ""

	reg := regexp.MustCompile(`{([a-z]+)}`)     // 创建匹配{name}/{age}的正则
	params := reg.FindAllStringSubmatch(tp, -1) // [[{name} name] [{age} age]]
	// 替换
	str = strings.Replace(tp, params[0][0], vars[params[0][1]], 1)
	str = strings.Replace(str, params[1][0], vars[params[1][1]], 1)

	return str
}

// 并发打印1-10，并按顺序打印
func gogogo() {
	for i := 0; i < 10; i++ {
		ch := make(chan int)
		go func(ch chan int) {
			fmt.Println(<-ch)
		}(ch)
		ch <- i
	}
	time.Sleep(2 * time.Second)
}
```

3. 时间复杂度计算方式

## Docker

1. Docker swarm
   [https://yeasy.gitbooks.io/docker_practice/swarm_mode/overview.html](https://yeasy.gitbooks.io/docker_practice/swarm_mode/overview.html)
2. Docker machine
   [https://www.cnblogs.com/sparkdev/p/7044950.html](https://www.cnblogs.com/sparkdev/p/7044950.html)

## Linux

1. 进程管理
2. netstat的使用

## Redis

1. 讲讲你对Redis的理解

```

```
