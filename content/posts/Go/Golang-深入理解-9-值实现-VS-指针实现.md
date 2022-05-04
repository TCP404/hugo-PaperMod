---
title: "Golang [深入理解] 9-值实现 VS 指针实现"
date: 2021-09-14 15:09:50
draft: false
toc: false
images:
categories: ["Golang", "Advanced"]
tags:
  - Golang
  - 值实现
  - 指针实现
math: true
---

都是实现，有啥区别？

<!--more-->

# 值实现 VS 指针实现

值接收者方法（Value receiver method）和指针接收者方法（Pointer receiver method），都可以被结构体变量（struct variable）或结构体指针（struct pointer）调用。

值接收者方法中，对接收者的修改，**不会**影响调用者。
指针接收者方法中，对接收者的修改，**会**影响调用者。

```go
type S struct {
    age int
}

func (s S) read() {	// 值实现，值接收者方法
    s.age = 50
}

func (s *S) write(n int) {	// 指针实现，指针接收者方法
    s.age = n
}

func main() {
    sV := A{age: 10}	// 结构体变量
    sP := &A{age: 10}	// 结构体指针
    
    sV.read()			// 结构体变量可以调用值实现的方法
    fmt.Println(sV.age)	// 10
    sV.write(15)		// 结构体变量可以调用指针实现的方法
    fmt.Println(sV.age)	// 15
    
    sP.read()			// 结构体指针可以调用值实现的方法
    fmt.Println(sP.age)	// 10
    sP.write(18)		// 结构体指针可以调用指针实现的方法
    fmt.Println(sP.age)	// 18
}
```

在上述栗子中，

-   sP 是结构体指针，在 sP 调用值方法 `sP.read()` 时，编译器会自动转换为 `(*sP).read()`；

-   sV 是结构体变量，在 sV 调用指针方法 `sV.write()` 时，编译器会自动转换为 `(&sV).write(15)`。

|                      | **值接收者方法**                | **指针接收者方法**                       |
| -------------------- | ------------------------------- | ---------------------------------------- |
| **结构体变量调用者** | 可以调用，类似于传值            | 可以调用，相当于 `(&sV).write(15)`       |
| **结构体指针调用者** | 可以调用，相当于 `(*sP).read()` | 可以调用，类似与传指针，所以会影响调用者 |
| **修改**             | 不影响调用者                    | 影响调用者                               |

## 如何选择

要看类型的本质。如果一个结构体类型的实例，应该是独一份的，那么就应该用指针接收者方法。

例如，文件结构体。每个文件都是独一份的，对应地，文件结构体返回一个文件对象，这个对象应该唯一的代表这个文件，所以文件结构体的方法应该使用指针接收者方法。

## 涉及接口

在**不涉及**接口的情况下，值实现和指针实现，结构体变量和结构体指针都调用。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo//blog/illustration-pic/Go/IMG/20210903230710.png)

但是在**涉及**接口的情况下，这两种实现还是有区别的。

- 使用值实现接口方法时，等于 “值实现 + 指针实现”；
- 使用指针实现接口方法时，仅等于 “指针实现”。

>值实现 = 值实现 + 指针实现
>
>指针实现 = 指针实现

调用的时候依然是结构体变量和结构体指针都能调值实现方法和指针实现方法，这一点没变。

区别的地方在于，当你要把结构体传递给接口变量时，会有区分。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo//blog/illustration-pic/Go/IMG/20210904000750.png)

举个栗子：

```go
type Ier interface {
    print()
}

type Sa struct {}
type Sb struct {}

func (a Sa) print() {}	// Sa 使用值实现
func (b *Sb) print() {}	// Sb 使用指针实现

func main() {
    sav := Sa{}		// 结构体变量，Sa 使用值实现了 Ier
    sap := &Sa{}	// 结构体指针，Sa 使用值实现了 Ier
    
    sbv := Sb{}		// 结构体变量，Sb 使用指针实现了 Ier
    sbp := &Sb{}	// 结构体变量，Sb 使用指针实现了 Ier
    
    sav.print()		// 结构体变量 可以调用 值实现方法
    sap.print()		// 结构体指针 可以调用 值实现方法
    sbv.print()		// 结构体变量 可以调用 指针实现方法
    sbp.print()		// 结构体指针 可以调用 指针实现方法
    
    var ier1 Ier
    ier1 = sav		// 值实现者 的 结构体变量 可以传递给 接口变量
    ier1 = sap		// 值实现者 的 结构体指针 可以传递给 接口变量
    
    // 指针实现者 的 结构体变量 不能传给 接口变量 
    ier1 = sbv		// cannot use sbv (type Sb) as type Ier in assignment: Sb does not implement Ier (print method has pointer receiver)
    ier1 = sbp		// 指针实现者 的 结构体指针 可以传递给 接口变量
}
```


