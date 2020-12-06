---
layout: single
title:  "go benchstat 工具"
date:   2020-12-06 14:00:00 +0800
categories: go
tags: [go, profile, performance]
---

go 官方提供了一个 benchstat 的命令行工具，用于计算和比较 benchmark 的统计结果。

## 安装
`go get -u golang.org/x/perf/cmd/benchstat`

## 使用
参看[文档](https://pkg.go.dev/golang.org/x/perf/cmd/benchstat)

```shell
$ benchstat -h
usage: benchstat [options] old.txt [new.txt] [more.txt ...]
options:
  -alpha α
    	consider change significant if p < α (default 0.05)
  -csv
    	print results in CSV form
  -delta-test test
    	significance test to apply to delta: utest, ttest, or none (default "utest")
  -geomean
    	print the geometric mean of each file
  -html
    	print results as an HTML table
  -norange
    	suppress range columns (CSV only)
  -sort order
    	sort by order: [-]delta, [-]name, none (default "none")
  -split labels
    	split benchmarks by labels (default "pkg,goos,goarch")
```

* 当输入是一个文件时：打印出该文件中的 benchmark 统计结果
* 当输入是一对文件时：分别打印出这两个文件的 benchmark 统计结果，以及它们的比较信息
* 当输入是三个及以上数量的文件时：分别打印出这多个文件的 benchmark 统计结果

## 举例

以 [How to write benchmarks in Go](https://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go) 文章中的 `Fib` 函数为例做基准测试。

（1）执行两次 benchmark 测试，并保存结果
第一次：
```shell
$ go test -run=NONE -bench=Fib40 -count=5 | tee -a old.txt
goos: darwin
goarch: amd64
BenchmarkFib40-12              2         502193602 ns/op
BenchmarkFib40-12              3         490639873 ns/op
BenchmarkFib40-12              3         505765331 ns/op
BenchmarkFib40-12              3         494034920 ns/op
BenchmarkFib40-12              3         493694745 ns/op
PASS
ok      _/Users/chenluxin/Codes/go/fib  14.372s
```

第二次：
```shell
$ go test -run=NONE -bench=Fib40 -count=5 | tee -a new.txt
goos: darwin
goarch: amd64
BenchmarkFib40-12              2         514712004 ns/op
BenchmarkFib40-12              3         493100204 ns/op
BenchmarkFib40-12              3         491940214 ns/op
BenchmarkFib40-12              3         494113238 ns/op
BenchmarkFib40-12              3         497425104 ns/op
PASS
ok      _/Users/chenluxin/Codes/go/fib  13.767s
```

（2）查看单次 benchmark 统计结果
```shell
$ benchstat old.txt
name      time/op
Fib40-12  497ms ± 2%
```
说明平均执行时间约为 497ms

（3）比较两次的统计结果
```shell
$ benchstat old.txt new.txt
name      old time/op  new time/op  delta
Fib40-12   497ms ± 2%   498ms ± 3%   ~     (p=1.000 n=5+5)
```

注意 delta 一列，值是`~`，没有显示百分比，说明两次测试结果没有显著差异。