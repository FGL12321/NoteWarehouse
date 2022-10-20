# Java单例设计模式

[TOC]

​		**本文主要是通过对Java单例模式的原理说明，并通过相关API带你了解Java单例模式的实现过程以及用途，希望对你有所帮助！**

如需获取更多源码，笔记，教程，请访问本人仓库，你的关注是我创作的动力！

链接：[https://gitee.com/fanggaolei/learning-notes-warehouse](

# 1.0Java单例模式的概述

==一句话说明：一个类只能创建一个对象，用于计数器，数据库连接池，读取配置文件等==

1.采用一定的方法保证在整个软件系统中，**对某一个类只能存在一个对象实例**，并且该类只能提供一个取得其对象实例的方法。

2.如果我们要让类在一个虚拟机中只产生一个对象，我们首先必须即将**类的构造器的访问权限设置为private**，这样就不能用new操作符在类的外部产生类的对象，但在**类的内部**可以产生类的对象。
因为在类的**外部**开始还**无法得到类的对象**，只能调用该类的某个**静态方法**以**返回类内部创建的对象**。静态方法只能访问类中的静态成员变量，所以，指向类内部产生的该**类的对象的变量也必须定义成静态的**。

**3.单例设计模式的优点**

单例设计模式只生成一个实例，减少了系统的开销，当一个对象的产生需要比较多的资源的时候，

   例如读去配置，产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久驻留内存的方式来解决

**4.单例设计模式的应用场景**

- **网站的计数器**，一般也是单例模式实现，否则难以同步。
- **应用程序的日志应用**，一般都使用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作，否则内容不好追加。
- **数据库连接池**的设计一般也是采用单例模式，因为数据库连接是一种数据库资源。
- 项目中，**读取配置文件的类**，一般也只有一个对象。没有必要每次使用配置文件数据，都生成一个对象去读取。
- **Application 也是单例的典型应用**
- Windows的**Task Manager (任务管理器)**就是很典型的单例模式
- Windows的**Recycle Bin (回收站)**也是典型的单例应用。在整个系统运行过程
  中，回收站一直维护着仅有的一个实例。

**5.区分饿汉式和懒汉式**

**饿汉式：对象加载时间过长。**

​                **饿汉式是线程安全**

​				==启动时加载==

**懒汉式：延迟对象的创建。**

​               **线程是不安全的**

​				==调用时加载==

# 2.0Java单例模式的API说明

## 2.1饿汉式

```java
package com.fang.singo_case;

public class SingletonTest1 {
    public static void main(String[] args) {

        Bank bank1=Bank.getInstance();
        Bank bank2=Bank.getInstance();

        System.out.println(bank1==bank2);

    }
}

class Bank{

    //1.私有化类的构造器
    private Bank(){
    }
    //2.内部创建类的对象，要求此对象必须声明为静态的
    private static Bank instance=new Bank();

    //3.提供公共的方法返回类的对象
    public static Bank getInstance(){  //说明：satic方法同类加载时一起创建
        return instance;
    }
}
```

![image-20220825150816043](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825150816043.png)

==运行结果表明：获取的两个对象为同一个对象==

## 2.2懒汉式

首先介绍一个关键字==synchronized==解决线程安全问题

```
synchronized（同步监视器相当于一个锁）{

//需要被同步的代码

}
```

```java
package com.fang.singo_case;

//单例设计模式的懒汉式表现
public class SingletonTest2 {
    public static void main(String[] args) {
        Bank2 order1=Bank2.getInstance();
        Bank2 order2=Bank2.getInstance();
        System.out.println(order1==order2);
    }
}
class Bank2{
    private  Bank2(){

    }
    private static Bank2 instance=null;

    public  static synchronized   Bank2 getInstance(){
        //方式1：如果多个run()调用此方法，存在线程不安全  增加一个synchronized  静态方法的锁是当前类的根Bank2.class
        if (instance == null) {
            //如果这个部分进入两个两个线程，则会导致线程不安全，因此需要synchronized修饰
            instance=new Bank2();
        }
        return instance;
    }


//    public  static synchronized   Bank2 getInstance2(){
//     方式2：效率低
//        synchronized (Bank2.this){
//            if (instance == null) {
//                instance=new Bank2();
//            }
//            return instance;
//        }
//
//    }


//    public  static synchronized   Bank2 getInstance2(){
        //方式3：效率更高
//      if (instance == null) {//在后面的线程到来后，不需要再进入下面的代码块，提高效率
//        synchronized (Bank2.this){
//            if (instance == null) {
//                instance=new Bank2();
//             }
//        }
//      }
//      return instance;
}
```

