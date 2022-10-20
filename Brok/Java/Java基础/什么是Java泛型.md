# Java泛型的清晰理解

[TOC]

## 1.1泛型的概念

​		**一个泛型方法，该方法在调用时可以接收不同类型的参数。根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用**

- 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的 **<E>**）。
- 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。
- 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
- 泛型方法体的声明和其他方法一样。注意类型参数只能代表引用型类型，不能是原始类型（像 **int、double、char** 等）。

## 1.2泛型方法的符号

- **E** - Element (在集合中使用，因为集合中存放的是元素)
- **T** - Type（Java 类）
- **K** - Key（键）
- **V** - Value（值）
- **N** - Number（数值类型）
- **？** - 表示不确定的 java 类型

## 1. 3 泛型方法-E类型实例

​		这段代码可以很好的解释，==“根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用”==这段话，如果利用普通的方法，在打印不同数据类型时，所调用的方法也会不同，但如果使用泛型，这个问题将被简化。

```java
public class GenericMethodTest
{
   // 泛型方法 printArray                         
   public static < E > void printArray( E[] inputArray )
   {
      // 输出数组元素            
         for ( E element : inputArray ){        
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
 
    public static void main( String args[] )
    {
        // 创建不同类型数组： Integer, Double 和 Character
        Integer[] intArray = { 1, 2, 3, 4, 5 };
        Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 };
        Character[] charArray = { 'H', 'E', 'L', 'L', 'O' };
 
        System.out.println( "整型数组元素为:" );
        printArray( intArray  ); // 传递一个整型数组
 
        System.out.println( "\n双精度型数组元素为:" );
        printArray( doubleArray ); // 传递一个双精度型数组
 
        System.out.println( "\n字符型数组元素为:" );
        printArray( charArray ); // 传递一个字符型数组
    } 
}
```

**运行结果：**

![image-20220826172051457](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826172051457.png)

## 1.4泛型类-T类型实例

​		下面的代码块，通过泛型类，创建了两个不同类型的对象，在调用完成相关的方法时，就能按照对象的返回值类型进行输出，这一点和[Java多态](https://blog.csdn.net/m0_58022371/article/details/126546281)有相似之处。

```java
public class Box<T> {
   
  private T t;
 
  public void add(T t) {
    this.t = t;
  }
 
  public T get() {
    return t;
  }
 
  public static void main(String[] args) {
    Box<Integer> integerBox = new Box<Integer>();//创建一个Integer类型的对象
    Box<String> stringBox = new Box<String>();//创建一个String类型的对象
 
    integerBox.add(new Integer(10));
    stringBox.add(new String("菜鸟教程"));
 
    System.out.printf("整型值为 :%d\n\n", integerBox.get());
    System.out.printf("字符串为 :%s\n", stringBox.get());
  }
}
```

**运行结果：**

![image-20220826173444400](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826173444400.png)

​			

## 1.5类型通配符-?实例

```java
import java.util.*;
 
public class GenericTest {
     
    public static void main(String[] args) {
        List<String> name = new ArrayList<String>();
        List<Integer> age = new ArrayList<Integer>();
        List<Number> number = new ArrayList<Number>();
        
        name.add("icon");
        age.add(18);
        number.add(314);
 
        getData(name);
        getData(age);
        getData(number);
       
   }
 
   public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }
}
```

**运行结果：**

![image-20220826174056847](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220826174056847.png)

​		因为 **getData()** 方法的参数是 **List<?>** 类型的，所以 **name，age，number** 都可以作为这个方法的实参，这就是通配符的作用。

