---
title: "Java-反射"
tags:
  - Java
  - 反射
categories: [Java]
date: 2019-06-09 00:58:48
draft: false
toc: false
images:
math: true
---

Java 神器：反射

<!-- more -->

```java
Person p = new Person();

/**获取*/
Class<?> clazz = p.getClass(); // 得知道类对象
Class<?> clazz =  Person.class;	//得知道类名
Class<?> clazz = Class.forName("com.tcp404.Person");//得知道类的全路径名


/** 获取方法 */
Method method = clazz.getDeclaredMethod("方法名", new Class(){[参数类型.class, 参数类型.class, ...]});
Methods[] methods = clazz.getDeclaredMethods();

/** 获取属性 */
Field field = clazz.getDeclaredField("属性名字");
Field[] fields = clazz.getDeclaredFields();

/** 获取构造器 */
Constructor<?> con = clazz.getDeclaredConstructor(new Class(){[参数类型.class, 参数类型.class, ...]});
Constructor<?>[] cs = clazz.getDeclaredConstructors();
```