[TOC]





# 4.0异常处理

## 4.1 关于异常的概述：

异常在Java语言中，将程序执行中发生的不正常情况称为“异常”。

**（开发过程中的语法错误和逻辑错误不是异常）**

**Java程序在执行过程汇总所发生的的异常事件分为两类。**

（1）Error：Java虚拟机无法解决的问题，JVM系统内部错误，资源耗尽等严重情况。

（2）Exception：其他因**编程错误或偶然的外在因素导致的一般性问题**，可以使用针对性的代码进行处理。

> 空指针访问
>
> 试图读去不存在的文件
>
> 网络连接中断
>
> 数组角标越界



**面对异常的处理方式：**

（1）**第一种方法时遇到错误就终止程序的运行。**另一种方法是由程序员**在编写程序时，就考虑到错误的检测、错误消息的提示，以及错误的处理。**

​          捕获错误最理想的状态的是在编译期间，但是有的错误只有在运行时才会发生，比如**除数为0，数组下标越界等问题。**

​           **分类：编译时异常和运行时异常。**



## 4.2**异常概述与异常体系结构**

![1641300673130](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641300673130.png)

**运行时异常：**

**是指编译器不要求强制处置的异常**，一般是指编程时的逻辑错误。是程序员应该积极避免其出现的异常

**Java.lang.RuntimeException**类及它的子 类都是运行时异常

对于这类异常可以不作处理，因为这类异常很普遍，若全处理可能会对程序的可读性和运行效率产生影响。



**编译时异常：**

是指编译器要求必须处置的异常，即程序在运行时由于外界因素造成的一般性异常。**编译器要求Java程序必须捕获或声明所有编译时异常。**

**对于这类异常如果不做处理，可能会带来意想不到的结果。**

**常见异常举例：**

**空指针异常**

```java
public class ErrorTest {
    public static void main(String[] args) {
       test1();
    }
    //Exception in thread "main" Java.lang.NullPointerException: Cannot load from int array because "arr" is null
    //表示空指针异常
    public static void test1() {
        int[] arr = null;
        System.out.println(arr[3]);
    }
}

public class ErrorTest {
    public static void main(String[] args) {
        //Exception in thread "main" Java.lang.StringIndexOutOfBoundsException:
        String string="aasdsadas";
        System.out.println(string.charAt(23));
    }
}

```

**栈异常和堆异常**

```java
public class ErrorTest {
    public static void main(String[] args) {
        //Exception in thread "main" Java.lang.StackOverflowError
        //1.栈溢出
        main(args);
        //Exception in thread "main" Java.lang.OutOfMemoryError: Java heap spaceat yichnag.ErrorTest.main
        //2.堆溢出
        Integer[]arr=new Integer[1024*1024*1024];
    }
}
```

## 4.3异常的处理机制

**（1）Java异常处理概括**

Java采用的异常处理机制，是将异常处理的程序代码集中放在一起，与正常的代码分开，使程序变的简洁优雅易于维护。

**（2）Java提供的是异常处理的抓抛模型**：

Java程序的执行过程中如果出现异常，会产生一个对应异常类的对象，该异常对象将被提交给Java运行时系统，这个过程称为抛出异常（throw）。



**（3）异常对象的生成**：

**由虚拟机自动生成**，程序运行过程中，虚拟机检测到程序发生了问题，**如果在当前代码中没有找到相应的处理程序**，就会在后台自动创建一个对应异常类的实例对象并抛出————自动抛出。一旦抛出对象以后，其后的代码就不再执行。

**由开发人员手动创建：**

```java
Exception exception=new ClassCastException();
```

**创建好的异常对象不抛出对程序没有任何影响，和创建一个普通对象一样。**



**（4）第一种方式：try-catch-finally**---自己解决处理

1.使用try将可能出现异常的代码包装起来，在执行的过程中，一旦出现异常，就会生成一个对应异常类的 对象，根据此对象的类型，去catch中进行匹配。

2.一旦try中的异常对象匹配到某一个catch时，就进入catch中进行异常的处理。一旦处理完成。就跳出当前结构（在没有写finally的情况），继续执行其后的代码

3.finally是可选的。

4.选择异常的时候吧范围小的写上面，范围大的写下面。

5.catch中常用的 方法：

6.在try结构中声明的变量，在出了try结构以后就不能再被调用。

7.使用此方法处理编译时异常，使得程序编译时不再报错，但是运行时仍会报错，相当于将一个编译时可能出现的异常延迟到运行时。



```java
 String getMessage()；//出现主要错误信息

 e.printStackTrace();//出现详细信息
```

**格式**

```java
try{
     //可能出现异常的代码
}catch(异常类型1 变量名1){
    //处理异常的方式
}catch(异常类型2 变量名2){

}

finally{//这个语句是可选的
  //一定会执行的代码
}
```

**举例**

```java
public class test_1 {
    public static void main(String[] args) {
       test1();
    }

    public static void test1(){
        String str="123";
        str="1234";
        try{
            int num=Integer.parseInt(str);
            System.out.println(num);
        }catch(NumberFormatException e){
            System.out.println("出现数值转化异常");
        }
    }
}
```

**finally{}//一定会执行的代码**

```java
public class test_1 {
    public static void main(String[] args) {
       test1();
    }
    public static void test1(){
        String str="123";
        str="1234";
        try{
            int num=Integer.parseInt(str);
            int[] arr=new int[10];
            System.out.println(arr[10]);//出现异常，理论来说应该跳出，但是finally中的代码依然被执行
        }catch(NullPointerException e){
            System.out.println("出现数值转化异常");
        }catch (Exception e){
            System.out.println("出现异常");

        } finally {
            System.out.println("finally中的代码已执行");
        }
    }
}
```

**finally中声明的一定是会被执行的代码，即使catch中又出现异常，try中有return语句，catch中有return语句等**

**什么情况下将代码写入finally中？**

数据库连接 输入输出流 网络编程中的Socket等资源，JVM不能自动回收，我们需要自己进行资源的释放。此时的资源释放一定要声明在finally中

**对于开放当中对于运行时异常不需要用try-catch去处理，因为其本身会自动输出**

**对于编译异常必须进行处理，如果不处理程序就无法运行下去。**



**（5）第二种方式：throws+异常---将异常抛出**  throws+异常类型

1.throws+异常类型 写在方法的声明处，指明方法执行时，可能会抛出的异常类型。一旦当方法体执行时，出现异常，仍会在异常代码处生成一个异常类的对象，此对象满足thows类型就会被抛出。

**2.当此异常被抛出后，异常代码的后续代码不再执行**

```java
public class test_1 {
    public static void main(String[] args) throws Exception{
        test1();
    }
    public static void test1(){
        String str="123";
        str="abcd";
        int num=Integer.parseInt(str);
        System.out.println(num);
    }
}
```

==总结：==

try-catch-finally：**真正将异常处理掉了**

throws的方式并没有将异常处理掉。只是将异常进行了抛出



**（6）子类重写的规则之一**

**子类重写的方法抛出异常类型不大于父类被重写的方法抛出的异常类型。**子类<=父类

**父类方法没有抛出异常，子类就不能抛出异常**

**（7）开发中如何选择这两种处理异常的方式**

​        **如果父类中被重写的方法没有throws方式处理异常，则子类重写的方法不能使用throws，意味着子类重写的方法中有异常，必须使用try-catch方式去处理。**

​        **执行的方法中，先后又调用了另外的几个方法，这几个方法是递进关系执行的，建议几个方法使用throw的方式进行处理。而执行的方法a可以考虑使用try-catch的方式进行处理。**



**（8）手动抛出异常**

手动的生成一个异常对象，并抛出（throw）

```java
public class test_1 {
    public static void main(String[] args) {
        try{
            Student s=new Student();
            s.regist(-1001);
            System.out.println(s);

        }catch (Exception e){
            //e.printStackTrace();
            System.out.println(e.getMessage());
        }
    }
}
class Student{
    private int id;
    public void regist(int id) throws Exception {
        if (id>0){
            this.id=id;
        }else {
            //System.out.println("您输入的数据非法！");
            throw new Exception("您输入的数据非法");
        }
    }
    public String toString(){
        return "Student [id="+id+"]";
    }
}
```

**（9）用户如何自定义异常**                               
1.继承现有的异常结构：RuntimeExceotion，Exception。           
2.提供全局常量 serialVersionUID                         
3.提供重载的构造器                                        

```java
public class test_1 {                                                      
    public static void main(String[] args) {                               
    }                                                                      
}                                                                          
class MyException extends RuntimeException{                                
       static final long serialVersionUID = -7034897190745766939L;         
       public MyException(){                                               
       }                                                                   
       public MyException(String str){                                     
              super(str);                                                  
       }                                                                   
}                                                                          
```

![1641300725469](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641300725469.png)

# 5.0多线程 

## 5.1基本概念理解

**（1）**`程序`：是为了完成特定任务，用某组语言编写的一组指令的集合。即指==一段静态代码==，静态对象。

---

**（2）**`进程 `：是程序一次执行过程，或是**==正在运行的一段程序==**。是一个**==动态的过程==**：有其自身的产生，存在，消亡的过程。`-----生命周期`

- 程序是静态的，进程是动态的。
- **==进程作为资源分配的单位==**，系统在运行时会为每个进程分配不同的内存区域

---

**（3）**`线程`:进程可进一步细化为线程，是一个程序内部的一条执行路径。

- 若一个进程同一时间==并行==执行多个线程，就是支持多线程的。     
- **==线程作为调度和执行的单位，每个线程拥有独立的运行栈和程序计数器(pc)，==**线程切换的开销小。
- 一个进程的多个线程共享相同的内存单元/内存地址空间——》他们从同一个堆中分配对象，可以访问相同的变量和对象。这使得线程间通信更加边界，高效。但多个线程操作共享资源的系统资源可能就会带来==**安全隐患。**==

---

**(4)**`并行`：多CPU同时执行多个任务。

**(5)**`并发`：一个CPU采用时间片同时执行多个任务。

---

**何时需要多线程？**

- 程序需要同时执行两个或多个任务。
- 程序需要实现一些需要等待的任务时，如用户输入，文件读写操作，网络操作，搜索等。
- 需要一些后台运行的程序时。

## 5.2线程的创建使用

**线程的创建**

:large_orange_diamond:**Thread类的介绍**

- Java语言的JVM允许程序运行多个线程，它通过Java.lang.Thread类来体现。

- Thread类的特性

  - 每个线程都是通过某个特定Thread对象的==run()方法来完成操作==的，经常把run()方法的主体称为==线程体==。

  - 通过该Thread对象的==start()方法来启动==这个线程，而非直接调用run()

    - `创建方法介绍`

      - **Thread()**:创建新的Thread对象
      - **Thread(String threadname)**：创建线程并指定线程实例名
      - **Thread(Runnable target)**:指定创建线程的目标对象,它实现了Runnable接口中的run方法
      - **Thread(Runnable target，String name)**:创建新的Thread对象。

    - `使用方法介绍`

      - **void start():** 启动线程，并执行对象的run()方法

      - **run():** 线程在被调度时执行的操作

      - **String getName():** 返回线程的名称

      - **void setName(String name)**:设置该线程名称

      - **static Thread currentThread():** 返回当前线程。

        `在Thread子类中就 是this，通常用于主线程和Runnable实现类`

      - **static void yield()**：线程让步 

        `暂停当前正在执行的线程，把执行机会让给优先级相同或更高的线程 `

        `若队列中没有同优先级的线程，忽略此方法`

      - **join()** **：**当某个程序执行流中调用其他线程的 join() 方法时，调用线程将 被阻塞，直到 join() 方法加入的 join 线程执行完为止 。`低优先级的线程也可以获得执行		`	

      - **static void sleep(long millis)**：(指定时间:毫秒) 

        `令当前线程在指定时间段内放弃对CPU控制,使其他线程有机会被执行,时间到后重排队.`

        `抛出InterruptedException异常`

      - **stop():** 强制线程生命期结束，不推荐使用 

      - **boolean isAlive()**：返回boolean，判断线程是否还活着

```java
/*
测试Thread中常用方法：
1.start():启动当前线程；调用当前线程run();
2.run():通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明写在此方法中
3.currentThread():静态方法，返回当前代码执行的线程。
4.getName()：获取当前线程的名字
5.setName():设置当前线程的名字
6.yield():释放当前CPU的执行权
7.join():在线程A中调用线程B的join()方法，此时线程A阻塞，当线程B执行结束，线程A继续执行。
8.sleep(long millitime)：让当前线程睡眠固定的时间
9.isAlive():判断当前线程是否还存活。
 */

class HelloThread extends Thread{  //ctrl+F12查找当前类中的方法
    public void run(){
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0){
                try {
                    Thread.sleep(100);//1000毫秒=1秒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+":"+i);
            }
//            if (i % 20 == 0) {   //yield()方法测试
//                Thread.yield();
//            }

        }
    }
    public  HelloThread(String name){//给线程命名的方式1
        super(name);
    }
}

public class ThreadMethodTest {
    public static void main(String[] args) {
        HelloThread h1 = new HelloThread("xian");
        //h1.setName("线程一");//给线程命名的方式2
        h1.start();


        //给主线程命名
        Thread.currentThread().setName("主线程");//给线程命名的方式3
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            if (i== 20) {                           //关于join方法的测试
                try {
                    h1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }
        System.out.println(h1.isAlive());//判断当前线程是否存活

    }
}
```



***

**:black_medium_square:线程的调度：**

调度策略：抢占式：高优先级的线程抢占CPU

**:black_small_square:Java的调度算法**

:artificial_satellite:同优先级线程组成先进先出队列(先到先服务）使用时间片策略

:artificial_satellite:对高优先级使用优先调度的抢占式策略

**:black_small_square:线程的优先等级划分：**

:one:MAX_PRIORITY:10

:two:MIN_PRIORITY:1

:three:NORM_PRIORITY:5

==直接定义在start()==

```java
System.out.println(Thread.currentThread().getPriority()+":"+ i);//获取优先级

 h1.setPriority(10);//更改优先级
 h1.start();
```

**:black_small_square:涉及的方法：**

:artificial_satellite:getPriority():返回线程优先值。

:artificial_satellite:setPriority(int newPriority):改变线程的优先级

:red_circle:==优先级高的情况下，并非一定会优先执行==



---



:trident:**继承Thread类**

1. 定义子类继承Thread的类的子类
2. 子类中重写Thread类中的run方法，==将此线程执行的操作声明在方法中==
3. 创建Thread子类对象，即创建线程对象
4. 调用线程对象start方法：启动线程，调用run方法

```java
//1.创建一个继承于Thread类的子类
class MyThread extends Thread{
    //2.重写Thread类中的run()
    public void run(){
        for (int i = 0; i < 100; i++) {
            if (i%2==0){
                System.out.println(Thread.currentThread().getName()+i);
            }
            
        }
    }
}
//
public class ThreadTest {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();//alt+enter  3.创建子类对象
        t1.start();//4.调用start方法     对于这个线程只能调用一次此方法
        //start方法有两个作用，1.启动当前线程   2.调用当前线程的run()方法

        //如下操作在main线程中执行
        for (int i = 0; i < 100; i++) {     //运行结果说明此时有两个线程在执行
            if(i%2==0){
                System.out.println(Thread.currentThread().getName()+i+"***主线程****");
            }

        }
    }
}
```

---

:trident: **实现Runnable接口**    

1. 定义子类实现Runnable接口。
2. 子类中重写Runnable接口中的run方法。
3. 通过Thread类含参构造器创建线程对象。
4. 将Runnable接口的子类对象作为实际参数传递给Thread类的构造器中
5. 调用Thread类的start方法：开启线程，调用Runnable子类接口的run方法。

```java
class Mthrea implements Runnable{

    @Override  //2.实现一下run方法
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i%2==0)
                System.out.println(Thread.currentThread().getName() + ":" +i);//获取线程名
        }

    }
}
public class ThreadTest1 {
    public static void main(String[] args) {
        //3.创建实现类的对象
        Mthrea mthrea = new Mthrea();
        //4.将Runnable接口的子类对象作为实际参数传递给Thread类的构造器中
        Thread thread = new Thread(mthrea);
        //5. 调用Thread类的start方法：开启线程，调用Runnable子类接口的run方法。
        thread.start();//start();方法 （1）启动线程 （2）调用当前线程的run

        //再启动一个线程，遍历100以内的偶数
        Thread thread2 = new Thread(mthrea);
        thread2.setName("第二线程");
        thread2.start();
    }
}

```



---

==:horse_racing:**创建线程的两种方式的比较：**==

:one:开发当中优先选择：实现Runnable接口的方式

​     原因：1.实现的方式没有类的单继承的局限性

​                 2.实现的方式更适合来处理==多个线程有共享数据==的情况

:two:联系：Thread类本身也实现了Runnable类的接口

 相同点：两种方式都需要重写run方法，将线程执行的逻辑声明在run()方法中。

---

:red_circle:**线程的补充：**

`守护线程`

`用户线程`

:black_circle:它们在几乎每个方面都是相同的，唯一的区别是判断JVM何时离开。

:black_circle:守护线程是用来服务用户线程的，通过在start()方法前调用，thread.srtDaemon(true)可以把一个用户线程变成一个守护线程。

:black_circle:Java垃圾回收就是一个典型的守护线程

:black_circle:若JVM中都是守护线程，当前JVM将退出

:black_circle:形象理解：兔死狗烹，鸟尽弓藏

## 5.3线程的生命周期

:red_circle:**==五种状态==**

:one:新建：==Thread类或其子类的对象被声明并创建时==，新生的线程对象处于新建状态

:two:就绪：==线程被start()后==，进入线程队列==等待CPU时间片==，此时它已具备了运行的条件，只是没分配到CPU资源

:three:运行：当就绪的线程被调度并==获得CPU资源时==,便进入运行状态， run()方法定义了线程的操作和功能

:four:阻塞：被人为挂起或执行输入输出操作时，让出==CPU 并临时中 止自己的执行==，进入阻塞状态 

:five:死亡：线程==完成了它的全部工作==或线程被==提前强制性地中止或出现异常导致结束==

---

![1641540597990](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641540597990.png)

![1641540632670](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641540632670.png)

---

## 5.4线程的同步

:red_circle:**概念**

​        即当有==一个线程在对内存进行操作==时，==其他线程都不可以对这个内存地址进行操作==，直到==该线程完成操作， 其他线程才能对该内存地址进行操作==，而其他线程又处于==等待状态==，实现线程同步的方法有很多，临界区对象就是其中一种。 

---

:red_circle:**如何进行线程同步**

**概括：对多条共享数据的语句，当一个线程执行完成后，其他的线程依次执行。**

**:triangular_flag_on_post:同步代码块：**

```java
synchronize(/*同步监视器*/){
   //需要被同步的代码
}
//说明：操作共享数据的代码，即为需要被同步的代码
//同步监视器：俗称“锁”。任何一个类的对象都可与充当锁
//要求：多个线程必须共有同一把锁      

//局限性：操作同步代码时只能有一个线程参与，其他线程等待。相当于一个单线程的过程，效率低。
//补充：在实现Runnable接口创建对象的过程中，可以考虑使用this充当同步监视器。
//在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑使用当前类充当同步监视器。
```

示例：

---

**:triangular_flag_on_post:同步方法：**

如果操作共享的数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。

在方法名上增加一个`synchronized`

总结：

:japanese_goblin:同步方法仍然涉及到同步监视器，只是不需要我们显式的声明

:japanese_goblin:非静态的同步方法，同步监视器是==this==

​     静态的同步方法，同步监视器是==当前类本身==

```java
class Window implements Runnable{ //同步监视器 this
    private  int ticket=100;

    public void run(){
        while (true){
            show();//调用show()方法
    }
    private synchronized void show() {
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName()+ ":卖票,票号为：" + ticket);
                    ticket--;
                }
        }
}
```

```java
/*
* 使用同步方法处理继承Thread类的方式中的线程安全问题
*
* */
public class WindowTest4 {
    public static void main(String[] args) {
        Window t1=new Window();
        Window t2=new Window();
        Window t3=new Window();

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();

    }
}
class Window extends Thread{
    private static int ticket=100;
    //private static final Object obj=new Object();//必须要共有同一把锁
    public void run(){
        while (true){
            show2();
        }
    }

    private static synchronized void show2() {
        //无static同步监视器有3个  改为静态即可 修改后同步监视器为当前类
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName()+ ":卖票,票号为：" + ticket);
            ticket--;
        }
    }
}
```

---

:red_circle:线程的死锁问题

:trident:什么是死锁？

不同的线程分贝占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。

出现死锁后不会出现异常，不会出现提示，只是所有线程都处于阻塞状态，无法继续。

:trident:如何避免死锁;

专门的算法，原则

尽量减少同步资源的定义

尽量避免嵌套同步

```java
/*
*死锁代码块的演示
* 演示线程的死锁问题
*
* */
public class ThreadTest {
    public static void main(String[] args) {
        StringBuffer s1=new StringBuffer();
        StringBuffer s2=new StringBuffer();
        new Thread(){
            //匿名对象
            public void run() {
               synchronized (s1){
                   s1.append("a");
                   s2.append("1");


                   try {//睡眠后容易造成死锁
                       Thread.sleep(100);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }


                   synchronized (s2) {
                     s1.append("b");
                     s2.append("2");

                     System.out.println(s1);
                     System.out.println(s2);
                 }
               }
            }
        }.start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (s2) {
                    s1.append("c");
                    s2.append("3");

                    synchronized (s1) {
                        s1.append("d");
                        s2.append("4");

                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }) .start();

    }
}

```

---

**:red_circle:Lock锁的介绍**

:small_red_triangle_down:从JDK 5.0开始，Java提供了更强大的线程同步机制——==通过显式定义同 步锁对象来实现同步同步锁使用==Lock对象充当。 

:small_red_triangle_down:Java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的 工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象。

:small_red_triangle_down:ReentrantLock 类实现了 Lock ，它==拥有与 synchronized 相同的并发性和 内存语义==，在实现线程安全的控制中，==较常用的是ReentrantLock，可以 显式加锁、释放锁==。

---

**synchronized** **与** **Lock** **的对比** 

1.Lock是==显式锁==（手动开启和关闭锁，别忘记关闭锁），synchronized是隐式锁,==出了作用域自动释放== 

2.**Lock只有代码块锁，synchronized有代码块锁和方法锁** 

3.使用Lock锁，==JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性==（提供更多的子类）

```java
import Java.util.concurrent.locks.ReentrantLock;

/*
* 解决线程安全的方式三：Lock锁---JDK5.0新增
* 
* 
* */
public class LockTest {
    public static void main(String[] args) {
        Window window = new Window();

        Thread thread1 = new Thread(window);
        Thread thread2 = new Thread(window);
        Thread thread3 = new Thread(window);

        thread3.start();
        thread2.start();
        thread1.start();
    }
    
}

class Window implements Runnable{
    private int ticket=100;
    //1.实例化ReentrantLock
    private ReentrantLock lock =new ReentrantLock();

    @Override
    public void run() {
        while (true){

            try{
                //2.调用lock方法
                lock.lock();
                if (ticket>0) {

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName()+":票号为："+ticket);
                    ticket--;
                }else {
                    break;
                }
            }finally {
                //3.调用解锁的方法
                lock.unlock();
            }

        }
    }
}

```

## 5.5线程的通信

:red_circle:基本概念

 线程与线程之间不是相互独立的个体，它们彼此之间需要相互通信和协作。

​       线程是操作==系统调度的最小单位==，有自己的栈空间，可以按照既定的代码逐步的执行，但是如果每个线程间都孤立的运行，那就会造资源浪费。所以在现实中，我们需要这些线程间可以按照指定的规则共同完成一件任务，所以这些==线程之间就需要互相协调，这个过程被称为线程的通信==。 

---

:japanese_goblin:线程通信的三种方式

:one: 共享内存：线程之间共享程序的公共状态，线程之间通过读-写内存中的公共状态来隐式通信。 

:two: 消息传递：线程之间没有公共的状态，线程之间必须通过明确的发送信息来显示的进行通信。 

:three: 管道输入/输出流的形式 

---

:red_circle:线程通信涉及到的方法：

`wait();`:一旦执行此方法，点前线程就进入阻塞状态，并释放同步监视器

`notify();`：一旦执行此方法，就会唤醒被wait的一个线程。如果多个线程被wait，就唤醒优先级高的

`notifyAll();`：一旦执行此方法，就会唤醒所有被wait的方法。



```java
/*
* 线程之间的通信
*使得线程1 线程2 交替打印数字
*wait(); notifyall();都是当前对象调用的
*三个方法的调用者必须是同步代码块或同步方法中的同步监视器否则会出现异常
*因为这三个方法必须有锁对象调用，而任意对象都可以作为synchronized的同步锁，因此这三个方法只能在Object*类中声明。
*三个方法均生命在同步代码块当中
* */
public class CommunicationTest {
    public static void main(String[] args) {
        Number number=new Number();
        Thread t1=new Thread(number);
        Thread t2=new Thread(number);
        t1.setName("线程1");
        t2.setName("线程2");
        t1.start();
        t2.start();
    }
}
class Number implements Runnable{
    private  int number=1;
    @Override
    public void run() {
        while (true){
            synchronized (this){


                notify();//唤醒线程 并释放锁
                if (number<=100){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName()+":"+number);
                    number++;

                    try {
                        wait();//使得调用wait方法的线程处于阻塞状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }else {
                    break;
                }
            }
        }
    }
}

```

---

==sleep()和wait()方法的异同==

1.形同点：`一旦方法执行都可以是的当前线程进入阻塞状态。`
2.不同点：
`两个方法声明的位置不同：	Thread类声明在sleep(),Object类中声明wait()。`
`调用的要求不同：sleep()可以在任何需要的场景下使用。wait()必须在同步代码块中使用。`
`关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中sleep()不会释放锁`

---

### 5.6新增线程的创建方式

:red_circle:**新增线程创建方式一：实现Callable接口**

---

**与使用Runnable相比，Callable功能更强大一些**

:small_orange_diamond:相比run()方法可以有返回值

:small_orange_diamond:方法可以抛出异常

:small_orange_diamond:支持泛型的返回值

需要借助FutureTesk类，比如获取返回结果

```java
/*
* 创建线程的方式三：实现Callable接口------JDK新增5.0
*
* 如何理解实现Callable接口的方式创建多线程，比实现Runnable接口强大。
*1.call()方法是可以有返回值的
*2.call()方法可以抛出异常，被外面的操作捕获，获取异常信息
*3.Callable是支持泛型的
* */

import Java.util.concurrent.Callable;
import Java.util.concurrent.ExecutionException;
import Java.util.concurrent.FutureTask;


public class ThreadNew {
    public static void main(String[] args) {
        //3.创建Callable接口实现类的对象
        NumThread numThread=new NumThread();
        //4.将此Callable接口实现类的对象作为参数传递到FutureTast构造器中，创建FutureTask的对象
        FutureTask task = new FutureTask(numThread);
        //5.将FutureTask的对象作为参数传递道Thread类的构造器中，创建Thread对象，并调用start()
        Thread thread = new Thread(task);
        thread.start();

        Object sum= null;
        try {
            //6.获取Callable中的Callable中call方法的返回值
            //get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值
            sum = task.get();
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("总和为："+sum);
    }
}
//1.创建一个实现Callable接口的实现类
class NumThread implements Callable {
    //2.实现call方法，将此线程需要执行的操作声明在call()方法中。
    @Override
    public Object call() throws Exception {
        int sum=0;
        for (int i = 1; i <=100 ; i++) {
            if(i%2==0){
                System.out.println(i);
                sum +=i;
            }
        }
        return sum;
    }
}
```

**:red_circle:新增线程创建方式二：使用线程池的方式**

---

:small_orange_diamond:**背景：**经常创建和销毁，使用量特别大的资源，比如并发情况下的线程对性能影响很大

:small_orange_diamond:**思路：**提前创建好多个线程，放入线程池汇总，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，         	实现重复利用。类似生活汇总的公共交通工具。

**:small_orange_diamond:好处：**提高相应速度  降低资源消耗  便于线程管理

```java
//创建并使用多线程的第四种方法：使用线程池
class MyThread implements Runnable {

	@Override
	public void run() {
		for (int i = 1; i <= 100; i++) {
			System.out.println(Thread.currentThread().getName() + ":" + i);
		}
	}

}

public class ThreadPool {
	public static void main(String[] args) {
		// 1.调用Executors的newFixedThreadPool(),返回指定线程数量的ExecutorService
		ExecutorService pool = Executors.newFixedThreadPool(10);
		// 2.将Runnable实现类的对象作为形参传递给ExecutorService的submit()方法中，开启线程
		// 并执行相关的run()
		pool.execute(new MyThread());//适用于Callable
		pool.execute(new MyThread());
		pool.execute(new MyThread());
		// 3.结束线程的使用
		pool.shutdown();

	}
}
```

![1641629843278](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641629843278.png)

# 6.0Java的常用类

## 6.1  字符串相关类

**:red_circle: String类**

:black_circle:**Sring特性**

**:one:String类：代表字符串，**Java程序中的所有字符串自勉之(如“abc”)都作为此类的实例实现。

:two:String是一个final类，代表==不可变的字符序列==。

:three:字符串是常量，用双引号引起来表示。他们的值在==创建之后不能更改==。

:four:String对象的字符串内容是存储在==一个字符数组Value[]中==的。

![1641707579217](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641707579217.png)

```
* String的使用
* 1.String声明为final，表示不可继承
* 2.String实现了Serializable接口：表示String可以比较大小
*         实现了Comparable接口：表示String可以比较大小
* 3.String内部定义了final char[] value 用于存储字符串数据
* 4.String：代表不可变的字符序列.简称：不可变性。
     说明：1.当对字符串进行重写赋值的时候，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
          2.当对现有的字符串进行连接操作的时候，也需要重新指定内存区域赋值，不能在原有的value上赋值。
          3.当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值。
* 5.通过/*字面量*/的方式（区别于new）给一个字符串赋值，此时的字符串声明在字符串常量池中。
* 6.字符串常量池中不会存储相同的字符串。
```

![1641708696860](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641708696860.png)

```java
public class StringTest {
    public static void main(String[] args) {
        test();
    }
    public static void test(){
     String s1="abc";//字面量的定义方式
     String s2="abc";
     s1="hello";

        System.out.println(s1 == s2);//比较s1和s2的地址值  比较两个对象的值

        System.out.println(s1);//hello
        System.out.println(s2);//abc


        //对字符串进行连接操作
        String s3="abc";
        s3 +="def";
        
        
        //对字符串进行修改操作
        String s4="abc";
        String s5=s4.replace("a", "m");
        System.out.println(s4);
        System.out.println(s5);
        
    }

}
```

**:black_circle:String的实例化方式**

方式一：通过字面量定义的方式

方式二：通过new+构造器的方式
说明：此时s3 s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值

![1641716477720](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641716477720.png)

**==关于面向对象：==**

![1641717357028](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641717357028.png)

![1641717478457](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641717478457.png)

**:black_circle:关于字符串的拼接操作**

![1641722043291](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641722043291.png)

![1641722258750](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641722258750.png)

:black_circle:字符串中的常用方法

![1641731131839](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641731131839.png)

![1641731153791](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641731153791.png)

![1641731168239](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641731168239.png)

![1641731548006](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641731548006.png)

**:red_circle:StringBuffer & StringBuilder**

**主要概括**

- **==String、StringBuffer、StringBuilder三者的异同？==**
- String：不可变的字符序列 【底层使用char[]存储 final】
- StringBuffer：可变的字符序列；线程安全，效率偏低【底层使用char[]存储】
- StringBulider：可变的字符序列；jdk5.0新增；线程不安全，效率高【底层使用char[]存储 】



- ==源码分析==：
- //1.StringBuffer在初始化时创建了一个长度为16的数组
- StringBuffer sb1=new StringBuffer;//char[] value =new char[16]
- //2.底层放入的字符串+16个空间
- StringBuffer sb2=new StringBuffer("abc");//value=new value char["abc".length()+16];

- **==问题1：System.out.println(sb2.length());==**
- 输出的长度为：0

- **==问题2：扩容问题：如果要添加的数据底层放不下了，那就需要扩容底层数组。==**
- 默认情况下，扩容为原来容量的二倍+2，同时将原有数组中的元素复制到新的数组中。

- **==指导意义：在开发当中建议使用StringBuffer(int capacity)或StringBuilder(int capacity)==**

**常用方法：**

​	StringBuffer append(xxx)：                                   提供了很多的append()方法，用于进行字符串拼接
​	StringBuffer delete(int start,int end)：                   删除指定位置的内容（开始和结束位置）
​	StringBuffer replace(int start, int end, String str)：把[start,end)位置替换为str
​	StringBuffer insert(int offset, xxx)：                       在指定位置插入xxx
​	StringBuffer reverse() ：                                        把当前字符序列逆转

​	public int indexOf(String str)                                
​	public String substring(int start,int end)
​	public int length()
​	public char charAt(int n )
​	public void setCharAt(int n ,char ch)

**三者效率对比:**                                                      

![1641868850711](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641868850711.png)

![1641868971901](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641868971901.png)

**效率从高到低排序：StringBuilder>StringBuffer>String**

## 6.2  JDK8之前的日期时间API 

**:red_circle:System静态方法**

```java
//System类提供的public static long currentTimeMillis()用来返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。
public static void test1(){    
   long time =System.currentTimeMillis();    
   System.out.println(time);
}
```

**:red_circle:Date类**

```java
    //Java.util.Date类
        //----Java.sql.Date类  数据库的中的Date
    /*
    * 1.两个构造器的使用
    * 2.两个方法的使用
    * 3.Java.sql.Date对应着数据库中的日期类型的变量
    * >如何实例化
    * >sql.Date---->util.Date对象
    * >如何将Java.util.Date对象装换为Java.sql.Date对象
    * */
    public static void test2(){
        //构造器一：Date():创建对应当前时间段Date的对象
         Date date = new Date();
        System.out.println(date);  //Tue Jan 11 11:39:27 CST 2022   **方法1**
        System.out.println(date.getTime());       //1641872367156   **方法2**


        //构造器二：创建孩子定毫秒数的Date对象
        Date date2=new Date(1241872367156L); 
        System.out.println(date2);

        //创建Java.sql.Date对象
        Java.sql.Date date3=new Java.sql.Date(1241872367156L);
        System.out.println(date3);//2009-05-09

        //如何将Java.util.Date对象装换为Java.sql.Date对象
        //情况一：
//        Date date4=new Java.sql.Date(1241872387156L);
//        Java.sql.Date date5=(Java.sql.Date) date4;
        //情况二：
        Java.sql.Date date7=new Java.sql.Date(date2.getTime());
    }
}
```

**:red_circle:Calender类**

```java
  public static void testCalendar(){
        //1.实例化
        //方式一：创建其子类(GregorianCalendar)的对象
        //方式二：调用其静态方法getInstance()
        Calendar calendar =Calendar.getInstance();
        //System.out.println(calendar.getCalendarType());

        //2.常用方法
        //get()
        int i=calendar.get(Calendar.DAY_OF_MONTH);//这个月的第几天
        System.out.println(i);
        System.out.println(calendar.get(Calendar.DAY_OF_WEEK));//这一周的第几天

        //set()
        calendar.set(Calendar.DAY_OF_MONTH,22);
        i=calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(i);
        //add()
        calendar.add(Calendar.DAY_OF_MONTH,3);//可以改成负数   相当于减去几天
        i=calendar.get(Calendar.DAY_OF_MONTH);        
        System.out.println(i);
        //getTime()     日历类改为date
        Date time = calendar.getTime();
        System.out.println(time);
        Date date = new Date();
        //setTime();    date改为日历类
        Date date1=new Date();
        calendar.setTime(date1);
        i=calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(i);
        
    }
```

![1641900888989](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641900888989.png)

**:red_circle:SimpleDateFormat类**

```java
 /*
    * SimpleDateFormatd的使用：SimpleDateFormat对日期Date类的格式化解析
    *1.两个操作
    * 1.1格式化：日期---->字符串
    * 1.2解析：  字符串-->日期
    *
    *2.SimpleDateFormat的实例化
    * */
    public static void testSimpleDateFormat() throws ParseException {
        //实例化SimpleDateFormat：使用默认的构造器
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat();

        //格式化：日期--->字符串
        Date date9 =new Date();
        System.out.println(date9);

        String format= simpleDateFormat.format(date9);
        System.out.println(format);

        //1.2解析：  字符串-->日期
        String str="19-2-18 上午11:43";//格式务必正确
        Date date10=simpleDateFormat.parse(str);
        System.out.println(date10);

        //*******按照指定的方式格式化和解析：调用带参数的构造器**********************************
        //SimpleDateFormat simpleDateFormat1 = new SimpleDateFormat("yyyyy.MMMMM.dd GGG hh:mm     aaa");//02019.二月.18 公元 11:43 上午
        SimpleDateFormat simpleDateFormat2 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss.dd ");//2019-02-18 11:43:00.18
        String format1=simpleDateFormat2.format(date10);
        System.out.println(format1);

        //解析:要求字符串必须是符合SimpleDateFormat的格式
        Date parse = simpleDateFormat2.parse("2019-02-18 11:43:00.18 ");
        System.out.println(parse);//Mon Feb 18 11:43:00 CST 2019

    }
```



## 6.3  JDK 8中新日期时间API

![1641949887047](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641949887047.png)

![1641950486799](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641950486799.png)

![1641950598249](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641950598249.png)

**:red_circle:LocalDate、LocalTime、LocalDateTime**

![1641950854584](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641950854584.png)

```java
public class Jdk6TimeTest {
    public static void main(String[] args) {
      test1();
    }
    /*
    * //LocalDate、LocalTime、LocalDateTime的使用
    *
    *
    * */
    public static void test1(){
        //now():获取当前日期，时间，日期+时间
//        LocalDate now = LocalDate.now();
//        LocalTime now1 = LocalTime.now();
//        LocalDateTime now2 = LocalDateTime.now();
//
//        System.out.println(now);
//        System.out.println(now1);
//        System.out.println(now2);


        //of:根据指定日期时间创建对象
        LocalDateTime localDateTime = LocalDateTime.of(2020, 10, 6, 12, 20);
        System.out.println(localDateTime);//2020-10-06T12:20

        //getXxx()  获取一些单独的时间
        System.out.println(localDateTime.getDayOfMonth());//6
        System.out.println(localDateTime.getDayOfWeek());//TUESDAY
        System.out.println(localDateTime.getDayOfYear());//280

        //withXxx操作  设置相关时间 体现不可变性
        LocalDateTime localDateTime1=localDateTime.withDayOfMonth(22);
        System.out.println(localDateTime);//2020-10-06T12:20
        System.out.println(localDateTime1);//2020-10-22T12:20

        LocalDateTime localDateTime2 = localDateTime.withHour(4);
        System.out.println(localDateTime2);//2020-10-06T04:20
        System.out.println(localDateTime);//2020-10-06T12:20

        //对时间的加减操作
        LocalDateTime localDateTime3 = localDateTime.plusMinutes(3);//在现有时间上加三分钟
        System.out.println(localDateTime3);//2020-10-06T12:23

        LocalDateTime localDateTime4 = localDateTime.minusDays(6);//在现有时间上面减去6天
        System.out.println(localDateTime4);//2020-09-30T12:20

    }
}
```

**:red_circle:Instant(瞬时)**

**Instant：时间线上的一个瞬时点。这可能被用来记录应用程序中的时间戳。**

![1641954986964](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1641954986964.png)

![1642036282868](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642036282868.png)

![1642036304986](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642036304986.png)

```java
public class Jdk6TimeTest {
    public static void main(String[] args) {
        test2();
    }
    public static void test2(){
        //now():获取本初子午线对应的时间
        Instant instant =Instant.now();
        System.out.println(instant);

        //添加时间的偏移量
        OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
        System.out.println(offsetDateTime);

        //示自1970年1月1日0时0分0秒（UTC）开始的秒数
        long milli=instant.toEpochMilli();//2022-01-12T10:51:20.314+08:00
        System.out.println(milli);//1641955880314

        //ofEpochMilli通过给定的毫秒数，来获取Instant实例
        Instant instant1 = Instant.ofEpochMilli(1641955880314L);
        System.out.println(instant1);
    }
}
```

**:red_circle:DateTimeFormatter**

![1642036332531](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642036332531.png)



**:red_circle:其他类**

![1642036355453](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642036355453.png)

## 6.4  Java比较器 

**1.Java比较器的使用背景**

Java中的对象，正常情况下，只能进行：==或！=。不能使用>或<的

但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。

如何实现?使用两个接口中的任何一个：Comparable或Comparator

**:red_circle:Comparable接口-自然排序**

1.像String、包装类等实现了Comparable接口，重写了compareTo(obj)方法，==给出了比较两个对象大小的方式。==
2.像String、包装类重写==compareTo()方法以后，进行了从小到大的排列==

3.重写compareTo(obj)的规则：
如果当前对象this大于形参对象obj，则返回正整数，
如果当前对象this小于形参对象obj，则返回负整数，
如果当前对象this等于形参对象obj，则返回零。

4.==对于自定义类来说，如果需要排序，我们可以让自定义类实现Comparable接口，重写compareTo(obj)方法。在compareTo(obj)方法中指明如何排序==

```java
//2.2 自定义类代码举例：
public class Goods implements  Comparable{

    private String name;
    private double price;

    //指明商品比较大小的方式:照价格从低到高排序,再照产品名称从高到低排序
    @Override
    public int compareTo(Object o) {
//        System.out.println("**************");
        if(o instanceof Goods){
            Goods goods = (Goods)o;
            //方式一：
            if(this.price > goods.price){
                return 1;
            }else if(this.price < goods.price){
                return -1;
            }else{
//                return 0;
               return -this.name.compareTo(goods.name);
            }
            //方式二：
//           return Double.compare(this.price,goods.price);
        }
//        return 0;
        throw new RuntimeException("传入的数据类型不一致！");
    }
// getter、setter、toString()、构造器：省略
}

```

**:red_circle:Comparator接口-定制排序**



**1.背景：**
当元素的类型没实现Java.lang.Comparable接口而又不方便修改代码，或者实现了Java.lang.Comparable接口的排序规则不适合当前的操作，那么可以==考虑使用 Comparator 的对象来排序==

**2.重写compare(Object o1,Object o2)方法，比较o1和o2的大小：**
如果方法返回正整数，则表示o1大于o2；
如果返回0，表示相等；
返回负整数，表示o1小于o2。

```java
3.2 代码举例：

Comparator com = new Comparator() {
    //指明商品比较大小的方式:照产品名称从低到高排序,再照价格从高到低排序
    @Override
    public int compare(Object o1, Object o2) {
        if(o1 instanceof Goods && o2 instanceof Goods){
            Goods g1 = (Goods)o1;
            Goods g2 = (Goods)o2;
            if(g1.getName().equals(g2.getName())){
                return -Double.compare(g1.getPrice(),g2.getPrice());
            }else{
                return g1.getName().compareTo(g2.getName());
            }
        }
        throw new RuntimeException("输入的数据类型不一致");
    }
}

使用：
Arrays.sort(goods,com);
Collections.sort(coll,com);
new TreeSet(com);
```

:red_circle:两种排序方式的对比

*    Comparable接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。
*    Comparator接口属于临时性的比较。

## 6.5  System类

:black_circle:System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于Java.lang包。
:black_circle:由于该类的构造器是private的，所以无法创建该类的对象，也就是无法实例化该类。其内部的成:black_circle:员变量和成员方法都是static的，所以也可以很方便的进行调用。
方法：
:timer_clock:native long currentTimeMillis()
:timer_clock:void exit(int status)
:timer_clock:void gc()
:timer_clock:String getProperty(String key)



## 6.6  Math类

**Math类**
Java.lang.Math提供了一系列静态方法用于科学计算。其方法的参数和返回值类型一般为double型。

## 6.7  BigInteger与BigDecimal

BigInteger类、BigDecimal类
**说明：**
① Java.math包的**BigInteger可以表示不可变的任意精度的整数。**
**② 要求数字精度比较高，用到Java.math.BigDecimal类**

**代码举例：**

![1642311737809](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642311737809.png)



# 7.0 枚举类注解

## 7.1枚举的使用

**:red_circle:入门**

![1642312437886](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642312437886.png)

**:red_circle:如何自定义枚举类**

**1.枚举类的理解**

* 1.枚举类的理解：类的对象只有有限个，确定的。我们称此类为枚举类
* 2.当需要定义一组常量时，强烈建议使用枚举类
* 3.如果枚举类中只一个对象，则可以作为单例模式的实现方式。



**2.定义枚举类**

* 方式一：jdk5.0之前，自定义枚举类
* 方式二：jdk5.0，可以使用enum关键字定义枚举类



**:red_circle:如何使用关键字enum定义枚举类**

```java
//使用enum关键字枚举类
enum Season1 {
    //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
    SPRING("春天","春暖花开"),
    SUMMER("夏天","夏日炎炎"),
    AUTUMN("秋天","秋高气爽"),
    WINTER("冬天","冰天雪地");

    //2.声明Season对象的属性:private final修饰
    private final String seasonName;
    private final String seasonDesc;

    //2.私化类的构造器,并给对象属性赋值

    private Season1(String seasonName,String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //4.其他诉求1：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

}
```

**:red_circle:Enum类的主要方法**

```java
4. 使用enum定义枚举类之后，枚举类常用方法：（继承于Java.lang.Enum类）
Season1 summer = Season1.SUMMER;
        //toString():返回枚举类对象的名称
        System.out.println(summer.toString());

//        System.out.println(Season1.class.getSuperclass());
        System.out.println("****************");
        //values():返回所的枚举类对象构成的数组
        Season1[] values = Season1.values();
        for(int i = 0;i < values.length;i++){
            System.out.println(values[i]);
        }
        System.out.println("****************");
        Thread.State[] values1 = Thread.State.values();
        for (int i = 0; i < values1.length; i++) {
            System.out.println(values1[i]);
        } 

        //valueOf(String objName):返回枚举类中对象名是objName的对象。
        Season1 winter = Season1.valueOf("WINTER");
        //如果没objName的枚举类对象，则抛异常：IllegalArgumentException
//        Season1 winter = Season1.valueOf("WINTER1");
        System.out.println(winter);

```

**:red_circle:实现接口的，枚举类**

```java
5. 使用enum定义枚举类之后，如何让枚举类对象分别实现接口：
interface Info{
    void show();
}

//使用enum关键字枚举类
enum Season1 implements Info{
    //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
    SPRING("春天","春暖花开"){
        @Override
        public void show() {
            System.out.println("春天在哪里？");
        }
    },
    SUMMER("夏天","夏日炎炎"){
        @Override
        public void show() {
            System.out.println("宁夏");
        }
    },
    AUTUMN("秋天","秋高气爽"){
        @Override
        public void show() {
            System.out.println("秋天不回来");
        }
    },
    WINTER("冬天","冰天雪地"){
        @Override
        public void show() {
            System.out.println("大约在冬季");
        }
    };
}

```



## 7.2 注解（Annotation）的使用

**:red_circle:注解(Annotation)概述**

（1）从 JDK 5.0 开始, Java 增加了对元数据(MetaData) 的支持, 也就是 Annotation(注解) 

（2）Annotation 其实就是代码里的**特殊标记**, 这些标记可以在编译, 类加 载, 运行时被读取, 并执行相应的处理。通过使用 Annotation, 程序员 可以在不改变原有逻辑的情况下, 在源文件中嵌入一些补充信息。代 码分析工具、开发工具和部署工具可以通过这些补充信息进行验证 或者进行部署。

（3）Annotation 可以像修饰符一样被使用, 可用于**修饰包,类,构造器, 方法，成员变量，参数 局部变量的声明,** 这些信息被保存在 Annotation 的 “name=value” 对中。

（4）在JavaSE中，注解的使用目的比较简单，例如标记过时的功能， 

忽略警告等。==在JavaEE/Android中注解占据了更重要的角色==，例如 

用来配置应用程序的任何切面，代替JavaEE旧版中所遗留的繁冗 

代码和XML配置等。



（5）未来的开发模式都是基于注解的，JPA是基于注解的，Spring2.5以 

上都是基于注解的，Hibernate3.x以后也是基于注解的，现在的 

Struts2有一部分也是基于注解的了，注解是一种趋势，一定程度上 

==可以说：框架 = 注解 + 反射 + 设计模式==。

---

**:red_circle:常见的Annotation示例**

使用 Annotation 时要在其前面增加 @ 符号, 并**把该** **Annotation** **当成 一个修饰符使用。**用于修饰它支持的程序元素 

**示例一：生成文档相关的注解** 

@author 标明开发该类模块的作者，多个作者之间使用,分割 

@version 标明该类模块的版本 

@see 参考转向，也就是相关主题 

@since 从哪个版本开始增加的 

@param 对方法中某参数的说明，如果没有参数就不能写 

@return 对方法返回值的说明，如果方法的返回值类型是void就不能写 

@exception 对方法可能抛出的异常进行说明 ，如果方法没有用throws显式抛出的异常就不能写 



**@param @return 和 @exception 这三个标记都是只用于方法的。** 

**@param的格式要求：@param 形参名 形参类型 形参说明** 

**@return 的格式要求：@return 返回值类型 返回值说明** 

**@exception的格式要求：@exception 异常类型 异常说明** 

**@param和@exception可以并列多个**



**示例二：在编译时进行格式检查(JDK内置的三个基本注解)** 

**@Override:** 限定重写父类方法, 该注解只能用于方法 

**@Deprecated:** 用于表示所修饰的元素(类, 方法等)已过时。通常是因为所修饰的结构危险或存在更好的选择 

**@SuppressWarnings:** 抑制编译器警告

---

**:red_circle:JDK中的元注解**

**元注解 ：对现有的注解进行解释说明的注解。** 
jdk 提供的4种元注解：
==Retention：==指定所修饰的 Annotation 的生命周期：SOURCE\CLASS（默认行为\RUNTIME只声明为RUNTIME生命周期的注解，才能通过反射获取。
==Target:==用于指定被修饰的 Annotation 能用于修饰哪些程序元素

出现的频率较低
==Documented:==表示所修饰的注解在被Javadoc解析时，保留下来。
==Inherited:==被它修饰的 Annotation 将具继承性。

---

**:red_circle:利用反射获取注解信息(在反射部分涉及)​**

**如何获取注解信息:通过发射来进行获取、调用。**
前提：要求此注解的元注解Retention中**声明的生命周期状态为：RUNTIME.**

---

**:red_circle:JDK8中注解的新特性​**

**JDK8中注解的新特性：可重复**



**注解、类型注解**

可重复注解：① 在MyAnnotation上声明@Repeatable，成员值为MyAnnotations.class
               ② MyAnnotation的Target和Retention等元注解与MyAnnotations相同。

**类型注解：**
ElementType.TYPE_PARAMETER **表示该注解能写在类型变量的声明语句中**（如：泛型声明。
ElementType.TYPE_USE **表示该注解能写在使用类型的任何语句中。**



# 8.0 Java集合

## 8.1Java集合框架概述

**1.集合与数组存储数据的概述**

==集合、数组都是对多个数据进行存储操作的结构，简称Java容器。==

说明：此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt,.jpg,.avi，数据库中)

**2.数组存储的特点**

 > ==一旦初始化以后，其长度就确定了。==
 > 数组一旦定义好，其==元素的类型也就确定==了。我们也就==只能操作指定类型的数据==了。
 >
 > *       比如：String[] arr;int[] arr1;Object[] arr2;

**3.数组存储的弊端**

* > 一旦初始化以后，==其长度就不可修改==。

* > 数组中提供的方法非常限，==对于添加、删除、插入数据等操作，非常不便==，同时效率不高。

* > 获取数组中实际元素的个数的需求，==数组没有现成的属性或方法可用==

* > 数组存储数据的==特点：有序、可重复==。对于==无序、不可重复的需求，不能满足==。

**4.集合存储的优点**

**解决数组存储数据方面的弊端。**



## 8.2  Collection接口方法 

**Collection**接口：单列数据，定义了存取一组对象的方法的集合 

**List**：元素有序、可重复的集合 

**Set**：元素无序、不可重复的集合 

![1642471082122](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642471082122.png)

**Map**接口：双列数据，保存具有映射关系“key-value对”的集合



![1642471102321](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642471102321.png)

---

:red_circle:15个方法

**1、添加** 

​      add(Object obj) 

​      addAll(Collection coll) 

**2、获取有效元素的个数** 

​      int size() 

**3、清空集合** 

​       void clear() 

**4、是否是空集合** 

​       boolean isEmpty() 

**5、是否包含某个元素** 

​       boolean contains(Object obj)：是通过元素的equals方法来判断是否 是同一个对象 

​       boolean containsAll(Collection c)：也是调用元素的equals方法来比 较的。拿两个集合的元素挨个     比较。

**6、删除** 

​       boolean remove(Object obj) ：通过元素的equals方法判断是否是 

要删除的那个元素。只会删除找到的第一个元素 

​      boolean removeAll(Collection coll)：取当前集合的差集 

**7、取两个集合的交集** 

​      boolean retainAll(Collection c)：把交集的结果存在当前集合中，不 影响c 

**8、集合是否相等** 

​      boolean equals(Object obj) 

**9、转成对象数组** 

​      Object[] toArray() 

**10、获取集合对象的哈希值** 

​       hashCode() 

**11、遍历** 

​       iterator()：返回迭代器对象，用于集合遍历

z

## 8.3  Iterator迭代器接口   

GOF给迭代器模式的定义为：提供一种方法访问一个**容器(container)对象**中各个元 

素，而又不需暴露该对象的内部细节。**迭代器模式，就是为容器而生。**类似于“公 

交车上的售票员”、“火车上的乘务员”、“空姐”。 

![1642559348385](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1642559348385.png)

```java
public class IteratorTest {
    @Test
    public void test7(){
        Collection coll=new ArrayList();

        coll.add(123);
        coll.add(456);
        coll.add(new String("Tom"));
        coll.add(456);
        coll.add(new Person("jerry",20));


        Iterator iterator=coll.iterator();
        //方式一：遍历数组中的各元素
//        System.out.println(iterator.next());
//        System.out.println(iterator.next());
//        System.out.println(iterator.next());
//        System.out.println(iterator.next());
//        System.out.println(iterator.next());
//        System.out.println(iterator.next());
        //方式二：
//        for (int i = 0; i < coll.size(); i++) {
//            System.out.println(iterator.next());
//        }
        //方式三：
        while (iterator.hasNext()){
            //①指针下移②将下移以后集合位置上的元素返回
            System.out.println(iterator.next());
        }

    }
     @Test
    //调用remove方法
    public void test8(){
        Collection coll=new ArrayList();

        coll.add(123);
        coll.add(456);
        coll.add(new String("Tom"));
        coll.add(456);
        coll.add(new Person("jerry",20));

        //移除remove（）
        Iterator iterator=coll.iterator();
        while (iterator.hasNext()){
            Object obj=iterator.next();
            if("Tom".equals(obj)){
                iterator.remove();
            }
        }
        Iterator iterator1 = coll.iterator();
        while (iterator1.hasNext()){
            System.out.println(iterator1.next());
        }

    }
}
```

==注意：==在调用it.next()方法之前必须调用it.hasNext()进行检测。若不调用，且下一条记录无效，直接调用it.next()会抛出异常 。



```java
/*
* jdk5.0新增了foreach循环，用于遍历集合，数组
*
* */
public class foreachtest {
    @Test
    public void test1(){
        Collection arrayList=new ArrayList();

        arrayList.add("AA");
        arrayList.add("BB");
        arrayList.add(123);
        arrayList.add(new Date());

        //for(集合元素的类型 局部变量 ：集合对象)
        //内部仍然调用了迭代器
        for (Object obj:arrayList) {
            System.out.println(obj);
        }
    }
    @Test
    public void test2(){
        int[] arr=new int[]{1,2,3,4,5,6,7};
        //for(数组元素的数据类型 局部变量：数组对象)
        for (int i:arr) {
            System.out.println(i);
        }
    }
}
```



## 8.4  Collection子接口一：List 

- 鉴于Java中数组用来==存储数据的局限性==，我们通常使用List代替数组

- List集合类中**元素有序，且可重复**，集合中的每个元素都有对应的顺序索引

- List容器中的元素都**对应一个整数型的序号记载其在容器中的位置**，可以**根据序号存取容器中的元素。**

- **JDK API中List接口的实现类常有：ArrayList，LinkedList和Vector。**

```java
 *Collection接口：单列集合，用来存储一个一个的对象
 *   list接口：存储有序的，可重复的数据-->“动态”数组
 *      ArrayList：作为List接口的主要实现类：线程不安全，效率高；底层使用Object[]elementData存储
 *      LinkedList：对于频繁的插入、删除操作，使用此类效率比ArryayList高；底层使用双向链表存储
 *      Vector：作为List接口的古老实现类；线程安全的，效率低；底层使用Object[] elementDate
 *
 *
 *一：ArrayList、LinkedList、Vector三者异同？
 * 三者相同点在于：三个类都是实现了List接口，存储数据的特点相同：存储有序的，可重复的数据
 * 不同：见上
 *
 *
 *
 * ArrayList的源码分析：
 *①jdk7情况下
 *    ArrayList list =new ArrayList();//底层创建了长度为10的Object[]数组elementData
 *    list.add(123);//elementDate[0]=new Integer(123);
 *
 *    list.add(11);//如果此次的添加导致底层elementData数组容量不够，则扩容。
 *    默认情况下，扩容为原来的容量的1.5倍，同时需要将原有数组中的数据复制到新的数组中
 *
 *    结论：建议开发中使用带参的构造器：ArrayList list=newArrayList(int capacity)
 *
 *②jdk 8中的ArrayList的变化：
 *
 *    ArrayList list=new ArrayList();//底层Object[]elementData初始化为{}.并没有创建长度为10的数组
 *
 *    list.add(123);//第一次调用add()时，底层才创建了长度为10的数组，并将数据123添加到elementData数组中
 *
 *③ 小结：jdk7中的ArrayList的对象的创建类似于单例模式的饿汉式，而jdk8中的ArrayList的对象的创建类似于单例模式的懒汉式，
 *   延迟了数组的创建，节省内存 。
 *
 * 3. LinkedList的源码分析：
 *      LinkedList list = new LinkedList(); 内部声明了Node类型的first和last属性，默认值为null
 *      list.add(123);//将123封装到Node中，创建了Node对象。
 *
 *      其中，Node定义为：体现了LinkedList的双向链表的说法
 *      private static class Node<E> {
               E item;
               Node<E> next;
               Node<E> prev;

               Node(Node<E> prev, E element, Node<E> next) {
               this.item = element;
               this.next = next;
               this.prev = prev;
           }
         }
 *
 * 4. Vector的源码分析：jdk7和jdk8中通过Vector()构造器创建对象时，底层都创建了长度为10的数组。
 *      在扩容方面，默认扩容为原来的数组长度的2倍。
 *
 *  面试题：ArrayList、LinkedList、Vector三者的异同？
 *  同：三个类都是实现了List接口，存储数据的特点相同：存储有序的、可重复的数据
 *  不同：见上
```

```java
* 5.List接口中的常用方法
 * void add(int index, Object ele):在index位置插入ele元素
 * boolean addAll(int index, Collection eles):从index位置开始将eles中的所有元素添加进来
 * Object get(int index):获取指定index位置的元素
 * int indexOf(Object obj):返回obj在集合中首次出现的位置
 * int lastIndexOf(Object obj):返回obj在当前集合中末次出现的位置
 * Object remove(int index):移除指定index位置的元素，并返回此元素
 * Object set(int index, Object ele):设置指定index位置的元素为ele
 * List subList(int fromIndex, int toIndex):返回从fromIndex到toIndex
 *
 * 总结：常用方法
 *     增:add()
 *     删:remeove()
 *     改:set(int index,Object ele)
 *     查:get(int index)
 *     插入:add(int index,Object ele)
 *     长度：size()
 *     遍历:①Iterator
 *         ②增强for循环
 *         ③普通的循环
 *
 *
 *
 * */
public class ListTest {
   @Test
   public  void test1(){
       ArrayList list=new ArrayList();
       list.add(123);
       list.add(456);
       list.add("AA");
       list.add(new Person("Tom",12));
       list.add(456);

       System.out.println(list);//[123, 456, AA, Person{name='Tom', age=12}, 456]
       //void add(int index, Object ele):在index位置插入ele元素
       list.add(1,"AA");
       System.out.println(list);//[123, AA, 456, AA, Person{name='Tom', age=12}, 456]插入到索引为一的位置

       //boolean addAll(int index, Collection eles):从index位置开始将eles中的所有元素添加进来
       List list1 = Arrays.asList(1, 2, 3);
       list.addAll(list1);
       System.out.println(list.size());   //此时数组中有9个元素

       //Object get(int index):获取指定index位置的元素
       System.out.println(list.get(0));


   }
   @Test
   public void test2(){
       ArrayList list=new ArrayList();
       list.add(123);
       list.add(456);
       list.add("AA");
       list.add(new Person("Tom",12));
       list.add(456);

       //int indexOf(Object obj):返回obj在集合中首次出现的位置      没有返回负一     有则返回第一次出现的位置
       System.out.println(list.indexOf(456));

       //int lastIndexOf(Object obj):返回obj在当前集合中末次出现的位置
       System.out.println(list.indexOf(456));

       //Object remove(int index):移除指定index位置的元素，并返回此元素
       Object remove = list.remove(0);
       System.out.println(remove);
       System.out.println(list);

       //Object set(int index, Object ele):设置指定index位置的元素为ele
       list.set(1,"cc");
       System.out.println(list);

       //List subList(int fromIndex, int toIndex):返回从fromIndex到toIndex   返回左闭右开之间的子集和
       List list1 = list.subList(2, 4);
       System.out.println(list1);
   }
   @Test
   public void test3(){
       ArrayList list=new ArrayList();
       list.add(123);
       list.add(456);
       list.add("AA");

       //方式一：迭代器遍历
       Iterator iterator=list.iterator();
       while(iterator.hasNext()){
           System.out.println(iterator.next());
       }
       //方式二：增强for循环
       for(Object obj:list){
           System.out.println(obj);
       }

       //方式三：普通的for循环
       for (int i = 0; i < list.size(); i++) {
           System.out.println(list.get(i));
       }

   }

}

```



## 8.5  Collection子接口二：Set

① Set接口是Collection的子接口，set接口没有提供额外的方法 

② Set 集合不允许包含相同的元素，如果试把两个相同的元素加入同一个 

​    Set 集合中，则添加操作失败。 

③ Set 判断两个对象是否相同不是使用 == 运算符，而是根据 equals() 方法

***

**Set实现类之一：HashSet**

![1644136669774](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644136669774.png)

 HashSet 是 Set 接口的典型实现，大多数时候使用 Set 集合时都使用这个实现类。 

HashSet 具有以下特点：
不能保证元素的排列顺序
HashSet 不是线程安全的
集合元素可以是 null

 底层拥有数组和链表 

***

**Set实现类之二：LinkedHashSet**

![1644136740400](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644136740400.png)

 LinkedHashSet 是 HashSet 的子类 

LinkedHashSet插入性能略低于 HashSet，但在迭代访问 Set 里的全部元素时有很好的性能。
LinkedHashSet 不允许集合元素重复。

 看似是有序的 



**Set实现类之三：TreeSet**

![1644136813358](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644136813358.png)

 TreeSet底层使用红黑树结构存储数据 

 TreeSet 是 SortedSet 接口的实现类，TreeSet 可以确保集合元素处于排序状态。 

 可以按照对象的指定属性进行排序 

![1644136868710](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644136868710.png)



## 8.6  Map接口 

:red_circle:**Map接口概述**

***

① Map与Collection并列存在。用于保存具有**映射关系**的数据:key-value 

② Map 中的 key 和 value 都可以是任何引用类型的数据 

③ Map 中的 key 用Set来存放，**不允许重复**，即同一个 Map 对象所对应 

的类，须重写hashCode()和equals()方法 

④ 常用String类作为Map的“键” 

⑤ key 和 value 之间存在单向一对一关系，即通过指定的 key 总能找到 

唯一的、确定的 value 

⑥ Map接口的常用实现类：HashMap、TreeMap、LinkedHashMap和 

Properties。==其中，HashMap是 Map 接口使用频率最高的实现类==

![1644137011469](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644137011469.png)



**Map实现类之一：HashMap**

***





![1644137062590](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644137062590.png)

![1644137113914](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1644137113914.png)

①HashMap是 Map 接口**使用频率最高**的实现类。 

②允许使用null键和null值，与HashSet一样，不保证映射的顺序。 

③所有的key构成的集合是Set:无序的、不可重复的。所以，key所在的类要重写： 

equals()和hashCode() 

④所有的value构成的集合是Collection:无序的、可以重复的。所以，value所在的类 

要重写：equals() 

⑤一个key-value构成一个entry 

⑥所有的entry构成的集合是Set:无序的、不可重复的 

⑦HashMap **判断两个** **key** **相等的标准**是：两个 key 通过 equals() 方法返回 true， 

hashCode 值也相等。 

⑧HashMap **判断两个** **value**相等的标准**是：两个 value 通过 equals() 方法返回 true。



**Map实现类之二：LinkedHashMap**

***

①LinkedHashMap 是 HashMap 的子类 

②在HashMap存储结构的基础上，使用了一对双向链表来记录添加 

元素的顺序 

③与LinkedHashSet类似，LinkedHashMap 可以维护 Map 的迭代 

顺序：迭代顺序与 Key-Value 对的插入顺序一致



**Map实现类之三：TreeMap** 

***

①TreeMap存储 Key-Value 对时，需要根据 key-value 对进行排序。 

TreeMap 可以保证所有的 Key-Value 对处于**有序**状态。 

②TreeSet底层使用**红黑树**结构存储数据 

③TreeMap 的 Key 的排序： 

④**自然排序**：TreeMap 的所有的 Key 必须实现 Comparable 接口，而且所有 的 Key 应该是同一个类的对象，否则将会抛出 ClasssCastException 

⑤**定制排序**：创建 TreeMap 时，传入一个 Comparator 对象，该对象负责对 TreeMap 中的所有 key 进行排序。此时不需要 Map 的 Key 实现 Comparable 接口 

⑥ TreeMap判断**两个****key****相等的标准**：两个key通过compareTo()方法或 者compare()方法返回0。



**Map实现类之四：Hashtable**

***

① Hashtable是个古老的 Map 实现类，JDK1.0就提供了。不同于HashMap， Hashtable是线程安全的。 

② Hashtable实现原理和HashMap相同，功能相同。底层都使用哈希表结构，查询 速度快，很多情况下可以互用。 

③ 与HashMap不同，Hashtable 不允许使用 null 作为 key 和 value 

④ 与HashMap一样，Hashtable 也不能保证其中 Key-Value 对的顺序 

⑤ Hashtable判断两个key相等、两个value相等的标准，与HashMap一致。



**Map实现类之五：Properties**

***

①Properties 类是 Hashtable 的子类，该对象用于处理属性文件 

②由于属性文件里的 key、value 都是字符串类型，所以 Properties 里的 key  和 value 都是字符串类型 

③存取数据时，建议使用setProperty(String key,String value)方法和 getProperty(String key)方法 



## 8.7  Collections工具类

# 9.0 泛型（更新）

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

## 1.3 泛型方法-E类型实例

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

