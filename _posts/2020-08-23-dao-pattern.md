---
layout: single
title:  "python 中的 DAO 模式"
date:   2020-08-23 10:20:00 +0800
categories: python
tags: [python, dao]
---

在学习 Redis University 的 RU102PY-Week1 课程时，遇到了 DAO 模式。之前一直以为这种都是 Java 领域的概念，看来是误解了。

[wiki](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1) 中的定义是：
> 数据访问对象（data access object，DAO）是为某种类型的数据库或其他持久性机制提供一个抽象接口的对象。通过映射应用程序对持久层的调用，DAO提供一些特定的数据操作，而无需暴露数据库细节。

回过头来看课程中使用 DAO 的场景。和数据库打交道时需要定义三个类：
* 代表业务领域模型（model）的类，通常只有属性， `getter`，`setter`。类似于 Java 中的 `POJO`
* 定义了和存储层交互的抽象方法或接口的抽象基类，也就是 `DAOBase`。这些接口是和具体存储方式**无关**的。
* 对应于某种具体存储方式，实现了上述接口的具体类，例如 `DAORedis`，`DAOPgsql` 等

> You can think of DAO as a design pattern that separates the interface for querying data from implementations of that interface that are specific to data stores like Redis or Postgres.

两个字，解耦。

借用课程中的图来展示得更清楚一些：
![DAO Pattern Diagram](https://s3.us-east-2.amazonaws.com/assets-university.redislabs.com/ru102py/dao_diagram.png)

注意到：
* Model 层使用了 Python3 的新特性 [`dataclass`](https://docs.python.org/3.8/library/dataclasses.html) 来定义模型，使得代码更简洁。

```python
@dataclass(frozen=True, eq=True)
class Site:
    """A solar power installation."""
    id: int
    capacity: float
    panels: int
    address: str
    city: str
    state: str
    postal_code: str
    coordinate: Union[Coordinate, None] = None
```

* DAO 抽象类继承了 abstract base class (ABC)，并定义了一系列 `abc.abstractmethod`。

```python
class SiteDaoBase(abc.ABC):
    @abc.abstractmethod
    def insert(self, site: Site) -> None:
        pass

    @abc.abstractmethod
    def insert_many(self, *sites: Site) -> None:
        pass

    @abc.abstractmethod
    def find_by_id(self, site_id: int) -> Site:
        pass

    @abc.abstractmethod
    def find_all(self) -> Set[Site]:
        pass 
```

关于 dataclass 和 ABC，值得另开笔记来学习。