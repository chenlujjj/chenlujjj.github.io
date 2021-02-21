---
layout: single
title:  "切片append操作的易错点"
date:   2021-02-21 20:00:00 +0800
categories: go
tags: [go, slice]
---

看下面这段程序：
```go
func main() {
	a := []int{1, 2, 3}
	fmt.Println(len(a), cap(a))

	b := append(a, 100)
	fmt.Println(len(b), cap(b))

	c := append(a, 101)
	fmt.Println(len(c), cap(c))
	fmt.Println(a, b, c)
}
```
打印出的a,b,c是`[1 2 3] [1 2 3 100] [1 2 3 101]`，符合预期。
但是如果把a的赋值改成如下形式，结果又是什么呢？
```go
var a []int
for i := 1; i <= 3; i++ {
    a = append(a, i)
}
```
此时，打印出的a,b,c变成了`[1 2 3] [1 2 3 101] [1 2 3 101]`，b和c居然是一样的，出现了bug！这是为何？

注意到两段程序中a,b,c切片的长度和容量分别是：
（1）第一段程序：len(a)=3, cap(a)=3, len(b)=4, cap(b)=6, len(c)=4, cap(c)=6
（2）第二段程序：len(a)=3, cap(a)=4, len(b)=4, cap(b)=4, len(c)=4, cap(c)=4

分析：
（1）在第一段程序中，切片a是用字面量声明的，其长度和容量相等。当执行`append`操作时，都需要开辟内存空间，对切片a复制后，再添加元素。所以切片b和c是不相干的。同时注意到，b和c的容量是a的容量的**两倍**，这正是切片的扩容方法。
（2）在第二段程序中，切片a是通过逐个添加元素得到的，容量比长度大，有继续添加元素的空间。当执行`append`操作时，无需再开辟内存空间，直接对切片a的底层数组做修改即可。所以切片b和c是共用同一个底层数组的（更全面地说，a，b，c都共用同一个底层数组）。执行`b := append(a, 100)`后，底层数组元素为`[1 2 3 100]`；执行`c := append(a, 101)`后，底层数组元素为`[1 2 3 101]`，这样一来，b和c也就一样了。

正确的做法：避免b和c共用底层数组。
```go
func main() {
	var a []int
	for i := 1; i <= 3; i++ {
		a = append(a, i)
	}
	fmt.Println(len(a), cap(a))

	b := append(append([]int{}, a...), 100)
	fmt.Println(len(b), cap(b))

	c := append(append([]int{}, a...), 101)
	fmt.Println(len(c), cap(c))
	fmt.Println(a, b, c)
}
```

补充阅读：[Golang slices gotcha
](https://blog.allegro.tech/2017/07/golang-slices-gotcha.html) 一文中有图形解释，很形象了。