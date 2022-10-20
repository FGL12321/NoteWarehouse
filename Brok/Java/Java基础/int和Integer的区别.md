# int和Integer的区别

[TOC]

​		**如果需要了解int和Integer的区别，那么必须了解Java的两种数据类型，如图所示：**

## 1.0 了解java的两种数据类型

![image-20220826195652998](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826195652998.png)

---

==为什么要有基本数据类型和引用数据类型：==

- 基本数据类型是编程语言内置定义的数据类型，不可简化的，它表示了真实的数字、字符
-  引用数据类型其实就是对基本数据类型的一个封装Java所有引用数据类型都继承于Object类，都是按照Java存储对象的内存模型进行存储，引用是存储在内存栈上的，而对象本身的值存储在内存堆上

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

## 2.0对比int和Integer

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

## 3.0 什么是自动拆箱和自动装箱

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

## 4.0 深入理解Integer

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
