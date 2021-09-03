---
layout: post
title: golang log 中 panic 和 fatal 区别
categories: [golang, panic, fatal]
description: golang log 中 panic 和 fatal 区别
keywords: golang, panic, fatal
---

## os.Exit
在讲两者区别之前我们先看一下 os.Exit () 函数的定义：
``` golang
func Exit(code int)

Exit causes the current program to exit with the given status code.
Conventionally, code zero indicates success, non-zero an error.
The program terminates immediately; deferred functions are not run.
```
注意两点：
- 应用程序马上退出。
- defer 函数不会执行。

## log.Fatal
再来看 log.Fatal 函数定义
``` golang
func Fatal(v ...interface{})

Fatal is equivalent to Print() followed by a call to os.Exit(1).
```

看源代码：`go/src/log/log.go`
``` golang
// Fatal is equivalent to l.Print() followed by a call to os.Exit(1).
func (l *Logger) Fatal(v ...interface{}) {
    l.Output(2, fmt.Sprint(v...))
    os.Exit(1)
}
```

总结起来 log.Fatal 函数完成：
- 打印输出内容
- 退出应用程序
- defer 函数不会执行

和 os.Exit () 相比多了第一步。

## panic
再来看内置函数 panic () 函数定义：
``` golang
// The panic built-in function stops normal execution of the current
// goroutine. When a function F calls panic, normal execution of F stops
// immediately. Any functions whose execution was deferred by F are run in
// the usual way, and then F returns to its caller. To the caller G, the
// invocation of F then behaves like a call to panic, terminating G's
// execution and running any deferred functions. This continues until all
// functions in the executing goroutine have stopped, in reverse order. At
// that point, the program is terminated and the error condition is reported,
// including the value of the argument to panic. This termination sequence
// is called panicking and can be controlled by the built-in function
// recover.
func panic(v interface{})
```
注意几点：
- 1.函数立刻停止执行 (注意是函数本身，不是应用程序停止)
- 2.defer 函数被执行
- 3.返回给调用者 (caller)
- 4.调用者函数假装也收到了一个 panic 函数，从而
-   4.1 立即停止执行当前函数
-   4.2 它 defer 函数被执行
-   4.3 返回给它的调用者 (caller)
- ...(递归重复上述步骤，直到最上层函数)
- 应用程序停止。

### panic 的行为
简单的总结 panic () 就有点类似 java 语言的 exception 的处理，因而 panic 的行为和 java 的 exception 处理行为就非常类似，行为结合 catch，和 final 语句块的处理流程。

下面给几个例子：

**例子 1**：log.Fatal
``` golang
package main

import (
    "log"
)

func foo() {
    defer func () { log.Print("3333")} ()
    log.Fatal("4444")
}

func main() {
    log.Print("1111")
    defer func () { log.Print("2222")} ()
    foo()
    log.Print("9999")
}
```
运行结果：
``` sh
$ go build && ./main
2018/08/20 17:48:44 1111
2018/08/20 17:48:44 4444
```
可见 defer 函数的内容并没有被执行，程序在 log.Fatal (...) 处直接就退出了。

**例子 2**：panic () 函数
``` golang
package main

import (
    "log"
)

func foo() {
    defer func () { log.Print("3333")} ()
    panic("4444")
}

func main() {
    log.Print("1111")
    defer func () { log.Print("2222")} ()
    foo()
    log.Print("9999")
}
```
运行结果：
``` sh
$ go build && ./main
2018/08/20 17:49:28 1111
2018/08/20 17:49:28 3333
2018/08/20 17:49:28 2222
panic: 4444

goroutine 1 [running]:
main.foo()
        /home/.../main.go:9 +0x55
main.main()
        /home/.../main.go:15 +0x82
```

可见所有的 defer 都被调用到了，函数根据父子调用关系所有的 defer 都被调用直到最上层。
当然如果其中某一层函数定义了 recover () 功能，那么 panic 会在那一层函数里面被截获，然后由 recover () 定义如何处理这个 panic，是丢弃，还是向上再抛出。(是不是和 exception 的处理机制一模一样呢？)

## 其他
在使用 gin 框架编写代码时，如果手动实现了异步日志，需要注意勿将 panic 级别日志也异步了，否则会导致整个程序中断，原因是 gin 框架中全局对 panic 进行 recover 的部分是捕捉不到这个 panic 的。