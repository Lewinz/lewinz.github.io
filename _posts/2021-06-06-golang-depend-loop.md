---
layout: post
title: golang 依赖循环
categories: [golang,依赖循环 ]
description: golang 依赖循环
keywords: golang,依赖循环
---

循环依赖本质上一个错误的设计，在 go 里面，直接把它作为了一个编译时的错误。

什么是循环依赖呢？

假如有一个结构体 A 和一个结构体 B，A 和 B 分别在包 a 和 b 中, 而且结构体 A 依赖 结构体 B, 并且结构体 B 依赖结构体 A, 如果写成这样就会导致 循环依赖 或者说是 循环导入。

没有代码 bb 什么，直接上代码

这是 a 的代码

``` golang
package a

import (
    "fmt"
    "my-test/importcicle/b"
)

type A struct {
}

func (a A) DoSomethingWithA() {
    fmt.Println(a)
}

func CreateA() *A {
    a := &A{}
    return a
}

func invokeSomethingFromB() {
    o := b.CreateB()
    o.DoSomethingWithB()
}
```
这是 b 的代码
``` golang
package b

import (
    "fmt"
    "my-test/importcicle/a"
)

type B struct {
}

func (b B) DoSomethingWithB() {
    fmt.Println(b)
}

func CreateB() *B {
    b := &B{}
    return b
}

func invokeSomethingFromA() {
    o := a.CreateA()
    o.DoSomethingWithA()
}
```
正如上面的代码，A 需要 B，B 也需要 A，这样就导致了循环导入或者说循环依赖。当你要编译这段代码时，会报错：
``` golang
import cycle not allowed
package my-test/importcicle
    imports my-test/importcicle/a
    imports my-test/importcicle/b
    imports my-test/importcicle/a
```
那怎么解决这种循环依赖，平时写代码，一时兴起，不小心写个循环依赖也是常有的事

通常为了避免这种循环依赖，可以在一个新的 package 中引入一个接口,比如新建一个 package 叫 x，这个 interface 中包含有 B 依赖 A 结构中所有的方法。
``` golang
package x
type ADoSomethingIntf interface {
  DoSomethingWithA()
}
```
现在修改 b 包中的代码，导入 x, 而不是导入 a
``` golang
import (
    "fmt"
    "my-test/importcicle/x"
)

type B struct {
}

func (b B) DoSomethingWithB() {
    fmt.Println(b)
}

func CreateB() *B {
    b := &B{}
    return b
}

// 注意 这里稍微做了些调整
func invokeSomethingFromA(o x.ADoSomethingIntf) {
    o.DoSomethingWithA()
}
```
但是如果仍然需要 A 的实例，那怎么办呢？

所以将不得不引入另外一个 package 比如说 y，y 将同时导入 a 和 b，它创建了 A 的一个实例并将它传递给 b。
``` golang
import (
    "my-test/importcicle/a"
    "my-test/importcicle/b"
)

func doSomethingWithY() {
    o := &a.A{}
    b.InvokeSomethingFromA(o)
}
```
这就是我解决循环依赖的方式，总结就是：

A 依赖于 B

B 依赖于 X

Y 依赖于 A 和 B

因此没有循环依赖