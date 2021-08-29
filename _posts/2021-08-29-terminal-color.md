---
layout: single
title:  "怎样在终端打印带颜色的字符"
date:   2021-08-29 17:00:00 +0800
categories: shell
tags: [terminal, shell, color]
---


本文简单总结下怎样在终端打印带颜色的字符串。

其实就是在要打印的内容前加上某些特定的**前缀字符序列**就行。

这些前缀字符序列分为四个部分。

（1）首先是“**转义字符**”，即ASCII字符 `ESC`（十进制表示为27）。
有三种形式，任意一种都能生效。
* `\e`
* `\x1b` 或者 `\x1B` ，这是十六进制表示
* `\033`，这是八进制表示

（2）`[` 字符

（3）选择图形再现（Select Graphic Rendition）参数，简称为**SGR参数**。
主要包括设置字符的前景色、背景色、粗体、斜体、下划线等。
颜色选择有黑、红、绿、黄、蓝、品红、青（蓝绿）和白色，以及对应的亮色。

⚠️ SGR参数可以组合使用，用分号`;`分隔。
具体参数可以查阅文末的WIKI。

（4）`m` 字符

---

举几个例子体会下：

打印蓝色字符

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164248.png)


打印底色为蓝色的字符

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164343.png)

打印字体为蓝色，底色为青色的字符

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164410.png)

打印字体为蓝色，底色为青色，并且加粗的字符

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164437.png)


打印字体为蓝色，底色为青色，并且加粗和斜体的字符

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164512.png)

属性重置

![](https://raw.githubusercontent.com/chenlujjj/imagebed/main/img/20210829164555.png)

---

参考：
* [ANSI转义序列](https://zh.wikipedia.org/wiki/ANSI%E8%BD%AC%E4%B9%89%E5%BA%8F%E5%88%97)
* [How to change the output color of echo in Linux](https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux)
