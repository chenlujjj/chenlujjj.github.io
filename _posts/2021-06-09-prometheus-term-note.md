---
layout: single
title:  "[笔记]Prometheus 术语"
date:   2021-06-09 23:30:00 +0800
categories: monitor
tags: [monitor, prometheus]
---

阅读[Prometheus storage: technical terms for humans](https://valyala.medium.com/prometheus-storage-technical-terms-for-humans-4ab4de6c3d48)记录笔记。

### Time Series

* 一个时间序列由一系列的 `(timestamp, value)`按照时间排序而成。
* 时间戳的精度是毫秒，value是64位浮点数。
* 每个时序都有名称，同时可以有任意数量的标签。
* 如果两个时序的名称或者标签的键值对有任一不同，那么这两个时序也就不同。

### Data Point or Sample

时序中的每个 `(timestamp, value)` 称作一个数据点，或者一个样本。

### Metric Types

Prometheus 中有四种指标类型：

1. Gauge
2. Counter
3. Summary
4. Histogram

### High Cardinality

TSDB 中存储的唯一时序数量叫做基数（cardinality）。
如果基数过大，会导致prometheus消耗更多的机器资源——CPU，RAM和磁盘IO。

延伸阅读：

* [PromCon 2019: Containing your Cardinality](https://promcon.io/2019-munich/slides/containing-your-cardinality.pdf)
* [如何避免高基数](https://www.robustperception.io/cardinality-is-key)

Prometheus 从v2.14.0版本开始在 `/status` 页面上会展示高基数的时序。

### Active time series

如果一个时序最近有被抓取到新数据，那么称它为活跃的。

Prometheus会把活跃时序最近增加的样本放在内存中，所以内存占用和活跃时序数量有很大关系，这点要引起注意。

Prometheus提供了 `prometheus_tsdb_head_series` 指标来暴露活跃时序数量。

网上有人提供了通过活跃时序数量和数据摄入速度来估算内存占用的[方法](https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion)，在部署时可以参考。

### Churn rate

新时序取代旧时序数据的速度叫做 churn rate。高的churn rate会增大基数，所以这也是个负面的指标。

Prometheus 暴露了 `prometheus_tsdb_head_series_created_total` 指标，可以用来估算churn rate：`rate(prometheus_tsdb_head_series_created_total[5m])`。也可以使用 `scrape_series_added` 指标来识别是哪个 target 带来的高churn rate。

### Ingestion rate

数据摄入速度，指的是每秒进入tsdb的数据量，可以用 `scrape_samples_scraped` 指标来计算。

也可以用以下公式做粗略估算：

* `ingestion_rate = targets_count * metrics_per_target / scrape_interval`
* `ingestion_rate = active_time_series / scrape_interval`

### Scrape interval

默认的 scrape interval 是1分钟，也可以在配置文件中设置全局和每个target的scrape interval。

注意：不建议对每个target单独配置scrape interval，因为这会带来不必要的复杂度。

### Retention

数据保留时间，默认是15天，过期数据会被自动删除掉。

最低的数据保留时间是2小时。
