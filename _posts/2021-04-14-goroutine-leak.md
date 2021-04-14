---
layout: single
title:  "Goroutine 泄露问题排查"
date:   2021-04-14 22:00:00 +0800
categories: go
tags: [go, goroutine, leak]
---

记录下今天排查一个goroutine泄露问题的过程：

1. 访问`/debug/pprof/` 查看goroutine数量，观察一下，如果数量持续增长说明确实有goroutine泄露。
2. 访问`/debug/pprof/goroutine?debug=2`查看具体的goroutine信息（`debug=1`页面的展示信息可读性较差），可以看出每个goroutine的调用堆栈，持续时间，属于何种操作（chan send/chan receive/select/IO wait/...）。
那么，其中持续时间比较久的很有可能就是泄露的goroutine。
3. 这次的碰到的泄漏问题，原因在于一块单生产者-多消费者代码写的不对，很多消费者协程没有结束（这个生产-消费模式的代码是从网上借鉴的，还是没有仔细思考就拿来用了）。 这也提醒我们，在编写goroutine代码时，要问问自己：**这个协程能够按照我们预期的方式结束吗？** （[Never start a goroutine without knowing how it will stop](https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop)）
4. 使用`runtime.NumGoroutine()`获取goroutine数量
5. 更多阅读：

* <https://www.ardanlabs.com/blog/2018/11/goroutine-leaks-the-forgotten-sender.html>

* <https://www.openmymind.net/Leaking-Goroutines/>

* <https://wudaijun.com/2018/04/go-pprof/>

* <https://lessisbetter.site/2019/05/18/go-goroutine-leak/>
