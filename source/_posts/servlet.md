---
title: servlet
mathjax: true
categories: http
tags: 
---
# Servlet

<!--more-->

## 一 Servlet基础使用

###  1. 用webapp模板创建maven项目 

###  2.导入依赖

```xml
 <dependencies>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
</dependencies>
```

###    3.配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">
</web-app>
```

### 4.编写Servlet类，重写DoPost和DoGet方法

```java
public class Servlet01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        
    }
}
```

### 5.在web.xml中注册Servlet类

```xml
	<servlet>
        <servlet-name>Servlet</servlet-name>
        <!-- 类的路径 -->
        <servlet-class>Servlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>Servlet</servlet-name>
        <!--映射路径-->
        <url-pattern>/s03</url-pattern>
    </servlet-mapping>
```

## 二.ServletContext

​	所有Servlet共用一个ServletContext



## 三.Response & Request

### 1.下载文件

#### 1.获取文件的路径

```java
String url=this.getServletContext().getRealPath("");//默认搜索tomcat中webapp文件夹
```

#### 2.获取下载的文件名

```java
String filename=url.substring(url.lastIndexOf("\\")+1);//需要转义字符
```

#### 3.让浏览器支持下载文件

 ```java
  resp.setHeader("Content-Disposition","attachment;filename="+filename);
 ```

#### 4.获取下载文件的输入流

```java
FileInputStream inputStream=new FileInputStream(url);
```

#### 5.创建缓冲区

```java
int len=0;
byte[] buf=new byte[1024];
```

#### 6.获取输出流

```java
ServletOutputStream outout=resp.getOutputStream();
```

#### 7.将文件输出流写入缓冲区  用输出流将数据输出到客户端

```java
while((len= inputStream.read())>0){
    outout.write(buf,0,len);
}
//关闭流
inputStream.close();
output.close();
```

### 2.重定向

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    /*
    resp.setHeader("Location","/down")
    resp.setStatus(302);
    */
    resp.sendRedirect("/down");
}
```



#### 1.重定向与转发的区别

相同点：

​	都会实现页面的跳转

不同点

​	请求转发时，url不会发生变化

​	重定向时，url会变为访问的新页面

### 3.简易登陆界面实现

#### 1.登陆页面 

```jsp
<%@page contentType="text/html;charset=UTF-8" %>
<html>
    <body>
        <form action="${pageContext.request.contextPath}/login">
            用户名：<input type="text" name="username"><br>
            密码：<input type="password" name="password"><br>
            <input type="submit">
        </form>
    </body>
</html>

```

#### 2.Request类get方法

```java
public class RequestTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String un=req.getParameter("username");
        String pwd=req.getParameter("password");
        //根据账号密码选择重定向界面
        if(un.equals("admin")&&pwd.equals("123123")) {
            resp.sendRedirect("/hello.jsp");
        } else {
            resp.sendRedirect("/wrong.jsp");
        }
    }
}
```

#### 3.重定向页面

hello.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <body>
        <h2>Hello Welcome!!</h2>
    </body>
</html>

```

wrong.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <body>
        <h2>Wrong!!</h2>
    </body>
</html>

```

4.web.xml中注册servlet

```xml
<servlet>
    <servlet-name>res</servlet-name>
    <servlet-class>RequestTest</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>res</servlet-name>
    <url-pattern>/login</url-pattern>
</servlet-mapping>
```



## 四.Cookie

### 1.使用方法

#### 	1.Cookie类

```java
public class Cookie01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");

        PrintWriter out=resp.getWriter();

        Cookie[] cookies=req.getCookies();//获取cookies

        if(cookies!=null){//若cookie不为空则遍历cookie找到键值对
            out.write("last vis time:");
            for(Cookie t:cookies){
                if(t.getName().equals("username")){
                    out.write(t.getValue());
                }
            }
        }else{
            out.print("first in");
        }
        //添加或更新cookie
        Cookie cookie=new Cookie("username","zzy");
        resp.addCookie(cookie);
    }
}

```

#### 	2.web.xml注册cookie类

### 2.注意事项

​	1.cookie的<key,value>中不能包含以下字符

```java
[ ] ( ) = , " / ? @ : ;
```

​	解决方法:

​	使用URLEncoder的encode方法编码,使用时使用URLDecoder的decode方法解码

```java
URLEncoder.encode("","UTF-8");
URLDecoder.decode("","UTF-8");
```

## 五 .Session

### 	1.Session类

```java
public class Session extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        HttpSession session=req.getSession();
        session.setAttribute("hhh","zzy");
        String id=session.getId();
        if(session.isNew()){
            resp.getWriter().write("is new    "+"id:"+id);
        }else{
            resp.getWriter().write("is old id:"+id);
        }
    }
}
```

### 	2.在web.xml中注册 



# Fliter

​		用处：在服务器和资源间过滤掉不需要处理请求或者加工请求

​		

## 	1.应用一：处理乱码问题

### 		1.fliter类

```java
public class codingFliter implements Filter{
	
    //服务器启动时启动
    public void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain fchain) throws IOException, ServletException {
        //乱码解决
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf-8");
		//传递filter
        fchain.doFilter(req,resp);
    }
	//服务器关闭时销毁
    public void destroy() {}
}
```

### 		2.web.xml中注册过滤器

```xml
<filter>
    <filter-name>codingf</filter-name>
    <filter-class>codingFliter</filter-class>
</filter>
<filter-mapping>
    <filter-name>codingf</filter-name>
    <!--过滤监听的端口号-->
    <url-pattern>/s1</url-pattern>
</filter-mapping>
```



