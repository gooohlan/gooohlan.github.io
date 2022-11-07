---
title: 自动编译proto文件
toc_number: false
tags:
  - Golang
  - protobuf
categories:
  - - 技术
    - 后端
cover: 'https://cdn.inkdp.cn/img/20210927213719.jpg'
keywords: 'golang, proto, grpc, protobuf, goland, ide, 自动编译'
abbrlink: 49446
date: 2021-09-27 19:54:56
updated: 2021-09-27 21:31:34
description:
---

### 前言

你是否和我一样厌烦了无休止的`protobuf`编译，是否对`protoc`的命令深恶痛绝，如果有，那请继续看下去。初接触`protobuf`的我，对他的各种编译命令深恶痛绝，生涩难记。

在同事的帮助下，弄了个编译proto的库：[proto_build](https://github.com/gooohlan/proto_build)，每次改完`proto`文件，直接执行就完事，再也不用去输入各种乱七八糟的命令，简直爽到飞起。

### 使用Goland实现自动编译

**Jetbrains**全家桶提供了`file watcher`的功能，可以实现对文件的监听，文件发生更改时可以执行某些操作，这与我们开发的程序结合，即可解放双手，实现自动编译。

![image-20210927211122811](https://cdn.inkdp.cn/img/20210927211122.png)

下载[proto_build程序包](https://github.com/gooohlan/proto_build/releases)，或下载[源码]((https://github.com/gooohlan/proto_build))后编译，打开**Jetbrains**家的`ide`，这里以`Goland`为例：

`Preferences` → `Tools` → `File Watcher`

![image-20210927211053278](https://cdn.inkdp.cn/img/20210927211053.png)

新建`File Watcher`：点击 `+` → `<custom>`

![image-20210927211252517](https://cdn.inkdp.cn/img/20210927211252.png)

选择监听文件类型`Protocol Buffer`，文件监听范围(根据自己实际需求选择，这里我选择当前项目)，选择上面下载或自行编译的运行程序，工作目录选择当前项目所在目录即可。

![image-20210927212113805](https://cdn.inkdp.cn/img/20210927212113.png)

![image-20210927212208720](https://cdn.inkdp.cn/img/20210927212208.png)

编辑`proto`文件，查看是否自动编译生成对应`.go`文件
