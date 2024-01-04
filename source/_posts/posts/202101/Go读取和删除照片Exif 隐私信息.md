---
title: Go读取和删除照片Exif隐私信息
abbrlink: 48989
date: 2021-01-25 18:35:30
updated: 2021-01-25 18:35:30
tags:
  - Golang
categories:
  - - 技术
cover: https://gooohlan.fishpi.cn/img/4cc9165193ad49be81eee216717bac82_9c26c634gy1fokyi7gt6aj21kw0wahdt1.jpg
keywords: Golang，Go，Exif，读取Exif，删除Exif
description: 使用Go读取照片中Exif的隐私信息，并删除它
---

# Exif

**可交换图像文件格式**（英语：Exchangeable image file format，官方简称**Exif**），是专门为数码相机的照片设定的文件格式，可以记录数码照片的属性信息和拍摄数据。

通过手机或者相机拍摄的照片中会携带Exif信息，而某些手机默认会携带地理位置等隐私信息，比如下面这张图片

![IMG_20210125_191526](https://gooohlan.fishpi.cn/img/IMG_20210125_191526.jpg)

它的Exif信息为：
![](https://gooohlan.fishpi.cn/img/20210125183002.png)

可以直接看到该照片使用**Redmi K30 Pro**拍摄于**2021:01:25 19:15:28**，甚至还可以直接看到经纬度信息(手机设置中可关闭)。这种未经过处理的照片直接上传到网上之后，可能会造成隐私的泄露。

# Go读取Exif信息

通过包[goexif/exif](https://github.com/rwcarlsen/goexif)，可以查看照片的全部Exif信息

```go
func GetExif() error {
   ImgFile := "./IMG_20210125_191526.jpg"
   file, err := os.Open(ImgFile)
   if err != nil {
      fmt.Println(err)
      return err
   }
   defer file.Close()
   x, err := exif.Decode(file)
   if err != nil {
      return err
   }
   fmt.Printf("%+v", x)
   return nil
}
```

因为没找到直接操作图片Exif的库(如果有欢迎留言告诉我)，只能通过解析图片的色彩空间，然后通过此色彩空间重新生成一张新的图片因为是重新生成的图片，也就相当于抹除了Exif信息

```go
func CreateNewImage(f, newF *os.File) error {
   m, _, err := image.Decode(f)
   if err != nil {
      return err
   }
   // 获取宽高
   width, height := m.Bounds().Max.X, m.Bounds().Max.Y
   var subImage image.Image
   img := m.(*image.YCbCr)
   subImage = img.SubImage(image.Rect(0, 0, width, height)).(*image.YCbCr)
   err = jpeg.Encode(newF, subImage, &jpeg.Options{100})
   return err
}
```

# 所有代码：

```go
package main

import (
   "fmt"
   "image"
   _ "image/gif"
   "image/jpeg"
   _ "image/jpeg"
   _ "image/png"
   "os"
   "path"
   "strings"

   "github.com/rwcarlsen/goexif/exif"
)

func main() {
   ImgFile := "./IMG_20210125_191526.jpg"
   fileNameWithSuffix := path.Base(ImgFile)                        // 获取文件名带后缀
   imgType := path.Ext(fileNameWithSuffix)                         // 获取文件后缀
   fileNameOnly := strings.TrimSuffix(fileNameWithSuffix, imgType) // 获取文件名
   newImageFile := fileNameOnly + "_no_exif" + imgType
   file, err := os.Open(ImgFile)
   if err != nil {
      fmt.Println("打开文件失败")
      return
   }
   defer file.Close()
   err = GetExif(file)
   if err != nil {
      panic(err)
   }
   // 设置文件偏移,重新从第一位开始读取
   file.Seek(0, 0)
   newFile, err := os.Create(newImageFile)
   if err != nil {
      fmt.Println("file create fail")
      panic(err)
   }
   defer newFile.Close()
   err = CreateNewImage(file, newFile)
   if err != nil {
      panic(err)
   }
}

func GetExif(file *os.File) error {
   x, err := exif.Decode(file)
   if err != nil {
      return err
   }
   fmt.Printf("%+v", x)

   return nil
}

func CreateNewImage(f, newF *os.File) error {
   m, _, err := image.Decode(f)
   if err != nil {
      return err
   }
   // 获取宽高
   width, height := m.Bounds().Max.X, m.Bounds().Max.Y
   var subImage image.Image
   img := m.(*image.YCbCr)
   subImage = img.SubImage(image.Rect(0, 0, width, height)).(*image.YCbCr)
   err = jpeg.Encode(newF, subImage, &jpeg.Options{100})
   return err
}

func Create(fileName string) (*os.File, error) {
   f, err := os.Create(fileName) // 创建文件
   if err != nil {
      fmt.Println("file create fail")
      return nil, err
   }
   return f, nil
}
```