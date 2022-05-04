---
title: "SSM-7-统一异常处理"
tags:
  - Java
  - SSM
  - 统一异常处理
categories:
  - Java
  - SSM
date: 2019-07-11 20:55:10
draft: false
toc: false
images:
math: true
---

遇到异常不要慌～

<!-- more -->

## 摘要

程序运行过程中总会遇到一些可预知的，不可预知的异常。如果不对这些异常进行捕捉和处理，就会导致程序崩溃、停止运行、闪退等诸多令人体验极差的现象。如果对这些异常一个一个单独处理，则代码显得很臃肿，耦合度高（独立性差），所以要用统一异常处理对所有的代码进行异常处理

SpringMVC处理异常有三种方式：

1. 简单异常处理SimpleMappingExceptionResolver
2. 实现HandlerExceptionResolver接口自定义异常
3. 使用@ExceptionHandler注解实现异常处理

在程序中可能出现异常的地方进行捕捉，程序发生异常被捕捉到后，就会调用我们编写的统一异常处理类进行处理

## 效果图

![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-7/1.png)


## 使用步骤

由于我们太完美了程序暂时没有什么异常，所以我们得自己制造点异常。

我们先做一个列表，把控制层，业务层，模型层的各种异常集中链接在这里。

```jsp
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page isELIgnored="false" %>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";
%>
<html>
<head>
    <base href="<%=basePath%>"/>
    <title>Exception</title>
</head>
<body>
<div id="body">
    <h3><a href="db">控制器中数据库异常</a></h3>
    <h3><a href="my">控制器中自定义异常</a></h3>
    <h3><a href="no">控制器中未知异常</a></h3>
</div>
</body>
</html>

```



如下图：

![2](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-7/2.png)


这些链接点击后跳转到控制层里进行处理。而程序不会无端端的就异常，所以我们得先在控制器里弄点异常：自己**主动**抛出一个*异常* 。

### 先主动搞个异常

```java
        throw new SQLException("控制器中数据库异常");
```

这句话就是主动抛出一个异常。

整体的代码如下：

```java
@Controller
public class ExceptionController{
    @RequestMapping("/db")
    public void db() throws Exception {
        throw new SQLException("控制器中数据库异常");
    }
    @RequestMapping("/my")
    public void my() throws Exception {
        throw new MyException("控制器中自定义异常");
    }
    @RequestMapping("/no")
    public void no() throws Exception {
        throw new Exception("控制器中未知异常");
    }
}
```

好了制造好了异常，现在开始来处理。一共有三种方法

### 实现HandlerExceptionResolver接口

#### 1.编写一个类实现HandlerExceptionResolver接口

异常抛出后，就会被捕捉，捕捉后就要进行处理啦。我们编写一个类，这个类要实现 `HandlerExceptionResolver接口`里的 `resolveException()` 方法。

```java
public class MyExceptionHandler implements HandlerExceptionResolver {

    /**
     * 重写接口方法
     *
     * @param arg0 servlet请求
     * @param arg1 servlet响应
     * @param obj  目标对象
     * @param ex   异常
     * @return 返回相应的视图
     */
    @Override
    public ModelAndView resolveException(HttpServletRequest arg0, HttpServletResponse arg1, Object obj, Exception ex) {
        Map<String, Object> model = new HashMap<>();
        model.put("ex", ex);
        // 根据不同错误转向不同页面（统一处理），即异常与view的对应关系
        
        // 如果异常属于自定义异常类型
        if (ex instanceof MyException) {
            return new ModelAndView("exception/my-error", model);
            // 如果异常属于数据库异常类型
        } else if (ex instanceof SQLException) {
            return new ModelAndView("exception/sql-error", model);
            // 如果异常属于未知类型
        } else {
            return new ModelAndView("exception/error", model);
        }
    }

}
```

控制层里制造了三种不同的异常 ，这些异常被捕捉后会被送到这里进行处理。这里主要就是根据异常的不同类型返回不同的视图页面。

### 3. 在SpringMVC配置文件中配置

```xml
<bean class="exception.MyExceptionHandler"/>
```



### 使用@ExceptionHandler注解实现异常处理

这种是最容易的一种方式，集成简单，扩展性好。

#### 1.创建一个BaseController类

创建一个抽象类，并在类中使用@ExceptionHandler注解声明异常处理方法

```java
public abstract class BaseController {
    /**
     * 异常处理方法
     * @param request Servlet请求
     * @param ex 异常实例
     * @return 返回一个视图页面的名称
     */
    @ExceptionHandler
    public String exception(HttpServletRequest request, Exception ex){
        request.setAttribute("ex", ex);
        if (ex instanceof SQLException) {
            return "404";
        } else if ( ex instanceof MyException) {
            return "exception/my-error";
        } else {
            return "exception/error";
        }
    }
}
```

#### 2.让其他控制器继承这个类

让其他需要处理异常的类继承这个BaseController类

```java
public class ExceptionController extends BaseController{
    ......
}
```

#### 3. 注意使用这种方法是配置文件中不要配置任何其他异常处理方法。

**注意使用这种方法是配置文件中不要配置任何其他异常处理方法！！！**

**注意使用这种方法是配置文件中不要配置任何其他异常处理方法！！！**

**注意使用这种方法是配置文件中不要配置任何其他异常处理方法！！！**

### 简单异常处理SimpleMappingExceptionResolver

```xml
	<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <!--定义默认的异常处理页面，当该异常类型注册时使用-->
        <property name="defaultErrorView" value="error"/>
        <!--定义异常处理页面用来获取异常信息的变量名，默认名为exception-->
        <property name="exceptionAttribute" value="ex"/>
        <!--定义需要特殊处理的异常，用类名活完全路径名为key，异常页名为值-->
        <property name="exceptionMappings">
            <props>
                <prop key="exception.MyException">my-error</prop>
                <prop key="java.sql.SQLException">sql-error</prop>
                <!--这里还可以继续扩展对不同异常类型的处理-->
            </props>
        </property>
    </bean>
```

在**SpringMVC配置文件**中配置 `SimpleMappingExceptionResolver异常处理器` ，即可实现对异常的统一处理。

主要分默认异常处理页面，获取异常信息的变量名，还有一些需要特殊处理的异常。

## 总结

最简单的使用@ExceptionHandler注解的方法，虽然有集成简单，可扩展性好等优点，但是该方法对已有代码存在入侵性，需要修改已有代码，是相关类继承于BaseController。

而第一种写一个HandlerExceptionResolver的实现类的方法则比较温和，不会对代码进行入侵。记得要在SpringMVC配置文件中托管这个实现类：`<bean class="exception.MyExceptionHandler"/>`

最后一种其实最方便啦，配置好就OK，不用增加什么文件。

