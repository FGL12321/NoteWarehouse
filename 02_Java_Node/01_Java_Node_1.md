[TOC]



# Java学习笔记

## Java简介

 Java是全球排名第一的编程语言，Java工程师也是市场需求最大的软件工程师，选择Java，就是选择了高薪 

### Java的诞生

 ![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1c950a7b02087bf40fe8767ff8d3572c11dfcf1b) 

​		Java最早是由SUN公司（已被Oracle收购）的[詹姆斯·高斯林](https://en.wikipedia.org/wiki/James_Gosling)（高司令，人称Java之父）在上个世纪90年代初开发的一种编程语言，最初被命名为Oak，目标是针对小型家电设备的嵌入式应用，结果市场没啥反响。谁料到互联网的崛起，让Oak重新焕发了生机，于是SUN公司改造了Oak，在1995年以Java的名称正式发布，原因是Oak已经被人注册了，因此SUN注册了Java这个商标。随着互联网的高速发展，Java逐渐成为最重要的网络编程语言。

​		Java介于编译型语言和解释型语言之间。编译型语言如C、C++，代码是直接编译成机器码执行，但是不同的平台（x86、ARM等）CPU的指令集不同，因此，需要编译出每一种平台的对应机器码。解释型语言如Python、Ruby没有这个问题，可以由解释器直接加载源码然后运行，代价是运行效率太低。而Java是将代码编译成一种“字节码”，它类似于抽象的CPU指令，然后，针对不同平台编写虚拟机，不同平台的虚拟机负责加载字节码并执行，这样就实现了“一次编写，到处运行”的效果。当然，这是针对Java开发者而言。对于虚拟机，需要为每个平台分别开发。为了保证不同平台、不同公司开发的虚拟机都能正确执行Java字节码，SUN公司制定了一系列的Java虚拟机规范。从实践的角度看，JVM的兼容性做得非常好，低版本的Java字节码完全可以正常运行在高版本的JVM上。

**随着Java的发展，SUN给Java又分出了三个不同版本：**

- Java SE：Standard Edition
- Java EE：Enterprise Edition
- Java ME：Micro Edition

这三者之间有啥关系呢？

```ascii
┌───────────────────────────┐
│Java EE                    │
│    ┌────────────────────┐ │
│    │Java SE             │ │
│    │    ┌─────────────┐ │ │
│    │    │   Java ME   │ │ │
│    │    └─────────────┘ │ │
│    └────────────────────┘ │
└───────────────────────────┘
```

​		简单来说，Java SE就是标准版，包含标准的JVM和标准库，而Java EE是企业版，它只是在Java SE的基础上加上了大量的API和库，以便方便开发Web应用、数据库、消息服务等，Java EE的应用使用的虚拟机和Java SE完全相同。

​		Java ME就和Java SE不同，它是一个针对嵌入式设备的“瘦身版”，Java SE的标准库无法在Java ME上使用，Java ME的虚拟机也是“瘦身版”。

​		毫无疑问，Java SE是整个Java平台的核心，而Java EE是进一步学习Web应用所必须的。我们熟悉的Spring等框架都是Java EE开源生态系统的一部分。不幸的是，Java ME从来没有真正流行起来，反而是Android开发成为了移动平台的标准之一，因此，没有特殊需求，不建议学习Java ME。

**因此我们推荐的Java学习路线图如下：**

​	首先要学习Java SE，掌握Java语言本身、Java核心开发技术以及Java标准库的使用；

​	如果继续学习Java EE，那么Spring框架、数据库开发、分布式架构就是需要学习的；

​	如果要学习大数据开发，那么Hadoop、Spark、Fl`ink这`些大数据平台就是需要学习的，他们都基于Java或Scala开发；



## JVM

### **Java的两种核心机制**

**Java虚拟机：**是一个虚拟的计算机，具有指令集并使用不同的存储机制。负责执行指令，管理数据，内存，寄存器

对于不同的平台有不同的虚拟机

只有某平台提供对应的虚拟机，Java程序才能在此平台上 运行

Java虚拟机机制屏蔽了底层运行平台的差别，实现了，一次编译导出运行

![1640525412038](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640525412038.png)

![1640525437653](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640525437653.png)

**垃圾收集机制：**Java语言**消除了程序员回收内存空间的责任**：它提供一种系统级线程跟踪存储空间的分配情况，并在JVM空闲时，检查并释放那些可被释放的存储空间。

**垃圾回收在Java程序运行过程中自动进行，程序员无法精确控制和干预**

Java程序还会出现内存泄漏和溢出吗？  答案：yes

### 对JVM虚拟机的准确介绍

![1640617540065](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640617540065.png)

**堆：**此方法区域的唯一目的就是**存放对象实例**，几乎所有的对象实例都在**这里分配内存。**

**栈：**虚拟机栈用来存储局部变量。**局部变量表存放了编译期可知长度的各种基本数据类型**，对象引用，**它不等同于对象本身**，是对象在**堆内存放的首地址**。方法执行完**自动释放** 。

**方法区**：用于存储已被虚拟机加载的**类信息，常量，静态变量，**即时编译器编译后的代码等数据。

**程序计数器：** 内存空间小，线程私有。字节码解释器工作是就是通过改变这个计数器的值来选取下一条需要执行指令的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖计数器完成 。



## JDK  JRE  JVM的关系

JDK：Java Develoment Kit  Java开发工具包———>包含了JRE

JRE：包括了Java虚拟机和Java程序所需的核心类库 ，如果想运行一个开发好的Java程序，计算机中只需安装JRE

![1640526306164](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640526306164.png)



**API：Application Programming Interface应用程序接口**

**Java程序的运行过程：**

![1640527219845](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640527219845.png)

# 1.0 Java程序基础

## 1.1变量与数据类型

![1640315114656](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640315114656.png)

**基本数据类型占用的字节数**

![1640315190507](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640315190507.png)



## 1.2运算符

**1.2.1算数运算**

![1640529971314](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640529971314.png)

**1.2.2赋值数运算**

![1640530007464](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530007464.png)

**1.2.3比较运算**

![1640530026985](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530026985.png)

**1.2.4逻辑运算**

![1640530050905](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530050905.png)

**1.2.5位运算符**

![1640530088872](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530088872.png)

## 1.3数组（Array）

### 1.3.1概念

 数组是多个相同类型数据按一定顺序排列的集合，并使用一个名字命名，并通过编号的方式对这些数据进行同意管理。

### 1.3.2数组的概述

- 数组是**引用数据类型**
- 创建数组对象会在内存中开辟一整块连续的内存空间，而数组名引用的是这块连续空间的**首地址**。
- 数组的长度**一旦确定就不能修改**
- 我们可以通过数组的下标或者索引的方式调用指定位置的元素

**数组可以分为**  

一维数组 二维数组 三维数组......

基本数据类型数组 引用数据类型数组（对象数组）

### 1.3.3一维数组的使用

**（1）声明**

![1640574491019](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640574491019.png)

**（2）初始化**

![1640574574594](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640574574594.png)

**（3）数组元素的引用**

![1640574633344](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640574633344.png)

（4）初始化

数组是**引用类型**，相当于**类的成员变量**，数组一经分配空间其中每个人元素也被按照成员变量同样的方式**被隐式初始化。**

![1640574797312](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640574797312.png)

**（5）创建基本数据类型数组**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640574998760.png" alt="1640574998760" style="zoom:80%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640575278788.png" alt="1640575278788" style="zoom:80%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640575337284.png" alt="1640575337284" style="zoom:80%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640575415171.png" alt="1640575415171" style="zoom:80%;" />

**（6）初识内存的简化结构**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640575606104.png" alt="1640575606104" style="zoom:80%;" />



### 1.3.4多维数组的使用

**1.3.4.1声明**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587172123.png" alt="1640587172123" style="zoom:67%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587183879.png" alt="1640587183879" style="zoom: 80%;" />

![1640587239809](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587239809.png)

**1.3.4.2二维数组的内存解析**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587312203.png" alt="1640587312203" style="zoom:67%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587395802.png" alt="1640587395802" style="zoom:67%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587414153.png" alt="1640587414153" style="zoom:67%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640587430924.png" alt="1640587430924" style="zoom:67%;" />

### 1.3.5数组中的排序算法

------

**内部排序**

整个排序过程不需要借助外部存储器，所有排序操作**都在内存中完成**。

------

**外部排序**

参与排序的数据非常多，数据量非常大，，计算机无法吧整个排序过程存放在内存中完成。**必须借助于外部存储器**。外部排序最**常见的是多路归并排序**。可以认为外部排序是由**多次内部排序组成**。

------

**十大排序算法**

**（1）冒泡排序**

**（2）快速排序**

![1640772890489](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640772890489.png)

**（3）算法性能排序**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640588665399.png" alt="1640588665399" style="zoom:67%;" />

**（4）内部排序方法性能比较**

**平均时间而言**

​       快速排序最佳，但在最坏的情况下时间性能不如堆排序和归并排序。

**算法简单性来看**

​      **简单算法：**直接选择排序  直接插入排序  冒泡排序

​      **复杂排序：** Shell排序 堆排序 快速排序 归并排序

**稳定性看**

​       **稳定的：**直接插入排序，冒泡排序 归并排序 

​       **不稳定排序：**直接选择排序 快婿排序 Shell排序  堆排序

**待排序的记录数n的大小看**

​      n较小时，宜采用简单排序；n较大时，宜采用改进排序

**（5排序算法的选择**

(1)

(2)

(3)

   **(6)Arrays工具类的使用**

![1640589585821](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640589585821.png)

## 1.4关键字&保留字&标识符

### 1.4.1关键字

**(1)对关键字的介绍**

**被Java语言赋予了特殊含义，用作专门用途的字符串。**

**特点：关键字中的所有字母都为小写**

**![1640528874258](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640528874258.png)**

------

#### 1.4.1.1 this

- **this的使用介绍：**

- 它在方法内部使用，即这个**方法所属对象的引用**。

- 它在构造器内部使用，表示该**构造器正在初始化的对象**。

  

- **this可以调用类的属性，方法和构造器**

- 当在方法内需要用到该方法的对象时，就用this

- 具体的：我们可以用this来区分属性和局部变量。

```java
//使用this，调用方法，属性
class Person{
     private String name;
     private int age;
     public Person{
         this.name=name;
         this.age=age;
     }
     public void getlnfo(){
           System.out.println("姓名："+name)；
           this.speak;     
     } 
     public void speak(){
           System.out.println("年龄："+this.age);
     }
  
}
```



![1640706116477](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640706116477.png)

![1640706133475](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640706133475.png)

------

#### 1.4.1.2 super

在Java类中使用super来调用父类中的指定操作

```
>super可以用来访问父类中定义的属性

>super可以用于调用父类中定义的成员方法

>super可以同于子类构造器中调用父类的构造器
```

**注意：**

```
>尤其当子父类同名成员时，可以用super表明调用的是父类中的成员

>super的追溯不仅限于直接父类

>super和this的用法相像，this代表本类对象的引用，super代表父类的内存空间的标识
```

**super与this的区别：**

![1640793205700](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640793205700.png)

------

#### 1.4.1.3 static

引入

我们希望有时候无论是否产生了对象，或者产生了多少个对象，某些特定的数据在内存中只有一份。

static 关键字的使用

1.static静态的

2.static可以用来修饰：属性，方法，代码块，内部类

3.使用static修饰属性：静态变量（类变量）

​           属性：按照是否使用static修饰，又分为:静态属性 VS 非静态属性（实例变量）

​           **实例变量**：我们创建了类的多个对象，每个对象，都独立具有一套类中的非静态变量。

​            **静态属性**：我们创建了类的多个对象，当多个对象共享一个静态变量。

  4.static修饰属性的其他说明

​            1.静态变量随着类的加载而加载，可以通过“类.静态变量”的方式进行调用。

​            2.静态变量的加载要早于兑现更多创建。

​            3.由于类只会加载一次，则静态变量会在内存中加载一次。存在方法区的静态域中。

​            4.静态属性举例：System.out;     Math.PI

​            ![1641030904946](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641030904946.png)

5.使用static修饰方法————》静态方法

​                  1.随着类的加载而加载，可以通过  类.静态方法  的方式进行调用

​                  2.静态方法中，只能调用静态的方法和属性。

​                     非静态方法中，既可以调用非静态方法或属性，也可以调用静态方法或属性。

6.static注意点：

​             在静态方法内不能使用this关键字，super关键字       

​             关于静态属性和静态方法的使用，可以从声明周期的角度去理解

7.在开发中，如何确定一个属性是否要声明为static？

​           属性可以被多个对象所共享的，不会随着对象的不同而不同的

   在开发中，如何确定一个方法是否要声明为static？

​           操作静态属性的方法，通常设置为static的

​          工具类中的方法，习惯上声明为static的，比如Math Arrays 

​                

```java
public class Static_1 {
    public static void main(String[] args) {
        Chinese c1 =new Chinese();
        c1.name="马龙";
        c1.age=40;
        c1.nation="CNN";

        Chinese c2=new Chinese();
        c2.name="马龙";
        c2.age=40;
        System.out.println(c2.nation);//输出结果为CNN
    }
}
class Chinese{
    String name;
    int age;
    static String nation;//声明一个静态变量
}
```

------

**<3>关于对final关键字的介绍：**

**final：最终的**

1.final可以用了修饰类，方法，变量。

2.final修饰类，final class A{   }//这个类就不能被继承了。

3.final标记的方法不能被子类重写。

4.final标记的变量称为常量，名称大写 ，只能被赋值一次。



### 1.4.2保留字

**instanceof**

**现有Java版本尚未使用，但以后的版本可能会作为关键字使用。**

**当自己命名这些标识符的时候要避免使用这些保留字**

![1640528913754](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640528913754.png)



**instanceof操作符的使用**

1.对象运算符instanceof来测试一个指定的对象是否是**指定类或它的子类的实**例，是则返回True，不是则返回Fales

- if（p2 instanceof Woman）{  //如果不是则直接输出false
- Woman w=(Wonman)p2;          
- w1.goShopping();
- System.out.println("Woman");//如果是Woman则输出woman
- }

2.为了避免在向下转型时出现异常，在其之前应该进行一次instanceof判断

如果a instanceof A返回True

​        a instanceof B也返回True

则说明：类B是类A的父类

### 1.4.3标识符

**Java对各种变量，方法，类等要素命名时使用的字符序列称为标识符。**

![1640529228816](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640529228816.png)

![1640529241394](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640529241394.png)

------



# 2.0 流程控制

## 2.3顺序结构

程序从上到下逐行地执行，中间没有任何判断和跳转。

## 2.1分支结构

### 2.1.1if-else语句

```java
switch(表达式){
case 常量1:
语句1;
// break;
case 常量2:
语句2;
// break; … …
case 常量N:
语句N;
// break;
default:
语句;
// break;
}
```

### 2.1.2switch-case语句

```java
String season = "summer";
switch (season) {
case "spring":
System.out.println("春暖花开");
break;
case "summer":
System.out.println("夏日炎炎");
break;
case "autumn":
System.out.println("秋高气爽");
break;
case "winter":
System.out.println("冬雪皑皑");
break;
default:
System.out.println("季节输入有误");
break; }
```

## 2.2循环结构

概述

![1640530559824](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530559824.png)



**2.2.1 for循环**

![1640530583977](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530583977.png)

![1640530595934](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530595934.png)

![1640530651135](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530651135.png)

**2.2.2 while循环**

![1640530688543](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530688543.png)

![1640530696985](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530696985.png)

**2.2.3 do-while循环**

![1640530722881](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530722881.png)

![1640530734321](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640530734321.png)

**补充：**



**嵌套循环**

**将一各循环体放在另一个循环体中就是嵌套循环**



**特殊流程控制语句**

**break：终止某个语句块的执行**

**continue：跳过其所在的循环语句的一次执行**

**return：直接结束整个方法**

# 3.0 面向对象编程

## 面向对象概述

![1640615940397](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640615940397.png)

## 3.1类

### 3.1.1概念

**类：是对一类事物的抽象描述，是抽象的，概念上的定义。**

### 3.1.2类的声明

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640616183077.png" alt="1640616183077" style="zoom: 80%;" />

**面向对象程序设计的重点是类的设计**

### 3.1.3属性

**3.1.3.1变量的分类**

**成员变量**:方法体外类体内。

**局部变量**：方法体内部声明的变量。

![1640618100419](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640618100419.png)

**3.1.3.2成员与局部变量的存储位置**

![1640618286362](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640618286362.png)

**3.1.3.3成员变量自动初始化赋值**

![1640618378843](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640618378843.png)

### 3.1.4方法

#### 方法概述

**方法的调用**

![1640619162471](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640619162471.png)

------

**关于main方法的介绍**

1.作为程序的入口

2.main()方法也是一个普通的静态方法

3.main()方法可以作为我们与控制台交互的方式

```
public class MainDemo
{
    public static void main(String[] args) {    //同样也是一个静态方法
       Main.main(new String[100]);
       show();
    }
    public static void show(){
        
    }
}

class Main{  //作为普通的静态方法
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {
            args[i]="print"+i;
            System.out.println(args[i]);

        }
    }
}
```

**1.为什么main()方法是静态的？**

 正因为 main 方法是静态的，JVM 调用这个方法就不需要创建任何包含这个 main 方法的实例 

 如果 main 方法不声明为静态的，JVM 就必须创建 main 类的实例，因为构造器可以被重载，JVM 就没法确定调用哪个 main 方法。 

 静态方法和静态数据加载到内存就可以直接调用而不需要像实例方法一样创建实例后才能调用，如果 main 方法是静态的，那么它就会被加载到 JVM 上下文中成为可执行的方法。 

**2.为什么main方法是公有的？**

 Java 指定了一些可访问的修饰符如：private、protected、public，任何方法或变量都可以声明为 public，Java 
可以从该类之外的地方访问。因为 main 方法是公共的，JVM 就可以轻松的访问执行它。 

**3.为什么main方法没有返回值？**

 因为 main 返回任何值对程序都没任何意义，所以设计成 void，意味着 main 不会有任何值返回。 

 **4.main()方法中字符串参数数组作用** 

 main()方法中字符串参数数组作用是接收命令行输入参数的，命令行的参数之间用空格隔开。 

```c
/**
* 打印main方法中的输入参数 
*/ 
public class TestMain { 

    public static void main(String args[]){ 

        System.out.println("打印main方法中的输入参数！"); 

        for(int i=0;i<args.length;i++){ 

            System.out.println(args[i]); 
        } 
    } 
}
```



**3.1.4.1那么什么是方法呢？**

-   Java方法是**语句的集合**，它们在一起执行一个功能。
-   方法是解决一类问题的步骤的有序组合
-   方法**包含于类或对象中**
-   方法在程序中被创建，在其他地方被引用
-   

**3.1.4.2方法的优点**

1. 使程序变得更简短而清晰。

2. 有利于程序维护。

   3. 可以提高程序开发的效率。
3. 提高了代码的重用性。
   5. 

**3.1.4.3方法的命名规则** 

   1.方法的名字的第一个单词应以小写字母作为开头，后面的单词则用大写字母开头写，不使用连接符。例如：   **addPerson**。

   2.下划线可能出现在 JUnit 测试方法名称中用以分隔名称的逻辑组件。一个典型的模式是：例如 **testPop_emptyStack**。 



**3.1.4.4方法的语法**

**方法包含一个方法头和一个方法体。下面是一个方法的所有部分：**

- **修饰符：**修饰符，这是可选的，告诉编译器如何调用该方法。定义了该方法的访问类型。
- **返回值类型 ：**方法可能会返回值。returnValueType 是方法返回值的数据类型。有些方法执行所需的操作，但没有返回值。在这种情况下，returnValueType 是关键字**void**。
- **方法名：**是方法的实际名称。方法名和参数表共同构成方法签名。
- **参数类型：**参数像是一个占位符。当方法被调用时，传递值给参数。这个值被称为实参或变量。参数列表是指方法的参数类型、顺序和参数的个数。参数是可选的，方法可以不包含任何参数。
- **方法体：**方法体包含具体的语句，定义该方法的功能。

![1640315494947](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640315494947.png)

------

**3.1.4.5构造方法**



**3.1.4.5.1构造器的概括**

**(1)构造器的特征**

- 具有与类相同的名称
- 不声明返回值类型
- 不能被static final synchronized abstract  native 修饰 不能有return语句作为返回值。

**隐式构造器**：系统默认提供

**显式定义**一个或多个构造器(有参,无参)

**注意**：

- Java语言中，每个类都至少有一个构造器
- 默认构造器的修饰符与所属类的修饰符一致
- 一旦定义了一个构造器，则系统不在默认提供默认构造器
- 一个类可以创建多个重载的构造器
- 父类的构造器，不可被子类继承

**3.1.4.5.2构造器的作用**

创建对象；给对象进行初始化

![1640684919252](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640684919252.png)

**3.1.4.5.3构造器的语法格式**

![1640684955415](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640684955415.png)

------

**3.1.4.6方法重载**

**3.1.4.6.1重载的概念**

在同一个类中，允许一个以上的同名方法，只要它们的参数个数或者参数类型不同即可。

**3.1.4.6.2重载的特点**

与返回值类型无关，只看参数列表，且参数列表必须不同（参数个数或者参数类型）。调用时根据方法参数列表的不同来区别。

#### 3.1.5.7参数传递的方法

**3.1.5.7.1方法参数值传递机制**

**形参**：方法声明的参数

**实参** ：方法调用时实际传给形参的参数值

**Java的实参值如何传入方法的呢？**

回答：只有一种——值传递。即将实际参数的值的副本（复制品）传入方法内，而参数本身不受影响。

- 形参是**基本数据类型**：将实参基本数据类型变量的**数据值**传递给形参。
- 形参是**引用数据类型**：将实参引用数据类型变量的**地址值**传递给形参。

### 3.1.5类的成员之代码块（或初始化块）

**1.代码块的作用：用来初始化 类，对象**



**2.是否用static进行修饰进行分类**

**静态代码块：**

​       1.可以有输出语句

​       2.随着类的加载而执行

​       3.可以对类的属性，类声明进行初始化操作

​       4.不可以对非静态的属性初始化。即不可以调用非静态的属性和方法

​       5.静态代码块的执行要优于非静态代码块

​       6.静态代码块随着类的加载而加载，且只加载一次

​       7.可以调用静态的属性或者方法



**非静态代码块：**

​       1.内部可以有输出语句

​       2.随着对象的创建而执行

​       3.每创建一个对象就执行一次

​       4.作用：可以在创建对象时，可以对对象的属性进行初始化

​       5.如果一个类中定义了多个非静态代码块，则按照声明的先后顺序执行

​       6.静态或非静态都可以进行调用



**3.对属性可以赋值的位置**

​       1.默认初始化

​       2.显式初始化

​       3.构造器中初始化

​       4.有了对象以后，可以通过对象.属性 或 对象.方法的方式，进行赋值。

​       **5.在代码块中也可以进行赋值**

**4.代码实例**

```java
public class test_1 {
    public static void main(String[] args) {
       String desc=ss.desc;//类的加载
        System.out.println(desc);
        ss s=new ss();//对象的创建
    }
}
class ss{
    static String desc="饺子真好吃";//饺子真好吃
    public ss(){//构造器
    }
    public static void  info(){//静态的属性会随着类的加载而加载
        System.out.println("我是一个快乐的人！");
    }
    //静态代码块————随着类的加载而执行
    static{
        System.out.println("吃饺子");
    }
    //非静态代码块————随着对象的创建而执行（创建几个对象就执行几次）
    {
        System.out.println("Hello world！");
    }
}
```



### 3.1.6类的成员之内部类

## 3.2对象

### 3.2.1概念

对象：是实际存在的该类事物的每个个体，因而也称为实例。

### 3.2.2内存解析

![1640616400152](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640616400152.png)

 

## 3.3封装

### 3.3.1为什么需要封装？

1.  封装可以被认为是一个保**护屏障，**防止该类的代码和数据被外部类定义的代码随机访问。 
2.  要访问该类的代码和数据，必须通过严格的接口控制。 
3.  封装最主要的功能在于我们能修改自己的实现代码，而不用修改那些调用我们代码的程序片段。 
4.  适当的封装可以让程式码更容易理解与维护，也加强了程式码的安全性。 

 程序涉及追求——>**高内聚，低耦合**

  **高内聚：类的内部数组操作细节自己完成，不允许外部干涉。**

  **低耦合：仅对外暴露少量的方法用于使用。**

### 3.3.1信息封装后如何进行访问？

:red_circle:1.Java中通过将私有数据声明为私有的（private），在提供公共的（public）方法：getXxx（）和setXxx（）实现对该属性的操作，以实现下述目的：

隐藏一个类中不需要对外提供的实现细节

使用者只能通过实现定制好的方法来访问数据，可以方便地加入控制逻辑，限制对属性的不合理操作。

便于修改，增强代码的可维护性。

### 3.3.2访问权限的概括

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640705606794.png" alt="1640705606794" style="zoom: 67%;" />

![1640705626310](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640705626310.png)

## 3.4继承

### 3.4.1继承的出现及其作用

**出现的原因：**

- 多个类在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要继承那个类即可。
- 子类也称派生类
- 父类成基类或者超类



**作用：**

- 减少了代码的荣誉，提高了代码的复用性
- 继承的出现更有利于功能的扩展
- 继承的出现让类与类之间产生了关系，提供了多态的前提



**继承了什么：**

- 子类继承了父类，就继承了父类的方法和属性
- 在子类中可以使用父类中定义的方法和属性，也可以创建新的数据和方法。
- 在Java中，继承的关键字用的是extends，即子类不是父类的子集而是对父类的扩展。

### 3.4.2继承的初始化流程

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640792069676.png" alt="1640792069676" style="zoom: 50%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640792103613.png" alt="1640792103613" style="zoom:80%;" />

### 3.4.3继承中方法的重写

**定义：**

在子类中可以根据需要从父类中继承来的方法进行改造，也称为方法的重置，覆盖。在程序执行时，子类的方法将覆盖父类的方法

**要求：**

- `子类重写的反复`噶必须和父类被重写的方法具有相同的方法名称，参数列表。

- 子类重写的方法的返回值类型不能大于父类被重写的方法的返回值类型

- 子类重写的方法使用的访问权限不能小于父类被重写的放大的访问权限

- > 子类不能重写父类中声明为private权限的写法

- 子类方法抛出的异常不能大于父类被重写方法的异常

  

**关于重写方法的举例：**

![1640792651196](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640792651196.png)



![1640792662495](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640792662495.png)

## 3.5多态（更新修改）

### 3.5.1 关于对多态的理解

**多态是同一个行为具有多个不同表现形式或形态的能力。**

**多态就是同一个接口，使用不同的实例而执行不同操作，如图所示：**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826154158592.png" alt="image-20220826154158592" style="zoom: 67%;" />

#### 1.1多态的优点

- 消除类型之间的耦合关系
- 可替换性
- 可扩充性
- 接口性
- 灵活性
- 简化性

#### 1.2多态存在的三个必要条件

- 继承
- 重写
- 父类引用指向子类对象

![image-20220826154452447](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826154452447.png)

#### 1.3多态的实现方式

​		**当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，再去调用子类的同名方法。**

​		**多态的好处：可以使程序有良好的扩展，并可以对所有类的对象进行通用处理。**

==下面我们通过一个代码示例，通过丰富的注释和解析，深入理解Java多态：==

```java
package com.fang.wordcount_1;

public class Test {
    public static void main(String[] args) {
        show(new Cat());  // 以 Cat 对象调用 show 方法
        show(new Dog());  // 以 Dog 对象调用 show 方法
        /**
         * 上面两行可以发现，show方法要求传入的是动物对象，因为猫和狗都继承了动物类，因此符合规范，
         *         同时体现出多态的优势：一个形参可以对应多个实参，同时这也是一个重写式多态
         */


        Animal a = new Cat();  // 向上转型：通过子类实例化父类
        a.eat();               // 调用的是 Cat 的 eat
        //a.work();如果运行这一行就会发现，无法调用work方法，因为动物类只有eat一个方法，从而cat失去了特有方法

        Cat c = (Cat)a;        // 向下转型：通过父类强制转化为子类
        c.work();        // 调用的是 Cat 的 work
        /**
         * 上面两行体现了向下转型的用处，我们可以知道，对象a目前是一个动物对象，不能执行猫或者狗的特有方法
         * 但是，如果通过向下转型，将动物a对象，转化为一个猫c对象，这样就可以调用猫的特有方法了
         */


        /**
         * 得出结论：
         * 向上转型 : 通过子类对象(小范围)实例化父类对象(大范围),这种属于自动转换
         * 向下转型 : 通过父类对象(大范围)实例化子类对象(小范围),这种属于强制转换
         */

    }

    public static void show(Animal a)  {
        a.eat();
        // 类型判断
        if (a instanceof Cat)  {  // 猫做的事情
            Cat c = (Cat)a;
            c.work();
        } else if (a instanceof Dog) { // 狗做的事情
            Dog c = (Dog)a;
            c.work();
        }
    }
}

//定义一个抽象类
abstract class Animal {
    abstract void eat();
}

//下面的每一个类继承抽象类，重写接口中的方法
class Cat extends Animal {
    public void eat() {
        System.out.println("吃鱼");
    }
    public void work() {
        System.out.println("抓老鼠");
    }
}

class Dog extends Animal {
    public void eat() {
        System.out.println("吃骨头");
    }
    public void work() {
        System.out.println("看家");
    }
}
```

如果你能理解整个代码的核心内容，那么你已经掌握了多态！

**对于多态我的理解是：同一个行为具有多个不同表现形式或形态的能力就是多态**

#### 1.3补充说明

多态一般分为两种：重写式多态和重载式多态。

**重载式多态**，也叫编译时多态。也就是说这种多态再编译时已经确定好了。重载大家都知道，==方法名相同而参数列表不同的一组方法就是重载==。在调用这种重载的方法时，==通过传入不同的参数最后得到不同的结果==。

> 但是这里是有歧义的，有的人觉得不应该把重载也算作多态。因为很多人对多态的理解是：程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，这种情况叫做多态。 这个定义中描述的就是我们的第二种多态—重写式多态。



**重写式多态**，也叫运行时多态。这种多态通过动态绑定（dynamic binding）技术来实现，是指在执行期间判断所引用对象的实际类型，根据其实际的类型调用其相应的方法。也就是说，只有程序运行起来，你才知道调用的是哪个子类的方法。 这种多态通过函数的重写以及向上转型来实现，我们上面代码中的例子就是一个完整的重写式多态。我们接下来讲的所有多态都是重写式多态，因为它才是面向对象编程中真正的多态。

==总而言之我的理解：重载式多态，在编码等过程中，并没有很好的体现出多态的优势，但是不得否认也是多态的一种编写方式，而给出的重写式多态案例中，相比于重载式多态，在编码思路和代码量以及聚合度方面都较好的体现出了多态的优势，==



### 3.5.2向下转型的使用

有了对象的多态性以后，内存中实际上加载了子类他有的属性和方法，但是由于变量声明为父类类型，导致编译时，只能调用父类中声明的方法，子类特有的属性和方法不能调用。

如何才能调用子类特有的属性和方法？

> **Man m1=(Man)p2;//将Person类型转换为Man类型**

使用强制类型转换符————向下转型转换->具有一定的风险->可能转换不成功出现异常

![1640940194811](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640940194811.png)

## 3.6抽象类与抽象方法

1. 随着继承层次中一个个新子类的定义，类变的越来越具体，而父类则更一般，更通用。

类的设计应该保证父类和子类能够共享特性，**有时将一个父类设计的非常抽象，以至于它没有具体的实例。**

   2.用abstract关键字来修饰一个类，这个类叫做抽象类。

   3.用abstract来修饰一个方法，该方法叫做抽象方法。

​           抽象方法：只有方法的声明，没有方法的实现，以分号结束。

​           例如：public abstact void talk();

   4.抽象方法要包含在抽象类中。

   5.抽象类不能创建对象，而是用来继承的，抽象类的子类必须重写父类的抽象方法，并提供方法体

​      如果没有重写全部的抽象方法，仍为抽象类。

   **6.不能修饰 变量 代码块 构造器 私有方法 静态方法 final方法 final类**

应用说明：

抽象类是用来模型化那些**父类无法确定全部实现**，而是由其**子类提供具体实现的对象的类**。

## 3.7匿名对象和抽象类的匿名子类

**一.匿名对象——没有名字的对象**

  匿名对象的应用场景

​     仅仅只调用一次方法的时候--------节省代码

​     匿名对象也可以作为实际参数传递

```
public class test_1 {
    public static void main(String[] args) {
       Car c1=new Car();
       c1.run();

       new Car().run();//匿名对象只适合对方法的一次调用，因为调用多次就会产生多个对象，不如用有名字的对象。
        //匿名对象是否可以调用属性并赋值？有什么意义？
         /*
         * 匿名对象可以调用属性，但是没有任何意义，调用后会变成垃圾。
         * 如果需要调用，需要用到有名字的对象
         * */
    }
}
class Car{
  String color;
  int num;

  public void run(){
      System.out.println("车运行");
  }
}
```

  

```
//作为参数的值进行传递
public class test_1 {
    public static void main(String[] args) {
        Car c1=new Car();
       method(c1);
        Car c2=new Car();
       method(c2);
    }
    //抽取提高代码的复用性
    public static void method(Car cc){
        cc.color="red";
        cc.num=8;
        cc.run();
    }
}
class Car{
  String color;
  int num;

  public void run(){
      System.out.println("车运行");
  }
}
```



**二.抽象类的匿名子类**

## 3.8接口

- 因Java不支持多重继承（从几个类中派生出一个子类），有了接口，就可以得到多重继承的效果。
- 接口的本质是契约，标准和规范。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641131808076.png" alt="1641131808076" style="zoom:80%;" />

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641131818976.png" alt="1641131818976" style="zoom:80%;" />



**概念：接口是抽象方法和常量值定义的集合（JDK8以后还可以定义静态方法和默认方法）。**

### **3.8.1接口的特点：**

- 用interface来定义
- 成员变量默认public static final修饰
- 抽象方法默认public abstract修饰
- 接口中没有构造器
- 接口采用多重继承机制
- 接口和类是两个并列的结构，从本质上来讲接口是一种特殊的抽象类，这种抽象类中只包含常量和方法的定义，而没有变量和方法的实现。

### **3.8.2实现接口的语法**

```java
class SubClass extends SuperClass implements InterfaceA{}
```

**先写extends后写implements**

一个类可以实现多个接口，接口也可以继承其他接口

实现接口的类中必须提供接口中所有方法的具体内容，方可实例化。否则，仍为抽象类

接口的主要作用就是被实现类实现

与继承关系类似，接口与实现类之间存在多态性

**接口与抽象类的对比**

![1641132717830](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641132717830.png)

```java
/*
接口使用：
1.接口使用上也满足多态性
2.接口实际上就是定义了一种规范
3.开发中体会面向对象编程
*/
//体现了接口的多态性
public class test_1 {
    public static void main(String[] args) {
        Computer com=new Computer();
        Flash flash=new Flash();
        com.trans(flash);
    }
}
class Computer{
    public void trans(USB usb){   //USB usb=new Flash();
        usb.start();  //执行的是重写之后的方法
        System.out.println("具体传输数据细节");
        usb.stop();
    }
}
interface USB{
    void start();
    void stop();
}
class Flash implements USB{
    @Override
    public void start() {
        System.out.println("U盘开始工作");
    }

    @Override
    public void stop() {
        System.out.println("U盘结束工作");
    }
}
```





## 3.9内部类（内容更新）

**介绍**

（1）当一个事物内部，还有一个部分需要一个完整的结构进行描述，而这个**完整的结构又只为外部事物提供服务**，那么整个内部的**完整结构最好使用 内部类**。（Java中允许将一个类A声明在另一个类B当中）

（2）**在Java中允许一个类的定义位于另一个类的内部**，前者称为内部类，后者称为外部类。

（3）Ineer class一般用在定义它的**类或语句块之内**，在**外部引用它时必须给出完整的名称**。

​         Ineer class的名字不能与包含它的外部类类名相同

**（4）分类**

​      **成员内部类：static成员内部类和非static成员内部类**

​      **局部内部类：方法内 代码块内 构造器内**

```java
public class hhh {
}
class Person{
    //静态成员内部类
    static class Dog{
    }
    //非静态成员内部类
    class bird{ 
    }
    public void method{
        class AA{
        //局部内部类
        }

        class BB{
        //局部内部类
        }

    }
}
```

**成员内部类的介绍**

**1.作为外部类的成员：**

- 可以调用外部类的结构
- 可以被static（一般是用来修饰类内的成员）修饰
- 可以被四种不同的权限修饰

```java
class Person{
    String name;
    int age;
    public void eat(){
        System.out.println("人吃饭！！！");
    }

    //非静态成员内部类
    final class Bird{

         String name;
         public Bird(){

         }
         public void sing(){
             System.out.println("我是一只小小鸟");
             Person.this.eat();//调用外部类的使用
         }
    }
}       
```

2.作为一个类：

- 类内 可以定义属性，方法，构造器。
- 可以被final修饰，表示此类不能被继承；言外之意不使用final就可以被继承。
- 可以被abstract修饰，修饰后不能被实例化

```java
    abstract static class Dog{
        String name;
        public Dog(){

        }
        public void sing(){
            System.out.println("汪汪");
        }
    }
    //非静态成员内部类
    final class Bird{

         String name;
         public Bird(){

         }
         public void sing(){
             System.out.println("我是一只小小鸟");
         }
    }
```



**主要关注一下几个问题：**

**如何实例化内部类的对象？**

```java
public class hhh {
    public static void main(String[] args) {
        //创建Dog实例（静态成员内部类）
        Person.Dog dog=new Person.Dog();
        dog.sing();
        //创建Bird实例(非静态的成员内部类)
        Person p=new Person();  //有了对象以后通过对象调用
        Person.Bird bird=p.new Bird();
        bird.sing();
    }
}
class Person{
    String name;
    int age;
    public void eat(){
        System.out.println("人吃饭！！！");
    }
    //静态成员内部类
    static class Dog{
        String name;
        public Dog(){

        }
        public void sing(){
            System.out.println("汪汪");
        }
    }
    //非静态成员内部类
    final class Bird{

         String name;
         public Bird(){

         }
         public void sing(){
             System.out.println("我是一只小小鸟");
             Person.this.eat();//调用外部类的使用
         }
    }
}    
```

**如何在从成员内部类中区分和调用外部类的结构（特别是当重名的时候）？**

```java
   final class Bird{

         String name;
         public Bird(){

         }
         public void sing(){
             System.out.println("我是一只小小鸟");
             Person.this.eat();//调用外部类的使用
         }
        public void display(String name){
            System.out.println(name);
            System.out.println(this.name);
            System.out.println(Person.this.name);
        }
    }
```

**开发中内部类的使用？**

```java
public class hhh {
    //开发当中很少见
    public void  method(){
        class AA{

        }

    }
    //返回一个实现了Comparable接口的类的对象
    public  Comparable getComparable(){
        class MyComparable implements Comparable{

            @Override
            public int compareTo(Object o) {
                return 0;
            }
        }
        return  new MyComparable();

    }
}
```



### 3.9.1Object类的使用

**1.介绍**

==Object类是所有Java类的根父类==

**如果在类的声明中未使用extends关键字知名其父类，则默认父类为Java.lang.Object类**

**2.功能**

Object类中的功能（属性，方法）就具有通用性

**属性：无**

finalize方法名，是进行垃圾回收的方法，当没有任何对象引用的时候，就会调用该方法，但是不要自己去调用这个方法，而是让JVM自己去调用

getClass（）获取当前对象的所属类

hashCode（）返回当前对象的哈希值

**toString（）**

1.当我们输出一个对象的引用时，实际上就是调用了当前对象的toString（）方法

2.toString（）的定义

```java
public String toString{
    return getClass().getName()+"@"+Integer.toHexString(hashCode());
}//返回类名和他的引用地址
```

3.像String Date  File 包装类 都重写了Object中的toString（）方法。

   使得在调用对象的toString（）方法时，返回“实体内容”的值

4.自定义类也可以重写toString())方法，当调用此方法时，返回对象的实体内容。



**equals（）对象比较**

对“==”复习

**可以使用在基本数据类型变量和引用数据类型变量中**

如果比较的是基本数据类型变量，**比较两个变量保存的数据是否相等**（不一定类型要相同）

如果比较的是引用数据类型对象，**比较两个对象的地址值是否相等**，即两个引用是否指向同一对象实体

```java
public class EqualsTest{
   public static void main(String[] args){
   int i=10;
   int j=10;
   double d=10.0;
   System.out.println(i==j);
   System.out.println(i==d);
   
   boolean b=true;
   
   char c=10;
   System.out.println(i==c); //true
   char c1='A';
   char c2=65;
   System.out.println(c1=c2);//true
   }
}
```

**二.equals（）方法的使用**

1.只能够适用于引用数据类型

2.如果该方法没有被重写过默认也是“==”

3.方法的使用

System.out.println(person1.equals(person2))

4.像String，Date，File，包装类等都重写了Object类中的equals（）方法

​          重写以后比较的是实际内容是否是相同的。

   

clone（） 

finalize（）

wait（）

notify（）

notifyAll（）



### 3.9.2包装类的使用Wrapper

![1640956705159](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640956705159.png)

**1.对于八种基本的数据类型定义相应的引用类型—包装类（封装类）**

**2.有了类的特点，就可以调用类中的方法，Java才是真正的面向对象。**

**基本数据类型 包装类 String类间的转换**

**1.基本数据类型------>包装类   装箱**

```java
1.调用包装类的构造器
int i=500;
Integer t=new Integer(i);
Integer In2 = new Integer("123");
Boolean b2=new Boolean(true123)//忽略大小写的情况下，只要是true就是true  如果不是就是false 

    boolean isMale; //默认值是false
    Boolean isFemale; //默认值是null
    //第一个为基本数据类型
    //第二个为一个类
```

**2.包装类------->转化为基本数据类型：调用包装类的xxxValue()  拆箱**

```java
public void test2(){
  Integer in1=new Integer(12);
  in1.intValue;
  System.out.println(i1+1);
  
  Float f1=new Float(12.3);
  float f2=f1.floatValue();
  System.out.println(f2+1);
}
```

**3.自动拆箱和自动装箱**

```
int num2=10；//自动装箱
Integer in1=num2；

boolean b1=true;
Boolean b2=b1;

in1 num3=in1；//自动拆箱
```

**4.基本数据类型，包装类----->String类型**

```java
方式1：连接运算
public void test4(){
  int num1=10;
  String str1=num1+"";//基本数据类型转换成String 
}

方式2：调用String重载的valueOf(Xxx xxx)
public void test4(){
    float f1=12.3f;            //基本数据类型转化为String
    String str2=String.valueOf(f1);//"12.3"
    
    Double d1=new Double(12.4);//包装类转化为String
    String d2=String.value(d1);
}

```

**5.String类型------->基本数据类型，包装类**

```java
调用”包装类“的parseXxx（）方法
public calss test5{
   String str1="123";
   int num2=Integer.parseInt(str1);
    
    String stu2="true1";
    boolean b1=Boolean.parseBoolean(str2);
}
```

![1640962917406](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1640962917406.png)



### 3.9.3int与Integer的区别

#### 1.0 了解java的两种数据类型

![image-20220826195652998](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826195652998.png)

---

==为什么要有基本数据类型和引用数据类型：==

- 基本数据类型是编程语言内置定义的数据类型，不可简化的，它表示了真实的数字、字符
- 引用数据类型其实就是对基本数据类型的一个封装Java所有引用数据类型都继承于Object类，都是按照Java存储对象的内存模型进行存储，引用是存储在内存栈上的，而对象本身的值存储在内存堆上

**引用数据类型：**
		它封装了数据和处理该数据的方法，比如Integer.parseInt(String)就是将String字符类型数据转换为Integer整型数据。

​		Java中大部分类和方法都是针对类类型对象的，比如==ArrayList集合类就只能以类作为它的存储对象，如果要储存int型数据就要将其包装成一个类即Integer，然后才能存入list==。

==引用类型在堆（效率较低）里，基本类型在栈里。==
栈空间小且连续，往往会被放在缓存。

​		引用类型cache miss（缓存未命中）率高且要多一次解引用。对象还要再多储存一个对象头，对基本数据类型来说空间浪费率太高，**因此放在堆（一种特殊的完全二叉树）中。**

​		逻辑上来讲，==java只有**包装类**就够了，为了运行速度，需要用到**基本数据类型**==。实际上，任何一门语言设计之初，都会优先考虑运行效率的问题，所以二者同时存在是合乎情理的，如果你了解Scala就会清楚的知道，二者的区别，从而也说明，==Java并不是完全面向对象的语言。==



**通过上面的介绍你或许已经知道了为什么需要Integer了**

​		万物皆可对象，为了能够将这些基本数据类型当成对象操作，Java为每一个数据类型都引入了对应的包装类型（wrapper class）。

![image-20220826200254175](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826200254175.png)

**那么他们的区别是什么呢？**

（1）Integer是int的包装类；int是基本数据类型；
（2）Integer变量必须实例化后才能使用；int变量不需要；
（3）Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ；
（4）Integer的默认值是null；int的默认值是0。

**或许文字叙述你并不理解，那么我们用代码的方式看一看吧！**

#### 2.0对比int和Integer

**1.由于是对象，因此他们的内存地址不同**

```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```

**2.包装类Integer和int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较**

```java
Integer i = new Integer(100);
int j = 100；
System.out.print(i == j); //true
```

**3.非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同**

```java
Integer i = new Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```

**4.两个非new生成的Integer对象，进行比较，两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false**

```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true

Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```

==第4条的原因： java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)。而java API中对Integer类型的valueOf的定义如下，对于-128到127之间的数，会进行缓存，Integer i = 127时，会将127这个Integer对象进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了。==

#### 3.0 什么是自动拆箱和自动装箱

**自动装箱：将基本数据类型重新转化为对象**

```java
 // 声明一个Integer对象，用到了自动的装箱：解析为:Integer num = Integer.valueOf(9);
	Integer num = 9;
```

**自动拆箱：将对象重新转化为基本数据类型**

```java
 / /声明一个Integer对象
	 Integer num = 9;
            
// 进行计算时隐含的有自动拆箱
	 System.out.print(num--);
```

因为**对象时不能直接进行运算的，而是要转化为基本数据类型后才能进行加减乘除**。对比：

```java
// 装箱
Integer num = 10;
// 拆箱
int num1 = num;
```

#### 4.0 深入理解Integer

```java
 			// 在-128~127 之外的数
            Integer num1 = 128;   
            Integer num2 = 128;           
            System.out.println(num1==num2);   //false
                        
            // 在-128~127 之内的数 
            Integer num3 = 9;   
            Integer num4 = 9;   
            System.out.println(num3==num4);   //true
```

**解析原因：**归结于java对于Integer与int的自动装箱与拆箱的设计，是一种模式：叫**享元模式**（flyweight）。
（1）加大对简单数字的重利用，Java定义在自动装箱时对于在-128~127之内的数值，**它们被装箱为Integer对象后，会存在内存中被重用，始终只存在一个对象。**
（2）**而如果在-128~127之外的数，被装箱后的Integer对象并不会被重用，即相当于每次装箱时都新建一个 Integer对象。**



==最后一点：我们都知道java中所有的类都继承自Object类，那么包装类同时也具有继承关系：==

![image-20220826210959994](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826210959994.png)









