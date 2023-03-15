[TOC]



# JavaWeb

## 1.HTTP协议

**什么是协议?**
协议是双方或多方相互之间约定好，大家都需要遵守的规则。叫协议。

**什么是HTTP协议?**
HTTP（hypertext transport protocol），即超文本传输协议。这个协议详细规定了浏览器和万维网服务器之间互相通信的规则。

HTTP协议是客户端和服务器之间通信时数据都需要遵守的规则。

客户端和服务器之间数据通信的内容，也叫报文。

HTTP就是一个通信规则，这个规则规定了客户端发送给服务器的报文格式，

也规定了服务器发送给客户端的报文格式。实际我们要学习的就是这两种报文。

客户端发送给服务器的称为”请求报文“

服务器发送给客户端的称为”响应报文“

## 2.请求数据格式

**请求首行；**
**请求头信息；**
**空行；**
**请求体；**



**get：获取数据**

==因为参数会放在url中，所以隐私性，安全性较差，请求的数据长度是有限制的==

**post：提交数据**

==post请求是没有的长度限制，请求数据是放在body中==

**深入理解**

1…GET 和 POST都是http请求方式， 底层都是 TCP/IP协议；通常GET 产生一个 TCP 数据包；POST 产生两个 TCP 数据包（但firefox是发送一个数据包）

2.对于 GET 方式的请求，浏览器会把 http header 和 data 一并发送出去，服务器响应 200（返回数据）表示成功；

而对于 POST，浏览器先发送 header，服务器响应 100， 浏览器再继续发送 data，服务器响应 200 （返回数据）。


## 3.响应数据格式

响应的HTTP协议格式
响应首行
响应头信息
空行
响应体



## 4.常见响应码

响应码对浏览器来说很重要，它告诉浏览器响应的结果；
200：请求成功，浏览器会把响应体内容（通常是html）显示在浏览器中；
404：请求的资源没有找到，说明客户端错误的请求了不存在的资源；
500：请求资源找到了，但服务器内部出现了错误；
302：重定向，当响应码为302时，表示服务器要求浏览器重新再发一个请求，服务器会发送一个响应头Location，它指定了新请求的URL地址；





# 前端基本语法格式

## 1.JavaScript





## 2.VUE

el：挂载点

data：数据

methods：方法

v-on：绑定事件 简写@

v-text：设置元素的文本值简写为{{}}

v-show：根据真假切换显示状态

v-if：根据元素真假切换元素显示状态（操作dom元素）

>DOM 代表文档对象模型。它是一个编程接口，允许我们从文档中创建、更改或删除元素。我们还可以向这些元素添加事件，使我们的页面更加动态。
>
>DOM 将 HTML 文档视为节点树。一个节点代表一个 HTML 元素。

v-bind：设置元素的属性

v-for：根据数据生成列表结构，语法是（item，index） in 数据

v-modul：便捷的设置和获取表单元素的值，绑定数据会和表单元素值相关联



## 3.ajax

## 4.Javascript

1.上方弹出框：alert("Hello World!"); 

2.显示时间：getElementById('demo').innerHTML=Date()

3.let 声明的变量只在 let 命令所在的代码块内有效。

   const 声明一个只读的常量，一旦声明，常量的值就不能改变。

4.定时器：

```
setTimeout(function () {
    document.getElementById("demo1").innerHTML="RUNOOB-1!";
}, 3000);
```

5.DOM元素控制：

```
<body>
<p id="intro">你好世界!</p>
<p>该实例展示了 <b>getElementById</b> 方法!</p>
<script>
	x=document.getElementById("intro");
	document.write("<p>文本来自 id 为 intro 段落: " + x.innerHTML + "</p>");
</script>
</body>


<body>
<p id="p1">Hello World!</p>
<p id="p2">Hello World!</p>
<script>
	document.getElementById("p2").style.color="blue";
	document.getElementById("p2").style.fontFamily="Arial";
	document.getElementById("p2").style.fontSize="larger";
</script>
<p>以上段落通过脚本修改。</p>
</body>
```



# Servlet

## 1.简介

1.[Servlet](https://so.csdn.net/so/search?q=Servlet&spm=1001.2101.3001.7020) 是 JavaWeb 三大组件之一。三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器。Servlet 是运行在服务器上的一个 java 小程序，==它可以接收客户端发送过来的请求，并响应数据给客户端

## 2.生命周期

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

- Servlet 通过调用 **init ()** 方法进行初始化。
- Servlet 调用 **service()** 方法来处理客户端的请求。
- Servlet 通过调用 **destroy()** 方法终止（结束）。
- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

![image-20230310133737408](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230310133737408.png)



![image-20230310133930318](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230310133930318.png)























