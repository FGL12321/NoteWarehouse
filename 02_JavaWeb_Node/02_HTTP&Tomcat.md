[TOC]



# HTTP&Tomcat

## 1. Web概述

### 1.1 Web和JavaWeb的概念

==Web是全球广域网，也称为万维网(www)，能够通过浏览器访问的网站。==
在我们日常的生活中，经常会使用浏览器去访问`百度`、`京东`等这些网站，这些网站统称为Web网站。

我们知道了什么是Web，那么JavaWeb又是什么呢？顾名思义==JavaWeb就是用Java技术来解决相关web互联网领域的技术栈。==

等学习完JavaWeb之后，同学们就可以使用Java语言开发我们上述所说的网站。而国内很多大型网站公司也是首选Java语言来解决web互联网相关的问题。那都有哪些公司的系统是使用Java语言的呢?
 ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/20210717183958532.png)
使用Java语言开发互联网系统是有很多技术栈需要大家了解，具体都有哪些呢?

### 1.2 JavaWeb技术栈

了解JavaWeb技术栈之前，有一个很重要的概念要介绍。

#### 1.2.1 B/S架构

什么是B/S架构?
B/S 架构：Browser/Server，浏览器/服务器 架构模式，它的特点是，客户端只需要浏览器，应用程序的逻辑和数据都存储在服务器端。浏览器只需要请求服务器，获取Web资源，服务器把Web资源发送给浏览器即可。大家可以通过下面这张图来回想下我们平常的上网过程：
![1627031933553](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627031933553.png)

* 打开浏览器访问百度首页，输入要搜索的内容，点击回车或百度一下，就可以获取和搜索相关的内容
* 思考下搜索的内容并不在我们自己的点上，那么这些内容从何而来？答案很明显是从百度服务器返回给我们的
* 日常百度的小细节，逢年过节百度的logo会更换不同的图片，服务端发生变化，客户端不需做任务事情就能获取最新内容
* 所以说B/S架构的好处：易于维护升级：服务器端升级后，客户端无需任何部署就可以使用到新的版本。
  了解了什么是B/S架构后，作为后台开发工程师的我们将来主要关注的是服务端的开发和维护工作。在服务端将来会放很多资源,都有哪些资源呢?

#### 1.2.2 静态资源

* 静态资源主要包含HTML、CSS、JavaScript、图片等，主要负责页面的展示。
* 我们之前已经学过前端网页制作`三剑客`(HTML+CSS+JavaScript),使用这些技术我们就可以制作出效果比较丰富的网页，将来展现给用户。但是由于做出来的这些内容都是静态的，这就会导致所有的人看到的内容将是一模一样。
* 在日常上网的过程中，我们除了看到这些好看的页面以外，还会碰到很多动态内容，比如我们常见的百度登录效果：
  ![1627037814180](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627037814180.png)
  `张三`登录以后在网页的右上角看到的是 `张三`，而`李四`登录以后看到的则是`李四`。所以不同的用户访问相同的资源看到的内容大多数是不一样的，要想实现这样的效果，光靠静态资源是无法实现的。

#### 1.2.3 动态资源

* 动态资源主要包含Servlet、JSP等，主要用来负责逻辑处理。
* 动态资源处理完逻辑后会把得到的结果交给静态资源来进行展示，动态资源和静态资源要结合一起使用。
* 动态资源虽然可以处理逻辑，但是当用户来登录百度的时候，就需要输入`用户名`和`密码`，这个时候我们就又需要解决的一个问题是，用户在注册的时候填入的用户名和密码、以及我们经常会访问到一些数据列表的内容展示(如下图所示)，这些数据都存储在哪里?我们需要的时候又是从哪里来取呢?
  ![1627038674340](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627038674340.png)

#### 1.2.4 数据库

* 数据库主要负责存储数据。
* 整个Web的访问过程就如下图所示：
  ![1627039320220](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627039320220.png)
  (1)浏览器发送一个请求到服务端，去请求所需要的相关资源;
  (2)资源分为动态资源和静态资源,动态资源可以是使用Java代码按照Servlet和JSP的规范编写的内容;
  (3)在Java代码可以进行业务处理也可以从数据库中读取数据;
  (4)拿到数据后，把数据交给HTML页面进行展示,再结合CSS和JavaScript使展示效果更好;
  (5)服务端将静态资源响应给浏览器;
  (6)浏览器将这些资源进行解析;
  (7)解析后将效果展示在浏览器，用户就可以看到最终的结果。
  在整个Web的访问过程中，会设计到很多技术，这些技术有已经学习过的，也有还未涉及到的内容，都有哪些还没有涉及到呢?

#### 1.2.5 HTTP协议

* HTTP协议：主要定义通信规则
* 浏览器发送请求给服务器，服务器响应数据给浏览器，这整个过程都需要遵守一定的规则，之前大家学习过TCP、UDP，这些都属于规则，这里我们需要使用的是HTTP协议，这也是一种规则。

#### 1.2.6 Web服务器

* Web服务器：负责解析 HTTP 协议，解析请求数据，并发送响应数据
* 浏览器按照HTTP协议发送请求和数据，后台就需要一个Web服务器软件来根据HTTP协议解析请求和数据，然后把处理结果再按照HTTP协议发送给浏览器
* Web服务器软件有很多，我们课程中将学习的是目前最为常用的==Tomcat==服务器

到这为止，关于JavaWeb中用到的技术栈我们就介绍完了，这里面就只有HTTP协议、Servlet、JSP以及Tomcat这些知识是没有学习过的，所以整个Web核心主要就是来学习这些技术。

### 1.3 Web核心课程安排

![1627043194238](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627043194238.png)

整个Web核心，我们总共有六个阶段的学习内容，分别是：

* 一：HTTP、Tomcat、Servlet
* 二：Request(请求)、Response(响应)
* 三：JSP、会话技术(Cookie、Session)
* 四：Filter(过滤器)、Listener(监听器)
* 五：Ajax、Vue、ElementUI
* 六：综合案例

(1)Request是从客户端向服务端发出的请求对象，

(2)Response是从服务端响应给客户端的结果对象，

(3)JSP是动态网页技术,

(4)会话技术是用来存储客户端和服务端交互所产生的数据，

(5)过滤器是用来拦截客户端的请求,

(6)监听器是用来监听特定事件,

(7)Ajax、Vue、ElementUI都是属于前端技术

这些技术都该如何来使用，我们后面会一个个进行详细的讲解。接下来我们来学习下HTTP、Tomcat和Servlet。 

## 2. HTTP

### 2.1 简介

**HTTP概念**

HyperText Transfer Protocol，超文本传输协议，规定了浏览器和服务器之间==数据传输的规则==。

* 数据传输的规则指的是请求数据和响应数据需要按照指定的格式进行传输。
* 如果想知道具体的格式，可以打开浏览器，点击`F12`打开开发者工具，点击`Network`来查看某一次请求的请求数据和响应数据具体的格式内容，如下图所示：

![1627046235092](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627046235092.png)

> 注意：在浏览器中如果看不到上述内容，需要清除浏览器的浏览数据。chrome浏览器可以使用ctrl+shift+Del进行清除。

==所以学习HTTP主要就是学习请求和响应数据的具体格式内容。==

**HTTP协议特点**

HTTP协议有它自己的一些特点，分别是：

* 基于TCP协议：面向连接，安全

  TCP是一种面向连接的(建立连接之前是需要经过三次握手)、可靠的、基于字节流的传输层通信协议，在数据传输方面更安全。

* 基于请求-响应模型的：一次请求对应一次响应

  请求和响应是一一对应关系

* HTTP协议是无状态协议：对于事物处理没有记忆能力。每次请求-响应都是独立的

  无状态指的是客户端发送HTTP请求给服务端之后，服务端根据请求响应数据，响应完后，不会记录任何信息。这种特性有优点也有缺点，

  * 缺点：多次请求间不能共享数据
  * 优点：速度快

  请求之间无法共享数据会引发的问题，如：

  * 京东购物，`加入购物车`和`去购物车结算`是两次请求，
  * HTTP协议的无状态特性，加入购物车请求响应结束后，并未记录加入购物车是何商品
  * 发起去购物车结算的请求后，因为无法获取哪些商品加入了购物车，会导致此次请求无法正确展示数据

  具体使用的时候，我们发现京东是可以正常展示数据的，原因是Java早已考虑到这个问题，并提出了使用`会话技术(Cookie、Session)`来解决这个问题。具体如何来做，我们后面会详细讲到。刚才提到HTTP协议是规定了请求和响应数据的格式，那具体的格式是什么呢?

### 2.2 请求数据格式

#### 2.2.1 格式介绍

请求数据总共分为三部分内容，分别是==请求行==、==请求头==、==请求体==

![1627050004221](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627050004221.png)

* 请求行： HTTP请求中的第一行数据，请求行包含三块内容，分别是 GET[请求方式] /[请求URL路径] HTTP/1.1[HTTP协议及版本]

  请求方式有七种,最常用的是GET和POST

* 请求头：第二行开始，格式为`key:value`形式

  请求头中会包含若干个属性，常见的HTTP请求头有：

  ```
  Host: 表示请求的主机名
  User-Agent: 浏览器版本,例如Chrome浏览器的标识类似Mozilla/5.0 ...Chrome/79，IE浏览器的标识类似Mozilla/5.0 (Windows NT ...)like Gecko；
  Accept：表示浏览器能接收的资源类型，如text/*，image/*或者*/*表示所有；
  Accept-Language：表示浏览器偏好的语言，服务器可以据此返回不同语言的网页；
  Accept-Encoding：表示浏览器可以支持的压缩类型，例如gzip, deflate等。
  ```

   ==这些数据有什么用处?==

  举例说明：服务端可以根据请求头中的内容来获取客户端的相关信息，有了这些信息服务端就可以处理不同的业务需求，比如：

  * 不同浏览器解析HTML和CSS标签的结果会有不一致，所以就会导致相同的代码在不同的浏览器会出现不同的效果
  * 服务端根据客户端请求头中的数据获取到客户端的浏览器类型，就可以根据不同的浏览器设置不同的代码来达到一致的效果
  * 这就是我们常说的浏览器兼容问题

* 请求体： POST请求的最后一部分，存储请求参数

  ![1627050930378](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627050930378.png)

  如上图红线框的内容就是请求体的内容，请求体和请求头之间是有一个空行隔开。此时浏览器发送的是POST请求，为什么不能使用GET呢？这时就需要回顾GET和POST两个请求之间的区别了：

  * GET请求请求参数在请求行中，没有请求体，POST请求请求参数在请求体中
  * GET请求请求参数大小有限制，POST没有

#### 2.2.2 实例演示

把 `代码\http` 拷贝到IDEA的工作目录中，比如`D:\workspace\web`目录，

![1627278511902](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627278511902.png)

使用IDEA打开

![1627278583127](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627278583127.png)

打开后，可以点击项目中的`html\19-表单验证.html`，使用浏览器打开，通过修改页面中form表单的method属性来测试GET请求和POST请求的参数携带方式。

![1627278725007](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627278725007.png)



**小结**：

1. 请求数据中包含三部分内容，分别是请求行、请求头和请求体

2. POST请求数据在请求体中，GET请求数据在请求行上

### 2.3 响应数据格式

#### 2.3.1 格式介绍

响应数据总共分为三部分内容，分别是==响应行==、==响应头==、==响应体==

![1627053710214](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627053710214.png)

* 响应行：响应数据的第一行,响应行包含三块内容，分别是 HTTP/1.1[HTTP协议及版本] 200[响应状态码] ok[状态码的描述]

* 响应头：第二行开始，格式为key：value形式

  响应头中会包含若干个属性，常见的HTTP响应头有：

  ```
  Content-Type：表示该响应内容的类型，例如text/html，image/jpeg；
  Content-Length：表示该响应内容的长度（字节数）；
  Content-Encoding：表示该响应压缩算法，例如gzip；
  Cache-Control：指示客户端应如何缓存，例如max-age=300表示可以最多缓存300秒
  ```

* 响应体： 最后一部分。存放响应数据

  上图中<html>...</html>这部分内容就是响应体，它和响应头之间有一个空行隔开。

#### 2.3.2 响应状态码

参考： 资料/1.HTTP/《响应状态码.md》

关于响应状态码，我们先主要认识三个状态码，其余的等后期用到了再去掌握：

* 200  ok 客户端请求成功
* 404  Not Found 请求资源不存在
* 500 Internal Server Error 服务端发生不可预期的错误

#### 2.3.3 自定义服务器

在前面我们导入到IDEA中的http项目中，有一个Server.java类，这里面就是自定义的一个服务器代码，主要使用到的是`ServerSocket`和`Socket`

```java
package com.iflytek;

import sun.misc.IOUtils;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
/*
    自定义服务器
 */
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket(8080); // 监听指定端口
        System.out.println("server is running...");
        while (true){
            Socket sock = ss.accept();
            System.out.println("connected from " + sock.getRemoteSocketAddress());
            Thread t = new Handler(sock);
            t.start();
        }
    }
}

class Handler extends Thread {
    Socket sock;

    public Handler(Socket sock) {
        this.sock = sock;
    }

    public void run() {
        try (InputStream input = this.sock.getInputStream()) {
            try (OutputStream output = this.sock.getOutputStream()) {
                handle(input, output);
            }
        } catch (Exception e) {
            try {
                this.sock.close();
            } catch (IOException ioe) {
            }
            System.out.println("client disconnected.");
        }
    }

    private void handle(InputStream input, OutputStream output) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(input, StandardCharsets.UTF_8));
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(output, StandardCharsets.UTF_8));
        // 读取HTTP请求:
        boolean requestOk = false;
        String first = reader.readLine();
        if (first.startsWith("GET / HTTP/1.")) {
            requestOk = true;
        }
        for (;;) {
            String header = reader.readLine();
            if (header.isEmpty()) { // 读取到空行时, HTTP Header读取完毕
                break;
            }
            System.out.println(header);
        }
        System.out.println(requestOk ? "Response OK" : "Response Error");
        if (!requestOk) {
            // 发送错误响应:
            writer.write("HTTP/1.0 404 Not Found\r\n");
            writer.write("Content-Length: 0\r\n");
            writer.write("\r\n");
            writer.flush();
        } else {
            // 发送成功响应:

            //读取html文件，转换为字符串
            BufferedReader br = new BufferedReader(new FileReader("http/html/a.html"));
            StringBuilder data = new StringBuilder();
            String line = null;
            while ((line = br.readLine()) != null){
                data.append(line);
            }
            br.close();
            int length = data.toString().getBytes(StandardCharsets.UTF_8).length;

            writer.write("HTTP/1.1 200 OK\r\n");
            writer.write("Connection: keep-alive\r\n");
            writer.write("Content-Type: text/html\r\n");
            writer.write("Content-Length: " + length + "\r\n");
            writer.write("\r\n"); // 空行标识Header和Body的分隔
            writer.write(data.toString());
            writer.flush();
        }
    }
}
```

上面代码，大家不需要自己写，主要通过上述代码，只需要大家了解到服务器可以使用java完成编写，是可以接受页面发送的请求和响应数据给前端浏览器的，真正用到的Web服务器，我们不会自己写，都是使用目前比较流行的web服务器，比如==Tomcat==

**小结**

1. 响应数据中包含三部分内容，分别是响应行、响应头和响应体

2. 掌握200，404，500这三个响应状态码所代表含义，分布是成功、所访问资源不存在和服务的错误

## 3. Tomcat

### 3.1 简介

#### 3.1.1 什么是Web服务器

Web服务器是一个应该程序（==软件==），对HTTP协议的操作进行封装，使得程序员不必直接对协议进行操作，让Web开发更加便捷。主要功能是"提供网上信息浏览服务"。

![1627058356051](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627058356051.png)

 Web服务器是安装在服务器端的一款软件，将来我们把自己写的Web项目部署到Web Tomcat服务器软件中，当Web服务器软件启动后，部署在Web服务器软件中的页面就可以直接通过浏览器来访问了。

**Web服务器软件使用步骤**

* 准备静态资源
* 下载安装Web服务器软件
* 将静态资源部署到Web服务器上
* 启动Web服务器使用浏览器访问对应的资源

上述内容在演示的时候，使用的是Apache下的Tomcat软件，至于Tomcat软件如何使用，后面会详细的讲到。而对于Web服务器来说，实现的方案有很多，Tomcat只是其中的一种，而除了Tomcat以外，还有很多优秀的Web服务器，比如：

![1627060368806](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627060368806.png)





Tomcat就是一款软件，我们主要是以学习如何去使用为主。具体我们会从以下这些方向去学习：

1. 简介： 初步认识下Tomcat

2. 基本使用： 安装、卸载、启动、关闭、配置和项目部署，这些都是对Tomcat的基本操作

3. IDEA中如何创建Maven Web项目

4. IDEA中如何使用Tomcat,后面这两个都是我们以后开发经常会用到的方式

首选我们来认识下Tomcat。

**Tomcat**

Tomcat的相关概念：

* Tomcat是Apache软件基金会一个核心项目，是一个开源免费的轻量级Web服务器，支持Servlet/JSP少量JavaEE规范。

* 概念中提到了JavaEE规范，那什么又是JavaEE规范呢?

  JavaEE： Java Enterprise Edition,Java企业版。指Java企业级开发的技术规范总和。包含13项技术规范：JDBC、JNDI、EJB、RMI、JSP、Servlet、XML、JMS、Java IDL、JTS、JTA、JavaMail、JAF。

* 因为Tomcat支持Servlet/JSP规范，所以Tomcat也被称为Web容器、Servlet容器。Servlet需要依赖Tomcat才能运行。

* Tomcat的官网： https://tomcat.apache.org/ 从官网上可以下载对应的版本进行使用。

**Tomcat的LOGO**

![1627176045795](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627176045795.png)

**小结**

通过这一节的学习，我们需要掌握以下内容：

1. Web服务器的作用

> 封装HTTP协议操作，简化开发
>
> 可以将Web项目部署到服务器中，对外提供网上浏览服务

2. Tomcat是一个轻量级的Web服务器，支持Servlet/JSP少量JavaEE规范，也称为Web容器，Servlet容器。

### 3.2 基本使用

Tomcat总共分两部分学习，先来学习Tomcat的基本使用，包括Tomcat的==下载、安装、卸载、启动和关闭==。

#### 3.2.1 下载

直接从官网下载

![1627178001030](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627178001030.png)

大家可以自行下载，也可以直接使用资料中已经下载好的资源，

Tomcat的软件程序  资料/2. Tomcat/apache-tomcat-8.5.68-windows-x64.zip

Tomcat的源码	资料/2. Tomcat/tomcat源码/apache-tomcat-8.5.68-src.zip

#### 3.2.2 安装

Tomcat是绿色版，直接解压即可

* 在D盘的software目录下，将`apache-tomcat-8.5.68-windows-x64.zip`进行解压缩，会得到一个`apache-tomcat-8.5.68`的目录，Tomcat就已经安装成功。

  ==注意==，Tomcat在解压缩的时候，解压所在的目录可以任意，但最好解压到一个不包含中文和空格的目录，因为后期在部署项目的时候，如果路径有中文或者空格可能会导致程序部署失败。

* 打开`apache-tomcat-8.5.68`目录就能看到如下目录结构，每个目录中包含的内容需要认识下,

  ![1627178815892](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627178815892.png)

  bin：目录下有两类文件，一种是以`.bat`结尾的，是Windows系统的可执行文件，一种是以`.sh`结尾的，是Linux系统的可执行文件。

  webapps：就是以后项目部署的目录

  到此，Tomcat的安装就已经完成。

#### 3.2.3 卸载

卸载比较简单，可以直接删除目录即可

#### 3.2.4 启动

双击： bin\startup.bat

![1627179006011](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627179006011.png)

启动后，通过浏览器访问 `http://localhost:8080`能看到Apache Tomcat的内容就说明Tomcat已经启动成功。

![1627199957728](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627199957728.png)

==注意==： 启动的过程中，控制台有中文乱码，需要修改conf/logging.prooperties

![1627199827589](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627199827589.png)

#### 3.2.5 关闭

关闭有三种方式 

* 直接x掉运行窗口：强制关闭[不建议]
* bin\shutdown.bat：正常关闭
* ctrl+c： 正常关闭

#### 3.2.6 配置

**修改端口**

* Tomcat默认的端口是8080，要想修改Tomcat启动的端口号，需要修改 conf/server.xml

![1627200509883](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627200509883.png)

> 注： HTTP协议默认端口号为80，如果将Tomcat端口号改为80，则将来访问Tomcat时，将不用输入端口号。

**启动时可能出现的错误**

* Tomcat的端口号取值范围是0-65535之间任意未被占用的端口，如果设置的端口号被占用，启动的时候就会包如下的错误

  ![1627200780590](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627200780590.png)

* Tomcat启动的时候，启动窗口一闪而过： 需要检查JAVA_HOME环境变量是否正确配置

![1627201248802](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627201248802.png)

#### 3.2.7 部署

* Tomcat部署项目： 将项目放置到webapps目录下，即部署完成。

  * 将 `资料/2. Tomcat/hello` 目录拷贝到Tomcat的webapps目录下

  * 通过浏览器访问`http://localhost/hello/a.html`，能看到下面的内容就说明项目已经部署成功。

    ![1627201572748](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627201572748.png)

    但是呢随着项目的增大，项目中的资源也会越来越多，项目在拷贝的过程中也会越来越费时间，该如何解决呢?

* 一般JavaWeb项目会被打包称==war==包，然后将war包放到Webapps目录下，Tomcat会自动解压缩war文件

  * 将 `资料/2. Tomcat/haha.war`目录拷贝到Tomcat的webapps目录下

  * Tomcat检测到war包后会自动完成解压缩，在webapps目录下就会多一个haha目录

  * 通过浏览器访问`http://localhost/haha/a.html`，能看到下面的内容就说明项目已经部署成功。

    ![1627201868752](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627201868752.png)

至此，Tomcat的部署就已经完成了，至于如何获得项目对应的war包，后期我们会借助于IDEA工具来生成。

### 3.3 Maven创建Web项目

介绍完Tomcat的基本使用后，我们来学习在IDEA中如何创建Maven Web项目，学习这种方式的原因是以后Tomcat中运行的绝大多数都是Web项目，而使用Maven工具能更加简单快捷的把Web项目给创建出来，所以Maven的Web项目具体如何来构建呢?

在真正创建Maven Web项目之前，我们先要知道Web项目长什么样子，具体的结构是什么?

#### 3.3.1 Web项目结构

Web项目的结构分为：开发中的项目和开发完可以部署的Web项目,这两种项目的结构是不一样的，我们一个个来介绍下：

* Maven Web项目结构： 开发中的项目

  ![1627202865978](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627202865978.png)

* 开发完成部署的Web项目

  ![1627202903750](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627202903750.png)

  * 开发项目通过执行Maven打包命令==package==,可以获取到部署的Web项目目录
  * 编译后的Java字节码文件和resources的资源文件，会被放到WEB-INF下的classes目录下
  * pom.xml中依赖坐标对应的jar包，会被放入WEB-INF下的lib目录下

#### 3.3.2 创建Maven Web项目

介绍完Maven Web的项目结构后，接下来使用Maven来创建Web项目，创建方式有两种：使用骨架和不使用骨架

**使用骨架**

> 具体的步骤包含：
>
> 1.创建Maven项目
>
> 2.选择使用Web项目骨架
>
> 3.输入Maven项目坐标创建项目
>
> 4.确认Maven相关的配置信息后，完成项目创建
>
> 5.删除pom.xml中多余内容
>
> 6.补齐Maven Web项目缺失的目录结构

1. 创建Maven项目

   ![1627227574092](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627227574092.png)

2. 选择使用Web项目骨架

   ![1627227650406](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627227650406.png)

3. 输入Maven项目坐标创建项目

   ![1627228065007](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627228065007.png)

4. 确认Maven相关的配置信息后，完成项目创建

   ![1627228413280](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627228413280.png)

5. 删除pom.xml中多余内容，只留下面的这些内容，注意打包方式 jar和war的区别

   ![1627228584625](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627228584625.png)

6. 补齐Maven Web项目缺失的目录结构，默认没有java和resources目录，需要手动完成创建补齐，最终的目录结果如下

   ![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627228673162.png)



**不使用骨架**

>具体的步骤包含：
>
>1.创建Maven项目
>
>2.选择不使用Web项目骨架
>
>3.输入Maven项目坐标创建项目
>
>4.在pom.xml设置打包方式为war
>
>5.补齐Maven Web项目缺失webapp的目录结构
>
>6.补齐Maven Web项目缺失WEB-INF/web.xml的目录结构

1. 创建Maven项目

   ![1627229111549](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229111549.png)

2. 选择不使用Web项目骨架

   ![1627229137316](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229137316.png)

3. 输入Maven项目坐标创建项目

   ![1627229371251](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229371251.png)

4. 在pom.xml设置打包方式为war,默认是不写代表打包方式为jar

   ![1627229428161](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229428161.png)

5. 补齐Maven Web项目缺失webapp的目录结构

   ![1627229584134](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229584134.png)

6. 补齐Maven Web项目缺失WEB-INF/web.xml的目录结构

   ![1627229676800](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229676800.png)

7. 补充完后，最终的项目结构如下：

   

   

   ![1627229478030](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229478030.png)

上述两种方式，创建的web项目，都不是很全，需要手动补充内容，至于最终采用哪种方式来创建Maven Web项目，都是可以的，根据各自的喜好来选择使用即可。

**小结**

1.掌握Maven Web项目的目录结构

2.掌握使用骨架的方式创建Maven Web项目

![1627204022604](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627204022604.png)

> 3.掌握不使用骨架的方式创建Maven Web项目

![1627204076090](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627204076090.png)

### 3.4 IDEA使用Tomcat

* Maven Web项目创建成功后，通过Maven的package命令可以将项目打包成war包，将war文件拷贝到Tomcat的webapps目录下，启动Tomcat就可以将项目部署成功，然后通过浏览器进行访问即可。
* 然而我们在开发的过程中，项目中的内容会经常发生变化，如果按照上面这种方式来部署测试，是非常不方便的
* 如何在IDEA中能快速使用Tomcat呢?

在IDEA中集成使用Tomcat有两种方式，分别是==集成本地Tomcat==和==Tomcat Maven插件==

#### 3.4.1 集成本地Tomcat

目标： 将刚才本地安装好的Tomcat8集成到IDEA中，完成项目部署，具体的实现步骤

1. 打开添加本地Tomcat的面板

   ![1627229992900](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627229992900.png)

2. 指定本地Tomcat的具体路径

   ![1627230313062](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627230313062.png)

3. 修改Tomcat的名称，此步骤可以不改，只是让名字看起来更有意义，HTTP port中的端口也可以进行修改，比如把8080改成80

   ![1627230366658](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627230366658.png)

4. 将开发项目部署项目到Tomcat中

   ![1627230913259](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627230913259.png)

   扩展内容： xxx.war和 xxx.war exploded这两种部署项目模式的区别?

   * war模式是将WEB工程打成war包，把war包发布到Tomcat服务器上

   * war exploded模式是将WEB工程以当前文件夹的位置关系发布到Tomcat服务器上
   * war模式部署成功后，Tomcat的webapps目录下会有部署的项目内容
   * war exploded模式部署成功后，Tomcat的webapps目录下没有，而使用的是项目的target目录下的内容进行部署
   * 建议大家都选war模式进行部署，更符合项目部署的实际情况

5. 部署成功后，就可以启动项目，为了能更好的看到启动的效果，可以在webapp目录下添加a.html页面

   ![1627233265351](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627233265351.png)

6. 启动成功后，可以通过浏览器进行访问测试

   ![1627232743706](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627232743706.png)

7. 最终的注意事项

   ![1627232916624](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627232916624.png)

   

至此，IDEA中集成本地Tomcat进行项目部署的内容我们就介绍完了，整体步骤如下，大家需要按照流程进行部署操作练习。

![1627205657117](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627205657117.png)

 #### 3.4.2 Tomcat Maven插件

在IDEA中使用本地Tomcat进行项目部署，相对来说步骤比较繁琐，所以我们需要一种更简便的方式来替换它，那就是直接使用Maven中的Tomcat插件来部署项目，具体的实现步骤，只需要两步，分别是：

1. 在pom.xml中添加Tomcat插件

   ```xml
   <build>
       <plugins>
       	<!--Tomcat插件 -->
           <plugin>
               <groupId>org.apache.tomcat.maven</groupId>
               <artifactId>tomcat7-maven-plugin</artifactId>
               <version>2.2</version>
           </plugin>
       </plugins>
   </build>
   ```

2. 使用Maven Helper插件快速启动项目，选中项目，右键-->Run Maven --> tomcat7：run

![1627233963315](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627233963315.png)

==注意：==

* 如果选中项目并右键点击后，看不到Run Maven和Debug Maven，这个时候就需要在IDEA中下载Maven Helper插件，具体的操作方式为： File --> Settings --> Plugins --> Maven Helper ---> Install,安装完后按照提示重启IDEA，就可以看到了。

![1627234184076](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1627234184076.png)

* Maven Tomcat插件目前只有Tomcat7版本，没有更高的版本可以使用
* 使用Maven Tomcat插件，要想修改Tomcat的端口和访问路径，可以直接修改pom.xml

```xml
<build>
    <plugins>
    	<!--Tomcat插件 -->
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
            	<port>80</port><!--访问端口号 -->
                <!--项目访问路径
					未配置访问路径： http://localhost:80/tomcat-demo2/a.html
					配置/后访问路径： http://localhost:80/a.html
					如果配置成 /hello,访问路径会变成什么?
						答案： http://localhost:80/hello/a.html
				-->
                <path>/</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**小结**

通过这一节的学习，大家要掌握在IDEA中使用Tomcat的两种方式，集成本地Tomcat和使用Maven的Tomcat插件。后者更简单，推荐大家使用，但是如果对于Tomcat的版本有比较高的要求，要在Tomcat7以上，这个时候就只能用前者了。