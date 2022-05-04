---
title: "C-Note"
tags:
  - C Language
  - note
categories: 
  - [C Language]
  - [Note]
date: 2019-06-09 13:03:26
draft: false
toc: false
images:
math: true
---

C语言笔记


<!--more-->



### 指针

定义指针：`dataType * pointerName = NULL`;
```c
int * ptr_a = NULL;
```
指针赋值：`pointerName = &varName`;
```c
int a = 10;
ptr_a = &a;
```

对于指针，其实就是存放地址的变量

`int a = 10;` 这句话等于申请了一块内存，放了个值10进去，起个别命叫`a` 。那这个空间的地址假设是 0028FF，那`ptr_a = &a` 这句话就是再申请一块内存，把0028FF放进去，起个别名叫`ptr_a` 。

`&` 这个符号是取地址的意思

`*` 这个符号则是取这个指针变量中，那个地址的内存中的 值，也就是直接操作内存，修改内存里的内容







### 值传递  引用传递

值传递：传递一份副本，传递后操作的只是副本，也就是对变量的副本进行操作，变量本身不会变化

```c
void change(int num)
{
    num ++;
}
void main()
{
	int num = 9;
	change(num);
	printf("num = %d", num);
}
// 结果：9
```

引用传递：`&var`， 传递的是地址，传递后操作的是该地址中的值，也就是对变量本身进行操作

```c
void change(int * num)
{
	*num ++;
}
void main()
{
	int num = 9;
    change(&num);
    printf("num = %d", num);
}
// 结果：10
```

值传递就是普通操作，引用传递就是操作指针（骚操作）

### 变量的生存周期

| 存储类型说明符 | 存储类型   | 说明                                                                                           |
|----------------|--------|----------------------------------------------------------------------------------------------|
| auto           | 自动存储   | 只能用于块作用域中的变量，局部变量默认情况下归为自动存储类型                                    |
| register       | 寄存器存储 | 只能用于块作用域中的变量，请求速度快，适用在循环次数多的循环变量/反复使用的变量。                 |
| static         | 静态存储   | 载入程序时创建对象，程序结束时对象消失。                                                         |
| extern         | 外部变量   | 说明符表示生命的变量定义在别处。作用域是整个程序，生存周期贯穿应用程序的开始和结束。跟static一样。 |





### 头文件

```c
#ifndef HEADFILENAME_H_INCLUDE
#define HEADFILENAME_H_INCLUDE
	
	// TODO

#endif
```



