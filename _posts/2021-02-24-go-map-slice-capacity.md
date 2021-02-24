---
layout: single
title:  "map 和 slice 声明初始容量"
date:   2021-02-24 23:30:00 +0800
categories: go
tags: [go, slice, map]
---

对于map和slice，如果使用过程中需要做频繁的扩容操作，而这部分代码又是热点代码，那么就有必要对其做性能优化。
一个常见的优化手段是在变量声明时，若事先知道其后续大小，则可指定其容量，避免在扩充过程中频繁的resize或reallocation操作。

编写测试代码可直观看出区别：
```go
// alloc.go
func map_alloc() {
	// m := make(map[int]int)     // 1
	m := make(map[int]int, 100)   // 2

	for i := 0; i < 100; i++ {
		m[i] = i
	}
}

func slice_alloc() {
	// s := make([]int, 0)        // 1
	s := make([]int, 0, 100)      // 2
	for i := 0; i < 100; i++ {
		s = append(s, i)
        fmt.Println("len: ", len(s), "cap: ", cap(s))
	}
}


// alloc_test.go
func BenchmarkMap(b *testing.B) {
	for i := 0; i < b.N; i++ {
		map_alloc()
	}
}

func BenchmarkSlice(b *testing.B) {
	for i := 0; i < b.N; i++ {
		slice_alloc()
	}
}
```

执行 `BenchmarkMap` 测试，两种情况结果如下：
（1）`BenchmarkMap-12    	  185745	      5708 ns/op	    5395 B/op	      16 allocs/op`
（2）`BenchmarkMap-12    	  406057	      2946 ns/op	    2924 B/op	       6 allocs/op`

可见：指定初始化容量的写法速度大约快一倍，分配内存少接近一半。

执行 `BenchmarkSlice` 测试，结果如下：
（1）`BenchmarkSlice-12    	 2624802	       452 ns/op	    2040 B/op	       8 allocs/op`
（2）`BenchmarkSlice-12    	20625380	        49.9 ns/op	       0 B/op	       0 allocs/op`
可见：指定恰当的初始化capacity，执行效率上快了将近10倍，而且没有内存分配操作。

> 为什么 `slice_alloc` 的第一种写法中，需要做8次内存分配操作呢？

如果打印出每次 `append` 后的切片长度和容量，就能看出其容量经历了8次变化：0 -> 1 -> 2 -> 4 -> 8 -> 16 -> 32 -> 64 -> 128。而第二种写法，其容量一直是100，保持不变。


---
参考：[Which is the nicer way to initialize a map?](https://stackoverflow.com/questions/31064688/which-is-the-nicer-way-to-initialize-a-map)
