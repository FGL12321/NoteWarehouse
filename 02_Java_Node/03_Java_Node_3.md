[TOC]



# 10.0 IO流（待补充）

## 10.1   File类的使用

## 10.2   IO流原理及流的分类 

## 10.3	节点流(或文件流) 

## 10.4	缓冲流 

## 10.5	转换流 

## 10.6	标准输入、输出流 

## 10.7	打印流

## 10.8	数据流 

## 10.9	对象流 

## 10.10	随机存取文件流 

## 10.11	NIO.2中Path、Paths、 Files类的使用 

# 11.0网络编程（待补充）

## 11.1网络通信要素概述 

## 11.2通信要素1：IP和端口号 

## 11.3通信要素2：网络协议 

## 11.4网络编程概述 

## 11.5TCP网络编程 

## 11.6UDP网络编程 

## 11.7URL编程

# 12.0Java反射机制（待补充）

## 12.1理解Class类并获取Class实例 

## 12.2类的加载与ClassLoader的理解 

## 12.3创建运行时类的对象 

## 12.4Java反射机制概述 

## 12.5获取运行时类的完整结构 

## 12.6调用运行时类的指定结构 

## 12.7反射的应用：动态代理 

# 13.0设计模式（更新）

## 13.1概述

设计模式是在大量的实践中总结和理论化之后优选的代码结构，编程风格，以解决问题的思考方式。

设计模式就像是经典的棋谱。不同的器具，我们用不同的棋谱，免去我们自己去思考和摸索。

- 单一职责原则 ：一个类只负责一项职责
- 开放-关闭原则 ：可以被扩展的，但是不可被修改
- 里氏替换原则：里氏替换原则的重点在不影响原功能，而不是不覆盖原方法
- 依赖倒转原则 ：依赖倒转原则的核心思想就是面向接口编程
- 接口隔离原则 ：客户端不应该依赖它不需要的接口
- 迪米特法则：将逻辑封装在类的内部，对外提供 public 方法，不对泄漏任何信息
- 组合/聚合复用原则 ：尽量使用组合/聚合，不要使用类继承



![1641039494153](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641039494153.png)

## 13.2单例设计模式

**解释:**

1.采用一定的方法保证在整个软件系统中，**对某一个类只能存在一个对象实例**，并且该类只能提供一个取得其对象实例的方法。

2.如果我们要让类在一个虚拟机中只产生一个对象，我们首先必须即将**类的构造器的访问权限设置为private**，这样就不能用new操作符在类的外部产生类的对象，但在**类的内部**可以产生类的对象。
因为在类的**外部**开始还**无法得到类的对象**，只能调用该类的某个**静态方法**以**返回类内部创建的对象**。静态方法只能访问类中的静态成员变量，所以，指向类内部产生的该**类的对象的变量也必须定义成静态的**。

**3.单例设计模式的优点**

单例设计模式只生成一个实例，减少了系统的开销，当一

个对象的产生需要比较多的资源的时候，

   例如读去配置，产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久主流内存的方式来解决

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

饿汉式：对象加载时间过长。

​                饿汉式是线程安全

懒汉式：延迟对象的创建。

​               线程是不安全的

​               



**举例1：饿汉式**--提前准备好

```java
//单例设计模式
public class SingletonTest1{
    public static void main(String[] args) {
        Bank bank1=Bank.getInstance();
        Bank bank2=Bank.getInstance();

        System.out.println(bank1==bank2);
    }
}

class Bank{
    //1.私有化类的构造器(防止在外面创建对象)
    private  Bank(){

    }
    //2.内部创建类的对象
    //4.要求此对象也必须声明为静态的
    private static Bank instance=new Bank();

    //3.提供公共的方法，返回类的对象
    public static Bank getInstance(){
        return instance;
    }
}
```

**举例2：懒汉式**--啥时候用啥时候造

```java
package package_no1.Singletom.test_2;
//单例设计模式的懒汉式表现
public class SingletonTest2 {
    public static void main(String[] args) {
         Order order1=Order.getInstance();
         Order order2=Order.getInstance();
        System.out.println(order1==order2);
    }
}
class Bank2/*Bank2=Order*/{
    private  Bank2(){

    }
    private static Bank2 instance=null;

    public  static synchronized   Bank2 getInstance(){
        //方式1：如果多个run()调用此方法，存在线程不安全  增加一个synchronized  锁是Bank().class
        if (instance == null) {
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
//      if (instance == null) {
//        synchronized (Bank2.this){
//            if (instance == null) {
//                instance=new Bank2();
//             }
//          }
//
//      }
//      return instance;
}
```



## 13.3模板方法的设计模式

 抽象类体现的就是一种**模板模式的设计**，抽象类作为多个子类的通用模板，子类在抽象类的基础上进行扩展，改造，但**子类总体上会保留抽象类的行为方式**。



**解决的问题：**

1.当功能内部一部分实现是确定的，一部分实现是不确定的。可以把不确定的部分暴露出去，让子类去实现。

   **换句话说，在软件开发中实现一个算法，整体步骤很固定，通用，这些步骤已经在父类中写好了。但是某些部分易变，易变部分可以抽象出来，供不同子类实现，这就是模板模式。**

```java
public class em_1 {
    public static void main(String[] args) {
        emm e=new SbuRem();
        e.spendTime();
    }

}
abstract class emm{
    //计算某段代码执行所花费的时间
    public void spendTime(){
        long start=System.currentTimeMillis();
        this.code();//不确定的，易变的部分
        long end=System.currentTimeMillis();
        System.out.println("花费的时间为："+(end-start));
    }
    public abstract void code();
}

class SbuRem extends  emm{
    @Override
    public void code() {
        for (int i = 2; i <1000 ; i++) {
            boolean isFlag=true;
            for (int j = 2; j <Math.sqrt(i) ; j++) { //计算1000以内的质数
                if(i%j==0){
                    isFlag=false;
                    break;
                }
            }
            if(isFlag){
                System.out.println(i);
            }
        }
    }
}
```

模板方法的设计模式。各个框架，类库中都有其影子，比较常见的是

数据库访问的封装

Junit单元测试

JavaWeb的Servlet中关于doGet/doPost方法调用

Hibernate中模板程序

Spring中JDBCTemlate、HibernateTemplate等

## 13.4代理模式（接口 ）

**概述：**

代理模式是Java中使用较多的设计模式，代理设计就是为其他对象提供一种代理以控制这个对象的访问。

```java
//代理模式的举例----静态代理
public class NetWorkTest {
    public static void main(String[] args) {
        Server server=new Server();
        ProxyServer proxyServer=new ProxyServer(server);
        proxyServer.browse();

    }
}

interface NetWork{
    public void browse();
}
//被代理类
class Server implements NetWork{
    @Override
    public void browse() {
        System.out.println("真实的服务器访问网络");
    }
}
//代理类
class ProxyServer implements NetWork{
    private NetWork work;

    public ProxyServer(NetWork work){
        this.work=work;
    }
    public void check(){
        System.out.println("联网之前的检查操作");
    }
    @Override
    public void browse() {
        check();
        work.browse();
    }
}
```

**应用场景：**

安全代理：屏蔽对真实角色的直接访问。
远程代理：通过代理类处理远程方法调用（RMI）
延迟加载：先加载轻量级的代理对象，真正需要再加载真实对象

**分类：**

静态代理（静态定义代理类）
动态代理（动态生成代理类）

## 13.5工厂模式（更新）

### 1.0工厂模式概述

- **简单工厂模式**  
- **工厂方法模式**   
- **抽象工厂模式**

#### 1.1 概念

**==一句话理解：工厂模式主要通过接口和抽象类的方式，实现调用者和生产者的分离，从而达到，在不同条件下创建不同对象的目的==**

**实现了创建者与调用者的分离，即将创建对象的具体过程，屏蔽隔离起来达到提高灵活性的目的。**

**其实设计模式和面向对象设计原则都是为了使得开发项目更加容易扩展和维护，解决方式就是一个“分工”。** 

#### 1.2  分类和本质说明

**简单工厂模式：用来生产同一等级结构中的任意产品（对于增加新的产品需要修改已有代码）**

**工厂方法模式：用来生产同一等级结构中的固定产品（支持增加任意产品）**

**抽象工厂模式：用来生产不同产品族的全部产品（对于增加新的产品，无能为力，支持增加产品族）**



**工厂模式的核心本质**

==实例化对象，用工厂方法代替new操作==

将选择实现类，创建对象统一管理和控制。从而将调用者和我们的实现类解耦。

### 2.0编码理解

#### 2.1无工厂模式

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

#### 2.2简单工厂模式

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

#### 2.3工厂方法模式

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

### 3.0 抽象工厂模式

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



## 14.6建造者模式（更新）

### 1.0概念理解

==一句话理解：建造者模式就是通过将多个简单对象封装为一个复杂对象的过程并十分重视零件的装配顺序==

**概念：**

​		建造者模式（Builder Pattern）**使用多个简单的对象一步一步构建成一个复杂的对象**。这种类型的设计模式属于**创建型模式**，它提供了**一种创建对象的最佳方式**。

​		**一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。**



**意图：**将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

**主要解决：**主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。

**优点：** 1、建造者独立，易扩展。 2、便于控制细节风险。

**缺点：** 1、产品必须有共同点，范围有限制。 2、如内部变化复杂，会有很多的建造类。

**使用场景：** 1、需要生成的对象具有复杂的内部结构。 2、需要生成的对象内部属性本身相互依赖。

**注意事项：**与工厂模式的区别是：建造者模式更加关注于零件装配的顺序。

### 2.0编码理解

#### 2.1编码说明

​		**本API主要通过快餐店的套餐，通过蔬菜汉堡，鸡肉汉堡，百事可乐，可口可乐，四种商品，以及他们的包装，纸盒或者瓶子，也可以理解为六个简单对象，通过六个简单对象构建出两个复杂的套餐对象。**

- 套餐一：蔬菜汉堡(纸盒)+可口可乐(瓶子)
- 套餐二：鸡肉汉堡(纸盒)+百事可乐(瓶子)

==同时说明食品包装对象的封装通过抽象类来决定，并已经提前决定好==

==套餐内容由建造者来确定，建造者只需，构建套餐内食物名称即可==

![image-20220825231510997](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825231510997.png)

#### 2.2编码实现

```java
import java.util.ArrayList;
import java.util.List;


/**********************接口准备*************************/
/**
 * 食品名
 * 包装
 * 价格
 */
interface Item {
    public String name();
    public Packing packing();
    public float price();
}


/**
 * 包装：纸盒，瓶子
 */
interface Packing {
    public String pack();
}

/**********************包装准备*************************/
//纸盒
class Wrapper implements Packing {
    public String pack() {
        return "Wrapper";
    }
}
//瓶子
class Bottle implements Packing {
    public String pack() {
        return "Bottle";
    }
}

/**********************食品准备*************************/
/**
 * 食品：
 * 汉堡类：蔬菜汉堡，鸡肉汉堡
 * 冷饮类：可口可乐，百事可乐
 */
//抽象汉堡类
abstract class Burger implements Item {

    //确定包装为纸盒
    public Packing packing() {  
        return new Wrapper();
    }
    public abstract float price();
}

//蔬菜汉堡
class VegBurger extends Burger {

    public float price() {
        return 25.0f;
    }

    public String name() {
        return "Veg Burger";
    }
}

//鸡肉汉堡
class ChickenBurger extends Burger {

    public float price() {
        return 50.5f;
    }

    public String name() {
        return "Chicken Burger";
    }
}

//抽象冷饮类
abstract class ColdDrink implements Item {

    //确定包装为瓶子
    public Packing packing() {
        return new Bottle();
    }
    public abstract float price();
}

//可口可乐
class Coke extends ColdDrink {

    @Override
    public float price() {
        return 30.0f;
    }

    public String name() {
        return "Coke";
    }
}

//百事可乐
class Pepsi extends ColdDrink {


    public float price() {
        return 35.0f;
    }

    public String name() {
        return "Pepsi";
    }
}

/**********************套餐方法准备*************************/

/**
 * 套餐方法：蔬菜套餐，肉食套餐
 */
class Meal {
    private List<Item> items = new ArrayList<Item>();

    //向套餐中添加食品
    public void addItem(Item item){
        items.add(item);
    }

    //展示套餐内容
    public void showItems(){
        for (Item item : items) {
            System.out.print("Item : "+item.name());
            System.out.print(", Packing : "+item.packing().pack());
            System.out.println(", Price : "+item.price());
        }
    }

    //计算套餐总和
    public float getCost(){
        float cost = 0.0f;
        for (Item item : items) {
            cost += item.price();
        }
        return cost;
    }
}

/**********************建造者对象准备*************************/

/**
 * 建造类型：蔬菜套餐，肉食套餐
 */
class MealBuilder {

    public Meal prepareVegMeal (){
        Meal meal = new Meal();       //向套餐中加入一些汉堡和饮料对象
        meal.addItem(new VegBurger());
        meal.addItem(new Coke());
        return meal;
    }

    public Meal prepareNonVegMeal (){
        Meal meal = new Meal();
        meal.addItem(new ChickenBurger());
        meal.addItem(new Pepsi());
        return meal;
    }
}

/**********************测试案例******************************/

public class BuilderPatternDemo {
    public static void main(String[] args) {

        //套餐创造者——>创建一个建造者对象
        MealBuilder mealBuilder = new MealBuilder();

        //调用对象中的蔬菜套餐对象创建方法
        Meal vegMeal = mealBuilder.prepareVegMeal();
        System.out.println("蔬菜套餐");
        //显示套餐项目及费用
        vegMeal.showItems();
        System.out.println("总价: " +vegMeal.getCost());//计算套餐总价

        /**
         * 此时我们在套餐vegMeal中封装了，蔬菜汉堡对象，可口可乐对象，纸盒包装对象，
         * 瓶子包装对象，将四个对象封装在一起就是建造者的核心内容。
         * 我们可以通过各种组合快速完成其他“套餐”的推出
         */


        //非蔬菜套餐
        Meal nonVegMeal = mealBuilder.prepareNonVegMeal();
        System.out.println("\n\n肉食套餐");
        //显示套餐项目及费用
        nonVegMeal.showItems();
        System.out.println("总价: " +nonVegMeal.getCost());
    }
}
```

程序运行结果：

![image-20220825231551207](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825231551207.png)

## 14.7原型模式

### 1.0基本理解

​		==一句话理解：原型模式是为了在后期调用中，减少系统开销，通过事先准备好的对象，使用深拷贝或浅拷贝的方式实现对象的重复使用==

**解释：深拷贝会复制引用对象，浅拷贝只会复制引用地址**

#### 1.1概念理解

​		**原型模式（Prototype Pattern）**是用于**创建重复的对象**，同时**又能保证性能**。这种类型的设计模式属于**创建型模式**，它提供了一种**创建对象的最佳方式。**

​		**这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。**当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以**缓存该对象**，在**下一个请求时返回它的克隆**，在需要的时候更新数据库，**以此来减少数据库调用**。

#### 1.2具体说明

**意图：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。**

**主要解决：在运行期建立和删除原型。**

何时使用： 1、当一个系统应该独立于它的产品创建，构成和表示时。 2、当要实例化的类是在运行时刻指定时，例如，通过动态装载。 3、为了避免创建一个与产品类层次平行的工厂类层次时。 4、当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。

**如何解决：利用已有的一个原型对象，快速地生成和原型对象一样的实例。**

**优点： 1、性能提高。 2、逃避构造函数的约束。**

**缺点： 1、配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。 2、必须实现 Cloneable 接口。**

**使用场景：**

-  1、资源优化场景。
-  2、类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。 
-  3、性能和安全要求的场景。
-  4、通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。
-  5、一个对象多个修改者的场景。
-  6、一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。 
-  7、在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。

==注意事项：与通过对一个类进行实例化来构造新对象不同的是，原型模式是通过拷贝一个现有对象生成新对象的。浅拷贝实现 Cloneable，重写，深拷贝是通过实现 Serializable 读取二进制流。==

### 2.0编码理解

#### 2.1编码说明

​		**我们将创建一个抽象类 *Shape* 和扩展了 *Shape* 类的实体类。下一步是定义类 *ShapeCache*，该类把 shape 对象存储在一个 *Hashtable* 中，并在请求的时候返回它们的克隆。**

![image-20220826093636216](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826093636216.png)

#### 2.2编码过程

```java
import java.util.Hashtable;

/*********************************************************/
/**
 * 克隆抽象类
 */
abstract class Shape implements Cloneable {

    private String id;
    protected String type;

    abstract void draw();

    public String getType(){
        return type;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Object clone() {
        Object clone = null;
        try {
            clone = super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return clone;
    }
}
/*********************************************************/
/**
 * 长方形实现类
 */
class Rectangle extends Shape {

    public Rectangle(){
        type = "Rectangle";
    }


    public void draw() {
        System.out.println("长方形");
    }
}

/**
 *正方形实现类
 */
class Square extends Shape {

    public Square(){
        type = "Square";
    }


    public void draw() {
        System.out.println("正方形");
    }
}


/**
 * 圆形实现类
 */
class Circle extends Shape {

    public Circle(){
        type = "Circle";
    }

    public void draw() {
        System.out.println("圆形");
    }
}

/*********************************************************/
/**
 * 形状缓存
 */
class ShapeCache {

    private static Hashtable<String, Shape> shapeMap = new Hashtable<String, Shape>();

    public static Shape getShape(String shapeId) {
        Shape cachedShape = shapeMap.get(shapeId);
        return (Shape) cachedShape.clone();
    }

    // 对每种形状都运行数据库查询，并创建该形状
    // shapeMap.put(shapeKey, shape);
    // 例如，我们要添加三种形状
    public static void loadCache() {
        Circle circle = new Circle();
        circle.setId("1");
        shapeMap.put(circle.getId(),circle);

        Square square = new Square();
        square.setId("2");
        shapeMap.put(square.getId(),square);

        Rectangle rectangle = new Rectangle();
        rectangle.setId("3");
        shapeMap.put(rectangle.getId(),rectangle);
    }
}
/*********************************************************/
public class PrototypePatternDemo {
    public static void main(String[] args) {

        ShapeCache.loadCache();

        Shape clonedShape = (Shape) ShapeCache.getShape("1");
        System.out.println("Shape : " + clonedShape.getType());

        Shape clonedShape2 = (Shape) ShapeCache.getShape("2");
        System.out.println("Shape : " + clonedShape2.getType());

        Shape clonedShape3 = (Shape) ShapeCache.getShape("3");
        System.out.println("Shape : " + clonedShape3.getType());
    }
}
```

#### 2.3运行结果

![image-20220826095141780](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826095141780.png)



# 15.Java8新特性Lambad表达式

#### 1.匿名函数实现

```java
public class lambda {
    public static void main(String[] args) {

        Cal cal=new Cal(){

            @Override
            public int add(int a,int b){
                return a+b;
            }
        };

        System.out.println(cal.add(1,2));

    }
    interface  Cal{
        int add(int a,int b);
    }
}
```

![image-20230220130125449](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230220130125449.png)

#### 2.最简单的Lambda

```java
public class Main1 {
    public static void main(String[] args) {
        

        Cal c=(int a,int b)->{return a+b;};
        System.out.println(c.add(1,2));

    }
    interface  Cal{
        int add(int a,int b);
    }
}
```

![image-20230220130507557](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230220130507557.png)

#### 3.其他简单实例

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

#### 4.Lambda表达式语法

（参数列表）->{方法体}

![image-20230220131015208](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230220131015208.png)
