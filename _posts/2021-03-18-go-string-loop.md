---
layout: single
title:  "Go string 循环操作"
date:   2021-03-18 23:00:00 +0800
categories: go
tags: [go, string]
---

在Go中，string可以看作是字节切片，即`[]byte`。
* 字符串的长度 `len(string)` 是相应的字节切片的长度
* 对字符串取索引或者按照索引循环，取的也是其中的字节
* 对字符串按照索引和值做循环，取的是 `rune`
* 记住，`type byte = uint8`, `type rune = int32`。 见 `builtin.go`：

```go
// byte is an alias for uint8 and is equivalent to uint8 in all ways. It is
// used, by convention, to distinguish byte values from 8-bit unsigned
// integer values.
type byte = uint8

// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```


理解下面这段代码的输出：
```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	s := "你好世界"

	fmt.Println(len(s))
	fmt.Println([]byte(s))
	fmt.Println(reflect.TypeOf(s[0]))
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}

	fmt.Println()
	fmt.Println([]rune(s))
	for i, c := range s {
		fmt.Printf("type: %v -- %c (%#U) starts at byte position %d\n", reflect.TypeOf(c), c, c, i)
	}
}


// Output:
// 12
// [228 189 160 229 165 189 228 184 150 231 149 140]
// uint8
// e4 bd a0 e5 a5 bd e4 b8 96 e7 95 8c 
// [20320 22909 19990 30028]
// type: int32 -- 你 (U+4F60 '你') starts at byte position 0
// type: int32 -- 好 (U+597D '好') starts at byte position 3
// type: int32 -- 世 (U+4E16 '世') starts at byte position 6
// type: int32 -- 界 (U+754C '界') starts at byte position 9
```

注：
* `%c` 表示 the character represented by the corresponding Unicode code point
* `%U` 表示	Unicode format

参考：
* [4 basic range loop (for-each) patterns](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/)
