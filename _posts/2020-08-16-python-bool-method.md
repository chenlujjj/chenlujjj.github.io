---
layout: single
title:  "Python boolness method"
date:   2020-08-16 12:00:00 +0800
categories: python
tags: [python]
toc: true
toc_sticky: true
toc_icon: "fish"
---

Python 中有的对象是有 bool 属性的，比如空列表、空字符串、空字典、数字 0 都可以表示为 `False`。

```
>>> bool(0)
False
>>> bool(1)
True
>>> bool({})
False
>>> bool([])
False
>>> bool("")
False
>>> if not []:
...     print("hi")
...
hi
```

隐藏在这后面的就是魔法方法 `__nonzero__` 或者 `__bool__` 的作用。注意：`__nonzero__` 是 Python2 中的方法，这个方法的名称是从 C 语言中借用的，到 Python3 变成了 `__bool__` 方法，更容易记住了。

一般，我们可以在定义 datamodel 类时使用该方法。例如：
```python
class A:
    def __init__(self, val):
        self.val = val
    
    def __bool__(self):
        return self.val > 8

a = A(5)
print(bool(a))

if not a:
    print(222)
```

```shell
# output
False
222
```