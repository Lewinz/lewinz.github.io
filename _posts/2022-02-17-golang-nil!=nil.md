---
layout: post
title: golang nil!=nil 问题
categories: [golang, nil]
description: golang nil!=nil 问题
keywords: golang, nil
---

## 问题一：`err` 为 `nil`，那么 `err != nil` 就一定为 `false` 吗？
上代码
``` golang
type Err struct {
	err string
}

func (e *Err) Error() string {
	return e.err
}

func returnErr() *Err {
	return nil
}

func TestErr(t *testing.T) {
	var err error

	err = returnErr()
	fmt.Println(err, err != nil)
}
```
首先 returnErr() 返回了一个值为 nil 的 *Err，然后赋值给了变量 err，那么 fmt 打印的结果是什么呢？实际上是：
``` shell
<nil> true
```

那为什么值为 nil 的 *Err 不等于 nil 呢？简单说，interface 被两个元素 value 和 type 所表示。只有在 value 和 type 同时为 nil 的时候，判断 interface == nil 才会为 true。而 err = returnErr() 这个过程中，虽然 value 为 nil，但 type 却为 *Err。

那怎么避免这个问题呢，err 变量不再前置声明，代码改为如下所示：
``` golang
type Err struct {
	err string
}

func (e *Err) Error() string {
	return e.err
}

func returnErr() *Err {
	return nil
}

func TestErr(t *testing.T) {
	err := returnErr()
	fmt.Println(err, err != nil)
}
```

这个时候 fmt 打印出来的结果就是：
``` shell
<nil> false
```

## 问题二：err == nil 的问题
先看标准库的一段代码：
``` golang
func (e *AddrError) Error() string {
    if e == nil { // 请注意，为什么这里会判断是否为 nil
        return "<nil>"
    }
    s := e.Err
    if e.Addr != "" {
        s = "address " + e.Addr + ": " + s
    }
    return s
}
```

为什么在方法内还要做一次 e == nil 的判断呢？方法和变量是存储在不同区域的，当我们使用空指针类型的变量调用方法时，该方法还是会被调用，只有在被进行解指针操作时，才会 panic。

``` golang
type Err struct {
	err string
}

func (e *Err) Error() string {
	return e.err
}

func (e *Err) PrintConstant() {
	fmt.Println("print constant")
}

func (e Err) PrintOther() {
	fmt.Println("print other")
}

func TestErr(t *testing.T) {
	var e *Err
	e.PrintConstant()
	e.Error()
	e.PrintOther()
}
```

上面例子中，`PrintConstant` 方法未进行解指针操作，是可以正常打印的；`Error`方法中进行了解指针操作，所以会 panic；`PrintOther` 方法的接受者是 Err 而非 *Err，在调用时就会进行解指针操作，所以也是会 panic 的。
