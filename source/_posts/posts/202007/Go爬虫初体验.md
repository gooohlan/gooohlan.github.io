title: Go爬虫初体验
date: '2020-07-09 17:36:46'
updated: '2020-10-21 01:12:39'
tags: [Golang, 学习]
categories:
  - [技术, 后端]
permalink: /articles/2020/07/09/1594287406684.html
cover: https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/szmmbizjpgNOM5HN2icXzxucy69Kn84IROe6uFoU12S3otjxj8deFsL4O9uSEs5p5CUkkzOiaaDJlEUnIJWlT0xhF7qE3U2GIQ640.jpeg
---

## 前言

  闲来无事的时候，偶尔也会看看漫画，但是鹅厂的操作大家都懂，想看最新的你就得给钱，本着白嫖精神，我找到了[扑飞漫画](http://www.pufei8.com/)，但是这网页的阅读体验一言难尽，他家的APP也是，动不动就加载失败，一等一半天。思来想去，还是弄个爬虫把图片都爬下来，然后想法弄到kindle里面岂不美哉。因为不会Python，所以只好用GO来写了，虽然没写过，但是可以现学嘛。

## 初识爬虫

  网上找了下资料，go的写爬虫也太简单了吧，几行代码就搞定了，比如下面这样，几行代码就把整个页面拿到了。

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	resp, err := http.Get("https://www.baidu.html")
	if err != nil {
		fmt.Println("http get error", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("read error", err)
		return
	}
	fmt.Println(string(body))
}
```

  得知爬虫这么简单的我笑出了声，剩下的工作也就是获取网址源码，然后正则解析出我想要的东西，循环以上步骤拿到图片地址下载即可，整体可分为三步。

1. 爬取漫画首页，获取目录标题极其链接
2. 通过目录拿到的链接去访问图片所在页面，然后解析出图片地址
3. 下载图片，并存储在指定页面

  有了如上结论的我开始奋笔疾书，然后当我进行到第二步获取图片地址的时候，发现爬下来的页面与网页实际的不一样。

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-5aa0dbd7.png)

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-1b67caf2.png)

  我获取到网页内容是一个loading页面，而不是这个页面最终的状态。起初的想法是因为网页一开始打开时这样，加载完后就正常了，所以我隔几秒在读取网页内容，事实证明这样不行。

## 意外之喜

  因为不知道啥原因导致的，所以也没找到解决办法，期间尝试了一下爬虫框架也依旧无果。但是也没有彻底放弃，没事就看看这页面的源码，希望能找出解决办法。

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-4b9ea23b.png)

  乍一看是某种解密的东西，尝试运行下发现可以得到这一页的图片地址，还剩下几个变量又都是干嘛的呢，好奇之下运行了一下。

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-82545a4e.png)

  如此一来，整个章节所有图片的地址也就都有了。问题也就是怎么在go里运行js，然后就找到了一个叫[otto](https://github.com/robertkrimen/otto)的包，完美。

## 开始书写

  因为在寻找解决办法的过程中接触了一些爬虫框架，整体感觉比源码撸方便一点，主要是不用自己写正则，可以像js操作DOM节点那样找到自己想要的内容，所以就换了框架[colly](github.com/gocolly/colly)来完成整改爬虫。

  定义一个全局变量，用于存放章节与之对应的链接以及该章节下的所有图片

```go
type Catalog struct {
   Title  string
   Url    string
   ImgArr []string
}

var catalog []*Catalog
```

### 1. 漫画首页

  第一步是爬取漫画的首页，通过获取目录内容去获取到章节的标题以及该章节的地址。比如[一人之下](http://www.pufei8.com/manhua/419/)，目录存放在 `id="play_0"`下的无序列表中，获取对应 `li`标签内 `a`标签内容即可。

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-bad891eb.png)

```go
c := colly.NewCollector()
c.OnHTML("#play_0", func(e *colly.HTMLElement) {
   e.ForEach("ul li a", func(i int, element *colly.HTMLElement) {
      href := element.Attr("href")
      title := element.Text
      title = coverGBKToUTF8(title) // 页面编码转为UTF8
      catalog = append(catalog, &Catalog{Url: PF + href, Title: title})
   })
})

c.OnRequest(func(r *colly.Request) {
})

c.Visit(MANHUA + strconv.Itoa(Mid))
```

  扑飞漫画整站的编码是 `gb2312`，因为go只认 `utf-8`，所以对于爬取的内容需要处理下。

```go
import "github.com/axgle/mahonia"

func coverGBKToUTF8(src string) string {
   return mahonia.NewDecoder("gbk").ConvertString(src)
}
```

### 2. 获取图片地址

  通过上面的代码，我们已经获取了所有目录对应地址，接下来需要做的只是去到对应页面，获取JS代码并执行，获得该章节下所有的图片地址

```go
// 获取该章节所有图片地址
func GetImgArr(catalog *Catalog) {
   c := colly.NewCollector()
   c.OnHTML("head script", func(e *colly.HTMLElement) {
      if e.Text != "" {
         JavaScript := coverGBKToUTF8(e.Text)
         JavaScript += " function f() {return photosr;} f();"
         vm := otto.New()
         value, err := vm.Run(JavaScript)
         if err != nil {
            log.Println("解析图片地址失败:", err)
         }
         imgStr, err := value.ToString()
         if err != nil {
            log.Println("图片地址解析出错:", err)
         }
         imgArr := strings.Split(imgStr, ",")
         catalog.ImgArr = imgArr[1:]
      }
   })

   c.OnRequest(func(r *colly.Request) {
   })

   c.Visit(catalog.Url)
}
```

### 3. 下载图片

  这个网上一搜图片一大堆，所以直接copy一份就可以了

```go
// 创建文件夹并存储图片
func CreateFileGetImg(catalog *Catalog, index int) {
   if catalog.Title == "通知" {
      return
   }
   dir := DIR + strconv.Itoa(index) + "--" + catalog.Title
   err := os.MkdirAll(dir, os.ModePerm)
   if err != nil {
      log.Println("创建文件夹失败")
   }

   for k, v := range catalog.ImgArr {
         resp, err := http.Get(ImgHeader + v)
         if err != nil {
            log.Println(err)
         }
         body, _ := ioutil.ReadAll(resp.Body)
         out, _ := os.Create(dir + "/" + strconv.Itoa(k+1) + ".jpg")
         io.Copy(out, bytes.NewReader(body))
         resp.Body.Close()

}
```

### 4. 细节处理

  在获取图片地址并下载图片的时候，可能会出现类似 `Get http://res.img.fffmanhua.com/2017/08/22/20/28acd5236d.jpg: EOF`的错误，我处理的方式是再获取一次，知道它不报错为止。

```go
GetImage:
		resp, err := http.Get(ImgHeader + v)
		if err != nil {
			log.Println(err)
			goto GetImage // 获取图片出错重新获取
		}
```

### 5. 并发

  上述代码已经可以完成我的程序需求了，但是面对漫画量很大的情况下，执行的时间还是很吃力的，所以需要加上并发。一开始的考虑是在获取章节的时候直接并发，顺道把图片也下载了。

```go
var wg sync.WaitGroup
	for k, v := range catalog {
		wg.Add(1)
		go func(v *Catalog, k int) {
			//获取图片地址
			GetImgArr(v)
			if runtime.NumGoroutine() > MaxNum {
				MaxNum = runtime.NumGoroutine()
			}
			//创建文件夹并获取图片
			CreateFileGetImg(v, len(catalog)-k)
			wg.Done()
		}(v, k)
	}
	wg.Wait()
```

  我在编辑器Goland里面运行的时候是没有问题的，但是当我编译后拿去Windows和Mac里运行就会出一大堆的问题，最后只能限制一下并发量的问题，最终的解决办法是并发获取图片目录信息，然后做一个队列去下载图片，同时最多下载10张，这样一来，性能也能够吃得消，虽然慢一点，但是它稳。

## 源码

https://github.com/InkDP/PF

## 后记

  吐槽一下，苹果有时候真的很坑，我在通过队列下载的时候，出现了 `catalog`数组中的 `imgArr`长度为为空的情况，调试了很久都没用，扔给别人在Windows上就乱跑。

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-cad22904.png)

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-573937e9.png)

&emsp;&emsp;最后的最后附上一张成功的截图
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-1a984f24.png)

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-c06579cf.png)

![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-fdd3f60d.png)
