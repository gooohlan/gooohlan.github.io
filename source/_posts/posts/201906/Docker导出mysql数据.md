title: Docker导出mysql数据
date: '2019-06-11 23:43:54'
updated: '2019-09-25 14:03:21'
tags: [MySQL, docker, Solo, shell]
permalink: /articles/2019/06/11/1560267833958.html
---
# 前言
&emsp;&emsp;前几天无意中在社区看到一个帖子（[记一次清空数据仓库的过程](https://hacpai.com/article/1553941852324)），讲的是自己无意中删库的经历。如文中所讲，大多时候删库这件事我们只是耳闻，并没有遇到过，可要是万一呢，到时候恐怕是追悔莫及，而且mysql也没有oracle的恢复机制，所以备份就成了一个非常有必要的操作。

&emsp;&emsp;由于没有相关操作经验，所以从零开始讲如何数据，毕竟我还是比较珍惜我的小博客的。

# 具体操作
&emsp;&emsp;以前也导出过sql文件，但是都是直接用Navicat导出就完事了，但是这次我想实现的是自动备份，最好写成脚本的方式。

&emsp;&emsp;基本思路：使用命令将数据库数据从docker容器中导出来，以时间戳命名。最多保持7天，过期文件自动删除。
### 导出mysql数据

#### 间接导出
&emsp;&emsp;mysql 导出数据的命令还是蛮简单的：`mysqldump -u 用户名 -p 数据库名 > 导出的文件名`，但这是linux里面执行的，我们的放在docker里面，所以要先进入容器，然后执行上述命令。然后你就会惊讶的发现，导出的文件在你的容器里面，然后你再从容器里面copy到你的主机上。这样做会在容器上产生大量sql文件，写定时任务是需要及时清理。

#### 直接导出

&emsp;&emsp;上述方法是可行的，但是过于麻烦，有没有一步到位的呢？很显然是有的，命令是这个样子的：`docker exec -it [docker容器名称/ID] mysqldump -u[数据库用户名] -p[数据库密码] [数据库名称] > [导出表格路径]`，比如我的`docker exec -it mysql mysqldump -uroot -p123456 solo > /var/www/solo.sql`。没有报错的话，导出的数据库文件就会到你指定的目录下了。
### 写成脚本方式运行
博主是不会写Shell脚本的，所以现学了下，参考[Shell脚本编程30分钟入门](https://github.com/qinjx/30min_guides/blob/master/shell.md)

> 间接导出(间接导出时请先创建相关文件目录)

```bash
#!/bin/sh
# 进入容器
docker exec -i mysql bash<<'EOF'
# solo 为数据库的名称
 XXX为数据库密码
mysqldump -uroot -pXXX solo > /mysqlData/$(date +%Y%m%d).sql
#删除超过1天的数据
find /mysqlData/ -mtime +1 -type f | xargs rm -rf
exit
EOF
# 将docker中的备份的数据拷贝到宿主机上。
docker cp mysql:/mysqlData/$(date +%Y%m%d).sql /var/www/html/solo/sqlData/
#删除超过7天的数据
find /var/www/html/solo/sqlData -mtime +7 -type f | xargs rm -rf

```

> 直接导出(导出时`exec`不需指定参数`-it`)

```bash
#!/bin/sh
# 进入你要保持数据的文件
cd /var/www/html/solo/sqlData
# 导出今日的sql
docker exec mysql mysqldump -uroot -p123123 solo >`date +%Y%m%d%H%M%S`.sql
# 删除7天前的sql(+号后面跟天数,N天前,find后指定目录)
find . -mtime +7 -type f | xargs rm -rf
``` 
### 后记
&emsp;&emsp;导出的sql文件中会出现一些版本备注信息等，这样的sql文件导入时有时会出现问题，会报错，导入之前需要将sql文件中的无用信息删除。
```
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
```
这样的内容在sql中还不是一两句，到处都是。这里提供一段正则，可快速匹配到sql文件中所有`/*!40101XXXXX*/;`，通过正则`\/\*\![0-9]+\s[\s\S]+?\s\*\/;\n`实现快速替换。
替换完后的效果如下所示，多舒服。
![image.png](https://img.hacpai.com/file/2019/06/image-3766354f.png)
