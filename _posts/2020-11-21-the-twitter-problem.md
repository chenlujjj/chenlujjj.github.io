---
layout: single
title:  "The Twitter Problem"
date:   2020-11-21 23:20:00 +0800
categories: design 
tags: [system design]
---

以下记录阅读教材 [The Twitter Problem](https://www.hiredintech.com/classrooms/system-design/lesson/67) 的笔记。

## 1、题目描述

> *“Design a simplified version of Twitter where people can post tweets, follow other people and favorite tweets.”*

* 发推特
* 关注别人
* 点赞推特

## 2、理清题意

我们要从题目描述的一句话中挖掘出深层次的题意。这个过程需要和面试官快速互动（发问-获得反馈）。

试着考虑这么一些问题：

* 用户数量
* 每天要发多少推特
* 系统每天承受的请求数有多少
* 每天的点赞数
* 平均每个用户关注多少人（这决定了整张用户图上有多少条边）
* 系统的高可用
* 请求延时要做到多少
* 边界情况，例如有很多粉丝的用户，被点赞很多次的推特

假定在和面试官交流后，得到如下数据作为输入：

* 1000 万用户
* 1000 万条推特/天
* 2000 万次点赞/天 （平均每条推特被点赞两次）
* 1 亿次 HTTP 请求/天
* 20 亿个关注关系（平均每个用户关注了200个其他人）

## 3、上层设计

套路：将架构设计分成两大部分来思考：

1. 如何处理请求
2. 如何存储数据

#### （1）请求处理逻辑

显然系统要处理这些请求：

1. 发推
2. 关注用户
3. 点赞
4. 展示用户和推特的信息

前三种请求是对数据库的写操作，最后一种请求是对数据库的读操作。

这个时候我们可以思考下大致的用户交互流程和界面。



上一节中得到，系统的输入请求数量是 1亿次/天，推算出平均的 QPS 是1150。考虑到请求数量在全天时间范围内不是平均分布的，系统繁忙时要处理的请求QPS应该会达到数千级别。

接下来我们要思考要承受这种级别的负载，需要什么？这是一个复杂的问题，牵涉到诸多变量：

* **请求性质**：对数据库的查询是否轻量，是否需要运行计算密集型的任务
* **具体实现方案**：有些解决方案擅长处理并发场景，内存消耗更少。这需要我们基于日常的知识积累和工作经验，对语言、框架的性质和优劣有所了解。

对于高负载的场景，一个通常的解决方案是做水平扩展，使用 Load Balancer。水平扩展的优点在于可扩展性更强，冗余，弹性，避免单点故障。

> 知识储备：了解常见的 LB 解决方案，如 nginx，HAProxy。

阅读材料：

1. [nginx - HTTP load balancer](https://nginx.org/en/docs/http/load_balancing.html)
2. [HAProxy - TCP/HTTP load balancer](https://www.haproxy.org/)

#### （2）数据存储

##### 数据量计算

1. 1000 万用户的用户信息数据，数据量不大
2. 每天 1000 万条推特，一年就是 36.5 亿条。 假设我们目标定在存储 100 亿条推特数据，每条推特140个字符，每个字符看作是 2 bytes。那么这100亿推特数据存储空间约是 2.8 TB
3. 20 亿条用户关注的关系，每个关注关系由两个用户 ID 组成。假设用 int32 存储用户 ID，那么关注关系的数据存储空间是 16 GB
4. 每天 2000 万条点赞，三年约是 200 亿次。每个点赞由用户ID （int32）和推特ID（int64）组成，即 12 bytes。总计是 240 GB

##### 数据库选型

考虑到用户和推特之间的关系，使用关系型数据库。另一个支撑论据是，很多大公司如推特，脸书也使用 MySQL 数据库来承载住比题目中还要高的多的请求。当然，它们是做了很多优化和调优的，比如 fork 一份代码再改。

阅读材料：

- [How Twitter used to store 250 million tweets a day some years ago](http://highscalability.com/blog/2011/12/19/how-twitter-stores-250-million-tweets-a-day-using-mysql.html)
- [How Facebook made MySql scale](https://gigaom.com/2011/12/06/facebook-shares-some-secrets-on-making-mysql-scale/)



##### 使用缓存

面试官可能会问为什么使用缓存要比直接读数据库好，我们要说出一些点来，比如：读磁盘要比读内存慢得多。

> 知识积累：记住一些速度的数量级，包括机械硬盘，固态硬盘，内存，CPU缓存

我们还需要考虑，数据库表的index选择，大数据量下如何做 partition

## 4、底层细节问题

面试官很可能针对某一方面继续深入发问，要求面试者解答一些设计上的细节问题。

### 数据库 schema

这是一类常见的问题。

对于当前这个小推特系统，很容易想到需要下列表：

* User: id, username, full name, password, desc, created_at, updated_at, ...
* Twitter: id, content, created_at, user_id
* connections: follower_id, followed_id, created_at
* favorites: user_id, twitter_id, created_at

再深入一些，从用户角度想想，用户在使用这套系统时会涉及到哪些数据库操作？据此改进 schma 设计

* 查看用户信息：给 User.username 加上索引
* 查看用户发的推特：给 Twitter.user_id 加上索引
* 查看用户最近一段时间发的推特：Twitter 表的 user_id 和 created_at 组成**联合索引**，并且顺序必须 user_id 在前
* 查看用户关注的人和粉丝：connections 表的 follower_id 和 followee_id 都加上索引
* 查看用户点赞的推特：联表查询，favorites 表的 user_id 和 twitter_id 都加上索引

> 知识积累：设计关系型数据库的 schema

### RESTful API

设计后端暴露给前端的接口，注意要符合 RESTful 规范。

## 5、额外的考量

### 读请求数增加

当读请求数剧增时，系统的瓶颈会出在哪呢？

首先想到的是**数据库**。

1. **replication**：增加数据库的实例数
2. **sharding**：将数据分散在不同的机器上，有利于减轻写操作的压力

> 知识积累：熟悉 scaling 关系型数据库的方法。

然后是 web 应用本身，对应的措施是增加 app 实例数量，将其加入到负载均衡中。

当请求数很高时，负载均衡本身也可能成为瓶颈和单点故障。我们可以使用基于 **DNS 的负载均衡**，将不同域名的请求导向不同的机器。

### 扩展（scaling）数据库

数据量越来越大时，需要考虑做 partition/sharding。

阅读材料：

- [Sharding and IDs at Instagram](http://instagram-engineering.tumblr.com/post/10853187575/sharding-ids-at-instagram)
- [Sharding Postgres at Instagram (video & slides)](http://www.databasesoup.com/2012/04/sharding-postgres-with-instagram.html)
- [Generating unique primary keys at Flickr when sharding>](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)

### 预期外的流量

想象下，当发生某个热点事件时，流量会激增，超出系统的负载能力。

一个典型的解决方案是节点的 **auto-scaling**，一些云厂商如 Amazon 已经提供了这种能力。
