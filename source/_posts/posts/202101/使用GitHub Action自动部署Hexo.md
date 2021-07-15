---
title: 使用GitHub Action自动部署Hexo
abbrlink: 3295
date: 2021-01-16 21:29:54
updated: 2021-06-22 20:28:15
tags:
  - Hexo
categories:
  - - 技术
    - 前端
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/1610127892296.png
keywords: Hexo,GitHub Action,自动部署,Github Page,博客
description: 本文主要介绍如何通过GitHub Action自动部署Hexo博客到GitHub Page以及自己的服务器
---

# 前言

&emsp;&emsp;`Hexo`+`Github Page`让很多没用服务器的人也拥有了自己的博客，添加好配置文件后可以通过`hexo d`直接部署到服务器上，这种方式部署到`Github Page`是挺方便，但是如果要部署到自己的服务器就比较麻烦，得编译后再上传到自己的服务器。

&emsp;&emsp;Github Actions 可以很方便实现 CI/CD 工作流，通过它可以实现抓取代码、运行测试、登录远程服务器，发布到第三方服务等等，我们使用它来自动部署博客到想部署的位置。

# 思路

&emsp;&emsp;建立`yourname.github.io`库，本地写好文章后推送到`hexo`分支，自动编译后部署到`master`分支，然后在上传到我自己的服务器，这样就实现了[inkdp.github.io](https://inkdp.github.io)和[inkdp.cn](https://inkdp.cn)的同时部署。

# 开始

## 创建所需仓库

创建`yourname.github.io`库，并创建`hexo`分支，`hexo`分支用户存储博客源码，默认的`master`分支用于存放编译好的GitHub源码。

## 配置部署密钥

使用以下命令生成部署密钥(一路回车到底即可)

```shell
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
# You will get 2 files:
#   gh-pages.pub (public key)
#   gh-pages     (private key)
```

<font color="#dd0000">切记将生成的密钥文件添加到`.gitignore`中！！！</font>

接下来,去仓库设置

*  复制`gh-pages.pub`的内容，仓库的 `Settings -> Deploy keys -> Add deploy key` 页面粘贴你的内容，`Title`可随意填写，<font color="#dd0000">并勾选`Allow write access`，实测可不配置</font>

* 复制`gh-pages`的内容，去仓库的`Settings -> Secrets -> Add a new secret` 页面上粘贴你的内容，`Name`字段填写为`ACTION_DEPLOY_KEY(可更改)`

  | 添加你的公钥                                                 | 成功                                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | ![image-20210117235120550](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210117235120550.png) | ![image-20210117235147362](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210117235147362.png) |

  | 添加你的私钥                                                 | 成功                                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | ![image-20210117235828106](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210117235828106.png) | ![image-20210117235912296](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210117235912296.png) |

  `GitHub Secret`可以用来存储一些私密内容，类似一些私钥，Key之类的，在CI中通过`${{ Secret Name}}`取出对应的值

## 编写 Github Actions

在`yourname.github.io`创建仓库根目录下创建 `.github/workflows/HexoCI.yml` 文件，目录结构如下

```shell
├── .github
│   └── workflows
│       └── HexoCI.yml
```

编写`HexoCI.yml`文件

```shell
name: CI
on:
  push:
    branches:
      - hexo
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 切换分支
        uses: actions/checkout@v2

      - name: 安装依赖
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci

      - name: 清除Hexo
        uses: heowc/action-hexo@main
        with:
          args: clean

      - name: 生成Hexo
        uses: heowc/action-hexo@main
        with:
          args: generate

      - name: 部署到master分支
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.CI_TOKEN }}
          external_repository: inkdp/inkdp.github.io
          publish_branch: master
          publish_dir: ./public
```

### 模版参数说明

- *name* 为此 Action 的名字

- *on* 触发条件，当满足条件时会触发此任务，这里的 `on.push.branches.$.hexo`是指当 `hexo` 分支收到 `push` 后执行任务

- *jobs* 为此 Action 下的任务列表

  - *jobs.{job}.name* 任务名称
  - *jobs.{job}.runs-on* 任务所需容器，可选值：`ubuntu-latest`、`windows-latest`、`macos-latest`。

  * jobs.{job}.steps* 一个步骤数组，可以把所要干的事分步骤放到这里。
    * *jobs.{job}.steps.$.name* 步骤名，编译时会会以 LOG 形式输出。
    * *jobs.{job}.steps.$.uses* 所要调用的 Action，可以到 https://github.com/actions 查看更多。
    * *jobs.{job}.steps.$.with* 一个对象，调用 Action 传的参数，具体可以查看所使用 Action 的说明。
  * *env* 为环境变量对象，用于放置一些环节变量

### 第三方 Actions

使用第三方 Actions 语法 `{owner}/{repo}@{ref}` 或者 `{owner}/{repo}/{path}@{ref}` 例如：

```shell
steps:
  - name: 切换分支
    uses: actions/checkout@v2
```
```shell
 - name: 清除Hexo
   uses: heowc/action-hexo@main
   with:
     args: clean
```

我使用的大部分都是第三方的Actions，更多第三方Actions可查看[官方 Actions 市场](https://github.com/marketplace?type=actions&query=checkout)

可以根据自己的需求书写对应的配置文件，可以参考我的[HexoCI.yml](https://github.com/InkDP/inkdp.github.io/blob/hexo/.github/workflows/HexoCI.yml)

### 额外配置

因为上述配置文件使用了第三方**Actions:[peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)**，所以需要额外配置**个人访问令牌**，配置路径`你的头像 -> Settings -> Personal access tokens -> Personal access tokens -> Generate new token`，勾选` repo`即可，创建成功后复制生成的`token`，回到项目设置中，参考第二步中配置私钥的方式，将`token`填入`Value`，再配置一个`Name`即可，下例中的`CI_TOKEN`就是我配置的

```shell
 - name: 部署到master分支
    uses: peaceiris/actions-gh-pages@v3
    with:
      personal_token: ${{ secrets.CI_TOKEN }}
      external_repository: inkdp/inkdp.github.io
      publish_branch: master
      publish_dir: ./public
```

| 配置个人访问令牌 | 成功 |
| ---------------- | ---- |
|  ![image-20210118005047841](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210118005047841.png) | ![image-20210118005143713](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210118005143713.png) |

### 部署到个人服务器(可选)

通过配置私钥到`GitHub Secret`实现服务器的免密登录，再`scp`编译好的静态页面到你的服务器，参考如下配置，其中`ACTION_DEPLOY_KEY`为免密登录的私钥，`SERVER_DIR`为服务器地址及目录

```shell
- name: 部署到个人服务器
  env:
    ACTION_DEPLOY_KEY: ${{ secrets.ACTION_DEPLOY_KEY }}
    SERVER_DIR: ${{ secrets.SERVER_DIR }}
  run: |
    mkdir -p ~/.ssh/
    echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa
    mv ./public ./www
    scp -o StrictHostKeyChecking=no -r ./www $SERVER_DIR
```

### 上传配置文件

`push`到`yourname.github.io`的`hexo`分支，到此仓库的`Actions` 页面查看当前 task

![image-20210118014615951](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-20210118014615951.png)

当任务完成后查看您的博客 `https://yourname.github.io`，如果不出意外的话已经可以看到自动部署的文章了，如有意外欢迎留言

### 可能出现的问题

- 问题1：

  ![image-20210622202346107](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20210622202346.png)

  出现该问题是node版本过低导致的，在`yaml`文件中指定node版本为`12+`，或者设置

  ```yaml
  highlight:
    enable: false
  ```
