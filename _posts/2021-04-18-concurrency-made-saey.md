---
layout: single
title:  "Concurrency made easy - GopherCon SG 2017"
date:   2021-04-18 17:00:00 +0800
categories: go
tags: [go, concurrency, talk]
---

Dave Cheney 的演讲总结：

* If you have to wait for the result of an operation, it's easier to do it yourself.
* Release locks and semaphores in the reverse order you acquired them.
* Channels aren't resources like files or sockets, you don't need to close them to free them.
* Acquire semaphores when you're ready to use them.
* Avoid mixing anonymous functions and goroutines.  -- loopclosure
* Before you start a goroutine, always know when, and how, it will stop.
