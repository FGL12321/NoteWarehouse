## 第一题：

### 需求：

分析以下需求用代码实现:

1. 定义形状类:

   功能：求面积，求周长

2. 定义圆形类Round:

   属性：半径，圆周率

   功能：求面积，求周长

3. 定义长方形类Rectangle:

		属性：长和宽
	功能：求面积，求周长			

4. 定义测试类:

		分别创建圆形和长方形对象，为相应的属性赋值
	分别调用求面积和求周长的方法



## 第二题：

### 需求：

1. 定义手机类

​	行为：打电话,发短信

2. 定义接口IPlay

​	行为：玩游戏

3. 定义旧手机类继承手机类

​	行为：继承父类的行为

4. 定义新手机继承手机类实现IPlay接口

   行为：继承父类的行为,重写玩游戏方法

5. 定义测试类

​	在测试类中定义一个 用手机的方法,要求该方法既能接收老手机对象,也能接收新手机对象

​	在该方法内部调用打电话,发短信以及新手机特有的玩游戏方法







## 第三题：

### 需求:

1. 接口IPlay中有一个方法playGame()，在测试类中如何调用该方法？

​		要求1.创建子类实现接口的方式实现
		要求2:用匿名内部类实现


2. 一个抽象类Fun中有一个抽象方法 fun() , 在测试类中如何调用该方法?

​		要求1.创建子类继承抽象类的方式实现
		要求2:用匿名内部类实现



## 第四题：

### 需求：

​	在控制台输出”HelloWorld”	

​	自己书写，不要用idea自动生成。

```java
interface Inter {
    void show(); 
}
class Outer { 
    //补齐代码 
}
public class OuterDemo {
    public static void main(String[] args) {
        Outer.method().show();
    }
}

```



## 第五题：

### 需求：

​	在测试类Test中创建A的对象a并调用成员方法methodA(),要求用两种方式实现 

​	自己书写，不要用idea自动生成。

```java
public class Test {
    public static void main(String[] args) {	
        
    }
}

//定义接口
interface InterA {
    void showA();	
}

class A {
    public void methodA(InterA a) {
        a.showA();		
    }	
}
```

## 第六题：

### 需求：

​	在测试类Test中创建B的对象b，并调用成员方法methodB

​	自己书写，不要用idea自动生成。

```java
public class Test {
    public static void main(String[] args) {

    }
}

//定义一个接口
interface InterB {
    void showB();	
}

//目标：调用成员方法methodB
class B {
    InterB b;
    public B(InterB b){
        this.b = b;
    }
    public void methodB(){
        b.showB();		
    }
}
```

