---
layout: single
title:  "学习 system-design-primer 的记录"
date:   2021-03-24 00:00:00 +0800
categories: design
tags: [system design]
---

在这里记录下学习[system-design-primer](https://github.com/donnemartin/system-design-primer)的过程，希望不要放弃得太快。


## System design topics: start here

### Step 1: Review the scalability video lecture

* [x] 观看学习视频讲座 [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4)，了解垂直扩展，水平扩展，缓存，负载均衡，数据库复制和分区等概念

### Step 2: Review the scalability article

* [x] 阅读[Scalability系列博客](https://www.lecloud.net/tagged/scalability/chrono)

## Performance vs scalability

如果服务**性能**的增长与**资源**的增加是成比例的，那么服务就是**可扩展**的。通常，提高性能意味着能够做更多的工作，但也可以是能够处理更大的工作单位，比如数据集增长的场景。

* [X] ~~*阅读文章 [A word on scalability](https://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)*~~ [2021-03-25]

<details>
<summary>摘录</summary>
为什么做到scalability那么难？一是因为，它必须要事先考虑，必须在系统架构设计时就得纳入考虑范围，很多算法在低负载、小数据集时表现很好，但是当请求速率增长，数据集增大，或者分布式系统的节点数增加时效率就大幅度下降了。二是因为，一个可扩展的系统很可能会面对异质性（heterogeneity）问题，这意味着系统里部分节点性能更强或者存储容量更大，这可能导致一些对均一性要求高的算法失效或是没法完全利用硬件资源。
</details>

* [ ] 浏览PPT [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)

## Latency vs throughput

**延迟**是指执行某些操作或者计算得到某些结果所需要的**时间**。
**吞吐量**是指在单位时间里能够执行这些操作或者计算的**数量**。
通常，我们的目标是在**可接受的延迟**下获取**最大的吞吐量**。

* [X] ~~*阅读 [Understanding latency vs throughput](https://community.cadence.com/cadence_blogs_8/b/sd/posts/understanding-latency-vs-throughput)*~~ [2021-03-25]

## Availability vs consistency

### CAP 理论

在一个分布式系统中，以下三者只能同时满足两项：

* **一致性**：每个读操作获取到的都是最近写入的结果或者报错
* **可用性**：每个请求都能获取到（不是报错的）响应，但是不保证获取到的是最新的信息
* **分区容错性**：当因网络故障导致任意分区时，系统依然能继续运行

网络是不可靠的，所以总是要保证分区容错性。那么就需要在一致性和可用性之间做出取舍。

CP（一致性和分区容错性）： 等待分区节点的响应可能会导致超时错误。如果业务需求需要原子读写，CP 是一个不错的选择。

AP（可用性和分区容错性）：响应返回的是在任意节点上可用数据的最近版本，可能并不是最新的。当分区问题解决后后，写入（操作）可能需要一些时间来传播。

如果业务需求允许**最终一致性**，或要求系统能够在有外部故障时继续运行，AP 是一个不错的选择。

* [ ] 阅读文章 [CAP theorem revisited](https://robertgreiner.com/cap-theorem-revisited/)
* [X] ~~*阅读文章（非常通俗易懂的解释，推荐） [A plain english introduction to CAP theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem)*~~ [2021-03-25]
* [ ] 常见问题解答 [CAP FAQ](https://github.com/henryr/cap-faq)
* [X] ~~*视频 [The CAP theorem](https://www.youtube.com/watch?v=k-Yaq8AHlFA)*~~ [2021-03-25]

> 使用多地ATM机器的例子来解释CAP理论，还提到了部分可用性的概念。
