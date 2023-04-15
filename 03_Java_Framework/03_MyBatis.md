# MyBatis

## 学习目标

- 理解MyBatis的作用

- 掌握MyBatis基本增删改查操作

- 掌握MyBatis的分页操作
- 掌握MyBatis的一对一和一对多

## Mybatis快速入门

### 框架介绍

* 框架是一款半成品软件，我们可以基于这个半成品软件继续开发，来完成我们个性化的需求！

* 如图:

  ![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/5b0988e595225.cdn.sohucs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg)

 

### ORM介绍

* `ORM(Object Relational Mapping)`： 对象关系映射

* 指的是持久化数据和实体对象的映射模式，为了解决面向对象与关系型数据库存在的互不匹配的现象的技术。

* 如图:

  ![1590919786415](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590919786415.png)

* 具体映射关系如下图:

![1590919824416](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590919824416.png)



> ORM框架：
>
> - MyBatis
> - JPA
> - Hibernate

### 原始jdbc操作



#### 测试SQL语句

```sql
CREATE DATABASE db1;
USE db1;

CREATE TABLE student(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20),
	age INT
);

INSERT INTO student VALUES (NULL,'张三',23);
INSERT INTO student VALUES (NULL,'李四',24);
INSERT INTO student VALUES (NULL,'王五',25);

SELECT * FROM student;
```



#### 查询操作

目标： 从student表中查询数据 映射到 Student类中



```java
package com.iflytek.bean;

public class Student {
    private Integer id;
    private String name;
    private Integer age;

    // 省略 setter和getter。。。
}

```





```java
@Test
public void query() throws Exception {

    // 加载数据库驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    // 数据库连接信息
    String url = "jdbc:mysql:///test";
    String username = "root";
    String password = "12345678";

    // 创建数据库连接
    Connection conn = DriverManager.getConnection(url, username, password);

    // 预编译SQL语句
    PreparedStatement ps = conn.prepareStatement("SELECT * FROM student;");

    // 执行查询
    ResultSet rs = ps.executeQuery();

    // 遍历结果
    while (rs.next()) {
        Student stu = new Student();
        stu.setId(rs.getInt("id"));
        stu.setName(rs.getString("name"));
        stu.setAge(rs.getInt("age"));
        System.out.println(stu);
    }

    // 关闭连接
    rs.close();
    ps.close();
    conn.close();

}
```



#### 插入数据

```java
@Test
public void save() throws Exception {

    Student stu = new Student(null, "赵六", 30);

    // 加载数据库驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    // 数据库连接信息
    String url = "jdbc:mysql:///test";
    String username = "root";
    String password = "12345678";

    // 创建数据库连接
    Connection conn = DriverManager.getConnection(url, username, password);

    // 预编译SQL语句
    PreparedStatement ps = conn.prepareStatement("INSERT INTO student VALUES (NULL,?,?);");
    ps.setString(1, stu.getName());
    ps.setInt(2, stu.getAge());

    // 执行查询
    int result = ps.executeUpdate();

    // 打印结果
    System.out.println(result);

    // 关闭连接
    ps.close();
    conn.close();

}
```



#### 原始jdbc操作的分析

* 原始 JDBC 的操作问题分析 

      1. 频繁创建和销毁数据库的连接会造成系统资源浪费从而影响系统性能。
      2. sql 语句在代码中硬编码，如果要修改 sql 语句，就需要修改 java 代码，造成代码不易维护。
      3. 查询操作时，需要手动将结果集中的数据封装到实体对象中。
      4. 增删改查操作需要参数时，需要手动将实体对象的数据设置到 sql 语句的占位符。 

* 原始 JDBC 的操作问题解决方案 

      1. 使用数据库连接池初始化连接资源。
      2. 将 sql 语句抽取到配置文件中。 
      3. 使用反射、内省等底层技术，将实体与表进行属性与字段的自动映射    



### 什么是Mybatis



![MyBatis logo](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/mybatis-logo.png)

MyBatis官网地址：<http://www.mybatis.org/mybatis-3/> 

中文文档：https://mybatis.org/mybatis-3/zh/index.html



- mybatis 是一个优秀的基于java的`持久层`框架，它`内部封装了jdbc`，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。


- mybatis通过`xml`或注解的方式将要执行的各种 statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的`sql语句`。


- 最后mybatis框架执行sql并将结果映射为java对象并返回(ResultSet->Java对象)。采用ORM思想解决了实体和数据库映射的问题，对jdbc 进行了封装，屏蔽了jdbc api 底层访问细节，使我们不用与jdbc api 打交道，就可以完成对数据库的持久化操作。





### Mybatis入门案例

本章节为快速入门案例，实现用MyBatis的方式去查询数据库表中的所有数据。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704162024849.png" alt="image-20210704162024849"  />

数据库表还是我们前面建好的student表

```sql
SELECT * FROM student
```



#### **MyBatis开发步骤**

1. 创建Maven工程
2. 添加相关依赖包
3. 编写Student实体类
4. 编写Mapper文件
5. 配置MyBatis环境
6. 编写测试类


#### 环境搭建

1)导入MyBatis的jar包

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.25</version>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>RELEASE</version>
    <scope>compile</scope>
</dependency>
```



2)  创建student数据表

![1590916243454](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590916243454.png)

3) 编写Student实体

```java
package com.iflytek.bean;


public class Student {
    private Integer id;
    private String name;
    private Integer age;
    //省略get个set方法


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```



4)在resources资源目录下创建`StudentMapper.xml`映射文件



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="com.iflytek.bean.Student">
        SELECT * FROM student
    </select>
</mapper>
```



5) 在resources资源目录下创建MyBatis配置文件MyBatisConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration 核心根标签-->
<configuration>

    <!--environments配置数据库环境，环境可以有多个。default属性指定使用的是哪个-->
    <environments default="mysql">
        <!--environment配置数据库环境  id属性唯一标识-->
        <environment id="mysql">
            <!-- transactionManager事务管理。  type属性，采用JDBC默认的事务-->
            <transactionManager type="JDBC"/>
            <!-- dataSource数据源信息   type属性 连接池-->
            <dataSource type="POOLED">
                <!-- property获取数据库连接的配置信息 -->
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql:///test" />
                <property name="username" value="root" />
                <property name="password" value="12345678" />
            </dataSource>
        </environment>
    </environments>

    <!-- mappers引入映射配置文件 -->
    <mappers>
        <!-- mapper 引入指定的映射配置文件   resource属性指定映射配置文件的名称 -->
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```



#### 编写测试代码



![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image.mamicode.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg)



```java
package com.iflytek.test;

import com.iflytek.bean.Student;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.List;

public class StudentTest01 {

    /**
     * 查询全部
     */
    @Test
    public void selectAll() throws Exception {
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过SqlSession工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //4.执行映射配置文件中的sql语句，并接收结果
        List<Student> list = sqlSession.selectList("StudentMapper.selectAll");

        //5.处理结果
        for (Student stu : list) {
            System.out.println(stu);
        }

        //6.释放资源
        sqlSession.close();
        is.close();
    }
}

```



#### 知识小结

* 框架       

  框架是一款半成品软件，我们可以基于框架继续开发，从而完成一些个性化的需求。

* ORM        

  对象关系映射，数据和实体对象的映射。

* MyBatis       

  是一个优秀的基于 Java 的持久层框架，它内部封装了 JDBC。

  

## MyBatis的相关api

### Resources

* org.apache.ibatis.io.Resources：加载资源的工具类。

* 核心方法

  ![1590917572321](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590917572321.png)

### 构建器SqlSessionFactoryBuilder

* org.apache.ibatis.session.SqlSessionFactoryBuilder：获取 SqlSessionFactory 工厂对象的功能类

* 核心方法

  ![1590916852504](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590916852504.png)

* 通过加载mybatis的核心文件的输入流的形式构建一个SqlSessionFactory对象

```java
String resource = "org/mybatis/builder/mybatis-config.xml"; 
InputStream inputStream = Resources.getResourceAsStream(resource); 
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder(); 
SqlSessionFactory factory = builder.build(inputStream);
```

其中， Resources 工具类，这个类在 org.apache.ibatis.io 包中。Resources 类帮助你从类路径下、文件系统或一个 web URL 中加载资源文件。

### 工厂对象SqlSessionFactory

* org.apache.ibatis.session.SqlSessionFactory：获取 SqlSession 构建者对象的工厂接口。

* 核心api

  ![1590917006637](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590917006637.png)



### SqlSession会话对象

* org.apache.ibatis.session.SqlSession：构建者对象接口。用于执行 SQL、管理事务、接口代理。

* 核心api

  ![1590917052849](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590917052849.png)

SqlSession 实例在 MyBatis 中是非常强大的一个类。在这里你会看到所有执行语句、提交或回滚事务和获取映射器实例的方法。



## MyBatis 映射配置文件

映射配置文件包含了数据和对象之间的映射关系以及要执行的 SQL 语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="com.iflytek.mapper.StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="com.iflytek.bean.Student">
        SELECT * FROM student
    </select>
</mapper>
```





### 查询功能

- select：查询功能标签。

* 属性        

  id：唯一标识， 配合名称空间使用。     

  parameterType：指定参数映射的对象类型。       

  resultType：指定结果映射的对象类型。

* SQL 获取参数:        #{属性名}

* 示例

  ```xml
  <select id="selectById" parameterType="int" resultType="com.iflytek.bean.Student">
      SELECT *
      FROM student
      WHERE id = #{id}
  </select>
  ```

* 测试代码

  ```java
  @Test
  public void selectById() throws Exception {
      //1.加载核心配置文件
      InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
  
      //2.获取SqlSession工厂对象
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
  
      //3.通过SqlSession工厂对象获取SqlSession对象
      SqlSession sqlSession = sqlSessionFactory.openSession();
  
      //4.执行映射配置文件中的sql语句，并接收结果
      Student stu = sqlSession.selectOne("StudentMapper.selectById", 1);
  
      //5.处理结果
      System.out.println(stu);
  
      //6.释放资源
      sqlSession.close();
      is.close();
  }
  ```

  

### 新增功能 

- insert：新增功能标签。

- 属性        

  id：唯一标识， 配合名称空间使用。     

  parameterType：指定参数映射的对象类型。       

  resultType：指定结果映射的对象类型。

- SQL 获取参数:        #{属性名}

* 示例

  ```xml
  <insert id="insert" parameterType="student">
      INSERT INTO student
      VALUES (#{id}, #{name}, #{age})
  </insert>
  ```

* 测试代码

  ```java
  @Test
  public void insert() throws Exception {
      //1.加载核心配置文件
      InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
  
      //2.获取SqlSession工厂对象
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
  
      //3.通过SqlSession工厂对象获取SqlSession对象
      SqlSession sqlSession = sqlSessionFactory.openSession();
  
      //4.执行映射配置文件中的sql语句，并接收结果
      Student stu = new Student();
      stu.setId(8);
      stu.setName("王五");
      stu.setAge(30);
  
      int result = sqlSession.insert("StudentMapper.insert", stu);
      // 5. 提交事务
      sqlSession.commit();
  
      //6.处理结果
      System.out.println(result);
  
      //6.释放资源
      sqlSession.close();
      is.close();
  }
  ```



### 修改功能

- update：修改功能标签。

- 属性        

  id：唯一标识， 配合名称空间使用。     

  parameterType：指定参数映射的对象类型。       

  resultType：指定结果映射的对象类型。

- SQL 获取参数:        #{属性名}

* 示例

  ```xml
  <update id="update" parameterType="student">
      UPDATE student
      SET name = #{name}, age  = #{age}
      WHERE id = #{id}
  </update>
  ```

* 测试代码

  ```java
  @Test
  public void update() throws Exception {
      //1.加载核心配置文件
      InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
  
      //2.获取SqlSession工厂对象
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
  
      //3.通过SqlSession工厂对象获取SqlSession对象
      SqlSession sqlSession = sqlSessionFactory.openSession();
  
      //4.执行映射配置文件中的sql语句，并接收结果
      Student stu = new Student();
      stu.setId(8);
      stu.setName("赵六");
      stu.setAge(33);
  
      int result = sqlSession.update("StudentMapper.update", stu);
      // 5. 提交事务
      sqlSession.commit();
  
      //6.处理结果
      System.out.println(result);
  
      //7.释放资源
      sqlSession.close();
      is.close();
  }
  ```

  

### 删除功能

- <delete>：查询功能标签。

- 属性        

  id：唯一标识， 配合名称空间使用。     

  parameterType：指定参数映射的对象类型。       

  resultType：指定结果映射的对象类型。

- SQL 获取参数:        #{属性名}

* 示例

  ```xml
  <delete id="delete" parameterType="int">
      DELETE
      FROM student
      WHERE id = #{id}
  </delete>
  ```

* 测试代码

  ```java
  @Test
  public void delete() throws Exception {
      //1.加载核心配置文件
      InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
  
      //2.获取SqlSession工厂对象
      SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
  
      //3.通过SqlSession工厂对象获取SqlSession对象
      SqlSession sqlSession = sqlSessionFactory.openSession();
  
      //4.执行映射配置文件中的sql语句，并接收结果
      int result = sqlSession.delete("StudentMapper.delete", 8);
      // 5. 提交事务
      sqlSession.commit();
  
      //6.处理结果
      System.out.println(result);
  
      //7.释放资源
      sqlSession.close();
      is.close();
  }
  ```

  

* 总结： 大家可以发现crud操作，除了标签名称以及sql语句不一样之外，其他属性参数基本一致。



### 映射配置文件小结

![1590918743943](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590918743943.png)



## Mybatis核心配置文件介绍



### 核心配置文件介绍

MyBatis的配置文件包含了影响MyBatis行为的信息。文档的结构如下:

- 顶层configuration 配置
- properties属性
- settings设置
- typeAliases类型命名
- typeHandlers类型处理器
- objectFactory对象工厂
- plugins插件
- environments环境
- environment环境变量
- transactionManager事务管理器
- dataSource数据源
- databaseIdProvider数据库厂商标识
- mappers映射器

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 配置文件的根元素 -->
<configuration>
    <!-- 属性：定义配置外在化 -->
    <properties></properties>
    <!-- 设置：定义mybatis的一些全局性设置 -->
    <settings>
       <!-- 具体的参数名和参数值 -->
       <setting name="" value=""/> 
    </settings>
    <!-- 类型名称：为一些类定义别名 -->
    <typeAliases></typeAliases>
    <!-- 类型处理器：定义Java类型与数据库中的数据类型之间的转换关系 -->
    <typeHandlers></typeHandlers>
    <!-- 对象工厂 -->
    <objectFactory type=""></objectFactory>
    <!-- 插件：mybatis的插件,插件可以修改mybatis的内部运行规则 -->
    <plugins>
       <plugin interceptor=""></plugin>
    </plugins>
    <!-- 环境：配置mybatis的环境 -->
    <environments default="">
       <!-- 环境变量：可以配置多个环境变量，比如使用多数据源时，就需要配置多个环境变量 -->
       <environment id="">
          <!-- 事务管理器 -->
          <transactionManager type=""></transactionManager>
          <!-- 数据源 -->
          <dataSource type=""></dataSource>
       </environment> 
    </environments>
    <!-- 数据库厂商标识 -->
    <databaseIdProvider type=""></databaseIdProvider>
    <!-- 映射器：指定映射文件或者映射类 -->
    <mappers></mappers>
</configuration>
~~~



### 数据库连接配置文件引入

- jdbc.properties

  ```properties
  driver=com.mysql.cj.jdbc.Driver
  url=jdbc:mysql:///test
  username=root
  password=12345678
  ```

* properties标签引入外部文件

  ~~~xml
  <!--引入数据库连接的配置文件-->
  <properties resource="jdbc.properties"/>
  ~~~

* 具体使用，如下配置

  ~~~xml
  <!-- property获取数据库连接的配置信息 -->
  <property name="driver" value="${driver}" />
  <property name="url" value="${url}" />
  <property name="username" value="${username}" />
  <property name="password" value="${password}" />
  ~~~

  

### 起别名

在编写Mapper配置文件的时候，每次都编写Bean类的全限定名（包名+类名）会显得很繁琐，我们可以为bean指定下别名，也就是用短的名称去替代长长的全限定名。

* <typeAliases>：为全类名起别名的父标签。

* <typeAlias>：为全类名起别名的子标签。

* 属性      

  type：指定全类名      

  alias：指定别名

* <package>：为指定包下所有类起别名的子标签。(别名就是类名)

* 如下图：

  ![1590919106324](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590919106324.png)

* 具体如下配置

  ~~~xml
  <!--起别名-->
  <typeAliases>
      <!--<typeAlias type="com.iflytek.bean.Student" alias="student"/>-->
      <package name="com.iflytek.bean"/>
  </typeAliases>
  ~~~





### 总结

![1590919367790](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590919367790.png)





## 接口代理方式实现Dao



​	`传统的Dao`开发，我们需要提供Dao接口，Dao实现类，Mapper配置文件（包含Sql），大家可以观察下面三段代码，会发现Dao实现类显得特别繁琐，并且代码重复度较高，所以本章节我们学习更加实用的Mapper代理开发，也就是我们只包含Dao接口和Mapper配置文件，在MyBatis中，Dao接口我们常称为Mapper接口，这其实是一个意思，叫法不同而已。

### 传统DAO模式【了解】



![image-20210913092404490](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210913092404490.png)



#### Dao接口

```java
package com.iflytek.dao;


import com.iflytek.bean.Student;

import java.util.List;
/*
    持久层接口
 */
public interface StudentDao{
    //查询全部
    List<Student> selectAll();

    //根据id查询
     Student selectById(Integer id);

    //新增数据
    Integer insert(Student stu);

    //修改数据
    Integer update(Student stu);

    //删除数据
     Integer delete(Integer id);
}

```



#### XML片段

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        SELECT * FROM student
    </select>

    <select id="selectById" parameterType="int" resultType="com.iflytek.bean.Student">
        SELECT *
        FROM student
        WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="com.iflytek.bean.Student">
        INSERT INTO student
        VALUES (#{id}, #{name}, #{age})
    </insert>

    <update id="update" parameterType="com.iflytek.bean.Student">
        UPDATE student
        SET name = #{name}, age  = #{age}
        WHERE id = #{id}
    </update>

    <delete id="delete" parameterType="int">
        DELETE
        FROM student
        WHERE id = #{id}
    </delete>
</mapper>
```



#### Dao实现类



```java
package com.iflytek.dao.impl;

import com.iflytek.bean.Student;
import com.iflytek.dao.StudentDao;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/*
    持久层实现类
 */
public class StudentDaoImpl implements StudentDao {

    /*
        查询全部
     */
    @Override
    public List<Student> selectAll() {
        List<Student> list = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try {
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            list = sqlSession.selectList("StudentMapper.selectAll");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
            if (sqlSession != null) {
                sqlSession.close();
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //6.返回结果
        return list;
    }

    /*
        根据id查询
     */
    @Override
    public Student selectById(Integer id) {
        Student stu = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try {
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            stu = sqlSession.selectOne("StudentMapper.selectById", id);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
            if (sqlSession != null) {
                sqlSession.close();
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //6.返回结果
        return stu;
    }

    /*
        新增功能
     */
    @Override
    public Integer insert(Student stu) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try {
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.insert("StudentMapper.insert", stu);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
            if (sqlSession != null) {
                sqlSession.close();
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //6.返回结果
        return result;
    }

    /*
        修改功能
     */
    @Override
    public Integer update(Student stu) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try {
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.update("StudentMapper.update", stu);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
            if (sqlSession != null) {
                sqlSession.close();
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //6.返回结果
        return result;
    }

    /*
        删除功能
     */
    @Override
    public Integer delete(Integer id) {
        Integer result = null;
        SqlSession sqlSession = null;
        InputStream is = null;
        try {
            //1.加载核心配置文件
            is = Resources.getResourceAsStream("MyBatisConfig.xml");

            //2.获取SqlSession工厂对象
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

            //3.通过工厂对象获取SqlSession对象
            sqlSession = sqlSessionFactory.openSession(true);

            //4.执行映射配置文件中的sql语句，并接收结果
            result = sqlSession.delete("StudentMapper.delete", id);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.释放资源
            if (sqlSession != null) {
                sqlSession.close();
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //6.返回结果
        return result;
    }
}

```



#### 测试代码

```java
package com.iflytek.test;

import com.iflytek.bean.Student;
import com.iflytek.dao.StudentDaoImpl;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class StudentTest01 {

    @Test
    public void selectAll() throws Exception {

        StudentDaoImpl dao = new StudentDaoImpl();
        List<Student> students = dao.selectAll();

        for (Student student : students) {
            System.out.println(student);
        }

    }
}

```



#### 小结

传统的Dao开发模式，需要开发人员自己去编写Dao的实现类，但是实现类中的代码大部分都是重复且有步骤的，这个时候框架使用代理方式可以帮助我们动态实现Dao的实现类。



### Mapper代理开发方式介绍

采用 Mybatis 的代理开发方式实现 DAO 层的开发，这种方式是我们后面进入企业的主流。

Mapper 接口开发方法只需要程序员编写Mapper 接口（相当于Dao 接口），由Mybatis 框架根据接口定义创建接口的`动态代理对象`，代理对象的方法体同上边Dao接口实现类方法。

Mapper 接口开发需要遵循以下规范：

> **1) Mapper.xml文件中的namespace与mapper接口的全限定名相同**
>
> **2) Mapper接口方法名和Mapper.xml中定义的每个statement的id相同**
>
> **3) Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同**
>
> **4) Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同**



**总结:** 

接口开发的方式: 程序员只需定义接口,就可以对数据库进行操作,那么具体的对象怎么创建?

1. 程序员负责定义接口

2. 在操作数据库,mybatis框架根据接口,通过动态代理的方式生成代理对象,负责数据库的crud操作



#### 编写StudentMapper接口



```java
package com.iflytek.mapper;

import com.iflytek.bean.Student;
import java.util.List;

/*
    持久层接口
 */
public interface StudentMapper {
    //查询全部
    List<Student> selectAll();

    //根据id查询
    Student selectById(Integer id);

    //新增数据
    Integer insert(Student stu);

    //修改数据
    Integer update(Student stu);

    //删除数据
    Integer delete(Integer id);

}

```



#### Mapper配置文件



```sql
<?xml version="1.0" encoding="UTF-8" ?>
<!--MyBatis的DTD约束-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    mapper：核心根标签
    namespace属性：名称空间
-->
<mapper namespace="com.iflytek.mapper.StudentMapper">
    <!--
        select：查询功能的标签
        id属性：唯一标识
        resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射对象类型
    -->
    <select id="selectAll" resultType="student">
        SELECT * FROM student
    </select>

    <select id="selectById" parameterType="int" resultType="com.iflytek.bean.Student">
        SELECT *
        FROM student
        WHERE id = #{id}
    </select>

    <insert id="insert" parameterType="com.iflytek.bean.Student">
        INSERT INTO student
        VALUES (#{id}, #{name}, #{age})
    </insert>

    <update id="update" parameterType="com.iflytek.bean.Student">
        UPDATE student
        SET name = #{name}, age  = #{age}
        WHERE id = #{id}
    </update>

    <delete id="delete" parameterType="int">
        DELETE
        FROM student
        WHERE id = #{id}
    </delete>
</mapper>
```



#### 整合Log4j



**添加依赖**

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```



**MyBatisConfig.xml**

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```



**log4j.properties**

在resources下创建log4j.properties文件，名称不能变

```properties
log4j.rootLogger=DEBUG,console

########################控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
#在控制台输出
log4j.appender.console.Target = System.out
#在DEBUG级别输出
log4j.appender.console.Threshold = DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
#日志格式
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

######################日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```





#### 测试代理方式



```java
package com.iflytek.test;

import com.iflytek.bean.Student;
import com.iflytek.mapper.StudentMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.List;

public class StudentTest02 {

    /**
     * 查询全部
     */
    @Test
    public void selectAll() throws Exception {
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过SqlSession工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        //4.获取StudentMapper
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

        //5.执行查询
        List<Student> list = mapper.selectAll();

        //6.处理结果
        for (Student stu : list) {
            System.out.println(stu);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }

}

```



#### 源码分析

> * **分析动态代理对象如何生成的？** 
>
>   通过动态代理开发模式，我们只编写一个接口，不写实现类，我们通过 getMapper() 方法最终获取到 `org.apache.ibatis.binding.MapperProxy `代理对象，然后执行功能，而这个代理对象正是 MyBatis 使用了 JDK 的动态代理技术，帮助我们生成了代理实现类对象。从而可以进行相关持久化操作。 
>
> 
>
> * **分析方法是如何执行的？**
>
>   动态代理实现类对象在执行方法的时候最终调用了 `mapperMethod.execute() `方法，这个方法中通过 switch 语句根据操作类型来判断是新增、修改、删除、查询操作，最后一步回到了 MyBatis 最原生的 SqlSession 方式来执行增删改查。    



### 知识小结

 接口代理方式可以让我们只编写接口即可，而实现类对象由 MyBatis 生成。 

>  **实现规则 ：**
>
>  1. 映射配置文件中的名称空间必须和 Dao 层接口的全类名相同。
>  2. 映射配置文件中的增删改查标签的 id 属性必须和 Dao 层接口的方法名相同。 
>  3. 映射配置文件中的增删改查标签的 parameterType 属性必须和 Dao 层接口方法的参数相同。 
>  4. 映射配置文件中的增删改查标签的 resultType 属性必须和 Dao 层接口方法的返回值相同。
>  5. 获取动态代理对象 SqlSession 功能类中的 getMapper() 方法。    







## 动态sql语句



### 动态sql语句概述

​	Mybatis 的映射文件中，前面我们的 SQL 都是比较简单的，有些时候业务逻辑复杂时，我们的 SQL是动态变化的，此时在前面的学习中我们的 SQL 就不能满足要求了。

官方文档：https://mybatis.org/mybatis-3/zh/dynamic-sql.html

参考的官方文档，描述如下：

> 动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。
>
> 使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。
>
> 如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。
>
> - if
> - choose (when, otherwise)
> - trim (where, set)
> - foreach



### 动态 SQL  之<**if>** ，\<where>

我们根据实体类的不同取值，使用不同的 SQL语句来进行查询。比如在 id如果不为空时可以根据id查询，如果username 不为空时还要加入用户名作为条件。这种情况在我们的多条件组合查询中经常会碰到。

如下：

```sql
 select * from student where id=? and name=?
```



#### 接口方法



```java
//多条件查询
List<Student> findByCondition(Student stu);
```



#### XML片段



```xml
<select id="findByCondition" parameterType="student" resultType="student">
    select * from student
    <where>
        <if test="id!=null">
            and id=#{id}
        </if>
        <if test="name!=null">
            and name=#{name}
        </if>
    </where>
</select>
```



#### 测试代码

```java
@Test
public void findByCondition() throws Exception {
    //1.加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

    //2.获取SqlSession工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

    //3.通过SqlSession工厂对象获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //4.获取StudentMapper
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

    //5.执行查询
    Student stu = new Student();
    stu.setId(2);
    stu.setName("李四");
    // stu.setAge(24);

    List<Student> list = mapper.findByCondition(stu);

    //6.处理结果
    for (Student student : list) {
        System.out.println(student);
    }

    //7.释放资源
    sqlSession.close();
    is.close();
}
```



#### 测试结果

当查询条件id和username都存在时，控制台打印的sql语句如下：

![image-20210704202137114](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704202137114.png)



当查询条件只有id存在时，控制台打印的sql语句如下：

![image-20210704202410861](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704202410861.png)



#### 总结语法:

~~~xml
<where>：条件标签。如果有动态条件，则使用该标签代替 where 关键字。
        where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。
    
<if>：条件判断标签。
<if test=“条件判断”>
	查询条件拼接
</if>
~~~



###  动态 SQL  之<**foreach>** 

*foreach* 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符，看它多智能！

> 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 *foreach*。
>
> 当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。
>
> 当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

循环执行sql的拼接操作，例如：

```sql
SELECT * FROM student  WHERE id IN (1,2,5)
```



#### 接口方法

```java
//根据多个id查询
List<Student> findByIds(List<Integer> ids);
```



#### XML片段

 ```xml
<select id="findByIds" parameterType="list" resultType="student">
    SELECT * FROM student
    <where>
        <foreach collection="list" open="id IN(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
 ```



#### 测试代码

```java
@Test
public void findByIds() throws Exception {
    //1.加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

    //2.获取SqlSession工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

    //3.通过工厂对象获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession(true);

    //4.获取StudentMapper接口的实现类对象
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

    List<Integer> ids = new ArrayList<>();
    ids.add(1);
    ids.add(2);
    ids.add(3);

    //5.调用实现类的方法，接收结果
    List<Student> list = mapper.findByIds(ids);

    //6.处理结果
    for (Student student : list) {
        System.out.println(student);
    }

    //7.释放资源
    sqlSession.close();
    is.close();
}
```



#### 总结语法:

> <foreach>：循环遍历标签。适用于多个参数或者的关系。
>
> ~~~xml
> <foreach collection=“” open=“” close=“” item=“” separator=“”>
> 	获取参数
> </foreach>
> ~~~
>
> 
>
> **属性：**
>
> - collection：参数容器类型， (list-集合， array-数组)。
> - open：开始的 SQL 语句。
> - close：结束的 SQL 语句。
> - item：参数变量名。
> - separator：分隔符。



### SQL片段抽取

Sql 中可将重复的 sql 提取出来，使用时用 include 引用即可，最终达到 sql 重用的目的

```xml
<sql id="selectStudent">
    SELECT * FROM student
</sql>

<select id="findByCondition" parameterType="student" resultType="student">
    <include refid="selectStudent" />
    <where>
        <if test="id!=null">
            and id=#{id}
        </if>
        <if test="name!=null">
            and name=#{name}
        </if>
    </where>
</select>

<select id="findByIds" parameterType="list" resultType="student">
    <include refid="selectStudent" />
    <where>
        <foreach collection="list" open="id IN(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```

#### 总结语法:

我们可以将一些重复性的 SQL 语句进行抽取，以达到复用的效果。 

~~~xml
- <sql>：抽取 SQL 语句标签。 
- <include>：引入 SQL 片段标签。 
  <sql id=“片段唯一标识”>抽取的 SQL 语句</sql> 
  <include refid=“片段唯一标识”/>
~~~



> **回忆：**
>
> 思考下我们已经学习了哪些MyBatis的标签？
>
> 



## 分页插件

### 分页插件介绍

![1590937779260](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1590937779260.png)

* 分页可以将很多条结果进行分页显示。 

* 如果当前在第一页，则没有上一页。如果当前在最后一页，则没有下一页。 

* 需要明确当前是第几页，这一页中显示多少条结果。    

* MyBatis分页插件总结

  1. 在企业级开发中，分页也是一种常见的技术。而目前使用的 MyBatis 是不带分页功能的，如果想实现分页的 功能，需要我们手动编写 LIMIT 语句。但是不同的数据库实现分页的 SQL 语句也是不同的，所以手写分页 成本较高。这个时候就可以借助分页插件来帮助我们实现分页功能。 

  2. PageHelper：第三方分页助手。将复杂的分页操作进行封装，从而让分页功能变得非常简单。    

     https://github.com/pagehelper/Mybatis-PageHelper

  

### 分页插件的使用

MyBatis可以使用第三方的插件来对功能进行扩展，分页助手PageHelper是将分页的复杂操作进行封装，使用简单的方式即可获得分页的相关数据

**开发步骤：**

①导入与PageHelper的jar包

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.1</version>
</dependency>
```

②在mybatis核心配置文件中配置PageHelper插件

~~~xml
<plugins>
    <!-- 注意：分页助手的插件  配置在通用mapper之前 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor" />
</plugins>
~~~

③测试分页数据获取

~~~java
@Test
public void testPageHelper() throws Exception {

    //1.加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

    //2.获取SqlSession工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

    //3.通过SqlSession工厂对象获取SqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //4.获取StudentMapper
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

    //5.执行查询


    //设置分页参数
    PageHelper.startPage(3,2);

    List<Student> list = mapper.selectAll();

    //6.处理结果
    for (Student stu : list) {
        System.out.println(stu);
    }

    //7.释放资源
    sqlSession.close();
    is.close();


}
~~~



![image-20210704212139555](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704212139555.png)



### 分页插件的参数获取

**获得分页相关的其他参数**：

```java
//5.执行查询
//设置分页参数
PageHelper.startPage(3, 2);

List<Student> list = mapper.selectAll();

//其他分页的数据
PageInfo<Student> pageInfo = new PageInfo<>(list);
System.out.println("总条数：" + pageInfo.getTotal());
System.out.println("总页数：" + pageInfo.getPages());
System.out.println("当前页：" + pageInfo.getPageNum());
System.out.println("每页显示长度：" + pageInfo.getPageSize());
System.out.println("是否第一页：" + pageInfo.isIsFirstPage());
System.out.println("是否最后一页：" + pageInfo.isIsLastPage());
```



![image-20210704212409977](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704212409977.png)



### 分页插件知识小结

​    分页：可以将很多条结果进行分页显示。 

- 分页插件 jar 包： pagehelper-5.1.10.jar、 jsqlparser-3.1.jar 

- <plugins>：集成插件标签。 

- 分页助手相关 API 

  - PageHelper：分页助手功能类。


  - startPage()：设置分页参数 
  - PageInfo：分页相关参数功能类。 
  - getTotal()：获取总条数 
  - getPages()：获取总页数
  - getPageNum()：获取当前页
  - getPageSize()：获取每页显示条数
  - getPrePage()：获取上一页 
  - getNextPage()：获取下一页 
  - isIsFirstPage()：获取是否是第一页 
  - isIsLastPage()：获取是否是最后一页    





> **作业：**
>
> 1. 独立实现MyBatis的Mapper代理开发的增删改查，必须使用到学习的动态SQL标签
> 2. 使用分页插件完成分页功能



## MyBatis的多表操作

### 多表模型介绍

我们之前学习的都是基于单表操作的，而实际开发中，随着业务难度的加深，肯定需要多表操作的。 

*  多表模型分类 一对一：在任意一方建立外键，关联对方的主键。
*  一对多：在多的一方建立外键，关联一的一方的主键。
*  多对多：借助中间表，中间表至少两个字段，分别关联两张表的主键。    



### 多表模型一对一操作

1. 一对一模型： 人和身份证，一个人只有一个身份证

![image-20210704221639473](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704221639473.png)



#### sql语句准备

~~~sql
CREATE TABLE person(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20),
	age INT
);
INSERT INTO person VALUES (NULL,'张三',23);
INSERT INTO person VALUES (NULL,'李四',24);
INSERT INTO person VALUES (NULL,'王五',25);

CREATE TABLE card(
	id INT PRIMARY KEY AUTO_INCREMENT,
	number VARCHAR(30),
	pid INT,
	CONSTRAINT cp_fk FOREIGN KEY (pid) REFERENCES person(id)
);
INSERT INTO card VALUES (NULL,'12345',1);
INSERT INTO card VALUES (NULL,'23456',2);
INSERT INTO card VALUES (NULL,'34567',3);
~~~



#### 目标Sql

查询所有card信息并关联查询person

```sql
SELECT
	c.id cid,
	number,
	pid,
	NAME,
	age 
FROM
	card c,
	person p 
WHERE
	c.pid = p.id
```



#### 实体类

**Person.java**

```java
package com.iflytek.bean;

public class Person {
    private Integer id;     //主键id
    private String name;    //人的姓名
    private Integer age;    //人的年龄

    public Person() {
    }

    public Person(Integer id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```



**Card.java**

```java
package com.iflytek.bean;

public class Card {
    private Integer id;     //主键id
    private String number;  //身份证号

    private Person p;       //所属人的对象

    public Card() {
    }

    public Card(Integer id, String number, Person p) {
        this.id = id;
        this.number = number;
        this.p = p;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    public Person getP() {
        return p;
    }

    public void setP(Person p) {
        this.p = p;
    }

    @Override
    public String toString() {
        return "Card{" +
                "id=" + id +
                ", number='" + number + '\'' +
                ", p=" + p +
                '}';
    }
}

```



#### XML文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.table01.OneToOneMapper">
    <!--配置字段和实体对象属性的映射关系-->
    <resultMap id="oneToOne" type="card">
        <id column="cid" property="id" />
        <result column="number" property="number" />
        <!--
            association：配置被包含对象的映射关系
            property：被包含对象的变量名
            javaType：被包含对象的数据类型
        -->
        <association property="p" javaType="person">
            <id column="pid" property="id" />
            <result column="name" property="name" />
            <result column="age" property="age" />
        </association>
    </resultMap>

    <select id="selectAll" resultMap="oneToOne">
        SELECT c.id cid,number,pid,NAME,age FROM card c,person p WHERE c.pid=p.id
    </select>
</mapper>
~~~



#### Mapper接口



```java
package com.iflytek.mapper;

import com.iflytek.bean.Card;

import java.util.List;

public interface OneToOneMapper {
    //查询全部
    List<Card> selectAll();
}

```



#### 测试代码

```java
package com.iflytek.test;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.iflytek.bean.Card;
import com.iflytek.bean.Student;
import com.iflytek.mapper.OneToOneMapper;
import com.iflytek.mapper.StudentMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class OneToOneTest {

    @Test
    public void selectAll() throws Exception {
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToOneMapper接口的实现类对象
        OneToOneMapper mapper = sqlSession.getMapper(OneToOneMapper.class);

        //5.调用实现类的方法，接收结果
        List<Card> list = mapper.selectAll();

        //6.处理结果
        for (Card c : list) {
            System.out.println(c);
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }

}

```



#### 运行结果

![image-20210704222101721](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704222101721.png)



#### 一对一配置总结:

~~~xml-dtd
<resultMap>：配置字段和对象属性的映射关系标签。
    id 属性：唯一标识
    type 属性：实体对象类型
<id>：配置主键映射关系标签。
<result>：配置非主键映射关系标签。
    column 属性：表中字段名称
    property 属性： 实体对象变量名称
<association>：配置被包含对象的映射关系标签。
    property 属性：被包含对象的变量名
    javaType 属性：被包含对象的数据类型
~~~



### 多表模型一对多操作



一对多模型： 一对多模型：班级和学生，一个班级可以有多个学生。    

![image-20210704223958298](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20210704223958298.png)



#### sql语句准备

```sql
CREATE TABLE classes(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(20)
);
INSERT INTO classes VALUES (NULL,'黑马一班');
INSERT INTO classes VALUES (NULL,'黑马二班');


CREATE TABLE student2(
	id INT PRIMARY KEY AUTO_INCREMENT,
	NAME VARCHAR(30),
	age INT,
	cid INT,
	CONSTRAINT cs_fk FOREIGN KEY (cid) REFERENCES classes(id)
);
INSERT INTO student2 VALUES (NULL,'张三',23,1);
INSERT INTO student2 VALUES (NULL,'李四',24,1);
INSERT INTO student2 VALUES (NULL,'王五',25,2);
INSERT INTO student2 VALUES (NULL,'赵六',26,2);
```



#### 目标Sql

```sql
SELECT
	c.id cid,
	c.NAME cname,
	s.id sid,
	s.NAME sname,
	s.age sage 
FROM
	classes c,
	student2 s 
WHERE
	c.id = s.cid
```



#### 实体类

Student2.java

```java
package com.iflytek.bean;

import java.util.List;

public class Student2 {
    private Integer id;     //主键id
    private String name;    //学生姓名
    private Integer age;    //学生年龄

    public Student2() {
    }

    public Student2(Integer id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```



Classes.java

```java
package com.iflytek.bean;

import java.util.List;

public class Classes {
    private Integer id;     //主键id
    private String name;    //班级名称

    private List<Student2> students; //班级中所有学生对象

    public Classes() {
    }

    public Classes(Integer id, String name, List<Student2> students) {
        this.id = id;
        this.name = name;
        this.students = students;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Student2> getStudents() {
        return students;
    }

    public void setStudents(List<Student2> students) {
        this.students = students;
    }

    @Override
    public String toString() {
        return "Classes{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", students=" + students +
                '}';
    }
}

```



#### OneToManyMapper配置文件



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.iflytek.mapper.OneToManyMapper">
    <resultMap id="oneToMany" type="classes">
        <id column="cid" property="id"/>
        <result column="cname" property="name"/>

        <!--
            collection：配置被包含的集合对象映射关系
            property：被包含对象的变量名
            ofType：被包含对象的实际数据类型
        -->
        <collection property="students" ofType="student2">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="sage" property="age"/>
        </collection>
    </resultMap>
    <select id="selectAll" resultMap="oneToMany">
        SELECT c.id cid,c.name cname,s.id sid,s.name sname,s.age sage FROM classes c,student2 s WHERE c.id=s.cid
    </select>
</mapper>
```



#### 接口

```java
package com.iflytek.mapper;

import com.iflytek.bean.Classes;

import java.util.List;

public interface OneToManyMapper {
    //查询全部
    List<Classes> selectAll();
}

```



#### 测试代码

```java
package com.iflytek.test;

import com.iflytek.bean.Classes;
import com.iflytek.bean.Student;
import com.iflytek.bean.Student2;
import com.iflytek.mapper.OneToManyMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.List;

public class OneToManyTest {
    @Test
    public void selectAll() throws Exception{
        //1.加载核心配置文件
        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");

        //2.获取SqlSession工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        //3.通过工厂对象获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);

        //4.获取OneToManyMapper接口的实现类对象
        OneToManyMapper mapper = sqlSession.getMapper(OneToManyMapper.class);

        //5.调用实现类的方法，接收结果
        List<Classes> classes = mapper.selectAll();

        //6.处理结果
        for (Classes cls : classes) {
            System.out.println(cls.getId() + "," + cls.getName());
            List<Student2> students = cls.getStudents();
            for (Student2 student : students) {
                System.out.println("\t" + student);
            }
        }

        //7.释放资源
        sqlSession.close();
        is.close();
    }
}

```
