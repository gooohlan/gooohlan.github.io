---
title: Redis初探
toc_number: false
date: 2022-10-06 20:03:18
updated: 2022-10-06 20:03:18
tags:
categories:
cover:
keywords:
description:
---

# Redis初探

带着问题去寻找答案往往会是最好的学习方法，开始探究前，先问几个关于Redis的问题：

- Redis有哪些数据结构，其底层是什么
- Redis持久化机制
- Redis为什么是单进程的
- Redis如何保证的原子性
- Redis过期策略及其淘汰机制
- Redis的瓶颈及解决方案

由简入难，一个一个的研究，对每个关键点做到知其然并知其所以然

## Redis有哪些数据结构，其底层结构是什么

`redis`的常见的5种数据结构，分别是`string(字符串)`、`list(列表)`、`hash(哈希)`、`set(集合)`、`sorted set(有序集合)`。这些数据结构是暴露给外部接口调用的。

底层的数据结构有6种：`robj`、`sds(简单动态字符串)`、`dict(字典)`、`intset(整数集合)`、`skiplist(跳跃表)`、`ziplist(压缩列表)`、`quicklist(快速列表)`。

关于底层数据结构的很多，这里就不赘述了：

- [Redis的五种数据结构的底层实现原理(推荐阅读)](https://blog.csdn.net/a745233700/article/details/113449889)

- [一文理解Redis底层数据结构](https://www.modb.pro/db/71948)
- [深入了解Redis底层数据结构](https://juejin.cn/post/6844903936520880135)

对于常用的数据结构已经有了理解，那么不常用的呢？

- Streams

  Redis Stream 主要用于消息队列（MQ，Message Queue），Redis 本身是有一个 Redis 发布订阅 (pub/sub) 来实现消息队列的功能，但它有个缺点就是消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃。

  简单来说发布订阅 (pub/sub) 可以分发消息，但无法记录历史消息。

  而 Redis Stream 提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

- Geospatial

  可以用来保存地理位置，并作位置距离计算或者根据半径计算位置等。有没有想过用Redis来实现附近的人？或者计算最优地图路径？

- HyperLogLog

  供不精确的去重计数功能，比较适合用来做大规模数据的去重统计，例如统计 UV


- Bitmaps

  位图是支持按 bit 位来存储信息，可以用来实现 **布隆过滤器（BloomFilter）**

- Bitfields

  将 Redis 字符串视为一个位数组，并且能够处理具有不同位宽和任意非（必要）对齐偏移量的特定整数字段。实际上，使用此命令可以将位偏移量为1234的带符号5位整数设置为特定值，从偏移量4567中检索31位无符号整数。类似地，该命令处理指定整数的递增和递减，提供保证和良好指定的溢出和下溢行为，用户可以配置

  
