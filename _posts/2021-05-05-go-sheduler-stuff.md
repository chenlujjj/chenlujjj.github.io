---
layout: single
title:  "Go 调度器有关资料"
date:   2021-05-05 15:30:00 +0800
categories: go
tags: [go, goroutine, sheduler]
---

（1）来自调度器作者的演讲视频，应该算是权威：
[Dmitry Vyukov — Go scheduler: Implementing language with lightweight concurrency](https://www.youtube.com/watch?v=-K11rY57K7k)

时长1h，前半小时讲Go调度器的发展，GMP模型等还能勉强听懂，后面开始讲栈，结合汇编，就看不懂了。

（2）Golang的协程调度器原理及GMP设计思想

- [文字讲义](https://github.com/aceld/golang/blob/main/2%E3%80%81Golang%E7%9A%84%E5%8D%8F%E7%A8%8B%E8%B0%83%E5%BA%A6%E5%99%A8%E5%8E%9F%E7%90%86%E5%8F%8AGMP%E8%AE%BE%E8%AE%A1%E6%80%9D%E6%83%B3%EF%BC%9F.md)
  
- [讲解视频](https://www.bilibili.com/video/BV19r4y1w7Nx?p=18&spm_id_from=pageDriver)

（3）tonybai的博客：

- [也谈goroutine调度器](https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)
  
- [译文：图解Go运行时调度器](https://tonybai.com/2020/03/21/illustrated-tales-of-go-runtime-scheduler/)
