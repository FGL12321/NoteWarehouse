# Java建造者设计模式

[TOC]

​		**本文主要通过使用Java相关API案例，以及关于建造者设计模式的概念等，帮助你深入理解本知识点，希望对你有所帮助！**

[点击获取更多资料](https://gitee.com/fanggaolei/learning-notes-warehouse)你的关注是我创作的动力！

# 1.0概念理解

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

# 2.0编码理解

## 2.1编码说明

​		**本API主要通过快餐店的套餐，通过蔬菜汉堡，鸡肉汉堡，百事可乐，可口可乐，四种商品，以及他们的包装，纸盒或者瓶子，也可以理解为六个简单对象，通过六个简单对象构建出两个复杂的套餐对象。**

- 套餐一：蔬菜汉堡(纸盒)+可口可乐(瓶子)
- 套餐二：鸡肉汉堡(纸盒)+百事可乐(瓶子)

==同时说明食品包装对象的封装通过抽象类来决定，并已经提前决定好==

==套餐内容由建造者来确定，建造者只需，构建套餐内食物名称即可==

![image-20220825231510997](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220825231510997.png)

## 2.2编码实现

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