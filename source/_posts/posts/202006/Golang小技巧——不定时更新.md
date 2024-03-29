---
title: Go小技巧——不定时更新
date: '2020-06-19 17:55:22'
updated: '2021-01-29 17:08:33'
tags:
  - Golang
categories:
  - - 技术
    - 后端
cover: 'https://gooohlan.fishpi.cn/img/20190201.jpg'
permalink: /articles/2020/06/19/1592560522403.html
abbrlink: 43219
keywords: 'Go小技巧，append，json'
description: 1.函数返回值定义, 2.JSON数组返回NULL, 3.append 函数常见操作
---

# 函数返回值定义

一般的函数定义都是：

```
func Test(a,b int) (int, int){}
```

然而go却可以这样：

```
func Test(a, b int) (c,d int) {}
```

你可能觉得没什么，但是对于我这种懒人来说，这东西可太方便了，因为go没有 `try...catch`，所以所有的错误都需要自己手动抛出，一个函数里你可能有N个↓

```go
if err != nil {
    return err
}
```

实际中，你绝对不会只返回一个 `err`，可能还夹杂着各种乱七八糟的东西，写一次还好，写多了你真的不会烦吗？然而有了第二种定义方式，不过你又多少个返回值，只需要一个 `return`即可搞定。
![991592559676.pic.jpg](https://gooohlan.fishpi.cn/img/991592559676.pic-f90f3138.jpg)

```go
func (b buriedPoint) Retention() (channel, projectId, startTime, endTime string, list []dbmodel.BuriedPointKey, data []map[string]string, err error) {
	//...
	//return channel, projectId, startTime, endTime, list, data, err
	return //选哪个不是一目了然吗，当然实际中不会让你返回这么多，这里有些夸张
}
```

当然上述的方法虽爽，但是也还是会有问题的，让我们再![991592559676.pic.jpg](https://gooohlan.fishpi.cn/img/991592559676.pic-f90f3138.jpg)

```go
func StringToInt(str string) (v int, err error) {
	if true {
		int, err := strconv.Atoi(str)
		if err != nil {
			return
		}
		v = int
	}
	return
}

```

乍一看没啥毛病，但是你运行下看看报不报错就完事了

```shell
./main.go:17:4: err is shadowed during return
```

出现问题就要去解决，提供两种方法↓

方法1：

```go
func StringToInt(str string) (v int, err error) {
	if true {
		int, rErr := strconv.Atoi(str)
		if err != nil {
			err = rErr
			return
		}
		v = int
	}
	return
}
```

方法2：

```go
func StringToInt(str string) (v int, err error) {
	if true {
		int, err := strconv.Atoi(str)
		if err != nil {
			return v, err
		}
		v = int
	}
	return
}
```

到这个时候，它是不是就不这么香了，是否预定义需要根据实际场景决定。

# JSON数组返回NULL

当你的接口返回一个数组，而且数组正好为空时↓

```json
{
    "code":200,
    "msg":"",
    "data":null,
}
```

你可能会返回这样的东西，那么你的前端看了可能会打人(我帮你们问过了)，去翻了下go官方的json包，发现了以下内容：

> Array and slice values encode as JSON arrays, except that []byte encodes as a base64-encoded string, and a nil slice encodes as the null JSON value.

借助翻译软件：

> 数组和切片值编码为JSON数组，但[] byte编码为base64编码的字符串，而nil slice编码为Null JSON值

日常定义数组时，我们一般采用如下两种方式初始化：

```go
var t []int
```

```go
t := []int{}
```

函数返回值定义中定义的与上述两周并无差异，所以也就会返回一样的

定时数组时使用 `make`就可以完全避免这种情况

```go
var t = make([]int, 0)
```

demo：

```go
func main() {
	data := Str{}
	data2 := Str{Array: []string{}}
	var arr = []string{}
	data.Array = arr
	buf, err := json.Marshal(&data)
	log.Println(string(buf), err)
	buf2, err2 := json.Marshal(&data2)
	log.Println(string(buf2), err2)
}

```

# append 函数常见操作

1. 将切片 b 的元素追加到切片 a 之后：`a = append(a, b...)`

2. 复制切片 a 的元素到新的切片 b 上：

   ```go
   b = make([]T, len(a))
   copy(b, a)
   ```

3. 删除位于索引 i 的元素：`a = append(a[:i], a[i+1:]...)`

4. 切除切片 a 中从索引 i 至 j 位置的元素：`a = append(a[:i], a[j:]...)`

5. 为切片 a 扩展 j 个元素长度：`a = append(a, make([]T, j)...)`

6. 在索引 i 的位置插入元素 x：`a = append(a[:i], append([]T{x}, a[i:]...)...)`

7. 在索引 i 的位置插入长度为 j 的新切片：`a = append(a[:i], append(make([]T, j), a[i:]...)...)`

8. 在索引 i 的位置插入切片 b 的所有元素：`a = append(a[:i], append(b, a[i:]...)...)`

9. 取出位于切片 a 最末尾的元素 x：`x, a = a[len(a)-1], a[:len(a)-1]`

10. 将元素 x 追加到切片 a：`a = append(a, x)`
