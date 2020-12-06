---
layout: single
title:  "go benchmark 使用"
date:   2020-12-06 15:00:00 +0800
categories: go
tags: [go, profile, performance]
---

执行基准测试的命令：
`go test -run=XXX -bench=. -benchmem -cpuprofile cpu.out -memprofile mem.out`

* 传递给 `-run` 和 `-bench` 的都是用于匹配函数名称的正则表达式。`-run` 用于匹配单元测试的函数名，`-bench` 用于匹配基准测试的函数名。
`-run=XXX` 用来排除所有的单元测试函数，毕竟不太可能有函数名为 `TestXXX`，也可以用 `-run=^$` 来达到同样效果。
`-bench=.` 表示匹配所有的 benchmark 函数。

* `-benchmem` 表示记录内存分配结果。测试的输出中 `Xxx allocs/op` 表示每次 op （或迭代）发生了多少次内存分配，`Xxx B/op` 表示每次 op 有多少字节被分配了。

* `-cpuprofile cpu.out -memprofile mem.out` 用于把 cpuprofile 和 memprofile 输出到文件供后续分析。通常可使用 `go tool pprof` 来分析。

* 每个函数的 benchmark 测试，默认情况下至少会运行1秒。可以通过传递命令行参数 `-benchtime=Ns` 来指定具体的运行时间。

* `-count=N` 表示运行该基准测试多少次。注意：不是多少次op或迭代，是设置整个测试运行的次数，例如：
```
$ go test -run=NONE -bench=Fib40 -count=5 
goos: darwin
goarch: amd64
BenchmarkFib40-12              2         519524515 ns/op
BenchmarkFib40-12              2         512280768 ns/op
BenchmarkFib40-12              2         512022998 ns/op
BenchmarkFib40-12              2         517819306 ns/op
BenchmarkFib40-12              2         514294170 ns/op
PASS
ok      _/Users/chenluxin/Codes/go/fib  7.874s
```
运行了5次测试，每次中有2次迭代。


* 测试输出中，函数名后的数字表示 `GOMAXPROCS` 的值，默认是 CPU 的核数。该值可以在执行测试时设置，例如 `GOMAXPROCS=2 go test ...`


参考：
* [How to write benchmarks in Go](https://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go)
* [What's that number in Go benchmark names?](https://mmcloughlin.com/posts/golang-benchmarks-gomaxprocs)
* [What does allocs/op and B/op mean in go benchmark?](https://stackoverflow.com/questions/35588474/what-does-allocs-op-and-b-op-mean-in-go-benchmark)
* [Real Life Go Benchmarking](https://rollout.io/blog/real-life-go-benchmarking/)
