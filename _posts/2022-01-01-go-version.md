---
layout: single
title:  "Go构建产物版本的那些事"
date:   2022-01-01 15:30:00 +0800
categories: go
tags: [go, version]
---

本文简单讨论两个问题：
（1）对于一个给定的go binary文件，如何获取构建它的go版本？
（2）如何将go版本、构建时间、构建环境、git分支和commit id等信息植入到binary文件中？


对于第一个问题，Dave Cheney 在他的博客[How to find out which Go version built your binary](https://dave.cheney.net/2017/06/20/how-to-find-out-which-go-version-built-your-binary)中已经做了详细阐述。简而言之，就是利用`runtime.Version()`函数和`runtime.buildVersion`变量来获取go版本。这在不同的操作系统上有着不同的实践形式。

* Linux/FreeBSD/OpenBSD/NetBSD：使用gdb命令
* Darwin：使用gdb（需要先预装好）或者delve或者lldb
* Windows：使用delve

不过，我注意到在 `runtime/extern.go` 文件中有这么一段代码：
```go
// buildVersion is the Go tree's version string at build time.
//
// If any GOEXPERIMENTs are set to non-default values, it will include
// "X:<GOEXPERIMENT>".
//
// This is set by the linker.
//
// This is accessed by "go version <binary>".
var buildVersion string

// Version returns the Go tree's version string.
// It is either the commit hash and date at the time of the build or,
// when possible, a release tag like "go1.3".
func Version() string {
	return buildVersion
}
```

也就是说，直接使用 `go version <binary>` 就可达到目的了。这种方法无需借助第三方工具，并且是OS无关的，应当是首选。 推测Dave在写博客时应该还没有这个命令吧。使用效果：
```shell
~/C/go/playground ❯ go version ./version
./version: go1.17
```


对于第二个问题，先说一下为什么要这么做。植入这些信息可以帮助我们快速识别出在某个测试或者生产环境中的binary是什么版本的，对应着代码库中的哪个分支和哪次提交，是谁构建出来的。这在排查问题时非常有用。

很多开源软件都提供这个feature，比如：
```shell
$ ./promtool --version
promtool, version 2.32.1 (branch: check-metrics-weight, revision: 290054ca419ddaa9f58e27a93a6f972dd6fd1202)
  build user:       chenluxin@Chens-MacBook-Pro.local
  build date:       20211219-07:06:46
  go version:       go1.17
  platform:         darwin/amd64
```

Prometheus社区的这项功能**应该**是由[promu工具](https://github.com/prometheus/promu)提供的，对此尚未做过细致研究。

我们自己写程序也可以轻松实现该功能，主要是借助于go编译工具的`-ldflags`参数。

我已经把相关demo代码放在了[goversion](https://github.com/chenlujjj/goversion)仓库中，可以拿来快速参考。

效果如下：
```shell
$ make build

$ ./main --version
version 1.0.0
build user: chenluxin
build date: 2022-01-01T08:09:49Z
go version: go1.17
platform: darwin/amd64
git branch: master
git commit: e5376223b6fa31d8893546eb77babc714dd589d0
```

有一个开源工具[govvv](https://github.com/ahmetb/govvv)做了更进一步的工作。它是在`go build`命令之上做了一层封装，将`GitCommit`、`GitBranch`、`Version`等信息在构建时加入到程序中，拿来即用，不用自己修改`main.go`、`Makefile`文件了。


参考：
* [Adding Version Information to Go Binaries](https://medium.com/the-go-journey/adding-version-information-to-go-binaries-e1b79878f6f2)
* [Compile git version inside go binary](https://gist.github.com/awalterschulze/d0f217db28e17b2ee56e8e8a4e68c28e)