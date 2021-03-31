---
title: 为你的Api加上Swagger
tags:
  - Golang
categories:
  - - 技术
    - 后端
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/mmexport1617101105435.jpg'
keywords: '‘Swagger, gin, golang'''
abbrlink: 21173
date: 2021-03-31 21:09:56
updated: 2021-03-31 23:46:56
---

&emsp;&emsp;一个好的API项目，文档是必不可少的，一份标准的API文档给前后端的沟通带来很多便利。手写，那自然是不可能的。`Swagger`可通过注释的方式实现接口文档，而且生成的接口文档可直接进行请求已完成测试。

# 安装Swagger

- go get

  ```
  go get -u github.com/swaggo/swag/cmd/swag
  ```
  
  因为swagger需要全局使用，请确保 `$GOPATH/bin` 加入`$PATH`，或将可执行文件移动到`$GOBIN`下

- 验证是否安装成功

  ```shell
  ❯ swag -v
  swag version v1.7.0
  ```



# gin-swagger

项目地址:https://github.com/swaggo/gin-swagger

## 引入包

在路由所在文件引入以下包

```go
import (
   swaggerFiles "github.com/swaggo/files"
   ginSwagger "github.com/swaggo/gin-swagger"
)
```

## 编写API描述

在`main.go`函数前添加项目描述文件

```go
// @title Swagger 生成文档测试
// @version 1.0
// @description 这是一个Swagger文档生成测试
// @host localhost:8088
func main() {

}
```

参考[官方示例](https://github.com/swaggo/swag/blob/master/README_zh-CN.md#%E5%A6%82%E4%BD%95%E4%B8%8Egin%E9%9B%86%E6%88%90)，书写对应所需内容

## 编写API注释

先看看官方的示例

```go
// @Summary Show a account
// @Description get string by ID
// @ID get-string-by-int
// @Accept  json
// @Produce  json
// @Param id path int true "Account ID"
// @Success 200 {object} model.Account
// @Header 200 {string} Token "qwerty"
// @Failure 400,404 {object} httputil.HTTPError
// @Failure 500 {object} httputil.HTTPError
// @Failure default {object} httputil.DefaultError
// @Router /accounts/{id} [get]
```

参照`Swagger`的注解规范以及实际情况编写

```go
// @Summary 获取标签
// @Description 通过id获取标签
// @ID get-string-by-int
// @Accept  json
// @Produce  json
// @Param id query int true "标签id"
// @Success 200 {object} TagData
// @Failure 400 {object} HTTPError
// @Router /tag [get]
func (t *tag) Get(c *gin.Context) {
}

// @Summary 添加标签
// @Description 添加标签
// @Accept  json
// @Produce  json
// @Param account body TagData true "标签内容"
// @Success 200 {object} HTTPOk
// @Failure 400 {object} HTTPError
// @Router /tag [POST]
func (t *tag) Add(c *gin.Context) {

}
```

## 生成

到你的项目文件下执行生成命令

```shell
❯ swag init
2021/03/31 23:38:59 Generate swagger docs....
2021/03/31 23:38:59 Generate general API Info, search dir:./
2021/03/31 23:38:59 Generating controller.TagData
2021/03/31 23:38:59 Generating controller.HTTPError
2021/03/31 23:38:59 Generating controller.HTTPOk
2021/03/31 23:38:59 create docs.go at docs/docs.go
2021/03/31 23:38:59 create swagger.json at docs/swagger.json
2021/03/31 23:38:59 create swagger.yaml at docs/swagger.yaml
```

执行成功后会在项目根目录生成文件`docs`

```
docs/
├────   docs.go
├────   swagger.json
└────   swagger.yaml
```

## 引入

在`main.go`中引入包`docs`

```go
import _ "goweb/docs"
```

## 验证

运行项目，访问一下 `http://127.0.0.1:8088/swagger/index.html`， 查看 `API` 文档生成是否正确

![image-20210331234603894](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210331234603894.png)

## 参考

* 本文源代码: [goweb](https://github.com/InkDP/goweb.git)