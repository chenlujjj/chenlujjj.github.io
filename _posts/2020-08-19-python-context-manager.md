---
layout: single
title:  "Python Context Manager"
date:   2020-08-19 22:30:00 +0800
categories: python
tags: [python]
toc: true
toc_sticky: true
toc_icon: "fish"
---

如何管理**资源**是写代码时必须要注意的一个问题。这里的资源常常指的是文件描述符、锁、网络或数据库连接等。在很多语言中，往往通过 `try-catch-finally` 的范式来处理。但是 Python 提供了 `with` 语句来做这类事情，更为简便、优雅：

```python
with open("foo.txt", "w") as f:
    f.write("how are you")
```

藏在强大的 `with` 语句后面的就是 Context Manager 了。通过它，我们也可以给自定义的类或者函数赋予使用 `with` 语句的能力。

（1）类
类需要实现两个“魔法”方法：`__enter__` 和 `__exit__`。`__enter__` 方法中定义了“进入” `with` 语句块时所做的事情，并且返回**资源对象**， `__exit__` 方法中定义了“退出” `with` 语句块时所做的事情，如资源的释放，不需要返回值。

这里有两点要注意：
* `__enter__` 方法的返回值会被用作 `with` 语句中 `as` 后的变量，如果没有 `return` 语句，该变量就是 `None`
* `__exit__` 方法除了 `self` 外，有三个固定参数 `exc_type, exc_value, exc_traceback` 是必不可少的

例子：
```python
class Foo(object):

    def __enter__(self):
        print("enter")
        return self

    def do(self):
        print("work")
    
    def __exit__(self, exc_type, exc_value, exc_traceback):
        print("exit")


with Foo() as f:
    f.do()
```

输出：
```shell
enter
work
exit
```

（2）函数
需要使用内置库 `contextlib`，其中有一个 `contextmanager` 装饰器。
函数要写成生成器的形式，`yield` 一个变量给调用方使用。

例子：
```python
from contextlib import contextmanager

@contextmanager
def bar():
    try:
        b = "hello"
        yield b
    finally:
        print(len(b))


with bar() as b:
    print(b)
```

输出：
```shell
hello
5
```


除了管理资源， context manager 还有其他应用场合。下方资料中的视频就给了一个很有意思的例子：`cd` 到某个目录，做完事情后再回到原来的目录：
```shell
cd /some/dir/
# do some work
cd -
```

我们当然不想每次都去写 `cd -` 这种代码，这个时候 context manager 就可以派上用场：
```python
import os

@contextmanager
def change_dir(destination):
    try:
        cwd = os.getcwd()
        os.chdir(destination)
        yield
    finally:
        os.chdir(cwd)


with change_dir("test"):
    print(os.listdir())

```

cool !

---

资料：
* [blog: python-context-managers-and-with-statement](https://dbader.org/blog/python-context-managers-and-with-statement)
* [Video: Python Tutorial: Context Managers - Efficiently Managing Resources](https://www.youtube.com/watch?v=-aKFBoZpiqA)
* [blog: Python with Context Managers](https://jeffknupp.com/blog/2016/03/07/python-with-context-managers/)