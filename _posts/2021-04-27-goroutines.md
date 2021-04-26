---
layout: single
title:  "[笔记]A complete journey with Goroutines"
date:   2021-04-27 00:30:00 +0800
categories: go
tags: [go, goroutine, runtime]
---

原文：[A complete journey with Goroutines](https://riteeksrivastava.medium.com/a-complete-journey-with-goroutines-8472630c7f5c)

### 什么是Goroutine？

* Goroutine是golang中实现并发（concurrently）执行任务的方式
* Goroutine是在线程基础上轻量级的抽象，相比起线程它们的创建和销毁都很便宜

### Goroutine 和线程的区别

* 内存消耗：goroutine最少仅需2kb内存，而线程需要1Mb。goroutine的栈大小可以根据需要扩大或缩小。
* 创建和销毁的代价：创建、销毁线程的代价较大，因为需要从OS获取资源。而goroutine是由go运行时负责创建和销毁的，这些操作相比起来代价很小。
* 切换代价：线程是**抢占式（preemptively）**调度的，切换过程中需要保存/恢复许多寄存器（16个通用目的的寄存器，程序计数器，栈指针，段寄存器等）。而goroutine是**合作式（cooperatively）**调度的，并不会和OS内核直接交互，当goroutine切换时，仅仅需要保存/恢复少量寄存器，所以代价比起线程小很多。

### Goroutine 的调度

前面说过，goroutine是合作式调度的。在合作式调度中，没有时间分片的概念。goroutine之间的切换只会在下面这些情况中发生：

* 对channel的阻塞式发送和接收操作
* go 语句 （但是不保证新的goroutine会立即执行）
* 阻塞的系统调用，比如文件和网络IO操作
* 被GC停止

下面介绍goroutine调度的GMP模型。

* G - Goroutine
* M - OS Thread （machine）
* P - Processor

对于一个Go程序，可用线程数量等于 GOMAXPROCS。Go使用M:N的调度器，可以利用机器上的多核：M个goroutine调度在N个OS线程上，而这N个线程又跑在最多 GOMAXPROCS 数量的 processor 上 （N<GOMAXPROCS）。

### 使用Goroutine的常见错误和避免方式

* 不检查FD限制和内存限制：当创建巨大数量的goroutine时，应该总是检查程序运行的资源限制，包括文件描述符和内存限制

### 关键词

**GOMAXPROCS** 用来控制程序中可用于执行goroutine的线程数量。当前Go版本中，默认是逻辑处理器的数量，即`runtime.NumCPU()`。

> 译注：控制线程数量，这句话暂时表示怀疑？我觉得，应该是GMP模型中P的数量。
