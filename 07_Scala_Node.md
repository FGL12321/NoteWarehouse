[TOC]



# Scala学习笔记

# 第一章：Scala入门（入门）

## 1.1 概述

### 1.1.1 为什么学习 Scala

**1）Spark—新一代内存级大数据计算框架，是大数据的重要内容。**

 **2）Spark就是使用Scala编写的。因此为了更好的学习Spark, 需要掌握Scala这门语言。**

 **3）Spark的兴起，带动Scala语言的发展！**

### 1.1.2 Scala 发展历史

**联邦理工学院的马丁·奥德斯基（Martin Odersky）于2001年开始设计Scala。** 

​		马丁·奥德斯基是编译器及编程的狂热爱好者，长时间的编程之后，希望发明一种 语言，能够让写程序这样的基础工作变得高效，简单。所以当接触到JAVA语言后，对 JAVA这门便携式，运行在网络，且存在垃圾回收的语言产生了极大的兴趣，所以决定 将函数式编程语言的特点融合到JAVA中，由此发明了两种语言==（Pizza & Scala）。==

**Pizza和Scala极大地推动了Java编程语言的发展。** 

⚫ JDK5.0 的泛型、增 强for循 环、自动类型转换等，都是从Pizza引入的新特性。 

⚫ JDK8.0 的类型推断、Lambda表达式就是从Scala引入的特性。 

​		JDK5.0和JDK8.0的编辑器就是马丁·奥德斯基写的，因此马丁·奥德斯基一个人的 战斗力抵得上一个Java开发团队。

### 1.1.3 Scala 和 Java 关系

![1659262031979](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659262031979.png)

### 1.1.4 Scala 语言特点

==Scala是一门以Java虚拟机（JVM）为运行环境并将面向对象和函数式编程的最佳特性结合在一起的 静态类型编程语言（静态语言需要提前编译的如：Java、c、c++等，动态语言如：js）。==

1）Scala是一门多范式的编程语言，Scala支持面向对象和函数式编程。（多范式，就是多种编程方 法的意思。有面向过程、面向对象、泛型、函数式四种程序设计方法。） 

2）Scala源代码（.scala）会被编译成Java字节码（.class），然后运行于JVM之上，并可以调用现有 的Java类库，实现两种语言的无缝对接。 

3）Scala单作为一门语言来看，非常的简洁高效**。** 

4）Scala在设计时，马丁·奥德斯基是参考了Java的设计思想，可以说Scala是源于Java，同时马丁·奥 德斯基也加入了自己的思想，将函数式编程语言的特点融合到JAVA中, 因此，对于学习过Java的同学， 只要在学习Scala的过程中，搞清楚Scala和Java相同点和不同点，就可以快速的掌握Scala这门语言。

## 1.2 Scala 环境搭建

1）安装步骤
（1）首先确保 JDK1.8 安装成功
（2）下载对应的 Scala 安装文件 scala-2.12.11.zip
（3）解压 scala-2.12.11.zip，我这里解压到 D:\Tools
（4）配置 Scala 的环境变量

注意 1：解压路径不能有任何中文路径，最好不要有空格。
注意 2：环境变量要大写 SCALA_HOME

2）测试
需求：计算两数 a 和 b 的和。
步骤
（1）在键盘上同时按 win+r 键，并在运行窗口输入 cmd 命令

（2）输入 Scala 并按回车键，启动 Scala 环境。然后定义两个变量，并计算求和。

![1659273488862](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659273488862.png)

## 1.3 Scala 插件安装

1）插件离线安装步骤 

（1）建议将该插件 scala-intellij-bin-2017.2.6.zip 文件，放到 Scala 的安装目录 

D:\Tools\scala-2.12.11 下，方便管理。 

（2）打开 IDEA，在左上角找到 File->在下拉菜单中点击 Setting... ->点击 Plugins->点击 

右 下 角 

Install plugin from disk… ， 找 到 插 件 存 储 路 径 

D:\Tools\scala-2.12.11\scala-intellij-bin-2017.2.6.zip，最后点击 ok。 

![1659273533850](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659273533850.png)

## 1.4 HelloWorld 案例

### 1.4.1 创建 IDEA 项目工程

1）打开 IDEA->点击左侧的 Flie->选择 New->选择 Project… 

2）创建一个 Maven 工程，并点击 next

3）GroupId 输入 com.atguigu->ArtifactId 输入 scala->点击 next->点击 Finish

4）指定项目工作目录空间

5）默认下，Maven 不支持 Scala 的开发，需要引入 Scala 框架。 

​		在 scala0513 项目上，点击右键-> Add Framework Support... ->选择 Scala->点击 OK

6）创建项目的源文件目录
		右键点击 main 目录->New->点击 Diretory -> 写个名字（比如 scala）。
		右键点击 scala 目录->Mark Directory as->选择 Sources root，观察文件夹颜色发生变化。

```scala
/**
 * object:关键字，声明一个单例对象（伴生对象）
 */
object HelloWorld {

  /**
   * 实现main方法：从外部可以直接调用执行的方法
   * def 方法名(参数名称:参数类型):方法返回值类型={方法体}
   * @param args
   */
  def main(args: Array[String]): Unit = {
          println("hello world")
    System.out.println("safsdfdasfasdf")
  }
}
```



### 1.4.2 class 和 object 说明

Scala完全面向对象，故Scala去掉了Java中非面向对象的元素，如static关键字，void类型 

​	**1）static** 

​	Scala无static关键字，由object实现类似静态方法的功能（类名.方法名）。 

​	**2）void** 

​	对于无返回值的函数，Scala定义其返回值类型为Unit类 class关键字和Java中的class关键字作用相同，用来定义一个类；

# 第二章：变量和数据类型（入门）

## 2.1 注释

![1659273987970](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659273987970.png)

## 2.2 变量和常量（重点）

```
//1.声明变量时，类型可以省略，编译器自动推导，即类型推导
    var a=10;

    //2.类型确定后，就不能修改，说明 Scala 是强数据类型语言。
    age = "tom" // 错误
    //3.变量声明时，必须要有初始值
    //4.在声明/定义一个变量时，可以使用 var 或者 val 来修饰，var 修饰的变量可改变，val 修饰的变量不可改。
     var num1 = 10 // 可变
	 val num2 = 20 // 不可变
    //5.var 修饰的对象引用可以改变，val 修饰的对象则不可改变，但对象的状态（值）
    //却是可以改变的。（比如：自定义对象、数组、集合等等）

    val alice=new Student("alice","20")
```

## 2.3 标识符的命名规范

Scala 对各种变量、方法、函数等命名时使用的字符序列称为标识符。即：凡是自己可 以起名字的地方都叫标识符。 

1）命名规则 

Scala 中的标识符声明，基本和 Java 是一致的，但是细节上会有所变化，有以下三种规 则： 

（1）以字母或者下划线开头，后接字母、数字、下划线

==（2）以操作符开头，且只包含操作符（+ - * / # !等）==

（3）用反引号`....`包括的任意字符串，即使是 Scala 关键字（39 个）也可以 

​		• package, import, class, object, trait, extends, with, type, for 

​		• private, protected, abstract, sealed, final, implicit, lazy, override 

​		• try, catch, finally, throw  

​		• if, else, match, case, do, while, for, return, yield 

​		• def, val, var  

​		• this, super 

​		• new 

​		• true, false, null

## 2.4 字符串输出

```scala
object Test04_String {
  def main(args: Array[String]): Unit = {

    var name: String = "jinlian"
    var age: Int = 18
    //（1）字符串，通过+号连接
    println(name + " " + age)

    //*用于将一个字符串复制多次
    println(name*3)

    //（2）printf 用法字符串，通过%传值。
    printf("name=%s age=%d\n", name, age)

    //（3）字符串，通过$引用
    //多行字符串，在 Scala中，利用三个双引号包围多行字符串就可以实现。
    //输入的内容，带有空格、\t 之类，导致每一行的开始位置不能整洁对齐。
    //应用 scala 的 stripMargin 方法，在 scala 中 stripMargin 默认
    //是“|”作为连接符，//在多行换行的行头前面加一个“|”符号即可。
    val s=
        """
      |select
      |  name,
      |  age
      |from user
      |where name="zhangsan"
      |""".stripMargin
      println(s)

    //如果需要对变量进行运算，那么可以加${}
    val s1 =
      s"""
         |select
         | name,
         | age
         |from user
         |where name="$name" and age=${age+2}
       """.stripMargin
    println(s1)
    val s2 = s"name=$name"
    println(s2)

    val num:Double=2.3456
    println(f"The num is ${num}%2.2f")//格式化模板字符串
  }
}

```

## 2.5 键盘输入

**StdIn.readLine()、StdIn.readShort()、StdIn.readDouble()** 

```java
object Test05_STDIN {
  def main(args: Array[String]): Unit = {
    // 1 输入姓名
    println("input name:")
    var name = StdIn.readLine()

    // 2 输入年龄
    println("input age:")
    var age = StdIn.readShort()

    // 3 输入薪水
    println("input sal:")
    var sal = StdIn.readDouble()

    // 4 打印
    println("name=" + name)
  }
}
```

**文件的输入输出**

```
object Test06_FileIO {

  def main(args: Array[String]): Unit = {
    //1.从文件中读取数据
    Source.fromFile("D:\\haha.txt").foreach(print)

    //2.将数据写入文件
    val writer=new PrintWriter(new File("D:\\haha.txt"))
    writer.write("asdf;sdaklfjsdklfjd;lksafjkl;adsfjk")
    writer.close()
  }
}
```

## 2.6 数据类型（重点）

​		由于Java有基本类型，而且基本类型不是真正意义的对象，即使后面产生了基本类型的包装类，但是仍 

然存在基本数据类型，所以**Java语言并不是真正意思的面向对象**。 

Java基本类型：char、byte、short、int、long、float、double、boolean

Java基本类型的包装类：Character、Byte、Short、Integer、Long、Float、Double、Boolean

![1659319894408](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659319894408.png)

**1）Scala中一切数据都是对象，都是Any的子类。** 

2）Scala中数据类型分为两大类：==数值类型（AnyVal）、引用类型（AnyRef）==，不管是值类型还是引用类型都是对象。

3）Scala数据类型仍然遵守，==低精度的值类型向高精度值类型，自动转换（隐式转换）==

4）Scala中的StringOps是对Java中的String增强

5）Unit：对应Java中的void，用于方法返回值的位置，表示方法没有返回值。==Unit是 一个数据类型，只有一个对象就是()。Void不是数据类型，只是一个关键字==

6）Null是一个类型，只 有一个对 象就 是null。它是所有引用类型（AnyRef）的子类。 

7）Nothing，是所有数据类型的子类，==主要用在一个函数没有明确返回值时使用==，因为这样我们可以把抛出的返回值，返回给任何的变量或者函数。

## 2.7 整数类型（Byte、Short、Int、Long）

1）整型分类

![1659320434580](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659320434580.png)

（1）Scala 各整数类型有固定的表示范围和字段长度，不受具体操作的影响，以保证 Scala 程序的可移植性。 

```scala
object TestDataType {
 def main(args: Array[String]): Unit = {
 // 正确
 var n1:Byte = 127
 var n2:Byte = -128
 // 错误
 // var n3:Byte = 128
 // var n4:Byte = -129
 } 
}
```

（2）Scala 的整型，默认为 Int 型，声明 Long 型，须后加‘l’或‘L’

```scala
object TestDataType {
 def main(args: Array[String]): Unit = {
 var n5 = 10
 println(n5)
 var n6 = 9223372036854775807L
 println(n6)
 } 
}
```

## 2.8 浮点类型（Float、Double）

![1659320619512](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659320619512.png)

Scala 的浮点型常量默认为 Double 型，声明 Float 型常量，须后加‘f’或‘F’。

```scala
object TestDataType {
  def main(args: Array[String]): Unit = {
  // 建议，在开发中需要高精度小数时，请选择 Double
  var n7 = 2.2345678912f
  var n8 = 2.2345678912
  println("n7=" + n7)
  println("n8=" + n8)
 } 
}

n7=2.2345679
n8=2.2345678912
```

## 2.9 字符类型（Char）

1）基本说明
**字符类型可以表示单个字符，字符类型是 Char。 2）案例实操**
（1）字符常量是用单引号 ' ' 括起来的单个字符。

（2）\t ：一个制表位，实现对齐的功能

（3）\n ：换行符

（4）\\ ：表示\ 

（5）\" ：表示"

```scala
object TestCharType {
 def main(args: Array[String]): Unit = {
 //（1）字符常量是用单引号 ' ' 括起来的单个字符。
 var c1: Char = 'a'
 println("c1=" + c1)
//注意：这里涉及自动类型提升，其实编译器可以自定判断是否超出范围，
 //不过 idea 提示报错
 var c2:Char = 'a' + 1
 println(c2)
 
  //（2）\t ：一个制表位，实现对齐的功能
  println("姓名\t 年龄")
     
  //（3）\n ：换行符
  println("西门庆\n 潘金莲")
     
  //（4）\\ ：表示\
  println("c:\\岛国\\avi")
     
  //（5）\" ：表示"
  println("同学们都说：\"大海哥最帅\"")
  } 
}
```

## 2.10 布尔类型：Boolean

1）基本说明
（1）布尔类型也叫 Boolean 类型，Booolean 类型数据只允许取值 true 和 false
（2）boolean 类型占 1 个字节。

```scala
object TestBooleanType {
 def main(args: Array[String]): Unit = {
 
 var isResult : Boolean = false
 var isResult2 : Boolean = true
 } 
}
```



## 2.11 Unit 类型、Null 类型和 Nothing 类型（重点）

![1659320922907](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659320922907.png)

（1）Unit 类型用来标识过程，也就是没有明确返回值的函数。

```scala
object TestSpecialType {
 def main(args: Array[String]): Unit = {
 def sayOk : Unit = {// unit 表示没有返回值，即 void
 
 }
 println(sayOk)
 } 
}
```

（2）Null 类只有一个实例对象，Null 类似于 Java 中的 null 引用。Null 可以赋值给任意引用类型（AnyRef），但是不能赋值给值类型（AnyVal）

```scala
 var cat = new Cat();
 cat = null // 正确
 var n1: Int = null // 错误
 println("n1:" + n1)
```

（3）Nothing，可以作为没有正常返回值的方法的返回类型，非常直观的告诉你这个方法不会正常返回，而且由于 Nothing 是其他任意类型的子类，他还能跟要求返回值的方法兼容。

（3）Nothing，可以作为没有正常返回值的方法的返回类型，非常直观的告诉你这个方法不会正常返回，而且由于 Nothing 是其他任意类型的子类，他还能跟要求返回值的方法兼容。

```scala
object TestSpecialType {
         def main(args: Array[String]): Unit = {
             def test() : Nothing={
             throw new Exception()
             }
         test
     } 
}
```

### 2.12 类型转换

### 2.12.1 数值类型自动转换

当 Scala 程序在进行赋值或者运算时，精度小的类型自动转换为精度大的数值类型，这 个就是自动类型转换（隐式转换）。数据类型按精度（容量）大小排序为： 

![1659321257533](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659321257533.png)

（1）自动提升原则：有多种类型的数据混合运算时，系统首先自动将所有数据转换成 精度大的那种数据类型，然后再进行计算。 

（2）把精度大的数值类型赋值给精度小的数值类型时，就会报错，反之就会进行自动 类型转换。 

（3）（byte，short）和 char 之间不会相互自动转换。 

（4）byte，short，char 他们三者可以计算，在计算时首先转换为 int 类型。

```scala
object TestValueTransfer {
     def main(args: Array[String]): Unit = {
     //（1）自动提升原则：有多种类型的数据混合运算时，系统首先自动将所有数据转换成精度大的那种数值类型，       然后再进行计算。
     var n = 1 + 2.0
     println(n) // n 就是 Double
     //（2）把精度大的数值类型赋值给精度小的数值类型时，就会报错，反之就会进行自动类型转换。
     var n2 : Double= 1.0
     //var n3 : Int = n2 //错误，原因不能把高精度的数据直接赋值和低精度。
     //（3）（byte，short）和 char 之间不会相互自动转换。
     var n4 : Byte = 1
     //var c1 : Char = n4 //错误
     var n5:Int = n4
     //（4）byte，short，char 他们三者可以计算，在计算时首先转换为 int类型。
     var n6 : Byte = 1
     var c2 : Char = 1
     // var n : Short = n6 + c2 //当 n6 + c2 结果类型就是 int
     // var n7 : Short = 10 + 90 //错误
     } 
}
```

### 2.12.2 强制类型转换2

1）基本说明 

  自动类型转换的逆过程，将精度大的数值类型转换为精度小的数值类型。使用时要加上 强制转函数，但可能造成精度降低或溢出，格外要注意。 

（1）将数据由高精度转换为低精度，就需要使用到强制转换 

（2）强转符号只针对于最近的操作数有效，往往会使用小括号提升优先级

```scala
object TestForceTransfer {
     def main(args: Array[String]): Unit = {
     //（1）将数据由高精度转换为低精度，就需要使用到强制转换
     var n1: Int = 2.5.toInt // 这个存在精度损失

     //（2）强转符号只针对于最近的操作数有效，往往会使用小括号提升优先级
     var r1: Int = 10 * 3.5.toInt + 6 * 1.5.toInt // 10 *3 + 6*1 = 36
     var r2: Int = (10 * 3.5 + 6 * 1.5).toInt // 44.0.toInt = 44
     println("r1=" + r1 + " r2=" + r2)
     } 
}
```

### 2.12.3 数值类型和 String 类型间转换

在程序开发中，我们经常需要将基本数值类型转成 String 类型。或者将 String 类型转成 基本数值类型。 

（1）基本类型转 String 类型（语法：将基本类型的值+"" 即可） 

（2）String 类型转基本数值类型（语法：s1.toInt、s1.toFloat、s1.toDouble、s1.toByte、s1.toLong、s1.toShort） 

```scala
object TestStringTransfer {
     def main(args: Array[String]): Unit = {
     //（1）基本类型转 String 类型（语法：将基本类型的值+"" 即可）
     var str1 : String = true + ""
     var str2 : String = 4.5 + ""
     var str3 : String = 100 +""
     //（2）String 类型转基本数值类型（语法：调用相关 API）
     var s1 : String = "12"
     var n1 : Byte = s1.toByte
     var n2 : Short = s1.toShort
     var n3 : Int = s1.toInt
     var n4 : Long = s1.toLong
     } 
}
```

（3）注意事项
		在将 String 类型转成基本数值类型时，要确保 String 类型能够转成有效的数据，比如我们可以把"123"，转成一个整数，但是不能把"hello"转成一个整数。

var n5:Int = "12.6".toInt 会出现 NumberFormatException 异常。

![1659322177363](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322177363.png)

**运行结果：-126**

# 第 三 章 运算符（入门）

## 3.1 算术运算符

![1659322312534](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322312534.png)

## 3.2 关系运算符（比较运算符）

![1659322331722](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322331722.png)

（2）需求 2：Java 和 Scala 中关于==的区别
         Java： ==比较两个变量本身的值，即两个对象在内存中的首地址；
         equals 比较字符串中所包含的内容是否相同。

​         ==Scala：更加类似于 Java 中的 equals，参照 jd 工具==

```scala
def main(args: Array[String]): Unit = {
  val s1 = "abc"
  val s2 = new String("abc")
  println(s1 == s2)
  println(s1.eq(s2))
}

输出结果：
true
false
```

## 3.3 逻辑运算符

![1659322489752](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322489752.png)

## 3.4 赋值运算符

![1659322566336](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322566336.png)

## 3.5 位运算符

![1659322577484](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659322577484.png)

## 3.6 Scala 运算符本质

在 Scala 中其实是没有运算符的，所有运算符都是方法。
	1）当调用对象的方法时，点.可以省略
	2）如果函数参数只有一个，或者没有参数，()可以省略

# 第四章：流程控制（入门）

## 4.1 分支控制 if-else

### 4.1.2 双分支

```
if (条件表达式) {
执行代码块
}
```

```scala
object TestIfElse {
     def main(args: Array[String]): Unit = {

     println("input age:")
     var age = StdIn.readShort()
         if (age < 18){
         println("童年")
         }
     } 
}
```

### 4.1.3 多分支

```scala
 object TestIfElse {
     def main(args: Array[String]): Unit = {
     println("input age:")
     var age = StdIn.readShort()
         if (age < 18){
         println("童年")
         }else{
         println("成年")
         }
 } 
}
```

```scala
object TestIfElse {
     def main(args: Array[String]): Unit = {
     println("input age")
     var age = StdIn.readInt()
         if (age < 18){
         println("童年")
         }else if(age>=18 && age<30){
         println("中年")
         }else{
         println("老年")
         }
     } 
}
```

Scala 中 if else 表达式其实是有返回值的，具体返回值取决于满足条件的代码体的最后一行内容。 

## 4.2 嵌套分支

在一个分支结构中又完整的嵌套了另一个完整的分支结构，里面的分支的结构称为内层。 分支外面的分支结构称为外层分支。嵌套分支不要超过 3 层。 

```scala
object TestIfElse {
 def main(args: Array[String]): Unit = {
 println("input age")
 var age = StdIn.readInt()
     val res :String = if (age < 18){
     "童年"
     }else {
         if(age>=18 && age<30){
         "中年"
         }else{
         "老年"
         }
     }
     println(res)
 } 
}
```

## 4.3 Switch 分支结构

在 Scala 中没有 Switch，而是使用模式匹配来处理。 

模式匹配涉及到的知识点较为综合，因此我们放在后面讲解。

## 4.4 For 循环控制

Scala 也为 for 循环这一常见的控制结构提供了非常多的特性，这些 for 循环的特性被称 为 for 推导式或 for 表达式。 

### 4.4.1 范围数据循环（To）

```
for(i <- 1 to 3){
 print(i + " ")
}
println()
```

（1）i 表示循环的变量，<- 规定 to

 （2）i 将会从 1-3 循环，前后闭合



### 4.4.2 范围数据循环（Until）

```
for(i <- 1 until 3) {
 print(i + " ")
}
println()
```

（1）这种方式和前面的区别在于 i 是从 1 到 3-1 

（2）即使前闭合后开的范围

```
object TestFor {
     def main(args: Array[String]): Unit = {
     for(i <- 1 until 5 + 1){
     println("宋宋，告别海狗人参丸吧" + i)
     }
 } 
}
```

### 4.4.3 循环守卫

```
for(i <- 1 to 3 if i != 2) {
 print(i + " ")
}
println()
```

（1）循环守卫，即循环保护式（也称条件判断式，守卫）。保护式为 true 则进入循环体内部，为 false 则跳过，类似于 continue。

```
object TestFor {
     def main(args: Array[String]): Unit = {
     for (i <- 1 to 5 if i != 3) {
     println(i + "宋宋")
     }
 } 
}
```

### 4.4.4 循环步长

```
for (i <- 1 to 10 by 2) {
println("i=" + i)
}
```

### 4.4.5 嵌套循环

```
for(i <- 1 to 3; j <- 1 to 3) {
 println(" i =" + i + " j = " + j)
}
```

**等价代码：**

```
for (i <- 1 to 3) {
 for (j <- 1 to 3) {
 println("i =" + i + " j=" + j)
 } 
}
```

### 4.4.6 引入变量

```
for(i <- 1 to 3; j = 4 - i) {
 println("i=" + i + " j=" + j)
}
```

（1）for 推导式一行中有多个表达式时，所以要加 ; 来隔断逻辑
（2）for 推导式有一个不成文的约定：当 for 推导式仅包含单一表达式时使用圆括号，
          当包含多个表达式时，一般每行一个表达式，并用花括号代替圆括号，如下

### 4.4.7 循环返回值

```
val res = for(i <- 1 to 10) yield i
println(res)
```

说明：将遍历过程中处理的结果返回到一个新 Vector 集合中，使用 yield 关键字。 

注意：开发中很少使用。 

### 4.4.8 倒序打印

```
for(i <- 1 to 10 reverse){
 println(i)
}
```

## 4.5 While 和 do..While 循环控制

While 和 do..While 的使用和 Java 语言中用法相同。

### 4.5.1 While 循环控制

```
object TestWhile {
 
     def main(args: Array[String]): Unit = {
     var i = 0
         while (i < 10) {
         println("宋宋，喜欢海狗人参丸" + i)
         i += 1
         }
     } 
}
```

### 4.5.2 do..while 循环控制

```
object TestWhile {
 def main(args: Array[String]): Unit = {
     var i = 0
     do {
     println("宋宋，喜欢海狗人参丸" + i)
     i += 1
     } while (i < 10)
 } 
}
```

## 4.6 循环中断

​		Scala 内置控制结构特地去掉了 break 和 continue，是为了更好的适应函数式编程，推荐使用函数式的风格解决break和continue的功能，而不是一个关键字。Scala中使用==breakable==控制结构来实现 break 和 continue 功能。

需求 1：采用异常的方式退出循环

```scala
def main(args: Array[String]): Unit = {
     try {
     for (elem <- 1 to 10) {
     println(elem)
     if (elem == 5) throw new RuntimeException
      }
     }catch {
     case e =>
     }
 println("正常结束循环")
}
```

需求 2：采用 Scala 自带的函数，退出循环

```
import scala.util.control.Breaks
def main(args: Array[String]): Unit = {
 Breaks.breakable(
	 for (elem <- 1 to 10) {
 		println(elem)
		if (elem == 5) Breaks.break()
 	 }
 )
 println("正常结束循环")
}
```

需求 3：对 break 进行省略

```
import scala.util.control.Breaks._
object TestBreak {
 def main(args: Array[String]): Unit = {
 
 breakable {
     for (elem <- 1 to 10) {
      println(elem)
      if (elem == 5) break
     }
 }
 
 println("正常结束循环")
 } 
}
```

需求 4：循环遍历 10 以内的所有数据，奇数打印，偶数跳过（continue）

```
object TestBreak {
 def main(args: Array[String]): Unit = {
     for (elem <- 1 to 10) {
     if (elem % 2 == 1) {
         println(elem)
         } else {
         println("continue")
         }
     }
 } 
}
```

## 4.7 多重循环

```
object TestWhile {
 def main(args: Array[String]): Unit = {
     for (i <- 1 to 9) {
         for (j <- 1 to i) {
         print(j + "*" + i + "=" + (i * j) + "\t")
         }
     println()
     }
 } 
}
```

 

# 第五章：函数式编程（核心）

**1）面向对象编程**
	解决问题，分解对象，行为，属性，然后通过对象的关系以及行为的调用来解决问题。
	对象：用户
	行为：登录、连接 JDBC、读取数据库
	属性：用户名、密码

​    ==Scala 语言是一个完全面向对象编程语言。万物皆对象==

​    ==对象的本质：对数据和行为的一个封装==

**2）函数式编程**
	解决问题时，将问题分解成一个一个的步骤，将每个步骤进行封装（函数），通过调用这些封装好的步骤，解决问题。
	例如：请求->用户名、密码->连接 JDBC->读取数据库
	==Scala 语言是一个完全函数式编程语言。万物皆函数。==

​    ==函数的本质：函数可以当做一个值进行传递==
**3）在 Scala 中函数式编程和面向对象编程完美融合在一起了。**

## 5.1 函数基础

### 5.1.1 函数基本语法

```java
  def main(args: Array[String]): Unit = {

    // （1）函数定义
    def f(arg: String): Unit = {
      println(arg)
    }

    // （2）函数调用
    // 函数名（参数）
    f("hello world")
  }
```

### 5.1.2 函数和方法的区别

1）核心概念
（1）为完成某一功能的程序语句的集合，称为函数。
（2）类中的函数称之方法。

2）案例实操
（1）Scala 语言可以在任何的语法结构中声明任何的语法
（2）函数没有重载和重写的概念；方法可以进行重载和重写
（3）Scala 中函数可以嵌套定义

```
 // (2)方法可以进行重载和重写，程序可以执行
  def main(): Unit = {
  }
  def main(args: Array[String]): Unit = {
    // （1）Scala 语言可以在任何的语法结构中声明任何的语法
    import java.util.Date

    new Date()

    // (2)函数没有重载和重写的概念，程序报错
    def test(): Unit ={
      println("无参，无返回值")
    }
    test()
    def test(name:String):Unit={
      println()
    }

    //（3）Scala 中函数可以嵌套定义
    def test2(): Unit ={
      def test3(name:String):Unit={
        println("函数可以嵌套定义")
      }
    }
  }
```



### 5.1.3 函数定义

1）函数定义
（1）函数 1：无参，无返回值
（2）函数 2：无参，有返回值
（3）函数 3：有参，无返回值
（4）函数 4：有参，有返回值
（5）函数 5：多参，无返回值
（6）函数 6：多参，有返回值

```
def main(args: Array[String]): Unit = {
    // 函数 1：无参，无返回值
    def test1(): Unit ={
      println("无参，无返回值")
    }
    test1()
    // 函数 2：无参，有返回值
    def test2():String={
      return "无参，有返回值"
    }
    println(test2())
    // 函数 3：有参，无返回值
    def test3(s:String):Unit={
      println(s)
    }
    test3("jinlian")
    // 函数 4：有参，有返回值
    def test4(s:String):String={
      return s+"有参，有返回值"
    }
    println(test4("hello "))
    // 函数 5：多参，无返回值
    def test5(name:String, age:Int):Unit={
      println(s"$name, $age")
    }
    test5("dalang",40)
  }
```



### 5.1.4 函数参数

1）案例实操
（1）可变参数
（2）如果参数列表中存在多个参数，那么可变参数一般放置在最后
（3）参数默认值，一般将有默认值的参数放置在参数列表的后面
（4）带名参数

```scala
  object Test05_STDIN {
  def main(args: Array[String]): Unit = {
    // （1）可变参数
    def test( s : String* ): Unit = {
      println(s)
    }
    // 有输入参数：输出 Array
    test("Hello", "Scala")
    // 无输入参数：输出 List()
    test()


    // (2)如果参数列表中存在多个参数，那么可变参数一般放置在最后
    def test2( name : String, s: String* ): Unit = {
      println(name + "," + s)
    }
    test2("jinlian", "dalang")
    
    // (3)参数默认值
    def test3( name : String, age : Int = 30 ): Unit = {
      println(s"$name, $age")
    }
    // 如果参数传递了值，那么会覆盖默认值
    test3("jinlian", 20)
    // 如果参数有默认值，在调用的时候，可以省略这个参数
    test3("dalang")
    // 一般情况下，将有默认值的参数放置在参数列表的后面
    def test4( sex : String = "男", name : String ): Unit = {
      println(s"$name, $sex")
    }
    // Scala 函数中参数传递是，从左到右
    //test4("wusong")
    
    //（4）带名参数
    test4(name="ximenqing")
  }
}

```

### 5.1.5 函数至简原则（重点）

1）至简原则细节 

（1）return 可以省略，Scala 会使用函数体的最后一行代码作为返回值 

（2）如果函数体只有一行代码，可以省略花括号 

（3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略） 

（4）如果有 return，则不能省略返回值类型，必须指定 

（5）如果函数明确声明 unit，那么即使函数体中使用 return 关键字也不起作用 

（6）Scala 如果期望是无返回值类型，可以省略等号 

（7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加 

（8）如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略 

（9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略

```scala
  def main(args: Array[String]): Unit = {
      // （0）函数标准写法
      def f( s : String ): String = {
        return s + " jinlian"
      }
      println(f("Hello"))
      // 至简原则:能省则省
      
      //（1） return 可以省略,Scala 会使用函数体的最后一行代码作为返回值
      def f1( s : String ): String = {
        s + " jinlian"
      }
      println(f1("Hello"))
      
      //（2）如果函数体只有一行代码，可以省略花括号
      def f2(s:String):String = s + " jinlian"
      
      //（3）返回值类型如果能够推断出来，那么可以省略（:和返回值类型一起省略）
      def f3( s : String ) = s + " jinlian"
      println(f3("Hello3"))
      
      
      //（4）如果有 return，则不能省略返回值类型，必须指定。
      def f4() :String = {
        return "ximenqing4"
      }
      println(f4())
      
      
      //（5）如果函数明确声明 unit，那么即使函数体中使用 return 关键字也不起作用
      def f5(): Unit = {
        return "dalang5"
      }
      println(f5())
      
      //（6）Scala 如果期望是无返回值类型,可以省略等号
      // 将无返回值的函数称之为过程
      def f6() {
        "dalang6"
      }
      println(f6())
      
      //（7）如果函数无参，但是声明了参数列表，那么调用时，小括号，可加可不加
      def f7() = "dalang7"
      println(f7())
      println(f7)
      
      //（8）如果函数没有参数列表，那么小括号可以省略,调用时小括号必须省略
      def f8 = "dalang"
      //println(f8())
      println(f8)
      
      //（9）如果不关心名称，只关心逻辑处理，那么函数名（def）可以省略
      def f9 = (x:String)=>{println("wusong")}
      def f10(f:String=>Unit) = {
        f("")
      }
      f10(f9)
      println(f10((x:String)=>{println("wusong")}))
    } 
```



## 5.2 函数高级

### 5.2.1 高阶函数

在 Scala 中，函数是一等公民。怎么体现的呢？
对于一个函数我们可以：定义函数、调用函数

```
object TestFunction {
 def main(args: Array[String]): Unit = {
	 // 调用函数
	 foo()
 }
	 // 定义函数
 	 def foo():Unit = {
 	 println("foo...")
	 } 
}
```

但是其实函数还有更高阶的用法

```
object TestFunction {
     def main(args: Array[String]): Unit = {
     
     //（1）调用 foo 函数，把返回值给变量 f
     //val f = foo()
     val f = foo
     println(f)
     
    //（2）在被调用函数 foo 后面加上 _，相当于把函数 foo 当成一个整体，传递给变量 f1
     val f1 = foo _
     foo()
     f1()

    //（3）如果明确变量类型，那么不使用下划线也可以将函数作为整体传递给变量
    var f2:()=>Int = foo 
     }
     
     def foo():Int = {
     println("foo...")
     } 
}
```

2）函数可以作为参数进行传递

```
def main(args: Array[String]): Unit = {
 
  //（1）定义一个函数，函数参数还是一个函数签名；f 表示函数名称;(Int,Int)表示输入两个 Int 参数；Int    //表示函数返回值
 def f1(f: (Int, Int) => Int): Int = {
   f(2, 4)
 }
 
 // （2）定义一个函数，参数和返回值类型和 f1 的输入参数一致
 def add(a: Int, b: Int): Int = a + b
 
 // （3）将 add 函数作为参数传递给 f1 函数，如果能够推断出来不是调用，_可以省略
 println(f1(add))
println(f1(add _))
//可以传递匿名函数
}
```

3）函数可以作为函数返回值返回

```
def main(args: Array[String]): Unit = {
     def f1() = {
     def f2() = {
    }
    f2 _
    }
    val f = f1()
    // 因为 f1 函数的返回值依然为函数，所以可以变量 f 可以作为函数继续调用
    f()
    // 上面的代码可以简化为
    f1()()
}
```

### 5.2.2 匿名函数

没有名字的函数就是匿名函数。 

(x:Int)=>{函数体} 

x：表示输入参数类型；Int：表示输入参数类型；函数体：表示具体代码逻辑

**传递匿名函数至简原则：** 

（1）参数的类型可以省略，会根据形参进行自动的推导 

（2）类型省略之后，发现只有一个参数，则圆括号可以省略；其他情况：没有数和参 数超过 1 的永远不能省略圆括号。 

（3）匿名函数如果只有一行，则大括号也可以省略 

（4）如果参数只出现一次，则参数省略且后面参数可以用_代替

需求 1：传递的函数有一个参数

需求 2：传递的函数有两个参数

### 5.2.3 高阶函数案例

```
// 1. 函数作为值进行传递
    val f1: Int=>Int = f //参数类型和返回值类型
    val f2 = f _
 
```

函数可以作为值进行传递

函数可作为参数进行纯涤

函数可以作为返回值进行返回

### 5.2.4 函数柯里化&闭包

闭包：函数式编程的标配
1）说明
	闭包：如果一个函数，访问到了它的外部（局部）变量的值，那么这个函数和他所处的环境，称为闭包
    函数柯里化：把一个参数列表的多个参数，变成多个参数列表。

### 5.2.5 递归

1）说明
一个函数/方法在函数/方法体内又调用了本身，我们称之为递归调用

### 5.2.6 控制抽象

1）值调用：把计算后的值传递过去

2）名调用：把代码传递过去

### 5.2.7 惰性加载

1）说明
		当函数返回值被声明为 lazy 时，函数的执行将被推迟，直到我们首次对此取值，该函数才会执行。这种函数我们称之为惰性函数。

```
def main(args: Array[String]): Unit = {
 lazy val res = sum(10, 30)
 println("----------------")
 println("res=" + res)
}

def sum(n1: Int, n2: Int): Int = {
 println("sum 被执行。。。")
 return n1 + n2
}
```

# 第六章：面向对象（核心）

Scala 的面向对象思想和 Java 的面向对象思想和概念是一致的。
Scala 中语法和 Java 不同，补充了更多的功能。

## 6.1 Scala 包

### 6.1.1 包的命名

![1659339412395](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659339412395.png)

### **6.1.2** **包说明（包语句）** 

![1659339423726](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659339423726.png)



1）说明
Scala 有两种包的管理风格，一种方式和 Java 的包管理风格相同，每个源文件一个包（包名和源文件所在路径不要求必须一致），包名用“.”进行分隔以表示包的层级关系，如com.atguigu.scala。另一种风格，通过嵌套的风格表示层级关系，如下

```
package com{
	package atguigu{
		package scala{
		
    } 
  } 
}
```

**第二种风格有以下特点：** 

（1）一个源文件中可以声明多个 package 

（2）子包中的类可以直接访问父包中的内容，而无需导包 

```scala
// 在同一文件中定义不同的包
package aaa{
  package bbb{

    object Test01_Package{
      def main(args: Array[String]): Unit = {
        import com.atguigu.scala.Inner
        println(Inner.in)
      }
    }
  }
}
```

### 6.1.3 包对象

​		在 Scala 中可以为每个包定义一个同名的包对象，定义在包对象中的成员，作为其对 应包下所有 class 和 object 的共享变量，可以被直接访问

```
package object com{
    val shareValue="share"
    def shareMethod()={}
}
```

1）说明

（1）若使用 Java 的包管理风格，则包对象一般定义在其对应包下的 package.scala文件中，包对象名与包名保持一致。

（2）如采用嵌套方式管理包，则包对象可与包定义在同一文件中，但是要保证包对象与包声明在同一作用域中。

### 6.1.4 导包说明

1）和 Java 一样，可以在顶部使用 import 导入，在这个文件中的所有类都可以使用。 

2）局部导入：什么时候使用，什么时候导入。**在其作用范围内都可以使用** 

3）通配符导入：import java.util._ 

4）给类起名：import java.util.{ArrayList=>JL} 

5）导入**相同包的**多个类：import java.util.{HashSet, ArrayList} 

6）屏蔽类：import java.util.{ArrayList =>_,_} 

7）导入包的绝对路径：new _root_.java.util.HashMap 

![1659339940545](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659339940545.png)

## 6.2 类和对象

### 6.2.1 定义类

```scala
package com.atguigu.chapter06

    //（1）Scala 语法中，类并不声明为 public，所有这些类都具有公有可见性（即默认就是 public）
    class Person {
    
    }
    //（2）一个 Scala 源文件可以包含多个类
    class Teacher{
    
    }
```

### 6.2.2 属性

属性是类的一个组成部分
1）基本语法

[修饰符] var|val 属性名称 [：类型] = 属性值

==注：Bean 属性（@BeanPropetry），可以自动生成规范的 setXxx/getXxx 方法==
2）案例实操

```scala
object Test03_Class {
  def main(args: Array[String]): Unit = {
    val student = new Student()
    println(student.age)
    println(student.sex)
    student.sex = "female"
    println(student.sex)
  }
}

class Student {
  // 定义属性
  private var name: String = "alice"
  @BeanProperty
  //默认是public
  var age: Int = _
  var sex: String = _
}
```

## 6.3 封装

​		封装就是把抽象出的数据和对数据的操作封装在一起，数据被保护在内部，程序的其它 部分只有通过被授权的操作（成员方法），才能对数据进行操作

（1）将属性进行私有化 

（2）提供一个公共的 set 方法，用于对属性赋值 

（3）提供一个公共的 get 方法，用于获取属性的值 。Java 封装操作如下

### 6.1.5 访问权限

1）说明 

​		在 Java 中，访问权限分为：public，private，protected 和默认。在 Scala 中，你可以通 过类似的修饰符达到同样的效果。但是使用上有区别。 

（1）Scala 中属性和方法的默认访问权限为 public，但 Scala 中无 public 关键字。 

（2）private 为私有权限，只在类的内部和伴生对象中可用。 

（3）==protected 为受保护权限，Scala 中受保护权限比 Java 中更严格，同类、子类可以访问，同包无法访问。==

（4）==private[包名]增加包访问权限，包名下的其他类也可以使用==

```scala
// 定义一个父类
class Person {
  private var idCard: String = "3523566"
  protected var name: String = "alice"
  var sex: String = "female"
  private[chapter06] var age: Int = 18

  def printInfo(): Unit = {
    println(s"Person: $idCard $name $sex $age")
  }
}

// 定义一个子类
class Worker extends Person {
  override def printInfo(): Unit = {
//    println(idCard)    // error
    name = "bob"
    age = 25
    sex = "male"
    println(s"Worker: $name $sex $age")
  }
}


object Test04_Access {
  def main(args: Array[String]): Unit = {
    // 创建对象
    val person: Person = new Person()
    println(person.age)
    println(person.sex)
    person.printInfo()
    var worker: Worker = new Worker()
    worker.printInfo()
  }
}

运行结果：
18
female
Person: 3523566 alice female 18
Worker: bob male 25
```





### 6.2.3 方法

```scala
// 定义一个类
class Student1() {
  // 定义属性
  var name: String = _
  var age: Int = _

  println("1. 主构造方法被调用")

  // 声明辅助构造方法
  def this(name: String) {
    this()    // 直接调用主构造器
    println("2. 辅助构造方法一被调用")
    this.name = name
    println(s"name: $name age: $age")
  }

  def this(name: String, age: Int){
    this(name)
    println("3. 辅助构造方法二被调用")
    this.age = age
    println(s"name: $name age: $age")
  }

  def Student1(): Unit = {
    println("一般方法被调用")
  }
}
```

### 6.2.4 创建对象

1）基本语法 

val | var 对象名 [：类型] = new 类型()

2）案例实操 

（1）val 修饰对象，不能改变对象的引用（即：内存地址），可以改变对象属性的值。 

（2）var 修饰对象，可以修改对象的引用和修改对象的属性值 

（3）自动推导变量类型不能多态，所以多态需要显示声明

### 6.2.5 构造器

​		和 Java 一样，Scala 构造对象也需要调用构造方法，并且可以有任意多个构造方法。Scala 类的构造器包括：主构造器和辅助构造器

1）基本语法

```
class 类名(形参列表) { // 主构造器
     // 类体
     def this(形参列表) { // 辅助构造器

     }
     def this(形参列表) { //辅助构造器可以有多个...

     } 
}
```

**说明：**

（1）辅助构造器，函数的名称 this，可以有多个，编译器通过参数的个数及类型来区分。 

（2）辅助构造方法不能直接构建对象，必须直接或者间接调用主构造方法。 

（3）==构造器调用其他另外的构造器，要求被调用构造器必须提前声明。== 

```scala
// 定义一个类
class Student1() {
  // 定义属性
  var name: String = _
  var age: Int = _

  println("1. 主构造方法被调用")

  // 声明辅助构造方法
  def this(name: String) {
    this()    // 直接调用主构造器
    println("2. 辅助构造方法一被调用")
    this.name = name
    println(s"name: $name age: $age")
  }

  def this(name: String, age: Int){
    this(name)  //简介调用主构造器
    println("3. 辅助构造方法二被调用")
    this.age = age
    println(s"name: $name age: $age")
  }

  def Student1(): Unit = {
    println("一般方法被调用")
  }
}
```

### 6.2.6 构造器参数

1）说明
Scala 类的主构造器函数的形参包括三种类型：**未用任何修饰、var 修饰、val 修饰**
（1）未用任何修饰符修饰，这个参数就是一个局部变量
（2）var 修饰参数，作为类的成员属性使用，可以修改
（3）val 修饰参数，作为类只读属性使用，不能修改

```scala
// 定义类
// 无参构造器
class Student2 {
  // 单独定义属性
  var name: String = _
  var age: Int = _
}

// 上面定义等价于
class Student3(var name: String, var age: Int)

// 主构造器参数无修饰
class Student4(name: String, age: Int){
  def printInfo(){
    println(s"student4: name = ${name}, age = $age")
  }
}

//等价
//class Student4(_name: String, _age: Int){
//  var name: String = _name
//  var age: Int = _age
//}

class Student5(val name: String, val age: Int) //无法修改值


class Student6(var name: String, var age: Int){
  var school: String = _

  def this(name: String, age: Int, school: String){
    this(name, age)
    this.school = school
  }

  def printInfo(){
    println(s"student6: name = ${name}, age = $age, school = $school")
  }
}
```

## 6.4 继承和多态

**1）基本语法**
class 子类名 extends 父类名 { 类体 } 

（1）子类继承父类的属性和方法
（2）scala 是单继承
**2）案例实操**

（1）子类继承父类的属性和方法
（2）继承的调用顺序：父类构造器->子类构造器

```scala
// 定义一个父类
class Person7() {
  var name: String = _
  var age: Int = _

  println("1. 父类的主构造器调用")

  def this(name: String, age: Int){
    this()
    println("2. 父类的辅助构造器调用")
    this.name = name
    this.age = age
  }

  def printInfo(): Unit = {
    println(s"Person: $name $age")
  }
}

// 定义子类
class Student7(name: String, age: Int) extends Person7(name, age) {
  var stdNo: String = _

  println("3. 子类的主构造器调用")

  def this(name: String, age: Int, stdNo: String){
    this(name, age)
    println("4. 子类的辅助构造器调用")
    this.stdNo = stdNo
  }
  
  //重写
  override def printInfo(): Unit = {
    println(s"Student: $name $age $stdNo")
  }
}
```

![1659401428655](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659401428655.png)

![1659401525693](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659401525693.png)

**Scala 中属性和方法都是动态绑定，而 Java 中只有方法为动态绑定**

Java中属性是静态绑定的

运行时判断绑定方法

**同一个接口有不同的实现**

```scala
object Test07_Inherit {
  def main(args: Array[String]): Unit = {
    val student1: Student7 = new Student7("alice", 18)
    val student2 = new Student7("bob", 20, "std001")

    student1.printInfo()
    student2.printInfo()

    val teacher = new Teacher
    teacher.printInfo()


    def personInfo(person: Person7): Unit = {
      person.printInfo()
    }

    println("=========================")

    val person = new Person7
    personInfo(student1)
    personInfo(teacher)
    personInfo(person)
  }
}
```

## 6.5 抽象类

### 6.5.1 抽象属性和抽象方法

1）基本语法
（1）定义抽象类：abstract class Person{} //通过 abstract 关键字标记抽象类
（2）定义抽象属性：val|var name:String //一个属性没有初始化，就是抽象属性
（3）定义抽象方法：def hello():String //只声明而没有实现的方法，就是抽象方法

```scala
// 定义抽象类
abstract class Person9{
  // 非抽象属性
  var name: String = "person"

  // 抽象属性
  var age: Int

  // 非抽象方法
  def eat(): Unit = {
    println("person eat")
  }

  // 抽象方法
  def sleep(): Unit
}
```

2）继承&重写
（1）如果父类为抽象类，那么子类需要将抽象的属性和方法实现，否则子类也需声明为抽象类

（2）重写非抽象方法需要用 override 修饰，重写抽象方法则可以不加 override。 

（3）子类中调用父类的方法使用 super 关键字

（4）子类对抽象属性进行实现，父类抽象属性可以用 var 修饰；
			子类对非抽象属性重写，父类非抽象属性只支持 val 类型，而不支持 var。
			==因为 var 修饰的为可变变量，子类继承之后就可以直接使用，没有必要重写==

```scala
// 定义具体的实现子类
class Student9 extends Person9 {
  // 实现抽象属性和方法
  var age: Int = 18

  def sleep(): Unit = {
    println("student sleep")
  }

  // 重写非抽象属性和方法
  //  override val name: String = "student"
  name = "student"   //直接进行赋值了

  override def eat(): Unit = {
    super.eat()
    println("student eat")
  }
}
```

### 6.5.2 匿名子类

和 Java 一样，可以通过包含带有定义或重写的代码块的方式创建一个匿名的子类。 

```scala
object Test10_AnnoymousClass {
  def main(args: Array[String]): Unit = {
    val person: Person10 = new Person10 {
      override var name: String = "alice"
      override def eat(): Unit = println("person eat")
    }
      
    println(person.name)
    person.eat()
      
  }
}

// 定义抽象类
abstract class Person10 {
  var name: String
  def eat(): Unit
}
```

## 6.6 单例对象（伴生对象）

​		Scala语言是完全面向对象的语言，所以并**没有静态的操作**（即在Scala中没有静态的概念）。但是为了能够和Java语言交互（因为Java中有静态概念），**就产生了一种特殊的对象来模拟类对象**，该对象为单例对象。**若单例对象名与类名一致，则称该单例对象这个类的伴生对象**，这个类的所有“静态”内容都可以放置在它的伴生对象中声明。



### 6.6.1 单例对象语法

**1）基本语法** 

```
object  Person{ 

val country:String="China" 

} 
```

2）说明 

（1）单例对象采用 **object** 关键字声明 

（2）单例对象对应的类称之为伴生类，**伴生对象的名称应该和伴生类名一致。** 

（3）单例对象中的**属性和方法都可以通过伴生对象名（类名）直接调用访问**。 

```scala
object Test11_Object {
  def main(args: Array[String]): Unit = {
//    val student = new Student11("alice", 18)
//    student.printInfo()

    val student1 = Student11.newStudent("alice", 18)
    student1.printInfo()

    val student2 = Student11.apply("bob", 19)
    student2.printInfo()

    val student3 = Student11("bob", 19) //apply的好处，直接使用，与上面等效
    student3.printInfo()
  }
}


// 定义类
class Student11 private(val name: String, val age: Int){
  def printInfo(){
    println(s"student: name = ${name}, age = $age, school = ${Student11.school}")
  }
}

// 伴生对象
object Student11{
  val school: String = "atguigu"

  // 定义一个类的对象实例的创建方法
  def newStudent(name: String, age: Int): Student11 = new Student11(name, age)

  def apply(name: String, age: Int): Student11 = new Student11(name, age)
}
```

### 6.6.2 apply 方法

1）说明 

（1）通过伴生对象的 apply 方法，实现不使用 new 方法创建对象。 

（2）如果想让主构造器变成私有的，可以在()之前加上 private。 

（3）apply 方法可以重载。 

（4）Scala 中 **obj(arg)**的语句实际是在调用该对象的 **apply** 方法，即 obj.apply(arg)。用 以统一面向对象编程和函数式编程的风格。 

（5）当使用 new 关键字构建对象时，调用的其实是类的构造方法，当直接使用类名构 建对象时，调用的其实时伴生对象的 apply 方法。 

### 6.7 特质（Trait）

​		Scala 语言中，采用特质 trait（特征）来代替接口的概念，也就是说，多个类具有相同 的特质（特征）时，就可以将这个特质（特征）独立出来，采用关键字 trait 声明。



### 6.7.1 特质声明



```scala
// 定义一个父类
class Person13 {
  val name: String = "person"
  var age: Int = 18
  def sayHello(): Unit = {
    println("hello from: " + name)
  }
  def increase(): Unit = {
    println("person increase")
  }
}

// 定义一个特质
trait Young {
  // 声明抽象和非抽象属性
  var age: Int
  val name: String = "young"

  // 声明抽象和非抽象的方法
  def play(): Unit = {
    println(s"young people $name is playing")
  }

  def dating(): Unit
}

class Student13 extends Person13 with Young {
  // 重写冲突的属性
  override val name: String = "student"

  // 实现抽象方法
  def dating(): Unit = println(s"student $name is dating")

  def study(): Unit = println(s"student $name is studying")

  // 重写父类方法
  override def sayHello(): Unit = {
    super.sayHello()
    println(s"hello from: student $name")
  }
}
```



### 6.7.2 特质基本语法

​		一个类具有某种特质（特征），就意味着这个类满足了这个特质（特征）的所有要素， 所以在使用时，也采用了 extends 关键字，如果有多个特质或存在父类，那么需要采用 with 关键字连接。

**1）基本语法：**
没有父类：class 类名 extends 特质 1 with 特质 2 with 特质 3 …
有父类：class 类名 extends 父类 with 特质 1 with 特质 2 with 特质 3…
**2）说明**
（1）类和特质的关系：使用继承的关系。

（2）当一个类去继承特质时，第一个连接词是 extends，后面是 with。 

（3）如果一个类在同时继承特质和父类时，应当把父类写在 extends 后。
**3）案例实操**
（1）特质可以同时拥有抽象方法和具体方法

（2）一个类可以混入（mixin）多个特质

（3）所有的 Java 接口都可以当做 Scala 特质使用

（4）动态混入：可灵活的扩展类的功能

### 6.7.3 特质叠加

由于一个类可以混入（mixin）多个 trait，且 trait 中可以有具体的属性和方法，若混入 的特质中具有相同的方法（方法名，参数列表，返回值均相同），必然会出现继承冲突问题。冲突分为以下两种

![1659403430463](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659403430463.png)

### 6.7.4 特质叠加执行顺序

当一个类混入多个特质的时候，scala 会对所有的特质及其父特质按照一定的顺序进行 排序，而此案例中的 super.describe()调用的实际上是排好序后的下一个特质中的 describe() 方法。，排序规则如下：

![1659403450828](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659403450828.png)

### 6.7.5 特质自身类型

自身类型可实现依赖注入的功能。 

### 6.7.6 特质和抽象类的区别

1.优先使用特质。一个类扩展多个特质是很方便的，但却只能扩展一个抽象类。 

2.如果你需要构造函数参数，使用抽象类。因为抽象类可以定义带参数的构造函数， 而特质不行（有无参构造）

## 6.8 扩展

### 6.8.1 类型检查和转换

1）说明 

（1）obj.isInstanceOf[T]：判断 obj 是不是 T 类型。 

（2）obj.asInstanceOf[T]：将 obj 强转成 T 类型。

（3）classOf 获取对象的类名。 

### 6.8.2 枚举类和应用类

1）说明 

枚举类：需要继承 Enumeration 

应用类：需要继承 App

### 6.8.3 Type 定义新类型

1）说明 

使用 type 关键字可以定义新的数据数据类型名称，本质上就是类型的一个别名



# 第七章：集合（核心）

## 7.1 集合简介

1）Scala 的集合有三大类：**序列 Seq、集 Set、映射 Map**，所有的集合都扩展自 Iterable特质。

2）对于几乎所有的集合类，Scala 都同时提供了可变和不可变的版本，分别位于以下两个包
不可变集合：scala.collection.immutable
可变集合： scala.collection.mutable

3）Scala 不可变集合，就是指该集合对象不可修改，每次修改就会返回一个新对象，而
不会对原对象进行修改。类似于 java 中的 String 对象

4）可变集合，就是这个集合可以直接对原对象进行修改，而不会返回新的对象。类似
于 java 中 StringBuilder 对象

### 7.1.1 不可变集合继承图

![1659344054205](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344054205.png)

1）Set、Map 是 Java 中也有的集合
2）Seq 是 Java 没有的，我们发现 List 归属到 Seq 了，因此这里的 **List 就和 Java 不是同一个**
概念了
3）我们前面的 for 循环有一个 1 to 3，就是 IndexedSeq 下的 Range
4）String 也是属于 IndexedSeq
5）我们发现经典的数据结构比如 Queue 和 Stack 被归属到 LinearSeq(线性序列) 6）大家注意 Scala 中的 Map 体系有一个 SortedMap，说明 Scala 的 Map 可以支持排序
7）IndexedSeq 和 LinearSeq 的区别：
（1）IndexedSeq 是通过索引来查找和定位，因此速度快，比如 String 就是一个索引集合，通过索引即可定位 

（2）**LinearSeq** 是线型的，即有头尾的概念，这种数据结构一般是通过遍历来查找

### 7.1.2 可变集合继承图

![1659344026057](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344026057.png)

## 7.2 数组

### 7.2.1 不可变数组

1）第一种方式定义数组

```
定义：val arr1 = new Array[Int](10)
```

（1）new 是关键字
（2）[Int]是指定可以存放的数据类型，如果希望存放任意数据类型，则指定 Any
（3）(10)，表示数组的大小，确定后就不可以变化

```scala
object Test01_ImmutableArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建数组
    val arr: Array[Int] = new Array[Int](5)
    // 另一种创建方式
    val arr2 = Array(12, 37, 42, 58, 97)
    println(arr)

    // 2. 访问元素
    println(arr(0))
    println(arr(1))
    println(arr(4))
//    println(arr(5))

    arr(0) = 12
    arr(4) = 57
    println(arr(0))
    println(arr(1))
    println(arr(4))

    println("========================")

    // 3. 数组的遍历
    // 1) 普通for循环
    for (i <- 0 until arr.length){ //不包含
      println(arr(i))
    }

    for (i <- arr.indices) println(arr(i))

    println("---------------------")

    // 2) 直接遍历所有元素，增强for循环
    for (elem <- arr2) println(elem)

    println("---------------------")

    // 3) 迭代器
    val iter = arr2.iterator

    while (iter.hasNext)
      println(iter.next())

    println("---------------------")

    // 4) 调用foreach方法
    arr2.foreach( (elem: Int) => println(elem) )

    arr.foreach( println )

    println(arr2.mkString("--"))

    println("========================")
    // 4. 添加元素
    val newArr = arr2.:+(73)
    println(arr2.mkString("--"))
    println(newArr.mkString("--"))

    val newArr2 = newArr.+:(30)
    println(newArr2.mkString("--"))

    val newArr3 = newArr2 :+ 15
    val newArr4 = 19 +: 29 +: newArr3 :+ 26 :+ 73
    println(newArr4.mkString(", "))
  }
}
```

### 7.2.2 可变数组

### 7.2.3 不可变数组与可变数组的转换

1）说明
   arr1.toBuffer //不可变数组转可变数组
   arr2.toArray //可变数组转不可变数组
（1）arr2.toArray 返回结果才是一个不可变数组，arr2 本身没有变化
（2）arr1.toBuffer 返回结果才是一个可变数组，arr1 本身没有变化

```
object Test02_ArrayBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变数组
    val arr1: ArrayBuffer[Int] = new ArrayBuffer[Int]()
    val arr2 = ArrayBuffer(23, 57, 92)

    println(arr1)
    println(arr2)

    // 2. 访问元素
//    println(arr1(0))     // error
    println(arr2(1))
    arr2(1) = 39
    println(arr2(1))

    println("======================")
    // 3. 添加元素
    val newArr1 = arr1 :+ 15
    println(arr1)
    println(newArr1)
    println(arr1 == newArr1)

    val newArr2 = arr1 += 19
    println(arr1)
    println(newArr2)
    println(arr1 == newArr2)
    newArr2 += 13
    println(arr1)

    77 +=: arr1      //添加到前面
    println(arr1)
    println(newArr2)

    arr1.append(36)     //末尾添加
    arr1.prepend(11, 76)//头部添加
    arr1.insert(1, 13, 59) //索引添加
    println(arr1)

    arr1.insertAll(2, newArr1)
    arr1.prependAll(newArr2)

    println(arr1)

    // 4. 删除元素
    arr1.remove(3)  //删除索引
    println(arr1)

    arr1.remove(0, 10)  //连续删除
    println(arr1)

    arr1 -= 13   //单个删除
    println(arr1)

    // 5. 可变数组转换为不可变数组
    val arr: ArrayBuffer[Int] = ArrayBuffer(23, 56, 98)
    val newArr: Array[Int] = arr.toArray
    println(newArr.mkString(", "))
    println(arr)

    // 6. 不可变数组转换为可变数组
    val buffer: mutable.Buffer[Int] = newArr.toBuffer
    println(buffer)
    println(newArr)
  }
}
```

### 7.2.4 多维数组

**1）多维数组定义** 

val arr = Array.ofDim[Double](3,4）

说明：二维数组中有三个一维数组，每个一维数组中有四个元素

```scala
object Test03_MulArray {
  def main(args: Array[String]): Unit = {
    // 1. 创建二维数组
    val array: Array[Array[Int]] = Array.ofDim[Int](2, 3)

    // 2. 访问元素
    array(0)(2) = 19
    array(1)(0) = 25

    println(array.mkString(", "))
    for (i <- 0 until array.length; j <- 0 until array(i).length){
      println(array(i)(j))
    }
    for (i <- array.indices; j <- array(i).indices){
      print(array(i)(j) + "\t")
      if (j == array(i).length - 1) println()
    }

    array.foreach(line => line.foreach(println))

    array.foreach(_.foreach(println))
  }
}
```

## 7.3 列表 List

### 7.3.1 不可变 List

1）说明
（1）List 默认为不可变集合

（2）创建一个 List（数据有顺序，可重复）

（3）遍历 List

（4）List 增加数据

（5）集合间合并：将一个整体拆成一个一个的个体，称为扁平化

（6）取指定数据 

（7）空集合 Nil 

```scala
object Test04_List {
  def main(args: Array[String]): Unit = {
    // 1. 创建一个List
    val list1 = List(23, 65, 87)
    println(list1)

    // 2. 访问和遍历元素
    println(list1(1))

    list1.foreach(println)

    // 3. 添加元素
    val list2 = 10 +: list1
    val list3 = list1 :+ 23
    println(list1)
    println(list2)
    println(list3)

    println("==================")

    val list4 = list2.::(51)
    println(list4)

    val list5 = Nil.::(13)
    println(list5)

    val list6 = 73 :: 32 :: Nil
    val list7 = 17 :: 28 :: 59 :: 16 :: Nil
    println(list7)

    // 4. 合并列表
    val list8 = list6 :: list7
    println(list8)

    val list9 = list6 ::: list7
    println(list9)

    val list10 = list6 ++ list7
    println(list10)

  }
}

```

### 7.3.2 可变 ListBuffer

1）说明 

（1）创建一个可变集合 ListBuffer 

（2）向集合中添加数据

（3）打印集合数据

```scala
object Test05_ListBuffer {
  def main(args: Array[String]): Unit = {
    // 1. 创建可变列表
    val list1: ListBuffer[Int] = new ListBuffer[Int]()
    val list2 = ListBuffer(12, 53, 75)

    println(list1)
    println(list2)

    println("==============")

    // 2. 添加元素
    list1.append(15, 62)
    list2.prepend(20)

    list1.insert(1, 19, 22)

    println(list1)
    println(list2)

    println("==============")

    31 +=: 96 +=: list1 += 25 += 11
    println(list1)

    println("==============")
    // 3. 合并list
    val list3 = list1 ++ list2
    println(list1)
    println(list2)

    println("==============")

    list1 ++=: list2
    println(list1)
    println(list2)

    println("==============")

    // 4. 修改元素
    list2(3) = 30
    list2.update(0, 89)
    println(list2)

    // 5. 删除元素
    list2.remove(2)
    list2 -= 25
    println(list2)
  }
}
```

## 7.4 Set 集合（无序不可重复）

​	默认情况下，Scala 使用的是不可变集合，如果你想使用可变集合，需要引用==scala.collection.mutable.Set 包==

### 7.4.1 不可变 Set

1）说明 

（1）Set 默认是不可变集合，数据无序 

（2）数据不可重复 

（3）遍历集合

```scala
object Test06_ImmutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1 = Set(13, 23, 53, 12, 13, 23, 78)
    println(set1)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 129
    println(set1)
    println(set2)
    println("==================")

    // 3. 合并set
    val set3 = Set(19, 13, 23, 53, 67, 99)
    val set4 = set2 ++ set3
    println(set2)
    println(set3)
    println(set4)

    // 4. 删除元素
    val set5 = set3 - 13
    println(set3)
    println(set5)
  }
}

```

### 7.4.2 可变 mutable.Set

**1）说明**
（1）创建可变集合 mutable.Set

（2）打印集合

（3）集合添加元素

（4）向集合中添加元素，返回一个新的 Set

（5）删除数据

```scala
import scala.collection.mutable

object Test07_MutableSet {
  def main(args: Array[String]): Unit = {
    // 1. 创建set
    val set1: mutable.Set[Int] = mutable.Set(13, 23, 53, 12, 13, 23, 78)
    println(set1)

    println("==================")

    // 2. 添加元素
    val set2 = set1 + 11
    println(set1)
    println(set2)

    set1 += 11
    println(set1)

    val flag1 = set1.add(10)
    println(flag1)
    println(set1)
    val flag2 = set1.add(10)
    println(flag2)
    println(set1)

    println("==================")

    // 3. 删除元素
    set1 -= 11
    println(set1)

    val flag3 = set1.remove(10)
    println(flag3)
    println(set1)
    val flag4 = set1.remove(10)
    println(flag4)
    println(set1)

    println("==================")

    // 4. 合并两个Set

  }
}
```

## 7.5 Map 集合

Scala 中的 Map 和 Java 类似，也是一个散列表，它存储的内容也是键值对（key-value）映射

### 7.5.1 不可变 Map

**1）说明** 

（1）创建不可变集合 Map 

（2）循环打印 

（3）访问数据 

（4）如果 key 不存在，返回 0 

```scala
object Test08_ImmutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: Map[String, Int] = Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)
    println(map1.getClass)

    println("==========================")
    // 2. 遍历元素
    map1.foreach(println)
    map1.foreach( (kv: (String, Int)) => println(kv) )

    println("============================")

    // 3. 取map中所有的key 或者 value
    for (key <- map1.keys){
      println(s"$key ---> ${map1.get(key)}")
    }

    // 4. 访问某一个key的value
    println("a: " + map1.get("a").get)
    println("c: " + map1.get("c"))
    println("c: " + map1.getOrElse("c", 0))

    println(map1("a"))
  }
}
```

### 7.5.2 可变 Map

1）说明
（1）创建可变集合
（2）打印集合
（3）向集合增加数据
（4）删除数据
（5）修改数据

```scala
import scala.collection.mutable

object Test09_MutableMap {
  def main(args: Array[String]): Unit = {
    // 1. 创建map
    val map1: mutable.Map[String, Int] = mutable.Map("a" -> 13, "b" -> 25, "hello" -> 3)
    println(map1)
    println(map1.getClass)

    println("==========================")

    // 2. 添加元素
    map1.put("c", 5)
    map1.put("d", 9)
    println(map1)

    map1 += (("e", 7))
    println(map1)

    println("====================")

    // 3. 删除元素
    println(map1("c"))
    map1.remove("c")
    println(map1.getOrElse("c", 0))

    map1 -= "d"
    println(map1)

    println("====================")

    // 4. 修改元素
    map1.update("c", 5)
    map1.update("e", 10)
    println(map1)

    println("====================")

    // 5. 合并两个Map
    val map2: Map[String, Int] = Map("aaa" -> 11, "b" -> 29, "hello" -> 5)
//    map1 ++= map2
    println(map1)
    println(map2)

    println("---------------------------")
    val map3: Map[String, Int] = map2 ++ map1
    println(map1)
    println(map2)
    println(map3)
  }
}
```

## 7.6 元组

1）说明
		元组也是可以理解为一个容器，可以存放各种相同或不同类型的数据。说的简单点，就是将多个无关的数据封装为一个整体，称为元组。

==注意：元组中最大只能有 22 个元素==

```scala
object Test10_Tuple {
  def main(args: Array[String]): Unit = {
    // 1. 创建元组
    val tuple: (String, Int, Char, Boolean) = ("hello", 100, 'a', true)
    println(tuple)

    // 2. 访问数据
    println(tuple._1)
    println(tuple._2)
    println(tuple._3)
    println(tuple._4)

    println(tuple.productElement(1))

    println("====================")
    // 3. 遍历元组数据
    for (elem <- tuple.productIterator)
      println(elem)

    // 4. 嵌套元组
    val mulTuple = (12, 0.3, "hello", (23, "scala"), 29)
    println(mulTuple._4._2)
  }
}

```

## 7.7 集合常用函数

### 7.7.1 基本属性和常用操作

（1）获取集合长度 

（2）获取集合大小 

（3）循环遍历 

（4）迭代器 

（5）生成字符串 

（6）是否包含

```scala
object Test11_CommonOp {
  def main(args: Array[String]): Unit = {
    val list = List(1,3,5,7,2,89)
    val set = Set(23,34,423,75)

    //    （1）获取集合长度
    println(list.length)

    //    （2）获取集合大小
    println(set.size)

    //    （3）循环遍历
    for (elem <- list)
      println(elem)

    set.foreach(println)

    //    （4）迭代器
    for (elem <- list.iterator) println(elem)

    println("====================")
    //    （5）生成字符串
    println(list)
    println(set)
    println(list.mkString("--"))

    //    （6）是否包含
    println(list.contains(23))
    println(set.contains(23))
  }
}

```

### 7.7.2 衍生集合

**1）说明** 

（1）获取集合的头

（2）获取集合的尾（不是头的就是尾）
（3）集合最后一个数据
（4）集合初始数据（不包含最后一个）
（5）反转
（6）取前（后）n 个元素
（7）去掉前（后）n 个元素
（8）并集
（9）交集
（10）差集
（11）拉链
（12）滑窗

```scala
object Test12_DerivedCollection {
  def main(args: Array[String]): Unit = {
    val list1 = List(1,3,5,7,2,89)
    val list2 = List(3,7,2,45,4,8,19)

    //    （1）获取集合的头
    println(list1.head)

    //    （2）获取集合的尾（不是头的就是尾）
    println(list1.tail)

    //    （3）集合最后一个数据
    println(list2.last)

    //    （4）集合初始数据（不包含最后一个）
    println(list2.init)

    //    （5）反转
    println(list1.reverse)

    //    （6）取前（后）n个元素
    println(list1.take(3))
    println(list1.takeRight(4))

    //    （7）去掉前（后）n个元素
    println(list1.drop(3))
    println(list1.dropRight(4))

    println("=========================")
    //    （8）并集
    val union = list1.union(list2)
    println("union: " + union)
    println(list1 ::: list2)

    // 如果是set做并集，会去重
    val set1 = Set(1,3,5,7,2,89)
    val set2 = Set(3,7,2,45,4,8,19)

    val union2 = set1.union(set2)
    println("union2: " + union2)
    println(set1 ++ set2)

    println("-----------------------")
    //    （9）交集
    val intersection = list1.intersect(list2)
    println("intersection: " + intersection)

    println("-----------------------")
    //    （10）差集
    val diff1 = list1.diff(list2)
    val diff2 = list2.diff(list1)
    println("diff1: " + diff1)
    println("diff2: " + diff2)

    println("-----------------------")
    //    （11）拉链
    println("zip: " + list1.zip(list2))
    println("zip: " + list2.zip(list1))

    println("-----------------------")
    //    （12）滑窗
    for (elem <- list1.sliding(3))
      println(elem)
    println("-----------------------")

    for (elem <- list2.sliding(4, 2))
      println(elem)

    println("-----------------------")
    for (elem <- list2.sliding(3, 3))
      println(elem)
  }
}
```

### 7.7.3 集合计算简单函数

1）说明
（1）求和
（2）求乘积
（3）最大值
（4）最小值
（5）排序

```scala
object Test13_SimpleFunction {
  def main(args: Array[String]): Unit = {
    val list = List(5,1,8,2,-3,4)
    val list2 = List(("a", 5), ("b", 1), ("c", 8), ("d", 2), ("e", -3), ("f", 4))

    //    （1）求和
    var sum = 0
    for (elem <- list){
      sum += elem
    }
    println(sum)

    println(list.sum)

    //    （2）求乘积
    println(list.product)

    //    （3）最大值
    println(list.max)
    println(list2.maxBy( (tuple: (String, Int)) => tuple._2 ))
    println(list2.maxBy( _._2 ))

    //    （4）最小值
    println(list.min)
    println(list2.minBy(_._2))

    println("========================")

    //    （5）排序
    // 5.1 sorted
    val sortedList = list.sorted
    println(sortedList)

    // 从大到小逆序排序
    println(list.sorted.reverse)
    // 传入隐式参数
    println(list.sorted(Ordering[Int].reverse))

    println(list2.sorted)

    // 5.2 sortBy
    println(list2.sortBy(_._2))
    println(list2.sortBy(_._2)(Ordering[Int].reverse))

    // 5.3 sortWith
    println(list.sortWith( (a: Int, b: Int) => {a < b} ))
    println(list.sortWith( _ < _ ))
    println(list.sortWith( _ > _))
  }
}

```

### 7.7.4 集合计算高级函数

1）说明
（1）过滤 遍历一个集合并从中获取满足指定条件的元素组成一个新的集合
（2）转化/映射（map） 将集合中的每一个元素映射到某一个函数
（3）扁平化
（4）扁平化+映射 注：flatMap 相当于先进行 map 操作，在进行 flatten 操作 集合中的每个元素的子元素映射到       某个函数并返回新集合
（5）分组(group) 按照指定的规则对集合的元素进行分组
（6）简化（归约）

（7）折叠

```scala
object Test14_HighLevelFunction_Map {
  def main(args: Array[String]): Unit = {
    val list = List(1,2,3,4,5,6,7,8,9)

    // 1. 过滤
    // 选取偶数
    val evenList = list.filter( (elem: Int) => {elem % 2 == 0} )
    println(evenList)

    // 选取奇数
    println(list.filter( _ % 2 == 1 ))

    println("=======================")

    // 2. 映射map
    // 把集合中每个数乘2
    println(list.map(_ * 2))
    println(list.map( x => x * x))

    println("=======================")

    // 3. 扁平化
    val nestedList: List[List[Int]] = List(List(1,2,3),List(4,5),List(6,7,8,9))

    val flatList = nestedList(0) ::: nestedList(1) ::: nestedList(2)
    println(flatList)

    val flatList2 = nestedList.flatten
    println(flatList2)

    println("=======================")

    // 4. 扁平映射
    // 将一组字符串进行分词，并保存成单词的列表
    val strings: List[String] = List("hello world", "hello scala", "hello java", "we study")
    val splitList: List[Array[String]] = strings.map( _.split(" ") )    // 分词
    val flattenList = splitList.flatten    // 打散扁平化

    println(flattenList)

    val flatmapList = strings.flatMap(_.split(" "))
    println(flatmapList)

    println("========================")

    // 5. 分组groupBy
    // 分成奇偶两组
    val groupMap: Map[Int, List[Int]] = list.groupBy( _ % 2)
    val groupMap2: Map[String, List[Int]] = list.groupBy( data => if (data % 2 == 0) "偶数" else "奇数")

    println(groupMap)
    println(groupMap2)

    // 给定一组词汇，按照单词的首字母进行分组
    val wordList = List("china", "america", "alice", "canada", "cary", "bob", "japan")
    println( wordList.groupBy( _.charAt(0) ) )
  }
}

```



### 7.7.5 普通 WordCount 案例

1）需求 

单词计数：将集合中出现的相同的单词，进行计数，取计数排名前三的结果 

2）需求分析

![1659408523447](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659408523447.png)

```scala
object Test17_CommonWordCount {
  def main(args: Array[String]): Unit = {
    val stringList: List[String] = List(
      "hello",
      "hello world",
      "hello scala",
      "hello spark from scala",
      "hello flink from scala"
    )

    // 1. 对字符串进行切分，得到一个打散所有单词的列表
//    val wordList1: List[Array[String]] = stringList.map(_.split(" "))
//    val wordList2: List[String] = wordList1.flatten
//    println(wordList2)
    val wordList = stringList.flatMap(_.split(" "))
    println(wordList)

    // 2. 相同的单词进行分组
    val groupMap: Map[String, List[String]] = wordList.groupBy(word => word)
    println(groupMap)

    // 3. 对分组之后的list取长度，得到每个单词的个数
    val countMap: Map[String, Int] = groupMap.map(kv => (kv._1, kv._2.length))

    // 4. 将map转换为list，并排序取前3
    val sortList: List[(String, Int)] = countMap.toList
      .sortWith( _._2 > _._2 )
      .take(3)

    println(sortList)
  }
}

```



### 7.7.6 复杂 WordCount 案例

```scala
object Test18_ComplexWordCount {
  def main(args: Array[String]): Unit = {
    val tupleList: List[(String, Int)] = List(
      ("hello", 1),
      ("hello world", 2),
      ("hello scala", 3),
      ("hello spark from scala", 1),
      ("hello flink from scala", 2)
    )

    // 思路一：直接展开为普通版本
    val newStringList: List[String] = tupleList.map(
      kv => {
        (kv._1.trim + " ") * kv._2
      }
    )
    println(newStringList)

    // 接下来操作与普通版本完全一致
    val wordCountList: List[(String, Int)] = newStringList
      .flatMap(_.split(" "))    // 空格分词
      .groupBy( word => word )     // 按照单词分组
      .map( kv => (kv._1, kv._2.size) )     // 统计出每个单词的个数
      .toList
      .sortBy(_._2)(Ordering[Int].reverse)
      .take(3)

    println(wordCountList)

    println("================================")

    // 思路二：直接基于预统计的结果进行转换
    // 1. 将字符串打散为单词，并结合对应的个数包装成二元组
    val preCountList: List[(String, Int)] = tupleList.flatMap(
      tuple => {
        val strings: Array[String] = tuple._1.split(" ")
        strings.map( word => (word, tuple._2) )
      }
    )
    println(preCountList)

    // 2. 对二元组按照单词进行分组
    val preCountMap: Map[String, List[(String, Int)]] = preCountList.groupBy( _._1 )
    println(preCountMap)

    // 3. 叠加每个单词预统计的个数值
    val countMap: Map[String, Int] = preCountMap.mapValues(
      tupleList => tupleList.map(_._2).sum
    )
    println(countMap)

    // 4. 转换成list，排序取前3
    val countList = countMap.toList
      .sortWith(_._2 > _._2)
      .take(3)
    println(countList)
  }
}

```



## 7.8 队列

​		Scala 也提供了队列（Queue）的数据结构，队列的特点就是先进先出。进队和出队的方 法分别为 enqueue 和 dequeue。 

## 7.9 并行集合

1）说明 

Scala 为了充分使用多核 CPU，提供了并行集合（有别于前面的串行集合），用于多核 环境的并行计算。 

# 第八章：模式匹配（其他特色）

Scala 中的模式匹配类似于 Java 中的 switch 语法

但是 scala 从语法中补充了更多的功能，所以更加强大。 

## 8.1 基本语法

模式匹配语法中，采用 ==match== 关键字声明，每个分支采用 ==case== 关键字进行声明，当需 要匹配时，会从第一个 case 分支开始，如果匹配成功，那么执行对应的逻辑代码，如果匹 配不成功，继续执行下一个分支进行判断。如果所有 case 都不匹配，那么会执行 case _分支， 类似于 Java 中 default 语句。 

```scala
object Test01_PatternMatchBase {
  def main(args: Array[String]): Unit = {
    // 1. 基本定义语法
    val x: Int = 5
    val y: String = x match {
      case 1 => "one"
      case 2 => "two"
      case 3 => "three"
      case _ => "other"
    }
    println(y)

    // 2. 示例：用模式匹配实现简单二元运算
    val a = 25
    val b = 13

    def matchDualOp(op: Char): Int = op match {
      case '+' => a + b
      case '-' => a - b
      case '*' => a * b
      case '/' => a / b
      case '%' => a % b
      case _ => -1
    }

    println(matchDualOp('+'))
    println(matchDualOp('/'))
    println(matchDualOp('\\'))

    println("=========================")

    // 3. 模式守卫
    // 求一个整数的绝对值
    def abs(num: Int): Int = {
      num match {
        case i if i >= 0 => i
        case i if i < 0 => -i
      }
    }

    println(abs(67))
    println(abs(0))
    println(abs(-24))
  }
}

```

1）说明 

（1）如果所有 case 都不匹配，那么会执行 case _ 分支，类似于 Java 中 default 语句， 

若此时没有 case _ 分支，那么会抛出 MatchError。 

（2）每个 case 中，不需要使用 break 语句，自动中断 case。 

（3）match case 语句可以匹配任何类型，而不只是字面量。 

（4）=> 后面的代码块，直到下一个 case 语句之前的代码是**作为一个整体执行**，可以 

使用{}括起来，也可以不括。 

## 8.2 模式守卫

如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫。 

## 8.3 模式匹配类型

### 8.3.1 匹配常量

1）说明 

Scala 中，模式匹配可以匹配所有的字面量，包括字符串，字符，数字，布尔值等等。

```scala
object Test02_MatchTypes {
  def main(args: Array[String]): Unit = {
    // 1. 匹配常量
    def describeConst(x: Any): String = x match {
      case 1 => "Int one"
      case "hello" => "String hello"
      case true => "Boolean true"
      case '+' => "Char +"
      case _ => ""
    }

    println(describeConst("hello"))
    println(describeConst('+'))
    println(describeConst(0.3))

    println("==================================")

    // 2. 匹配类型
    def describeType(x: Any): String = x match {
      case i: Int => "Int " + i
      case s: String => "String " + s
      case list: List[String] => "List " + list
      case array: Array[Int] => "Array[Int] " + array.mkString(",")
      case a => "Something else: " + a
    }

    println(describeType(35))
    println(describeType("hello"))
    println(describeType(List("hi", "hello")))
    println(describeType(List(2, 23)))
    println(describeType(Array("hi", "hello")))
    println(describeType(Array(2, 23)))

    // 3. 匹配数组
    for (arr <- List(
      Array(0),
      Array(1, 0),
      Array(0, 1, 0),
      Array(1, 1, 0),
      Array(2, 3, 7, 15),
      Array("hello", 1, 30),
    )) {
      val result = arr match {
        case Array(0) => "0"
        case Array(1, 0) => "Array(1, 0)"
        case Array(x, y) => "Array: " + x + ", " + y    // 匹配两元素数组
        case Array(0, _*) => "以0开头的数组"
        case Array(x, 1, z) => "中间为1的三元素数组"
        case _ => "something else"
      }

      println(result)
    }

    println("=========================")

    // 4. 匹配列表
    // 方式一
    for (list <- List(
      List(0),
      List(1, 0),
      List(0, 0, 0),
      List(1, 1, 0),
      List(88),
      List("hello")
    )) {
      val result = list match {
        case List(0) => "0"
        case List(x, y) => "List(x, y): " + x + ", " + y
        case List(0, _*) => "List(0, ...)"
        case List(a) => "List(a): " + a
        case _ => "something else"
      }
      println(result)
    }

    // 方式二
    val list1 = List(1, 2, 5, 7, 24)
    val list = List(24)

    list match {
      case first :: second :: rest => println(s"first: $first, second: $second, rest: $rest")
      case _ => println("something else")
    }


    println("===========================")
    // 5. 匹配元组
    for (tuple <- List(
      (0, 1),
      (0, 0),
      (0, 1, 0),
      (0, 1, 1),
      (1, 23, 56),
      ("hello", true, 0.5)
    )){
      val result = tuple match {
        case (a, b) => "" + a + ", " + b
        case (0, _) => "(0, _)"
        case (a, 1, _) => "(a, 1, _) " + a
        case (x, y, z) => "(x, y, z) " + x + " " + y + " " + z
        case _ => "something else"
      }
      println(result)
    }
  }
}

```

### 8.3.2 匹配类型

​			需要进行类型判断时，可以使用前文所学的 isInstanceOf[T]和 asInstanceOf[T]，也可使 用模式匹配实现同样的功能。

### 8.3.3 匹配数组

​		scala 模式匹配可以对集合进行精确的匹配，例如匹配只有两个元素的、且第一个元素 为 0 的数组。

### 8.3.4 匹配列表

### 8.3.5 匹配元组

### 8.3.6 匹配对象及样例类

## 8.4 变量声明中的模式匹配



## 8.5 for 表达式中的模式匹配



## 8.6 偏函数中的模式匹配(了解)

​		偏函数也是函数的一种，通过偏函数我们可以方便的对输入参数做更精确的检查。例如 该偏函数的输入类型为 List[Int]，而我们需要的是第一个元素是 0 的集合，这就是通过模式 匹配实现的。

# 第九章：异常（其他特色）

语法处理上和 **Java** **类似**，但是又不尽相同

## 9.1 Java 异常处理

（1）Java 语言按照 try—catch—finally 的方式来处理异常 

（2）不管有没有异常捕获，都会执行 finally，因此通常可以在 finally 代码块中释放资源

（3）可以有多个 catch，分别捕获对应的异常，这时需要把范围小的异常类写在前面，把范围大的异常类写在后面，否则编译错误。

## 9.2 Scala 异常处理

1）我们将可疑代码封装在 try 块中。在 try 块之后使用了一个 catch 处理程序来捕获异常。如果发生任何异常，catch 处理程序将处理它，程序将不会异常终止。
2）Scala 的异常的工作机制和 Java 一样，但是 Scala 没有“checked（编译期）”异常， 即 Scala 没有编译异常这个概念，异常都是在运行的时候捕获处理。

3）异常捕捉的机制与其他语言中一样，如果有异常发生，catch 子句是按次序捕捉的。因此，在 catch 子句中，越具体的异常越要靠前，越普遍的异常越靠后，如果把越普遍的异常写在前，把具体的异常写在后，在 Scala 中也不会报错，但这样是非常不好的编程风格。

4）finally 子句用于执行不管是正常处理还是有异常发生时都需要执行的步骤，一般用于对象的清理工作，这点和 Java 一样。

5）用 throw 关键字，抛出一个异常对象。所有异常都是 Throwable 的子类型。throw 表达式是有类型的，就是 Nothing，因为 Nothing 是所有类型的子类型，所以 throw 表达式可

6）java 提供了 throws 关键字来声明异常。可以使用方法定义声明异常。它向调用者函 数提供了此方法可能引发此异常的信息。它有助于调用函数处理并将该代码包含在 try-catch 块中，以避免程序异常终止。在 Scala 中，可以使用 throws 注解来声明异常

# 第十章：隐式转换（其他特色）

==当编译器第一次编译失败的时候，会在当前的环境中查找能让代码编译通过的方法，用=于将类型进行转换，实现二次编译==

## 10.1 隐式函数

隐式转换可以在不需改任何代码的情况下，扩展某个类的功能。 

## 10.2 隐式参数

​		普通方法或者函数中的参数可以通过 implicit 关键字声明为隐式参数，调用该方法时，就可以传入该参数，编译器会在相应的作用域寻找符合条件的隐式值。
1）说明
（1）同一个作用域中，相同类型的隐式值只能有一个
（2）编译器按照隐式参数的类型去寻找对应类型的隐式值，与隐式值的名称无关。
（3）隐式参数优先于默认参数



## 10.3 隐式类

在 Scala2.10 后提供了隐式类，可以使用 implicit 声明类，隐式类的非常强大，同样可
以扩展类的功能，在集合中隐式类会发挥重要的作用。
1）隐式类说明

（1）其所带的构造参数有且只能有一个
（2）隐式类必须被定义在“类”或“伴生对象”或“包对象”里，即隐式类不能是顶级的。

## 10.4 隐式解析机制

1）说明
（1）首先会在当前代码作用域下查找隐式实体（隐式方法、隐式类、隐式对象）。（一般是这种情况）
（2）如果第一条规则查找隐式实体失败，会继续在隐式参数的类型的作用域里查找。类型的作用域是指与该类型相关联的全部伴生对象以及该类型所在包的包对象。

# 第十一章：泛型（其他特色）

## 11.1 协变和逆变

1）语法 

class MyList[**+T**]{ //协变 

}  

class MyList[**-T**]{ //逆变 

} 

class MyList[**T**] //不变 

2）说明 

协变：Son 是 Father 的子类，则 MyList[Son] 也作为 MyList[Father]的“子类”。 

逆变：Son 是 Father 的子类，则 MyList[Son]作为 MyList[Father]的“父类”。 

不变：Son 是 Father 的子类，则 MyList[Father]与 MyList[Son]“无父子关系”。

## 11.2 泛型上下限

1）语法
Class PersonList[T <: Person]{ //泛型上限
}
Class PersonList[T >: Person]{ //泛型下限
}

2）说明
泛型的上下限的作用是对传入的泛型进行限定。

## 11.3 上下文限定

1）语法
def f[A : B](a: A） = println(a) //等同于 def f[A](a:A(implicit arg:B[A])=println(a)
2）说明
上下文限定是将泛型和隐式转换的结合产物，以下两者功能相同，使用上下文限定[A : Ordering]之后，方法内无法使用隐式参数名调用隐式参数，需要通过 implicitly[Ordering[A]]获取隐式变量，如果此时无法查找到对应类型的隐式变量，会发生出错误。

# 第十二章：总结（其他特色）

## 12.1 开发环境

要求掌握必要的 Scala 开发环境搭建技能。

## 12.2 变量和数据类型

掌握 var 和 val 的区别

掌握数值类型（Byte、Short、Int、Long、Float、Double、Char）之间的转换关系

## 12.3 流程控制

掌握 if-else、for、while 等必要的流程控制结构，掌握如何实现 break、continue 的功能。

## 12.4 函数式编程

掌握高阶函数、匿名函数、函数柯里化、闭包、函数参数以及函数至简原则。

## 12.5 面向对象

掌握 Scala 与 Java 继承方面的区别、单例对象（伴生对象）、构造方法、特质的用法及功能。

## 12.6 集合

掌握常用集合的使用、集合常用的计算函数。

## 12.7 模式匹配

掌握模式匹配的用法

## 12.8 下划线

掌握下划线不同场合的不同用法

## 12.9 异常

掌握异常常用操作即可

## 12.10 隐式转换

掌握隐式方法、隐式参数、隐式类，以及隐式解析机制

## 12.11 泛型

掌握泛型语法



# 第十三章：IDEA快捷键（其他特色）

![1659344609013](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344609013.png)

![1659344630019](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344630019.png)

![1659344642365](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344642365.png)

![1659344653370](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659344653370.png)