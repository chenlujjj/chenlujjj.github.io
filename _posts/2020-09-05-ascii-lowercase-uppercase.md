---
layout: single
title:  "大小写字母转换技巧"
date:   2020-09-05 22:45:00 +0800
categories: programming
tags: [ascii, programming]
---

学到一个英文大小写字母间快速转换的小技巧：
（1）大写转换成小写：和 `0x20` 做或运算，即：
```
'A' | 0x20 = 'a'
```
（2）小写转换成大写：和 `0xdf` 做与运算，即：
```
'a' & 0xdf = 'A'
```


解释一下：在 ascii 表中，'A-Z' 是 '0x41-0x5A'，而 'a-z' 是 '0x61-0x7A'，每对大小写字母间都相差了 '0x20'。
所以大写转小写，就是加上 '0x20'，和 `0x20` 做或运算；小写转大写，就是减去 '0x20'，和 `0x20` 取反得到的 `0xdf` 做与运算。

```
'A' | 0x20 = 'a'
'A' | 0x20 & (~0x20) = 'a' & (~0x20)  // 等式两边同时做运算
'A' = 'a' & 0xdf
```

Ref:
* [The oldest trick in the ASCII book](https://blog.cloudflare.com/the-oldest-trick-in-the-ascii-book/)
* [ASCII Programming Tricks](https://c-for-dummies.com/blog/?p=1017)