---
layout: single
title:  "了解ClickHouse中的LowCardinality数据类型"
date:   2022-02-19 17:00:00 +0800
categories: database
tags: [ClickHouse]
---


[Low Cardinality 数据类型](https://clickhouse.com/docs/en/sql-reference/data-types/lowcardinality/) 用来“包装”其他的数据类型，语法是`LowCardinality(data_type)`，这里的`data_type`可以是`String`，`FixedString`，`Date`，`DateTime`，除了`Decimal`外的其他数字类型。它本质上是对被包装的`data_type`做了[字典编码](https://en.wikipedia.org/wiki/Dictionary_coder)，使用得当可以显著提升`SELECT`查询性能。

但是它并不适合所有的数据类型。在[allow_suspicious_low_cardinality_types](https://clickhouse.com/docs/en/operations/settings/settings/#allow_suspicious_low_cardinality_types)这一配置项中提到，ClickHouse默认限制了对小于8字节的固定长度数据使用LowCardinality，因为这往往会增大磁盘、内存的占用。


[Reducing ClickHouse Storage Cost with the Low Cardinality Type – Lessons from an Instana Engineer](https://www.instana.com/blog/reducing-clickhouse-storage-cost-with-the-low-cardinality-type-lessons-from-an-instana-engineer/) 指出，使用Low Cardinality数据类型既可以降低磁盘占用，也能够提高查询速度。LowCardinality这个名称其实有一定的误导性——即使该列数据的可能取值数量达到百万量级，性能依然不错。


[A Magical Mystery Tour of the LowCardinality Data Type](https://altinity.com/blog/2019/3/27/low-cardinality) 结合一个具体数据集的例子介绍了LowCardinality对存储和查询性能的好处，并指明了这种数据类型适用的场合：
> It works the best when **source strings are lengthy and the number of distinct values is not that big**. There is no hard limit in ClickHouse, the results are usually good if number of distinct values is below 10M. For big tables with many partitions and parts, the total number of distinct values can even be higher, if 10M limit is preserved at part level.

实际上，ClickHouse是用单独的一个或者多个文件存储LowCardinality的字典。如果一张表里所有LowCardinality列的独特值不超过8192，那么就用一个共同的文件来存储；如果独特值的数量超过这个限制，那么就每个LowCardinality列会用一个文件来存其对应的字典。

我认为这里的8192的限制就是[low_cardinality_max_dictionary_size](https://clickhouse.com/docs/en/operations/settings/settings/#low_cardinality_max_dictionary_size)配置项的默认值。

相对于Enum，LowCardinality的灵活性更好，毕竟Enum需要修改表操作才容许添加新的值选项。


[String Optimization](https://github.com/ClickHouse/clickhouse-presentations/blob/master/meetup19/string_optimization.pdf)这个Slides中介绍了更多实现细节。以`LowCardinality(String)`为例，ClickHouse内部存储了字典、反向索引（key是字符串，value是position）以及用position来代替原始字符串的列。
![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20220221235603.png)
