---
title: "SSM-5-拦截器"
tags:
  - Java
  - SSM
  - 拦截器
categories:
  - Java
  - SSM
date: 2019-07-07 12:04:18
draft: false
toc: false
images:
math: true
---

拦截器其实就是一种中间件

<!-- more -->

## 摘要

拦截器 Interceptor 主要用于在 `请求到达控制器之前进行验证` ，有点类似过滤器。主要用于拦截用户请求并做出相应的处理。通常应用在**权限验证，记录请求信息的日志，判断用户是否登录等**功能上。

主要的实现方法有两种：

1. 通过实现 `HandlerInterceptor接口` 或继承 `HandlerInterceptor接口` 的实现类来定义。
2. 通过实现 `WebRequestInterceptor接口` 或继承`WebRequestInterceptor接口` 的实现类来定义。

## 效果图

拦截器的拦截时间点在这里面分三种：

1. 请求到达控制器之前调用，比如点击我的购物车的时候，判断用户有没有登录先
2. 控制器调用之后，返回视图之前。
3. 解析试图之后。比如访问了XXX的QQ空间，就在视图解析完后记录到访问记录中

\>_

![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-4/1.png)


这里的案例只是在这些方法里打印了一些话来表示出拦截器在什么时间点做了什么

## 使用步骤

### 1. 配置

先在springmvc-servlet.xml中配置拦截器。

```xml
<mvc:interceptors>
    <bean class="interceptor.AllInterceptor"/>
</mvc:interceptors>
```

这样的配置方式默认对所有请求拦截，也就是任何请求要到达控制器之前都会经过拦截器。



这种方式的话就是拦截所有的请求，但是请求如果是到`/abc`这个控制器的话就例外，不拦截。

```xml
<mvc:interceptors>
    <mvc:interceptor>
       <mvc:mapping path="/**"/>
       <mvc:exclude-mapping path="/abc"/>
        <bean class="interceptor.Interceptor1"/>
    </mvc:interceptor>
</mvc:interceptors>
```

这种呢就是对发送到`/6`这个控制器的请求，进行指定的拦截器拦截。

```xml
<mvc:interceptors>
   <mvc:interceptor>
        <mvc:mapping path="/6"/>
        <bean class="interceptor.Interceptor2"/>
   </mvc:interceptor>
</mvc:interceptors>
```



### 2. 实现接口

HandlerInterceptor接口中已经有默认的实现方法了，所以implements之后并不会提醒重写方法。

preHandle：在**请求到达控制器之前**进行拦截

postHandle：**控制器调用之后，解析试图之前**进行拦截

afterCompletion：在**解析试图之后**进行拦截



```java
package interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AllInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle，请求到达控制器之前");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle,控制器调用之后，解析试图之前");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion,解析视图之后");
    }
}
```

## 总结

拦截器是一个比较重要的东西。使用也简单，主要是两点

1. 编写拦截器的工作，preHandle要干嘛，postHandle要干嘛，afterCompletion要干嘛。
2. 在springmvc配置文件里编写拦截规则，是全部都拦截用一个全局拦截器？还是某个控制器用某个拦截器？