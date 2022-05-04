---
title: "SSM-4-数据绑定"
tags:
  - Java
  - SSM
  - 数据绑定
categories:
  - Java
  - SSM
date: 2019-07-07 10:04:18
draft: false
toc: false
images:
math: true
---

先把数据绑定起来

<!--more-->

## 摘要

数据绑定是将用户参数输入值绑定到领域模型（POJO）的一种特性，在SpringMVC的Controller和View参数传递数据的过程中，所有HTTP请求参数的类型均为字符串，如果模型需要绑定的类型为double或int，则需要手动进行类型转换，而有了数据绑定后：

1. 不需要手动转型数据了。
2. 当表单输入验证失败时，会重新生成一个HTML表单，无需重新填写已输入字段。

## 效果图

第一次加载的样子

![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-4/1.png)


验证失败后重新加载的样子，前面填过的数据依然保留着

![2](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-4/2.png)


验证成功后进入这个页面

![3](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-4/3.png)




## 使用步骤

### 1. 导入jar包

maven项目：配置pom.xml

```xml
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
</dependency>
```

非maven项目：将下载好的 `taglibs` 和 `jstl` 的jar放到 `webapps\WEB-INF\lib` 目录下

### 2. web.xml中配置编码过滤器

```xml
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



### 3. 编写领域模型

定义各种属性和getter&setter方法，太简单，不赘述

### 4. 编写控制层Controller

因为视图层用的都是表单标签库，所以数据要在controller层定义

然后通过model.addAttribute()方法将数据传送出去

```java
@Controller
@RequestMapping("/four")
public class FourController {
    private static final Log logger = LogFactory.getLog(FourController.class);

    private final FourUserService fourUserService;
    @Autowired
    public FourController(FourUserService fourUserService) {
        this.fourUserService = fourUserService;
    }

    /**
     * 第一次加载表单会触发这个方法
     */
    @RequestMapping("/input")
    public String inputUser(Model model) {
        HashMap<String, String> hobbys = getMap();
		// 虽然是第一次加载，但是也要在model添加一个领域模型的对象，否则表单标签无法找到modelAttribute属性指定的form backing object
        model.addAttribute("user", new User());
        
        model.addAttribute("hobbys", hobbys);
        model.addAttribute("carrers", new String[]{"教师", "学生", "码农", "民工"});
        model.addAttribute("houseRegister", new String[]{"北京", "上海", "广州", "深圳"});
        return "userAdd";
    }

    /**
     * 表单提交的时候会触发这个方法
     */
    @RequestMapping("/save")
    public String addUser(@ModelAttribute User user, Model model) {
        // 进入业务层验证业务逻辑
        if (fourUserService.addUser(user)) {
            logger.info("成功");
            // 重定向到本控制器里的list控制方法
            return "redirect:list";
        }
        logger.info("失败");
        // 验证失败的时候就到了这里，相当于重新加载，但是会带着已经填过的数据，用户就不用再次填写已经填写过的输入，注意这里没有在model.addAttribute()中new bean()。看不懂请对比上面的input方法
        HashMap<String, String> hobbys = getMap();
        model.addAttribute("hobbys", hobbys);
        model.addAttribute("carrers", new String[]{"教师", "学生", "码农", "民工"});
        model.addAttribute("houseRegister", new String[]{"北京", "上海", "广州", "深圳"});
        return "userAdd";
    }

    @RequestMapping("/list")
    public String listUsers(Model model) {
        List<User> users = fourUserService.getUsers();
        model.addAttribute("users", users);
        return "userList";
    }

    private HashMap<String, String> getMap() {
        HashMap<String, String> hobbys = new HashMap<>(5);
        hobbys.put("篮球", "篮球");
        hobbys.put("乒乓球", "乒乓球");
        hobbys.put("电玩", "电玩");
        hobbys.put("游泳", "游泳");
        return hobbys;
    }
}
```



### 5. 编写业务逻辑层Service

业务层，业务逻辑都在这里判断

```java
package service.four;

import org.springframework.stereotype.Service;
import po.four.User;

import java.util.ArrayList;

/**
 * @author pr919
 */
@Service
public class FourUserServiceImpl implements FourUserService {
    private static ArrayList<User> users = new ArrayList<>();

    @Override
    public boolean addUser(User u) {
        String magicValue = "民工";
        if (!magicValue.equals(u.getCarrer()) && !"".equals(u.getUsername())){
            users.add(u);
            return true;
        }
        return false;
    }

    @Override
    public ArrayList<User> getUsers() {
        return users;
    }
}

```



### 6. 编写视图层View

视图层的**表单标签**通过**控制层**和**领域模型**绑定在一起，实现了数据绑定。

```jsp
<%-- userAdd.jsp --%>

<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%--
  Created by IntelliJ IDEA.
  User: pr919
  Date: 2019.6.24
  Time: 14:41
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page isELIgnored="false" %>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";
%>
<html>
<head>
    <base href="<%=basePath%>"/>
    <title>dataBind</title>
</head>
<body>
<%--@elvariable id="user" type="po.four.User"--%>
<form:form modelAttribute="user" mothod="post" action="four/save">
    <fieldset>
        <legend>添加一个用户</legend>
        <p>
            <label>用户名：</label>
            <form:input path="username"/>
        </p>
        <p>
            <label>爱好：</label>
            <%--@elvariable id="hobbys" type="java.util.List"--%>
            <form:checkboxes items="${hobbys}" path="hobby"/>
        </p>
        <p>
            <label>朋友：</label>
            <form:checkbox path="friends" value="张三"/>1
            <form:checkbox path="friends" value="李四"/>2
            <form:checkbox path="friends" value="王五"/>3
            <form:checkbox path="friends" value="赵六"/>4
        </p>

        <p>
            <label>职业：</label>
            <form:select path="carrer">
                <option></option>
                请选择职业
                <%--@elvariable id="carrers" type="java.util.List"--%>
                <form:options items="${carrers}"/>
            </form:select>
        </p>
        <p>
            <label>户籍：</label>
            <form:select path="houseRegister">
                <option></option>
                请选择户籍
                <%--@elvariable id="houseRegister" type="java.util.List"--%>
                <form:options items="${houseRegister}"/>
            </form:select>
        </p>
        <p>
            <label>个人描述：</label>
            <form:textarea path="remark" rows="5"/>
        </p>
        <p id="buttons">
            <input id="reset" type="reset" />
            <input id="submit" type="submit" value="添加">
        </p>
    </fieldset>
</form:form>
</body>
</html>

```



```jsp
<%-- userAdd.jsp --%>

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--
  Created by IntelliJ IDEA.
  User: pr919
  Date: 2019.6.24
  Time: 15:18
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page isELIgnored="false" %>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";
%>
<html>
<head>
    <base href="<%=basePath%>"/>
    <title>dataList</title>
</head>
<body>
<h1>用户列表</h1>
<a href="<c:url value="four/input"/>">继续添加</a>
<table>
    <tr>
        <th>用户名</th>
        <th>兴趣爱好</th>
        <th>朋友</th>
        <th>职业</th>
        <th>户籍</th>
        <th>个人描述</th>
    </tr>

    <c:forEach items="${users}" var="user">
        <tr>
            <td>${user.username}</td>
            <td>
                <c:forEach items="${user.hobby}" var="hobby">
                    ${hobby}&nbsp;
                </c:forEach>
            </td>
            <td>
                <c:forEach items="${user.friends}" var="friend">
                    ${friend}&nbsp;
                </c:forEach>
            </td>
            <td>${user.carrer}</td>
            <td>${user.houseRegister}</td>
            <td>${user.remark}</td>
        </tr>
    </c:forEach>
</table>
</body>
</html>

```



## 总结

目前来说，除了不用手动转型之外，我还没想到这个数据绑定有什么用。如果是为了验证输入的话，用javaScript或ajax就够了。

