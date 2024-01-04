---
title: Navicat Premium for Mac 破解教程
date: '2019-10-24 12:14:48'
updated: '2019-10-24 12:24:50'
tags:
  - 破解
  - MySQL
cover: 'https://gooohlan.fishpi.cn/img/20180424.jpg'
permalink: /articles/2019/10/24/1571890488789.html
abbrlink: 38535
---


## 前言
本教程破解的版本为Navicat Premium 12.1.27，理论上支持12.0.24~最新版，所以在你开始破解时请确认你的版本

## 下载并安装

进入[Navicat Premium](https://www.navicat.com.cn/download/navicat-premium)，选中对应软件进行下载，安装就不多说了，整安装就可以了

## 编译

### 1. 安装依赖  

首先你的确认你安装了brew，没有的话就先去装一个吧，然后安装下列库  
```  
brew install openssl  
brew install capstone  
brew install keystone  
brew install rapidjson  
brew install libplist  
```
### 2. 克隆项目

克隆Mac分支，并编译keygen和patcher
```
git clone -b mac --single-branch https://github.com/DoubleLabyrinth/navicat-keygen.git
cd navicat-keygen
make all
```
![image.png](https://gooohlan.fishpi.cn/img/image-605b5c43.png)

编译成功后当前目录下的bin文件下回出现两个可执行文件
```
ls bin/
```
![image.png](https://gooohlan.fishpi.cn/img/image-6e4f4f06.png)

### 3. 备份

* 备份好`Navicat Premium.app/Contents/MacOS/Navicat Premium `，防止翻车(不怕翻车可跳过)
* 备份好Navicat中所有已保存的数据库连接(包括密码)，我没备份(可跳过）
* 移除所有Navicat在钥匙链中保持的密码，可通过搜索`navacat`来找到他们
   ![image.png](https://gooohlan.fishpi.cn/img/image-b4f15694.png)

### 4. 使用navicat-patcher替换公钥：
```
   Usage:
    navicat-patcher <Navicat installation path> [RSA-2048 Private Key File]

        <Navicat installation path>    Path to `Navicat Premium.app`.
                                       Example:
                                           /Applications/Navicat\ Premium.app/
                                       This parameter must be specified.

        [RSA-2048 Private Key File]    Path to a PEM-format RSA-2048 private key file.
                                       This parameter is optional.
```
* `Navicat installation path` ：`Navicat Premium.app`的路径，必填
* `RSA-2048 Private Key File`：PEM格式的RSA-2048的私钥路径，可选，不填会在当前目录下生成一个新的RSA-2047密钥文件`RegPrivateKey.pem`

默认如下：  
```  
./navicat-patcher /Applications/Navicat\ Premium.app/
```
![image.png](https://gooohlan.fishpi.cn/img/image-27449b40.png)
![image.png](https://gooohlan.fishpi.cn/img/image-5c6f876b.png)
### 5. 生成一份自动签名 的代码签名证书
   * 打开钥匙串访问
   * 选择创建证书
   * 输入名称"Navicat"，身份类型，证书类型
   * 点击创建

   ![image.png](https://gooohlan.fishpi.cn/img/image-99b4f449.png)
   ![image.png](https://gooohlan.fishpi.cn/img/image-8b2aa128.png)
   ![image.png](https://gooohlan.fishpi.cn/img/image-ea68d143.png)

### 6. 签名
```
codesign -f -s "Navicat" /Applications/Navicat\ Premium.app/
```
![image.png](https://gooohlan.fishpi.cn/img/image-c224bb16.png)
## 激活
### 1. 使用`navicat-keygen`生成序列号和激活码  
```  
Usage:  
 navicat-keygen <RSA-2048 Private Key File>  

 <RSA-2048 Private Key File>    Path to a PEM-format RSA-2048 private key file.  
 This parameter must be specified.  
```  
* `RSA-2048 Private Key File`，PEM格式的RSA-2048密钥文件路径，既上文中提到的`RegPrivateKey.pem`  

默认如下：  
```  
./navicat-keygen ./RegPrivateKey.pem  
```  
输入语言以及主版本号后会得到一个序列号  
![image.png](https://gooohlan.fishpi.cn/img/image-acf9356f.png)  
使用这个序列号来激活Navicat  
接下来会要求你输入用户名以及组织名，随意填写即可  
之后你会被要求填入请求码  
**请不要关闭注册机！**  
**请不要关闭注册机！**  
**请不要关闭注册机！**
### 2. 断网并启动Navicat premium完成激活
* 启动时点击注册
* 在注册页面输入注册机给你的序列号，点击激活

![image.png](https://gooohlan.fishpi.cn/img/image-0aa226c9.png)

* 一般都会激活失败，这时点击手动激活即可

![image.png](https://gooohlan.fishpi.cn/img/image-81692bde.png)

* 手动激活的窗口会给到你一个请求码，复制并粘贴到注册机里面，**两次回车结束输入**

![image.png](https://gooohlan.fishpi.cn/img/image-eaa0ba85.png)

* 不出意外的话，你会得到一个激活码，复制它并粘贴到navicat的手动激活窗口
![image.png](https://gooohlan.fishpi.cn/img/image-0b68abdb.png)   
![image.png](https://gooohlan.fishpi.cn/img/image-e8dfb932.png)
* 最后点击激活，没出问题的话就激活成功了
![image.png](https://gooohlan.fishpi.cn/img/image-d3b2e298.png)
![image.png](https://gooohlan.fishpi.cn/img/image-50a37e9d.png)

## 参考
[Navicat Keygen](https://github.com/DoubleLabyrinth/navicat-keygen/blob/mac/README.zh-CN.md)
## 后记
由于删除了钥匙串的密码，所以原有连接里的密码可能需要重新输入
