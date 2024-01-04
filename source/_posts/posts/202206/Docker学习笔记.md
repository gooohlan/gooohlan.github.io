---
title: Docker学习笔记
toc_number: true
tags:
  - 学习
  - Docker
categories:
  - - 技术
    - 后端
cover: 'https://gooohlan.fishpi.cn/img/20220530164429.png'
keywords: 'Docker, 容器'
description: >-
  Docker 是一个用于开发、发布和运行应用程序的开放平台。Docker 使您能够将应用程序与基础结构分离开来，从而可以快速交付软件。使用
  Docker，您可以像管理应用程序一样管理基础结构。通过利用 Docker
  的快速发布、测试和部署代码的方法，可以显著减少编写代码和在生产环境中运行代码之间的延迟。
abbrlink: 10048
date: 2022-06-01 01:59:28
updated: 2022-06-04 17:50:28
---


### 概述

Docker 是一个用于开发、发布和运行应用程序的开放平台。Docker 使您能够将应用程序与基础结构分离开来，从而可以快速交付软件。使用 Docker，您可以像管理应用程序一样管理基础结构。通过利用 Docker 的快速发布、测试和部署代码的方法，可以显著减少编写代码和在生产环境中运行代码之间的延迟。

#### docker是什么？

- Docker 是一个应用打包、分发、部署的工具
- 你也可以把它理解为一个轻量的虚拟机，它只虚拟你软件需要的运行环境，多余的一点都不要，
- 而普通虚拟机则是一个完整而庞大的系统，包含各种不管你要不要的软件。

官方解释：

Docker提供一个能力来打包和运行应用，在一个被称为容器的松散的孤立环境中。这种隔离和安全机制可以使你同时在一个主机上运行很多的容器。容器是轻量级，因为它们不需要额外的应用管理负载，直接在主机的内核当中运行。意味着可以在给定的硬件当中运行更多的容器（比虚拟机成本要低很多）。**甚至可以在虚拟机当中运行docker**！

提供了一个工具和平台来管理你的容器的生命周期:

- 使用容器开发应用及支持的组件
- 容器称为了一个单元用于发布和测试你的应用
- 准备发布的时候，作为一个容器或合理安排的服务，无论是产品环境还是云环境，都会是一致的

#### 跟普通虚拟机的对比

| 特性   | 普通虚拟机                                                   | Docker                                               |
| ------ | :----------------------------------------------------------- | ---------------------------------------------------- |
| 跨平台 | 通常只能在桌面级系统运行，例如 Windows/Mac，无法在不带图形界面的服务器上运行 | 支持的系统非常多，各类 windows 和 Linux 都支持       |
| 性能   | 性能损耗大，内存占用高，因为是把整个完整系统都虚拟出来了     | 性能好，只虚拟软件所需运行环境，最大化减少没用的配置 |
| 自动化 | 需要手动安装所有东西                                         | 一个命令就可以自动部署好所需环境                     |
| 兼容性 | 兼容性不高，不同系统差异大                                   | 兼容性好，不同系统都一样部署方式                     |

#### 可以用docker来做什么？

1. 快速，一致发布应用

   docker通过允许开发者在标准的环境使用本地容器提供应用和服务，来简单化开发生命周期。容器对于不断地集成（continuous integration）、发布（continuous delivery）（CI/CD）工作流是非常重要的！

2. 响应式部署

   docker基于容器的平台具有具有高度的可移植性的工作负载。docker 容器可以运行在开发者的本地笔记本、在数据中心的物理机或虚拟机当中、云环境或者混合环境等等。

   docker的可移植性和轻量级的特征可以使之很轻易地在真实环境当中，根据商业需要，动态地管理工作负载，按比例增加或减少应用服务 。

3. 运行更多的负载在相同的硬件中

   docker是一个轻量并且快速。相对于应用管理的虚拟机设备，提供了一个可见、成本低廉的选择，所以你可以更加充分地运行本的计算 能力以达到商业布标。docker更喜欢高密度环境用于小型、中型部署，这样可以用更少地资源做更多的事。

4. 快速安装测试/学习软件，用完就丢（类似小程序），不把时间浪费在安装软件上。例如 Redis / MongoDB / ElasticSearch / ELK

5. 多个版本软件共存，不污染系统，例如 Python2、Python3，Redis4.0，Redis5.0

####  重要概念：<font color="red">镜像、容器</font>、仓库

- 镜像

  一个镜像就是一个只读模板，指示创建容器 。大多数情况下，一个镜像会基于另一个镜像以及一些额外的自定义选项。比如，你可以构建一个基于ubuntu镜像，但是安装apache网站服务器和应用，能够像配置的一样运行起来。

  你可以创建自己的镜像或者可以仅仅使用那些已经被其它人创建发布的镜像。为了创建你自己的镜像，你需要创建一个dockerfile，使用相同的语法来定义和步骤来创建镜像并运行。

- 容器

  一个容器就是就是一个镜像的实例，你可以通过Docker API或命令行创建、启动、停止、移动或者删除。一个容器可以连接多个网络或者基于当前的状态创造更多的镜像。

  默认情况下，容器会与其它容器进行隔离，可以控制容器的网络、存储或其他底层系统。容器由镜像定义像其它配置选项一样

- 仓库

  仓库类似Git的远程仓库，集中存放镜像文件。

三者的关系：

![img](https://gooohlan.fishpi.cn/img/20220602082051.png)

#### Docker的好处

1. 模块化

   Docker 容器化方法非常注重在不停止整个应用的情况下，单独截取部分应用进行更新或修复的能力。除了这种基于微服务的方法，您还可以采用与[面向服务的架构（SOA）](https://baike.baidu.com/item/SOA/2140650)类似的使用方法，在多个应用间共享进程。

2. 层和镜像版本控制

   每个 Docker 镜像文件都包含多个层。这些层组合在一起，构成单个镜像。每当镜像发生改变时，就会创建一个新的镜像层。用户每次发出命令（例如 *run* 或 *copy*）时，都会创建一个新的镜像层。

   Docker 重复使用这些层来构建新容器，借此帮助加快流程构建。镜像之间会共享中间变化，从而进一步提升速度、[规模](http://developers.redhat.com/blog/2016/03/09/more-about-docker-images-size/)以及效率。版本控制是镜像层本身自带的能力。每次发生新的更改时，您大都会获得一个内置的更改日志，实现对容器镜像的全盘管控。

3. 回滚

   回滚也许是层最值得一提的功能。每个镜像都拥有多个层。举例而言，如果您不喜欢迭代后的镜像版本，完全可以通过回滚，返回之前的版本。这一功能还支持敏捷开发方法，帮助[持续实施集成和部署（CI/CD）](https://www.redhat.com/zh/topics/devops/what-is-ci-cd)，使其在工具层面成为一种现实。

4. 快速部署

   启动和运行新硬件、实施部署并投入使用，这在过去一般需要数天时间。投入的心力和成本往往也让人不堪重负。基于 Docker 的容器可将部署时间缩短到几秒。通过为每个进程构建容器，您可以快速将这些类似进程应用到新的应用程序中。而且，由于无需启动操作系统即可添加或移动容器，因此大幅缩短了部署时间。除此之外，得益于这种部署速度，您可以轻松无虞、经济高效地创建和销毁容器创建的数据。

   因此，Docker 技术是一种更加精细、可控、基于微服务的技术，可为企业提供更高的效率价值。

####  Docker的缺点

Docker 本身非常适合用于管理单个容器。但随着您开始使用越来越多的容器和容器化应用，并把它们划分成数百个部分，很可能会导致管理和编排变得非常困难。最终，您需要后退一步，对容器实施分组，以便跨所有容器提供网络、安全、遥测等服务。于是，k8s 应运而生。

### 使用Docker安装软件

#### 传统安装软件缺点

- 安装麻烦，可能有各种依赖，需要提前安装各种东西。比如：ElasticSearch，ElK
- 部分软件并不能在所有系统完美运行，各种兼容问题，甚至无法安装。例如在Windows上安装最新redis
- 软件多版本之间不能共存
- 不同系统安装方式不一样
- 安装一堆软件，占用系统内存，拖慢电脑运行速度

#### Docker安装的优点

- 一个命令就可以安装好，快速方便
- 有大量的镜像，可直接使用
- 没有系统兼容问题，Linux 专享软件也照样跑
- 支持软件多版本共存
- 用完就丢，不拖慢电脑速度
- 不同系统和硬件，只要安装好 Docker 其他都一样了，一个命令搞定所有

#### 例：使用Docker安装Redis

`docker run -d -p 6379:6379 --name redis redis`

轻松搞定

![image-20220531084506717](https://gooohlan.fishpi.cn/img/20220531084506.png)

### Docker常见命令

![Docker常用命令](https://gooohlan.fishpi.cn/img/20220602080651.svg)

#### 服务

- 查看docker版本信息

  ```shell
  docker version
  ```

- 查看docker简要信息

  ```shell
  docker -v
  ```

- 启动docker(linux)

  ```shell
  systemctl start docker
  ```

- 关闭docker(linux)

  ```shell
  systemctl stop docker
  ```

- 设置开机启动

  ```shell
  systemctl enable docker
  ```

- 重启docker服务

  ```shell
  service docker restart
  ```

- 关闭docker服务

  ```shell
  service docker stop
  ```

#### 镜像

镜像仓库中存储这大量官方镜像，可以从仓库直接获取镜像并运行，比如[Docker Hub](https://hub.docker.com/search?q=&type=)，拉取慢可以更换镜像源以提速，以腾讯加速源为例，编辑`/etc/docker/daemon.json`配置文件，添加一下内容即可

```sh
{
   "registry-mirrors": [
       "https://mirror.ccs.tencentyun.com"
  ]
}
```

##### 获取镜像

- 搜索镜像

  ```shell
  docker search 镜像名
  ```

- 获取镜像

  ```shell
  docker pull [OPTIONS] 镜像名称[:TAG|@DIGEST]
  ```

  OPTIONS:

  - `--all-tags,-a`  下载存储库中所有标记的镜像
  - `--disable-content-trust`  跳过镜像验证，默认为`true`
  - `--platform` 如果服务器支持多平台，则设置平台
  - `--quiet,-q` 取消详情输出

  | `--all-tags`,`-a`         |        | 下载存储库中的所有标记图像       |
  | ------------------------- | ------ | -------------------------------- |
  | `--disable-content-trust` | `true` | 跳过图像验证                     |
  | `--platform`              |        | 如果服务器支持多平台，则设置平台 |
  | `--quiet`,`-q`            |        | 抑制详细输出                     |

##### 镜像管理

- 列出镜像

  ```shell
  docker images
  docker image ls

- 删除镜像

  ```shell
  docker rmi <镜像Id|镜像名称> [<镜像Id|镜像名称>,...]
  ```

- 导出镜像

  ```sh
  docekr save
  ```

- 导入镜像

  ```shell
  docker load
  ```

##### Dockerfile构建镜像

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

**Dockerfile常见指令：**

- FORM：指定基础镜像
- RUN：执行命令
- WORKDIR：指定工作目录
- COPY：复制文件
- ADD：更高级的复制文件
- CMD：容器启动命令
- ENV：设置环境变量
- EXPOST：声明端口
- VOLUME：挂载目录

其他指令还有：ENTRYPOINT、ARG、USER、HEALTHCHECK、LABEL…，详见[Environment replacement](https://docs.docker.com/engine/reference/builder/#environment-replacement)

Dockerfiles示例：

```dockerfile
FROM golang:1.16.7

ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

WORKDIR /build
COPY . .
RUN go build -ldflags "-s -w"

EXPOSE 8080

FROM alpine:latest

WORKDIR /build
COPY --from=0 /build ./

CMD ./demo
```

**镜像构建**

```shell
docker build [OPTIONS] PATH | URL | -
```

OPTIONS：

- **--build-arg=[] :**设置镜像创建时的变量；
- **--cpu-shares :**设置 cpu 使用权重；
- **--cpu-period :**限制 CPU CFS周期；
- **--cpu-quota :**限制 CPU CFS配额；
- **--cpuset-cpus :**指定使用的CPU id；
- **--cpuset-mems :**指定使用的内存 id；
- **--disable-content-trust :**忽略校验，默认开启；
- **-f :**指定要使用的Dockerfile路径；
- **--force-rm :**设置镜像过程中删除中间容器；
- **--isolation :**使用容器隔离技术；
- **--label=[] :**设置镜像使用的元数据；
- **-m :**设置内存最大值；
- **--memory-swap :**设置Swap的最大值为内存+swap，"-1"表示不限swap；
- **--no-cache :**创建镜像的过程不使用缓存；
- **--pull :**尝试去更新镜像的新版本；
- **--quiet, -q :**安静模式，成功后只输出镜像 ID；
- **--rm :**设置镜像成功后删除中间容器；
- **--shm-size :**设置/dev/shm的大小，默认值是64M；
- **--ulimit :**Ulimit配置。
- **--squash :**将 Dockerfile 中所有的操作压缩为一层。
- **--tag, -t:** 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
- **--network:** 默认 default。在构建期间设置RUN指令的网络模式

**镜像运行**

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

#### 容器

##### 容器操作

- 启动容器：启动容器有两种方式：一是基于镜像新建容器，二是启动终止状态的容器

  ```shell
  # 新建并启动
  docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
  # 启动已终止容器
  docker start <容器ID|容器名>
  ```

  `docker run`命令详见：[Docker run reference](https://docs.docker.com/engine/reference/run/#general-form)

- 查看容器

  ```shell
  # 列出所有运行中的容器
  docker ps
  # 列出所有容器(运行中和已停止)
  docker ps -a
  ```

- 停止容器

  ```shel
  docker stop <容器ID|容器名称>
  ```

- 重启容器

  ```shell
  docker restart <容器ID|容器名称>
  ```

- 删除容器

  删除的容器必须是已停止
  ```shell
  docker rm <容器ID|容器名称>
  ```

##### 进入容器

```shell
docker exec [OPTIONS] <容器ID|容器名> 命令 [ARG...]
```

使用`docker exec`命令需要容器在运行中，OPTIONS参数详见：[OPTIONS](https://docs.docker.com/engine/reference/commandline/exec/#options)

##### 导入和导出

- 导出容器

  ```shell
  docker export <容器ID|容器名称>
  ```

- 导入容器

  ```shell
  docker import <路径>
  ```

##### 其他

- 查看日志:

  ```shell
  docker logs [OPTIONS] <容器ID|容器名>
  ```

  OPTIONS参数列表:

  - **--details :** 显示日志详情
  - **--follow, –f :** 日志滚动输出
  - **--since : **显示某个开始时间的所有日志
  - **--tail, -t :** 从日志末尾显示的行数
  - **--timestamps, -t : **显示时间戳
  - **--until : **显示某个时间内的所有日志

- 复制文件

  ```shell
  # 从主机复制到容器
  docker cp 主机路径 <容器ID|容器名称>:容器类内路径 
  # 从容器复制到主机
  docker cp <容器ID|容器名称>:容器类内路径 主机路径
  ```

### Docker Compose

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

docker compose使用的3个步骤：

- 使用`Dockerfile`定义应用程序环境
- 使用`docker-compose.yaml`定义构成应用程序的服务，这样它们在隔离环境中一起运行
- 最后，执行`docker-compose up`命令来启动并运行整个应用程序

`docker-compose.yaml`示例配置

```yaml
version: '3.3'
services:
  web:
    build: .
    links:
      - redis
    container_name: demo-web
    ports:
      - "8080:8080"
  redis:
    image: redis:6.2.5
    volumes:
      - ~/dockerData/redis/conf/redis.conf:/etc/redis/redis.conf
      - ~/dockerData/redis/data:/data
    ports:
      - "6379:6379"
    container_name: demo-web-redis
```

运行`docker-compose up`即可启动应用程序，如果需要后台运行可以使用`docker-compose up -d`

#### 配置指令参考：

- version: 指定本`yaml`依从的`compose`哪个版本制定的
- services：运行的服务列表
- build：指定为构建镜像上下文路径，或者作为具有在上下文指定的路径的对象，以及可选的 Dockerfile 和 args：
  - context：上下文路径。
  - dockerfile：指定构建镜像的 Dockerfile 文件名。
  - args：添加构建参数，这是只能在构建过程中访问的环境变量。
  - labels：设置构建镜像的标签。
  - target：多层构建，可以指定构建哪一层。
- volumes：文件挂载文件、目录映射
- image：所使用的镜像名
- links：对应容器ip映射到容器内，上文所示中的应用程序就可以使用`redis:6379`访问redis
- container_name：容器名称
- ports：端口映射,把docker内部的端口暴露出来
- restart：重启操作[no|always|on-failure|unless-stoppen]
- env_file：环境变量文件
- environment：环境变量
- external_links：链接其他的容器服务
- depends_on：由于启动顺序是随机的，如果有依赖关系时添加
- ….

[docker-compose 参数 官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.docker.com%2Fcompose%2Fcompose-file%2F%23environment)

#### docker compose命令

| 命令                                                         | 详情                                        |
| ------------------------------------------------------------ | ------------------------------------------- |
| [docker compose build](https://docs.docker.com/engine/reference/commandline/compose_build/) | 构建或重建服务                              |
| [docker compose convert](https://docs.docker.com/engine/reference/commandline/compose_convert/) | 格式化配置文件                              |
| [docker compose cp](https://docs.docker.com/engine/reference/commandline/compose_cp/) | 在服务容器和本地文件系统之间复制文件/文件夹 |
| [docker compose create](https://docs.docker.com/engine/reference/commandline/compose_create/) | 为服务创建容器                              |
| [docker compose down](https://docs.docker.com/engine/reference/commandline/compose_down/) | 停止并移除容器、网络                        |
| [docker compose events](https://docs.docker.com/engine/reference/commandline/compose_events/) | 从容器接收实时事件                          |
| [docker compose exec](https://docs.docker.com/engine/reference/commandline/compose_exec/) | 在正在运行的容器中执行命令                  |
| [docker compose images](https://docs.docker.com/engine/reference/commandline/compose_images/) | 列出所创建的容器使用的镜像                  |
| [docker compose kill](https://docs.docker.com/engine/reference/commandline/compose_kill/) | 强制停止容器                                |
| [docker compose logs](https://docs.docker.com/engine/reference/commandline/compose_logs/) | 插入容器日志                                |
| [docker compose ls](https://docs.docker.com/engine/reference/commandline/compose_ls/) | 列出正在运行的compose项目                   |
| [docker compose pause](https://docs.docker.com/engine/reference/commandline/compose_pause/) | 暂停服务                                    |
| [docker compose port](https://docs.docker.com/engine/reference/commandline/compose_port/) | 打印端口绑定的公共端口                      |
| [docker compose ps](https://docs.docker.com/engine/reference/commandline/compose_ps/) | 容器列表                                    |
| [docker compose pull](https://docs.docker.com/engine/reference/commandline/compose_pull/) | 拉取服务镜像                                |
| [docker compose push](https://docs.docker.com/engine/reference/commandline/compose_push/) | 推送服务镜像                                |
| [docker compose restart](https://docs.docker.com/engine/reference/commandline/compose_restart/) | 重新启动容器                                |
| [docker compose rm](https://docs.docker.com/engine/reference/commandline/compose_rm/) | 移除已停止的服务容器                        |
| [docker compose run](https://docs.docker.com/engine/reference/commandline/compose_run/) | 对服务运行一次性命令                        |
| [docker compose start](https://docs.docker.com/engine/reference/commandline/compose_start/) | 启动服务                                    |
| [docker compose stop](https://docs.docker.com/engine/reference/commandline/compose_stop/) | 停止服务                                    |
| [docker compose top](https://docs.docker.com/engine/reference/commandline/compose_top/) | 显示正在运行的服务进程                      |
| [docker compose unpause](https://docs.docker.com/engine/reference/commandline/compose_unpause/) | 停止服务                                    |
| [docker compose up](https://docs.docker.com/engine/reference/commandline/compose_up/) | 创建和启动容器                              |
| [docker compose version](https://docs.docker.com/engine/reference/commandline/compose_version/) | 显示 Docker Compose 版本信息                |

### 进阶

- kubernetes学习笔记：学习中
- ……