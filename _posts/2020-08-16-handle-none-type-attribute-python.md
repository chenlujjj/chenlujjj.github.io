---
layout: single
title:  "Handle AttributeError of NoneType object"
date:   2020-08-16 12:30:00 +0800
categories: python
tags: [python]
toc: true
toc_sticky: true
toc_icon: "fish"
---

Python 是一种解释型语言，有时在运行的时候会对 `None` 对象获取属性值，从而抛出异常。例如：
```
>>> foo = None
>>> foo.bar
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'NoneType' object has no attribute 'bar'
```

这种情况，有的时候确实是因为程序没写好，这是可以避免的，最简单的手段就是做个判空。但有的时候出于代码简洁的考虑，并不想做很多的 if 判断，那么可以使用 `getattr` 来处理。
```
>>> a = getattr(foo, "bar", None)
>>> a
>>> a is None
True
```