---
title: "SSM-6-上传和下载"
tags:
  - Java
  - SSM
  - 上传下载
categories:
  - Java
  - SSM
date: 2019-07-10 01:04:18
draft: false
toc: false
images:
math: true
---

来了，上下都来了～

<!-- more -->

## 摘要

文件上传和下载是web经常要面对的问题。

上传的方式有多种，包括：

1. 使用文件流手工编程上传
2. 基于commons-fileupload组件的文件上传
3. 基于Servlet3及以上版本文件上传

下载经常有两种方式：

1. 通过超链接实现下载，但是会暴露下载文件的真实位置，并且只能下载存放在web应用所在的目录下的文件。
2. 利用程序编码实现下载，可以增加安全访问控制，还可以从任意位置提供下载的数据，比如数据库。

## 效果图

### 上传



![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/1.png)


选择文件前



![2](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/2.png)


选择文件后



![3](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/3.png)


上传文件后



### 下载



![4](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/4.png)




可以被下载的文件列表



![5](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/5.png)


选择文件存储位置



## 使用步骤

#### 导入jar包

SpringMVC框架的文件上传是基于commons-fileupload组件，所以需要commons-fileupload和commons-io的jar包。

maven项目：配置pom.xml

```xml
<!--上传下载-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

非maven项目：将下载好的 `commons-fileupload` 和 `commons-io` 的jar放到 `webapps\WEB-INF\lib` 目录下

### 单文件上传

#### 1. 编写领域模型

文件上传后保存在服务器里是一个对象，这个对象我们得定义它的属性。文件上传的时候的文件描述（备注），文件的创建时间等等，最重要的文件本身的类型是 `MultipartFile` 

```java
public class FileDomain {
    private String description;
    private MultipartFile myFile;
    
    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public MultipartFile getMyFile() {
        return myFile;
    }

    public void setMyFile(MultipartFile myFile) {
        this.myFile = myFile;
    }
}
```

#### 2. 编写一个提交页面

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
    <title>onFile</title>
</head>
<body>
<form action="six/oneFileUpLoad" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="myFile"/><br>
    文件描述：<input type="text" name="description"/><br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```

input标签的type属性为file时，就变成选择文件了。

![6](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/6.png)


**要注意的是：表单form的enctype属性应设置为 `multipart/form-data`**。然后就可以把表单提交到控制器了： `action="six/oneFileUpload"`



#### 3. 编写控制器

```java
@Controller
@RequestMapping("/six")
public class SixController {
    private static final Log logger = LogFactory.getLog(SixController.class);

    @RequestMapping("/oneFileUpLoad")
    public String oneFileUpload(@ModelAttribute FileDomain fileDomain, HttpServletRequest request) {
        // 获取存储路径
        String realpath = request.getServletContext().getRealPath("fileUpload");
        // 获取文件名
        String fileName = fileDomain.getMyFile().getOriginalFilename();
        // 如果文件名不为null && 目标文件夹不存在，则新建文件夹
        assert fileName != null;
        File targetFile = new File(realpath, fileName);
        if (!targetFile.exists()) {
            targetFile.mkdirs();
        }
        // 开始上传文件
        try {
            fileDomain.getMyFile().transferTo(targetFile);
            logger.info("成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "6/showOne";
    }
}
```

表单提交到控制器后就由控制器进行处理。

- 先获取realPath：文件保存到服务器的哪个地方，也叫存储路径。文件上传后都是存在服务器的，所以这里说的路径是指在服务器上的存储路径。
- 然后获取文件的文件名
- 如果realPath这个目录不存在，就创建目录
- 然后就开始上传文件。上传的关键方法就是 `transferTo()` 

这里上传成功就会跳转到 `/WEB-INF/jsp/6/showOne` 页面

\>_

![3](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/3.png)


#### 4. 编写结果页面

也就是上面要跳转的 `/WEB-INF/jsp/6/showOne` 页面

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
    <title>ShowOne</title>
</head>
<body>
${fileDomain.description}<br>
${fileDomain.myFile.originalFilename}
</body>
</html>
```



### 多文件上传

多文件上传和单文件的上传过程是相同的，但是他们的模型是不同的。多文件上传的模型是列表list

#### 1.领域模型

```java
public class MultiFileDomain {
   private List<String> description;
   private List<MultipartFile> myFile;

    public List<String> getDescription() {
        return description;
    }

    public void setDescription(List<String> description) {
        this.description = description;
    }

    public List<MultipartFile> getMyFile() {
        return myFile;
    }

    public void setMyFile(List<MultipartFile> myFile) {
        this.myFile = myFile;
    }
}
```

跟单文件上传的类型差不多，不同在于用list去装载。

#### 2.编写提交页面

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
    <title>MultiFile</title>
</head>
<body>
<form action="six/mutliFile" method="post" enctype="multipart/form-data">
    选择文件1：<input type="file" name="myFile"/> <br>
    文件描述1：<input type="text" name="description"/><br>
    选择文件2：<input type="file" name="myFile"/> <br>
    文件描述2：<input type="text" name="description"/><br>
    选择文件3：<input type="file" name="myFile"/> <br>
    文件描述3：<input type="text" name="description"/><br>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```

同样是enctype属性要设置成 `multipart/form-data` ，通过action属性提交到控制器

#### 3.控制器

```java
@Controller
@RequestMapping("/six")
public class SixController {
    private static final Log logger = LogFactory.getLog(SixController.class);
    
	@RequestMapping("/mutliFile")
    public String multiFile(@ModelAttribute MultiFileDomain multiFileDomain, HttpServletRequest request) {
        // 获取真实路径
        String realpath = request.getServletContext().getRealPath("fileUpload");
        // 如果目标文件夹不存在则新建文件夹
        File targetDir = new File(realpath);
        if (!targetDir.exists()) {
            targetDir.mkdirs();
        }
        // 将文件放置到list中
        List<MultipartFile> files = multiFileDomain.getMyFile();
        // 将list中的文件逐个上传
        for (MultipartFile file : files) {
            String fileName = file.getOriginalFilename();
            assert fileName != null;
            File targetFile = new File(realpath, fileName);
            try {
                file.transferTo(targetFile);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        logger.info("成功");
        return "6/showMulti";
    }
}
```

控制器里前部分也是相同的，获取存储路径，获取文件名，检查目标目录是否存在。

区别在于：将传过来的文件放在一个新的List中

```java
List<MultipartFile> files = multiFileDomain.getMyFile();
```

然后循环遍历整个List，将文件一个一个上传，然后返回 `/WEB-INF/jsp/6/showMulti` 页面



### 下载

利用程序实现下载需要设置两个报头：

1. **设置Content-Type的值为：application/x-msdownload**，因为web服务器需要告诉浏览器所输出内容的类型不是普通文件文件或是HTML，而是一个要保存到本地的下载文件。
2. **设置Content-Disposition的值为：attachment**，后面还可以指定filename参数。filename参数就是那个默认保存文件名。如下图红框所示：

![6](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/6.png)


```java
response.setHeader("Content-Type", "application/x-msdownload");
response.setHeader("Content-Disposition", "attachment;filename=" + toUTF8(filename));
```



#### 1. 编写跳转页面

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
    <title>MultiFile</title>
</head>
<body>
    
    
    
    <a href="six/showDownFiles">文件下载</a>
    
    
    
</form>
</body>
</html>
```

整个页面只有一个a标签的链接。点击之后会跳转到 `showDownFiles` 控制器



#### 2.编写列表控制器

```java
@Controller
@RequestMapping("/six")
public class SixController {
    private static final Log logger = LogFactory.getLog(SixController.class);
    
	@RequestMapping("/showDownFiles")
    public String showDown(HttpServletRequest request, Model model) {
        // 获取存储目录
        String realpath = request.getServletContext().getRealPath("fileUpload");
        // 实例化为一个File对象
        File dir = new File(realpath);
        // 将目录下的文件放到File数组中
        File[] files = dir.listFiles();
        // 创建一个数组列表，用于返回到前台页面
        ArrayList<String> fileName = new ArrayList<>();
        assert files != null;
        // 逐一将目录下的文件添加到数组列表中
        for (File file : files) {
            fileName.add(file.getName());
        }
        // 将数组列表返回给前台页面
        model.addAttribute("files", fileName);
        return "6/showDownFiles";
    }
}
```

列出控制器还是比较简单的：

1. 获取存储位置
2. 将路径实例化成一个File对象
3. 将存储位置里的文件放到File数组中
4. 创建一个数组列表，然后将文件逐个添加到数组列表中
5. 将数组列表返回给前台页面

#### 3. 编写下载列表展示页面

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page isELIgnored="false" %>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";
%>
<html>
<head>
    <base href="<%=basePath%>"/>
    <title>ShowDownFiles</title>
</head>
<body>
<table>
    <tr><td>被下载文件名</td></tr>
    <c:forEach items="${files}" var="filename"><%--@elvariable id="filesname" type="po.six.FileDomain"--%>
        <tr>
            <td><a href="six/down?filename=${filename}">${filename}</a></td>
        </tr>
    </c:forEach>
</table>
</body>
</html>
```

这里用了表单标签库中的 `<c:forEach>` 标签，将存储目录中的文件列出来，列出来的每一项都是一个超链接，链接指向 `six/down` 这个下载控制器。效果如下：

![4](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/4.png)


#### 4.编写下载控制器

```java
@Controller
@RequestMapping("/six")
public class SixController {
    private static final Log logger = LogFactory.getLog(SixController.class);
    
	@RequestMapping("/down")
    public String down(@RequestParam String filename, HttpServletRequest request, HttpServletResponse response) {
        try {
            // 获取存储位置
            String savePath = request.getServletContext().getRealPath("fileUpload");
            // 设置报头
            response.setHeader("Content-Type", "application/x-msdownload");
            response.setHeader("Content-Disposition", "attachment;filename=" + toUTF8(filename));
            // 读取服务器上的文件并写入到输出对象中，也就是下载
            FileInputStream in = new FileInputStream(savePath + "\\" + filename);
            ServletOutputStream out = response.getOutputStream();
            out.flush();
            int aRead;
            byte[] b = new byte[1024];
            while ((aRead = in.read(b)) != -1) {
                out.write(b, 0, aRead);
            }
            out.flush();
            out.close();
            in.close();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        logger.info("下载成功");
        return null;
    }
}
```

下载控制器则做了几件事情：

1. 获取存储目录
2. 设置报头
3. 将目标文件从输入流读取出来，转移到Servlet输出流写入进去，完成下载这个动作

![5](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/SSM-6/5.png)


因为报头的 Content-Disposition 设置为 attachment，所以下载之前浏览器会询问保存位置。

## 总结

上传和下载是web中很重要的一项功能，需要彻底掌握。上传的方式有多种，基于commons-fileupload的文件上传还是比较容易的，关键代码就是transferTo()方法。而下载选择用编码实现是为了不暴露文件的真实位置，还有就是可以下载任意位置的文件，所以可以将文件放在数据库中，然后让下载控制器去数据库中读取下载。虽然步骤比较麻烦，但是安全，规范。

