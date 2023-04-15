# 九.Filter&Listener

## 1，Filter

### 1.1  Filter概述

Filter 表示过滤器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一。Servlet 我们之前都已经学习过了，Filter和Listener 我们今天都会进行学习。

过滤器可以把对资源的请求==拦截==下来，从而实现一些特殊的功能。

如下图所示，浏览器可以访问服务器上的所有的资源（servlet、jsp、html等）

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823184519509.png" alt="image-20210823184519509" style="zoom:50%;" />

而在访问到这些资源之前可以使过滤器拦截来下，也就是说在访问资源之前会先经过 Filter，如下图

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823184657328.png" alt="image-20210823184657328" style="zoom:57%;" />

拦截器拦截到后可以做什么功能呢？

==过滤器一般完成一些通用的操作。==比如每个资源都要写一些代码完成某个功能，我们总不能在每个资源中写这样的代码吧，而此时我们可以将这些代码写在过滤器中，因为请求每一个资源都要经过过滤器。

我们之前做的品牌数据管理的案例中就已经做了登陆的功能，而如果我们不登录能不能访问到数据呢？我们可以在浏览器直接访问首页 ，可以看到 `查询所有` 的超链接

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823185720197.png" alt="image-20210823185720197" style="zoom:70%;" />

当我点击该按钮，居然可以看到品牌的数据

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823185932418.png" alt="image-20210823185932418" style="zoom:70%;" />

这显然和我们的要求不符。我们希望实现的效果是用户如果登陆过了就跳转到品牌数据展示的页面；如果没有登陆就跳转到登陆页面让用户进行登陆，要实现这个效果需要在每一个资源中都写上这段逻辑，而像这种通用的操作，我们就可以放在过滤器中进行实现。这个就是==权限控制==，以后我们还会进行细粒度权限控制。过滤器还可以做 `统一编码处理`、 `敏感字符处理` 等等…

### 1.2  Filter快速入门

#### 1.2.1  开发步骤

进行 `Filter` 开发分成以下三步实现

* 定义类，实现 Filter接口，并重写其所有方法

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823191006878.png" alt="image-20210823191006878" style="zoom:60%;" />

* 配置Filter拦截资源的路径：在类上定义 `@WebFilter` 注解。而注解的 `value` 属性值 `/*` 表示拦截所有的资源

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823191037163.png" alt="image-20210823191037163" style="zoom:67%;" />

* 在doFilter方法中输出一句话，并放行

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823191200201.png" alt="image-20210823191200201" style="zoom:60%;" />

  > 上述代码中的 `chain.doFilter(request,response);` 就是放行，也就是让其访问本该访问的资源。

#### 1.2.2  代码演示

创建一个项目，项目下有一个 `hello.jsp` 页面，项目结构如下：

![image-20230226123134038](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230226123134038.png)

`pom.xml` 配置文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>filter-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <port>80</port>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

`hello.jsp` 页面内容如下：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>hello JSP~</h1>
</body>
</html>
```

我们现在在浏览器输入 `http://localhost/filter-demo/hello.jsp` 访问 `hello.jsp` 页面，这里是可以访问到 `hello.jsp` 页面内容的。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823192353031.png" alt="image-20210823192353031" style="zoom:67%;" />

接下来编写过滤器。过滤器是 Web 三大组件之一，所以我们将 `filter` 创建在 `com.iflytek.web.filter` 包下，起名为 `FilterDemo`

```java
@WebFilter("/*")
public class FilterDemo implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("FilterDemo...");
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void destroy() {
    }
}

```

重启启动服务器，再次重新访问 `hello.jsp` 页面，这次发现页面没有任何效果，但是在 `idea` 的控制台可以看到如下内容

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823193759365.png" alt="image-20210823193759365" style="zoom:70%;" />

上述效果说明 `FilterDemo` 这个过滤器的 `doFilter()` 方法执行了，但是为什么在浏览器上看不到 `hello.jsp` 页面的内容呢？这是因为在 `doFilter()` 方法中添加放行的方法才能访问到 `hello.jsp` 页面。那就在 `doFilter()` 方法中添加放行的代码

```java
//放行
 chain.doFilter(request,response);
```

再次重启服务器并访问 `hello.jsp` 页面，发现这次就可以在浏览器上看到页面效果。

**`FilterDemo` 过滤器完整代码如下：**

```java
@WebFilter("/*")
public class FilterDemo implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("1.FilterDemo...");
        //放行
        chain.doFilter(request,response);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void destroy() {
    }
}

```

### 1.3  Filter执行流程

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823194830074.png" alt="image-20210823194830074" style="zoom:70%;" />

如上图是使用过滤器的流程，我们通过以下问题来研究过滤器的执行流程：

* 放行后访问对应资源，资源访问完成后，还会回到Filter中吗？

  从上图就可以看出肯定 ==会== 回到Filter中

* 如果回到Filter中，是重头执行还是执行放行后的逻辑呢？

  如果是重头执行的话，就意味着 `放行前逻辑` 会被执行两次，肯定不会这样设计了；所以访问完资源后，会回到 `放行后逻辑`，执行该部分代码。

通过上述的说明，我们就可以总结Filter的执行流程如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823195434581.png" alt="image-20210823195434581" style="zoom:70%;" />

接下来我们通过代码验证一下，在 `doFilter()` 方法前后都加上输出语句，如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823195828596.png" alt="image-20210823195828596" style="zoom:70%;" />

同时在 `hello.jsp` 页面加上输出语句，如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823200028284.png" alt="image-20210823200028284" style="zoom:70%;" />

执行访问该资源打印的顺序是按照我们标记的标号进行打印的话，说明我们上边总结出来的流程是没有问题的。启动服务器访问 `hello.jsp` 页面，在控制台打印的内容如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823200202153.png" alt="image-20210823200202153" style="zoom:80%;" />

以后我们可以将对请求进行处理的代码放在放行之前进行处理，而如果请求完资源后还要对响应的数据进行处理时可以在放行后进行逻辑处理。

### 1.4  Filter拦截路径配置

拦截路径表示 Filter 会对请求的哪些资源进行拦截，使用 `@WebFilter` 注解进行配置。如：`@WebFilter("拦截路径")` 

拦截路径有如下四种配置方式：

* 拦截具体的资源：/index.jsp：只有访问index.jsp时才会被拦截
* 目录拦截：/user/*：访问/user下的所有资源，都会被拦截
* 后缀名拦截：*.jsp：访问后缀名为jsp的资源，都会被拦截
* 拦截所有：/*：访问所有资源，都会被拦截

通过上面拦截路径的学习，大家会发现拦截路径的配置方式和 `Servlet` 的请求资源路径配置方式一样，但是表示的含义不同。

### 1.5  过滤器链

#### 1.5.1  概述

过滤器链是指在一个Web应用，可以配置多个过滤器，这多个过滤器称为过滤器链。

如下图就是一个过滤器链，我们学习过滤器链主要是学习过滤器链执行的流程

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823215835812.png" alt="image-20210823215835812" style="zoom:70%;" />

上图中的过滤器链执行是按照以下流程执行：

1. 执行 `Filter1` 的放行前逻辑代码
2. 执行 `Filter1` 的放行代码
3. 执行 `Filter2` 的放行前逻辑代码
4. 执行 `Filter2` 的放行代码
5. 访问到资源
6. 执行 `Filter2` 的放行后逻辑代码
7. 执行 `Filter1` 的放行后逻辑代码

以上流程串起来就像一条链子，故称之为过滤器链。

#### 1.5.2  代码演示

* 编写第一个过滤器 `FilterDemo` ，配置成拦截所有资源

  ```java
  @WebFilter("/*")
  public class FilterDemo implements Filter {
  
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
  
          //1. 放行前，对 request数据进行处理
          System.out.println("1.FilterDemo...");
          //放行
          chain.doFilter(request,response);
          //2. 放行后，对Response 数据进行处理
          System.out.println("3.FilterDemo...");
      }
  
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
      }
  
      @Override
      public void destroy() {
      }
  }
  ```

* 编写第二个过滤器 `FilterDemo2` ，配置炒年糕拦截所有资源

  ```java
  @WebFilter("/*")
  public class FilterDemo2 implements Filter {
  
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
  
          //1. 放行前，对 request数据进行处理
          System.out.println("2.FilterDemo...");
          //放行
          chain.doFilter(request,response);
          //2. 放行后，对Response 数据进行处理
          System.out.println("4.FilterDemo...");
      }
  
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
      }
  
      @Override
      public void destroy() {
      }
  }
  
  ```

* 修改 `hello.jsp` 页面中脚本的输出语句

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
      <h1>hello JSP~</h1>
      <%
          System.out.println("3.hello jsp");
      %>
  </body>
  </html>
  ```

* 启动服务器，在浏览器输入 `http://localhost/filter-demo/hello.jsp` 进行测试，在控制台打印内容如下

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823221222468.png" alt="image-20210823221222468" style="zoom:70%;" />

  

  从结果可以看到确实是按照我们之前说的执行流程进行执行的。

#### 1.5.3  问题

上面代码中为什么是先执行 `FilterDemo` ，后执行 `FilterDemo2` 呢？

我们现在使用的是注解配置Filter，而这种配置方式的优先级是按照过滤器类名(字符串)的自然排序。

比如有如下两个名称的过滤器 ： `BFilterDemo` 和 `AFilterDemo` 。那一定是 `AFilterDemo` 过滤器先执行。

### 1.6  案例

#### 1.6.1  需求

访问服务器资源时，需要先进行登录验证，如果没有登录，则自动跳转到登录页面

#### 1.6.2  分析

我们要实现该功能是在每一个资源里加入登陆状态校验的代码吗？显然是不需要的，只需要写一个 `Filter` ，在该过滤器中进行登陆状态校验即可。而在该 `Filter` 中逻辑如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823223214525.png" alt="image-20210823223214525" style="zoom:70%;" />

#### 1.6.3  代码实现

##### 1.6.3.1  创建Filter

在 `brand-demo` 工程创建 `com.iflytek.web.filter`  包，在该下创建名为 `LoginFilter` 的过滤器

```java
@WebFilter("/*")
public class LoginFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
      
    }

    public void init(FilterConfig config) throws ServletException {
    }

    public void destroy() {
    }
}
```

##### 1.6.3.2  编写逻辑代码

在 `doFilter()` 方法中编写登陆状态校验的逻辑代码。

我们首先需要从 `session` 对象中获取用户信息，但是 `ServletRequest` 类型的 requset 对象没有获取 session 对象的方法，所以此时需要将 request对象强转成 `HttpServletRequest` 对象。

```java
HttpServletRequest req = (HttpServletRequest) request;
```

然后完成以下逻辑

* 获取Session对象
* 从Session对象中获取名为 `user` 的数据
* 判断获取到的数据是否是 null
  * 如果不是，说明已经登陆，放行
  * 如果是，说明尚未登陆，将提示信息存储到域对象中并跳转到登陆页面

代码如下：

```java
@WebFilter("/*")
public class LoginFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        HttpServletRequest req = (HttpServletRequest) request;
   
        //1. 判断session中是否有user
        HttpSession session = req.getSession();
        Object user = session.getAttribute("user");

        //2. 判断user是否为null
        if(user != null){
            // 登录过了
            //放行
            chain.doFilter(request, response);
        }else {
            // 没有登陆，存储提示信息，跳转到登录页面

            req.setAttribute("login_msg","您尚未登陆！");
            req.getRequestDispatcher("/login.jsp").forward(req,response);
        }
    }

    public void init(FilterConfig config) throws ServletException {
    }

    public void destroy() {
    }
}
```

##### 1.6.3.3  测试并抛出问题

在浏览器上输入 `http://localhost:8080/brand-demo/` ，可以看到如下页面效果

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823224843179.png" alt="image-20210823224843179" style="zoom:60%;" />

从上面效果可以看出没有登陆确实是跳转到登陆页面了，但是登陆页面为什么展示成这种效果了呢？

##### 1.6.3.4  问题分析及解决

因为登陆页面需要 `css/login.css` 这个文件进行样式的渲染，下图是登陆页面引入的css文件图解

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823225411925.png" alt="image-20210823225411925" style="zoom:70%;" />

而在请求这个css资源时被过滤器拦截，就相当于没有加载到样式文件导致的。解决这个问题，只需要对所以的登陆相关的资源进行放行即可。还有一种情况就是当我没有用户信息时需要进行注册，而注册时也希望被过滤器放行。

综上，我们需要在判断session中是否包含用户信息之前，应该加上对登陆及注册相关资源放行的逻辑处理

```java
//判断访问资源路径是否和登录注册相关
//1,在数组中存储登陆和注册相关的资源路径
String[] urls = {"/login.jsp","/imgs/","/css/","/loginServlet","/register.jsp","/registerServlet","/checkCodeServlet"};
//2,获取当前访问的资源路径
String url = req.getRequestURL().toString(); 

//3,遍历数组，获取到每一个需要放行的资源路径
for (String u : urls) {
    //4,判断当前访问的资源路径字符串是否包含要放行的的资源路径字符串
    /*
    	比如当前访问的资源路径是  /brand-demo/login.jsp
    	而字符串 /brand-demo/login.jsp 包含了  字符串 /login.jsp ，所以这个字符串就需要放行
    */
    if(url.contains(u)){
        //找到了，放行
        chain.doFilter(request, response);
        //break;
        return;
    }
}
```

##### 1.6.3.5  过滤器完整代码

```java
@WebFilter("/*")
public class LoginFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
        HttpServletRequest req = (HttpServletRequest) request;
        
        //判断访问资源路径是否和登录注册相关
        //1,在数组中存储登陆和注册相关的资源路径
        String[] urls = {"/login.jsp","/imgs/","/css/","/loginServlet","/register.jsp","/registerServlet","/checkCodeServlet"};
        //2,获取当前访问的资源路径
        String url = req.getRequestURL().toString(); 

        //3,遍历数组，获取到每一个需要放行的资源路径
        for (String u : urls) {
            //4,判断当前访问的资源路径字符串是否包含要放行的的资源路径字符串
            /*
                比如当前访问的资源路径是  /brand-demo/login.jsp
                而字符串 /brand-demo/login.jsp 包含了  字符串 /login.jsp ，所以这个字符串就需要放行
            */
            if(url.contains(u)){
                //找到了，放行
                chain.doFilter(request, response);
                //break;
                return;
            }
        }
   
        //1. 判断session中是否有user
        HttpSession session = req.getSession();
        Object user = session.getAttribute("user");

        //2. 判断user是否为null
        if(user != null){
            // 登录过了
            //放行
            chain.doFilter(request, response);
        }else {
            // 没有登陆，存储提示信息，跳转到登录页面

            req.setAttribute("login_msg","您尚未登陆！");
            req.getRequestDispatcher("/login.jsp").forward(req,response);
        }
    }

    public void init(FilterConfig config) throws ServletException {
    }

    public void destroy() {
    }
}
```

## 2，Listener

### 2.1  概述

* Listener 表示监听器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一。

* 监听器可以监听就是在 `application`，`session`，`request` 三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件。

  request 和 session 我们学习过。而 `application` 是 `ServletContext` 类型的对象。

  `ServletContext` 代表整个web应用，在服务器启动的时候，tomcat会自动创建该对象。在服务器关闭时会自动销毁该对象。

### 2.2  分类

JavaWeb 提供了8个监听器：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210823230820586.png" alt="image-20210823230820586" style="zoom:80%;" />

这里面只有 `ServletContextListener` 这个监听器后期我们会接触到，`ServletContextListener` 是用来监听 `ServletContext` 对象的创建和销毁。

`ServletContextListener` 接口中有以下两个方法

* `void contextInitialized(ServletContextEvent sce)`：`ServletContext` 对象被创建了会自动执行的方法
* `void contextDestroyed(ServletContextEvent sce)`：`ServletContext` 对象被销毁时会自动执行的方法

### 2.3  代码演示

我们只演示一下 `ServletContextListener` 监听器

* 定义一个类，实现`ServletContextListener` 接口
* 重写所有的抽象方法
* 使用 `@WebListener` 进行配置

代码如下：

```java
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        //加载资源
        System.out.println("ContextLoaderListener...");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        //释放资源
    }
}
```

启动服务器，就可以在启动的日志信息中看到 `contextInitialized()` 方法输出的内容，同时也说明了 `ServletContext` 对象在服务器启动的时候被创建了。