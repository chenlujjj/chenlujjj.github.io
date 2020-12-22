---
layout: single
title:  "[笔记] Go Anti-Patterns"
date:   2020-12-22 23:00:00 +0800
categories: go
tags: [go]
---

视频地址：[Go Anti-Patterns](https://www.youtube.com/watch?v=ltqV6pDKZD8)

* **Tiny Package Syndrome**: 指很小的包，极端的例子是每个包（目录下）只有一个文件。这太冗余了。
* **Permature Exportation**: 这和上一点“反模式”相关。如果包的层级很深，那么很多变量、类型、函数就必须得定义为 exported 的，而这往往没有必要。
* **package util**: 
    - package names should have **semantic meaning**
    - package names should describe **purpose** of the package, not its contents
    - `util` says nothing

* **Config Structs**: 
  * Increase coupling: 使用 config struct 往往会演变成一个 struct 到处传，带来了不必要的耦合
  * Obfuscate the API of the functions or types that accept them: 读代码的人没法立即看出函数接受了什么具体参数，第一眼只能看到 config struct
  * Hides complexity

* **Pointer all the things**: 
  * Pointers are about **ownership of data**
  * Not necessarily faster
  * Additional overhead
  * 我的理解：并不是所有场景都是使用指针更好，比如有时候用指针作为函数参数，而该函数“恰好“有意无意的对指针指向的值做了改动，而这可能并不是我们预期的，带来了潜在bug。

* **context.Value**: 要谨慎使用 `context.Value()`, `context.WithValue()`, 因为它们没有类型，语义并不清晰
* **Asynchronous APIs**: 
  * synchronous APIs are easier to make asynchronous than vice versa
  * use channels internally, don't expose them

* **If-Then-Else**: 这条是在讲使用卫语句让函数可读性更高
  * handle errors early and return often
  * keep common or happy paths de-dented
  * when it's not possible, refactor and/or redesign

* **Panic in Lib**: 这么做的话，会导致使用库的代码都要做 recover
  * return errors, don't panic
  * only panic when an error can't be handled directly or the handling can't be delegated to the upstream caller
  * mark up errors with `github.com/pkg/errors` to add context before returning them

* **Interface all the things**: 尽管常说要“面向接口编程”，接口有很多好处，包括松耦合，可测试性等，但是在一切能用上接口的地方都添加接口，无疑是走向了另一个极端
  * Go proverb: **The bigger the interface, the weaker the abstraction**
  * 接口是对共性行为的抽象，是对大量使用场景思考、总结后“浮现”出来的。并不是先定义好接口，然后编写实现接口的结构体及其方法，这就有点为了接口而接口了。

* **Naked Return Values**: 不带返回值的 return 语句，这通常是在签名中包含 named return value 的函数中使用。对于小型函数，一般没问题。当函数变大时，就容易出错了，因为函数中声明的同名变量会 shadow 返回值变量。
* **interface{}**:
  * Use the empty interface when dealing with **unknown data**
  * Otherwise try to tease out an interface that declares the behavior you need

