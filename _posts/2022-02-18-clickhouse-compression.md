---
layout: single
title:  "了解ClickHouse中的数据压缩"
date:   2022-02-18 00:25:00 +0800
categories: database
tags: [ClickHouse]
---

了解ClickHouse中的数据压缩有助于在配置、建表时选择恰当的压缩方法，减少数据占用的磁盘空间，节省资源。

ClickHouse 支持两种通用数据压缩方法：
* LZ4
* ZSTD

两者相比，LZ4更快，但是压缩比要小一些。[Compression in ClickHouse](https://altinitydb.medium.com/compression-in-clickhouse-81ea2049cc2)一文中提供了一份针对样本数据的测试报告。

因此，可以对不同用途的数据使用不同的压缩方式。比如，对于由小的parts组成的热数据使用LZ4压缩来提高查询速度，而对于合并成大parts的历史数据使用ZSTD压缩来降低磁盘占用。

对于MergeTree表引擎可以设置数据压缩方法：
```
<compression>
    <case>
      <min_part_size>...</min_part_size>
      <min_part_size_ratio>...</min_part_size_ratio>
      <method>...</method>
      <level>...</level>
    </case>
    ...
</compression>
```
* `min_part_size`：数据part的最小大小
* `min_part_size_ratio`：数据part大小和表大小的比例
* `method`：压缩方式，可选的值有 lz4，lz4hc，zstd
* `level`：压缩等级，见后文所述。

可以设置多个`<case>`条件块。一个数据part如果匹配到了其中一个条件，那么就会使用对应的压缩方法；如果匹配到了多个条件，那么就使用第一个匹配到的；如果没有匹配到任何条件，那么使用 lz4。


除了全局配置，还可以在建表语句中，使用`CODEC`关键字指定每一列的压缩方法，例如：
```sql
CREATE TABLE codec_example
(
    dt Date CODEC(ZSTD),
    ts DateTime CODEC(LZ4HC),
    float_value Float32 CODEC(NONE),
    double_value Float64 CODEC(LZ4HC(9)),
    value Float32 CODEC(Delta, ZSTD)
)
ENGINE = <Engine>
```
注意最后一行的`CODEC(Delta, ZSTD)`，这表示CODEC可以由多个组合，就像管道一样使用。

ClickHouse支持多种通用和特定用途的编解码器。

通用编解码器有：
* NONE：不压缩
* LZ4：默认使用的无损压缩算法
* LZ4HC[(level)]：LZ4 HC（high compression，高压缩比）算法，压缩等级可以配置。等级可选项是[1, 12]，默认等级是9，推荐范围是[4, 9]
* ZSTD[(level)]：ZSTD 压缩算法，压缩等级同样可配置，可选项范围是[1, 22]，默认值是1

特定用途的编解码器在有某种特点的数据上可以达到更好的效果，包括：
* Delta
* DoubleDelta：适用于单调的Counter型数据
* Gorilla：适用于变化缓慢的浮点数
* T64


参考资料：
* [ClickHouse文档：compression](https://clickhouse.com/docs/en/operations/server-configuration-parameters/settings/#server-settings-compression)

* [ClickHouse文档：Column Compression Codecs](https://clickhouse.com/docs/en/sql-reference/statements/create/table/#codecs)
* [LZ4压缩算法](https://github.com/lz4/lz4)
* [New Encodings to Improve ClickHouse Efficiency](https://altinity.com/blog/2019/7/new-encodings-to-improve-clickhouse) 比较了各种编码方式在不同数据集上的效果，可供参考。