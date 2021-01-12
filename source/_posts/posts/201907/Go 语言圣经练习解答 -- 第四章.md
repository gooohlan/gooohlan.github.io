---
title: go 语言圣经练习解答 -- 第四章 (关闭)
date: '2019-07-16 15:54:22'
updated: '2019-07-22 13:50:36'
tags:
  - Golang
  - 学习
  - 教程
categories:
  - - 技术
    - 后端
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/demo2.jpg'
permalink: /articles/2019/06/12/1560331304695.html
abbrlink: 6212
---
## go语言圣经(The Go Programming Language)第四章练习题答案

#### 练习 4.1： 编写一个函数，计算两个SHA256哈希码中不同bit的数目。（参考2.6.2节的 PopCount函数。)

> 解题思路

* 循环字节数组
* 循环字节bit，对比是否相同
<iframe style="border:1px solid" src="https://wide.b3log.org/playground/1de1b0989661e96a455fb5f24c9752d2.go" width="99%" height="600"></iframe>

#### 练习 4.2： 编写一个程序，默认打印标准输入的以SHA256哈希码，也可以通过命令行标准参 数选择SHA384或SHA512哈希算法。
> 解题思路

* 获取命令行输入的参数
* 通过命令行参数返回值
<iframe style="border:1px solid" src="https://wide.b3log.org/playground/8a883544635b919cd2f822312adb81cf.go" width="99%" height="600"></iframe>
> 实际效果

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-0d9ee8d7.png)

#### 练习 4.3: 重写reverse函数,使用数组指针代替slice。
> 解题思路(无)

<iframe style="border:1px solid" src="https://wide.b3log.org/playground/12a10f74a7fc189d271d8c6aa1de3d95.go" width="99%" height="600"></iframe>

#### 练习 4.4: 编写一个rotate函数,通过一次循环完成旋转。

> 解题思路

* 从新创建一个数组，新数组下标为原数组下标加上偏移值
* 如果超出最大长度则从最左边开始

<iframe style="border:1px solid" src="https://wide.b3log.org/playground/c7b0b5fc4d02333f29038bd4953cf880.go" width="99%" height="600"></iframe>

#### 练习 4.6: 写一个函数在原地完成消除[]string中相邻重复的字符串的操作。
> 解题思路

* **原地完成消除** / **相邻重复**
* 原地消除表示必须在原有的数组上操作
* 遇到相同的先前移一位
* 下标保持不动继续检测当前位置是否跟下一位重复
<iframe style="border:1px solid" src="https://wide.b3log.org/playground/7504134d6e7f8ea5167e9afc393338bc.go" width="99%" height="600"></iframe>

#### 练习 4.6: 编写一个函数,原地将一个UTF-8编码的[]byte类型的slice中相邻的空格(参考
unicode.IsSpace)替换成一个空格返回
> 解题思路

* 基本一4.5一致，只是判断字母变成了判断空格

<iframe style="border:1px solid" src="https://wide.b3log.org/playground/4699347511e21456ae37d8d335046732.go" width="99%" height="600"></iframe>

#### 练习 4.7: 修改reverse函数用于原地反转UTF-8编码的[]byte。是否可以不用分配额外的内
存?

> 解题思路

* 与原本的reverse基本一致
<iframe style="border:1px solid" src="https://wide.b3log.org/playground/1f127d8f4ff6c25d302547ebef5b5a35.go" width="99%" height="600"></iframe>
