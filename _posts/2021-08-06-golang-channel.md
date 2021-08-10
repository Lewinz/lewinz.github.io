---
layout: post
title: golang channel
categories: [golang, channel]
description: golang channel
keywords: golang, channel
---

## 基本定义
channel 是 Go 语言中的一个核心类型，可以把它看成管道。并发核心单元通过它就可以发送或者接收数据进行通讯，这在一定程度上又进一步降低了编程的难度。channel 是一个数据类型，主要用来解决 go 程的同步问题以及协程之间数据共享（数据传递）的问题。

1. channle 本质上是一个数据结构 ——（队列），数据是先进先出。
2. 具有线程安全机制，多个 go 程访问时，不需要枷锁，也就是说 channel 本身是线程安全的。（写入/写出 数据操作都有互斥锁机制保护，线程安全。）
3. channel 是有类型的，如一个 string 类型的 channel 只能存放 string 类型数据。

## channel 遍历读取
### 简单的读
data:=<-ch   （如果读多次，需要用循环）

``` golang
var ch8 = make(chan int, 6)
 
func mm1() {
	for i := 0; i < 10; i++ {
		ch8 <- 8 * i
	}
 
}
func main() {
	go mm1()
	for i:=0;i<100;i++{
		fmt.Print(<-ch8, "\t")
	}
}
```

注：
1. 写入的次数与读取的次数需要一致（本例是 10）；
2. 如果读的次数多于写的次数会发生：`fatal error: all goroutines are asleep - deadlock!` ，若在 mm1 中对 ch8 进行关闭（执行  close (ch8) ），多于的次数读到的数据为 0（数据默认值）。
3. 读的次数少于写的次数，会读取出次数对应的内容，不会报错。

### 断言方式
if  value, ok := <-ch; ok {

}

``` golang
var ch8 = make(chan int, 6)
 
func mm1() {
	for i := 0; i < 10; i++ {
		ch8 <- 8 * i
	}
	close(ch8)
}

func main() {
	go mm1()
	for {
		if data, ok := <-ch8; ok {
			fmt.Print(data,"\t")
		} else {
			break
		}
	}
}
```

1. 如果写端没有写数据，也没有关闭。<-ch; 会阻塞  ---【重点】
2. 如果写端写数据， value 保存 <-ch 读到的数据。 ok 被设置为 true
3. 如果写端关闭。 value 为数据类型默认值。ok 被设置为 false

### range 方式
for num := range ch {    
}

``` golang
var ch8 = make(chan int, 6)
 
func mm1() {
	for i := 0; i < 10; i++ {
		ch8 <- 8 * i
	}
	close(ch8)
}
func main() {
 
	go mm1()
	for {
		for data := range ch8 {
			fmt.Print(data,"\t")
		}
		break
	}
}
```
注：写完之后一定要关闭（ 执行：close (ch8)  ），否则会出现以下运行结果：

## 总结
特别说明：以上实例都是子 go 程写，主 go 程读。如在子 go 程中写，另一个子 go 程中读，不管哪种方法，都不会出现以上错误问题。（多次实例验证）

总结：通过以上验证，为了保证程序的健壮性，在设计程序时，最好将 channel 的读、写分别在子 go 程中进行。写完数据之后，记得关闭 channel。

补充一点：
1. channel 不像文件一样需要经常去关闭，只有当你确实没有任何发送数据了，或者你想显式的结束 range 循环之类的，才去关闭 channel；
2. 关闭 channel 后，无法向 channel 再发送数据 (引发 panic 错误后导致接收立即返回零值)；
3. 关闭 channel 后，可以继续从 channel 接收数据；
4. 对于 nil channel，无论收发都会被阻塞。