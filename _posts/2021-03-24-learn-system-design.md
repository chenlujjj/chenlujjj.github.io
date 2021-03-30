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

* [X] ~~*阅读文章 [CAP theorem revisited](https://robertgreiner.com/cap-theorem-revisited/)*~~ [2021-03-29]
* [X] ~~*阅读文章（非常通俗易懂的解释，推荐） [A plain english introduction to CAP theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem)*~~ [2021-03-25]
* [X] ~~*常见问题解答 [CAP FAQ](https://github.com/henryr/cap-faq)*~~ [2021-03-29]
* [X] ~~*视频 [The CAP theorem](https://www.youtube.com/watch?v=k-Yaq8AHlFA)*~~ [2021-03-25]

> 使用多地ATM机器的例子来解释CAP理论，还提到了部分可用性的概念。

* [ ] [对CAP理论的简要证明](https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/)

## Consistency patterns

### Weak consistency

在数据写入后，读取操作可能看得到，也可能看不到写入的数据。

memcached 使用的就是这种方式。弱一致性在一些实时应用场景中，如视频通话、实时多人游戏等，工作得很好。

### Eventual consistency

在数据写入后，读取操作**最终**可以看得到写入的数据（往往在若干毫秒内）。数据被**异步复制**。

DNS和邮件系统使用这种方式。最终一致性在高可用系统中工作得很好。

### Strong consistency

在数据写入后，读取操作立刻可以看得到写入的数据。数据被**同步复制**。

文件系统和关系型数据库使用这种方式。强一致性在需要保证**事务**的系统中工作得很好。

延伸阅读

* [ ] [Transactions across data centers](https://snarfed.org/transactions_across_datacenters_io.html)


## Availability patterns

有两种模式来支撑高可用：fail-over（故障切换）和replication（复制）。

### Fail-over

#### Active-passive

工作-备用故障切换：

* 工作机器和待机的备用机器间会发送心跳。当心跳中断时，备用机替代工作机，恢复服务
* 宕机时间是由备用机处在“热”待机状态还是需要从“冷”待机状态启动而决定的
* 只有工作机器处理流量
* 工作-备用故障切换也被称作**主从切换**（master-slave failover）

#### Active-active

双工作切换：

* 两台工作机都处理流量，分摊负载
* 也被称作**主主切换**（master-master failover）

#### failover的缺点

* 增加了更多硬件资源和额外的复杂度
* 如果在写入数据被复制到备用机前，工作机就宕机了，数据会有丢失的风险


### Replication

包括主从复制和主主复制，这在后面的数据库部分会讲到。

### Availability in numbers

可用性通常用“几个9”来描述，例如99.99%就叫做“四个9”，对应一年的宕机时间要小于53min。

## Domain name system

DNS 将域名转换为IP地址。
DNS是分层的。

* NS 记录（域名服务） ─ 指定解析域名或子域名的 DNS 服务器
* MX 记录（邮件交换） ─ 指定接收信息的邮件服务器
* A 记录（地址） ─ 指定域名对应的 IP 地址 （A是指Address）
* CNAME（规范） ─ 将一个域名映射到另一个域名或 CNAME 或 A 记录 （C是指Canonical）


一些DNS服务可以通过几种方式路由流量：

* 加权轮询调度
  * 放置流量进入处于维护中的服务器
  * 在不同大小的集群之间做到平衡
  * A/B测试
* 基于延迟的调度
* 基于地理位置的调度

**延伸阅读**：

* [ ] [DNS architecture](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd197427(v=ws.10)?redirectedfrom=MSDN)
* [ ] 系列文章，可供查阅 [DNS articles](https://support.dnsimple.com/categories/dns/)

## Content delivery network

CDN 由分布在世界各地的代理服务器组成，它从靠近用户的位置提供内容。通常，HTML/CSS/JS，图片和视频等静态内容由 CDN 提供。当然，也有一些CDN能够过支持动态内容。DNS 解析会告知客户端连接哪台服务器。

CDN 可以从以下两方面提高性能:

* 用户从近距离的数据中心获取内容
* 服务器不用处理那些由CDN完成的请求


### Push CDNs

每当你的服务器上有变动时，Push CDN都要接收新的内容。你负责提供内容，上传到CDN，改写URL将其指向CDN。你可以配置内容什么时候过期和什么时候更新。只有新内容或者有变更时，内容才会被上传，所以这种方式最小化了流量，但是最大化了存储。

小流量的站点，或者是内容更新不频繁的站点，适合使用push CDN。（试想一下，对于大流量或者更新频繁的网站，如果还使用push CDN的话，将内容同步到CDN就变成了一份沉重的负担）

### Pull CDNs

当第一个用户请求内容时，Pull CDN从你的服务器上获取新的内容。你改写URL，将其指向CDN。这会导致更慢的请求速度，直到内容在CDN上缓存好了为止。

TTL决定了内容被缓存多久。Pull CDN最小化了存储空间，但是带来了流量的浪费——如果内容过期了，但是实际上没有变化，也要从主服务器上再拉取一次。

大流量的站点适合使用 pull CDN，因为这样一来只有最近被请求的内容才会存放在CDN上，流量更均匀地分配。


**延伸阅读**

* [ ] 论文[Globally distributed content delivery](https://figshare.com/articles/journal_contribution/Globally_distributed_content_delivery/6605972)
* [X] ~~*博客[The Differences Between Push And Pull CDNs](http://www.travelblogadvice.com/technical/the-differences-between-push-and-pull-cdns/)*~~ [2021-03-31]


## Load Balancer


### Layer 4 load balancing

### Layer 7 load balancing

### Horizontal scaling

## Reverse proxy (web server)

### Load balancer vs reverse proxy

## Appllicaton layer

### Microservices

### Service discovery

## Database

### RDBMS

关系型数据库**事务**的特点是 ACID：

* Atomicity 原子性：事务里的操作要么全部完成，要么全部不完成
* Consistency 一致性：任何事务都是将数据库从一个有效状态转换为另一个有效状态
* Isolation 隔离性：事务并发执行和顺序执行，结果都是一样 （事务之间是相互隔离的）
* Durability 持久性：事务提交后，对数据库的修改是永久的，即使系统故障也不会丢失

有多种技术来扩展关系型数据库：**主从复制，主主复制，联邦，分片，非规范化和SQL调优**。

#### Master-slave replication

#### Master-master replication

#### Federation

#### Sharding

#### Denormalization

#### SQL tuning

### NoSQL

#### Key-value store

#### Document store

#### Wide column store

#### Graph Database

### SQL or NoSQL

## Cache

## Asynchronism

## Communication

### TCP

### UDP

### RPC

### REST

## Security

## Appendix
