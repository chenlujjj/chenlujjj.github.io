---
layout: single
title:  "【笔记】了解ClickHouse的稀疏主索引"
date:   2022-04-22 21:50:00 +0800
categories: database
tags: [ClickHouse]
---


ClickHouse官网最近更新了文档，浏览时发现一篇文章对主索引介绍得非常详细，图文并茂，值得阅读学习：[A Practical Introduction to Sparse Primary Indexes in ClickHouse](https://clickhouse.com/docs/en/guides/improving-query-performance/sparse-primary-indexes)

这里记录一些笔记，以供后续查阅回顾。

建表的DDL语句中指定 **PRIMARY KEY** 和 **ORDER BY**。主键和排序键都可以是单列，也可以是多列的元组。当都是单列时，两者必须相同；当排序键是元组时，可以不同，但主键必须是排序键的前缀。

建表语句中的`index_granularity`，默认值为8192，意思是对于每8192行组成的一组数据（称之为“granule”），主索引有一个index entry（称之为“mark”）。


插入的行按照主键排序（**字典序的升序**）存储在磁盘上。

一张表只能有一个主键，因为它定义了磁盘上数据行的排列顺序。当然主键本身可以是一个元组。


主键索引的数据量一般比较小，能够放入内存，查询`system.parts`表的`primary_key_bytes_in_memory`可知。



要让新增的索引对已存在的列生效，需要做`MATERIALIZE INDEX`操作。


granule是数据处理的最小不可分单元，而不是数据行。
granule是一个逻辑概念，而不是对应着物理上的存储。


主键文件 `primary.idx`的内容是一个未被压缩的平铺的数组，其中依次存储了每个granule中的主键的最小值（最后一个granule是存储的最大值）。对应着每个granule，`primary.idx`中都有一个entry，叫做mark。因为它“标记”了一段数据范围。

在查询时，主键被用来选取granule（使用**二分搜索**），这也就是查询的第一阶段 —— granule selection。
在第二阶段 —— data reading中，ClickHouse需要知道定位到的granule的物理位置，而这些信息存储在mark文件中。表中的每一列都有其对应的mark文件，文件名是 `<column>.mrk`。

mark文件内容也是一个未压缩的平铺数组，这个数组长度就是granule的数量，数组的元素包含两部分：
（1）block_offset：在被压缩的列数据文件（即`<column>.bin`文件）中，包含选中granule的block的offset。这个block可能会包含多个granule。
（2）granule_offset：在解压后的block数据中，选中的granule的offset。

这样通过两个offset就能够定位并读取到选中granule的内容，然后做进一步处理。

> 为什么需要mark文件，而不是直接在`primary.idx`中存储granule的物理位置信息？
>
> 回答：因为主键索引要加载到内存中，这是出于节约内存的考虑。



当查询需要对复合索引（compound key）中不是第一个索引的列做过滤时，ClickHouse使用 "generic exclusion search" 算法，它的查询效率和第一个索引的列的基数有关，该列相对过滤条件中的列基数更低时查询效率高。
（详见[Performance issues when filtering on key columns after the first](https://clickhouse.com/docs/en/guides/improving-query-performance/sparse-primary-indexes/#performance-issues-when-filtering-on-key-columns-after-the-first)）



使用多主键（multiple indexes）来加速查询的三种方法：
1. 用不同的主键创建第二张表
2. 在已存在的表上创建物化视图
3. 对已存在的表增加投影（projection）

* 第一种方法，查询需要根据过滤条件指定用哪张表，插入数据时需要显式地插入到两张表中来保证数据的同步。
* 第二种方法，附加的表是隐藏的，数据插入原表时会自动同步到隐藏表，但是查询时还是要指定用哪张表。
* 第三种方法，读写对用户来说都是透明的，数据会自动同步，查询会自动选择合适的表。
