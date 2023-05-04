# Servlet基础

## 1. 什么是Servlet

​		![image-20230322191729086](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322191729086.png)

## 2. 手动实现Servlet程序

  1. 编写一个类实现Servlet接口

  2. 实现service方法，处理请求，并相应数据

     ```java
     package com.bupt;
     
     import javax.servlet.*;
     import java.io.IOException;
     
     public class HelloServlet implements Servlet {
         @Override
         public void init(ServletConfig servletConfig) throws ServletException {
     
         }
     
         @Override
         public ServletConfig getServletConfig() {
             return null;
         }
     
         /*
         * service方法是专门用来处理请求和相应的
         * */
         @Override
         public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
             System.out.println("Hello world 被访问了！！");
         }
     
         @Override
         public String getServletInfo() {
             return null;
         }
     
         @Override
         public void destroy() {
     
         }
     }
     ```

  3. 到web.xml中去配置servlet程序的访问地址

     ```html
     <!--servlet标签给Tomcat配置Servlet程序-->
     <servlet>
       <!--servlet-name标签给Servlet程序起一个别名（一般是类名）-->
       <servlet-name>HelloServlet</servlet-name>
       <!--servlet-class是Servlet程序的全类名-->
       <servlet-class>com.bupt.HelloServlet</servlet-class>
     </servlet>
     
     
     <!--servlet-mapping标签给Servlet程序配置访问地址-->
     <servlet-mapping>
       <!--servlet-name标签的作用是告诉服务器，我当前配置的地址给哪个Servlet程序使用-->
       <servlet-name>HelloServlet</servlet-name>
       <!--url-pattern标签配置访问地址
           / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径     （这个工程路径在Tomcat里配置）
           /hello  表示地址为：http://ip:port/工程路径/hello            (hello可以随便更换，可以自定义)
       -->
       <url-pattern>/hello</url-pattern>
     </servlet-mapping>
     ```

其中常见的错误

1. 

![image-20230322194650626](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322194650626.png)

2. <servlet>中的servlet-name和<servlet-mapping>中的servlet-name不匹配



## 3. Servlet-URL地址如何定位到Servlet程序中去访问

![image-20230322200114325](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322200114325.png)



## 4. Servlet生命周期

![image-20230322201018015](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322201018015.png)

## 5. GET和POST请求的分发处理

  1. 在webapp文件夹下面创建一个a.html

     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <title>Title</title>
     </head>
     <body>
         <!--表单提交数据到servlet程序访问的地址-->
         <form action="http://localhost:8080/springmvc_test1_war_exploded/hello" method="get">
             <input type="submit">
         </form>
     </body>
     </html>
     ```

2. 对servlet程序进行相应修改

   ```java
   package com.bupt;
   
   import javax.servlet.*;
   import javax.servlet.http.HttpServletRequest;
   import java.io.IOException;
   
   public class HelloServlet implements Servlet {
       @Override
       public void init(ServletConfig servletConfig) throws ServletException {
   
       }
   
       @Override
       public ServletConfig getServletConfig() {
           return null;
       }
   
       /*
       * service方法是专门用来处理请求和相应的
       * */
       @Override
       public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
           //类型转换（因为HttpServletRequest有getMethod()方法）
           HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
           //获取请求的方式
           String method = httpServletRequest.getMethod();
           if ("GET".equals(method)) {
               doGet();
           } else if ("POST".equals(method)) {
               doPost();
           }
       }
   
       public void doGet() {
           System.out.println("get请求");
           System.out.println("get请求");
       }
   
       public void doPost() {
           System.out.println("post请求");
           System.out.println("post请求");
       }
   
       @Override
       public String getServletInfo() {
           return null;
       }
   
       @Override
       public void destroy() {
   
       }
   }
   ```

## 6. 通过继承HttpServlet实现Servlet程序

![image-20230322210802481](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322210802481.png)

```java
package com.bupt;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet2 extends HttpServlet {
    /*
    * doGet()方法在get请求时调用
    * doPost()方法在post请求时调用
    * */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("Hello Servlet get方法");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("Hello Servlet post方法");
    }
}
```

```html
<servlet>
  <servlet-name>HelloServlet2</servlet-name>
  <servlet-class>com.bupt.HelloServlet2</servlet-class>
</servlet>

<servlet-mapping>
  <servlet-name>HelloServlet2</servlet-name>
  <url-pattern>/hello2</url-pattern>
</servlet-mapping>
```

## 7. IDEA菜单生成servlet程序

![image-20230322212735323](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322212735323.png)

![image-20230322212802964](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322212802964.png)

之后再重写doPost()和doGet()方法，然后配置<servlet-mapping>

## 8. Servlet类的继承体系

![image-20230322213917159](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322213917159.png)

## 9. ServletConfig类

三大作用

1. 可以获取Servlet程序的别名servlet-name的值
2. 获得初始化参数init-param
3. 获取ServletContext对象

```java
public class HelloServlet implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        //重写init方法要加上父类方法的调用，不然后面部分会报空指针异常
        super.init(config);
//        1. 可以获取Servlet程序的别名servlet-name的值
        System.out.println("HelloServlet的别名是" + servletConfig.getServletName());
//        2. 获得初始化参数init-param
        System.out.println(servletConfig.getInitParameter("userName"));
        System.out.println(servletConfig.getInitParameter("url"));
//        3. 获取ServletContext对象
        System.out.println(servletConfig.getServletContext());
    }
    .
    .
    //实现接口而重写的其他方法省略了
    .
    .
    .
    .
}
```

其中第二个作用时，要在web.xml中进行配置

```html
<servlet>
  <!--servlet-name标签给Servlet程序起一个别名（一般是类名）-->
  <servlet-name>HelloServlet</servlet-name>
  <!--servlet-class是Servlet程序的全类名-->
  <servlet-class>com.bupt.HelloServlet</servlet-class>
  <!--init-param是初始化参数-->
  <init-param>
    <!--是参数名-->
    <param-name>userName</param-name>
    <!--是参数值-->
    <param-value>root</param-value>
  </init-param>
  <!--可以初始化很多参数-->
  <init-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost:3306/test</param-value>
  </init-param>
</servlet>
```

## 10. ServletContext类

![image-20230323112548908](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323112548908.png)	

ServletContext在web工程部署启动时创建，在Web工程停止的时候销毁



2. 四个作用

   首先进行ServletContext的配置

   ```html
   <servlet>
     <servlet-name>ContextServlet</servlet-name>
     <servlet-class>com.bupt.ContextServlet</servlet-class>
   </servlet>
   <servlet-mapping>
     <servlet-name>ContextServlet</servlet-name>
     <url-pattern>/contextServlet</url-pattern>
   </servlet-mapping>
   ```

   1. 获取web.xml中配置的上下文参数context-param

      ```java
      //        1. 获取web.xml中配置的上下文参数context-param
      ServletContext context = getServletConfig().getServletContext();
      String username = context.getInitParameter("username");
      System.out.println("context-param参数username值是：" + username);
      System.out.println("context-param参数password值是：" + context.getInitParameter("password"));
      ```

   2. 获取当前的工程路径，格式：/工程路径

      ```java
      //        2. 获取当前的工程路径，格式：/工程路径
      System.out.println("当前工程路径：" + context.getContextPath());
      ```

   3. 获取共工程部署后在服务器硬盘上的绝对路径

      ```java
      //        3. 获取共工程部署后在服务器硬盘上的绝对路径
      /*
      *       / 斜杠被服务器解析地址为：http://ip:port/工程名/       映射到IDEA代码的Web目录
      * */
      System.out.println("工程部署的路径是：" + context.getRealPath("/"));
      ```

   4. 像Map一样存取数据

      ```java
      //        4. 像Map一样存取数据
      context.setAttribute("key1", "value1");
      System.out.println("context中获取数据key1的值为：" + context.getAttribute("key1"));
      ```

   
# Http协议

## 1. 概念

   ![image-20230323153020997](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323153020997.png)

## 2. 请求的HTTP协议格式

![image-20230323153857760](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323153857760.png)

### 1. GET请求

![image-20230323153939206](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323153939206.png)

![image-20230323153810369](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323153810369.png)

### 2. POST请求

![image-20230323154143497](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323154143497.png)

![image-20230323154748712](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323154748712.png)

### 3. 常用请求头的说明

![image-20230323154843402](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323154843402.png)

### 4.哪些是GET请求，哪些是POST请求

![image-20230323155027816](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323155027816.png)

## 3. 响应的HTTP协议格式

### 		1. 组成

![image-20230323155158152](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323155158152.png)

![image-20230323160128059](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323160128059.png)

### 2. 常见的响应码说明

![image-20230323160549577](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323160549577.png)

### 3. MIME类型说明

![image-20230323160734521](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323160734521.png)

### 4. 谷歌浏览器查看HTTP协议

按F12进入开发者模式

![image-20230323161341974](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323161341974.png)

# HttpServletRequest类

## 1.作用

![image-20230323161942264](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323161942264.png)

## 2.常用方法

![image-20230323162227122](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323162227122.png)

## 3. doGet和doPOST的中文乱码解决

```java
//设置请求字体的字符集为UTF-8，从而解决post的中文乱码问题
request.setCharacterEncoding("UTF-8");
```

## 4. 请求转发

1. 概念

   ![image-20230323165106357](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323165106357.png)

![image-20230323171016272](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323171016272.png)

```java
public class Servlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取请求参数（办事的材料）查看
        String username = request.getParameter("username");
        System.out.println("在Servlet1（柜台1）中查看参数（材料）" + username);
        //给材料盖一个张，并传递到Servlet2（柜台2）去查看
        request.setAttribute("key1", "柜台1的章");
        //问路：Servlet2（柜台2）怎么走
        /*
        * 请求转发必须要以斜杠打头， / 斜杠表示地址为： http:ip:port/工程名/      ，映射到IDEA代码的web目录
        * */
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
        //走向Servlet2（柜台2）
        requestDispatcher.forward(request, response);
    }
```

```java
public class Servlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取请求参数（办事的材料）查看
        String username = request.getParameter("username");
        System.out.println("在Servlet2（柜台2）中查看参数（材料）" + username);
        //查看柜台1是否盖章
        Object key1 = request.getAttribute("key1");
        System.out.println("柜台1是否有章：" + key1);
        //处理自己的业务
        System.out.println("Servlet2处理自己的业务");
    }
```

## 5. base标签的作用

![image-20230323195017323](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323195017323.png)

## 6. Web中的相对路径和绝对路径

![image-20230323195126612](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323195126612.png)

## 7. web中/ 斜杠的不同意义

![image-20230323195456895](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323195456895.png)

# HttpServletResponse类

## 1.作用

![image-20230323195827059](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323195827059.png)

## 2. 两个输出流的说明

![image-20230323200555774](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323200555774.png)

## 3. 如何往客户端回传数据

例如：回传字符串数据

```java
PrintWriter writer = response.getWriter();
writer.write("response's content!!!!");
```

## 4.解决响应的中文乱码

方法一

```java
//设置服务器字符集UTF-8
response.setCharacterEncoding("UTF-8");
//通过响应头，设置浏览器也使用UTF-8字符集
response.setHeader("Context-Type", "text/html; charset=UTF-8");
```

方法二

```java
//它同时设置服务器和客户端都使用UTF-8字符集，还设置了响应头
//此方法一定要在获取流对象之前调用才有效
response.setContentType("text/html; charset=UTF-8");
```

## 5. 请求重定向

![image-20230323205357967](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230323205357967.png)

```java
public class Response1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("曾到此一游 Response1！！！！");
 //方法一
        //设置状态响应码302，表示重定向，（已搬迁）
        response.setStatus(302);
        //设置响应头，说明新的地址在哪里。Location表示：后面的地址是编译地址（固定用法）
        response.setHeader("Location", "Http://localhost:8080//springmvc_test1_war_exploded/response2");
 //方法二
        response.sendRedirect("Http://localhost:8080//springmvc_test1_war_exploded/response2")
    }
```

```java
public class Response2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().write("response2's result!!!!");
    }
```

# 浏览器和Session之间关联的技术内幕

![image-20230329152759556](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230329152759556.png)

