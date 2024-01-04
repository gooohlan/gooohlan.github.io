---
title: go发现网络活跃主机
tags:
  - Golang
categories:
  - - 技术
    - 后端
keywords: 'golang, go, ping, 抓包'
description: go开发一个发现网络活跃主机的嗅探器。
abbrlink: 27211
date: 2021-08-29 16:59:53
updated: 2021-08-29 16:59:53
cover: https://gooohlan.fishpi.cn/img/20210829173455.png
---

### 前言

&emsp;&emsp;最近实际开发中突然遇到一个比较有意思的需求，大体内容是我需要从前端给我的一堆主机IP中拿出其中处于活跃状态的主机，说简单的就是能`ping`通的主机，乍一看不难，就想着去网上copy一份解决方案，但是乱七八糟一堆却没有实际解决问题的，大多是通过调用ping命令来实现，或者就是携带端口扫描的，这些都与我目前想实现的方式相背，所以都不能采用。

### 思考🤔

不调用主机ping命令的情况下，自己去实现一个ping，基于这个思路，继续寻找解决方案，这才倒是有了，但都是通过ICMP协议，启动时需要sudo权限，这又与我所需要的相背，我需要的是不依赖于ICMP也可以Ping的方案。

### 正文

本文主角：[go-ping](https://github.com/go-ping/ping)，它可以通过 UDP 发送一个“非特权”ping，与我的实际需求完全一致

![image-20210829172002459](https://gooohlan.fishpi.cn/img/20210829172002.png)

看一个官方示例的ping程序：

```go
pinger, err := ping.NewPinger("www.google.com")
if err != nil {
	panic(err)
}

// Listen for Ctrl-C.
c := make(chan os.Signal, 1)
signal.Notify(c, os.Interrupt)
go func() {
	for _ = range c {
		pinger.Stop()
	}
}()

pinger.OnRecv = func(pkt *ping.Packet) {
	fmt.Printf("%d bytes from %s: icmp_seq=%d time=%v\n",
		pkt.Nbytes, pkt.IPAddr, pkt.Seq, pkt.Rtt)
}

pinger.OnDuplicateRecv = func(pkt *ping.Packet) {
	fmt.Printf("%d bytes from %s: icmp_seq=%d time=%v ttl=%v (DUP!)\n",
		pkt.Nbytes, pkt.IPAddr, pkt.Seq, pkt.Rtt, pkt.Ttl)
}

pinger.OnFinish = func(stats *ping.Statistics) {
	fmt.Printf("\n--- %s ping statistics ---\n", stats.Addr)
	fmt.Printf("%d packets transmitted, %d packets received, %v%% packet loss\n",
		stats.PacketsSent, stats.PacketsRecv, stats.PacketLoss)
	fmt.Printf("round-trip min/avg/max/stddev = %v/%v/%v/%v\n",
		stats.MinRtt, stats.AvgRtt, stats.MaxRtt, stats.StdDevRtt)
}

fmt.Printf("PING %s (%s):\n", pinger.Addr(), pinger.IPAddr())
err = pinger.Run()
if err != nil {
	panic(err)
}
```

实际运行效果:

> PING www.baidu.com (14.215.177.38):
> 24 bytes from 14.215.177.38: icmp_seq=0 time=29.801ms
> 24 bytes from 14.215.177.38: icmp_seq=1 time=37.256ms
> 24 bytes from 14.215.177.38: icmp_seq=2 time=36.844ms
> 24 bytes from 14.215.177.38: icmp_seq=3 time=36.24ms
> 24 bytes from 14.215.177.38: icmp_seq=4 time=30.73ms
> 24 bytes from 14.215.177.38: icmp_seq=5 time=36.381ms
> 24 bytes from 14.215.177.38: icmp_seq=6 time=36.173ms
> 24 bytes from 14.215.177.38: icmp_seq=7 time=29.764ms
> 24 bytes from 14.215.177.38: icmp_seq=8 time=29.755ms
> 24 bytes from 14.215.177.38: icmp_seq=9 time=29.676ms
> 24 bytes from 14.215.177.38: icmp_seq=10 time=29.63ms
> 24 bytes from 14.215.177.38: icmp_seq=11 time=36.294ms
> 24 bytes from 14.215.177.38: icmp_seq=12 time=36.003ms
>
> --- www.baidu.com ping statistics ---
> 13 packets transmitted, 13 packets received, 0% packet loss
> round-trip min/avg/max/stddev = 29.63ms/33.426693ms/37.256ms/3.295477ms

他通过发送和接收到的包来计算丢失率，在`pinger.OnFinish`时即可得到结果，但是我们实际运用的时候却不能去手动关闭这个`ping`程序来得到结果，通过阅读源码发现：

```go
type Pinger struct {
	// Interval is the wait time between each packet send. Default is 1s.
	Interval time.Duration

	// Timeout specifies a timeout before ping exits, regardless of how many
	// packets have been received.
	Timeout time.Duration
    .....
}
```

有了等待时间与超时时间，就可以通过超时而结束，在`pinger.OnFinish`中查看是否有收到包，以此判断主机是否处于活跃状态

### 最终代码

```go
func PingIP(ip string) bool {
   pinger, err := ping.NewPinger(ip)
   if err != nil {
      return false
   }
   var b bool
   pinger.Interval = 1 * time.Millisecond // 发送间隔
   pinger.Timeout = 5 * time.Millisecond // 超时时间
   pinger.OnFinish = func(stats *ping.Statistics) {
      if stats.PacketsRecv != 0 {
         b = true
      }
   }

   err = pinger.Run()
   if err != nil {
      return false
   }
   return b
}
```

通过与具体业务的整合，即可得出一个发现活跃服务器
