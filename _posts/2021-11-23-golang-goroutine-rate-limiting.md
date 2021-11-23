---
layout: post
title: golang 并发两种限速方式
categories: [golang, goroutine, rate-limiting]
description: golang 并发两种限速方式
keywords: golang, goroutine, rate-limiting
---

## 引子
golang 提供了 goroutine 快速实现并发编程，在实际环境中，如果 goroutine 中的代码要消耗大量资源时（CPU、内存、带宽等），我们就需要对程序限速，以防止 goroutine 将资源耗尽。
以下面伪代码为例，看看 goroutine 如何拖垮一台 DB。假设 userList 长度为 10000，先从数据库中查询 userList 中的 user 是否在数据库中存在，存在则忽略，不存在则创建。
``` golang
//不使用goroutine，程序运行时间长，但数据库压力不大
for _,v:=range userList {
    user:=db.user.Get(v.ID)
    if user==nil {
        newUser:=user{ID:v.ID,UserName:v.UserName}
        db.user.Insert(newUser)
    }
}

//使用goroutine，程序运行时间短，但数据库可能被拖垮
for _,v:=range userList {
    u:=v
    go func(){
        user:=db.user.Get(u.ID)
        if user==nil {
            newUser:=user{ID:u.ID,UserName:u.UserName}
            db.user.Insert(newUser)
        }
    }()
}

select{}
```

在示例中，DB 在 1 秒内接收 10000 次读操作，最大还会接受 10000 次写操作，普通的 DB 服务器很难支撑。针对 DB，可以在连接池上做手脚，控制访问 DB 的速度，这里我们讨论两种通用的方法。

## 方案一
在限速时，一种方案是丢弃请求，即请求速度太快时，对后进入的请求直接抛弃。

### 实现
实现逻辑如下：
``` golang
package main

import (
    "sync"
    "time"
)

//LimitRate 限速
type LimitRate struct {
    rate     int
    begin    time.Time
    count    int
    lock     sync.Mutex
}

//Limit Limit
func (l *LimitRate) Limit() bool {
    result := true
    l.lock.Lock()
    //达到每秒速率限制数量，检测记数时间是否大于1秒
    //大于则速率在允许范围内，开始重新记数，返回true
    //小于，则返回false，记数不变
    if l.count == l.rate {
        if time.Now().Sub(l.begin) >= time.Second {
            //速度允许范围内，开始重新记数
            l.begin = time.Now()
            l.count = 0
        } else {
            result = false
        }
    } else {
        //没有达到速率限制数量，记数加1
        l.count++
    }
    l.lock.Unlock()

    return result
}

//SetRate 设置每秒允许的请求数
func (l *LimitRate) SetRate(r int) {
    l.rate = r
    l.begin = time.Now()
}

//GetRate 获取每秒允许的请求数
func (l *LimitRate) GetRate() int {
    return l.rate
}
```
### 测试
下面是测试代码：
``` golang
package main

import (
    "fmt"
)

func main() {
    var wg sync.WaitGroup
    var lr LimitRate
    lr.SetRate(3)
    
    for i:=0;i<10;i++{
        wg.Add(1)
            go func(){
                if lr.Limit() {
                    fmt.Println("Got it!")//显示3次Got it!
                }            
                wg.Done()
            }()
    }
    wg.Wait()
}
```
运行结果

``` shell
Got it!
Got it!
Got it!
```
只显示 3 次 Got it!，说明另外 7 次 Limit 返回的结果为 false。限速成功。

## 方案二
在限速时，另一种方案是等待，即请求速度太快时，后到达的请求等待前面的请求完成后才能运行。这种方案类似一个队列。

### 实现
``` golang
//LimitRate 限速
type LimitRate struct {
    rate       int
    interval   time.Duration
    lastAction time.Time
    lock       sync.Mutex
}

//Limit 限速
package main

import (
    "sync"
    "time"
)

func (l *LimitRate) Limit() bool {
    result := false
    for {
        l.lock.Lock()
        //判断最后一次执行的时间与当前的时间间隔是否大于限速速率
        if time.Now().Sub(l.lastAction) > l.interval {
            l.lastAction = time.Now()
                result = true
            }
        l.lock.Unlock()
        if result {
            return result
        }
        time.Sleep(l.interval)
    }
}

//SetRate 设置Rate
func (l *LimitRate) SetRate(r int) {
    l.rate = r
    l.interval = time.Microsecond * time.Duration(1000*1000/l.Rate)
}

//GetRate 获取Rate
func (l *LimitRate) GetRate() int {
    return l.rate 
}
```
### 测试
``` golang
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    var lr LimitRate
    lr.SetRate(3)
    
    b:=time.Now()
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            if lr.Limit() {
                fmt.Println("Got it!")
            }
            wg.Done()
        }()
    }
    wg.Wait()
    fmt.Println(time.Since(b))
}
```
运行结果
``` shell
Got it!
Got it!
Got it!
Got it!
Got it!
Got it!
Got it!
Got it!
Got it!
Got it!
3.004961704s
```
与方案一不同，显示了 10 次 Got it! 但是运行时间是 3.00496 秒，同样每秒没有超过 3 次。限速成功。

## 改造
回到最初的例子中，我们将限速功能加进去。这里需要注意，我们的例子中，请求是不能被丢弃的，只能排队等待，所以我们使用方案二的限速方法。

``` golang
var lr LimitRate//方案二
//限制每秒运行20次，可以根据实际环境调整限速设置，或者由程序动态调整。
lr.SetRate(20)

//使用goroutine，程序运行时间短，但数据库可能被拖垮
for _,v:=range userList {
    u:=v
    go func(){
        lr.Limit()
        user:=db.user.Get(u.ID)
        if user==nil {
            newUser:=user{ID:u.ID,UserName:u.UserName}
            db.user.Insert(newUser)
        }
    }()
}
select{}
```