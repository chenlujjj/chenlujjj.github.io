---
layout: single
title:  "【翻译】Go 程序诊断"
date:   2023-01-20 22:00:00 +0800
categories: go
tags: [go]
---

原文地址：https://tip.golang.org/doc/diagnostics

## 介绍

Go 生态系统提供了大量 API 和工具来诊断 Go 程序中的逻辑和性能问题。本文总结了可用的工具，并帮助 Go 用户针对他们的特定问题选择合适的工具。

诊断解决方案可分为以下几类：

- **分析（Profiling）**：分析工具分析 Go 程序的复杂性和消耗，例如其内存使用情况和频繁调用的函数，以识别 Go 程序中代价高的部分。
- **跟踪（Tracing）**：跟踪是一种检测代码以分析调用或用户请求的整个生命周期中的延迟的方法。跟踪提供了每个组件对系统整体延迟的贡献程度的概览。跟踪可以跨越多个 Go 进程。
- **调试（Debugging）**：调试允许我们暂停 Go 程序并检查其执行情况。可以通过调试验证程序状态和流程。
- **运行时统计和事件（Runtime statistics and events）**：运行时统计和事件的收集和分析提供了 Go 程序健康状况的高层次概览。指标的峰值/下降有助于我们识别吞吐量、利用率和性能的变化。

注意：**某些诊断工具可能会相互干扰**。例如，精确的内存分析会扭曲 CPU 分析，而 goroutine 阻塞分析会影响调度程序跟踪。单独使用各个工具以获得更精确的信息。

## 分析

分析对于识别高代价或经常调用的代码段很有用。Go 运行时以[pprof 可视化工具](https://github.com/google/pprof/blob/master/doc/README.md)期望的格式 提供 [分析数据](https://tip.golang.org/pkg/runtime/pprof/)。分析数据可以在测试期间通过 `go test` 命令或从 [net/http/pprof](https://tip.golang.org/pkg/net/http/pprof/)包提供的端点收集。用户需要收集分析数据并使用 pprof 工具来过滤和可视化顶级（*译注：原文是 top*）代码路径。 

[runtime/pprof](https://tip.golang.org/pkg/runtime/pprof)包提供的预定义 profile：

- **cpu**：CPU profile 指明程序在哪花费了时间，即主动消耗 CPU 周期（而不是在睡眠或等待 I/O ）。
- **heap**：Heap profile 报告内存分配样本；用于监视当前和历史内存使用情况，并检查内存泄漏。
- **threadcreate**：Thread creation profile 报告导致创建新 OS 线程的程序部分。
- **goroutine**：Goroutine profile 报告所有当前 goroutine 的堆栈跟踪。
- **block**：Block profile 显示 goroutines 阻塞等待同步原语（包括定时器通道）的位置。Block profile 在默认情况下不启用 ；使用`runtime.SetBlockProfileRate`来打开。
- **mutex**：Mutex profile 报告锁的竞争。当你认为 CPU 由于互斥竞争而未得到充分利用时，请使用这个 profile。Mutex profile 在默认情况下不启用，参看`runtime.SetMutexProfileFraction`来打开。

**我可以使用哪些其他分析器来分析 Go 程序？**

在 Linux 上，[perf 工具](https://perf.wiki.kernel.org/index.php/Tutorial) 可用于分析 Go 程序。Perf 可以分析和展开 cgo/SWIG 代码和内核，因此它有助于深入了解 native/内核性能瓶颈。在 macOS 上， 可以使用 [Instruments](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/) 套件分析 Go 程序。

**我可以分析我的生产服务吗？**

是的。在生产环境中分析程序是安全的，但是启用一些分析（例如 CPU 分析）会增加程序消耗。你应该会看到性能下降。可以通过在生产中打开分析器之前测量分析器的开销来估计性能损失。

你可能希望定期分析你的生产服务。特别是在单个进程有很多副本的系统中，周期性地选择一个随机副本是一个安全选项。选择一个生产进程，每 Y 秒分析 X 秒，保存结果以进行可视化和分析；然后定期重复这个过程。可以手动和/或自动审查结果以发现问题。Profile 的收集可能会相互干扰，因此建议一次只收集一个 profile。

**可视化分析数据的最佳方法是什么？**

Go 使用 `[go tool pprof](https://github.com/google/pprof/blob/master/doc/README.md)` 来提供 profile 数据的文本、图形和 [callgrind](http://valgrind.org/docs/manual/cl-manual.html)可视化。 阅读[Profiling Go programs](https://blog.golang.org/profiling-go-programs) 来查看它们在实践中的应用。

![](https://tip.golang.org/images/diagnostics/pprof-text.png) 
以文本形式列出消耗最高的调用。

![](https://tip.golang.org/images/diagnostics/pprof-dot.png)  将消耗最高的调用可视化为图形。

Weblist 视图在 HTML 页面中逐行显示源代码中的高消耗部分。在下面示例中，`runtime.concatstrings` 花费了 530ms，每行代码的消耗也展示在列表里。

![](https://tip.golang.org/images/diagnostics/pprof-weblist.png)  将消耗最高的的调用可视化为 weblist。

另一种可视化 profile 数据的方法是[火焰图](http://www.brendangregg.com/flamegraphs.html)。火焰图允许你在特定的层级路径中移动，因此可以放大/缩小特定的代码部分。[pprof](https://github.com/google/pprof) 支持火焰图。

![](https://tip.golang.org/images/diagnostics/flame.png)  
火焰图提供可视化来发现代价最高的代码路径。

**我是否仅限于内置的 profiles ？**

除了运行时提供的内容之外，Go 用户还可以通过[pprof.Profile](https://tip.golang.org/pkg/runtime/pprof/#Profile)创建他们的自定义 profiles， 并使用现有工具来检查它们。

**我可以在不同的路径和端口上提供 profiler handlers (/debug/pprof/...) 吗？**

是的。默认情况下`net/http/pprof`包将 handlers 注册到默认的 mux，但你也可以自行注册从包导出的 handlers。

例如，以下示例将 `pprof.Profile` handler 注册在 `:7777` 端口的 `/custom_debug_path/profile` 路径上：

```go
package main

import (
    "log"
    "net/http"
    "net/http/pprof"
)

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/custom_debug_path/profile", pprof.Profile)
    log.Fatal(http.ListenAndServe(":7777", mux))
}
```

## 跟踪

跟踪是一种检测代码以分析整个调用链生命周期中的延迟的方法。Go 提供 [golang.org/x/net/trace](https://godoc.org/golang.org/x/net/trace) 包作为每个 Go 节点的最小跟踪后端，并提供带有简单仪表板的最小检测库。Go 还提供了一个执行跟踪器来跟踪一个时间间隔内的运行时事件。

跟踪使我们能够：

- 在 Go 进程中检测和分析应用程序延迟。
- 测量长调用链中特定调用的成本。
- 找出利用率和性能改进项。如果没有跟踪数据，瓶颈并不总是很明显。

在单体系统中，从程序的构建模块中收集诊断数据相对容易。所有模块都在一个进程中并共享公共资源，上报日志、错误和其他诊断信息。一旦你的系统超出了单个进程并开始变得分布式，就很难跟踪从前端 Web 服务器开始到其所有后端的调用，直到将响应返回给用户。这就是分布式跟踪在检测和分析你的生产系统方面发挥重要作用的地方。

分布式跟踪是一种检测代码以分析用户请求整个生命周期中的延迟的方法。当系统是分布式的并且传统的分析和调试工具无法扩展时，你可能希望使用分布式跟踪工具来分析用户请求和 RPC 的性能。

分布式跟踪使我们能够：

- 检测和分析大型系统中应用程序的延迟。
- 跟踪用户请求生命周期内的所有 RPC，并查​​看仅在生产中可见的集成问题。
- 找出可以应用于我们系统的性能改进项。许多瓶颈在收集跟踪数据之前并不明显。

Go 生态系统为每个跟踪系统和后端不可知的库提供各种分布式跟踪库。

**有没有办法自动拦截每个函数调用并创建跟踪？**

Go 不提供自动拦截每个函数调用和创建 trace span 的方法。你需要手动修改代码来创建、结束和标注 span。

**我应该如何在 Go 库中传播 trace headers ？**

你可以在 [`context.Context`](https://tip.golang.org/pkg/context#Context) 中传播 trace 标识符和标签。 业界还没有规范的 trace key 或trace headers 的通用表示。每个 trace 提供者负责在其 Go 库中提供用于传播的工具。

**来自标准库或运行时的其他哪些低级事件可以包含在跟踪中？**

标准库和运行时正在尝试公开几个额外的 API 来通知低级别的内部事件。例如， [`httptrace.ClientTrace`](https://tip.golang.org/pkg/net/http/httptrace#ClientTrace) 提供 API 以跟踪发出请求的生命周期中的低级事件。目前正在致力于从运行时执行跟踪器中获取低级运行时事件，并允许用户定义和记录他们的用户事件。

## 调试

调试是识别程序为何运行不正常的过程。调试器使我们能够了解程序的执行流程和当前状态。有几种调试方式；本节将只关注把调试器附加到程序和 core dump 调试。

Go 用户主要使用以下调试器：

- [Delve](https://github.com/derekparker/delve)：Delve 是 Go 编程语言的调试器。它支持 Go 的运行时概念和内置类型。Delve 正在努力成为一个功能齐全的可靠 Go 程序调试器。
- [GDB](https://tip.golang.org/doc/gdb)：Go 通过标准的 Go 编译器和 Gccgo 提供 GDB 支持。堆栈管理、线程和运行时包含了和 GDB 预期的执行模型有很大不同的方面，即使程序是使用 gccgo 编译的，它们会迷惑调试器。尽管 GDB 可以用来调试 Go 程序，但它并不理想，可能会造成混乱。

**调试器与 Go 程序的配合情况如何？**

`gc` 编译器执行优化，例如函数内联和变量注册。这些优化有时会让使用调试器进行调试变得更加困难。目前正在努力提高为优化的二进制文件生成的 DWARF 信息的质量。在这些改进可用之前，我们建议在构建被调试代码时禁用优化。以下命令构建一个没有编译器优化的包：

$ go build -gcflags=all="-N -l"

（译注：通过 `go tool compile --help` 可知，`-N` 表示 "disable optimizations"， `-l` 表示 "disable inlining"）

作为改进工作的一部分，Go 1.10 引入了一个新的编译器 flag `-dwarflocationlists`。这个 flag 使得编译器添加位置列表，帮助调试器处理优化的二进制文件。以下命令构建了一个具有优化同时也有 DWARF 位置列表的包：

$ go build -gcflags="-dwarflocationlists=true"

**推荐的调试器用户界面是什么？**

尽管 delve 和 gdb 都提供了 CLI，但大多数编辑器和 IDE 都提供了调试专用的用户界面。

**是否可以对 Go 程序进行事后调试？**

core dump 文件包含了正在运行的进程的内存 dump 及其进程状态。它主要用于程序的事后调试，并在程序仍在运行时了解其状态。这两种情况使得调试 core dump 成为事后分析和生产服务的良好诊断辅助工具。可以从 Go 程序中获取 core 文件并使用 delve 或 gdb 进行调试，请参阅 [core dump debugging](https://tip.golang.org/wiki/CoreDumpDebugging) 获取详细指导。

（译注：这个指南的链接已经失效。）

## 运行时统计信息和事件

运行时提供内部事件的统计和报告，让用户可以在运行时级别诊断性能和利用率问题。

用户可以监控这些统计数据，以更好地了解 Go 程序的整体健康状况和性能。一些经常监控的统计数据和状态有：

- [`runtime.ReadMemStats`](https://tip.golang.org/pkg/runtime/#ReadMemStats) 报告与堆分配和垃圾收集相关的指标。内存统计信息对于监视进程消耗了多少内存资源、进程是否可以很好地利用内存以及捕获内存泄漏非常有用。
- [`debug.ReadGCStats`](https://tip.golang.org/pkg/runtime/debug/#ReadGCStats) 读取有关垃圾收集的统计信息。查看有多少资源花费在 GC 暂停上很有用。它还报告垃圾收集器暂停时间线和暂停时间百分位数。
- [`debug.Stack`](https://tip.golang.org/pkg/runtime/debug/#Stack) 返回当前堆栈跟踪。堆栈跟踪有助于查看当前有多少 goroutines 在运行，它们在做什么，以及它们是否被阻塞。
- [`debug.WriteHeapDump`](https://tip.golang.org/pkg/runtime/debug/#WriteHeapDump) 暂停所有 goroutines 的执行，并允许你将 heap dump 到文件中。Heap dump 是 Go 进程内存在给定时间的快照。它包含所有分配的对象以及 goroutines、finalizers 等。
- `[runtime.NumGoroutine](https://tip.golang.org/pkg/runtime#NumGoroutine)` 返回当前 goroutines 的数量。可以监视该值以查看是否使用了足够的 goroutines，或检测 goroutine 泄漏。

### 执行跟踪器

Go 附带一个运行时执行跟踪器来捕获各种运行时事件。调度、系统调用、垃圾收集、堆大小和其他事件由运行时收集，并可通过 go tool trace 进行可视化。执行跟踪器是一种检测延迟和利用率问题的工具。你可以检查 CPU 的使用情况，以及网络或系统调用在什么时候会是 goroutine 抢占的原因。

跟踪器可用于：

- 了解你的 goroutines 是如何执行的。
- 了解一些核心运行时事件，例如 GC 运行。
- 识别不良的并行执行。

但是，它不适合识别热点，例如分析内存或 CPU 使用率过高的原因。首先使用分析（profiling）工具来解决它们。

![](https://tip.golang.org/images/diagnostics/tracer-lock.png)

上图中，go tool trace 可视化显示执行开始是正常的，然后变得串行化。它表明可能存在共享资源的锁竞争，从而导致瓶颈。

请参阅[`go tool trace`](https://tip.golang.org/cmd/trace/) 来收集和分析运行时跟踪。

### GODEBUG

如果设置了相应的 [GODEBUG](https://tip.golang.org/pkg/runtime/#hdr-Environment_Variables) 环境变量，运行时也会发出事件和信息 。

- GODEBUG=gctrace=1 在每次收集时打印垃圾收集器事件，总结收集的内存量和暂停时间。
- GODEBUG=inittrace=1 打印已完成包初始化工作的执行时间和内存分配信息的摘要。
- GODEBUG=schedtrace=X 每 X 毫秒打印一次调度事件。

GODEBUG 环境变量可用于禁止在标准库和运行时中使用指令集扩展。

- GODEBUG=cpu.all=off 禁用所有可选指令集扩展。
- GODEBUG=cpu.*extension*=off 禁止使用来自指定指令集扩展的指令。  
  *extension*是指令集扩展的小写名称，例如*sse41*或*avx*。
