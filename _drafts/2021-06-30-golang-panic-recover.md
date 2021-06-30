---
layout: post
title: golang 中的 panic 与 recover
categories: [golang, panic, recover]
description: golang 中的 panic 与 recover
keywords: golang, panic, recover
---

![golang-panic-recover](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_panic_recover_1.png)

图示为 panic 触发的递归延迟调用
- panic 能够改变程序的控制流，调用 panic 后会立刻停止执行当前函数的剩余代码，并在当前 Goroutine 中递归执行调用方的 defer；
- recover 可以中止 panic 造成的程序崩溃。它是一个只能在 defer 中发挥作用的函数，在其他作用域中调用不会发挥作用；

## 现象
- panic 只会触发当前 Goroutine 的 defer；
- recover 只有在 defer 中调用才会生效；
- panic 允许在 defer 中嵌套多次调用；

### 跨协程失效
现象是 panic 只会触发当前 Goroutine 的延迟函数调用：
``` golang
func main() {
	defer println("in main")
	go func() {
		defer println("in goroutine")
		panic("")
	}()

	time.Sleep(1 * time.Second)
}

$ go run main.go
in goroutine
panic:
...
```

当我们运行这段代码时会发现 main 函数中的 defer 语句并没有执行，执行的只有当前 Goroutine 中的 defer。

defer 关键字对应的 runtime.deferproc 会将延迟调用函数与调用方所在 Goroutine 进行关联。所以当程序发生崩溃时只会调用当前 Goroutine 的延迟调用函数也是非常合理的。

![golang-panic-recover](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/golang_panic_recover_2.png)
图示为 panic 触发当前 Goroutine 的延迟调用

多个 Goroutine 之间没有太多的关联，一个 Goroutine 在 panic 时也不应该执行其他 Goroutine 的延迟函数。

### 失效的崩溃恢复
下面的代码试图在主程序中进行 recover 的调用来阻止 panic 的发生，但我们会发现程序并没有正常退出。
``` golang
func main() {
	defer fmt.Println("in main")
	if err := recover(); err != nil {
		fmt.Println(err)
	}

	panic("unknown err")
}

$ go run main.go
in main
panic: unknown err

goroutine 1 [running]:
main.main()
	...
exit status 2
```

仔细分析一下这个过程就能理解这种现象背后的原因，recover 只有在发生 panic 之后调用才会生效。然而在上面的控制流中，recover 是在 panic 之前调用的，并不满足生效的条件，所以我们需要在 defer 中使用 recover 关键字。

### 嵌套崩溃
Go 语言中的 panic 是可以多次嵌套调用的。一些熟悉 Go 语言的读者很可能也不知道这个知识点，如下所示的代码就展示了如何在 defer 函数中多次调用 panic：

``` golang
func main() {
	defer fmt.Println("in main")
	defer func() {
		defer func() {
			panic("panic again and again")
		}()
		panic("panic again")
	}()

	panic("panic once")
}

$ go run main.go
in main
panic: panic once
	panic: panic again
	panic: panic again and again

goroutine 1 [running]:
...
exit status 2
```
从上述程序输出的结果，我们可以确定程序多次调用 panic 也不会影响 defer 函数的正常执行，所以使用 defer 进行收尾工作一般来说都是安全的。

## 数据结构
panic 关键字在 Go 语言的源代码是由数据结构 runtime._panic 表示的。每当我们调用 panic 都会创建一个如下所示的数据结构存储相关信息：

``` golang
type _panic struct {
	argp      unsafe.Pointer
	arg       interface{}
	link      *_panic
	recovered bool
	aborted   bool
	pc        uintptr
	sp        unsafe.Pointer
	goexit    bool
}
```
- argp 是指向 defer 调用时参数的指针；
- arg 是调用 panic 时传入的参数；
- link 指向了更早调用的 runtime._panic 结构；
- recovered 表示当前 runtime._panic 是否被 recover 恢复；
- aborted 表示当前的 panic 是否被强行终止；

从数据结构中的 link 字段我们就可以推测出以下的结论：panic 函数可以被连续多次调用，它们之间通过 link 可以组成链表。

结构体中的 pc、sp 和 goexit 三个字段都是为了修复 runtime.Goexit 带来的问题引入的 1。runtime.Goexit 能够只结束调用该函数的 Goroutine 而不影响其他的 Goroutine，但是该函数会被 defer 中的 panic 和 recover 取消 2，引入这三个字段就是为了保证该函数的一定会生效。

## 程序崩溃
先介绍分析 panic 函数是终止程序的实现原理。编译器会将关键字 panic 转换成 runtime.gopanic，该函数的执行过程包含以下几个步骤：

- 创建新的 runtime._panic 并添加到所在 Goroutine 的 _panic 链表的最前面；
- 在循环中不断从当前 Goroutine 的 _defer 中链表获取 runtime._defer 并调用 runtime.reflectcall 运行延迟调用函数；
- 调用 runtime.fatalpanic 中止整个程序；

``` golang
func gopanic(e interface{}) {
	gp := getg()
	...
	var p _panic
	p.arg = e
	p.link = gp._panic
	gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

	for {
		d := gp._defer
		if d == nil {
			break
		}

		d._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

		reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))

		d._panic = nil
		d.fn = nil
		gp._defer = d.link

		freedefer(d)
		if p.recovered {
			...
		}
	}

	fatalpanic(gp._panic)
	*(*int)(nil) = 0
}
```

需要注意的是，我们在上述函数中省略了三部分比较重要的代码：
- 恢复程序的 recover 分支中的代码；
- 通过内联优化 defer 调用性能的代码 3；
  * runtime: make defers low-cost through inline code and extra funcdata
- 修复 runtime.Goexit 异常情况的代码；

> Go 语言在 1.14 通过 runtime: ensure that Goexit cannot be aborted by a recursive panic/recover 提交解决了递归 panic 和 recover 与 runtime.Goexit 的冲突。

runtime.fatalpanic 实现了无法被恢复的程序崩溃，它在中止程序之前会通过 runtime.printpanics 打印出全部的 panic 消息以及调用时传入的参数：

``` golang
func fatalpanic(msgs *_panic) {
	pc := getcallerpc()
	sp := getcallersp()
	gp := getg()

	if startpanic_m() && msgs != nil {
		atomic.Xadd(&runningPanicDefers, -1)
		printpanics(msgs)
	}
	if dopanic_m(gp, pc, sp) {
		crash()
	}

	exit(2)
}
```

打印崩溃消息后会调用 runtime.exit 退出当前程序并返回错误码 2，程序的正常退出也是通过 runtime.exit 实现的。

## 崩溃恢复
到这里我们已经掌握了 panic 退出程序的过程，接下来将分析 defer 中的 recover 是如何中止程序崩溃的。编译器会将关键字 recover 转换成 runtime.gorecover：

``` golang
func gorecover(argp uintptr) interface{} {
	gp := getg()
	p := gp._panic
	if p != nil && !p.recovered && argp == uintptr(p.argp) {
		p.recovered = true
		return p.arg
	}
	return nil
}
```

该函数的实现很简单，如果当前 Goroutine 没有调用 panic，那么该函数会直接返回 nil，这也是崩溃恢复在非 defer 中调用会失效的原因。在正常情况下，它会修改 runtime._panic 的 recovered 字段，runtime.gorecover 函数中并不包含恢复程序的逻辑，程序的恢复是由 runtime.gopanic 函数负责的：

``` golang
func gopanic(e interface{}) {
	...

	for {
		// 执行延迟调用函数，可能会设置 p.recovered = true
		...

		pc := d.pc
		sp := unsafe.Pointer(d.sp)

		...
		if p.recovered {
			gp._panic = p.link
			for gp._panic != nil && gp._panic.aborted {
				gp._panic = gp._panic.link
			}
			if gp._panic == nil {
				gp.sig = 0
			}
			gp.sigcode0 = uintptr(sp)
			gp.sigcode1 = pc
			mcall(recovery)
			throw("recovery failed")
		}
	}
	...
}
```

上述这段代码也省略了 defer 的内联优化，它从 runtime._defer 中取出了程序计数器 pc 和栈指针 sp 并调用 runtime.recovery 函数触发 Goroutine 的调度，调度之前会准备好 sp、pc 以及函数的返回值：

``` golang
func recovery(gp *g) {
	sp := gp.sigcode0
	pc := gp.sigcode1

	gp.sched.sp = sp
	gp.sched.pc = pc
	gp.sched.lr = 0
	gp.sched.ret = 1
	gogo(&gp.sched)
}
```

当我们在调用 defer 关键字时，调用时的栈指针 sp 和程序计数器 pc 就已经存储到了 runtime._defer 结构体中，这里的 runtime.gogo 函数会跳回 defer 关键字调用的位置。

runtime.recovery 在调度过程中会将函数的返回值设置成 1。从 runtime.deferproc 的注释中我们会发现，当 runtime.deferproc 函数的返回值是 1 时，编译器生成的代码会直接跳转到调用方函数返回之前并执行 runtime.deferreturn：
``` golang
func deferproc(siz int32, fn *funcval) {
	...
	return0()
}
```

跳转到 runtime.deferreturn 函数之后，程序就已经从 panic 中恢复了并执行正常的逻辑，而 runtime.gorecover 函数也能从 runtime._panic 结构中取出了调用 panic 时传入的 arg 参数并返回给调用方。

## 小结
分析程序的崩溃和恢复过程比较棘手，代码不是特别容易理解。我们在本节的最后还是简单总结一下程序崩溃和恢复的过程：

- 编译器会负责做转换关键字的工作；
  * 将 panic 和 recover 分别转换成 runtime.gopanic 和 runtime.gorecover；
  * 将 defer 转换成 runtime.deferproc 函数；
  * 在调用 defer 的函数末尾调用 runtime.deferreturn 函数；
- 在运行过程中遇到 runtime.gopanic 方法时，会从 Goroutine 的链表依次取出 runtime._defer 结构体并执行；
- 如果调用延迟执行函数时遇到了 runtime.gorecover 就会将 _panic.recovered 标记成 true 并返回 panic 的参数；
  * 在这次调用结束之后，runtime.gopanic 会从 runtime._defer 结构体中取出程序计数器 pc 和栈指针 sp 并调用 runtime.recovery 函数进行恢复程序；
  * runtime.recovery 会根据传入的 pc 和 sp 跳转回 runtime.deferproc；
  * 编译器自动生成的代码会发现 runtime.deferproc 的返回值不为 0，这时会跳回 runtime.deferreturn 并恢复到正常的执行流程；
- 如果没有遇到 runtime.gorecover 就会依次遍历所有的 runtime._defer，并在最后调用 runtime.fatalpanic 中止程序、打印 panic 的参数并返回错误码 2；

分析的过程涉及了很多语言底层的知识，源代码阅读起来也比较晦涩，其中充斥着反常规的控制流程，通过程序计数器来回跳转，不过对于我们理解程序的执行流程还是很有帮助。

## 参考
[博客](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-panic-recover/#fn:3)