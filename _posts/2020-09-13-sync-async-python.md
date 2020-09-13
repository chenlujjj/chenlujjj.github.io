---
layout: single
title:  "[笔记] Sync vs. Async Python"
date:   2020-09-13 23:00:00 +0800
categories: python
tags: [python, async]
---

> 这是阅读博客 [Sync vs. Async Python: What is the Difference?](https://blog.miguelgrinberg.com/post/sync-vs-async-python-what-is-the-difference) 时记录下的笔记。

## 含义
sync 和 async 指的是应用程序使用并发（concurrency）的两种方式：
* 同步服务使用 **OS 的线程和进程**实现并发，即可以是线程，或者进程，或者是两者的结合。缺点是并发能力受到 worker 数量的限制。
* 异步服务运行在由 loop 控制的**单个进程且单个线程**中。loop 是task manager and scheduler，负责创建 task 来执行客户端发起的请求。异步任务是 short-lived 的，当请求完成时就会销毁。loop 和 task 的交互方式是这样的：当 task 需要等待时，比如读写文件，数据库连接等，它会“告诉” loop 要等待什么，然后将程序控制权交给 loop；loop 会寻找另一个可运行的 task 执行；当等待结束时，如读文件完成，loop 会重新“唤醒” task 继续执行。 这种 suspend/resume 的执行模式很容易让人联想到 generator，而这就是 Python asyncio 的实现方式。

## Python 中实现 async 的两种方式

所有的异步程序都需要上述的 suspend/resume feature，而在具体实现方式上，分为了两种：

**（1）基于 coroutines**
这个领域有 Python3 内置的 asyncio 库，还有一些第三方库，如 Trio，Curio，Twisted。
具体到编写异步 web 应用，也有一些基于 coroutines 的异步框架可供选择：aiohttp，sanic，FastAPI，Tornado。

**（2）基于 greenlet**
基于 coroutine 的方法依赖于一些语言特性和特定语法，如 `yield`，`await`，`async`。而 greenlet 则没有这种限制，也就是说，可以**异步地执行同步的代码**。
基于 greenlet 的库著名的有 **Gevent** 和 **Eventlet**。它们都有对于 async loop 的实现，并且支持 “monkey-patching”，即给同步的代码打上“补丁”后就能以异步的方式执行。

## Async 要比 Sync 快吗

不能一概而论，需要具体分析。

首先要知道的是，有两个因素会影响并发程序的性能：**上下文切换和可扩展性**。
* **Context-Switching**：对于同步应用，上下文切换是由操作系统完成的，对于应用程序来说这是一个无法配置和调优的黑盒。而对于异步应用，上下文切换是由 loop 完成的。asyncio 库实现的 loop 不是特别高效，**uvloop** 用 C 实现，性能更好。Gevent 使用的 event loop 也是 C 实现的，而 Eventlet 的 loop 是 Python 写的。 注意，一个高度优化过的 async loop 在做上下文切换时确实要比 OS 更高效，但是这种优势只有在高并发场景下才能体现出来。对于大多数应用程序，同步和异步的上下文切换的性能差距并不明显。
* **Scalablity**：异步应用由于可以快速的创建成百上千的 task，其可扩展性要比同步应用更好。但是，如果任务是 CPU 密集型的，异步应用并不会比同步应用更快（毕竟，CPU 执行指令的速度是固定的）。如果任务是 IO 密集型的，异步应用能够更加有效地利用 CPU，达到更高的性能。

基于上述两个因素，只有在特定场景下异步才会比同步要快：
* 高负载，高并发
* I/O 密集型任务
* 针对于“快”的度量，我们关注的是**单位时间里的请求数量**，即 QPS，而不是**每个请求的处理时间**。如果是以后者作为指标，异步的速度可能还不及同步，考虑到有更多的并发任务在竞争 CPU 资源。

## Bonus
对于 loop 概念，给作者提了问题，他的回答是：
> It is called "loop", but you can think of it as a task scheduler. Nothing to do with for-loops or while-loops. The async loop is just a piece of code that orchestrates the running of all the tasks. You could say that the async loop loops on the task list, giving each a chance to run.