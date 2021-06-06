---
layout: post
title: golang 依赖循环
categories: [golang,依赖循环]
description: golang 依赖循环
keywords: golang,依赖循环
---

循环依赖本质上一个错误的设计，在go里面，直接把它作为了一个编译时的错误。

什么是循环依赖呢？

假如有一个结构体 A 和一个结构体B，A和 B分别在包a和b中, 而且结构体A 依赖 结构体B, 并且结构体B 依赖结构体A, 如果写成这样就会导致 循环依赖 或者说是 循环导入。

没有代码bb什么，直接上代码

这是a的代码

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
这是b的代码
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
正如上面的代码，A需要B，B也需要A，这样就导致了循环导入或者说循环依赖。当你要编译这段代码时，会报错：
``` golang
import cycle not allowed
package my-test/importcicle
    imports my-test/importcicle/a
    imports my-test/importcicle/b
    imports my-test/importcicle/a
```
那怎么解决这种循环依赖，平时写代码，一时兴起，不小心写个循环依赖也是常有的事

通常为了避免这种循环依赖，可以在一个新的package中引入一个接口,比如新建一个package叫x，这个interface中包含有B依赖A结构中所有的方法。
``` golang
package x
type ADoSomethingIntf interface {
  DoSomethingWithA()
}
```
现在修改b包中的代码，导入x, 而不是导入a
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
但是如果仍然需要A的实例，那怎么办呢？

所以将不得不引入另外一个package比如说y，y将同时导入a和b，它创建了A的一个实例并将它传递给b。
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

A依赖于B

B依赖于X

Y依赖于A和B

因此没有循环依赖