---
layout: single
title:  "Go 中 for range 的使用易错点"
date:   2021-01-20 23:40:00 +0800
categories: go
tags: [go]
---

看下面这段程序：
```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func main() {
	jim := Person{
		Name: "jim",
		Age:  20,
	}
	lily := Person{
		Name: "lily",
		Age:  18,
	}
	tom := Person{
		Name: "tom",
		Age:  21,
	}
	people := []Person{jim, lily, tom}

	namePersonMap := make(map[string]*Person)
	// look at this
	for _, p := range people {
		namePersonMap[p.Name] = &p
	}

	fmt.Println(namePersonMap)
	for k, v := range namePersonMap {
		fmt.Println(k, v)
	}
}
```
程序想做的事情很简单：将Person的切片转化为Person.Name和*Person的映射。
然后程序的输出却大大出乎我意料：
```
map[jim:0xc0000a6020 lily:0xc0000a6020 tom:0xc0000a6020]
jim &{tom 21}
lily &{tom 21}
tom &{tom 21}
```

可以看出，所得map的value都一样，就是for循环中最后一轮迭代用到的Person对象--tom的指针。

**这是为什么呢？**

因为在上述程序的 `for range` 表达式中，用到的`p`并不是`people`切片中的元素本身，而是元素的拷贝。并且`p`会在迭代中重复使用，也就是说：“装载” `p` 变量的内存地址在第一轮迭代中放置了jim的拷贝，在第二轮迭代中放置了lily的拷贝，在第三轮迭代中放置了tom的拷贝。所以，最终得到的`namePersonMap`中的value都是一致的，即为指向“tom的拷贝”的指针。

正确的写法是使用切片的index来做循环，这样就不会对切片中元素做拷贝了：
```go
for i := range people {
    namePersonMap[people[i].Name] = &people[i]
}
```
输出是：
```
map[jim:0xc00019a000 lily:0xc00019a018 tom:0xc00019a030]
lily &{lily 18}
tom &{tom 21}
jim &{jim 20}
```
得到了我们想要的结果。


详细的解释可以参看[官方spec](https://golang.org/ref/spec)的**For statements with range clause**一节。重点是这几句话：
> For each entry it assigns **iteration values** to corresponding **iteration variables** if present and then executes the block.
> The iteration values are assigned to the respective iteration variables as in an assignment statement.
> The iteration variables may be declared by the "range" clause using a form of **short variable declaration (:=)**. In this case their types are set to the types of the respective iteration values and their scope is the block of the "for" statement; **they are re-used in each iteration**. If the iteration variables are declared outside the "for" statement, after execution their values will be those of **the last iteration**.

所以，以后在使用 for range 时，还得多长一个心眼，尤其是这种涉及到指针的操作，一不小心就会造成难以排查的bug。


---

延伸阅读：https://www.ardanlabs.com/blog/2017/06/for-range-semantics.html
文中提到：`for _, p := range people` 是基于 value semantics 的写法，而`for i := range people` 是基于 pointer semantics 的写法。作者建议：

> I want you to decide what semantic each type is using and be **consistent**. The more consistent you are with the semantic for a piece of data, the better off your code base will be.

使用上的不一致，往往容易产生bug。