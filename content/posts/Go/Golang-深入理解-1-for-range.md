---
title: "Golang [深入理解] 1-for & range"
date: 2021-09-14T15:06:47
draft: false
toc: false
mathjax: true
abbrlink: b56ec5ea
categories: ["Golang", "Advanced"]
tags:
  - Golang
  - 性能比较
---

两种遍历方式的使用与区别

<!--more-->

# for 和 range 区别

range 可以用来遍历 “引用六兄弟” 中的三个：slice、map、chan，加上数组 array。

对于数组和 slice，还可以用 for 来遍历。

## 基本使用

### slice/array

>   -   对于 range 循环，循环次数会在循环开始前计算好，在循环过程中修改切片长度并不会改变本次循环的次数。
>   -   对于空切片，循环次数为 0。

```go
s := []int{11, 99, 16, 55}

for i, v := range s {
    fmt.Println(i, v)
}

// Output:
0 11
1 99
2 16
3 55
```

这种方式中，v 是 s 中元素的一份拷贝，对 v 的修改并不会影响 s 中原本的元素。

```go
s := []int{11, 99, 16, 55}

for i, v := range s {
    v += 10		// v 只是一份拷贝，并不会影响 s 里的元素
}

fmt.Println(s)

// Output:
[11 99 16 55]
```

除非 s 中的元素是指针，则对 v 的修改会影响 s，下面将会讲到。

此外采用下面两种方式也可以修改 s 中的元素。

```go
s := []int{11, 99, 16, 55}

for i := 0; i < len(s); i++ {
    s[i] += 10
}

fmt.Println(s)

// Output:
[21 109 26 65]
```

```go
s := []int{11, 99, 16, 55}

for i := range s {
    s[i] += 10
}

fmt.Println(s)

// Output:
[21 109 26 65]
```



### map

>   -   迭代过程中，删除还没遍历到的键值对，则该键值对不会被遍历。
>
>   -   迭代过程中，如果创建新的键值对，则新的键值对有可能被遍历，也有可能不会被遍历。
>
>   -   对于空字典，遍历次数为 0。

```go
m := map[byte]int{
	11: 10,
	22: 20,
	33: 30,
	44: 40,
}

for k, v := range m {
	v += 5	// 仅仅是对拷贝的修改，不会影响 m
	fmt.Println(k, v)
}

fmt.Println(m)

// Output：
22 25
33 35
44 45
11 15
map[11:10 22:20 33:30 44:40]
```

同样这种方式对 v 进行修改是不会影响 m 的。

下面这种就可以

```go
m := map[byte]int{
	11: 10,
	22: 20,
	33: 30,
	44: 40,
}

for k := range m {
	m[k] += 1	// 这种方式就会修改到 m 的元素
	fmt.Println(k, m[k])
}

fmt.Println(m)

// Output:
11 11
22 21
33 31
44 41
map[11:11 22:21 33:31 44:41]
```



### chan

>-   发送给信道的值，可以通过 range 遍历，直到信道被关闭，才会退出循环。
>
>-   如果是未分配内存的 nil 信道，循环将永远阻塞。

```go
ch := make(chan byte)

go func() {
    ch <- 'b'
    ch <- 'o'
    ch <- 'i'
    ch <- 'i'
    close(ch)
}()

for c := range ch {
    fmt.Println(string(c))
}

// Output:
b
o
i
i
```

## 性能比较

接下来我们主要针对基本类型和结构体类型，比较他们使用 for 和 range 的性能。

只有 slice 和 array 才有办法 for 和 range 都使用，所以接下来的比对如下：

|              | for            | range            |
| ------------ | -------------- | ---------------- |
| **基本类型** | for + []int    | range + []int    |
| **结构体**   | for + []struct | range + []struct |

### `for + []int` VS `range + []int`

主要方式有 3 种：

-   `for i:=0; i < len(s); i++`
-   `for i := range s`
-   `for i, v := range s`

>先说结论：
>
>遍历 `[]int` ，三种方式并没太大差别。

#### []int

```go
// 生成切片
func geneI(n int) []int {
	nums := make([]int, 0, n)
	rand.Seed(time.Now().Unix())
	for z := 0; z < n; z++ {
		nums = append(nums, rand.Int())
	}
	return nums
}

// for 遍历
func I_For(nums []int) {
	n := len(nums)
	for i := 0; i < n; i++ {
		_ = nums[i]
	}
}

// range i 遍历
func I_RangeI(nums []int) {
	for i := range nums {
		_ = nums[i]
	}
}

// range iv 遍历
func I_RangeIV(nums []int) {
	for i, v := range nums {
		_, _ = i, v
	}
}

func BenchmarkI_For(b *testing.B) {
	nums := geneI(1 << 20)
	for i := 0; i < b.N; i++ {
		I_For(nums)
	}
}

func BenchmarkI_RangeI(b *testing.B) {
	nums := geneI(1 << 20)
	for i := 0; i < b.N; i++ {
		I_RangeI(nums)
	}
}

func BenchmarkI_RangeIV(b *testing.B) {
	nums := geneI(1 << 20)
	for i := 0; i < b.N; i++ {
		I_RangeIV(nums)
	}
}
```

执行后的结果如下：

```bash
$ go version 
go1.16.6 linux/amd64

$ go test . -bench=I_ -benchmem
goos: linux
goarch: amd64
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkI_For-4            2685            431122 ns/op            3124 B/op          0 allocs/op
BenchmarkI_RangeI-4         2686            431759 ns/op            3123 B/op          0 allocs/op
BenchmarkI_RangeIV-4        2688            432650 ns/op            3120 B/op          0 allocs/op
PASS
ok   3.586s
```

从结果可以看出，在遍历基本类型切片时，不管是时间上（ns/op），还是空间上（B/op），还是系统调用上（allocs/op），都没啥差别。

>   现在是 2021-8-19，以上是 go 1.16.6 版本测试的结果，并没有太大差别，不像网上说的 for 比 range 快。

#### []struct

```go
const N = 1 << 20
type Per struct {
	ins [2048]byte
	age int
}

func geneS(n int) []Per {
	persons := make([]Per, 0, n)
	rand.Seed(time.Now().Unix())
	for i := 0; i < n; i++ {
		persons = append(persons, Per{age: rand.Int(), ins: [2048]byte{'a'}})
	}
	return persons
}

func S_For(persons []Per) {
	n := len(persons)
	for i := 0; i < n; i++ {
		_ = persons[i].age
	}
}

func S_RangeI(persons []Per) {
	for i := range persons {
		_ = persons[i].age
	}
}

func S_RangeIV(persons []Per) {
	for i, v := range persons {
		_, _ = i, v.age
	}
}

func BenchmarkS_For(b *testing.B) {
	persons := geneS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		S_For(persons)
	}
}
func BenchmarkS_RangeI(b *testing.B) {
	persons := geneS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		S_RangeI(persons)
	}
}
func BenchmarkS_RangeIV(b *testing.B) {
	persons := geneS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		S_RangeIV(persons)
	}
}
```

运行结果

```bash
$ go test . -bench=S_ -benchmem
goos: linux
goarch: amd64
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkS_For-4            2850            421009 ns/op               0 B/op          0 allocs/op
BenchmarkS_RangeI-4         2850            420993 ns/op               0 B/op          0 allocs/op
BenchmarkS_RangeIV-4        2848            421131 ns/op               0 B/op          0 allocs/op
PASS
ok     7.330s
```

可以看到基本也没太大区别。

#### []*struct

>   结论：即使是结构体指针切片，跑出来的结果几乎相差无几

```go
func PS_For(persons []*Per) {
	n := len(persons)
	for i := 0; i < n; i++ {
		_ = persons[i].age
	}
}

func PS_RangeI(persons []*Per) {
	for i := range persons {
		_ = persons[i].age
	}
}

func PS_RangeIV(persons []*Per) {
	for i, v := range persons {
		_, _ = i, v.age
	}
}

func BenchmarkPS_For(b *testing.B) {
	persons := genePS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		PS_For(persons)
	}
}

func BenchmarkPS_RangeI(b *testing.B) {
	persons := genePS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		PS_RangeI(persons)
	}
}
func BenchmarkPS_RangeIV(b *testing.B) {
	persons := genePS(N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		PS_RangeIV(persons)
	}
}
```

运行结果：

```bash
$ go test . -bench=S_ -benchmem  
goos: linux
goarch: amd64
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkS_For-4                83856697                14.13 ns/op            0 B/op          0 allocs/op
BenchmarkS_RangeI-4             66149228                17.66 ns/op            0 B/op          0 allocs/op
BenchmarkS_RangeIV-4            72271676                16.61 ns/op            0 B/op          0 allocs/op
BenchmarkPS_For-4               71399964                21.43 ns/op            0 B/op          0 allocs/op
BenchmarkPS_RangeI-4            63293829                18.62 ns/op            0 B/op          0 allocs/op
BenchmarkPS_RangeIV-4           63576286                18.57 ns/op            0 B/op          0 allocs/op
PASS
ok       7.565s
```

## 能修改元素的遍历

前面说 `for i, v := range s` 这种方式，v 是一份拷贝，对 v 的修改不会影响 s 中的元素。

但是如果 v 是个指针，对 v 的修改还是会影响的。

```go
a := 10
b := 20
c := 30

s := []*int{&a, &b, &c}

fmt.Println(a, b, c)
for _, v := range s {
    *v += 1
}
fmt.Println(a, b, c)

// Output：
10 20 30
11 21 31
```

再举个结构体的栗子：

```go
type Person struct {
    name string
    age  int
}

persons := make([]*Person, 0, 3)
persons = append(persons, &Person{age: 15}, &Person{age: 25}, &Person{age: 35})

fmt.Println(persons[0], persons[1], persons[2])
for _, v := range persons {
    v.age += 1
}
fmt.Println(persons[0], persons[1], persons[2])

// Output:
&{ 15} &{ 25} &{ 35}
&{ 16} &{ 26} &{ 36}
```

## 总结

遍历总共有 3 种方式：

1.   **for**：`for i:=0; i<n; i++`
2.   **rangeI**：`for i := range items`
3.   **rangeIV**：`for i, v := range items`

三种方式都各有所长。其中 rangeIV 中 v 是 items 中元素的一份拷贝，如果 v 不是指针，那么对于 v 的修改并不会影响 items 里面的元素，如果 v 是指针，则能够影响。

正是因为 v 是一份拷贝，所以在一些旧版本中可能会因此降低了性能，但在新版本（至少 go 1.16.6 ，我并没有去求证）中，从基准测试来看是没有什么区别，可能在新版本中做了优化。


