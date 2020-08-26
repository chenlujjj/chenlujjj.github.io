---
layout: single
title:  "Python iterable & iterator"
date:   2020-08-26 23:30:00 +0800
categories: python
tags: [python]
---

这里来梳理一下 python 中和 for 循环有关的两个概念：iterable 和 iterator。

## Iterable

Iterable，可以作为形容词也可作为名词使用。当我们说某个对象 obj 是 iterable 时，那它就可以用在 for 循环中：
```python
for item in obj:
    # do sth
```
Python 内置的许多数据结构，包括 `str`, `list`, `tuple`, `set`, `dict` 都是 iterable。

iterable 对象都实现了 `__iter__`方法，该方法返回一个 iterator。也就是说，将 iterable 对象传入内置的 `iter` 函数可以得到相应的 iterator。

```python
>>> iter('foobar')                             # String
<str_iterator object at 0x036E2750>

>>> iter(['foo', 'bar', 'baz'])                # List
<list_iterator object at 0x036E27D0>

>>> iter(('foo', 'bar', 'baz'))                # Tuple
<tuple_iterator object at 0x036E27F0>

>>> iter({'foo', 'bar', 'baz'})                # Set
<set_iterator object at 0x036DEA08>

>>> iter({'foo': 1, 'bar': 2, 'baz': 3})       # Dict
<dict_keyiterator object at 0x036DD990>
```

## Iterator

iterator 对象也都实现了 `__iter__` 方法，且该方法返回值就是它**本身**。
所以，**iterator 同时也是 iterable**，也能用在 for 循环中。

```python
>>> si = iter('foobar')
>>> si
<str_iterator object at 0x1045cd220>
>>> for x in si:
...     print(x)
...
f
o
o
b
a
r
>>> iter(si)
<str_iterator object at 0x1045cd220>
>>> id(si)
4368159264
>>> id(iter(si))
4368159264
```

对于 iterator 可以调用内置的 `next` 方法，不断获取其下一个值，直到抛出 `StopIteration` 异常。

```python
>>> next(si)
'f'
>>> next(si)
'o'
>>> next(si)
'o'
>>> next(si)
'b'
>>> next(si)
'a'
>>> next(si)
'r'
>>> next(si)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

注意：python2 和 3 中的 iterator 有一点不同：
（1）python2 中 iterator 有 `next` 方法，所以还可以这样获取下一个值：
```python
>>> si.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
（2）python3 中 iterator 没有 `next` 方法，只有 `__next__` 方法：
```python
>>> si.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'str_iterator' object has no attribute 'next'
```

## dict 的循环

对 `dict` 对象调用 `iter` 方法，返回 `<dict_keyiterator object>`。所以直接对 `dict` 使用 for 循环，就是不断获取下一个 key。

```python
>>> d = {"a":1, "b": 2}
>>> di = iter(d)
>>> di
<dict_keyiterator object at 0x104604d10>
>>> dir(di)
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__length_hint__', '__lt__', '__ne__', '__new__', '__next__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']
>>> for k in d:
...     print(k)
...
a
b
>>> for k in di:
...     print(k)
...
a
b
```

调用 `dict.keys()` 得到一个 `dict_keys` 对象，这是一个 iterable，但不是 iterator，可用于对 key 做迭代。

```python
>>> dk = d.keys()
>>> dk
dict_keys(['a', 'b'])
>>> type(dk)
<class 'dict_keys'>
>>> dir(dk)
['__and__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__ne__', '__new__', '__or__', '__rand__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__ror__', '__rsub__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__xor__', 'isdisjoint']
>>> next(dk)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'dict_keys' object is not an iterator
>>> for k in dk:
...     print(k)
...
a
b
```

类似的，还有 `dict.values()`，用于对 value 做迭代：

```python
>>> dv = d.values()
>>> type(dv)
<class 'dict_values'>
>>> dv
dict_values([1, 2])
>>> dir(dv)
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']
>>> for v in dv:
...     print(v)
...
1
2
```

当然，最常用的还是 `dict.items()`，用于同时对 key 和 value 做迭代：

```python
>>> ditem = d.items()
>>> ditem
dict_items([('a', 1), ('b', 2)])
>>> type(ditem)
<class 'dict_items'>
>>> d.iteritems()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'dict' object has no attribute 'iteritems'
>>> dir(ditem)
['__and__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__ne__', '__new__', '__or__', '__rand__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__ror__', '__rsub__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__xor__', 'isdisjoint']
>>> for i in ditem:
...     print(i)
...     print(isinstance(i, tuple))
...
('a', 1)
True
('b', 2)
True
```

由上可见，`dict.items()` 中每一个"元素"是一个由 key 和 value 组成的 tuple。所以我们通常这样使用：

```python
>>> for k, v in ditem:
...     print("key: %s, value: %d" % (k, v))
...
key: a, value: 1
key: b, value: 2
```

:warning: 上述关于 `dict` 的实验都是在 python3 环境下。对于 python2， `dict.keys()`, `dict.values()`, `dict.items()` 返回值都是**列表**。而 `dict.iteritems()` 返回的才是一个 iterator。
为防混淆，就不多加阐述了。

---

Ref:
* [Python "for" Loops (Definite Iteration)](https://realpython.com/python-for-loop)