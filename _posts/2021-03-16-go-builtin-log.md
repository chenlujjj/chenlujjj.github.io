---
layout: single
title:  "Go内置log库"
date:   2021-03-16 23:00:00 +0800
categories: go
tags: [go, log]
---

Go语言内置的log库提供的API很简洁：
* `Default`方法返回内置的标准`Logger`：无prefix，日志写到 `os.Stderr`
* `New`方法用于自定义`Logger`，该方法的函数签名是 `func New(out io.Writer, prefix string, flag int) *Logger`。可见只要是实现了`io.Writer`的，都可以作为日志写入的对象，如标准输出，标准错误，`/dev/null` 等。这也暗合Linux“一切即文件”的思想。
* `Logger`结构体，包括内置的标准`Logger`，提供了一系列写日志和设置日志属性的API，包括`Print[f|ln]`，`Fatal[f|ln]`， `Panic[f|ln]`，`SetOutput`，`SetFlags`，`SetPrefix` 等。
* `Logger`结构体中含有互斥锁 `sync.Mutex`，用于保证写日志是并发安全的。换句话说，可以在多个goroutine中同时使用`Logger`来写日志。查看源码可知，`Output`方法和`SetXxx`方法都有用到这个互斥锁。

想法：
* 内置的log库并没有level（DEBUG/WARN/ERROR）的概念，这点和python的log库不一样
* log库对日志格式的控制是通过flags完成的，可定制化不多
* 使用互斥锁可能会造成大量写日志时效率不高，这或许是产生许多第三方日志库的原因之一

参考：
* [godoc](https://golang.org/pkg/log)
* [source code](https://golang.org/src/log/log.go)
* [Using The Log Package In Go](https://www.ardanlabs.com/blog/2013/11/using-log-package-in-go.html)