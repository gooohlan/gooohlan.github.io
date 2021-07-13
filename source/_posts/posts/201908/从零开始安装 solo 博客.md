---
title: 从零开始安装 solo 博客
date: '2019-08-06 00:18:51'
updated: '2020-04-17 11:32:50'
tags:
  - Solo
  - Docker
categories:
  - - 技术
    - 后端
permalink: /articles/2019/08/06/1565021931775.html
cover: 'https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/20180919.jpg'
abbrlink: 19060
toc_number: false
---

之前也写过类似的帖子，但是由于那时自己的各种原因写的不是特别好，所以今天写一个聚合帖，记录从购买服务器到安装solo然后通过nginx反向代理，最后升级https的全过程。此贴献给完全无基础的人，所以废话较多，见谅

# 1. 购买服务器

首先你得有自己的服务器，有的话就跳过。服务商可选的有很多，比如：[阿里云](https://cn.aliyun.com/)、[腾讯云](https://cloud.tencent.com/)、[百度云](https://cloud.baidu.com/)、还有一些香港的服务商以及国外的（有特殊需求的可以考虑下）。腾讯和阿里对于新用户以及学生都有很大的优惠，配置的话如果只挂solo，买最低配1G1核1M即可。国内的几家都可以关注下，不定时会有很好的优惠活动。我比较推荐[阿里云](https://cn.aliyun.com/)，前段时间刚买了一台3年才668多，不知道活动结束没有。阿里云购买服务器时会要求你安装系统（不知道可不可以不选），推荐选择centos。

# 2. 购买域名(可不买)

建议还是买一个域名，直接通过IP访问的话不是特别好。购买域名时不要盯着 `.com`，`.cn`这种比较热门的域名，往往很贵。还有尽量选择可备案的域名，否则就会像[鼠鼠在碎觉](https://sszsj.cc:444/)一样只能挂载444端口上运行。可通过[域名.信息](http://xn--eqrt2g.xn--vuq861b/)查看可备案域名。如果你服务器买着国外的话似乎就不用备案。

# 3. 域名解析与备案

服务器和域名购买完后需要将域名解析到服务器，有些服务商可能不支持跨服务商解析，腾讯云域名可以解析阿里云服务器。解析过程大概需要10分钟。解析完成后如下图所示：
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-5ff5a25e.png)
如果不备案的话，80端口与443端口大概率会被封。所以备案还是需要的，备案的过程有点麻烦需要耐心。大致步骤如下：
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-4527c63e.png)
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-056ca9d6.png)
接入备案很快，几天就完事了。

# 4. 为服务器安装系统

给服务器安装系统不用像你给自己装系统那样麻烦，很方便，由于我使用的centos，所以推荐安装centos，以免出现不必要的错误。后续操作嘘设置服务器安全组，开发如下端口：`80`、`8080`、`3306`、`443`、`22`。具体左右我会在下面一一说明，`22用来远程连接服务器使用`，windows用户可下载xshell远程链接服务器。

# 5. 安装docker

直接使用yum安装，简单快捷

```
#安装 Docker
yum -y install docker

#启动 Docker 后台服务
service docker start

#测试运行 hello-world
docker run hello-world
```

出现hello world 就证明安装正常了
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-cc55a2ad.png)

# 6.安装Mysql

版本随意，我这里选择的5.6，你可以选择更高版本的，这个没关系，不影响使用

```
# 安装mysql:5.6,直接docker run 他会自动去官方镜想下载
# MYSQL_ROOT_PASSWORD=你的数据库密码
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6

# docker安装的mysql默认允许远程连接，可以使用Navicat等软件连接数据库
# 进入容器mysql
docker exec -it mysql bash

# 进入数据库 p后面跟你的密码
mysql -uroot -pXXX

# 创建数据库(数据库名:solo;字符集utf8mb4;排序规则utf8mb4_general_ci)
create database solo DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
# 出现Query OK, 1 row affected (0.00 sec)表示成功
#退出数据库
exit
#退出容器
exit
```

# 7. 安装solo

直接运行以下命令

```
docker run --detach --name solo --network=host \
--env RUNTIME_DB="MYSQL" \
--env JDBC_USERNAME="root" \
--env JDBC_PASSWORD="123456" \
--env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
--env JDBC_URL="jdbc:mysql://127.0.0.1:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
--rm \
b3log/solo --listen_port=8080 --server_scheme=http --server_host=www.jinjianh.com
```

上面的命令建议手敲，免得出错，参数说明

* `--env JDBC_PASSWORD="123456"` 将123456换成你的密码
* `--listen_port=8080` 监听的端口
* `--server_scheme=http` 请求方式，暂时使用http，后面我们会换成https
* `--server_host=www.jinjianh.com` 你的域名，如果你没有域名可以写ip地址
* `--rm`因为这个容器后面要删掉，带上rm会省很多事。

命令成功执行没有报错的话，通过 `docker ps`查看执行的容器列表中是否存在solo，存在这表示启动成功，直接访问你的域名加:8080即可访问你的博客，[金戋博客--http://www.jinjianh.com:8080](http://www.jinjianh.com:8080/)
如果你尚在备案中，你可以收藏本帖，后面等备案通过了在研究后面的部分。
如果你不想使用nginx也不想升级https，那么你可以先执行 `docker stop solo`，然后将上面 `--listen_port=8080`的 `8080`换成 `80`，然后去掉 `--rm`，再执行一次就ok。

# 8. 安装nginx

安装nginx前，我们现在本地建立几个文件，用于存放nginx的配置文件等

```
# 切换到服务器根目录
cd /
# 创建主目录
mkdir dockerData
# 创建文件
mkdir dockerData/nginx dockerData/nginx/conf dockerData/nginx/logs dockerData/nginx/www dockerData/nginx/ssl
```

上面的 `dockerData`可以换成自己喜欢的名字

* `dockerData/nginx` 用于存放docker下nginx自定义文件
* ` dockerData/nginx/conf` 存放nginx配置文件
* `dockerData/nginx/log` 存放nginx日志文件
* `dockerData/nginx/www` 存放nginx访问的资源文件
* `dockerData/nginx/ssl` 存放ssl证书
  启动nginx

```
docker run --name nginx -p 80:80 -d --rm nginx
```

如果你没有备案，可以将上面的 `80:80`换成 `8081:80`，因为这个东西一会儿也要删掉，所以加上 `--rm`参数，命令执行玩后通过 `docker ps`查看nginx是否在运行，在运行的情况下访问你的域名加端口号查看是否正常安装，`80`直接省略。如下表示访问成功
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-ec38061d.png)
导出配置文件

* `docker cp nginx:/etc/nginx/nginx.conf /dockerData/nginx/conf/nginx.conf` 导出配置文件nginx.conf
* `docker cp nginx:/etc/nginx/conf.d /dockerData/nginx/conf/conf.d` 导出配置为你nginx.conf
  执行 `docker stop nginx`，会自动删除现在的nginx容器，然后执行如下命令重新启动一个nginx容器

```
docker run -d -p 80:80 --name nginx \
-v /dockerData/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /dockerData/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /dockerData/nginx/www:/usr/share/nginx/html \
-v /dockerData/nginx/logs:/var/log/nginx nginx
```

* `-v /dockerData/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \` 挂载配置文件 `nginx.conf`
* `-v /dockerData/nginx/conf/conf.d:/etc/nginx/conf.d` 挂载配置文件 `default.conf`
* `-v /dockerData/nginx/www:/usr/share/nginx/html ` 挂载项目文件
* `-v /dockerData/nginx/logs:/var/log/nginx` 挂载配置文件

访问你的域名，你会发现报错了

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-9c8de340.png)
这时我们可以前往 `/dockerData/nginx/logs`下查看日志文件

`2019/08/05 14:57:54 [error] 6#6: *3 directory index of "/usr/share/nginx/html/" is forbidden, client: 121.32.33.217, server: localhost, request: "GET / HTTP/1.1", host: "www.jinjianh.com" `

因为 `/usr/share/nginx/html/`被挂载到了服务器上面的 `/dockerData/nginx/www`目录下，原来的欢迎页面在 `dockerData/nginx/www`是没有的，所有就报错了，这里我们随便建一个。

```
# 打开项目文件
cd /dockerData/nginx/www
# 使用vim 创建并编辑文件
vim index.html
# 此时我们会进入vim界面，按 i 插入，然后输入
<h1>Hello Docker-Nginx</h1>
# 输入完后，按 esc，然后输入 :wq
```

再次访问我们的域名就可以看到我们刚刚写的 `h1`标签内容

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-70a4130f.png)

# 9. 申请ssl证书，将http升级为https(可跳过)

`https`想比与 `http`来说，最核心的内容就是多了一个ssl证书，证书是可以免费申请的。

## 腾讯云

访问 [SSL证书选购](https://buy.cloud.tencent.com/ssl?fromSource=ssl) 申请

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-05285d99.png)

私钥可不填写
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-98544bc3.png)

选择手动DNS验证

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-44754a07.png)

可直接前往 [SSL 证书 域名验证指引 - 操作指南 - 文档中心 - 腾讯云](https://cloud.tencent.com/document/product/400/4142#ManualVerification)查看

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-3e6695e5.png)
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-8828eaa2.png)

前往[证书管理-控制台](https://console.cloud.tencent.com/ssl)等待验证通过即可

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-4c7bcc44.png)

将 `nginx`下文件上传到 `/dockerData/nginx/ssl`目录下即可

## 阿里云

访问[云盾证书服务](https://common-buy.aliyun.com/?spm=5176.10695662.958455.3.3f9140d55mPzFH&commodityCode=cas#/buy)申请，访问后如果有无内容可复制 https://common-buy.aliyun.com/?spm=5176.10695662.958455.3.3f9140d55mPzFH&commodityCode=cas#/buy 打开
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-2f8ed8b3.png)

走一下支付流程，然后申请

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-af887011.png)
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-a3de4d98.png)
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-8ef94cdf.png)
操作流程基本与腾讯云一样，我就不详细说明了

## 其他平台

免费的证书平台有很多，也可以去别的地方申请

## 上传证书

一下示例为腾讯云证书，阿里云
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-f9c0bb7e.png)

验证完后，我们下载证书，解压后得到

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-5b610f8d.png)

最后把 `Nginx`下的两个文件上传至服务器 `/dockerDat/nginx/ssl`目录下，别的服务商申请的证书也一样，将最后的ssl证书放到 `/dockerDat/nginx/ssl`下即可

## 配置nginx配置文件

```
cd /dockerData/nginx/conf/conf.d
vim default.conf

# 参考我的配置，配置自己的default.conf文件
server {
    listen       443;
    server_name  localhost;
    ssl on;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    ssl_certificate /ssl/1_www.jinjianh.com_bundle.crt;  # ssl 证书目录
    ssl_certificate_key /ssl/2_www.jinjianh.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
# ......
}
server{
  listen 80;
  server_name localhost;
  rewrite ^(.*) https://$host$1 permanent;
}
# 按esc，然后输入:wq保持退出
```

不重要的部分我省略了，可根据自己服务器配置做出相应调整
由于我们现在用的nginx容器并未监听443端口，所以需要删除现在的容器，重新启动一个新的nginx容器

```
docker stop nginx  # 停止容器
docker rm nginx # 删除容器
# 启动新的
docker run -d -p 80:80 -p 443:443 --name nginx \
-v /dockerData/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /dockerData/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /dockerData/nginx/ssl:/ssl/ \
-v /dockerData/nginx/www:/usr/share/nginx/html \
-v /dockerData/nginx/logs:/var/log/nginx nginx
```

* `-p 443:443` 监听443端口
* `-v /dockerData/nginx/ssl:/ssl/` 挂载ssl证书目录

访问查看，一切正常
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-d2e1477f.png)

# 10. 将solo通过nginx方向代理实现https访问

让solo还是跑在8080端口上，通过nginx代理到443端口即可，由于我们上面启动solo时添加了 `--rm`参数，只需要 `docker stop solo`即可自动删除solo容器，然后我们重新启动一个solo容器

```
docker run --detach --name solo --network=host \
--env RUNTIME_DB="MYSQL" \
--env JDBC_USERNAME="root" \
--env JDBC_PASSWORD="123123" \
--env JDBC_DRIVER="com.mysql.cj.jdbc.Driver" \
--env JDBC_URL="jdbc:mysql://127.0.0.1:3306/solo?useUnicode=yes&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC" \
b3log/solo --listen_port=8080 --server_scheme=https --server_host=www.jinjianh.com --server_port=
```

* `--server_scheme=http`换成 `--server_scheme=https`即可
* `--server_port`：最终访问端口，使用浏览器默认的 80 或者 443 的话值留空即可

然后我们去配置nginx配置文件，实现nginx反向代理

```
cd /dockerData/nginx/conf/conf.d
vim default.conf
 location / {
        proxy_pass http://backend$request_uri;
        proxy_set_header  Host $http_host;
        proxy_set_header  X-Real-IP $remote_addr;
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
    }
# 替换上面部分即可
# 按esc，然后输入:wq保持退出
```
## 注意！！！Nginx反代理上面的方式可能出现问题参考[Nginx反代](https://hacpai.com/article/1492881378588#NGINX-%E5%8F%8D%E4%BB%A3)

## 注意！！！Nginx反代理上面的方式可能出现问题参考[Nginx反代](https://hacpai.com/article/1492881378588#NGINX-%E5%8F%8D%E4%BB%A3)

## 注意！！！Nginx反代理上面的方式可能出现问题参考[Nginx反代](https://hacpai.com/article/1492881378588#NGINX-%E5%8F%8D%E4%BB%A3)

重启nginx，`docker restart nginx`

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-2c7eb017.png)

完美

# 11. 后记

## 皮肤挂载

* 在你的服务器上创建一个目录用于存放皮肤，比如我的 `/dockerData/solo/skins/`
* 然后将你要挂载的皮肤放到上面那个目录下
* 最后删除当前容器 重新启动一个容器，添加参数 `--v /dockerData/solo/skins/:/opt/solo/skins/`，这个添加时要注意位置，要添加到 `b3log/solo --listen...` 的上面一排
* 使用挂载皮肤时，默认会使用 `Pingsu`,

## 皮肤推荐

我开源了两款皮肤 [solo-nexmoe](https://hacpai.com/article/1566468138289)，因为我很懒的原因，solo-star没有手机端，所以你可以多挂载一款皮肤们比如官方皮肤 `Pinghsu`，如果你没有这款皮肤就会报错，没有请前往[solo-skins](https://github.com/b3log/solo-skins)下载

## 数据库占用内存过大优化

由于我们购买的服务器是内存只有1G，然后docker安装的mysql虽然很快，但是实际上占用内存非常大，之前服务器在腾讯云的时候就经常挂掉，排查了很久才发现是docker下mysql的问题，迁移到阿里云后倒是没出先挂掉的问题，但是服务器内存占用也一直在90%以上，所以我们对mysql容器进行一些优化。
由于容器内不能vim，所以我们将mysql的配置文件复制到服务器上改了之后再复制回去，也可以将配置文件挂载到服务器上，过程我不多讲，只讲核心部分。

这里注意，如果你要删除容器重新挂载的话，请提前备份mysql数据，不然你就属于删库了
这里注意，如果你要删除容器重新挂载的话，请提前备份mysql数据，不然你就属于删库了
这里注意，如果你要删除容器重新挂载的话，请提前备份mysql数据，不然你就属于删库了
重要的话说三遍

在配置文件 `/etc/mysql/mysql.conf.d/mysqld.cnf`中添加

```
performance_schema_max_table_instances=400
table_definition_cache=400
table_open_cache=256
```

```
# 从容器中复制到服务器
docker cp mysql:/etc/mysql/mysql.conf.d/mysqld.cnf /dockerData/mysql
# 从服务器复制到容器
docker cp /dockerData/mysql/mysqld.cnf mysql:/etc/mysql/mysql.conf.d/mysqld.cnf
```

改完之后记得重启mysql，`docker restart mysql`

## 启用lute

**此内容适用于solo3.6.5+**

- 启动Lute参考[Lute HTTP 使用指南](https://hacpai.com/article/1569240189601)
- 在solo启动参数末尾追加 `--lute_http=http://127.0.0.1:8249`/`--lute_http=http://localhost:8249`/`--lute_http=`
  solo成功启动后在终端输入 `docker logs solo`,日志显示有 `luteAvailable=true`即表示启用lute成功
