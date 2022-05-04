---
title: "Golang [深入理解] 7-关于 nil"
date: 2021-09-14 15:09:02
draft: false
toc: false
images:
categories: ["Golang", "Advanced"]
tags:
  - Golang
  - 引用类型
math: true
---

关于你可能知道的 nil 的使用

<!--more-->

# 关于 nil

`nil` 是啥？golang 中的 nil 相当于其他语言中的 NULL。

但 nil 的本质是啥？到底是一种类型？还是一种变量？还是一个关键字？

按照博主 [奇伢云存储](https://mp.weixin.qq.com/s/sHLYy_4XA6254-vLmlu0IA) 的介绍，nil 是一个特殊变量，也是一种触发机制。



>```go
>// nil is a predeclared identifier representing the zero value for a
>// pointer, channel, func, interface, map, or slice type.
>var nil Type // Type must be a pointer, channel, func, interface, map, or slice type
>
>// Type is here for the purposes of documentation only. It is a stand-in
>// for any Go type, but represents the same type for any given function
>// invocation.
>type Type int
>```
>
>在编译器层面，nil 是使用一个变量来表示。

nil 其实是语言层面给出的：在编译器中，nil 是一个 Type 型变量，Type 类型 是以 int 为基础的自定义类型。

但这不代表我们使用语言编程程序时能把 nil 当作一个变量。比如你把 nil 赋值给一个 byte 型变量，是不可以的，因为编译器不让。这是编译器的特权，你可没有！

这里明确一点，能够赋值 nil 的类型只有 “引用六兄弟  SMCIFP”：

- `slice`
- `map`
- `chan`
- `interface`
- `func`
- `pointer`

这就是编译器给你这位语言使用者的权限。在上面的源码中 `var nil Type` 后面的注释就有写。

所以除了这 6 兄弟之外，别的类型都**不可以**赋值 nil，或者和 nil 比较。

>   你可以把这 “引用六兄弟 SMCIFP” 理解为一种 nilable 的类型。
>
>   所谓 nilable，就是可以赋值为 nil（able to assign with nil），可以和 nil 比较（able to compare to nil）。
>
>   如果用 go 提案中用泛型表示，即：
>
>   ```go
>   type nilable interface {
>       type []int, map[int]int, chan int, interface{}, unsafe.Pointer, func
>   }
>   
>   func compare[T nilable] (a T) bool {
>       return a == nil
>   }
>   
>   func assign[T nilable] (a T) {
>       a = nil
>   }
>   ```
>
>   并不严谨，仅仅用于表达 nilable 这个意思而已。



于是在编译器编译你的代码的时候，如果它看到你写出 `a == nil` 或 `a = nil` 这样的代码时，它要确认 a 是 nilable 的（是引用六兄弟 SMCIFP）。所以说 nil 是一种触发机制。

那么这六兄弟有什么特殊呢？

我们将 go 中的类型分为`值类型`和`引用类型` 两类。

- 值类型就是 int 系列、uint 系列等；
- 引用类型就是 “引用六兄弟 SMCIFP”。

在 golang 中，使用变量前需要先声明变量。声明变量的本质是分配一块内存。

Clang（golang 的前辈）在声明变量是分配内存，但也只是分配内存，并不管这块内存里是不是干净的，所以会出现野指针此类问题。

我写 Clang 的时候就会有一个习惯，在声明变量的时候顺带初始化一个零值。

```c
int sum = 0;
double price = 0.0; 
int* p = NULL;
```

这样的习惯可以有效的预防一些意料之外的情况。

但是 golang 是二十一世纪的语言，它的变量声明在分配内存时是「置 0 分配」，也就是说 golang 可以保证你声明的变量分配的那块内存是干净的，你可以放心的使用。

所以在 golang 中声明一个变量但没有初始化，这个变量的值是其类型的零值，即：

```go
var sum int     // sum == 0
var str string  // str == ""
var p *int      // p == nil
```

## SMCIFP

>   先声明一点：这六兄弟中，slice、map、chan、interface 是一种复合结构，pointer 和 func 是一个指针。

其实上面引用中的这句话已经能大概的看出答案了，因为他们是指针，所以是 nilable。我们可以看看编译器中这“六兄弟”的结构。



slice

```go
type slice struct {
   array unsafe.Pointer         // 管理的内存块首地址
   len   int                    // 动态数组实际使用大小
   cap   int                    // 动态数组内存大小
}
```

map

```go
// A header for a Go map.
type hmap struct {
    // Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
    // Make sure this stays in sync with the compiler's definition.
    count     int // # live cells == size of map.  Must be first (used by len() builtin)
    flags     uint8
    B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
    noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
    hash0     uint32 // hash seed

    buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

    extra *mapextra // optional fields
}
```

chan

```go
type hchan struct {
    qcount   uint           // total data in the queue
    dataqsiz uint           // size of the circular queue
    buf      unsafe.Pointer // points to an array of dataqsiz elements
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index
    recvx    uint   // receive index
    recvq    waitq  // list of recv waiters
    sendq    waitq  // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}
```

interface

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

func 和 pointer 本身就是一个指针。

可以看出，map 和 chan 两个都是比较大的结构体，而当你声明一个 map 或 chan 时，你必须通过 `make()` 才能使用。为什么？

因为这两个结构体比较大，为了提高性能，你声明的时候 `var c chan int` 的时候，c 是个指针，当你 `make()` 的时候才会分配内存，也就是初始化 `hchan`。map 也是同样的道理。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo//blog/illustration-pic/Go/IMG/IMG20210821110117.png)

像图中那样，slice、map、chan、interface 不管他们有多大，例如 slice 有 1000 个元素，map 有 10万对键值对，chan 缓存中有 100万 个数据，那都是他们背后的数组的大小；他们本身的大小，只有一个结构体那么大。

所以编译器在判断 nil 的时候其实非常快，因为这几个类型本身是指针，编译器只需要像如下判断即可：

- 对于 slice 的时候，判断 s 中的 array 字段是不是零值；
- 对于 map、chan 的时候，判断 m、c 自身是不是零值；
- 对于 interface 的时候，判断 i 自身是不是零值；
- 对于 pointer、func 的时候，判断 p、f 自身是不是零值；

同样，在赋值 nil 的时候，也是把他们自身赋值为 nil 即可。

除了 slice，因为它的结构比较小，总共就 24Byte（32位机中 12B），所以声明的时候直接初始化了一个 slice 结构，对 array 字段进行维护。

具体细节，推荐仔细阅读 [https://mp.weixin.qq.com/s/sHLYy_4XA6254-vLmlu0IA](https://mp.weixin.qq.com/s/sHLYy_4XA6254-vLmlu0IA)，这篇文章讲的非常通俗和详细。



## 总结

1. nil 只能用于引用六兄弟 SMCIFP；
2. 变量只是一个指向一块内存的指针
3. slice 结构较小，所以声明的时候就直接分配内存了，map 和 chan 需要 make() 才分配内存。
4. 除了 pointer 和 func，其他都是使用对应结构体来管理
5. 对 MCIFP 的判断 nil 和 赋值 nil，都是对变量本身进行操作，而 Slice 是对 array 字段进行操作。

再次强烈推荐阅读 [https://mp.weixin.qq.com/s/sHLYy_4XA6254-vLmlu0IA](https://mp.weixin.qq.com/s/sHLYy_4XA6254-vLmlu0IA)。




