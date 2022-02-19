---
layout: single
title:  "了解ClickHouse中的part和partition"
date:   2022-02-19 17:00:00 +0800
categories: database
tags: [ClickHouse]
---

part包含了一张表中部分行的数据。一个part在磁盘上就是一个单独的目录。
part 目录的命名规则：`{partition_name}_{min_block_number}_{max_number_block}_{level}`，例如 `202202_1_6017_451`


`system.parts`表中每一行记录了一个part的信息，主要包括了：
* `partition`：分区名
* `name`：part名
* `part_type`：part存储格式，分为`Wide`（每一列在文件系统中被存储为单独的文件）和`Compact`（所有列被存储为一个文件）两种
* `active`：表示这个数据part是否活跃；不活跃的parts由合并为更大的part后剩余的源part、损坏的数据part等
* `rows`：这个part中包含了多少行
* `min_block_number` 和 `max_block_number`：合并后组成当前part的最小和最大的parts数量
* `level`：合并树的深度，0表示当前part是由数据插入产生的，而不是由其他parts合并而来。
* `path`：part文件的绝对路径
* `data_compressed_bytes`和`data_uncompressed_bytes`：part中压缩和未压缩的数据大小
* ...

parts 是在`INSERT`时产生的，它们被周期性地合并为更大的parts。

每个part目录中都有一个`minmax_{PARTITIONING_KEY_COLUMN}.idx`文件，该文件记录了用作分区key的列的最大和最小值。这些最大最小值会加载到内存中，供 `SELECT` 查询语句使用。


partition是parts的逻辑集合。理想情况下一个partition只有一个part。

partition是在建表语句中用`PARTITION BY expr`语句指定的。

注意：
1. 在大部分情况下不需要partition key，即使需要，大部分情况下也用不着比month更精细的partition
2. partition并不会加速查询
3. 只有相同partition表达式值的parts才会被合并。这意味着**不应该用太精细的partition**（超过一千个分区）。否则，会因为文件系统中的文件数量过多和需要打开的文件描述符过多，导致 `SELECT` 查询效率不佳。
4. 可以使用 `OPTIMIZE` 语句来触发part的合并


参考：
* [system.parts文档](https://clickhouse.com/docs/en/operations/system-tables/parts/)
* [Custom Partitioning Key文档](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/custom-partitioning-key/)
* [DB::Exception: Too many parts (600). Merges are processing significantly slower than inserts  #3174](https://github.com/ClickHouse/ClickHouse/issues/3174#issuecomment-423435071)
* [How to understand part and partition of ClickHouse?](https://stackoverflow.com/questions/60142967/how-to-understand-part-and-partition-of-clickhouse)