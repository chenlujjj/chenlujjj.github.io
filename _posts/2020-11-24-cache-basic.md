---
layout: single
title:  "Cache 基础知识"
date:   2020-11-24 23:30:00 +0800
categories: design
tags: [cache]
---

## 缓存失效 Cache Invalidation

| 名称 | 释义 |优点 | 缺点|
|:---|:---|:--|:--|
| Write-through cache| 数据同时写入缓存和数据库中|<ul><li>保持缓存和数据库的数据一致性</li><li>允许写入后快速读取</li><li>系统故障如断电时不会丢失数据</li></ul> |写操作的 latency 较高 |
| Write-around cache| 数据绕过缓存直接写入数据库中 | 减轻缓存写的压力 |读取最近写入的数据时，会发生 cache miss，再从数据库中读，这会产生较高的读延迟 |
| Write-back cache| 数据写入缓存后，一次请求就结束了，返回给客户端。对数据库的写操作会后续再执行（比如满足特定条件或者一段时间后） | 对于写操作密集的应用，带来低延迟和高吞吐量 | 系统故障时可能会丢数据 |


## 缓存驱逐策略 Cache eviction policies

常见的策略有：
1. **First In First Out (FIFO)**: The cache evicts the first block accessed first without any regard to how often or how many times it was accessed before.
2. **Last In First Out (LIFO)**: The cache evicts the block accessed most recently first without any regard to how often or how many times it was accessed before.
3. **Least Recently Used (LRU)**: Discards the least recently used items first.
4. **Most Recently Used (MRU)**: Discards, in contrast to LRU, the most recently used items first.
5. **Least Frequently Used (LFU)**: Counts how often an item is needed. Those that are used least often are discarded first.
6. **Random Replacement (RR)**: Randomly selects a candidate item and discards it to make space when necessary.