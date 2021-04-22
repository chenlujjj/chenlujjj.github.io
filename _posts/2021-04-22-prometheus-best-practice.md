---
layout: single
title:  "PromCon 2017: Best Practices and Beastly Pitfalls - Julius Volz"
date:   2021-04-22 23:00:00 +0800
categories: prometheus
tags: [prometheus, talk]
---

看了演讲[PromCon 2017: Best Practices and Beastly Pitfalls - Julius Volz](https://www.youtube.com/watch?v=_MNYuTNfTb4&list=WL&index=27)，记录一些有用的知识点：

### Instrumentation

（1）选择测量哪些指标是有讲究的，有两个有名的挑选方法：

* "USE Method" (for resources like queues, CPUs, disks...)
Utilization, Saturation, Errors
* "RED Method" (for things like endpoints)
Request rate, Error rate, Duration (distribution)

（2）每个label的组合都代表一条series，所以为了降低基数（cardinality）要尽量避免值的选项非常多的label，例如ip地址，user id 等。

（3）为了便于计算错误率，通常记录总数和失败数作为指标，而不是成功数和失败数。前者只需要做一次除法，后者还要多做一次加法。

### Alert

（1）Always have an up-ness and presence alert per job： 这样一来，指标抓取失败或者没有在配置中的情况也能够报警出来。

（2）for duration 不能配置的太短或者不配，这样很容易触发报警。一般建议是至少5分钟。
但也不能太长，一般最多1小时。

（3）报警规则中要保留有用的label信息，这些label很有可能会在alertmanager的的静默/路由中用到。


### Query

（1）查询表达式中应该总是带上job标签，因为指标名称只有在同一个job下才具有唯一性，带上job标签可以避免指标名冲突，导致查询结果不准确。当然如果有其他标签，能够保证指标名的唯一性也行。

（2）当需要同时使用rate和sum函数时，记住：先rate再sum，不要先sum再rate。因为rate函数可以纠正Counter类型指标因为重置带来的偏差，如果先sum可能会掩盖Counter重置，进而导致计算结果不准确。
