---
layout: single
title:  "自定义 go vet check/golangci-lint linter"
date:   2020-08-27 21:00:00 +0800
categories: go
tags: [go, golangci-lint, vet]
---

## go vet check

[go vet](https://golang.org/cmd/vet/) 是 go 自带的一个代码静态分析工具，包含了一系列 check 规则，能够帮助我们检查代码中的潜在问题。

基本用法：

```sh
$ go vet
$ go vet /some/path/...
$ go tool vet help    # 查看所有 check
$ go tool vet help loopclosure  # 查看某个 check，如 loopclosure 的详细检查规则
$ go vet -loopclosure ./...   # 只激活 loopclosure 做代码分析
$ go vet -vettool=$(which shadow) ./...  # 使用第三方或者自定义的 check 做代码分析
```



vet 的 main 函数代码在 [go 主仓库](https://github.com/golang/go/blob/master/src/cmd/vet/main.go) 中，由代码可见，vet 其实是一些 `Analyzer` 的集合。这些 `Analyzer` 在 [golang/tools 仓库](https://github.com/golang/tools/tree/master/go/analysis/passes) 中。



**如何自定义 vet check ？**

大体思路是清晰的，即仿照上述的 tools 仓库中 `go/analysis/passes` 目录下其他 check 的写法来自定义 check。然后如何集成进 go vet 就有两种选择了：

（1）修改 vet main 函数，将 `custom_check.Analyzer` 加入 `unitchecker.Main` 的入参，然后打包出定制后的 `go vet`

（2）将自定义 check 打包成可执行命令，处理方式和代码结构可以参考 `go/analysis/passes/shadow` 目录。然后执行 `go install` 安装这个 check 到 `GOBIN` 目录下。这时候就能用 `go vet -vettool=$(which custom_check)` 来调用该 check 了

显然，方法 2 更好，因为不用修改原生的 vet 命令。



实现 vet check，关键是编写 `func run(pass *analysis.Pass) (interface{}, error)` 函数，这里的细节可以参见：https://godoc.org/golang.org/x/tools/go/analysis



---

## golangci-lint linter


golangci-lint 是一款业界流行的开源 lint 工具，是一系列 linter 的集合。它同样支持自定义 linter。对于 [private linter](https://golangci-lint.run/contributing/new-linters/#how-to-add-a-private-linter-to-golangci-lint)，即不适合开源或者针对某些特定问题的，需要通过插件来完成。

编写 linter 和 go vet check 其实差别不大，都是借助于 `golang.org/x/tools` 库（具体说，是其中的 `go/analysis`）。如果已经完成了 go vet check，再编写 linter，代码基本上可以复制过来用。

编写 linter 时踩了一些坑：

1. golangci-lint 和 linter 都必须设置环境变量 `CGO_ENABLED=1` 构建，linter plugin 才能被正常加载。因为 golangci-lint 的官方 release 都是基于 `CGO_ENABLED=0` 构建的，所以使用 release 的 binary 并不能 work。必须 clone golangci-lint 的代码，自行构建出 golangci-lint binary 使用。构建方法是在代码根目录下执行 `make build`。关于这点的讨论见 [issue1276](https://github.com/golangci/golangci-lint/issues/1276)。
2. linter 的依赖中如果和 golangci-lint 的依赖有重叠，**必须**和后者的依赖版本保持一致，否则无法加载成功。
3. 运行 golangci-lint 时，除了要按官方文档所述添加 `linters-settings:custom` 配置外，还**必须显式**地激活自定义 linter 才行。这和官方文档相悖，所以我提了个 [issue](https://github.com/golangci/golangci-lint/issues/1335)，看看后续是否有维护者回应，不过问题也不大。

`.golangci-lint.yml` 示例：

```yaml
linters:
  enable:
    - example

linters-settings:
 custom:
  example:
   path: ./example.so
   description: The description of the linter
   # original-url: github.com/golangci/example-linter
```

注意，这里的 `path` 可以是绝对路径，也可以是相对于执行 `golangci-lint` 命令时所在目录的路径。

linter 的目录结构可以参考 [example-plugin-linter](https://github.com/golangci/example-plugin-linter) 和 [logwercase](https://github.com/alexisvisco/logwercase)。



---

**Refs**

* [Go: Vet Command Is More Powerful Than You Think](https://medium.com/a-journey-with-go/go-vet-command-is-more-powerful-than-you-think-563e9fdec2f5)
* 推荐！[Writing Useful go/analysis Linter](https://disaev.me/p/writing-useful-go-analysis-linter/)