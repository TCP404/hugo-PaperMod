---
title: "Golang [技术启发] 安全函数 SaveFunc"
date: 2022-01-08T01:50:48+08:00
draft: false
toc: false
images:
math: true
categories: ["Golang", "Advanced"]
tags:
  - Golang
  - 事务
  - 安全函数
  - SaveFunc
---

安全地执行不安全的函数

<!--more-->

## 场景

在日常开发中，可能会遇到一种场景：

开启一个事务去执行某个任务，这个任务的其中一个子任务有可能失败，但是你又不想这个失败影响整个事务，只想打印一下日志。这种场景就可以用上安全函数 SaveFunc。

举个栗子，在修改简历时，我希望能发送一条通知给关注了这份简历的人，而发送通知有可能失败，甚至引发 panic。



```go {hl_lines=[9, 22]}
func UpdateResume(rid Resume, data ResumeData) bool {
    if data == nil {
        return false
    }

    // 启动一个事务更新简历
    err := newTransaction(func(){
        rid.SetData(data)
        SendNotice()    // 可能因这个函数 panic 而导致整个 UpdateResume 事务失败
    })

    if err != nil {
        log.Printf("transaction fail. Cause: ", err)
        return false
    }
    log.Printf("transaction success.")
    return true
}

func SendNotice() {
    if Search(ctx) == nil {
        panic("can not get resume.")
    }

    notice := new(Notice)
    if !notice.Send() {
        log.Println("sent notice fail.")
    }
    log.Println("sent notice success.")
}
```



在发送通知的函数 `SendNotice()` 如果 Search 不到资源则会直接 panic，这样会影响到整个事务失败回滚，从而 `UpdateResume()` 也失败。

这样显然不合理，总不能因为你发送通知失败就不让我更新简历吧？所以对于这样的场景，我们要对 `SendNotice()` 做安全处理。

## SaveFunc

那么就轮到主角 `SaveFunc()` 登场了：

```go
import "github.com/pkg/errors"

type UnsaveFunc func () error

func SaveFunc(fn UnsaveFunc) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = errors.Wrap(r)
        }
    }()

    return fn()
}
```

`SaveFunc()` 其实非常简单，函数名你可以换成你自己喜欢的名字。

它仅仅做一件非常普通的事情，把不安全的函数 fn 传递进来，在这里面执行，即使 fn 发生 panic 也会被 defer 中捕获，然后 recover，使得程序不会被中断。

这样加了一层用于兜底，就不怕不安全的函数会影响外层事务的执行了。

具体使用如下：

```go {hl_lines=[21]}
type UnsaveFunc func () error

func SaveFunc(fn UnsaveFunc) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = errors.Wrap(r)
        }
    }()

    return fn()
}

func UpdateResume(rid Resume, data ResumeData) bool {
    if data == nil {
        return false
    }

    // 启动一个事务更新简历
    err := newTransaction(func(){
        rid.SetData(data)
        if e := SaveFunc(SendNotice()); e != nil {
            log.Println(e)
        }
    })

    if err != nil {
        log.Printf("transaction fail. Cause: ", err)
        return false
    }
    log.Printf("transaction success.")
    return true
}


// 给不安全函数增加一个返回值 error 以符合 UnsaveFunc 定义
func SendNotice() error {
    if Search(ctx) == nil {
        panic("can not get context.")
    }

    notice := new(Notice)
    if !notice.Send() {
        return errors.New("sent notice fail.")
    }
    return nil
}
```

上面的代码将原本的例子中不安全的函数 `SendNotice()` 增加了一个返回值 error 以符合 UnsaveFunc 的定义。

然后在调用的地方 `UpdateResume()` 用 `SaveFunc(SendNotice)` 将函数传递进去。这样即使 `SendNotice()` 发生 panic 也会被 SaveFunc 处理掉，不会影响 `UpdateResume()`。

但这样有三个不好的地方：

1. 它破坏了原本的 `SendNotice()`，是一种侵入性的改造；
2. 它限定了不安全函数 `SendNotice()` 不能携带参数；
3. 它限定了不安全函数 `SendNotice()` 不能返回结果。

这对这几个问题，我们可以用闭包来解决。

## 使用闭包替代侵入性改造

```go {hl_lines=["13-18",28]}
func SendNotice() {
    if Search(ctx) == nil {
        panic("can not get context.")
    }

    notice := new(Notice)
    if !notice.Send() {
        log.Println("sent notice fail.")
    }
    log.Println("sent notice success.")
}

func ClosureSendNotice() UnsaveFunc {
    return func () error {
        SendNotice()
        return nil
    }
}

func UpdateResume(rid Resume, data ResumeData) bool {
    if data == nil {
        return false
    }

    // 启动一个事务更新简历
    err := newTransaction(func(){
        rid.SetData(data)
        if e := SaveFunc(ClosureSendNotice()); e != nil {
            log.Println(e)
        }
    })

    if err != nil {
        log.Printf("transaction fail. Cause: ", err)
        return false
    }
    log.Printf("transaction success.")
    return true
}
```

我们使用一个闭包返回一个 UnsaveFunc，将不安全的函数 `SendNotice()` 放在里面，这样就不会造成侵入性改造，也不需要更改 UnsaveFunc 的定义了。

这样就解决第一个问题。

## 使用闭包解决参数问题

为了说明这个问题，我们先让 `SendNotice()` 需要接收一个参数。


```go {hl_lines=[7, 14, 16, 29]}
func SendNotice(rid Resume) {
    if Search(ctx) == nil {
        panic("can not get context.")
    }

    notice := new(Notice)
    notice.rid = rid
    if !notice.Send() {
        log.Println("sent notice fail.")
    }
    log.Println("sent notice success.")
}
```

这里其实也很简单

```go {hl_lines=[7]}
func SendNotice(rid Resume) {
    if Search(ctx) == nil {
        panic("can not get context.")
    }

    notice := new(Notice)
    notice.rid = rid
    if !notice.Send() {
        log.Println("sent notice fail.")
    }
    log.Println("sent notice success.")
}

func ClosureSendNotice(rid Resume) UnsaveFunc {
    return func () error {
        SendNotice(rid)
        return nil
    }
}

func UpdateResume(rid Resume, data ResumeData) bool {
    if data == nil {
        return false
    }

    // 启动一个事务更新简历
    err := newTransaction(func(){
        rid.SetData(data)
        if e := SaveFunc(ClosureSendNotice(rid)); e != nil {
            log.Println(e)
        }
    })

    if err != nil {
        log.Printf("transaction fail. Cause: ", err)
        return false
    }
    log.Printf("transaction success.")
    return true
}
```

这里其实也很简单，只需要给闭包函数写上 `SendNotice()` 需要的参数，然后闭包中的 `SendNotice()` 直接调用就行。

参数会栈逃逸到堆上，这点不需要担心作用域的问题。

## 使用闭包解决返回值问题

这个问题好像跟最初的场景相悖了。

如果不安全函数 `SendNotice()` 有返回值，那么在 `UpdateResume()` 中的事务有两种情况：

1. 事务中 **不需要** 用到 `SendNotice()` 的返回值
2. 事务中 **需要** 用到 `SendNotice()` 的返回值

```go
func UpdateResume(rid Resume, data ResumeData) bool {
    ......
    
    err := newTransaction(func(){
        rid.SetData(data)
        if e := SaveFunc(ClosureSendNotice(rid)); err != nil {
            log.Println(e)
        }
    })
    
    ......
}
```

如果不需要用到，那么就不需要考虑返回值的事情了。

如果需要用到，说明事务中依赖 `SendNotice()` 的返回值，这样的话，`SendNotice()` 失败不就意味着整个事务失败了吗。那还费劲心机搞个 SaveFunc 干嘛？

所以返回值这点，可以不处理。

照目前这个场景是这种结论，我还没想出其他场景。

如果非要处理返回值这个问题也不是没有办法，但是要连同 SaveFunc 一起改造，也很不正确。

```go
type UnsaveFunc func () (interface{}, error)

func SaveFunc(fn UnsaveFunc) (arg interface{}, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = errors.Wrap(r)
        }
    }()

    return fn()
}

func SendNotice() bool {
    if Search(ctx) == nil {
        panic("can not get resume.")
    }

    notice := new(Notice)
    if !notice.Send() {
        log.Println("sent notice fail.")
        return false
    }
    log.Println("sent notice success.")
    return true
}

func ClosureSendNotice() UnsaveFunc {
    return func () (interface{}, error) {
        res := SendNotice()
        if !res {
            return res, errors.New("fail.")
        }
        return res, nil
    }
}

func UpdateResume(rid Resume, data ResumeData) bool {
    if data == nil {
        return false
    }

    // 启动一个事务更新简历
    err := newTransaction(func(){
        rid.SetData(data)

        res, e := SaveFunc(ClosureSendNotice(rid))
        if e != nil {
            log.Println(e)
        }
        log.Println(res)
    })

    if err != nil {
        log.Printf("transaction fail. Cause: ", err)
        return false
    }
    log.Printf("transaction success.")
    return true
}
```