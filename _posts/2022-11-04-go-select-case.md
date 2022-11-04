---
layout: single
title:  "使用 select case 语句的注意点"
date:   2022-11-04 23:00:00 +0800
categories: go
tags: [go]
---

## 在 for 循环的 select case 中使用 break 和 continue

看下面两段代码：

（1）continue

```go
func main() {
	data := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		data <- i
	}
	close(data)

	var a int
	var ok bool
	for {
		select {
		case a, ok = <-data:
			if !ok {
				return
			}
			if a == 3 {
				continue
			}
		}
		println(a)
	}
}
```

输出是：

```
1
2
4
5
```

（2）break

```go
func main() {
	data := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		data <- i
	}
	close(data)

	var a int
	var ok bool
	for {
		select {
		case a, ok = <-data:
			if !ok {
				return
			}
			if a == 3 {
				break
			}
		}
		println(a)
	}
}
```

输出是：

```
1
2
3
4
5
```

可见：

* `continue` 的效果是直接跳转到 `for` 循环的下一次迭代，和常规理解无异
* `break` 的效果是跳出 `select` 语句，而**不是**跳出 `for` 循环，和常规认知不一样，这需要注意

如果想让 `break` 跳出 `for` 循环，该这么写：

```go
loop:
	for {
		select {
		case a, ok = <-data:
			if !ok {
				return
			}
			if a == 3 {
				break loop
			}
		}
		println(a)
	}
```

输出是：

```
1
2
```

在 go spec 中对 `continue` 和 `break` 语句的行为有准确描述，和上述实验结果相符：

> A "break" statement terminates execution of the innermost ["for"](https://go.dev/ref/spec#For_statements), ["switch"](https://go.dev/ref/spec#Switch_statements), or ["select"](https://go.dev/ref/spec#Select_statements) statement within the same function.

> A "continue" statement begins the next iteration of the innermost enclosing ["for" loop](https://go.dev/ref/spec#For_statements) by advancing control to the end of the loop block. The "for" loop must be within the same function.



## select 不会阻塞在对已经 close 的 channel 做发送操作的 case 分支上

看代码：

```go
func main() {
	data := make(chan int, 1)
	close(data)

	for {
		select {
		case data <- 1:
			println("send")
		default:
			println("default")
		}
	}
}
```

本来以为输出会是 `default`，毕竟第一个 `case` 分支里向 channel 里发不进去呀。

然而程序实际上 panic 了：

```
panic: send on closed channel

goroutine 1 [running]:
main.main()
        /Users/chenluxin/Codes/go/playground/select.go:9 +0x4b
exit status 2
```

在 go spec 中对 select statement 的说明：

> A "select" statement chooses which of a set of possible [send](https://go.dev/ref/spec#Send_statements) or [receive](https://go.dev/ref/spec#Receive_operator) operations will proceed.

> If one or more of the communications can proceed, a single one that can proceed is chosen via a uniform pseudo-random selection. Otherwise, if there is a default case, that case is chosen. If there is no default case, the "select" statement blocks until at least one of the communications can proceed.

所以， 对已经 close 的 channel 发送数据也被视为一种 `will proceed` 的操作。

需要注意的是，对 nil channel 接收或发送则**不是**一种可以 `proceed` 的操作：

```
func main() {
	data := make(chan int, 1)
	data <- 1
	close(data)
	data = nil

	select {
	case data <- 1:
		println("send")
	default:
		println("default")
	}
}
```

这段代码输出是 `default`。



关于为何 `select` 可以挑选对已经 close 的 channel 做发送操作的 `case` 分支来执行，已经有人问过这个问题了：https://github.com/golang/go/issues/11344。Go 维护者的理由主要有：

* 对 channel 的操作在`select` 语句内部的表现应该和此操作独立存在时的表现一致
* 如果 `select` 不能挑选这种 `case` 分支，那么会导致 receiver 能够通过 close 操作来告知 sender 停止发送数据（receiver 通知 sender），而这和 close 的本来设计意图——sender 告诉 receiver 发送已经结束（sender 通知 receiver），相违背

## 参考

* [Go Spec: Select statements](https://go.dev/ref/spec#Select_statements)
* [runtime: select should ignore closed channels #11344](https://github.com/golang/go/issues/11344)



