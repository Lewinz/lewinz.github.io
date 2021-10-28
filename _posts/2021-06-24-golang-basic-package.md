---
layout: post
title: Golang 基础包
categories: [golang, 基础包]
description: Golang 基础包
keywords: golang, 基础包
---

## time 包

golang 时间可分为时间点和时间段
- 时间点 (Time)
- 时间段 (Duration)

除此之外，还提供了其他几种类型
- 时区 (Location)
- Ticker
- Timer (定时器)

### 时间点 (Time)
#### 初始化

``` golang
// func Now() Time
fmt.Println(time.Now())

// func Parse(layout, value string) (Time, error)
time.Parse("2016-01-02 15:04:05", "2018-04-23 12:24:51")

// func ParseInLocation(layout, value string, loc *Location) (Time, error) (layout 已带时区时可直接用 Parse)
time.ParseInLocation("2006-01-02 15:04:05", "2017-05-11 14:06:06", time.Local)

// func Unix(sec int64, nsec int64) Time
time.Unix(1e9, 0)

// func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time
time.Date(2018, 1, 2, 15, 30, 10, 0, time.Local)

// func (t Time) In(loc *Location) Time 当前时间对应指定时区的时间
loc, _ := time.LoadLocation("America/Los_Angeles")
fmt.Println(time.Now().In(loc))

// func (t Time) Local() Time
t := time.Now().Local()
```

#### 格式化
##### to string

格式化为字符串我们需要使用 `time.Format` 方法来转换成我们想要的格式
``` golang
fmt.Println(time.Now().Format("2006-01-02 15:04:05"))  // 2018-04-24 10:11:20
fmt.Println(time.Now().Format(time.UnixDate))         // Tue Apr 24 09:59:02 CST 2018
```

`Format` 函数中可以指定你想使用的格式，同时 `time` 包中也给了一些我们常用的格式
``` golang
const (
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
)
```

** 注意 **: `galang` 中指定的特定时间格式为 `"2006-01-02 15:04:05 -0700 MST"`， 为了记忆方便，按照美式时间格式 月日时分秒年 外加时区 排列起来依次是 01/02 03:04:05PM ‘06 -0700，刚开始使用时需要注意。

##### to time stamp
``` golang
func (t Time) Unix() int64
func (t Time) UnixNano() int64

fmt.Println(time.Now().Unix())

// 获取指定日期的时间戳
dt, _ := time.Parse("2016-01-02 15:04:05", "2018-04-23 12:24:51")
fmt.Println(dt.Unix())

fmt.Println(time.Date(2018, 1,2,15,30,10,0, time.Local).Unix())
```

#### 其他
`time` 包还提供了一些常用的方法，基本覆盖了大多数业务，从方法名就能知道代表的含义就不一一说明了。

``` golang
func (t Time) Date() (year int, month Month, day int)
func (t Time) Clock() (hour, min, sec int)
func (t Time) Year() int
func (t Time) Month() Month
func (t Time) Day() int
func (t Time) Hour() int
func (t Time) Minute() int
func (t Time) Second() int
func (t Time) Nanosecond() int
func (t Time) YearDay() int
func (t Time) Weekday() Weekday
func (t Time) ISOWeek() (year, week int)
func (t Time) IsZero() bool
func (t Time) Local() Time
func (t Time) Location() *Location
func (t Time) Zone() (name string, offset int)
func (t Time) Unix() int64
```

### 时间段 (Duartion)
介绍完了时间点，我们再来介绍时间段，即 `Duartion` 类型， 我们业务也是很常用的类型。

``` golang
// func ParseDuration(s string) (Duration, error)
tp, _ := time.ParseDuration("1.5s")
fmt.Println(tp.Truncate(1000), tp.Seconds(), tp.Nanoseconds())

func (d Duration) Hours() float64
func (d Duration) Minutes() float64
func (d Duration) Seconds() float64
func (d Duration) Nanoseconds() int64
func (d Duration) Round(m Duration) Duration         // 四舍五入
func (d Duration) Truncate(m Duration) Duration      // 向下取整
```

### 时区 (Location)
我们在来介绍一下时区的相关的函数

``` golang
// 默认 UTC    
loc, err := time.LoadLocation("") 
// 服务器设定的时区，一般为 CST
loc, err := time.LoadLocation("Local")
// 美国洛杉矶 PDT
loc, err := time.LoadLocation("America/Los_Angeles")

// 获取指定时区的时间点
local, _ := time.LoadLocation("America/Los_Angeles")
fmt.Println(time.Date(2018,1,1,12,0,0,0, local))
```

可以在 `$GOROOT/lib/time/zoneinfo.zip` 文件下看到所有时区。

### 时间运算
好了，基础的类型我们介绍完，现在开始时间运算相关的函数，也是日常业务中我们大量应用的。

``` golang
// func Sleep(d Duration)   休眠多少时间，休眠时处于阻塞状态，后续程序无法执行
time.Sleep(time.Duration(10) * time.Second)

// func After(d Duration) <-chan Time  非阻塞, 可用于延迟
time.After(time.Duration(10) * time.Second)

// func Since(t Time) Duration 两个时间点的间隔
start := time.Now()
fmt.Println(time.Since(start))   // 等价于 Now().Sub(t)， 可用来计算一段业务的消耗时间

func Until(t Time) Duration     //  等价于 t.Sub(Now())，t 与当前时间的间隔

// func (t Time) Add(d Duration) Time
fmt.Println(dt.Add(time.Duration(10) * time.Second))   // 加

func (t Time) Sub(u Time) Duration                    // 减 

// func (t Time) AddDate(years int, months int, days int) Time
fmt.Println(dt.AddDate(1, 1, 1))

// func (t Time) Before(u Time) bool
// func (t Time) After(u Time) bool
// func (t Time) Equal(u Time) bool          比较时间点时尽量使用 Equal 函数
```

### 使用场景
#### 日期时间差
``` golang
dt1 := time.Date(2018, 1, 10, 0, 0, 1, 100, time.Local)
dt2 := time.Date(2018, 1, 9, 23, 59, 22, 100, time.Local)
// 不用关注时区，go 会转换成时间戳进行计算
fmt.Println(dt1.Sub(dt2))
```

#### 基于当前时间的前后运算
``` golang
now := time.Now()

// 一年零一个月一天之后
fmt.Println(now.Date(1,1,1))
// 一段时间之后
fmt.Println(now.Add(time.Duration(10)*time.Minute))

// 计算两个时间点的相差天数
dt1 = time.Date(dt1.Year(), dt1.Month(), dt1.Day(), 0, 0, 0, 0, time.Local)
dt2 = time.Date(dt2.Year(), dt2.Month(), dt2.Day(), 0, 0, 0, 0, time.Local)
fmt.Println(int(math.Ceil(dt1.Sub(dt2).Hours() / 24)))
```

#### 时区转换
``` golang
// time.Local 用来表示当前服务器时区
// 自定义地区时间
secondsEastOfUTC := int((8 * time.Hour).Seconds())
beijing := time.FixedZone("Beijing Time", secondsEastOfUTC)
fmt.Println(time.Date(2018,1,2,0,0,0,0, beijing))  // 2018-01-02 00:00:00 +0800 Beijing Time  

// 当前时间转为指定时区时间
fmt.Println(time.Now().In(beijing))

// 指定时间转换成指定时区对应的时间
dt, err := time.ParseInLocation("2006-01-02 15:04:05", "2017-05-11 14:06:06", time.Local)

// 当前时间在零时区年月日   时分秒  时区
year, mon, day := time.Now().UTC().Date()     // 2018 April 24 
hour, min, sec := time.Now().UTC().Clock()    // 3 47 15
zone, _ := time.Now().UTC().Zone()            // UTC
```

#### 比较两个时间点
``` golang
dt := time.Date(2018, 1, 10, 0, 0, 1, 100, time.Local)
fmt.Println(time.Now().After(dt))     // true
fmt.Println(time.Now().Before(dt))    // false

// 是否相等 判断两个时间点是否相等时推荐使用 Equal 函数
fmt.Println(dt.Equal(time.Now()))
```

#### 设置执行时间
通过 `time.After` 函数与 `select` 结合使用可用于处理程序超时设定
``` golang
select {
  case m := <- c:
    // do something
  case <- time.After(time.Duration(1)*time.Second):
    fmt.Println("time out")
}
```

#### Ticker 类型
`Ticker` 类型包含一个 `channel` ，有时我们会遇到每隔一段时间执行的业务 (比如设置心跳时间等)，就可以用它来处理，这是一个重复的过程
``` golang
// 无法取消
tick := time.Tick(1 * time.Minute)
for _ = range tick {
  // do something
}

// 可通过调用 ticker.Stop 取消
ticker := time.NewTicker(1 * time.Minute)
for _ = range tick {
  // do something
}
```

#### Timer 类型
`Timer` 类型用来代表一个单独的事件，当设置的时间过期后，发送当前的时间到 `channel`, 我们可以通过以下两种方式来创建
``` golang
func AfterFunc(d Duration, f func()) *Timer   // 指定一段时间后指定的函数
func NewTimer(d Duration) *Timer     
```

以上两函数都可以使用 `Reset`, 这个有个需要注意的地方是使用 `Reset` 时需要确保 `t.C` 通道被释放时才能调用，以防止发生资源竞争的问题，可通过以下方式解决
``` golang
if !t.Stop() {
  <-t.C
}
t.Reset(d)
```

## os 包
Go 语言的 os 包中提供了操作系统函数的接口，是一个比较重要的包。顾名思义，os 包的作用主要是在服务器上进行系统的基本操作，如文件操作、目录操作、执行命令、信号与中断、进程、系统状态等等。

### 常用函数
- Hostname
函数定义:
``` golang
func Hostname() (name string, err error)
```

Hostname 函数会返回内核提供的主机名。
- Environ
函数定义:
``` golang
func Environ() []string
```

Environ 函数会返回所有的环境变量，返回值格式为 “key=value” 的字符串的切片拷贝。
- Getenv
函数定义:
``` golang
func Getenv(key string) string
```

Getenv 函数会检索并返回名为 key 的环境变量的值。如果不存在该环境变量则会返回空字符串。
- Setenv
函数定义:
``` golang
func Setenv(key, value string) error
```

Setenv 函数可以设置名为 key 的环境变量，如果出错会返回该错误。
- Exit
函数定义:
``` golang
func Exit(code int)
```

Exit 函数可以让当前程序以给出的状态码 code 退出。一般来说，状态码 0 表示成功，非 0 表示出错。程序会立刻终止，并且 defer 的函数不会被执行。
- Getuid
函数定义:
``` golang
func Getuid() int
```

Getuid 函数可以返回调用者的用户 ID。
- Getgid
函数定义:
``` golang
func Getgid() int
```

Getgid 函数可以返回调用者的组 ID。
- Getpid
函数定义:
``` golang
func Getpid() int
```

Getpid 函数可以返回调用者所在进程的进程 ID。
- Getwd
函数定义:
``` golang
func Getwd() (dir string, err error)
```

Getwd 函数可以返回一个对应当前工作目录的根路径。如果当前目录可以经过多条路径抵达（因为硬链接），Getwd 会返回其中一个。
- Mkdir
函数定义:
``` golang
func Mkdir(name string, perm FileMode) error
```

Mkdir 函数可以使用指定的权限和名称创建一个目录。如果出错，会返回 *PathError 底层类型的错误。
- MkdirAll
函数定义:
``` golang
func MkdirAll(path string, perm FileMode) error
```

MkdirAll 函数可以使用指定的权限和名称创建一个目录，包括任何必要的上级目录，并返回 nil，否则返回错误。权限位 perm 会应用在每一个被该函数创建的目录上。如果 path 指定了一个已经存在的目录，MkdirAll 不做任何操作并返回 nil。
- Remove
函数定义:
``` golang
func Remove(name string) error
```

Remove 函数会删除 name 指定的文件或目录。如果出错，会返回 *PathError 底层类型的错误。

RemoveAll 函数跟 Remove 用法一样，区别是会递归的删除所有子目录和文件。

### exec，signal，user 子包
#### exec
exec 包可以执行外部命令，它包装了 os.StartProcess 函数以便更容易的修正输入和输出，使用管道连接 I/O，以及作其它的一些调整。

``` golang
func LookPath(file string) (string, error)
```

在环境变量 PATH 指定的目录中搜索可执行文件，如果 file 中有斜杠，则只在当前目录搜索。返回完整路径或者相对于当前目录的一个相对路径。

示例代码如下：
``` golang
package main
import (
    "fmt"
    "os/exec"
)
func main() {
    f, err := exec.LookPath("main")
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(f)
}
```
运行结果如下：
``` shell
main.exe
```

#### user
可以通过 os/user 包中的 Current () 函数来获取当前用户信息，该函数会返回一个 User 结构体，结构体中的 Username、Uid、HomeDir、Gid 分别表示当前用户的名称、用户 id、用户主目录和用户所属组 id，函数原型如下：
``` golang
func Current() (*User, error)
```

示例代码如下：
``` golang
package main
import (
    "log"
    "os/user"
)
func main() {
    u, _ := user.Current()
    log.Println("用户名：", u.Username)
    log.Println("用户 id", u.Uid)
    log.Println("用户主目录：", u.HomeDir)
    log.Println("主组 id：", u.Gid)
    // 用户所在的所有的组的 id
    s, _ := u.GroupIds()
    log.Println("用户所在的所有组：", s)
}
```
运行结果如下：
``` shell
2019/12/13 15:12:14 用户名： LENOVO-PC\Administrator
2019/12/13 15:12:14 用户 id S-1-5-21-711400000-2334436127-1750000211-000
2019/12/13 15:12:14 用户主目录： C:\Users\Administrator
2019/12/13 15:12:14 主组 id： S-1-5-22-766000000-2300000100-1050000262-000
2019/12/13 15:12:14 用户所在的所有组： [S-1-5-32-544 S-1-5-22-000 S-1-5-21-777400999-2344436111-1750000262-003]
```

#### signal
一个运行良好的程序在退出（正常退出或者强制退出，如 Ctrl+C，kill 等）时是可以执行一段清理代码的，将收尾工作做完后再真正退出。一般采用系统 Signal 来通知系统退出，如 kill pid，在程序中针对一些系统信号设置了处理函数，当收到信号后，会执行相关清理程序或通知各个子进程做自清理。

Go 语言中对信号的处理主要使用 os/signal 包中的两个方法，一个是 Notify 方法用来监听收到的信号，一个是 stop 方法用来取消监听。
``` golang
func Notify(c chan<- os.Signal, sig ...os.Signal)
```

其中，第一个参数表示接收信号的 channel，第二个及后面的参数表示设置要监听的信号，如果不设置表示监听所有的信号。

【示例 1】使用 Notify 方法来监听收到的信号：
``` golang
package main
import (
    "fmt"
    "os"
    "os/signal"
)
func main() {
    c := make(chan os.Signal, 0)
    signal.Notify(c)
    // Block until a signal is received.
    s := <-c
    fmt.Println("Got signal:", s)
}
```
运行该程序，然后在 CMD 窗口中通过 Ctrl+C 来结束该程序，便会得到输出结果：
``` shell
Got signal: interrupt
```

【示例 2】使用 stop 方法来取消监听：
``` golang
package main
import (
    "fmt"
    "os"
    "os/signal"
)
func main() {
    c := make(chan os.Signal, 0)
    signal.Notify(c)
    signal.Stop(c) // 不允许继续往 c 中存入内容
    s := <-c       //c 无内容，此处阻塞，所以不会执行下面的语句，也就没有输出
    fmt.Println("Got signal:", s)
}
```
因为使用 Stop 方法取消了 Notify 方法的监听，所以运行程序没有输出结果。

### os.Open 参数
``` golang
os.OpenFile(name string, flag int, perm os.FileMode) (*os.File, error)
```

flag 参数：
|   参数名  |   含义    |
|   O_RDONLY    |	打开只读文件    |
|   O_WRONLY    |	打开只写文件    |
|   O_RDWR  |	打开既可以读取又可以写入文件    |
|   O_APPEND    |	写入文件时将数据追加到文件尾部  |
|   O_CREATE    |	如果文件不存在，则创建一个新的文件  |
|   O_EXCL  |	文件必须不存在，然后会创建一个新的文件  |
|   O_SYNC  |	打开同步 I/0    |
|   O_TRUNC |   文件打开时可以截断  |

示例：
``` golang
file, err := os.OpenFile("a.txt", os.O_APPEND|os.O_WRONLY, os.ModeAppend)
```
## sync 包
`sync` 整个包都围绕这 `Locker` 进行，这是一个 `interface` ：
``` golang
type Locker interface {
        Lock()
        Unlock()
}
```
只有两个方法，Lock() 和 Unlock()。

另外该包下的对象，在使用过之后，** 千万不要复制 **。

### 为什么需要锁？
在并发的情况下，多个线程或协程同时去修改一个变量，可能会出现如下情况：
``` golang
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var a = 0

    // 启动 100 个协程，需要足够大
    // var lock sync.Mutex
    for i := 0; i < 100; i++ {
        go func(idx int) {
            // lock.Lock()
            // defer lock.Unlock()
            a += 1
            fmt.Printf("goroutine %d, a=%d\n", idx, a)
        }(i)
    }

    // 等待 1s 结束主程序
    // 确保所有协程执行完
    time.Sleep(time.Second)
}
```
观察打印结果，是否出现 a 的值是相同的情况（未出现则重试或调大协程数），答案：是的。

显然这不是我们想要的结果。出现这种情况的原因是，协程依次执行：  
从寄存器读取 a 的值 -> 然后做加法运算 -> 最后写会寄存器。

试想，此时一个协程取出 a 的值 3，正在做加法运算（还未写回寄存器）。同时另一个协程此时去取，取出了同样的 a 的值 3。最终导致的结果是，两个协程产出的结果相同，a 相当于只增加了 1。

所以，锁的概念就是，我正在处理 a（锁定），你们谁都别和我抢，等我处理完了（解锁），你们再处理。这样就实现了，同时处理 a 的协程只有一个，就实现了同步。

把上面代码里的注释取消掉再试下。

### 什么是互斥锁 Mutex？
什么是互斥锁？它是锁的一种具体实现，有两个方法：
``` golang
func (m *Mutex) Lock()
func (m *Mutex) Unlock()
```

在首次使用后不要复制该互斥锁。对一个未锁定的互斥锁解锁将会产生运行时错误。

一个互斥锁只能同时被一个 goroutine 锁定，其它 goroutine 将阻塞直到互斥锁被解锁（重新争抢对互斥锁的锁定）。如：
``` golang
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    ch := make(chan struct{}, 2)

    var l sync.Mutex
    go func() {
        l.Lock()
        defer l.Unlock()
        fmt.Println("goroutine1: 我会锁定大概 2s")
        time.Sleep(time.Second * 2)
        fmt.Println("goroutine1: 我解锁了，你们去抢吧")
        ch <- struct{}{}
    }()

    go func() {
        fmt.Println("groutine2: 等待解锁")
        l.Lock()
        defer l.Unlock()
        fmt.Println("goroutine2: 哈哈，我锁定了")
        ch <- struct{}{}
    }()

    // 等待 goroutine 执行结束
    for i := 0; i < 2; i++ {
        <-ch
    }
}
```

注意，平时所说的锁定，其实就是去锁定互斥锁，而不是说去锁定一段代码。也就是说，当代码执行到有锁的地方时，它获取不到互斥锁的锁定，会阻塞在那里，从而达到控制同步的目的。

### 什么是读写锁 RWMutex?
那么什么是读写锁呢？它是针对读写操作的互斥锁，读写锁与互斥锁最大的不同就是可以分别对 读、写 进行锁定。一般用在大量读操作、少量写操作的情况：
``` golang
func (rw *RWMutex) Lock()
func (rw *RWMutex) Unlock()

func (rw *RWMutex) RLock()
func (rw *RWMutex) RUnlock()
```

由于这里需要区分读写锁定，我们这样定义：
- 读锁定（RLock），对读操作进行锁定
- 读解锁（RUnlock），对读锁定进行解锁
- 写锁定（Lock），对写操作进行锁定
- 写解锁（Unlock），对写锁定进行解锁

在首次使用之后，不要复制该读写锁。不要混用锁定和解锁，如：Lock 和 RUnlock、RLock 和 Unlock。因为对未读锁定的读写锁进行读解锁或对未写锁定的读写锁进行写解锁将会引起运行时错误。

如何理解读写锁呢？
- 同时只能有一个 goroutine 能够获得写锁定。
- 同时可以有任意多个 gorouinte 获得读锁定。
- 同时只能存在写锁定或读锁定（读和写互斥）。

也就是说，当有一个 goroutine 获得写锁定，其它无论是读锁定还是写锁定都将阻塞直到写解锁；当有一个 goroutine 获得读锁定，其它读锁定任然可以继续；当有一个或任意多个读锁定，写锁定将等待所有读锁定解锁之后才能够进行写锁定。所以说这里的读锁定（RLock）目的其实是告诉写锁定：有很多人正在读取数据，你给我站一边去，等它们读（读解锁）完你再来写（写锁定）。

使用例子：
``` golang
package main

import (
    "fmt"
    "math/rand"
    "sync"
)

var count int
var rw sync.RWMutex

func main() {
    ch := make(chan struct{}, 10)
    for i := 0; i < 5; i++ {
        go read(i, ch)
    }
    for i := 0; i < 5; i++ {
        go write(i, ch)
    }

    for i := 0; i < 10; i++ {
        <-ch
    }
}

func read(n int, ch chan struct{}) {
    rw.RLock()
    fmt.Printf("goroutine %d 进入读操作...\n", n)
    v := count
    fmt.Printf("goroutine %d 读取结束，值为：%d\n", n, v)
    rw.RUnlock()
    ch <- struct{}{}
}

func write(n int, ch chan struct{}) {
    rw.Lock()
    fmt.Printf("goroutine %d 进入写操作...\n", n)
    v := rand.Intn(1000)
    count = v
    fmt.Printf("goroutine %d 写入结束，新值为：%d\n", n, v)
    rw.Unlock()
    ch <- struct{}{}
}
```

### WaitGroup 例子
WaitGroup 用于等待一组 goroutine 结束，用法很简单。它有三个方法：
``` golang
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
```
Add 用来添加 goroutine 的个数。Done 执行一次数量减 1。Wait 用来等待结束：
``` golang
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 5; i++ {
        // 计数加 1
        wg.Add(1)
        go func(i int) {
            // 计数减 1
            defer wg.Done()
            time.Sleep(time.Second * time.Duration(i))
            fmt.Printf("goroutine%d 结束 \ n", i)
        }(i)
    }

    // 等待执行结束
    wg.Wait()
    fmt.Println("所有 goroutine 执行结束")
}
```

注意，`wg.Add()` 方法一定要在 `goroutine` 开始前执行哦。

### Cond 条件变量
Cond 实现一个条件变量，即等待或宣布事件发生的 goroutines 的会合点，它会保存一个通知列表。基本思想是当某中状态达成，goroutine 将会等待（Wait）在那里，当某个时刻状态改变时通过通知的方式（Broadcast，Signal）的方式通知等待的 goroutine。这样，不满足条件的 goroutine 唤醒继续向下执行，满足条件的重新进入等待序列。
``` golang
type Cond struct {
    noCopy noCopy
  
    // L is held while observing or changing the condition
    L Locker
  
    notify  notifyList // 通知列表
    checker copyChecker
}
```

``` golang
func NewCond(l Locker) *Cond
func (c *Cond) Broadcast()
func (c *Cond) Signal()
func (c *Cond) Wait()
```

Wait 方法、Signal 方法和 Broadcast 方法。它们分别代表了等待通知、单发通知和广播通知的操作。

我们来看一下 Wait 方法：
``` golang
func (c *Cond) Wait() {
    c.checker.check()
    t := runtime_notifyListAdd(&c.notify)
    c.L.Unlock()
    runtime_notifyListWait(&c.notify, t)
    c.L.Lock()
}
```

它的操作为：加入到通知列表 -> 解锁 L -> 等待通知 -> 锁定 L。其使用方法是：
``` golang
c.L.Lock()
for !condition() {
    c.Wait()
}
... make use of condition ...
c.L.Unlock()
```

举个例子：
``` golang
// Package main provides ...
package main

import (
    "fmt"
    "sync"
    "time"
)

var count int = 4

func main() {
    ch := make(chan struct{}, 5)

    // 新建 cond
    var l sync.Mutex
    cond := sync.NewCond(&l)

    for i := 0; i < 5; i++ {
        go func(i int) {
            // 争抢互斥锁的锁定
            cond.L.Lock()
            defer func() {
                cond.L.Unlock()
                ch <- struct{}{}
            }()

            // 条件是否达成
            for count > i {
                cond.Wait()
                fmt.Printf("收到一个通知 goroutine%d\n", i)
            }
            
            fmt.Printf("goroutine%d 执行结束 \ n", i)
        }(i)
    }

    // 确保所有 goroutine 启动完成
    time.Sleep(time.Millisecond * 20)
    
    // 锁定一下
    fmt.Println("broadcast...")
    cond.L.Lock()
    count -= 1
    cond.Broadcast()
    cond.L.Unlock()

    time.Sleep(time.Second)
    fmt.Println("signal...")
    cond.L.Lock()
    count -= 2
    cond.Signal()
    cond.L.Unlock()

    time.Sleep(time.Second)
    fmt.Println("broadcast...")
    cond.L.Lock()
    count -= 1
    cond.Broadcast()
    cond.L.Unlock()

    for i := 0; i < 5; i++ {
        <-ch
    }
}
```

### Pool 临时对象池
`sync.Pool` 可以作为临时对象的保存和复用的集合。其结构为：
``` golang
type Pool struct {
    noCopy noCopy

    local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
    localSize uintptr        // size of the local array

    // New optionally specifies a function to generate
    // a value when Get would otherwise return nil.
    // It may not be changed concurrently with calls to Get.
    New func() interface{}
}

func (p *Pool) Get() interface{}
func (p *Pool) Put(x interface{})
```

新键 Pool 需要提供一个 New 方法，目的是当获取不到临时对象时自动创建一个（不会主动加入到 Pool 中），Get 和 Put 方法都很好理解。

深入了解过 Go 的同学应该知道，Go 的重要组成结构为 M、P、G。Pool 实际上会为每一个操作它的 goroutine 相关联的 P 都生成一个本地池。如果从本地池 Get 对象的时候，本地池没有，则会从其它的 P 本地池获取。因此，Pool 的一个特点就是：可以把由其中的对象值产生的存储压力进行分摊。

它有着以下特点：

- Pool 中的对象在仅有 Pool 有着唯一索引的情况下可能会被自动删除（取决于下一次 GC 执行的时间）。
- goroutines 协程安全，可以同时被多个协程使用。

> GC 的执行一般会使 Pool 中的对象全部移除。

那么 Pool 都适用于什么场景呢？从它的特点来说，适用与无状态的对象的复用，而不适用与如连接池之类的。在 fmt 包中有一个很好的使用池的例子，它维护一个动态大小的临时输出缓冲区。

官方例子：
``` golang
package main

import (
    "bytes"
    "io"
    "os"
    "sync"
    "time"
)

var bufPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func timeNow() time.Time {
    return time.Unix(1136214245, 0)
}

func Log(w io.Writer, key, val string) {
    // 获取临时对象，没有的话会自动创建
    b := bufPool.Get().(*bytes.Buffer)
    b.Reset()
    b.WriteString(timeNow().UTC().Format(time.RFC3339))
    b.WriteByte(' ')
    b.WriteString(key)
    b.WriteByte('=')
    b.WriteString(val)
    w.Write(b.Bytes())
    // 将临时对象放回到 Pool 中
    bufPool.Put(b)
}

func main() {
    Log(os.Stdout, "path", "/search?q=flowers")
}
```

打印结果：  
`2006-01-02T15:04:05Z path=/search?q=flowers`

### Once 执行一次
使用 sync.Once 对象可以使得函数多次调用只执行一次。其结构为：
``` golang
type Once struct {
    m    Mutex
    done uint32
}

func (o *Once) Do(f func())
```

用 done 来记录执行次数，用 m 来保证保证仅被执行一次。只有一个 Do 方法，调用执行。
``` golang
package main

import (
    "fmt"
    "sync"
)

func main() {
    var once sync.Once
    onceBody := func() {
        fmt.Println("Only once")
    }
    done := make(chan bool)
    for i := 0; i < 10; i++ {
        go func() {
            once.Do(onceBody)
            done <- true
        }()
    }
    for i := 0; i < 10; i++ {
        <-done
    }
}
```
打印结果：  
`Only once`

## content 包