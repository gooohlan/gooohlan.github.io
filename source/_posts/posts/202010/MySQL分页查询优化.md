---
title: MySQL分页查询优化
date: '2020-10-21 01:06:19'
updated: '2020-10-21 01:13:29'
tags:
  - MySQL
  - 学习
categories:
  - - 技术
    - 后端
  - 踩坑日记
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20200617.jpg'
permalink: /articles/2020/10/21/1603213579436.html
abbrlink: 19624
---


### 前言

Mysql慢查询优化，一直是开发中不可避免的问题，当然面试的时候也是。

今天的面试中，面试的最后一道题：“如何提供分页查询”，我自信的写下 `LIMIT`，认为此题十拿九稳，面试官此后的问题为当 `offset`到一定数量的时候怎么优化，因为之前没有遇到过类似的问题，而且也没有量特别大的分页，所以这个问题只能作罢。

### 复盘

回家后弄了个大概有快20W数据的表，实测一下，查询速度是否会因为 `limit`边大而边长。
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-f1d086a6.png)

如上图所示，同样的查询条件下，因为 `limit`增大查询速度确实变慢了很多。

#### why?

对于limit子句 `LIMIT [offset,] row_count`，官网说明如下

* The`offset` specifies the offset of the first row to return. The`offset` of the first row is 0, not 1.
* The`row_count` specifies the maximum number of rows to return.
* 翻译一下就是：
* `offset`参数指定要返回的第一行的偏移量。第一行的偏移量为`0`，而不是`1`。
* `count`指定要返回的最大行数。
*

因为要偏移到 `offset`处，所以就要先扫描前 `offset`行，所以随着 `limit`边大，也就越来越慢。

#### 解决

随意Google一下，就找到了两种解决办法，分别贴出对于SQL以作参考
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-dbe47374.png)

因为数据量小，然后数据没有特意去设计，所以整体来说效果一般，但是还是有所提升

#### 思考

针对limit的优化，更多的应该是让limit去尽量少的偏移数据，具体步骤如下：

* 使用索引列或者主键作为`order by`操作列
* 记录上次查询的主键，作为下次查询时主键的筛选条件
  ![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-bef7270a.png)

参考：[性能优化之分页查询](https://segmentfault.com/a/1190000017059239?utm_source=sf-related)
