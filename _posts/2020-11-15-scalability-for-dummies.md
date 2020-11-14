---
layout: single
title:  "Scalability for Dummies 笔记"
date:   2020-11-15 00:40:00 +0800
categories: design 
tags: [system design, scalability]
---

阅读了一个系列博客 [Scalability for Dummies](https://www.lecloud.net/tagged/scalability)，内容比较浅显易懂，讲述了 scalability 的基本概念和实践，适合我这种小白了。

该系列博客分为四个部分：Clones, Database, Cache, Asynchronism。

### 1. Clones
   主要提及负载均衡和多实例部署。强调了一条可扩展性的黄金法则：
   > every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory
   
   即每个实例对于用户都要是无状态的。Session 使用中心化存储，能被所有实例访问，可以是数据库或者是可持久化的缓存（如 Redis）。

### 2. Database
   当数据库成为并发瓶颈时，有两种选择：（1）继续使用 MySQL 数据库，进行一些调优措施，包括主从复制（写主库，读从库，并视需要提高master的内存），分片，denormalization，SQL 调优；（2）使用 可扩展性更好的 NoSQL 数据库，如 MongoDB，CouchDB 等。

### 3. Cache
   这里特指 in-memory cache，例如 Memcached 或 Redis，而非 file-based caching。缓存介于应用和数据存储层之间，使用得当可大大加快服务访问速度。 主要有两种使用方式：（1）缓存数据库查询请求：key = hash(query)，value=result dataset。这种方式的弊端在于处理缓存过期比较困难。试想一下，修改了某块数据后，如何找出能查询出该数据的所有query，并将它们从缓存中删除呢？（2）缓存对象：这是更好的做法。当对象属性改变时，将整个对象从缓存中删除即可。

### 4. Asynchronism
   异步机制用于处理无法即可返回的耗时请求，两种做法：（1）提前处理耗时任务，获得结果后存起来，这样用户请求时就能很快返回了。比如在用户访问量不大的晚间运行定时任务。但这种做法只适用于常规化的、能提前预知的任务。（2）简化的模型是：用户请求 => 任务加入队列 => worker 处理任务 => 任务完成后通知到用户。
