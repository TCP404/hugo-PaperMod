---
title: "Golang [深入理解] 10-实现抽象类"
date: 2021-09-14 15:10:01
draft: false
toc: false
images:
categories: ["Golang", "Advanced"]
tags:
  - Golang
  - 面向对象
  - 抽象类
math: true
---

虽然不推荐，但免不了要用到。

<!--more-->

# 实现抽象类

最近在仿写一个程序的代码，那个程序使用的是「TypeScript」，一种面向对象的、动态的、强类型的语言，而 Golang 是静态的、强类型的、并且没有明确表明是面向对象的一种语言。

Golang 官方对于 Go 是否是面向对象语言的回答是：Yes and no。官方的态度就是我没说我是面向对象，但是你可以按照面向对象的范式去编写程序。

基本上面向对象的“封装、继承、多态、抽象”，在 Go 语言层面都有一定的实现，例如：

-   使用 struct 进行封装
-   在 struct 中使用匿名字段进行继承和多态
-    使用接口实现抽象

但是对于其他一些概念上如：静态、抽象类等则没有直接的语法和机制进行实现。但是还是有一些方法可以实现，今天只讨论怎么实现，不讨论这样实现是否违背 Golang 的设计哲学。编程的范式有很多种，最常见的无非面向过程、面向对象、函数式这三种，三者各有优劣，咱们不作无畏之争。

## 为什么需要抽象类：抽象类本质

### 类
>   在面向对象中，将所有的事物按照其**属性**和**行为**分门别类，也就是面向对象的基本单位之一 —— 类 Class。

例如学校里的同学都是学生，
他们有共同的属性是学号、班级、姓名、年龄，
他们有共同的行为是上课、实训、吃饭、睡觉，
所以可以将属性和行为作为基本元素，组成一个 `学生类`。

再例如学校里的大人都是教职工，
他们有共同的属性是工号、科室、姓名、年龄，
他们有共同的行为是授课、带实训、吃饭、睡觉，
所以可以将属性和行为作为基本元素，组成一个 `教师类`。

### 继承
>   多个类之间的属性、行为会有重复，将这些重复提取出来，提升一层，就成了父类，原本的那些类就称作子类，它们去继承这个父类，就能继续拥有这些重复的属性和行为。这就是继承 extends

例如学生类和教师类都有重复属性：姓名、年龄；重复行为：吃饭、睡觉。

学生和老师共同点是他们都是人，所以将姓名、年龄、吃饭、睡觉提取出来，组成一个新的类 —— `人类`。

然后学生类和教师类继承人类，就能拥有这四个重复的属性和行为。

### 抽象

>   有时候重复的行为虽然做的事情是一样的，但是具体怎么做是不同的。
>
>   那么这时这个行为就是一种抽象的行为。那你在定义这个行为的时候就不能将内容写死。于是就需要声明这个行为是一种抽象行为，给它定义为一个抽象方法。

例如老师的授课行为，都是授课，但是英语老师的授课方式是听说读写、化学老师的授课方式是燃烧爆炸、语文老师的授课方式是之乎者也。

所以我们要把授课声明为一个抽象方法。

>   你也许会觉得我照样可以写死啊，然后语文老师类继承教师类，然后重写授课这个方法，化学老师类继承教师类，然后也重写授课这个方法，这样也行啊。
>
>   是的，这样的确可以，不过教师类中写死的那部分就成了一种“默认行为”。当子类重写的时候，执行重写的行为，没有重写则执行这种“默认行为”。
>
>   不过，当你站在顶层设计的时候，还要为一种抽象行为设置默认，会不会优点顾此失彼哦？

其实当你站在上帝视角设计时，你应该不太愿意陷入细节里，而更愿意把握好大局。我设计一个教师类，我指定它有这些行为，具体怎么实现，谁继承谁负责。就像架构师在设计系统时关注这里用什么技术、那里用什么结构才能让系统更稳定、性能更好，而不是设计到一半陷入到“这个判断这样写是否效率更高”的细节里。所以当有这些抽象行为时，将他们定义为抽象方法更好。

而从另一个角度来说，任何人，只要他有具体的授课内容，即使他没有工号、没有科室，但他也是教书育人、为人师表不是么？例如孔子，他没有工号、没有科室，但是他教书育人，弟子三千圣贤七十二，不也被尊为至圣先师嘛。

所以这也是一个稍微哲学的问题，它从更加开放的角度去评判，一个人是否是老师，不是看他有没有教师资格证，有没有工号，有没有科室，只要他实现具体的授课方式，他就是一个老师。

到这里我想你应该能理解 Golang 中，一个结构体 S 只要实现了接口 I 中的所有方法，就表示 S 实现了 I，A 是一个 I 类型。

一个司机结构体 `Driver`，只要实现了教师接口 `Teacher` 中的授课方法 `teach()`，
那么 `Driver` 就是一个 `Teacher`，他就是一个教练，
`Driver` 就是一个 `Teacher` 类型。

### 接口
>   所以在设计中，我们总会倾向于去定义一些规则，也就是这些抽象行为，然后让实现者自己来实现。这就是接口——interface。

在 Golang 和 面向对象的典型代表 Java 中都有对接口的支持。在 Java 中一个类实现某个接口时要显式的使用 `implements` 关键字，在 Golang 中则是某个接口体具有接口中的所有方法的具体实现即可。一种是侵入式的，一种是非侵入式的。

### 抽象类
>   有时候我就是想在设计某个类型的时候，它有的行为是抽象的，我不指定它的默认行为；而有的行为我就给它个默认行为。
>
>   那么这种时候可以使用抽象类。接口是绝对抽象的，类是绝对具象的，而抽象类则是在绝对抽象和绝对具象之间。

就像第一句话说的，有些行为我就想给它个默认行为，有些则不想。而接口不会有具体实现，类必须有具体实现，而抽象类则是可以有具体实现，也可以没有具体实现。

例如教师类有授课、打卡上班两种行为。授课方式多种多样，打卡上班方式也多种多样。那我现在只声明授课是一种抽象行为，谁继承谁自行实现，再给打卡上班指定一种默认行为方式 —— 指纹打卡。

如此当 语文老师类 继承了 教师类 以后，必须实现授课的具体行为，也就是实现抽象方法；至于打卡上班行为，可以重写个人脸识别打卡，也可以不重写，那就默认指纹打卡。

**所以抽象类更像一种中庸之道。取绝对抽象和绝对具象的折中，增加了程序的灵活性。**

### 小结

在设计时，将一类事物的共有的属性和行为提取出来，定义为一类。

将多个类中重复的部分剥离出来，定义为父类，让子类去继承。

为一类事物定性时可以不关注具体其具体行为，只关注其有没有这个行为即可。这些行为称为抽象行为，即抽象方法。将这些抽象行为封装起来，就是一个接口。

为一类事物定性时，想要给一些抽象行为设置默认行为，就改用抽象类。

## Golang 实现抽象类

Golang 中的接口是非侵入式的，接口中的方法是没有具体实现的，抽象类是可以有具体实现的，于是将接口和结构体结合起来，实现抽象类。

抽象类分两种情况：具有默认行为的抽象方法使用到 `this` 的 和 没使用到的。

### 没使用到 this 的情况

```go
// 定义抽象类
type IAbsClass interface {	// 使用接口定义抽象行为
    abs()					// 抽象方法，需要子类实现
}

type AbsClass struct {}				// 定义抽象类结构体
func (this AbsClass) common()  {	// 具有默认行为的抽象方法，没使用到 this
    fmt.Println("AbsClass common()")
}

// --------------------------- 分割线 -----------------------------
// 子类1继承抽象类
type SubClass1 struct {
    AbsClass
    name string
}
func (this SubClass1) abs() {				// 子类实现抽象方法
    fmt.Println("SubClass1 implement abs().")
}
func NewSubClass1(name string) *SubClass1 {	// 子类构造方法
    s := new(SubClass1)
    s.name = name
    return s
}

// 子类2继承抽象类
type SubClass2 struct {
    AbsClass
    name string
}
func (this SubClass2) abs() {		// 子类实现抽象方法
    fmt.Println("SubClass2 implement abs().")
}
func (this SubClass2) common() {	// 子类重写抽象方法
    fmt.Println("SubClass2 overwrite common().")
}
func NewSubClass2(name string) *SubClass2 {	// 子类构造方法
    s := new(SubClass)
    s.name = name
    return s
}
```

通过接口 `IAbsClass` 定义抽象方法 `abs()`，然后定义一个配对的结构体 `AbsClass`，结构体实现那些具有默认行为的抽象方法 `common()`。

然后子类 `SubClass1` 和 `SubClass2` 继承抽象类，也就是嵌入匿名字段 `AbsClass`，然后自己实现抽象方法 `abs()` 。

而且 `SubClass2` 还对有默认行为的 `common()` 不满意，自己进行了重写。

使用方法如下：

```go
func main() {
    sub1 := NewSubClass1("Boii")
    sub2 := NewSubClass2("Eva")
    
    sub1.abs()		// "SubClass1 implement abs()."
    sub2.abs()		// "SubClass2 implement abs()."
    
    sub1.common()	// "AbsClass common()"
    sub2.common()	// "SubClass2 overwrite common()."
}
```

因为 `SubClass2` 对具有默认行为的抽象方法 `common()` 进行了重写，所以调用时调的是重写的方法。

### 使用到 this 的情况

Golang 中本身并不是明确支持继承，所以如果我们为抽象方法定义默认行为是需要使用到 this 的话，Golang 不会做自动转换，它会认为这个 this 就是 `AbsClass` 本身，而不是 `SubClass1` 或 `SubClass2`。

基于这种情况，我们需要 3 步：

1.   在 `AbsClass` 中加入一个接口字段 `i IAbsClass`
2.   在使用 this 的时候换成 `this.i`
3.   由于 i 是个接口类型，子类 `SubClass1、SubClass2` 在实例化时得将自己注册进去。

``` go
// 抽象类
type IAbsClass interface {
    abs()
}

type AbsClass struct {
    i IAbsClass	// == 增加一个接口字段 == 
}

func (this AbsClass) common1() {	// 具有默认行为的抽象方法，没使用到 this
    fmt.Println("AbsClass common()")
}
func (this AbsClass) common2() {	// 具有默认行为的抽象方法，使用到了 this
    this.i.abs()	// == 不能直接 this，要用 this.i ==
}

// --------------------------- 分割线 -----------------------------
// 子类1继承抽象类
type SubClass1 struct {
    AbsClass
    name string
}
func (this SubClass1) abs() {	// 子类实现抽象方法
    fmt.Println("SubClass1 implement abs().")
}
func NewSubClass1(name string) *SubClass1 {	// 子类构造方法
    s := new(SubClass1)
    s.AbsClass.i = s	// == 将自己注册进去 ==
    s.name = name
    return s
}

// 子类2继承抽象类
type SubClass2 struct {
    AbsClass
    name string
}
func (this SubClass2) abs() {// 子类实现抽象方法
    fmt.Println("SubClass2 implement abs().")
}
func NewSubClass2(name string) *SubClass2 {	// 子类构造方法
    s := new(SubClass)
    s.name = name
    s.AbsClass.i = s	// == 将自己注册进去 ==
    return s
}
```

这样通过子类 `SubClass1、SubClass2` 调用 `common()` 时才能正确的执行。

```go
func main() {
    sub1 := NewSubClass1("Boii")
    sub2 := NewSubClass2("Eva")
    
    sub1.common2()	// "SubClass1 implement abs()."
    sub2.common2()	// "SubClass2 implement abs()."
}
```



## 总结

通过有无这种行为去定性一个类，而不是通过如何实现这种行为去定性，这就是面向接口编程。

将一些行为聚合在一起，不实现它们，就是一个接口。如果要给这些行为一个默认行为，那就用抽象类。

Java 中的抽象类和接口都支持成员变量，但在 Golang 中实现抽象类时，无法兼顾成员变量，因为接口不可以有变量。虽然可以把抽象类的成员变量放在抽象类结构体中，也可以在子类中访问到，但是无法做到泛化：

```go
func test(ia IAstClass) {
    fmt.Println(ia.name)	// 编译失败，接口中没有变量
}

func test(a AstClass) {
    fmt.Println(a.name)		// 编译失败，无法将子类传进来 test(sub1)
}

func test(ia IAstClass) {
    fmt.Println(ia.(AstClass).name)	// 编译失败，test(sub1) 会报错说断言失败
}
```

当然可以在抽象类接口中，为抽象类结构体增加 getter、setter 方法，然后抽象类结构体实现 getter、setter 方法，但其实也不是好。

```go
// 抽象类
type IAbsClass interface {
    abs()
    getAbsName() string
}

type AbsClass struct {
    i IAbsClass	// == 增加一个接口字段 == 
    absName string
}

func (a AbsClass) getAbsName() string {
    return a.absName
}

// --------------------------- 分割线 -----------------------------

func test(ia IAstClass) {
    fmt.Println(ia.getAstName())
}
```

属性多了可以将所有的 getter、setter 放在一个接口 `IGetAstAttr` 中，然后 `IAstClass` 中嵌入这个 `IGetAstAttr`，或者更进一步使用选项设计模式管理所有 getter、setter。

```go
// 抽象类
type IAbsClass interface {
    IGetAstAttr
    abs()
}

type IGetAstAttr interface {
    getAbsName()  string
    getAbsAge()   int
    getAbsEmail() string
    
    setAbsName()  string
    setAbsAge()   int
    setAbsEmail() string
}

type AbsClass struct {
    i IAbsClass	// == 增加一个接口字段 == 
    // 抽象类的成员变量
    absName  string
	absAge   int
    absEmail string
}

func (a AbsClass) getAbsName() string {
    return a.absName
}

// --------------------------- 分割线 -----------------------------

func test(ia IAstClass) {
    fmt.Println(ia.getAstName())
}
```

但是我总觉得这样违背了 Golang 的设计哲学。我也不喜欢这种脱离本质的东西。如果已经到了这种程度，倒不如重新设计，看看如何通过组合替代继承。
