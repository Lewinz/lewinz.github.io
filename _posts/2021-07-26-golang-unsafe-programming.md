---
layout: post
title: golang 不安全编程
categories: [golang, unsafe, programming]
description: golang 不安全编程
keywords: golang, unsafe, programming
---

不安全编程？用 go 语言以来也没发现有啥不安全的啊，而且 go 里面有垃圾回收，也不需要我们来管理内存。当听到不安全编程这几个字，唯一能想到的也就是指针了，只有指针才可能导致不安全问题。我们知道 go 中是有指针的，但是 go 的指针并不能像 C 语言中的指针一样可以进行运算，因此在提供了指针的便利性的同时，又保证了安全。关于 go 中的指针我们之前已经说过了，以及它都做了哪些限制。

但是在 go 中，可以通过一个叫做 unsafe 的包让指针突破限制，从而进行运算，可一旦用不好就会导致很严重的问题，所以我们说这是不安全编程。但即便如此我们还是可以使用的，因为用好了在某些场景下能够带来很大的便利，而且 go 的内部也在大量的使用 unsafe 这个包。

## go 语言中的指针
尽管 go 的指针没有 C 的指针那么强大，但是能够获取一个变量的地址，并且能通过地址来改变存储的值，我个人认为已经足够了。
``` golang
package main

import "fmt"

func pass_by_value(num int) {
    num = 3
}

func pass_by_pointer(num *int) {
    *num = 3
    num = nil
}

func main() {

    num := 1
    pass_by_value(num)
    fmt.Println("传递值：", num) //传递值： 1
    pass_by_pointer(&num)
    fmt.Println("传递指针：", num) //传递指针： 3
}
```

我们知道 go 的函数传递方式是值传递，不管传递什么，都是拷贝一份出来。而且函数里面形参叫什么是无所谓，这里我们函数的形参不叫 num，叫其他的也无所谓。

pass_by_value 中接收一个整型，当我们传递 num 的时候，会把 num 的值拷贝一份出来传进去，此时函数里面无论做什么修改，都不会影响外面的 num，因为不是一个东西。

pass_by_pointer 中接收一个指针，那么传递 &num 的时候，依旧会把指针拷贝一份；我们说 go 只有值传递，传递指针的话也是把指针拷贝一份。由于是拷贝，所以两者没有任何关系，只不过它们存储的地址是一样的，但就变量本身来说，里面的 num 这个 * int 类型的变量和我们传递的 &num 没有关系。由于存储的地址一样，所以两者操作的都是同一片内存，因此 *num = 3 之后是会影响外面的 num 的。但是指针也是拷贝，所以函数里面的 num = nil 跟外面没关系。



所以 go 的指针在改变内存的值的时候和 C 是一样的，但是它和 C 中的指针相比，又弱化了许多。
- 弱化一：go中的指针不能进行数学运算
- 弱化二：go中不同类型的指针不能进行转化或者赋值
- 弱化三：go中不同类型的指针不能进行比较

## unsafe：不安全编程
我们知道 go 的指针实际上是类型安全的，因为 go 编译器对类型的检测是十分的严格，让你在享受指针带来的便利时，又给指针施加了很多制约来保证安全。但是保证安全是需要以牺牲效率为代价的，如果你能保证写出的程序就是安全的，那么你可以使用 go 中的不安全指针，从而绕过类型系统的检测，让你的程序运行的更快。

如果是一个高阶 go 程序员的话，怎么能不会 unsafe 包呢？它可以绕过 go 的类型系统的检测，直接访问内存，增加效率。go 中的很多限制，比如不能操作结构体中的未导出成员等等，但是有了 unsafe 包，就可以直接突破这些限制。所以这个包叫做 unsafe，我们称使用 unsafe 为不安全编程，因为它很危险，官方也不推荐使用，估计正因为如此也设计了这么个名字吧。但是你底层都在大量使用，那我们为什么不能用。

我们刚才提到了不安全指针，那么我们先来看看什么是不安全指针。
``` golang
package unsafe
type ArbitraryType int
type Pointer *ArbitraryType

func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr
```
unsafe 包下面只有一个 unsafe.go 文件，这个文件里面把注释去掉就上面 6 行代码，是的你没有看错。当然功能肯定都内嵌在编译器里面，至于怎么实现的我们就不管啦，看看怎么用就行了。我们先来看看这两行：
``` golang
type ArbitraryType int
type Pointer *ArbitraryType
```
Arbitrary 表示任意的，所以这个 Pointer 可以是任何类型的指针，比如：*int、*string、*float64 等等。也就是说任何类型的指针都可以传递给它。
``` golang
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    var a int
    var s string
    var f float64
    fmt.Println(unsafe.Pointer(&a)) //0xc000062080
    fmt.Println(unsafe.Pointer(&s)) //0xc00004e1c0
    fmt.Println(unsafe.Pointer(&f)) //0xc000062088
}
```
unsafe.Pointer() 是有返回值的，返回的当然也是一个指针，但是这个指针同样是无法进行运算的。如果无法运算，那么我们还是无法实现通过指针自增的方式，访问数组的下一个元素啊。别急，所以还有一个整数类型：uintptr，我们 unsafe.Pointer() 是可以和 uintptr 互相转化的，而这个 uintptr 是可以运算的，并且它还足够大。至少我们目前看到了两个功能：
- 任何类型的指针都可以和unsafe.Pointer相互转化
- unsafe.Pointer可以和uintptr互相转化

但是需要注意的是，uintptr 并没有指针的含义，所以它指向的内存是会被回收的；而 unsafe.Pointer 有指针的含义，可以确保其指向的对象不会被回收。

## 使用 unsafe 带你突破限制
那么我们就来看看 unsafe 这个包具有哪些黑魔法，以及它有能帮助我们实现什么功能。

### 像 C 语言一样访问数组或切片
``` golang
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    //这里把数字弄成没有规律的，就不用1 2 3 4 5 6了
    var arr = []int{177, 123, 3, 221, 5, 1211}

    //获取第二个元素的指针，我们也不从头获取
    //因为从中间获取都可以的话，那么从头获取肯定可以
    //然后传给unsafe.Pointer()，将*int转化成Pointer类型
    pointer := unsafe.Pointer(&arr[1])

    //注意了：下面要将Pointer转成uintptr，因为Pointer是不能运算的
    u_pointer := uintptr(pointer)
    //此时的u_pointer就相当于C中的指针了，但是还有一点不同
    //C中的指针直接++即可，指针会自动移到到下一个元素的位置
    //而go中的uintptr相当于一个整型，我们不能++，而是需要+8，因为一个int占8个字节，所以go中需要加上元素所占的大小
    //所以我们发现C中的+n是从当前元素开始，移动n个元素，不管元素是什么类型。
    //但是go的+n是移动n个字节。
    //所以C中的指针+2 等于 go中uintptr + 2 * (元素类型所占的字节)
    u_pointer += 16 //移动两个元素

    //然后再转回来，要先转成Pointer，再转成对应的指针类型
    pointer = unsafe.Pointer(u_pointer)

    //这个pointer是我们通过&arr[1]也就是*int类型的指针得到的，那么结果也要转成*int
    int_pointer := (*int)(pointer)
    // 打印了221，结果是正确的
    fmt.Println(*int_pointer) // 221

    //这里也可以转成*string，即便我们的pointer是通过*int得到的
    //因为Pointer可以是任何指针类型
    string_pointer := (*string)(pointer)
    //也是可以打印的，但是通过*来访问内存的话就会报错，panic: runtime error: invalid memory address or nil pointer dereference
    fmt.Println(string_pointer) //0xc00008c048

    //这里我们再加上1，不加8，那么会出现什么后果
    //我们知道再加上8，就会访问221后面的5
    u_pointer += 1
    fmt.Println(*(*int)(unsafe.Pointer(u_pointer))) // 360287970189639680
    //我们看到此时得到的是一个我们也不知道从哪里来的脏数据，所以一定要加上对应的字节
}
```
所以我们发现 unsafe.Pointer 就类似于一座桥，*T 通过 Pointer 转成 uintptr，然后进行指针运算，运算完成之后，再通过 Pointer 转回 *T，此时的 *T 就是我们想要的了。

### 指针访问结构体
我们知道结构体是可以有字段的，那么我们也可以把结构体想象成数组，字段想象成数组的元素
``` golang
package main

import (
    "fmt"
    "unsafe"
)

type score struct {
    math    int
    english int
    history int
}

func main() {
    s := score{math: 90, english: 92, history: 85}

    //我们看到通过unsafe.Pointer的方式，获取结构体的指针，可以直接转换为结构体第一个字段的指针
    p := unsafe.Pointer(&s)
    fmt.Println(*(*int)(p)) //90
    //math字段是一个整型，那么p转为uintptr之后加上8，就可以转换成第二个字段的指针
    fmt.Println(*(*int)(unsafe.Pointer(uintptr(p) + 8))) //92
    //同理加上16就是第三个
    fmt.Println(*(*int)(unsafe.Pointer(uintptr(p) + 16))) //85

    //这里显然就是一个乱七八糟的值了
    fmt.Println(*(*int)(unsafe.Pointer(uintptr(p) + 29))) //70709489434624
}
```
我们知道切片是一个结构体，有三个字段，分别是指向底层数组的指针，以及大小和容量。
``` golang
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    //申请大小为5，容量为10的切片
    s := make([]int, 5, 10)

    //第一个元素显然是指向底层数组的指针，大小也是8个字节。我们来看第二个和第三个
    //虽然有些长，但是从内往外的话，还是很好看懂的。如果不习惯的话可以多写几行
    fmt.Printf("长度：%d\n", *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + 8)))  //长度：5
    fmt.Printf("容量：%d\n", *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + 16))) //容量：10
}
```
我们看到 unsafe 包还是很强大的，之所以叫 unsafe 是因为如果用不好后果会很严重。但是如果能正确使用的话，能够做到很多之前做不到的事情。

### 获取对象的大小
我们目前可以使用 unsafe 做很多事情了，但是还不够，我们看到 unsafe 这个包除了给我们提供了 Pointer 这个类型之外，还给我们提供了三个函数。
``` golang
func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr
```
这三个函数返回的都是 uintptr 类型，这个类型你就看成是整型即可，它是可以和数字进行运算的，可以转为 int。我们先来看看 Sizeof：
``` golang
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    a := 123
    b := "h"
    c := []int{1, 2, 3}
    fmt.Println(unsafe.Sizeof(a)) //8
    //关于字符串为什么是16
    //go中的字符串在底层是一个结构体，这个结构体有两个元素
    //一个是字符串的首地址，一个是字符串的长度
    //所以是16，因为go的字符串底层对应的是一个字符数组
    fmt.Println(unsafe.Sizeof(b)) //16

    //切片我们说过底层也是一个结构体，有三个字段，指向底层数组的指针、大小、容量，所以是24个字节
    fmt.Println(unsafe.Sizeof(c)) //24
}
```
go 中的 Sizeof 和 C 中的 sizeof 还是比较类似的，但是 go 中的 Sizeof 不能接收类型本身， 比如你可以传入一个 123，但是你不能传入一个 int，这是不行的。至于获取一个字符串的大小结果是 16，这个是由 go 底层字符串的结构决定的。对了，当我们获取一个结构体的大小的时候，我们看到貌似是将结构体中的每一个字段的值的大小进行相加，至少目前看来是这样的。

### 获取结构体成员的偏移量
对于一个结构体来说，可以使用 Offsetof 来获取结构体成员的偏移量，进而获取成员的地址，从而改变内存的值。这里提一句：结构体会被分配一块连续的内存，结构体的地址也 代表 了第一个成员的地址。但是你懂的，我们不可能直接通过对结构体的地址加上 * 来获取第一个成员的值，只能通过 unsafe.Pointer 转化，然后再转化成对应类型的指针，才能获取。
``` golang
package main

import (
    "fmt"
    "unsafe"
)

type girl struct {
    //对应的字节数
    name   string   // 16
    age    int      // 8
    gender string   //16
    hobby  []string //24
}

func main() {
    g := girl{"mashiro", 17, "f", []string{"画画", "开车"}}
    //首先这几步操作应该不需要解释了，直接想象成数组即可
    fmt.Println(*(*string)(unsafe.Pointer(&g)))                                          // mashiro
    fmt.Println(*(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + 16)))               // 17
    fmt.Println(*(*string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + 16 + 8)))        // f
    fmt.Println(*(*[]string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + 16 + 8 + 16))) // [画画 开车]

    //我们看到即使对具有不同字段类型的结构体，依旧可以自由操作，只要搞清楚每个字段的大小即可
    *(*[]string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + 16 + 8 + 16)) =
        append(*(*[]string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + 16 + 8 + 16)), "料理")
    fmt.Println(g) // {mashiro 17 f [画画 开车 料理]}

    //我们看到，即便操作起来没有问题，但是有一个缺陷，就是我们必须要事先计算好每一个字段占多少个字节，尽管我们可以通过unsafe.Sizeof可以很方便的计算。
    //但是有没有不用计算的方法呢？显然有，就是我们说的Offsetof。但是这个Offsetof又有点特殊，它表示的是偏移量
    //比如我想访问hobby这个字段，那么这么做可以，直接以&g为起点，此时偏移量为0，加上unsafe.Offsetof(g.hobby)，直接偏移到hobby
    fmt.Println(*(*[]string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + unsafe.Offsetof(g.hobby)))) // [画画 开车 料理]

    //其余的也是一样，获取哪个字段，直接传入哪个字段即可，个人觉得这个Offsetof比自己计算要方便一些
    fmt.Println(*(*string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + unsafe.Offsetof(g.name))))   // mashiro
    fmt.Println(*(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + unsafe.Offsetof(g.age))))       // 17
    fmt.Println(*(*string)(unsafe.Pointer(uintptr(unsafe.Pointer(&g)) + unsafe.Offsetof(g.gender)))) // f
}
```
而且我们知道，如果在别的包里面，结构体里的字段没有大写，那么是无法导出的，然鹅即便如此，我们依旧可以通过 unsafe 包绕过这些限制。
``` golang
package hahaha

type OverWatch struct {
    name   string
    age    int
    Gender string
    weapon string
}
```
这些字段有三个没有大写，理论上是无法导出的，因为 golang 会进行检测，但是使用 unsafe 就可以绕过这些检测。
``` golang
package main

import (
    "fmt"
    "hahaha"
    "unsafe"
)

func main() {
    hero := new(hahaha.OverWatch)
    //设置name
    *(*string)(unsafe.Pointer(uintptr(unsafe.Pointer(hero)))) = "麦克雷"
    //设置age，因为Offsetof需要指定访问的字段，而字段又没有被导出，所以无法通过Offsetof的方式
    //因此需要手动计算对应类型的偏移量，因为是string类型，所以加上一个Sizeof("")，当然也可以手动填上16
    *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(hero)) + unsafe.Sizeof(""))) = 37
    //这个就可以直接设置了，因为被导出了
    hero.Gender = "男"
    //老规矩，这里是两个string加上一个int的大小
    *(*string)(unsafe.Pointer(uintptr(unsafe.Pointer(hero)) + unsafe.Sizeof("")*2 + unsafe.Sizeof(123))) = "维和者"
    fmt.Println(*hero) // {麦克雷 37 男 维和者}
}
```
## 小结
参考于：<https://qcrao.com/2019/06/03/dive-into-go-unsafe/>，用原文作者的话来说就是：

> unsafe 包绕过了 go 的类型系统，达到直接操作内存的目的，使用它有一定的风险性。但是在某些场景下，使用 unsafe 包提供的函数会提升代码的效率，go 源码中也是大量使用 unsafe 包。

> uintptr 可以和 unsafe.Pointer 进行相互转换，uintptr 可以进行数学运算。这样，通过 uintptr 和 unsafe.Pointer 的结合就解决了 go 指针不能进行数学运算的限制。通过 unsafe 相关函数，可以获取结构体私有成员的地址，进而对其做进一步的读写操作，突破 go 的类型安全限制。关于 unsafe 包，我们更多关注它的用法。

> 顺便说一句，unsafe 包用多了之后，也不觉得它的名字有多么地不 "美观" 了。相反，因为使用了官方并不提倡的东西，反而觉得有点酷炫，或许这就是叛逆的感觉吧。个人非常赞同，觉得真的很酷。

