# Java工厂模式+抽象工厂模式

​		**本文主要通过介绍工厂设计模式的基本原理概念+Java API实现，带你深度了解Java工厂设计模式。希望对你有所帮助！**

如需获取更多源码，笔记，教程，请访问本人仓库，请[点击获取](https://gitee.com/fanggaolei/learning-notes-warehouse)你的关注是我创作的动力！

[TOC]



# 1.0工厂模式概述

- **简单工厂模式**  
- **工厂方法模式**   
- **抽象工厂模式**

# 1.1 概念

**==一句话理解：工厂模式主要通过接口和抽象类的方式，实现调用者和生产者的分离，从而达到，在不同条件下创建不同对象的目的==**

**实现了创建者与调用者的分离，即将创建对象的具体过程，屏蔽隔离起来达到提高灵活性的目的。**

**其实设计模式和面向对象设计原则都是为了使得开发项目更加容易扩展和维护，解决方式就是一个“分工”。** 

# 1.2  分类和本质说明

**简单工厂模式：用来生产同一等级结构中的任意产品（对于增加新的产品需要修改已有代码）**

**工厂方法模式：用来生产同一等级结构中的固定产品（支持增加任意产品）**

**抽象工厂模式：用来生产不同产品族的全部产品（对于增加新的产品，无能为力，支持增加产品族）**



**工厂模式的核心本质**

==实例化对象，用工厂方法代替new操作==

将选择实现类，创建对象统一管理和控制。从而将调用者和我们的实现类解耦。

# 2.0编码理解

## 2.1无工厂模式

**创建者和调用者在一起**

```java
package com.fang.factory;


interface Car{
    void run();
}

class Audi implements Car{

    public void run() {
        System.out.println("奥迪已生产");
    }
}

class Byd implements Car{

    public void run() {
        System.out.println("比亚迪已生产");
    }
}

public class Client_1 {
    public static void main(String[] args) {
        Car audi=new Audi();//创建者
        Byd byd = new Byd();
        audi.run();//调用者
        byd.run();
        //创建者和调用者在一起
    }
}
```

## 2.2简单工厂模式

![image-20220825212245790](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825212245790.png)

**体现创建者与调用者分离**

**缺点：对于增加新产品，不修改代码无法扩展违反了开闭原则，对扩展开放，对修改封闭**

```java
interface Car{
    void run();
}

class Audi implements Car{

    public void run() {
        System.out.println("奥迪已生产");
    }
}

class Byd implements Car{

    public void run() {
        System.out.println("比亚迪已生产");
    }
}

//工厂类-体现创造
class Factory{
    public static Car getCar(String type){
        if ("奥迪".equals(type)){
            return new Audi();
        }else if ("比亚迪".equals(type)){
            return new Byd();
        }else {
            return null;
        }
    }
}


public class Client_2{
    public static void main(String[] args) {
        
        //体现调用
        Car audi=Factory.getCar("奥迪");
        audi.run();

        Car byd = Factory.getCar("比亚迪");
        byd.run();
        //创建者与调用者分离
    }
}

```

## 2.2工厂方法模式

```java

interface Car{
    void run();
}

//两个实现类
class Audi implements Car{

    public void run() {
        System.out.println("奥迪run");
    }
}

class Byd implements Car{

    public void run() {
        System.out.println("比亚迪run");
    }
}

//工厂接口
interface Factory{
    Car getCar();
}
//两个工厂类
class AudiFactory implements Factory{
    public Audi getCar(){
        return new Audi();
    }
}

class BydFactory implements Factory{
    public Byd getCar(){
        return new Byd();
    }
}

public class Client_3 {
    public static void main(String[] args) {
        Car a=new AudiFactory().getCar();
        Car b=new BydFactory().getCar();
        a.run();
        b.run();
    }
}
```

​		==简单工厂和工厂方法模式并没有真正避免了代码的改动，简单工厂需要更改相关if else语句，而工厂方法模式需要将具体工厂角色写死。==

**但均以体现创建者和调用者的分离**

# 3.0 抽象工厂模式

![image-20220825212126156](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825212126156.png)

​		**抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。**

​		在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

**抽象工厂必然用到抽象方法对于抽象类的说明：**

抽象类不可以实例化

继承一个抽象类，必须实现抽象类中的抽象方法，除非子类也是抽象类。

抽象方法是一个模板或约束，避免了子类的随意性，需要实现它的类必须重写它的抽象那个方法。

```java
//汽车方法接口
interface Car{
    void run();
}
/******************************************/
//两个实现类
class Audi implements Car{

    public void run() {
        System.out.println("奥迪run");
    }
}

class Byd implements Car{

    public void run() {
        System.out.println("比亚迪run");
    }
}
/******************************************/

//颜色方法接口
interface Color {
    void fill();
}
/******************************************/
//两个实现类
class Red implements Color {
    public void fill() {
        System.out.println("红色");
    }
}

class Green implements Color {
    public void fill() {
        System.out.println("绿色");
    }
}
/******************************************/
//创建抽象类获取工厂
abstract class AbstractFactory {
    public abstract Color getColor(String color);
    public abstract Car getCar(String shape);
}
/******************************************/
class CarFactory extends AbstractFactory {

    public Car getCar(String type){
        if ("奥迪".equals(type)){
            return new Audi();
        }else if ("比亚迪".equals(type)){
            return new Byd();
        }else {
            return null;
      }
    }
    public Color getColor(String color) {
        return null;
    }
}
/******************************************/
class ColorFactory extends AbstractFactory {

    public Car getCar(String shape){
        return null;
    }

    public Color getColor(String color) {
        if(color == null){
            return null;
        }
        if(color.equalsIgnoreCase("红色")){
            return new Red();
        } else if(color.equalsIgnoreCase("绿色")){
            return new Green();
        } else {
            return null;
        }
    }
}
/******************************************/
//工厂建造器-工厂的工厂
class FactoryProducer {
    public static AbstractFactory getFactory(String choice){
        if(choice.equals("car")){
            return new CarFactory();
        } else if(choice.equals("color")){
            return new ColorFactory();
        }
        return null;
    }
}
/******************************************/
public class Client_4 {
    public static void main(String[] args) {

        //获取汽车创建工厂
        AbstractFactory car = FactoryProducer.getFactory("car");
        Car audi = car.getCar("奥迪");
        audi.run();

        //获取颜色创建工厂
        AbstractFactory color = FactoryProducer.getFactory("color");
        //获取颜色为 Red 的对象
        Color color1 = color.getColor("红色");
        color1.fill();
    }
}
```

