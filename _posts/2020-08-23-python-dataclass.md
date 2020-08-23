---
layout: single
title:  "python dataclass"
date:   2020-08-23 11:00:00 +0800
categories: python
tags: [python, dataclass]
---

每个 Python 开发者或多或少都写过类似如下的代码：
```python
class Foo:
    def __init__(self, attr_a, attr_b):
        self.attr_a = attr_a
        self.attr_b = attr_b

    def __repr__(self):
        return f"<Foo attr_a: {self.attr_a}, attr_b: {self.attr_b}>"
```

我们为了定义一个只存储状态或者属性的类，就需要写这么多代码，有点说不过去呀。隔壁的 Java 还是有 lombok 的各种注解来辅助开发呢。感觉 Python 没有道理在简洁性上输给 Java ...

为了解放开发者劳动力，基于提案 [PEP 557](https://www.python.org/dev/peps/pep-0557/)，从 **Python 3.7** 开始在内置库中添加了 `dataclasses`。利用装饰器和 type hint，能简化上面的代码：
```python
from dataclasses import dataclass

@dataclass
class Foo:
    attr_a: str
    attr_b: str
```

That's it! 是不是简单明了许多呢？

`dataclass` 装饰器默认地会给被装饰的类添加三个魔法方法：`__init__`，`__repr__`，`__eq__`。它可以接受一些参数，来设定为类添加哪些魔法方法，默认参数是：
`@dataclass(init=True, repr=True, eq=True, order=False, unsafe_hash=False, frozen=False)`

`frozen=True` 表示这个类在初始化完成后，其属性是不可变的，不能再被 `set` 和 `del` 了。如果给上个例子中的 `Foo` 类设置了 `frozen=True`，那么下面这段代码会抛异常：
```python
foo = Foo(1, 2)
foo.attr_a = 10
```

> 注意：这里 `Foo` 的初始化参数是两个 `int`，而不是类定义中的 `str`，这是合法的，因为 type hint 并不会强制类型，Python 总归是一中动态类型语言。


输出：
```shell
> python3.8 data.py                                                                                                 at 00:19:09
Traceback (most recent call last):
  File "data.py", line 9, in <module>
    foo.attr_a = 10
  File "<string>", line 4, in __setattr__
dataclasses.FrozenInstanceError: cannot assign to field 'attr_a'
```
其底层原理是为类添加了 `__setattr__` 和 `__delattr__` 方法，这两个方法在被调用时会抛出 `FrozenInstanceError`。



属性可以有默认值：`attr_b: int = 10`

为了对每个属性有更细粒度的控制，可以使用 `dataclasses.field(*, default=MISSING, default_factory=MISSING, repr=True, hash=None, init=True, compare=True, metadata=None)` 赋值给属性，来设置：是否将该属性纳入 `__init__`，`__repr__` 等。

`dataclasses.asdict(instance, *, dict_factory=dict)`：将对象转化为字典，很有用的一个方法。想想之前在开发 web app 时，为了将 model 对象转化为 http response，经常要给 model 类添加 `to_dict` 方法，比较麻烦且无聊。如今可以考虑用 `asdict` 快速解决了。

有时候需要为初始化的属性执行一些后处理操作，这可以通过自定义 `__post_init__` 方法来实现。该方法会被生成的 `__init__` 方法调用。


---

Refs:
* [Official Doc - Data Classes](https://docs.python.org/3.8/library/dataclasses.html)
* [Real Python - The Ultimate Guide to Data Classes in Python 3.7](https://realpython.com/python-data-classes/)