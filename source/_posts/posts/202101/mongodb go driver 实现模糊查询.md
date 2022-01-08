---
title: MongoDB GO Driver 实现模糊查询
tags:
  - MongoDB
  - 学习
  - Golang
categories:
  - - 技术
    - 后端
cover: 'https://cdn.inkdp.cn/img/1610127892293.jpg'
keywords: '模糊查询, MongoDB, Golang, MongoDB GO Driver'
description: 使用MongoDB官方驱动实现MongoDB GO Driver 实现模糊查询
abbrlink: 61018
date: 2021-01-19 11:23:21
updated: 2021-01-20 00:19:20
---

# MongoDB的模糊查询

&emsp;&emsp;模糊查询时数据库应用中不可缺少的一步，**MySQL**中使用`like`和或者`regexp`来实现实现模糊查询，而**MongoDB**则使用`$regex`操作符或直接使用正则表达式对象来实现。

| MySQL                                         | MongoDB                                  |
| :-------------------------------------------- | :--------------------------------------- |
| select * from users where name like ’%InkDP%’ | db.users.find({name: {$regex: /InkDP/}}) |
| select * from users where name regexp ’InkDP’ | db.users.find({name: /InkDP/})           |

更多相关的语法可查看官方文档：[$regex](https://docs.mongodb.com/manual/reference/operator/query/regex/)，就不再做多讨论。

# 使用MongoDB GO Driver进行查询

先来看看我们的数据源：

```shell
 db.users.find({})
{ "_id" : ObjectId("600704fffc9b483f284d0bc3"), "name" : "1InkDP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc4"), "name" : "InkDPPP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc5"), "name" : "InkDP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc6"), "name" : "inkdp123" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc7"), "name" : "abcdef" }
{ "_id" : ObjectId("60070500fc9b483f284d0bc8"), "name" : "test" }
```

然后执行模糊查询：

```shell
db.users.find({name:{$regex: /InkDP/,$options: "i"}})
{ "_id" : ObjectId("600704fffc9b483f284d0bc3"), "name" : "1InkDP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc4"), "name" : "InkDPPP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc5"), "name" : "InkDP" }
{ "_id" : ObjectId("600704fffc9b483f284d0bc6"), "name" : "inkdp123" }
```

## 错误尝试

上述方式是MongoDB的命令行的执行方式，如果我们直接在Go里面直接这样写是行不通的

```go
filter := bson.M{
   "name": bson.M{
      "$regex":   "/InkDP/",
      "$options": "i",
   },
}
```

当你兴高采烈地拿着上面的查询条件去查询时，你会发现它会返回一个空数组给你

## 正确的使用方式

```go
filter := bson.M{
	"name": primitive.Regex{
		Pattern:"/InkDP/",
		Options: "i",
	},
}
```

执行后发现还是没有，一番查找后才发现`Pattern`不再额外需要两个`/`，直接填写正则内容即可，所以我们改为：

```go
filter := bson.M{
   "name": primitive.Regex{
      Pattern:"InkDP",
      Options: "i",
   },
}
```

执行结果为：

```shell
{ID:ObjectID("600704fffc9b483f284d0bc3") Name:1InkDP}
{ID:ObjectID("600704fffc9b483f284d0bc4") Name:InkDPPP}
{ID:ObjectID("600704fffc9b483f284d0bc5") Name:InkDP}
{ID:ObjectID("600704fffc9b483f284d0bc6") Name:inkdp123}
```

与命令行查找的一致，说明没有问题
