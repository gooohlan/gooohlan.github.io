---
title: 记一次MySQL查询优化
tags:
  - MySQL
categories:
  - - 技术
    - 后端
keywords: 'MySQL慢查询优化,笛卡尔积,MySQL'
description: 多表查询导致的慢查询，从几分钟优化到几秒，再优化到几毫秒；笛卡尔积扫描几十亿数据到扫描几十条数据的最终解决方案
abbrlink: 12472
date: 2021-12-08 21:00:04
updated: 2021-12-08 21:00:04
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20211208234135.png
---

### 起因：

&emsp;&emsp;快下班的时候被同事A叫住，说是某个连表查询导致整个程序卡住，连Debug都停止了，让我帮忙瞅瞅，本着乐于助人的精神，我爽快的答应了。

### 排查：

大致代码如下:

```go
func GetCustomerInfo(){
    ....
    db.Select(".......")
	db.Joins("LEFT JOIN customer_info ON customer_info.customer_id = customer.id")
	db.Joins("LEFT JOIN customer_group ON  customer_group.id = customer.group_id ")
	db.Joins("LEFT JOIN vest_customer_relation ON vest_customer_relation.customer_id = customer.id") // 新加的代码
    ......
    err = db.Debug()Order("customer.id DESC").Limit(limit).Offset(offset).Find(&customers).Error
    return customers, err
}
```

&emsp;&emsp;因为同事A的描述是加了新的连表导致程序直接卡住，再加上打断点调试时都是走到最后一步卡住，倒也没有考虑可能是SQL的问题，甚至觉得是Gorm里面什么奇奇怪怪的错误导致的(原因大概是因为`Gorm`的`Debug()`没有触发，后续猜想应该是需要语句执行完毕才会打印出对应的SQL)。

&emsp;&emsp;因为上诉排查无果，也猜想到可能是因为慢SQL的原因，所以随即手写SQL测试：

```mysql
SELECT * FROM `customer`
	LEFT JOIN customer_info ON customer_info.customer_id = customer.id
	LEFT JOIN customer_group ON customer_group.id = customer.group_id
	LEFT JOIN vest_customer_relation ON vest_customer_relation.customer_id = customer.id
WHERE
	`customer`.`deleted_at` IS NULL
ORDER BY
	customer.id DESC
	LIMIT 10
```

然后，很久过去了……。这是一个慢SQL确认无疑，不过另我好奇的是不过多`join`了一个表而已，为何会这么夸张，让我祭出大杀器`EXPLAIN`：

```mysql
EXPLAIN SELECT * FROM `customer`
	LEFT JOIN customer_info ON customer_info.customer_id = customer.id
	LEFT JOIN customer_group ON customer_group.id = customer.group_id
	LEFT JOIN vest_customer_relation ON vest_customer_relation.customer_id = customer.id
WHERE
	`customer`.`deleted_at` IS NULL
ORDER BY
	customer.id DESC
	LIMIT 10
```

![image-20211208215724805](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20211208215724.png)

从上图不难看出对于表`customer`与`vest_customer_relation`为全表扫描，`customer`倒是理所应当，但是对表`vest_customer_relation`也全表扫描就属实有点离谱了，因为Mysql默认连接方式为[笛卡尔积](https://zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E7%A7%AF)，所以上诉SQL运行时扫描的数据为`33437 * 2 * 1 * 64686`，大概{% label 40亿 red %}的样子，而且据我所知，同事A的业务要写完还需要连接一个表，无论为未写上去的表被扫描的数量是多少，后续的增长都是以{% label 40亿 red %}为单位，这都是一个非常可怕的数量。

### 解决：

&emsp;&emsp;仔细观察`EXPLAIN`的结果得知，表`vest_customer_relation`是没有索引的，所以每次连接表的时候都会去全表扫描，这才导致了一次查询扫描了{% label 40亿 red %}条数据，为表`vest_customer_relation`加上索引即可：

```mysql
ALTER TABLE `gva`.`vest_customer_relation` 
ADD INDEX `vest_customer_relation_customer_id_index`(`customer_id`) USING BTREE;
```

此时执行上文SQL查看效果：

![image-20211208223117625](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20211208223117.png)

可以明显看到，新建的索引精准命中，表`vest_customer_relation`只扫描了一行，总扫描条数也就降到了{% label  3W green %}左右，达到了理想的状态。

### 思考：

&emsp;&emsp;到了这里，问题已经得到了解决，但却不是最优的解决方案，由于用户数据增加的缘故，数据量还会继续增加，如果以后每次遇到类似的问题都通过索引来解决的话，显然不是最佳方案，索引滥用也会导致各种问题。业务问题业务解决，我们应该避免更多笛卡尔积的产生，将SQL拆分，通过业务代码将数据组装才是最佳的解决方案，简单拆分上诉SQL得到：

```mysql
EXPLAIN SELECT * FROM `customer`
	LEFT JOIN customer_info ON customer_info.customer_id = customer.id
	LEFT JOIN customer_group ON customer_group.id = customer.group_id
WHERE
	`customer`.`deleted_at` IS NULL
ORDER BY
	customer.id DESC
	LIMIT 10;
EXPLAIN SELECT * FROM `vest_customer_relation` WHERE customer_id in (3,4,5,6,7,8,9,10,11,12)
```

![image-20211208224854719](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20211208224854.png)

![image-20211208224910207](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20211208224910.png)

因为提前删除了索引，所以两段SQL分别扫描了{% label  3W green %}和{% label  6W green %}而已，加起来也不过{% label  10W green %}，后续在加上索引，完全在可接受的范围之类，后续通过索引之类的优化可以达到更佳。取消所有连表，最后扫描的行数应该为40行，这才是最佳解决方案。

### 后记：

&emsp;&emsp;从SQL语句不难看出，此需求为一个分页查询，那么上诉解决方案仍不是最佳解决方案，卖个关子，可以小小的思考下。

{% hideBlock 解决方案,bg,color %}
[MySQL分页查询优化](https://www.inkdp.cn/articles/2020/10/21/1603213579436.html)
{% endhideBlock %}

