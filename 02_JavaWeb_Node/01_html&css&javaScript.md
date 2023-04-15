[TOC]



# 一.Web开发介绍

## 1 什么是web开发

**Web**：全球广域网，也称为**万维网**(www **W**orld **W**ide **W**eb)，能够通过浏览器访问的**网站**。

所以**Web开发**说白了，就是**开发网站**的，例如下图所示的网站：**淘宝**，**京东**等等

![1667546541068](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667546541068.png)



那么我们知道了web开发是开发网站的，那么我们需要学习哪些知识呢？以及这些知识在我们整个网站开发中占据什么位置呢？对于这些问题，我们就必须知道网站整体的工作流程。



## 2 网站的工作流程

接下来我们先来看看网站的工作流程，这样才能在我们的脑海中构建初步的知识架构体系。

1.首先我们需要通过**浏览器**访问发布到**前端服务器**中的**前端程序**，这时候前端程序会将前端代码返回给浏览器。如下图所示：

![1667546920773](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667546920773.png)



2.浏览器得到前端代码，此时浏览器会将前端代码进行解析，然后展示到浏览器的窗口中，这时候我们就看到了**网站**的**页面**，如下图所示：

![1667547421140](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667547421140.png)

3.但是此时这个页面是没有数据的，因为数据在我们的数据库中，所以我们浏览器需要根据**前端代码中指定的后台服务器的地址** 向 我们的**后台服务器**（内部有java程序）发起**请求**，后台服务器再去从**数据库**中获取数据，然后返回给浏览器。

![1667547561387](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667547561387.png)

4.浏览器拿到后台返回的数据后，然后将数据展示在前端资源也就是**网页**上，然后我们就看到了如下图所示的完整的内容



![1667547604517](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667547604517.png)



**整个流程如下：**

1.浏览器先向前端服务器请求**前端资源**，也就是我们所说的**网页**

2.浏览器再向**后台服务器**发起请求，获取**数据**

3.浏览器将得到的后台**数据**填充到**网页**上，然后展示给用户去看

## 3 网站的开发模式

接下来我们来聊聊网站的开发模式，主要有2种：前端台分离和混合开发

**前后台分离**：（**目前企业开发的主流，**市场占有率70%以上）这种开发模式的特点如下

- 前端人员开发前端程序，前端程序单独部署到前端服务器上

- 后端人员开开发后端程序，后端程序单独部署到后端服务器上


![1667548530745](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667548530745.png)

**混合开发：**（早期的开发技术，目前慢慢退出市场），这种开发模式的特点是：前端人员开发的代码和后端人员开发的代码在同一个项目中，一起打包部署。

![1667548590602](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667548590602.png)

## 4 网站的开发技术

最后我们来看看web阶段需要学习哪些技术呢？如下图我们列举了课程中需要学习的知识点

![1667548969631](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1667548969631.png)

以下是图表的方式整理了我们web阶段要学习的技术和其对应的作用

前端web开发：

| 技术       | 描述                                          |
| ---------- | --------------------------------------------- |
| HTML       | 用于构建网站的基础结构的                      |
| css        | 用于美化页面的，作用和化妆或者整容作用一样    |
| JavaScript | 实现网页和用户的交互                          |
| Vue        | 主要用于将数据填充到html页面上的              |
| Element    | 主要提供了一些非常美观的组件                  |
| Nginx      | 一款web服务器软件，可以用于部署我们的前端工程 |

后端web开发：

| 技术       | 描述                                           |
| ---------- | ---------------------------------------------- |
| Maven      | 一款java中用于管理项目的软件                   |
| Mysql      | 最常用的一款数据库软件之一                     |
| SpringBoot | spring家族的产品，当前最为主流的项目开发技术。 |
| Mybatis    | 用于操作数据库的框架                           |

所以只有我们学完上述的技术，我们才能开发出一个麻雀虽小，五脏俱全的网站。

​	



# 二.HTML

## 介绍

HTML 是一门语言，所有的网页都是用HTML 这门语言编写出来的，也就是HTML是用来写网页的，像京东，12306等网站有很多网页。

![image-20210811151737929](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811151737929.png)

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811151658928.png" alt="image-20210811151658928" style="zoom:80%;" />

这些都是网页展示出来的效果。而HTML也有专业的解释

==HTML(HyperText Markup Language)：超文本标记语言：==

* 超文本：超越了文本的限制，比普通文本更强大。除了文字信息，还可以定义图片、音频、视频等内容

  如上图看到的页面，我们除了能看到一些文字，同时也有大量的图片展示；有些网页也有视频，音频等。这种展示效果超越了文本展示的限制。

* 标记语言：由标签构成的语言

  之前学习的XML就是标记语言，由一个一个的标签组成，HTML 也是由标签组成 。我们在浏览器页面右键可以查看页面的源代码，如下

  ![image-20210811152519471](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811152519471.png)

  可以看到如下内容，就是由一个一个的标签组成的

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811152558939.png" alt="image-20210811152558939" style="zoom:80%;" />

这些标签不像XML那样可以自定义，==HTML中的标签都是预定义好的，运行在浏览器上并由浏览器解析，==然后展示出对应的效果。例如我们想在浏览器上展示出图片就需要使用预定义的 `img` 标签；想展示可以点击的链接的效果就可以使用预定义的 `a` 标签等。

HTML 预定义了很多标签，由于我们是Java工程师、是做后端开发，所以不会每个都学习，页面开发是有专门的前端工程来开发。那为什么我们还要学习呢？在公司中或多或少大家也会涉及到前端开发。

简单的给大家聊一下开发流程：

以后我们是通过Java程序从数据库中查询出来数据，然后交给页面进行展示，这样用户就能通过在浏览器通过页面看到数据。

==W3C标准：==

W3C是万维网联盟，这个组成是用来定义标准的。他们规定了一个网页是由三部分组成，分别是：

* 结构：对应的是 HTML 语言
* 表现：对应的是 CSS 语言
* 行为：对应的是 JavaScript 语言

HTML定义页面的整体结构；CSS是用来美化页面，让页面看起来更加美观；JavaScript可以使网页动起来，比如轮播图也就是多张图片自动的进行切换等效果。

为了更好的给大家表述这三种语言的作用。我们通过具体的页面给大家说明。

如下只是使用HTML语言编写的页面的结构：

![image-20210811155026210](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811155026210.png)

可以看到页面是比较丑的，但是每一部分其实都已经包含了。接下来咱们加上 CSS 进行美化看到的效果如下：

![image-20210811155211181](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811155211181.png)

瞬间感觉好看多了，这就是CSS的作用，用来美化页面的。接下来再加上JavaScript试试

![image-20210811155404506](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811155404506.png)

在上图中可以看到多了轮播图，在浏览器上它是会自动切换图片的，并且切换的动态效果是很不错的。

看到了前端编写的这三个技术效果后，我们今天学习的是HTML，学习HTML其实就是学习预定义的这些标签。

## 快速入门

### 需求：编写如下图效果的页面

![image-20230201160255644](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201160255644.png)

要实现这个页面，我们需要从以下三步进行实现

* 新建文本文件，后缀名改为 .html

  页面文件的后缀名是 .html，所以需要该后缀名

* 编写 HTML 结构标签

  HTML 是由一个一个的标签组成的，但是它也用于表示结构的标签

  ```html
  <html>
  	<head>
      	<title> </title>
      </head>
      <body>
          
      </body>
  </html>
  ```

  html标签是根标签，下面有 `head` 标签和 `body` 标签这两个子标签。而 `head` 标签的 `title` 子标签是用来定义页面标题名称的，它定义的内容会展示在浏览器的标题位置，如下图红框标记

  ![image-20230201160309359](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201160309359.png)

  `body` 标签的内容会被展示在内容区中，如下图红框标记

  ![image-20230201160319668](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201160319668.png)

  

* 在<body>中定义文字

### 代码如下：

```html
<html>
	<head>
    	<title>html 快速入门</title>
    </head>
    <body>
        这是内容页面...
    </body>
</html>
```

同学们在访问其他网站页面时会看到字体颜色是五颜六色的，我们可以该字体颜色吗？当然可以了

`font` 标签就可以使用，该标签有一个 `color` 属性可以设置字体颜色，如： <font color='red'></font> 就是将文字设置成了红颜色。那么我们只需要将需要变成红色的文字放在标签体部分就可以了，如下：

```html
<html>
	<head>
    	<title>html 快速入门</title>
    </head>
    <body>
        <font color='red'>这是内容页面...</font>
    </body>
</html>
```

### 总结：

* HTML 文件以.htm或.html为扩展名

* HTML 结构标签

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811161810610.png" alt="image-20210811161810610" style="zoom:70%;" />

* HTML 标签不区分大小写

  如上案例中的 `font` 写成 `Font` 也是一样可以展示出对应的效果的。

* HTML 标签属性值 单双引皆可

  如上案例中的color属性值使用双引号也是可以的。<font color="red"></font> 

* HTML 语法松散

  比如 font 标签不加结束标签也是可以展示出效果的。但是建议同学们在写的时候还是不要这样做，严格按照要求去写。



## 基础标签

基础标签就是一些和文字相关的标签，如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811171740881.png" alt="image-20210811171740881" style="zoom:80%;" />

接下来我们挨个进行讲解

### 标题标签

* 创建模块

  在 Idea 中创建模块，而我们现在不需要写java代码，所以 `src` 目录就可以删除掉。在模块下创建一个html文件夹，该我们今天的所以的页面文件所部放在该文件夹下。模块目录如下

  ![image-20210811172254664](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811172254664.png)

* 创建页面文件

  选中 `html` 文件夹右键创建页面文件（01-基础标签.html）

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811172511287.png" alt="image-20210811172511287" style="zoom:80%;" />

  创建好后 idea 会自动加上结构标签，如下

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811172704525.png" alt="image-20210811172704525" style="zoom:80%;" />

  我们只需要在 `body` 标签中书写标签。

* 书写标题标签

  标题标签中 h1最大，h6最小。

  ```html
  <h1>我是标题 h1</h1>
  <h2>我是标题 h2</h2>
  <h3>我是标题 h3</h3>
  <h4>我是标题 h4</h4>
  <h5>我是标题 h5</h5>
  <h6>我是标题 h6</h6>
  ```

* 通过浏览器查看效果

  idea 提供了快捷的打开方式，如下图

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811172942861.png" alt="image-20210811172942861" style="zoom:80%;" />

  浏览器展示效果如下：

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811173034453.png" alt="image-20210811173034453" style="zoom:80%;" />

### hr标签

`hr` 标签在浏览器中呈现出 横线 的效果。

在页面文件中书写 hr 标签

```
<hr>
```

效果如下：

![image-20210811173605496](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811173605496.png)



### 字体标签

font：字体标签

* face 属性：用来设置字体。如 "楷体"、"宋体"等

* color 属性：设置文字颜色。颜色有三种表示方式

  * **英文单词**：red,pink,blue...

    这种方式表示的颜色特别有限，所以一般不用。

  * **rgb(值1,值2,值3)**：值的取值范围：0~255  

    此种方式也就是三原色（红绿蓝）设置方式。 例如： rgb(255,0,0)。

    这种书写起来比较麻烦，一般不用。

  * **#值1值2值3**：值的范围：00~FF

    这种方式是rgb方式的简化写法，以后基本都用此方式。

    值1表示红色的范围，值2表示绿色的范围，值3表示蓝色范围。例如： #ff0000

* size 属性：设置文字大小

代码演示：

```html
<font face="楷体" size="5" color="#ff0000">彩色字体</font>
```

效果如下：

 <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230201160850087.png" alt="image-20230201160850087" style="zoom:50%;" />

> 注意：
>
> font 标签已经不建议使用了，以后如果要改变文字字体，大小，颜色可以使用 CSS 进行设置。

### 换行标签

在页面文件中书写如下内容

```
刚察草原绿草如茵，沙柳河水流淌入湖。藏族牧民索南才让家中，茶几上摆着馓子、麻花和水果，炉子上刚煮开的奶茶香气四溢……

6月8日下午，习近平总书记来到青海省海北藏族自治州刚察县沙柳河镇果洛藏贡麻村，走进牧民索南才让家中，看望慰问藏族群众。
```

 在浏览器展示的效果如下：

![image-20210811175442896](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811175442896.png)

我们可以看到并没有换行。如果要实现换行效果，需要使用 换行标签（br标签）。

修改页面文件内容如下：

```
刚察草原绿草如茵，沙柳河水流淌入湖。藏族牧民索南才让家中，茶几上摆着馓子、麻花和水果，炉子上刚煮开的奶茶香气四溢……<br>

6月8日下午，习近平总书记来到青海省海北藏族自治州刚察县沙柳河镇果洛藏贡麻村，走进牧民索南才让家中，看望慰问藏族群众。
```

浏览器打开效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811175649409.png" alt="image-20210811175649409" style="zoom:80%;" />

现在就有换行效果了。

### 段落标签

上面文字展示的效果还是不太好，我们想让每一段上下都加空行。此时就需要使用段落标签（p标签）

在页面文件中书写如下内容：

```html
<p>
刚察草原绿草如茵，沙柳河水流淌入湖。藏族牧民索南才让家中，茶几上摆着馓子、麻花和水果，炉子上刚煮开的奶茶香气四溢……
</p>
<p>
6月8日下午，习近平总书记来到青海省海北藏族自治州刚察县沙柳河镇果洛藏贡麻村，走进牧民索南才让家中，看望慰问藏族群众。
</p>
```

在浏览器展示的效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811180041023.png" alt="image-20210811180041023" style="zoom:80%;" />

这种效果就会比之前的效果好一些，呈现出段落的效果。

### 加粗、斜体、下划线标签

* b：加粗标签
* i：斜体标签
* u：下划线标签，在文字的下方有一条横线

代码如下：

```html
<b>沙柳河水流淌</b><br>
<i>沙柳河水流淌</i><br>
<u>沙柳河水流淌</u><br>
```

在浏览器展示的效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811180336928.png" alt="image-20210811180336928" style="zoom:80%;" />

### 居中标签

center ：文本居中

代码如下：

```html
<hr>
<center>
    <b>沙柳河水流淌</b>
</center>
```

在浏览器效果如下：

![image-20210811180702247](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811180702247.png)





> 注意：在上图页面中版权所有里有特殊字符，需要使用转义字符。有如下转义字符：
>
> <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811180929858.png" alt="image-20210811180929858" style="zoom:70%;" /> 

## 图片、音频、视频标签

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811181303117.png" alt="image-20210811181303117" style="zoom:70%;" />

* img：定义图片

  * src：规定显示图像的 URL（统一资源定位符）

  * height：定义图像的高度

  * width：定义图像的宽度

* audio：定义音频。支持的音频格式：MP3、WAV、OGG 

  * src：规定音频的 URL

  * controls：显示播放控件

* video：定义视频。支持的音频格式：MP4, WebM、OGG
  * src：规定视频的 URL
  * controls：显示播放控件

**尺寸单位：**

height属性和width属性有两种设置方式：

* 像素：单位是px
* 百分比。占父标签的百分比。例如宽度设置为 50%，意思就是占它的父标签宽度的一般（50%）

**资源路径：**

图片，音频，视频标签都有src属性，而src是用来指定对应的图片，音频，视频文件的路径。此处的图片，音频，视频就称为资源。资源路径有如下两种设置方式：

* 绝对路径：完整路径

  这里的绝对路径是网络中的绝对路径。 格式为： 协议://ip地址:端口号/资源名称。

  如：

  ```
  <img src="https://th.bing.com/th/id/R33674725d9ae34f86e3835ae30b20afe?rik=Pb3C9e5%2b%2b3a9Vw&riu=http%3a%2f%2fwww.desktx.com%2fd%2ffile%2fwallpaper%2fscenery%2f20180626%2f4c8157d07c14a30fd76f9bc110b1314e.jpg&ehk=9tpmnrrRNi0eBGq3CnhwvuU8PPmKuy1Yma0zL%2ba14T0%3d&risl=&pid=ImgRaw" width="300" height="400">
  ```

  这里src属性的值就是网络中的绝对路径。

* 相对路径：相对位置关系

  找页面和其他资源的相对路径。

  > ./    表示当前路径
  >
  > ../   表示上一级路径
  >
  > ../../   表示上两级路径

  如模块目录结构如下：

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811190840184.png" alt="image-20210811190840184" style="zoom:80%;" />

  在 `01-基础标签.html` 里的标签中找不同的图片，路径写法不同

  ```html
  <!--在该页面找a.jpg，就需要先回到上一级目录，该级目录有img目录，进入该目录就可以找到 a.jpg图片-->
  <img src="../img/a.jpg" width="300" height="400">
  <!--该页面和aa.jpg 是在同一级下，所以可以直接写 图片的名称，也可以写成  ./aa.jpg-->
  <img src="aa.jpg" width="300" height="400">
  ```

使用这些标签的代码如下：

```html
<img src="../img/a.jpg" width="300" height="400">
<audio src="b.mp3" controls></audio>
<video src="c.mp4" controls width="500" height="300"></video>
```

在浏览器展示的效果如下：

![image-20210811191514642](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811191514642.png)

## 超链接标签

在网页中可以看到很多超链接标签，如下

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811191725308.png" alt="image-20210811191725308" style="zoom:80%;" />

上图红框中的都是超链接，当我们点击这些超链接时会跳转到其他的页面或者资源。而超链接使用的是 `a` 标签。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811191852726.png" alt="image-20210811191852726" style="zoom:70%;" />

`a` 标签属性：

* href：指定访问资源的URL 

* target：指定打开资源的方式
  * _self：默认值，在当前页面打开
  * _blank：在空白页面打开

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
	<a href="https://www.baidu.com" target="_self">点我有惊喜</a>
</body>
</html>
```



当我们将 `target` 属性值设置为 `_blank`，会新打开一个Tab页展示新的网页。

## 列表标签

HTML 中列表分为

* 有序列表

  如下图，页面效果中是有标号对每一项进行标记的。

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811192825145.png" alt="image-20210811192825145" style="zoom:80%;" />

* 无序列表

  如下图，页面效果中没有标号对每一项进行标记，而是使用 点 进行标记。

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811192905834.png" alt="image-20210811192905834" style="zoom:80%;" />

**标签说明：**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811193105881.png" alt="image-20210811193105881" style="zoom:60%;" />

有序列表中的 `type` 属性用来指定标记的标号的类型（数字、字母、罗马数字等）

无序列表中的 `type` 属性用来指定标记的形状

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ol type="A">
        <li>咖啡</li>
        <li>茶</li>
        <li>牛奶</li>
    </ol>
    
    <ul type="circle">
        <li>咖啡</li>
        <li>茶</li>
        <li>牛奶</li>
    </ul>
</body>
</html>
```

## 表格标签

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811193819851.png" alt="image-20210811193819851" style="zoom:80%;" />

如上图就是一个表格，表格可以使用如下标签定义

* table ：定义表格

  * border：规定表格边框的宽度

  * width ：规定表格的宽度

  * cellspacing：规定单元格之间的空白

* tr ：定义行

  * align：定义表格行的内容对齐方式

* td ：定义单元格

  * rowspan:规定单元格可横跨的行数

  * colspan:规定单元格可横跨的列数

* th：定义表头单元格

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<table border="1" cellspacing="0" width="500">
    <tr>
        <th>序号</th>
        <th>品牌logo</th>
        <th>品牌名称</th>
        <th>企业名称</th>
    </tr>
    <tr align="center">
        <td>010</td>
        <td><img src="../img/三只松鼠.png" width="60" height="50"></td>
        <td>三只松鼠</td>
        <td>三只松鼠</td>
    </tr>

    <tr align="center">
        <td>009</td>
        <td><img src="../img/优衣库.png" width="60" height="50"></td>
        <td>优衣库</td>
        <td>优衣库</td>
    </tr>

    <tr align="center">
        <td>008</td>
        <td><img src="../img/小米.png" width="60" height="50"></td>
        <td>小米</td>
        <td>小米科技有限公司</td>
    </tr>
</table>
</body>
</html>
```

## 布局标签

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811194410699.png" alt="image-20210811194410699" style="zoom:80%;" />

这两个标签，一般都是和css结合到一块使用来实现页面的布局。

`div`标签 在浏览器上会有换行的效果，而 `span` 标签在浏览器上没有换行效果。

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div>我是div</div>
    <div>我是div</div>
    <span>我是span</span>
    <span>我是span</span>
</body>
</html>
```

**浏览器效果如下：**

![image-20210811194739313](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210811194739313.png)

## 表单标签

表单标签效果大家其实都不陌生，像登陆页面、注册页面等都是表单。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812215311168.png" alt="image-20210812215311168" style="zoom:80%;" />

像这样的表单就是用来采集用户输入的数据，然后将数据发送到服务端，服务端会对数据库进行操作，比如注册就是将数据保存到数据库中，而登陆就是根据用户名和密码进行数据库的查询操作。

表单是很重要的标签，需要大家重点来学习。

### 表单标签概述

> 表单：在网页中主要负责数据采集功能，使用<form>标签定义表单
>
> 表单项(元素)：不同类型的 input 元素、下拉列表、文本域等

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812215704511.png" alt="image-20210812215704511" style="zoom:80%;" />

`form` 是表单标签，它在页面上没有任何展示的效果。需要借助于表单项标签来展示不同的效果。如下图就是不同的表单项标签展示出来的效果。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812215857298.png" alt="image-20210812215857298" style="zoom:70%;" />

### form标签属性

* **action：规定当提交表单时向何处发送表单数据，该属性值就是URL**

  以后会将数据提交到服务端，该属性需要书写服务端的URL。而今天我们可以书写 `#` ，表示提交到当前页面来看效果。

* **method ：规定用于发送表单数据的方式**

  method取值有如下两种：

  * get：默认值。如果不设置method属性则默认就是该值
    * 请求参数会拼接在URL后边
    * url的长度有限制 4KB
  * post：
    * 浏览器会将数据放到http请求消息体中
    * 请求参数无限制的

### 代码演示

由于表单标签在页面上没有任何展示的效果，所以在演示的过程是会先使用 `input` 这个表单项标签展示输入框效果。

代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form>
        <input type="text">
        <input type="submit">
    </form>
</body>
</html>
```

浏览器展示效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812220926114.png" alt="image-20210812220926114" style="zoom:90%;" /> 

从效果可以看到页面有一个输入框，用户可以在数据框中输入自己想输入的内容，点击提交按钮以后会将数据发送到服务端，当然现在肯定不能实现。现在我们可以将 `form` 标签的 `action` 属性值设置为 `#` ，将其将数据提交到当前页面。还需要注意一点，要想提交数据，`input` 输入框必须设置 `name` 属性。代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="#">
        <input type="text" name="username">
        <input type="submit">
    </form>
</body>
</html>
```

浏览器展示效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812221656295.png" alt="image-20210812221656295" style="zoom:80%;" /> 

在输入框输入 `hehe` ，然后点击 `提交` 按钮，就能看到如下效果

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812221801965.png" alt="image-20210812221801965" style="zoom:80%;" /> 

我们可以看到在浏览器的地址栏的URL后拼接了我们提交的数据。`username` 就是输入框 `name` 属性值，而 `hehe` 就是我们在输入框输入的内容。

接下来我们来聊 `method` 属性，默认是 `method = 'get'`，所以该取值就会将数据拼接到URL的后面。那我们将 `method` 属性值设置为 `post`，浏览器的效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812222334790.png" alt="image-20210812222334790" style="zoom:80%;" /> 

从上图可以看出数据并没有拼接到 URL 后，那怎么看提交的数据呢？我们可以使用浏览器的开发者工具来查看

![image-20210812222623912](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812222623912.png)

按照如上步骤操作能看到如下页面

![image-20210812223004607](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812223004607.png)

重新提交数据后，可以看到提交的数据，如下图

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812223150373.png" alt="image-20210812223150373" style="zoom:80%;" />

## 表单项标签

表单项标签有很多，不同的表单项标签有不同的展示效果。表单项标签可以分为以下三个：

* \<input>：表单项，通过type属性控制输入形式

  `input` 标签有个 `type` 属性。 `type` 属性的取值不同，展示的效果也不一样

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812223956360.png" alt="image-20210812223956360" style="zoom:80%;" />

  

* \<select>：定义下拉列表，\<option> 定义列表项 

  如下图就是下拉列表的效果：

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812223708205.png" alt="image-20210812223708205" style="zoom:80%;" /> 

* \<textarea>：文本域

  如下图就是文本域效果。它可以输入多行文本，而 `input` 数据框只能输入一行文本。

  ![image-20210812223744522](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812223744522.png) 

> ==注意：==
>
> * 以上标签项的内容要想提交，必须得定义 `name` 属性。
> * 每一个标签都有id属性，id属性值是唯一的标识。
> * 单选框、复选框、下拉列表需要使用 `value` 属性指定提交的值。

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="#" method="post">
        <input type="hidden" name="id" value="123">

        <label for="username">用户名：</label>
        <input type="text" name="username" id="username"><br>

        <label for="password">密码：</label>
        <input type="password" name="password" id="password"><br>

        性别：
        <input type="radio" name="gender" value="1" id="male"> <label for="male">男</label>
        <input type="radio" name="gender" value="2" id="female"> <label for="female">女</label>
        <br>

        爱好：
        <input type="checkbox" name="hobby" value="1"> 旅游
        <input type="checkbox" name="hobby" value="2"> 电影
        <input type="checkbox" name="hobby" value="3"> 游戏
        <br>

        头像：
        <input type="file"><br>

        城市:
        <select name="city">
            <option>北京</option>
            <option value="shanghai">上海</option>
            <option>广州</option>
        </select>
        <br>

        个人描述：
        <textarea cols="20" rows="5" name="desc"></textarea>
        <br>
        <br>
        <input type="submit" value="免费注册">
        <input type="reset" value="重置">
        <input type="button" value="一个按钮">
    </form>
</body>
</html>
```

在浏览器的效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812224152747.png" alt="image-20210812224152747" style="zoom:80%;" />

# 三.CSS

##  概述

==CSS 是一门语言，用于控制网页表现。==我们之前介绍过W3C标准。W3C标准规定了网页是由以下组成：

* 结构：HTML
* 表现：CSS
* 行为：JavaScript

CSS也有一个专业的名字：==Cascading Style Sheet（层叠样式表）。==

如下面的代码， `style` 标签中定义的就是css代码。该代码描述了将 div 标签的内容的字体颜色设置为 红色。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div{
            color: red;
        }
    </style>
</head>
<body>
    <div>Hello CSS~</div>
</body>
</html>
```

在浏览器中的效果如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812225424174.png" alt="image-20210812225424174" style="zoom:60%;" />

## css 导入方式

css 导入方式其实就是 css 代码和 html 代码的结合方式。CSS 导入 HTML有三种方式：

* 内联样式：在标签内部使用style属性，属性值是css属性键值对

  ```html
  <div style="color: red">Hello CSS~</div>
  ```

  > 给方式只能作用在这一个标签上，如果其他的标签也想使用同样的样式，那就需要在其他标签上写上相同的样式。复用性太差。

* 内部样式：定义<style>标签，在标签内部定义css样式

  ```html
  <style type="text/css">
  	div{
  		color: red;
      }
  </style>
  ```

  > 这种方式可以做到在该页面中复用。

* 外部样式：定义link标签，引入外部的css文件

  编写一个css文件。名为：demo.css，内容如下:

  ```css
  div{
  	color: red;
  }
  ```

  在html中引入 css 文件。

  ```html
  <link rel="stylesheet"  href="demo.css">
  ```

  > 这种方式可以在多个页面进行复用。其他的页面想使用同样的样式，只需要使用 `link` 标签引入该css文件。

**代码演示：**

项目目录结构如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812231514311.png" alt="image-20210812231514311" style="zoom:80%;" />

编写页面 `02-导入方式.html`，内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        span{
            color: red;
        }
    </style>
    <link href="../css/demo.css" rel="stylesheet">
</head>
<body>
    <div style="color: red">hello css</div>

    <span>hello css </span>

    <p>hello css</p>
</body>
</html>
```

## css 选择器

css 选择器就是选取需设置样式的元素（标签），比如如下css代码：

```css
div {
	color:red;
}
```

如上代码中的 `div` 就是 css 中的选择器。我们只讲下面三种选择器：

* 元素选择器

  格式：

  ```css
  元素名称{color: red;}
  ```

  例子：

  ```
  div {color:red}  /*该代码表示将页面中所有的div标签的内容的颜色设置为红色*/
  ```

* id选择器

  格式：

  ```css
  #id属性值{color: red;}
  ```

  例子：

  html代码如下：

  ```html
  <div id="name">hello css2</div>
  ```

  css代码如下：

  ```css
  #name{color: red;}/*该代码表示将页面中所有的id属性值是 name 的标签的内容的颜色设置为红色*/
  ```

* 类选择器

  格式：

  ```css
  .class属性值{color: red;}
  ```

  例子：

  html代码如下：

  ```html
  <div class="cls">hello css3</div>
  ```

  css代码如下：

  ```css
  .cls{color: red;} /*该代码表示将页面中所有的class属性值是 cls 的标签的内容的颜色设置为红色*/
  ```

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div{
            color: red;
        }

        #name{
            color: blue;
        }

        .cls{
            color: pink;
        }
    </style>

</head>
<body>
    <div>div1</div>
    <div id="name">div2</div>
    <div class="cls">div3</div>
    <span class="cls">span</span>
</body>
</html>
```



## css 属性

css属性我们不作为重点讲解。我们简单的看一下css的文档

![image-20210812233107495](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812233107495.png)

css有很多css属性，你要想把它们都学会，需要花费很长的时间。而我们作为java程序员，不需要重点掌握这部分内容。对于网页三剑客中css是对我们要求最低的。给大家简单介绍一下文档怎么查看即可，如下我们看一个 `background-color` 属性

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812233415060.png" alt="image-20210812233415060" style="zoom:80%;" />

点击进去后能看到下面界面

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210812233510734.png" alt="image-20210812233510734" style="zoom:70%;" />

上面就列举了该属性的具体的使用，你也可以点击下面的 `亲自试一试` 看效果。

# 四.JavaScript

## JavaScript简介

==JavaScript 是一门跨平台、面向对象的脚本语言==，而Java语言也是跨平台的、面向对象的语言，只不过Java是编译语言，是需要编译成字节码文件才能运行的；JavaScript是脚本语言，不需要编译，由浏览器直接解析并执行。

JavaScript 是用来控制网页行为的，它能使网页可交互；那么它可以做什么呢？如改变页面内容、修改指定元素的属性值、对表单进行校验等，下面是这些功能的效果展示：

* **改变页面内容**

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814173417834.png" alt="image-20210814173417834" style="zoom:80%;" />

  当我点击上面左图的 `点击我` 按钮，按钮上面的文本就改为上面右图内容，这就是js 改变页面内容的功能。

* **修改指定元素的属性值**

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814173719505.png" alt="image-20210814173719505" style="zoom:70%;" />

  当我们点击上图的 `开灯` 按钮，效果就是上面右图效果；当我点击 `关灯` 按钮，效果就是上面左图效果。其他这个功能中有两张灯泡的图片（使用img标签进行展示），通过修改 img 标签的 src 属性值改变展示的图片来实现。

* **对表单进行校验**

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814174242688.png" alt="image-20210814174242688" style="zoom:70%;" />

  在上面左图的输入框输入用户名，如果输入的用户名是不满足规则的就展示右图(上) 的效果；如果输入的用户名是满足规则的就展示右图(下) 的效果。

JavaScript 和 Java 是完全不同的语言，不论是概念还是设计，只是名字比较像而已。但是==基础语法类似==，所以我们有Java的学习经验，再学习JavaScript 语言就相对比较容易些。

JavaScript（简称：JS） 在 1995 年由 Brendan Eich 发明，并于 1997 年成为一部ECMA 标准。ECMA规定了一套标准就叫 `ECMAScript` ，所有的客户端校验语言必须遵守这个标准，当然 JavaScript 也遵守了这个标准。ECMAScript 6 (简称ES6) 是最新的 JavaScript 版本（发布于 2015 年)，我们的课程就是基于最新的 `ES6` 进行讲解。

## JavaScript引入方式

JavaScript 引入方式就是 HTML 和 JavaScript 的结合方式。JavaScript引入方式有两种：

* 内部脚本：将 JS代码定义在HTML页面中
* 外部脚本：将 JS代码定义在外部 JS文件中，然后引入到 HTML页面中

###  内部脚本

在 HTML 中，JavaScript 代码必须位于 `<script>` 与 `</script>` 标签之间

**代码如下：**

`alert(数据)` 是 JavaScript 的一个方法，作用是将参数数据以浏览器弹框的形式输出出来。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script>
    alert("hello js1");
</script>
</body>
</html>
```

**效果如下：**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814181419691.png" alt="image-20210814181419691" style="zoom:70%;" />

从结果可以看到 js 代码已经执行了。

> ==提示：==
>
> * 在 HTML 文档中可以在任意地方，放置任意数量的<script>标签。如下图
>
>   ```html
>   <!DOCTYPE html>
>   <html lang="en">
>   <head>
>       <meta charset="UTF-8">
>       <title>Title</title>
>       <script>
>           alert("hello js1");
>       </script>
>   </head>
>   <body>
>   
>   <script>
>       alert("hello js1");
>   </script>
>   
>   </body>
>   </html>
>   <script>
>       alert("hello js1");
>   </script>
>   ```
>
> * 一般把脚本置于 <body> 元素的底部，可改善显示速度
>
>   因为浏览器在加载页面的时候会从上往下进行加载并解析。 我们应该让用户看到页面内容，然后再展示动态的效果。

### 外部脚本

**第一步：定义外部 js 文件。如定义名为 demo.js的文件**

项目结构如下：

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814182345236.png" alt="image-20210814182345236" style="zoom:80%;" />

demo.js 文件内容如下：

```js
alert("hello js");
```

**第二步：在页面中引入外部的js文件**

在页面使用 `script` 标签中使用 `src` 属性指定 js 文件的 URL 路径。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script src="../js/demo.js"></script>
</body>
</html>
```

> ==**注意：**==
>
> * 外部脚本不能包含 `<script>` 标签
>
>   在js文件中直接写 js 代码即可，不要在 js文件 中写 `script` 标签
>
> * `<script>` 标签不能自闭合
>
>   在页面中引入外部js文件时，不能写成 `<script src="../js/demo.js" />`。

## JavaScript基础语法

### 书写语法

* 区分大小写：与 Java 一样，变量名、函数名以及其他一切东西都是区分大小写的

* 每行结尾的分号可有可无

  如果一行上写多个语句时，必须加分号用来区分多个语句。

* 注释

  * 单行注释：// 注释内容
  * 多行注释：/* 注释内容 */

  > 注意：JavaScript 没有文档注释

* 大括号表示代码块

  下面语句大家肯定能看懂，和 java 一样 大括号表示代码块。

  ```js
  if (count == 3) { 
     alert(count); 
  } 
  ```

### 输出语句

js 可以通过以下方式进行内容的输出，只不过不同的语句输出到的位置不同

* **使用 window.alert() 写入警告框**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
      
  <script>
      window.alert("hello js");//写入警告框
  </script>
  </body>
  </html>
  ```

  上面代码通过浏览器打开，我们可以看到如下图弹框效果

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814181419691.png" alt="image-20210814181419691" style="zoom:70%;" />

* **使用 document.write() 写入 HTML 输出**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
      
  <script>
      document.write("hello js 2~");//写入html页面
  </script>
  </body>
  </html>
  ```

  上面代码通过浏览器打开，我们可以在页面上看到 `document.write(内容)` 输出的内容

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814190302845.png" alt="image-20210814190302845" style="zoom:80%;" />

* **使用 console.log() 写入浏览器控制台**

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  <script>
      console.log("hello js 3");//写入浏览器的控制台
  </script>
  </body>
  </html>
  ```

  上面代码通过浏览器打开，我们可以在不能页面上看到  `console.log(内容)` 输出的内容，它是输出在控制台了，而怎么在控制台查看输出的内容呢？在浏览器界面按 `F12` 就可以看到下图的控制台

  ![image-20210814190906202](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210814190906202.png)

### 变量

JavaScript 中用 var 关键字（variable 的缩写）来声明变量。格式 `var 变量名 = 数据值;`。而在JavaScript 是一门弱类型语言，变量==可以存放不同类型的值==；如下在定义变量时赋值为数字数据，还可以将变量的值改为字符串类型的数

```js
var test = 20;
test = "张三";
```

js 中的变量名命名也有如下规则，和java语言基本都相同

* 组成字符可以是任何字母、数字、下划线（_）或美元符号（$）
* 数字不能开头
* 建议使用驼峰命名

JavaScript 中 `var` 关键字有点特殊，有以下地方和其他语言不一样

* 作用域：全局变量

  ```js
  {
      var age = 20;
  }
  alert(age);  // 在代码块中定义的age 变量，在代码块外边还可以使用
  ```

* 变量可以重复定义

  ```js
  {
      var age = 20;
      var age = 30;//JavaScript 会用 30 将之前 age 变量的 20 替换掉
  }
  alert(age); //打印的结果是 30
  ```

针对如上的问题，==ECMAScript 6 新增了 `let `关键字来定义变量。==它的用法类似于 `var`，但是所声明的变量，只在 `let` 关键字所在的代码块内有效，且不允许重复声明。

例如：

```js
{
    let age = 20;
}
alert(age); 
```

运行上面代码，浏览器并没有弹框输出结果，说明这段代码是有问题的。通过 `F12` 打开开发者模式可以看到如下错误信息

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815170848426.png" alt="image-20210815170848426" style="zoom:80%;" />

而如果在代码块中定义两个同名的变量，IDEA 开发工具就直接报错了

> <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815170952829.png" alt="image-20210815170952829" style="zoom:80%;" />

==ECMAScript 6 新增了 const关键字，用来声明一个只读的常量。一旦声明，常量的值就不能改变。== 通过下面的代码看一下常用的特点就可以了

> <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815171128095.png" alt="image-20210815171128095" style="zoom:80%;" />

我们可以看到给 PI 这个常量重新赋值时报错了。

### 数据类型

JavaScript 中提供了两类数据类型：`原始类型` 和 `引用类型`。

> 使用 typeof 运算符可以获取数据类型
>
> `alert(typeof age);` 以弹框的形式将 age 变量的数据类型输出

原始数据类型：

* **number**：数字（整数、小数、NaN(Not a Number)）

  ```js
  var age = 20;
  var price = 99.8;
  
  alert(typeof age); // 结果是 ： number
  alert(typeof price);// 结果是 ： number
  ```

  > ==注意：== NaN是一个特殊的number类型的值，后面用到再说

* **string**：字符、字符串，单双引皆可

  ```js
  var ch = 'a';
  var name = '张三'; 
  var addr = "北京'欢迎您‘";
  
  alert(typeof ch); //结果是  string
  alert(typeof name); //结果是  string
  alert(typeof addr); //结果是  string
  ```

  > ==注意：==在 js 中 双引号和单引号都表示字符串类型的数据

* **boolean**：布尔。true，false

  ```js
  var flag = true;
  var flag2 = false;
  
  alert(typeof flag); //结果是 boolean
  alert(typeof flag2); //结果是 boolean
  ```

* **null**：对象为空

  ```js
  var obj = null;
  
  alert(typeof obj);//结果是 object
  ```

  为什么打印上面的 obj 变量的数据类型，结果是object；这个官方给出了解释，下面是从官方文档截的图

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815173003408.png" alt="image-20210815173003408" style="zoom:80%;" />

* **undefined**：当声明的变量未初始化时，该变量的默认值是 undefined

  ```js
  var a ;
  alert(typeof a); //结果是 undefined
  ```

### 运算符

JavaScript 提供了如下的运算符。大部分和 Java语言 都是一样的，不同的是 JS 关系运算符中的 `==` 和 `===`，一会我们只演示这两个的区别，其他运算符将不做演示

* 一元运算符：++，--

* 算术运算符：+，-，*，/，%

* 赋值运算符：=，+=，-=…

* 关系运算符：>，<，>=，<=，!=，\==，===…

* 逻辑运算符：&&，||，!

* 三元运算符：条件表达式 ? true_value : false_value 

####  \==和===区别

**概述:**

* ==：

  1. 判断类型是否一样，如果不一样，则进行类型转换

  2. 再去比较其值

* ===：js 中的全等于

  1. 判断类型是否一样，如果不一样，直接返回false
  2. 再去比较其值

**代码：**

```js
var age1 = 20;
var age2 = "20";

alert(age1 == age2);// true
alert(age1 === age2);// false
```



#### 类型转换

上述讲解 `==` 运算符时，发现会进行类型转换，所以接下来我们来详细的讲解一下 JavaScript 中的类型转换。

* 其他类型转为number

  * string 转换为 number 类型：按照字符串的字面值，转为数字。如果字面值不是数字，则转为NaN

    将 string 转换为 number 有两种方式：

    * 使用 `+` 正号运算符：

      ```js
      var str = +"20";
      alert(str + 1) //21
      ```

    * 使用 `parseInt()` 函数(方法)：

      ```js
      var str = "20";
      alert(parseInt(str) + 1);
      ```

    > ==建议使用 `parseInt()` 函数进行转换。==

  * boolean 转换为 number 类型：true 转为1，false转为0

    ```js
    var flag = +false;
    alert(flag); // 0
    ```

* 其他类型转为boolean

  * number 类型转换为 boolean 类型：0和NaN转为false，其他的数字转为true
  * string 类型转换为 boolean 类型：空字符串转为false，其他的字符串转为true
  * null类型转换为 boolean 类型是 false
  * undefined 转换为 boolean 类型是 false

  **代码如下：**

  ```js
  // var flag = 3;
  // var flag = "";
  var flag = undefined;
  
  if(flag){
      alert("转为true");
  }else {
      alert("转为false");
  }
  ```

**使用场景：**

在 Java 中使用字符串前，一般都会先判断字符串不是null，并且不是空字符才会做其他的一些操作，JavaScript也有类型的操作，代码如下：

```js
var str = "abc";

//健壮性判断
if(str != null && str.length > 0){
    alert("转为true");
}else {
    alert("转为false");
}
```

但是由于 JavaScript 会自动进行类型转换，所以上述的判断可以进行简化，代码如下：

```js
var str = "abc";

//健壮性判断
if(str){
    alert("转为true");
}else {
    alert("转为false");
}
```



### 流程控制语句

JavaScript 中提供了和 Java 一样的流程控制语句，如下

* if 
* switch
* for
* while
* do while

#### if 语句

```js
var count = 3;
if (count == 3) {
    alert(count);
}
```

#### switch 语句

```js
var num = 3;
switch (num) {
    case 1:
        alert("星期一");
        break;
    case 2:
        alert("星期二");
        break;
    case 3:
        alert("星期三");
        break;
    case 4:
        alert("星期四");
        break;
    case 5:
        alert("星期五");
        break;
    case 6:
        alert("星期六");
        break;
    case 7:
        alert("星期日");
        break;
    default:
        alert("输入的星期有误");
        break;
}
```

#### for 循环语句

```js
var sum = 0;
for (let i = 1; i <= 100; i++) { //建议for循环小括号中定义的变量使用let
    sum += i;
}
alert(sum);
```

#### while 循环语句

```js
var sum = 0;
var i = 1;
while (i <= 100) {
    sum += i;
    i++;
}
alert(sum);
```

#### do while 循环语句

```js
var sum = 0;
var i = 1;
do {
    sum += i;
    i++;
}
while (i <= 100);
alert(sum);
```

### 函数

函数（就是Java中的方法）是被设计为执行特定任务的代码块；JavaScript 函数通过 function 关键词进行定义。

#### 定义格式

函数定义格式有两种：

* 方式1

  ```js
  function 函数名(参数1,参数2..){
      要执行的代码
  }
  ```

* 方式2

  ```js
  var 函数名 = function (参数列表){
      要执行的代码
  }
  ```

> ==注意：==
>
> * 形式参数不需要类型。因为JavaScript是弱类型语言
>
>   ```js
>   function add(a, b){
>       return a + b;
>   }
>   ```
>
>   上述函数的参数 a 和 b 不需要定义数据类型，因为在每个参数前加上 var 也没有任何意义。
>
> * 返回值也不需要定义类型，可以在函数内部直接使用return返回即可

#### 函数调用

函数调用函数：

```js
函数名称(实际参数列表);
```

eg：

```js
let result = add(10,20);
```

> ==注意：==
>
> * JS中，函数调用可以传递任意个数参数
>
> * 例如  `let result = add(1,2,3);` 
>
>   它是将数据 1 传递给了变量a，将数据 2 传递给了变量 b，而数据 3 没有变量接收。

## JavaScript常用对象

JavaScript 提供了很多对象供使用者来使用。这些对象总共分类三类

* 基本对象

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815183147297.png" alt="image-20210815183147297" style="zoom:80%;" />

* BOM 对象

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815183207660.png" alt="image-20210815183207660" style="zoom:80%;" />

* DOM对象

  DOM 中的对象就比较多了，下图只是截取部分

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815183225718.png" alt="image-20210815183225718" style="zoom:80%;" />

这小节我们先学习基本对象，而我们先学习 `Array` 数组对象和 `String` 字符串对象。

### Array对象

JavaScript Array对象用于定义数组

####  定义格式

数组的定义格式有两种：

* 方式1

  ```js
  var 变量名 = new Array(元素列表); 
  ```

  例如：

  ```js
  var arr = new Array(1,2,3); //1,2,3 是存储在数组中的数据（元素）
  ```

* 方式2

  ```js
  var 变量名 = [元素列表];
  ```

  例如：

  ```js
  var arr = [1,2,3]; //1,2,3 是存储在数组中的数据（元素）
  ```

  ==注意：Java中的数组静态初始化使用的是{}定义，而 JavaScript 中使用的是 [] 定义==

#### 元素访问

访问数组中的元素和 Java 语言的一样，格式如下：

```js
arr[索引] = 值;
```

**代码演示：**

```js
 // 方式一
var arr = new Array(1,2,3);
// alert(arr);

// 方式二
var arr2 = [1,2,3];
//alert(arr2);

// 访问
arr2[0] = 10;
alert(arr2)
```

#### 特点

JavaScript 中的数组相当于 Java 中集合。数组的长度是可以变化的，而 JavaScript 是弱类型，所以可以存储任意的类型的数据。

例如如下代码：

```js
// 变长
var arr3 = [1,2,3];
arr3[10] = 10;
alert(arr3[10]); // 10
alert(arr3[9]);  //undefined
```

上面代码在定义数组中给了三个元素，又给索引是 10 的位置添加了数据 10，那么 `索引3` 到 `索引9` 位置的元素是什么呢？我们之前就介绍了，在 JavaScript 中没有赋值的话，默认就是 `undefined`。

如果给 `arr3` 数组添加字符串的数据，也是可以添加成功的

```js
arr3[5] = "hello";
alert(arr3[5]); // hello
```

#### 属性

Array 对象提供了很多属性，如下图是官方文档截取的

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815190319166.png" alt="image-20210815190319166" style="zoom:80%;" />

而我们只讲解 `length` 属性，该数组可以动态的获取数组的长度。而有这个属性，我们就可以遍历数组了

```js
var arr = [1,2,3];
for (let i = 0; i < arr.length; i++) {
    alert(arr[i]);
}
```

#### 方法

Array 对象同样也提供了很多方法，如下图是官方文档截取的

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815190601340.png" alt="image-20210815190601340" style="zoom:80%;" />

而我们在课堂中只演示 `push` 函数和 `splice` 函数。

* push 函数：给数组添加元素，也就是在数组的末尾添加元素

  参数表示要添加的元素

  ```js
  // push:添加方法
  var arr5 = [1,2,3];
  arr5.push(10);
  alert(arr5);  //数组的元素是 {1,2,3,10}
  ```

* splice 函数：删除元素

  参数1：索引。表示从哪个索引位置删除

  参数2：个数。表示删除几个元素

  ```js
  // splice:删除元素
  var arr5 = [1,2,3];
  arr5.splice(0,1); //从 0 索引位置开始删除，删除一个元素 
  alert(arr5); // {2,3}
  ```

### String对象

String对象的创建方式有两种

* 方式1：

  ```js
  var 变量名 = new String(s); 
  ```

* 方式2：

  ```js
  var 变量名 = "数组"; 
  ```

**属性：**

String对象提供了很多属性，下面给大家列举了一个属性 `length` ，该属性是用于动态的获取字符串的长度

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815192504884.png" alt="image-20210815192504884" style="zoom:60%;" />

**函数：**

String对象提供了很多函数（方法），下面给大家列举了两个方法。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815192544172.png" alt="image-20210815192544172" style="zoom:70%;" />

String对象还有一个函数 `trim()` ，该方法在文档中没有体现，但是所有的浏览器都支持；它是用来去掉字符串两端的空格。

代码演示：

```js
var str4 = '  abc   ';
alert(1 + str4 + 1);
```

上面代码会输出内容 `1  abc  1`，很明显可以看到 abc 字符串左右两边是有空格的。接下来使用 `trim()` 函数

```js
var str4 = '  abc   ';
alert(1 + str4.trim() + 1);
```

输出的内容是 `1abc1` 。这就是 `trim()` 函数的作用。

`trim()` 函数在以后开发中还是比较常用的，例如下图所示是登陆界面

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815193420418.png" alt="image-20210815193420418" style="zoom:80%;"/> 

用户在输入用户名和密码时，可能会习惯的输入一些空格，这样在我们后端程序中判断用户名和密码是否正确，结果肯定是失败。所以我们一般都会对用户输入的字符串数据进行去除前后空格的操作。

### 自定义对象

在 JavaScript 中自定义对象特别简单，下面就是自定义对象的格式：

```js
var 对象名称 = {
    属性名称1:属性值1,
    属性名称2:属性值2,
    ...,
    函数名称:function (形参列表){},
	...
};
```

调用属性的格式：

```js
对象名.属性名
```

调用函数的格式：

```js
对象名.函数名()
```

接下来通过代码演示一下，让大家体验一下 JavaScript 中自定义对象

```js
var person = {
        name : "zhangsan",
        age : 23,
        eat: function (){
            alert("干饭~");
        }
    };


alert(person.name);  //zhangsan
alert(person.age); //23

person.eat();  //干饭~
```

## BOM

BOM：Browser Object Model 浏览器对象模型。也就是 JavaScript 将浏览器的各个组成部分封装为对象。

我们要操作浏览器的各个组成部分就可以通过操作 BOM 中的对象来实现。比如：我现在想将浏览器地址栏的地址改为 `https://www.itheima.com` 就可以通过使用 BOM 中定义的 `Location` 对象的 `href` 属性，代码： `location.href = "https://itheima.com";` 

 BOM 中包含了如下对象：

* Window：浏览器窗口对象
* Navigator：浏览器对象
* Screen：屏幕对象
* History：历史记录对象
* Location：地址栏对象

下图是 BOM 中的各个对象和浏览器的各个组成部分的对应关系

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815194911914.png" alt="image-20210815194911914" style="zoom:70%;" />

BOM 中的 `Navigator` 对象和 `Screen` 对象基本不会使用，所以我们的课堂只对 `Window`、`History`、`Location` 对象进行讲解。

### Window对象

window 对象是 JavaScript 对浏览器的窗口进行封装的对象。

#### 获取window对象

该对象不需要创建直接使用 `window`，其中 `window. ` 可以省略。比如我们之前使用的 `alert()` 函数，其实就是 `window` 对象的函数，在调用是可以写成如下两种

* 显式使用 `window` 对象调用

  ```js
  window.alert("abc");
  ```

* 隐式调用

  ```
  alert("abc")
  ```

#### window对象属性

`window` 对象提供了用于获取其他 BOM 组成对象的属性

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815200625592.png" alt="image-20210815200625592" style="zoom:80%;" />

也就是说，我们想使用 `Location` 对象的话，就可以使用 `window` 对象获取；写成 `window.location`，而 `window.` 可以省略，简化写成 `location` 来获取 `Location` 对象。

#### window对象函数

`window` 对象提供了很多函数供我们使用，而很多都不常用；下面给大家列举了一些比较常用的函数

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815201323329.png" alt="image-20210815201323329" style="zoom:80%;" />

> `setTimeout(function,毫秒值)` : 在一定的时间间隔后执行一个function，只执行一次
> `setInterval(function,毫秒值)` :在一定的时间间隔后执行一个function，循环执行

**confirm代码演示：**

```js
// confirm()，点击确定按钮，返回true，点击取消按钮，返回false
var flag = confirm("确认删除？");

alert(flag);
```

下图是 `confirm()` 函数的效果。当我们点击 `确定` 按钮，`flag` 变量值记录的就是 `true` ；当我们点击 `取消` 按钮，`flag` 变量值记录的就是 `false`。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815201600493.png" alt="image-20210815201600493" style="zoom:80%;" />

而以后我们在页面删除数据时候如下图每一条数据后都有 `删除` 按钮，有可能是用户的一些误操作，所以对于删除操作需要用户进行再次确认，此时就需要用到 `confirm()` 函数。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815202406490.png" alt="image-20210815202406490" style="zoom:70%;" />

**定时器代码演示：**

```js
setTimeout(function (){
    alert("hehe");
},3000);
```

当我们打开浏览器，3秒后才会弹框输出 `hehe`，并且只会弹出一次。

```js
setInterval(function (){
    alert("hehe");
},2000);
```

当我们打开浏览器，每隔2秒都会弹框输出 `hehe`。

#### 案例

**需求：每隔1秒，灯泡切换一次状态**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815203345262.png" alt="image-20210815203345262" style="zoom:70%;" />

需求说明：

有如下页面效果，实现定时进行开灯、关灯功能

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815203623739.png" alt="image-20210815203623739" style="zoom:80%;" />

初始页面环境

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JavaScript演示</title>
</head>
<body>

<input type="button" onclick="on()" value="开灯">
<img id="myImage" border="0" src="../imgs/off.gif" style="text-align:center;">
<input type="button" onclick="off()" value="关灯">

<script>
    function on(){
        document.getElementById('myImage').src='../imgs/on.gif';
    }

    function off(){
        document.getElementById('myImage').src='../imgs/off.gif'
    }

</script>
</body>
</html>
```

代码实现：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JavaScript演示</title>
</head>
<body>

<input type="button" onclick="on()" value="开灯">
<img id="myImage" border="0" src="../imgs/off.gif" style="text-align:center;">
<input type="button" onclick="off()" value="关灯">

<script>
    function on(){
        document.getElementById('myImage').src='../imgs/on.gif';
    }

    function off(){
        document.getElementById('myImage').src='../imgs/off.gif'
    }
    
    //定义一个变量，用来记录灯的状态，偶数是开灯状态，奇数是关灯状态
    var x = 0;
    //使用循环定时器
    setInterval(function (){
        if(x % 2 == 0){//表示是偶数，开灯状态，调用 on() 函数
            on();
        }else {  //表示是奇数，关灯状态，调用 off() 函数
            off();
        }
        x ++;//改变变量的值
    },1000);

</script>
</body>
</html>
```

###  History对象

History 对象是 JavaScript 对历史记录进行封装的对象。

* History 对象的获取

  使用 window.history获取，其中window. 可以省略

* History 对象的函数

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815224826535.png" alt="image-20210815224826535" style="zoom:70%;" />

  这两个函数我们平时在访问其他的一些网站时经常使用对应的效果，如下图

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815225059114.png" alt="image-20210815225059114" style="zoom:80%;" />

  当我们点击向左的箭头，就跳转到前一个访问的页面，这就是 `back()` 函数的作用；当我们点击向右的箭头，就跳转到下一个访问的页面，这就是 `forward()` 函数的作用。

### Location对象

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815225243560.png" alt="image-20210815225243560" style="zoom:80%;" />

Location 对象是 JavaScript 对地址栏封装的对象。可以通过操作该对象，跳转到任意页面。

#### 获取Location对象

使用 window.location获取，其中window. 可以省略

```js
window.location.方法();
location.方法();
```



#### Location对象属性

Location对象提供了很对属性。以后常用的只有一个属性 `href`

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815225707580.png" alt="image-20210815225707580" style="zoom:80%;" />

**代码演示：**

```js
alert("要跳转了");
location.href = "https://www.baidu.com";
```

在浏览器首先会弹框显示 `要跳转了`，当我们点击了 `确定` 就会跳转到 百度 的首页。



#### 案例

**需求：3秒跳转到百度首页**

**分析：**

1. 3秒跳转，由此可以确定需要使用到定时器，而只跳转一次，所以使用 `setTimeOut()`
2. 要进行页面跳转，所以需要用到 `location` 对象的 `href` 属性实现

**代码实现：**

```js
document.write("3秒跳转到首页..."); 
setTimeout(function (){
    location.href = "https://www.baidu.com"
},3000);
```

## DOM

### 概述

DOM：Document Object Model 文档对象模型。也就是 JavaScript 将 HTML 文档的各个组成部分封装为对象。

DOM 其实我们并不陌生，之前在学习 XML 就接触过，只不过 XML 文档中的标签需要我们写代码解析，而 HTML 文档是浏览器解析。封装的对象分为

* Document：整个文档对象
* Element：元素对象
* Attribute：属性对象
* Text：文本对象
* Comment：注释对象

如下图，左边是 HTML 文档内容，右边是 DOM 树

![image-20210815231028430](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815231028430.png)

**作用：**

JavaScript 通过 DOM， 就能够对 HTML进行操作了

* 改变 HTML 元素的内容
* 改变 HTML 元素的样式（CSS）
* 对 HTML DOM 事件作出反应
* 添加和删除 HTML 元素

**DOM相关概念：**

DOM 是 W3C（万维网联盟）定义了访问 HTML 和 XML 文档的标准。该标准被分为 3 个不同的部分：

1. 核心 DOM：针对任何结构化文档的标准模型。 XML 和 HTML 通用的标准

   * Document：整个文档对象

   * Element：元素对象

   * Attribute：属性对象

   * Text：文本对象

   * Comment：注释对象

2. XML DOM： 针对 XML 文档的标准模型

3. HTML DOM： 针对 HTML 文档的标准模型

   该标准是在核心 DOM 基础上，对 HTML 中的每个标签都封装成了不同的对象

   * 例如：`<img>` 标签在浏览器加载到内存中时会被封装成 `Image` 对象，同时该对象也是 `Element` 对象。
   * 例如：`<input type='button'>` 标签在浏览器加载到内存中时会被封装成 `Button` 对象，同时该对象也是 `Element` 对象。

### 获取 Element对象

HTML 中的 Element 对象可以通过 `Document` 对象获取，而 `Document` 对象是通过 `window` 对象获取。

`Document` 对象中提供了以下获取 `Element` 元素对象的函数

* `getElementById()`：根据id属性值获取，返回单个Element对象
* `getElementsByTagName()`：根据标签名称获取，返回Element对象数组
* `getElementsByName()`：根据name属性值获取，返回Element对象数组
* `getElementsByClassName()`：根据class属性值获取，返回Element对象数组

**代码演示：**

下面有提前准备好的页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <img id="light" src="../imgs/off.gif"> <br>

    <div class="cls">传智教育</div>   <br>
    <div class="cls">黑马程序员</div> <br>

    <input type="checkbox" name="hobby"> 电影
    <input type="checkbox" name="hobby"> 旅游
    <input type="checkbox" name="hobby"> 游戏
    <br>
    <script>
		//在此处书写js代码
    </script>
</body>
</html>
```

1. 根据 `id` 属性值获取上面的 `img` 元素对象，返回单个对象

   ```js
   var img = document.getElementById("light");
   alert(img);
   ```

   结果如下：

   <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210815233232924.png" alt="image-20210815233232924" style="zoom:80%;" />

   从弹框输出的内容，也可以看出是一个图片元素对象。

2. 根据标签名称获取所有的 `div` 元素对象

   ```js
   var divs = document.getElementsByTagName("div");// 返回一个数组，数组中存储的是 div 元素对象
   // alert(divs.length);  //输出 数组的长度
   //遍历数组
   for (let i = 0; i < divs.length; i++) {
       alert(divs[i]);
   }
   ```

3. 获取所有的满足 `name = 'hobby'` 条件的元素对象

   ```js
   //3. getElementsByName：根据name属性值获取，返回Element对象数组
   var hobbys = document.getElementsByName("hobby");
   for (let i = 0; i < hobbys.length; i++) {
       alert(hobbys[i]);
   }
   ```

4. 获取所有的满足 `class='cls'` 条件的元素对象

   ```js
   //4. getElementsByClassName：根据class属性值获取，返回Element对象数组
   var clss = document.getElementsByClassName("cls");
   for (let i = 0; i < clss.length; i++) {
       alert(clss[i]);
   }
   ```

### HTML Element对象使用

HTML 中的 `Element` 元素对象有很多，不可能全部记住，以后是根据具体的需求查阅文档使用。

下面我们通过具体的案例给大家演示文档的查询和对象的使用；下面提前给大家准备好的页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <img id="light" src="../imgs/off.gif"> <br>

    <div class="cls">传智教育</div>   <br>
    <div class="cls">黑马程序员</div> <br>

    <input type="checkbox" name="hobby"> 电影
    <input type="checkbox" name="hobby"> 旅游
    <input type="checkbox" name="hobby"> 游戏
    <br>
    <script>
        //在此处写js低吗
    </script>
</body>
</html>
```

**需求：**

1. 点亮灯泡

   此案例由于需要改变 `img` 标签 的图片，所以我们查询文档，下图是查看文档的流程：

   <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E6%9F%A5%E7%9C%8B%E6%96%87%E6%A1%A3.png" alt="image-20210815233232924" style="zoom:100%;" />

   代码实现：

   ```js
   //1，根据 id='light' 获取 img 元素对象
   var img = document.getElementById("light");
   //2，修改 img 对象的 src 属性来改变图片
   img.src = "../imgs/on.gif";
   ```

2. 将所有的 `div` 标签的标签体内容替换为 `呵呵`

   ```js
   //1，获取所有的 div 元素对象
   var divs = document.getElementsByTagName("div");
   /*
           style:设置元素css样式
           innerHTML：设置元素内容
       */
   //2，遍历数组，获取到每一个 div 元素对象，并修改元素内容
   for (let i = 0; i < divs.length; i++) {
       //divs[i].style.color = 'red';
       divs[i].innerHTML = "呵呵";
   }
   ```

3. 使所有的复选框呈现被选中的状态

   此案例我们需要看 复选框 元素对象有什么属性或者函数是来操作 复选框的选中状态。下图是文档的查看

   ![image-20210816000520457](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816000520457.png)

   代码实现：

   ```js
   //1，获取所有的 复选框 元素对象
   var hobbys = document.getElementsByName("hobby");
   //2，遍历数组，通过将 复选框 元素对象的 checked 属性值设置为 true 来改变复选框的选中状态
   for (let i = 0; i < hobbys.length; i++) {
       hobbys[i].checked = true;
   }
   ```

## 事件监听

要想知道什么是事件监听，首先先聊聊什么是事件？

HTML 事件是发生在 HTML 元素上的“事情”。比如：页面上的 `按钮被点击`、`鼠标移动到元素之上`、`按下键盘按键` 等都是事件。

事件监听是JavaScript 可以在事件被侦测到时==执行一段逻辑代码。==例如下图当我们点击 `开灯` 按钮，就需要通过 js 代码实现替换图片

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816194143246.png" alt="image-20210816194143246" style="zoom:80%;" />

再比如下图输入框，当我们输入了用户名 `光标离开` 输入框，就需要通过 js 代码对输入的内容进行校验，没通过校验就在输入框后提示 `用户名格式有误!`

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816194333252.png" alt="image-20210816194333252" style="zoom:90%;" />

### 事件绑定

JavaScript 提供了两种事件绑定方式：

* 方式一：通过 HTML标签中的事件属性进行绑定

  如下面代码，有一个按钮元素，我们是在该标签上定义 `事件属性`，在事件属性中绑定函数。`onclick` 就是 `单击事件` 的事件属性。`onclick='on（）'` 表示该点击事件绑定了一个名为 `on()` 的函数

  ```html
  <input type="button" onclick='on()’>
  ```

  下面是点击事件绑定的 `on()` 函数

  ```js
  function on(){
  	alert("我被点了");
  }
  ```

* 方式二：通过 DOM 元素属性绑定

  如下面代码是按钮标签，在该标签上我们并没有使用 `事件属性`，绑定事件的操作需要在 js 代码中实现

  ```html
  <input type="button" id="btn">
  ```

  下面 js 代码是获取了 `id='btn'` 的元素对象，然后将 `onclick` 作为该对象的属性，并且绑定匿名函数。该函数是在事件触发后自动执行

  ```js
  document.getElementById("btn").onclick = function (){
      alert("我被点了");
  }
  ```

**代码演示：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <!--方式1：在下面input标签上添加 onclick 属性，并绑定 on() 函数-->
    <input type="button" value="点我" onclick="on()"> <br>
    <input type="button" value="再点我" id="btn">

    <script>
        function on(){
            alert("我被点了");
        }
      	//方式2：获取 id="btn" 元素对象，通过调用 onclick 属性 绑定点击事件
        document.getElementById("btn").onclick = function (){
            alert("我被点了");
        }
    </script>
</body>
</html>
```

### 常见事件

上面案例中使用到了 `onclick` 事件属性，那都有哪些事件属性供我们使用呢？下面就给大家列举一些比较常用的事件属性

| 事件属性名  | 说明                     |
| ----------- | ------------------------ |
| onclick     | 鼠标单击事件             |
| onblur      | 元素失去焦点             |
| onfocus     | 元素获得焦点             |
| onload      | 某个页面或图像被完成加载 |
| onsubmit    | 当表单提交时触发该事件   |
| onmouseover | 鼠标被移到某元素之上     |
| onmouseout  | 鼠标从某元素移开         |

* `onfocus` 获得焦点事件。

  如下图，当点击了输入框后，输入框就获得了焦点。而下图示例是当获取焦点后会更改输入框的背景颜色。

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816214900928.png" alt="image-20210816214900928" style="zoom:80%;" />

* `onblur  ` 失去焦点事件。

  如下图，当点击了输入框后，输入框就获得了焦点；再点击页面其他位置，那输入框就失去焦点了。下图示例是将输入的文本转换为大写。

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816215235969.png" alt="image-20210816215235969" style="zoom:80%;" />

* `onmouseout  ` 鼠标移出事件。

* `onmouseover  `  鼠标移入事件。

  如下图，当鼠标移入到 苹果 图片上时，苹果图片变大；当鼠标移出 苹果图片时，苹果图片变小。

  <img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816220149093.png" alt="image-20210816220149093" style="zoom:70%;" />

* `onsubmit  ` 表单提交事件

  如下是带有表单的页面

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
      <form id="register" action="#" >
          <input type="text" name="username" />
          <input type="submit" value="提交">
      </form>
      <script>
          
      </script>
  </body>
  </html>
  ```

  如上代码的表单，当我们点击 `提交` 按钮后，表单就会提交，此处默认使用的是 `GET` 提交方式，会将提交的数据拼接到 URL 后。现需要通过 js 代码实现阻止表单提交的功能，js 代码实现如下：

  1. 获取 `form` 表单元素对象。
  2. 给 `form` 表单元素对象绑定 `onsubmit` 事件，并绑定匿名函数。
  3. 该匿名函数如果返回的是true，提交表单；如果返回的是false，阻止表单提交。

  ```js
  document.getElementById("register").onsubmit = function (){
      //onsubmit 返回true，则表单会被提交，返回false，则表单不提交
      return true;
  }
  ```



## JavaScript综合案例

### 案例效果介绍

- 在“姓名、年龄、性别”三个文本框中填写信息后，添加到“学生信息表”列表（表格）中。

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E7%BB%BC%E5%90%88%E6%A1%88%E4%BE%8B-%E6%B7%BB%E5%8A%A0%E5%8A%9F%E8%83%BD%E5%88%86%E6%9E%90.png)

### 添加功能的分析

1. 为添加按钮绑定单击事件。
2. 创建 tr 元素。
3. 创建 4 个 td 元素。
4. 将 td 添加到 tr 中。
5. 获取文本框输入的信息。
6. 创建 3 个文本元素。
7. 将文本元素添加到对应的 td 中。
8. 创建 a 元素。
9. 将 a 元素添加到对应的 td 中。
10. 将 tr 添加到 table 中。

### 添加功能的实现

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>动态表格</title>

    <style>
        table{
            border: 1px solid;
            margin: auto;
            width: 500px;
        }

        td,th{
            text-align: center;
            border: 1px solid;
        }
        div{
            text-align: center;
            margin: 50px;
        }
    </style>

</head>
<body>

<div>
    <input type="text" id="name" placeholder="请输入姓名" autocomplete="off">
    <input type="text" id="age"  placeholder="请输入年龄" autocomplete="off">
    <input type="text" id="gender"  placeholder="请输入性别" autocomplete="off">
    <input type="button" value="添加" id="add">
</div>

    <table id="tb">
        <caption>学生信息表</caption>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>性别</th>
            <th>操作</th>
        </tr>

        <tr>
            <td>张三</td>
            <td>23</td>
            <td>男</td>
            <td><a href="JavaScript:void(0);" onclick="drop(this)">删除</a></td>
        </tr>

        <tr>
            <td>李四</td>
            <td>24</td>
            <td>男</td>
            <td><a href="JavaScript:void(0);" onclick="drop(this)">删除</a></td>
        </tr>

    </table>

</body>
<script>
    //一、添加功能
    //1.为添加按钮绑定单击事件
    document.getElementById("add").onclick = function(){
        //2.创建行元素
        let tr = document.createElement("tr");
        //3.创建4个单元格元素
        let nameTd = document.createElement("td");
        let ageTd = document.createElement("td");
        let genderTd = document.createElement("td");
        let deleteTd = document.createElement("td");
        //4.将td添加到tr中
        tr.appendChild(nameTd);
        tr.appendChild(ageTd);
        tr.appendChild(genderTd);
        tr.appendChild(deleteTd);
        //5.获取输入框的文本信息
        let name = document.getElementById("name").value;
        let age = document.getElementById("age").value;
        let gender = document.getElementById("gender").value;
        //6.根据获取到的信息创建3个文本元素
        let nameText = document.createTextNode(name);
        let ageText = document.createTextNode(age);
        let genderText = document.createTextNode(gender);
        //7.将3个文本元素添加到td中
        nameTd.appendChild(nameText);
        ageTd.appendChild(ageText);
        genderTd.appendChild(genderText);
        //8.创建超链接元素和显示的文本以及添加href属性
        let a = document.createElement("a");
        let aText = document.createTextNode("删除");
        a.setAttribute("href","JavaScript:void(0);");
        a.setAttribute("onclick","drop(this)");
        a.appendChild(aText);
        //9.将超链接元素添加到td中
        deleteTd.appendChild(a);
        //10.获取table元素，将tr添加到table中
        let table = document.getElementById("tb");
        table.appendChild(tr);
    }
</script>
</html>
```

### 删除功能的分析

- **删除功能介绍**

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E7%BB%BC%E5%90%88%E6%A1%88%E4%BE%8B-%E5%88%A0%E9%99%A4%E5%8A%9F%E8%83%BD%E5%88%86%E6%9E%90.png)

- **删除功能分析**

1. 为每个删除超链接添加单击事件属性。
2. 定义删除的方法。
3. 获取 table 元素。
4. 获取 tr 元素。
5. 通过 table 删除 tr。

### 删除功能的实现

```js
//二、删除的功能
//1.为每个删除超链接标签添加单击事件的属性
//2.定义删除的方法
function drop(obj){
//3.获取table元素
let table = obj.parentElement.parentElement.parentElement;
//4.获取tr元素
let tr = obj.parentElement.parentElement;
//5.通过table删除tr
table.removeChild(tr);
}
```







## 表单验证案例

### 需求

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816225925955.png" alt="image-20210816225925955" style="zoom:60%;" />

有如下注册页面，对表单进行校验，如果输入的用户名、密码、手机号符合规则，则允许提交；如果不符合规则，则不允许提交。

完成以下需求：

1. 当输入框失去焦点时，验证输入内容是否符合要求

2. 当点击注册按钮时，判断所有输入框的内容是否都符合要求，如果不合符则阻止表单提交

### 环境准备

下面是初始页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>欢迎注册</title>
    <link href="../css/register.css" rel="stylesheet">
</head>
<body>
    <div class="form-div">
        <div class="reg-content">
            <h1>欢迎注册</h1>
            <span>已有帐号？</span> <a href="#">登录</a>
        </div>
        <form id="reg-form" action="#" method="get">
            <table>
                <tr>
                    <td>用户名</td>
                    <td class="inputs">
                        <input name="username" type="text" id="username">
                        <br>
                        <span id="username_err" class="err_msg" style="display: none">用户名不太受欢迎</span>
                    </td>
                </tr>

                <tr>
                    <td>密码</td>
                    <td class="inputs">
                        <input name="password" type="password" id="password">
                        <br>
                        <span id="password_err" class="err_msg" style="display: none">密码格式有误</span>
                    </td>
                </tr>

                <tr>
                    <td>手机号</td>
                    <td class="inputs"><input name="tel" type="text" id="tel">
                        <br>
                        <span id="tel_err" class="err_msg" style="display: none">手机号格式有误</span>
                    </td>
                </tr>
            </table>
            <div class="buttons">
                <input value="注 册" type="submit" id="reg_btn">
            </div>
            <br class="clear">
        </form>

    </div>


    <script>

    </script>
</body>
</html>
```

### 验证输入框

此小节完成如下功能：

* 校验用户名。当用户名输入框失去焦点时，判断输入的内容是否符合 `长度是 6-12 位` 规则，不符合使 `id='username_err'` 的span标签显示出来，给出用户提示。
* 校验密码。当密码输入框失去焦点时，判断输入的内容是否符合 `长度是 6-12 位` 规则，不符合使 `id='password_err'` 的span标签显示出来，给出用户提示。
* 校验手机号。当手机号输入框失去焦点时，判断输入的内容是否符合 `长度是 11 位` 规则，不符合使 `id='tel_err'` 的span标签显示出来，给出用户提示。

代码如下：

```js
//1. 验证用户名是否符合规则
//1.1 获取用户名的输入框
var usernameInput = document.getElementById("username");

//1.2 绑定onblur事件 失去焦点
usernameInput.onblur = function () {
    //1.3 获取用户输入的用户名
    var username = usernameInput.value.trim();

    //1.4 判断用户名是否符合规则：长度 6~12
    if (username.length >= 6 && username.length <= 12) {
        //符合规则
        document.getElementById("username_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("username_err").style.display = '';
    }
}

//1. 验证密码是否符合规则
//1.1 获取密码的输入框
var passwordInput = document.getElementById("password");

//1.2 绑定onblur事件 失去焦点
passwordInput.onblur = function() {
    //1.3 获取用户输入的密码
    var password = passwordInput.value.trim();

    //1.4 判断密码是否符合规则：长度 6~12
    if (password.length >= 6 && password.length <= 12) {
        //符合规则
        document.getElementById("password_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("password_err").style.display = '';
    }
}

//1. 验证手机号是否符合规则
//1.1 获取手机号的输入框
var telInput = document.getElementById("tel");

//1.2 绑定onblur事件 失去焦点
telInput.onblur = function() {
    //1.3 获取用户输入的手机号
    var tel = telInput.value.trim();

    //1.4 判断手机号是否符合规则：长度 11
    if (tel.length == 11) {
        //符合规则
        document.getElementById("tel_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("tel_err").style.display = '';
    }
}
```

### 验证表单

当用户点击 `注册` 按钮时，需要同时对输入的 `用户名`、`密码`、`手机号` ，如果都符合规则，则提交表单；如果有一个不符合规则，则不允许提交表单。实现该功能需要获取表单元素对象，并绑定 `onsubmit` 事件

```js
//1. 获取表单对象
var regForm = document.getElementById("reg-form");

//2. 绑定onsubmit 事件
regForm.onsubmit = function () {
    
}
```

`onsubmit` 事件绑定的函数需要对输入的 `用户名`、`密码`、`手机号` 进行校验，这些校验我们之前都已经实现过了，这里我们还需要再校验一次吗？不需要，只需要对之前校验的代码进行改造，把每个校验的代码专门抽象到有名字的函数中，方便调用；并且每个函数都要返回结果来去决定是提交表单还是阻止表单提交，代码如下：

```js
//1. 验证用户名是否符合规则
//1.1 获取用户名的输入框
var usernameInput = document.getElementById("username");

//1.2 绑定onblur事件 失去焦点
usernameInput.onblur = checkUsername;

function checkUsername() {
    //1.3 获取用户输入的用户名
    var username = usernameInput.value.trim();

    //1.4 判断用户名是否符合规则：长度 6~12
    var flag = username.length >= 6 && username.length <= 12;
    if (flag) {
        //符合规则
        document.getElementById("username_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("username_err").style.display = '';
    }
    return flag;
}

//1. 验证密码是否符合规则
//1.1 获取密码的输入框
var passwordInput = document.getElementById("password");

//1.2 绑定onblur事件 失去焦点
passwordInput.onblur = checkPassword;

function checkPassword() {
    //1.3 获取用户输入的密码
    var password = passwordInput.value.trim();

    //1.4 判断密码是否符合规则：长度 6~12
    var flag = password.length >= 6 && password.length <= 12;
    if (flag) {
        //符合规则
        document.getElementById("password_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("password_err").style.display = '';
    }
    return flag;
}

//1. 验证手机号是否符合规则
//1.1 获取手机号的输入框
var telInput = document.getElementById("tel");

//1.2 绑定onblur事件 失去焦点
telInput.onblur = checkTel;

function checkTel() {
    //1.3 获取用户输入的手机号
    var tel = telInput.value.trim();

    //1.4 判断手机号是否符合规则：长度 11
    var flag = tel.length == 11;
    if (flag) {
        //符合规则
        document.getElementById("tel_err").style.display = 'none';
    } else {
        //不合符规则
        document.getElementById("tel_err").style.display = '';
    }
    return flag;
}
```

而 `onsubmit` 绑定的函数需要调用 `checkUsername()` 函数、`checkPassword()` 函数、`checkTel()` 函数。

```js
//1. 获取表单对象
var regForm = document.getElementById("reg-form");

//2. 绑定onsubmit 事件
regForm.onsubmit = function () {
    //挨个判断每一个表单项是否都符合要求，如果有一个不合符，则返回false

    var flag = checkUsername() && checkPassword() && checkTel();

    return flag;
}
```

## RegExp对象

RegExp 是正则对象。正则对象是判断指定字符串是否符合规则。

如下图是百度贴吧中的帖子

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210816235112754.png" alt="image-20210816235112754" style="zoom:70%;" />

我们可以通过爬虫技术去爬取该页面源代码，然后获取页面中所有的邮箱，后期我们可以给这些邮箱地址发送推广的邮件。那么问题来了，如何才能知道页面内容中哪些事邮箱地址呢？这里就可以使用正则表达式来匹配邮箱。

在 js 中对正则表达式封装的对象就是正则对象。

### 正则对象使用

#### 创建对象

正则对象有两种创建方式：

* 直接量方式：注意不要加引号

  ```js
  var reg = /正则表达式/;
  ```

* 创建 RegExp 对象

  ```js
  var reg = new RegExp("正则表达式");
  ```

#### 函数

`test(str)` ：判断指定字符串是否符合规则，返回 true或 false

### 正则表达式

从上面创建正则对象的格式中可以看出不管哪种方式都需要正则表达式，那么什么是正则表达式呢？

正则表达式定义了字符串组成的规则。也就是判断指定的字符串是否符合指定的规则，如果符合返回true，如果不符合返回false。

正则表达式是和语言无关的。很多语言都支持正则表达式，Java语言也支持，只不过正则表达式在不同的语言中的使用方式不同，js 中需要使用正则对象来使用正则表达式。

正则表达式常用的规则如下：

* ^：表示开始

* $：表示结束

* [ ]：代表某个范围内的单个字符，比如： [0-9] 单个数字字符

* .：代表任意单个字符，除了换行和行结束符

* \w：代表单词字符：字母、数字、下划线(_)，相当于 [A-Za-z0-9_]

* \d：代表数字字符： 相当于 [0-9]

量词：

* +：至少一个

* *：零个或多个

* ？：零个或一个

* {x}：x个

* {m,}：至少m个

* {m,n}：至少m个，最多n个

**代码演示：**

```js
// 规则：单词字符，6~12
//1,创建正则对象，对正则表达式进行封装
var reg = /^\w{6,12}$/;

var str = "abcccc";
//2,判断 str 字符串是否符合 reg 封装的正则表达式的规则
var flag = reg.test(str);
alert(flag);
```

### 改进表单校验案例

表单校验案例中的规则是我们进行一系列的判断来实现的，现在学习了正则对象后，就可以使用正则对象来改进这个案例。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>欢迎注册</title>
    <link href="../css/register.css" rel="stylesheet">
</head>
<body>

<div class="form-div">
    <div class="reg-content">
        <h1>欢迎注册</h1>
        <span>已有帐号？</span> <a href="#">登录</a>
    </div>
    <form id="reg-form" action="#" method="get">

        <table>

            <tr>
                <td>用户名</td>
                <td class="inputs">
                    <input name="username" type="text" id="username">
                    <br>
                    <span id="username_err" class="err_msg" style="display: none">用户名不太受欢迎</span>
                </td>

            </tr>

            <tr>
                <td>密码</td>
                <td class="inputs">
                    <input name="password" type="password" id="password">
                    <br>
                    <span id="password_err" class="err_msg" style="display: none">密码格式有误</span>
                </td>
            </tr>


            <tr>
                <td>手机号</td>
                <td class="inputs"><input name="tel" type="text" id="tel">
                    <br>
                    <span id="tel_err" class="err_msg" style="display: none">手机号格式有误</span>
                </td>
            </tr>

        </table>

        <div class="buttons">
            <input value="注 册" type="submit" id="reg_btn">
        </div>
        <br class="clear">
    </form>

</div>


<script>

    //1. 验证用户名是否符合规则
    //1.1 获取用户名的输入框
    var usernameInput = document.getElementById("username");

    //1.2 绑定onblur事件 失去焦点
    usernameInput.onblur = checkUsername;

    function checkUsername() {
        //1.3 获取用户输入的用户名
        var username = usernameInput.value.trim();

        //1.4 判断用户名是否符合规则：长度 6~12,单词字符组成
        var reg = /^\w{6,12}$/;
        var flag = reg.test(username);

        //var flag = username.length >= 6 && username.length <= 12;
        if (flag) {
            //符合规则
            document.getElementById("username_err").style.display = 'none';
        } else {
            //不合符规则
            document.getElementById("username_err").style.display = '';
        }
        return flag;
    }

    //1. 验证密码是否符合规则
    //1.1 获取密码的输入框
    var passwordInput = document.getElementById("password");

    //1.2 绑定onblur事件 失去焦点
    passwordInput.onblur = checkPassword;

    function checkPassword() {
        //1.3 获取用户输入的密码
        var password = passwordInput.value.trim();

        //1.4 判断密码是否符合规则：长度 6~12
        var reg = /^\w{6,12}$/;
        var flag = reg.test(password);

        //var flag = password.length >= 6 && password.length <= 12;
        if (flag) {
            //符合规则
            document.getElementById("password_err").style.display = 'none';
        } else {
            //不合符规则
            document.getElementById("password_err").style.display = '';
        }
        return flag;
    }

    //1. 验证手机号是否符合规则
    //1.1 获取手机号的输入框
    var telInput = document.getElementById("tel");

    //1.2 绑定onblur事件 失去焦点
    telInput.onblur = checkTel;

    function checkTel() {
        //1.3 获取用户输入的手机号
        var tel = telInput.value.trim();

        //1.4 判断手机号是否符合规则：长度 11，数字组成，第一位是1
        //var flag = tel.length == 11;
        var reg = /^[1]\d{10}$/;
        var flag = reg.test(tel);
        if (flag) {
            //符合规则
            document.getElementById("tel_err").style.display = 'none';
        } else {
            //不合符规则
            document.getElementById("tel_err").style.display = '';
        
        return flag;
    }

    //1. 获取表单对象
    var regForm = document.getElementById("reg-form");

    //2. 绑定onsubmit 事件
    regForm.onsubmit = function () {
        //挨个判断每一个表单项是否都符合要求，如果有一个不合符，则返回false

        var flag = checkUsername() && checkPassword() && checkTel();

        return flag;
    }
</script>
</body>
</html>
```

