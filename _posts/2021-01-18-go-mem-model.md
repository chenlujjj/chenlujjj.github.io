---
layout: single
title:  "[笔记] Go Memory Model"
date:   2021-01-18 23:00:00 +0800
categories: go
tags: [go, memory]
---

翻译、整理自：[The Go Memory Model](https://golang.org/ref/mem)。

## Advice
如果一段程序中需要修改一块被多个 goroutine 同时访问的数据，那么必须将这些访问串行化。这样才能避免出错。
通常的方法是用channel操作或者其他同步原语（见sync和sync/atomic包）。

## Happens Before

解释了代码的乱序执行。乱序可能来自于编译器和处理器。
> Within a single goroutine, reads and writes must behave as if they executed in the order specified by the program. That is, compilers and processors may reorder the reads and writes executed within a single goroutine only when the reordering does not change the behavior within that goroutine as defined by the language specification. Because of this reordering, the execution order observed by one goroutine may differ from the order perceived by another. For example, if one goroutine executes a = 1; b = 2;, another might observe the updated value of b before the updated value of a.

定义 happens before:
* e1 happens before e2 => e2 happens after e1
* e1 既不 happens before e2，也不 happens after e2，=> e1 和 e2 是并发执行的（happen concurrently）

**Within a single goroutine, the happens-before order is the order expressed by the program.**


## Synchronization

### Initialization

> If a package p imports package q, the completion of q's `init` functions happens before the start of any of p's.

> The start of the function `main.main` happens after all `init` functions have finished.

### Goroutine creation

> The go statement that starts a new goroutine **happens before** the goroutine's execution begins.

是说：go 语句是在goroutine执行之前的。

### Goroutine destruction
The exit of a goroutine is not guaranteed to happen before any event in the program.

要达成某种相对顺序的话，得使用某种同步机理，比如锁或者通道通信。


### Channel communication
（1）
> A send on a channel **happens before** the **corresponding** receive from that channel completes.

形象记忆：接收肯定要在**对应的**发送之后嘛，如果都还没发送，接收什么呢？

注意：这里的channel无所谓是否有buffer。

```go
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
```
这段程序中，可以保证先对a赋值再打印。

（2）
> The closing of a channel **happens before** a receive that returns a zero value because the channel is closed.

将上一段程序的 `c <- 0` 换成 `close(c)` ，也是一样的效果。

（3）
> A receive from an **unbuffered** channel **happens before** the send on that channel completes.

形象记忆：对于一个没有缓冲区的channel，要往里面发送东西肯定得在接收之后啊；但是对于有缓冲区的channel就没有这个限制了，毕竟里面是有空间的，没人接收也可以照发不误。

例子：可以保证打印 hello world。
```go
package main

var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}

func main() {
	go f()
	c <- 0
	print(a)
}
```
（4）这是第（3）点的延伸。

> The kth receive on a channel with capacity C happens before the k+Cth send from that channel completes.

比如说，对于一个容量为10的通道，第1次接收操作肯定在第11次发送操作之前（想想看，如果没做完第1次接收操作，第11次发送是没法进行的，因为通道已经满了）。


### Locks
> For any `sync.Mutex` or `sync.RWMutex` variable `l` and n < m, call n of `l.Unlock()` **happens before** call m of `l.Lock()` returns.

这是说：获取锁（写锁）必须在释放锁（写锁）之后。


> For any call to `l.RLock` on a `sync.RWMutex` variable `l`, there is an n such that the `l.RLock` **happens (returns) after** call n to `l.Unlock` and the matching `l.RUnlock` **happens before** call n+1 to `l.Lock`.

这是说：对于 `sync.RWMutext`，（1）如果写锁被持有了，那么获取读锁必须在释放该写锁之后；（2）如果试图再次持有写锁，必须在该读锁释放之后。即按照 Unlock -> Rlock -> RUnlock -> Lock 的顺序执行。

### Once

> A single call of `f()` from `once.Do(f)` **happens (returns) before** any call of `once.Do(f)` returns.

这是说：如果有多个goroutine运行了`once.Do(f)`，那么其中只有一个goroutine会执行`f()`，并且其他goroutine的`once.Do(f)`会阻塞，直到`f()`执行完毕返回。
这在某些场景下可以用来同步。

例如：
```go
package main

import (
	"sync"
	"time"
)

var a string
var once sync.Once

func setup() {
	a = "hello, world"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}

func main() {
	twoprint()
	time.Sleep(time.Second)
}
```
这段程序会把 "hello, world" 打印两遍。
