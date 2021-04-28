---
layout: single
title:  "[翻译]Go的工作窃取调度器"
date:   2021-04-27 23:00:00 +0800
categories: go
tags: [go, goroutine, runtime, scheduler]
---

原文：[Go's work-stealing scheduler](https://rakyll.org/scheduler/)

Go的调度器的职责是将可运行的goroutine分配在多个OS线程上，而这些线程又运行在**一个或者多个**处理器上。在多线程计算调度中，有两种范式：工作分享和工作窃取。

* 工作分享（work-sharing）：当一个处理器生成新的线程时，会试图将其中一些线程转移到其他处理器上，指望它们能够被空闲或者利用率低的处理器使用。
* 工作窃取（work-stealing）：一个使用率低的处理器“窃取”其他处理器的线程。

线程转移在工作窃取模式中比在工作分享模式中发生得更少。当所有处理器都有工作要做时，没有线程会被转移。而一旦有一个空闲的处理器时，就要考虑线程转移了。

Go从1.1版本开始有一个工作窃取调度器，这出自Dmitry Vyukov之手。这篇文章会深入解释什么是工作窃取调度器以及Go是如何实现它的。

## 调度基础

Go 有一个M:N的调度器来利用多处理器。在任意时刻，M个goroutine需要被调度在N个OS线程上，而这N个线程又是运行在**最多 GOMAXPROCS**个处理器上的。Go的调度器使用下列术语来指代goroutine，线程和处理器：

* G：goroutine
* M：OS线程（machine）
* P：processor

有两种goroutine队列：全局队列和P的局部队列。每个M应该被分配给一个P。当被阻塞或者在执行系统调用时，P可能没有M。**在任意时刻，最多有 GOMAXPROCS 个P。在任意时刻，每个P只能执行一个M。** 如果需要，调度器可以创建更多个M。

![scheduler-concepts](/assets/images/scheduler-concepts.png)

每轮调度做的事情其实就是在寻找一个可执行的goroutine并执行它。在每轮调度过程中，按如下顺序搜寻goroutine：

```go
runtime.schedule() {
    // only 1/61 of the time, check the global runnable queue for a G.
    // if not found, check the local queue.
    // if not found,
    //     try to steal from other Ps.
    //     if not, check the global runnable queue.
    //     if not found, poll network.
}
```

一旦找到一个可运行的G，就会执行它直到被阻塞。

备注：看起来似乎全局队列优于局部队列，但是时不时地检查全局队列对于避免M只从局部队列调度直到队列中没有goroutine为止至关重要。

> 译注：这段备注没看明白。如果没有全局队列会有什么影响呢？

## 窃取

当一个新的G被创建或者一个已存在的G变成可运行状态时，它会被推入当前P的可运行goroutine的列表中。当P完成执行G时，它试图将G从该列表中弹出。如果列表为空，P从其他P中随机选择一个，然后从被选中的P的局部队列中窃取一半可运行的goroutine。

![scheduler-stealing](/assets/images/scheduler-stealing.png)

在上述例子中，P2找不到可以运行的goroutine。所以，它会随机地选择另一个处理器P1，然后从它的局部队列中窃取三个goroutine。P2能够运行这些goroutine，这就使得调度工作在多个处理器之间被更加公平地分配了。

## 自旋线程

调度器总是希望将尽可能多的goroutine分配到M上来利用处理器，但是同时我们又需要停止过多的工作来节省CPU和能量。与此相矛盾的是，调度器也应该要可扩展到高吞吐量和CPU密集型的程序。

如果性能非常重要，那么持续抢占不仅代价昂贵，而且对高吞吐量的程序来说也是一个问题。OS线程不应该在可运行的goroutine之间频繁切换（hand-off），因为这会导致延迟增加。除此之外，在存在系统调用的情况下，OS线程需要不断地阻塞和解除阻塞。这是昂贵的，增加了大量的开销。

为了最小化切换（hand-off），Go调度器实现了“自旋线程”。自旋线程会消耗一些额外的CPU，但它们会最小化对OS线程的抢占。一个线程如果满足以下条件之一，那么它就是自旋的:

* 一个被分配给P的M正在寻找一个可运行的goroutine
* 一个没有分配给P的M正在寻找可用的P
* 如果有一个空闲的P并且没有其他的自旋线程时，调度器会启动一个额外的线程，并在它准备一个goroutine时“旋转”它

**在任意时刻，最多只有 GOMAXPROCS 个自旋M。** 当一个自旋线程找到事情做时，就脱离了旋转状态了。

如果有没分配给P的空闲M，那么分配给P的空闲线程就不会阻塞。当新的goroutines被创建或者M被阻塞时，调度器确保至少有一个自旋的M。这确保没有可运行的goroutines可以运行；并且避免过多的M阻塞/解除阻塞。

> 译注：这里最后一段话实在没看明白...


## 结论

Go调度器做了很多工作来避免OS线程的过度抢占，这包括通过工作窃取将它们调度到利用率低的处理器上，实现“自旋”线程来避免经常发生阻塞/解除阻塞的转换。

调度事件可以通过[tracer](https://golang.org/cmd/trace/)来追踪。如果发现程序对处理器的利用率低，可以通过这个工具来进一步调查原因。


### 参考

* [The Go runtime scheduler source](https://github.com/golang/go/blob/master/src/runtime/proc.go)
* [Scalable Go Scheduler design document](https://golang.org/s/go11sched)
* [The Go scheduler by Daniel Morsing](https://morsmachine.dk/go-scheduler)