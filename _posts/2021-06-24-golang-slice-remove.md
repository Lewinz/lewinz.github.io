---
layout: post
title: Golang slice 删除元素性能比较
categories: [golang,slice]
description: Golang slice 删除元素性能比较
keywords: golang,slice
---

写了两种对一个 slice 中删除特定元素的方法，并做了性能对比，在这里记录一下。

假设我们的切片有 0 和 1，我们要删除所有的 0，此处有三种方法：

第一种方法：
``` golang
func DeleteSlice(a []int) []int{
	for i := 0; i < len(a); i++ {
		if a[i] == 0 {
			a = append(a[:i], a[i+1:]...)
			i--
		}
	}
	return a
}
```
解释：这里利用常见的方法对 slice 中的元素进行删除，注意删除时，后面的元素前移，i 应该后移一位。

第二种方法：
``` golang
func DeleteSlice1(a []int) []int {
	ret := make([]int, 0, len(a))
	for _, val := range a {
		if val == 1 {
			ret = append(ret, val)
		}
	}
	return ret
}
```
解释：这种方法最容易理解，重新使用一个 slice，将不合理的过滤掉。缺点是需要开辟另一个 slice 的空间，优点是容易理解，而且不对原来的 slice 进行操作。

第三种方法：
``` golang
func DeleteSlice2(a []int) []int{
	j := 0
	for _, val := range a {
		if val == 1 {
			a[j] = val
			j++
		}
	}
	return a[:j]
}
```
解释：这里利用一个 index，记录应该下一个有效元素应该在的位置，遍历所有元素，当遇到有效元素，index 加一，否则不加，最终 index 的位置就是所有有效元素的下一个位置。最后做一个截取就行了。这种方法会对原来的 slice 进行修改。

这里对三种方法做了性能测试，测试代码如下：
``` golang
package main
 
import (
	"testing"
)
 
func handle(data []int) {
	return
}
const N = 100
 
func getSlice()[]int {
	a := []int{}
	for i := 0; i < N; i++ {
		if i % 2 == 0 {
			a = append(a, 0)
		} else {
			a = append(a, 1)
		}
	}
	return a
}
 
func BenchmarkDeleteSlice(b *testing.B) {
	for i := 0; i < b.N; i++ {
		 data := DeleteSlice(getSlice())
		 handle(data)
	}
}
 
func BenchmarkDeleteSlice1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		data := DeleteSlice1(getSlice())
		handle(data)
	}
}
 
func BenchmarkDeleteSlice2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		data := DeleteSlice2(getSlice())
		handle(data)
	}
}
```
测试结果如下（slice 大小为 100）：

![image_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_slice_remove_1.png)

加大 slice 大小进行测试（slice 大小为 10000）：

![image_2](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_slice_remove_2.png)

继续加大（slice 大小为 100000）

![image_3](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_slice_remove_3.png)

slice 大小为 10^6:

![image_4](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_slice_remove_4.png)

可以看出：

第一种方法在 slice 大小比较小时，比第 2、3 种方法慢一倍左右。但是 slice 大小变大时，性能显著下降。

第 2 种方法和第 3 种方法差距基本处于同一量级，但是第 3 种方法稍快一些。但是当 slice 大小增加到 10^6 级别时，第三种方法的优势就显现出来。