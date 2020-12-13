---
layout: single
title:  "[笔记] Practical Go: Real world advice for writing maintainable Go programs"
date:   2020-12-13 21:00:00 +0800
categories: go
tags: [go]
toc: true
toc_sticky: true
toc_icon: "fish"
---

[原文链接](https://dave.cheney.net/practical-go/presentations/qcon-china.html)

# 1. Guiding principles
Go 语言的三大优点：Simplicity，Readability，Productivity

## 1.1 Simplicity
简洁性是可靠性的前提。
编写简单的、看起来显然没问题的软件要比编写复杂的、看起来没有明显问题的软件难得多。

## 1.2 Readability
代码是写出来给人看的，是程序员们用来交流的媒介，以你可读性非常重要。
可读性是可维护性的必要条件。

## 1.3 Productivity
Go 有很多带来高生产率的特点：
* 编译快速
* 自动format
* 无需编写和维护复杂的构建脚本
* 部署方便

# 2. Identifiers
标识符，可以理解为名字：变量名、函数名、类型名、包名、等等。
差的名字往往意味着差的设计。

## 2.1 Choose identifiers for clarity, not brevity
简短的名字不一定好，关键是要表达清楚意思。

好名字的特点：
* **concise**：没有冗余信息，信噪比高
* **descriptive**：名字应该描述变量的应用场景，函数或者方法的行为和结果，而不是它们的内容和具体实现
* **predictable**：通过名字我们要能够推断出它的使用方式

## 2.2 Identifier length
标识符的长度并不是越长或越短越好，要选择正确的长度，这与标识符的**声明**和**使用**的位置相距远近有关。

下面是一些命名指导：
* 当标识符的声明和最后一次使用的代码距离比较短时，使用短的名字就好
* 长的名字要能匹配上其携带的信息量
* 变量名中不要有类型
* 常量名应该描述它持有的值，而不是这个值的使用方式
* 偏好：循环和分支里的变量使用**单字母名称**；参数和返回值使用**单个单词名称**；函数和包级别的变量使用**多个单词的名称**
* 偏好：使用单个单词声明方法、接口和包
* 包被调用时，它的名字会被使用，要利用好这一点

### 2.2.1 Context is key
一个命名原则是，要结合上下文来分析。同样的名字也许在函数参数里适用，却在循环体里不合适。

## 2.3 Don't name your variables for their types
作者建议：避免将类型作为变量名的后缀，比如 `var usersMap map[string]*User` 就不是一个好名字，用 `users` 足够。


## 2.4 Use a consistent naming style

在一个代码库中，对同样类型或者表达同一意思的变量应该使用统一的命名，比如 `*sql.DB` 就保持用 `db` 作名字，不要这里用`db`，那里用`d`, `dbase` 等。

## 2.5 Use a consistent declaration style

变量声明的方式应该统一。

* 当声明变量，但不初始化时，用 `var`
* 当声明并初始化变量时，用 `:=`
  
## 2.6 Be a team player

* 保持和团队风格一致更重要
* 如果风格符合 gofmt 规范，那么就没必要对风格做 code review


# 3. Comments

每条注释应当做，并且只做下面三件中的一件事情：
* what：解释做了什么
* how：解释怎么做的
* why：解释为什么要这么做

## 3.1 Comments on variables and constants should describe their contents not their purpose

变量和常量的命名用来描述 purpose，注释用来描述 content。

对于没有初始值的变量，注释还应该描述哪块代码，比如某个函数，负责将其初始化。

## 3.2 Always document public symbols

给每一个公开的变量，常量，函数，方法添加注释，这些都会体现在包的文档中。

有一个例外是，不需要给实现接口的方法添加注释。

作者建议：在编写函数时，尝试先写注释。如果觉得写注释很困难，那这可能意味着将要写的代码也很难懂。

### 3.2.1 Don’t comment bad code, rewrite it
不一定要立刻重写，可以先记个 TODO，作为技术债的一部分，后面再还债。关键是不要忘记。

### 3.2.2 Rather than commenting a block of code, refactor it

当要添加注释时，不妨先问问自己，有没有办法改善这段代码使得其不需要注释呢？
一个函数应该做一件事，如果要为函数中的一段代码添加注释时，可以考虑将其抽出为一个单独的函数。

# 4. Package Design

## 4.1 A good package starts with its name
给包命名时，思考这样一个问题：“这个包提供了什么服务？”而不是“这个包里有哪些类型？”。 这有助于取一个好的包名。

### 4.1.1 Good package names should be unique
如果发现两个包重名，这表明：
1. 包名太通用化了
2. 两个包的功能有重叠，可能要重新设计或者把两个包合并

## 4.2 Avoid package names like `base`, `common`, or `util`

这种包名提供的信息太少。
对于 `util` 包，可以将不同类别的工具函数移到不同的包里，比如和字符串处理相关的函数放在 `strings` 包中。
对于 `common` 和 `base` 包，考虑将这些包和引用它们的包，例如 client，server，合并为一个包。

## 4.3 Return early rather than nesting deeply
使用卫语句先处理异常、边界情形，将 happy path 放到最后。

## 4.4 Make the zero value useful
没有被初始化的变量的值，是它的类型的零值。
要让零值也变得有用。
这样一来，声明好的变量，无需初始化，就能使用了。这方面的例子有：`sync.Mutex`, `bytes.Buffer`, `slice` 等。


## 4.5 Avoid package level state

非常不建议使用包级别的变量。这种变量维护着全局可见的状态。这会加大耦合：当变量的类型发生改变，或者程序修改了变量值时，其他依赖于该变量的值和类型的函数都会被破坏。

改进办法是，将这类变量纳入相关的结构体中，作为一个域。

# 5. Project Structure

## 5.1 Consider fewer, larger packages

避免过度使用包，即创建很多包。太多的包会导致太多的公有类型，进而导致过宽过浅的API设计。

给Java程序员的建议：
* A Java package is equivalent to a single .go source file
* A Go package is equivalent to a whole Maven module

### 5.1.1 Arrange code into files by import statements
对于一个包，什么时候该用一个go文件，什么时候该把这个文件拆分成若干个文件？ 要把握好这个度，作者给出了几条建议：
* 编写一个包时，从单个文件开始，文件名和目录名相同
* 随着包的增大，考虑按照**职责**将单文件拆分成多个文件
* 如果发现几个文件有着相同的引用声明，考虑将它们合并
* 不同的文件是负责所在包的不同领域

tip：文件名用名词。

### 5.1.2 Prefer internal tests to external tests
当给http包编写测试时，测试文件放在包目录下，但是测试文件的包声明有两种选择：
1. 声明为 `package http`，这是 internal test
2. 声明为 `package http_test`，这是 external test

作者倾向于 internal test，理由是这可以对每个函数和方法做测试，即使是私有函数/方法。
注意：Example 测试应该作为 external test。

### 5.1.3 Use `internal` packages to reduce your public API surface

把项目会用到的，但不想暴露给其他项目的包放在 internal 目录下。

## 5.2 Keep package main small as small as possible

main 包和 main 函数都要尽可能简单，做的事情少一些。

# 6. API Design

这一部分作者认为是**最重要的**，因为前几部分可以视为是具体实现层面的建议，并没有强制性，而 API 设计如果没做好的话，修改起来非常困难，会破坏兼容性。

## 6.1 Design APIs that are hard to misuse

> APIs should be easy to use and hard to misuse. — Josh Bloch

### 6.1.1 Be wary of functions which take several parameters of the same type

要当心那些有若干个相同类型参数的函数，这些函数被错误使用的几率较大，而且在 code review 时不容易被发现。

## 6.2 Design APIs for their default use case

针对常见的use case设计API，在这类使用场景下，不应该要求API的调用者提供他们不关心的参数。

### 6.2.1 Discourage the use of nil as a parameter

避免使用 nil 作为函数参数，这可能会导致调用者的误解。

### 6.2.2 Prefer var args to []T parameters

比起 slice 作为参数，更推荐使用可变长参数：
* slice作为参数时：很多情况下只需要一个元素，也得包装成一个slice，然后传递给函数使用，不方便
* slice作为参数时：传入空slice或者nil也能通过编译，但可能会得到意想不到的结果。这需要编写额外的检查逻辑和测试
* 可变长参数可以在编译时防止传递空参数，例如 `func anyPositive(first int, rest ...int) bool`

## 6.3 Let functions define the behaviour they requires

“面向接口编程”的又一个范例。

例如：编写一个函数用来将文档内容写入文件，`func Save(w io.Writer, doc *Document) error` 函数就比 `func Save(f *os.File, doc *Document) error` 好得多，可扩展，可测试。

# 7. Error handling

## 7.1 Eliminate error handling by eliminating errors

通过消除error来消除error处理。 --> 修改代码使得没有error可处理。

### 7.1.1 Counting lines

一个读文件的例子，使用 `bufio.NewScanner` 替代 `bufio.NewReader` 来避免循环中进行错误判断。

### 7.1.2 WriteResponse


## 7.2 Only handle an error once

对于每个错误，需要处理，并只处理一次。

### 7.2.1 Adding context to errors

对调用函数返回的error，使用 `fmt.Errorf("sth happend: %v", err)` 加上上下文信息再返回给上层函数。这有助于最终的错误信息展示。

### 7.2.2 Wrapping errors with github.com/pkg/errors

上面的做法有一个缺点：会掩盖掉原始 error 的类型。当然，这在多数情况下无关紧要。
多数情况是指，我们对于error的值往往只会做这些操作：
1. 检查是不是 nil
2. 打印或者记录日志

但也有一些情况，需要获得原始 error，这里就推荐使用作者编写的 `github.com/pkg/errors` 包。
`errors.Wrap` 包装error，`errors.Cause` 获得原始error。


# 8. Concurrency

## 8.1 Keep yourself busy or do the work yourself

不要滥用 goroutine。
> if your goroutine cannot make progress until it gets the result from another, oftentimes it is simpler to just do the work yourself rather than to delegate it.

## 8.2 Leave concurrency to the caller

将是否要异步执行函数的决定权给到函数的调用方，而不是在函数本身内部发起协程实现并发。

## 8.3 Never start a goroutine without knowning when it will stop
