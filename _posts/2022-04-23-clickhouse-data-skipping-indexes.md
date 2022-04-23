---
layout: single
title:  "【笔记】了解ClickHouse的跳数索引"
date:   2022-04-23 16:00:00 +0800
categories: database
tags: [ClickHouse]
---

原文地址：[Understanding ClickHouse Data Skipping Indexes](https://clickhouse.com/docs/en/guides/improving-query-performance/skipping-indexes)

---

跳数索引这个名字的含义是，这种索引能够帮助数据库在查询时跳过没有匹配值的一段数据。

跳数索引只存在于MergeTree系列表引擎中。

跳数索引的参数：

* 索引名
* 索引表达式
* 类型
* GRANULARITY：每个索引的block由 GRANULARITY 个granules组成

创建跳数索引后，每个data part目录中会相应地增加两个文件：

* `skpidx{index_name}.idx`：包含了索引表达式的值
* `skpidx{index_name}.mrk2`：包含了对应的数据列文件的offset



如果查询语句的WHERE表达式中的过滤条件和跳数索引表达式能够匹配，ClickHouse就会使用该索引来决定block是否需要处理还是可以直接跳过（如果没有被主索引排除的话）。



跳数索引的类型有：

（1）minmax

该索引不需要附加参数，存储了每个block中索引表达式的最小和最大值。如果表达式是一个元组，它会分别存储元组中各个元素的最小最大值。

这种索引适用于那些按值排列的列。它只能用在索引表达式是**标量或者元组**的场景，而对于表达式是数组或者map类型的不起作用。



（2）set

该索引有一个参数，意指单个block中的值集合的max_size。set中包含了block中的所有不重复的值。注意，如果不重复值的数量超过max_size，那么set会是空的。

这种索引适用于每组granules中值是低基数的列。

这种索引的成本、性能依赖于block的基数，如果基数过大，那么查询成本会很大。



（3）Bloom Filter Types

Bloom filter的空间效率高，适用于判断给定值是否属于一个大量值组成的集合中。该类索引能够用于数组和map类型。

注意：由于Bloom filter的false positive特性，该类索引**不能**用在 `!=`， `NOT LIKE`的查询中。

此类索引中又包含三类：

1. `bloom_filter`：有一个参数，表示允许的false positive概率，默认值是0.025
2. `tokenbf_v1`：有三个参数——bloom filter的大小，以字节为单位；哈希函数的数量；哈希函数的种子值。在实践中我们可以使用[Bloom Filter Calculator ](https://hur.st/bloomfilter/)结合待存储的数据特点来选择合适的参数。这个索引只能用于 **String，FixedString，Map**数据类型。token的意思是把字符串按非数字和字母的字符分割开。
3. `ngrambf_v1`：它比`tokenbf_v1`还更多一个参数，表示ngram的大小。



每种类型的跳数索引都有其适用的一系列函数，见[Functions Support](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree/#functions-support)。比如，set和基于Bloom filter的索引都是无序的，所以不能用于range范围查询。而minmax就很适合范围查询了。



跳数索引有两个设置项：

1. `use_skip_indexes`：值是0或1，默认值是1，即使用跳数索引
2. `force_data_skipping_indexes`：值是用逗号分隔的索引名的列表。如果查询没有用到这个/这些索引，会抛出异常。这个设置的意义在于防止一些不使用索引而过度消耗服务端资源的查询。见[force_data_skipping_indices](https://clickhouse.com/docs/en/operations/settings/#settings-force_data_skipping_indices)的例子。



注意：实践中，跳数索引的类型选取和参数设置还是要通过在真实数据上的测试来决定，它的行为有时候单靠思想实验不是那么好预测的。
