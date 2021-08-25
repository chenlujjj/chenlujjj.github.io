---
layout: single
title:  "了解 Bloom Filter"
date:   2021-08-12 23:00:00 +0800
categories: data-structure
tags: [bloom filter]
---


最近多次看到Bloom Filter这个数据结构，于是搜集了一些资料加以了解，整理如下。

---
Bloom Filter 是一种用于快速判断数据在不在集合中的数据结构。
在我们日常写代码中碰到此类需求可能会不假思索地使用诸如map、set的编程语言内置的数据结构，相比之下，Bloom filter的优势在于它有着比map、set**更低的空间复杂度**，缺点是不能保证百分百准确：
* 说没有，那么一定没有；
* 说有，不一定就有 （false positive）—— 可能有，也可能没有。

所以说，这是一种 **probabilisitc data structure**。

---

设计布隆过滤器时，通常要考虑这些问题：
* bit array 的长度：过短的话，整个数组会很快被填满，此时布隆过滤器就没有作用了，因为判断结果都是“存在”。
* 用几个hash函数：过多或者过少都容易产生误判（false positive），见后续的错误率计算公式；另一方面过多的函数会增大计算成本。
* 如何选择hash函数：**（1）计算速度；（2）函数间是否独立；（3）计算结果是否均匀分布（uniformly distributed）**，常见的有HashMix，fnv族，murmur族哈希函数等。这个[回答](https://stackoverflow.com/a/40343867/8018606)里有对常见函数的详细比较。

---

错误（false positive）率计算[^1][^2]：
假定bit array的长度是m，总共有k个hash函数。
并且假设hash函数选取数组中每个位置的概率是等同的，即均匀分布。

计算错误率的过程如下：
1. 某个bit不被一个hash函数设置为1的概率是：1-1/m
2. 某个bit不被这k个hash函数设置为1的概率是：(1-1/m)^k
3. 若已经插入了n个元素，那么某个bit仍然是0的概率是：(1-1/m)^(k*n)
4. 那么这个bit是1的概率就是：1-(1-1/m)^(k*n)
5. 现在要测试某个元素在不在集合中。发生误判，即该元素实际不在却被判断为在的概率是：(1-(1-1/m)^(k*n)) ^ k

由此公式可得
* m越大，错误率趋近于0；m越小（最小为1），错误率趋近为1。
* 错误率和n成正比。
* 当k=3，n=500，m=5000 时，错误率为1.7%；当k增大到4，其他参数不变时，错误率为1.18%；当k再增大到20，其他参数不变时，错误率为5.4%。所以错误率和k并不是单调的关系，而是存在一个k的最优值使得错误率最低。 推导可得，**k的最优值是 `(m/n)*ln2`**。


有一个[网页计算器](https://hur.st/bloomfilter/)，可以帮助快速计算出在给定元素数量n，hash函数个数k时，要达到某个较小的错误率P，所需要的bit array长度m；或者给定n和m，计算使得P最小的k值。

---

Bloom filter的应用：
* LSM Tree， 例如在 Cassandra数据库中就有使用，可以快速判断某个key在不在SSTable中，从而减少不必要的磁盘IO。
* Chrome浏览器使用布隆过滤器来判断某个URL是不是有风险[^3]。
* medium 使用布隆过滤器来判断用户是否读过某篇文章[^4]

---

Bloom filter 的实现：
* 这里有一个[Java实现](https://github.com/gkcs/Competitive-Programming/blob/master/src/main/java/main/java/course/BloomFilter.java)，但是对其中的位操作过程没太看懂。

---

拓展：
* 基本的布隆过滤器不允许删除元素，而 **counting bloom filter** 考虑了元素删除的场景。简单的说，其原理是将bit array换成了int array，当插入新元素时，会将相应的数组数字加一；而删除元素时则将相应的数组数字减一。见此[视频](https://www.youtube.com/watch?v=em2j7sLhoyI)。

---

参考：
* 介绍性视频 [Bloom Filters Explained by Example](https://www.youtube.com/watch?v=gBygn3cVP80)
* 非常棒的概述文章 [Bloom Filters by Example](https://llimllib.github.io/bloomfilter-tutorial/zh_CN/)，如果想探索一些高级话题，其引用的资料也值得一看。



[^1]:  [What are Bloom Filters? - Hashing](https://www.youtube.com/watch?v=bgzUdBVr5tE)

[^2]: [Bloom Filter Wiki](https://en.wikipedia.org/wiki/Bloom_filter)

[^3]: [Bloom filter for System Design | Bloom filter applications | learn bloom filter easily](https://www.youtube.com/watch?v=Bay3X9PAX5k)

[^4]: [What are Bloom filters?](https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff)
