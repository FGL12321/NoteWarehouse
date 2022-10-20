[TOC]



# 登录注册案例—剖析SSM组件—超详说明



​		**本案例主要围绕SSM登录注册案例，说明SSM的jar包用途以及SSM的整个运行流程，帮助你了解关于SSM的整体架构知识，希望对你有所帮助。**

如需获取更多源码，笔记，教程，请访问本人仓库，你的关注是我创作的动力！

链接：[https://gitee.com/fanggaolei/learning-notes-warehouse](https://gitee.com/fanggaolei/learning-notes-warehouse)

![image-20220822212203990](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822212203990.png)

# 1.0 材料准备

## 1.1 数据库

![image-20220822212632666](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822212632666.png)

## 1.2 工程准备

![image-20220821192006881](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220821192006881.png)

## 1.3Tomcat准备

![image-20220821192232878](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220821192232878.png)

# 2.0 SSM环境配置

## 2.1jar包配置及用途说明

**1.常用Spring jar包说明**

```
 <!-- Spring相关  -->
     <dependency>
       <!-- Spring框架基本核心  -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>4.2.4.RELEASE</version>
    </dependency>
    
    <dependency>
       <!-- Spring框架IOC -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>4.2.4.RELEASE</version>
    </dependency>
    
     <dependency>
    <!--  Spring面向切面编程 AOP  -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>4.3.7.RELEASE</version>
    </dependency>
    
    <dependency>
     <!-- 为Spring 核心提供了大量扩展，使用Spring ApplicationContext特性时所需的全部类 -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.2.4.RELEASE</version>
    </dependency>
    
    <dependency>
      <!--  Spring表达式解析 -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>4.2.4.RELEASE</version
    </dependency>
      
    <dependency>
       <!--  Spring-Jdbc  事物控制-->
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.3.7.RELEASE</version>
    </dependency>
    
    <dependency>
      <!-- Spring-test用于测试  -->
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.3.7.RELEASE</version>
    </dependency>
    
    <dependency>
     <!-- mybatis-spring整合包，这样就可以通过spring配置bean的方式进行mybatis配置了，
        不然需要单独使用mybatis的配置-->
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>

```

**2.Mybatis Jar包说明**

```
<dependencies> 

<!-- Mybatis核心 --> 
<dependency> 
	<groupId>org.mybatis</groupId> 
	<artifactId>mybatis</artifactId> 
	<version>3.5.7</version> 
</dependency> 
```

**3.0 SpringMvc jar包说明**

```
    <dependency>
         <!--springMVC核心jar包-->
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>4.3.7.RELEASE</version>
    </dependency>

    <dependency>
         <!--引入servlet依赖-->
         <groupId>javax.servlet</groupId>
         <artifactId>javax.servlet-api</artifactId>
         <version>3.1.0</version>
         <scope>provided</scope>
    </dependency>
    
    <dependency>
         <!--jstl标签库，开源jsp标签库-->
         <groupId>javax.servlet</groupId>
         <artifactId>jstl</artifactId>
         <version>1.2</version>
    </dependency>
```

**4.0其他Jar包**

```
<dependency>
	<!--数据库依赖-->
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>
<dependency>
	<!--单元测试依赖-->
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>

 <dependency>
   <!--更强的单元测试依赖-->
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>RELEASE</version>
    <scope>compile</scope>
 </dependency>

<dependency>
    <!--基于Java平台的一套数据处理工具,很方便完成 Java对象 和 Json对象的转换-->
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.8</version>
</dependency>

<dependency>
     <!--c3p0数据库连接池-->
	 <groupId>com.mchange</groupId>
	 <artifactId>c3p0</artifactId>
	 <version>0.9.5.2</version>
</dependency>	
```

## 2.2项目pom.xml配置

```
<dependencies>
  <!--1.数据库依赖-->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
  </dependency>

  <!--2.引入mybatis依赖-->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
  </dependency>

  <!-- 返回json字符串的支持-->
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.8</version>
  </dependency>


  <!--3.引入单元测试依赖-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.3.7.RELEASE</version>
  </dependency>

  <!--4.引入日志依赖-->
  <dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
  </dependency>

  <!--5.mybais相关依赖 用于自动逆向工程-->
  <dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.5</version>
  </dependency>

  <!--6.springMVC相关依赖  SpringMVC核心包  间接导入了启动mvc框架的所有依赖-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.7.RELEASE</version>
  </dependency>

  <!--7.引入相关servlet依赖-->
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
  </dependency>

  <!--  8.Spring-Jdbc  事物控制-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.7.RELEASE</version>
  </dependency>

  <!-- 9.Spring-test  -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.3.7.RELEASE</version>
  </dependency>

  <!--  10.Spring面向切面编程 AOP  -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.7.RELEASE</version>
  </dependency>

  <!-- 11.mybatis整合spring的适配包-->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.1</version>
  </dependency>

  <dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>RELEASE</version>
    <scope>compile</scope>
  </dependency>

  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>compile</scope>
  </dependency>
  
  <dependency>
     <!--c3p0数据库连接池-->
	 <groupId>com.mchange</groupId>
	 <artifactId>c3p0</artifactId>
	 <version>0.9.5.2</version>
</dependency>	
</dependencies>
```

## 2.3项目Web.xml及SSM配置文件

接下来我们配置以下3个内容

![image-20220822212435322](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822212435322.png)

**1.web.xml**

当服务启动时首先加载web.xml中的资源文件，如前端控制器，字符编码过滤器等。

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
           version="2.5">


    <!-- 配置 Spring MVC 的前端控制器 -->
    <welcome-file-list>
        <welcome-file>log.jsp</welcome-file>
    </welcome-file-list>


    <!--配置文件加载-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <servlet>
        <!--servlet-name  名称-->
        <servlet-name>DispatcherServlet</servlet-name>
        <!--路径  class-->
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <!-- 配置初始化参数，用于读取 Spring MVC 的配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <!-- 应用加载时创建    优先级别  1最高级别-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern> <!--路径  什么情况下会开启-->
    </servlet-mapping>


    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class> org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

**2.application-dao.xml：**

用于编写Mybatis的设置，此配置文件是为了将对数据库的操作转换为mapper映射器注入到spring中。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>


    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/><!--开启驼峰命名规则-->
    </settings>
    <typeAliases>
        <package name="com.fang.log_reg.bean"/>  <!--配置别名-->
    </typeAliases>

</configuration>

```

**3.application-service.xml：**

用于配置Spring的IOC和AOP以及数据库连接等。

```
    <!-- 1.扫描器，不扫控制器-->
    <context:component-scan base-package="com.fang.ssmhe">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <context:component-scan base-package="com.fang.ssmhe.service"/>
    <!--  Spring的配置文件，这里主要配置和业务逻辑有关的  -->
    <!-- =================== 数据源，事务控制，xxx ================ -->
    <context:property-placeholder/>
    <bean id="pooledDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://117.33.150.3:3306/ssm_crud?3useUnicode=true?characterEncoding=utf8"/>
        <property name="user" value="root"/>
        <property name="password" value="fgl123"/>
    </bean>


    <!-- ================== 配置和MyBatis的整合===============  -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--  指定mybatis全局配置文件的位置  -->
        <property name="configLocation" value="classpath:mybatis.xml"/>
        <property name="dataSource" ref="pooledDataSource"/>
        <!--  指定mybatis，mapper文件的位置  -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>
    <!--  配置扫描器，将mybatis接口的实现加入到ioc容器中  -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 扫描所有dao接口的实现，加入到ioc容器中  -->
        <property name="basePackage" value="com.fang.ssmhe.dao"/>
    </bean>
    <!--  配置一个可以执行批量的sqlSession  -->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
        <constructor-arg name="executorType" value="BATCH"/>
    </bean>


    <!--  ===============事务控制的配置 ================ -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 控制住数据源   -->
        <property name="dataSource" ref="pooledDataSource"/>
    </bean>
    <!-- 开启基于注解的事务，使用xml配置形式的事务（必要主要的都是使用配置式）   -->
    <aop:config>
        <!--  切入点表达式  -->
        <aop:pointcut expression="execution(* com.fang.ssmhe.service..*(..))" id="txPoint"/>
        <!--  配置事务增强  -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
    </aop:config>
    <!-- 配置事务增强，事务如何切入   -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--  所有方法都是事务方法  -->
            <tx:method name="*"/>
            <!-- 以get开始的所有方法   -->
            <tx:method name="get*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!--  Spring配置文件的核心点（数据源、与mybatis的整合，事务控制）  -->
</beans>
    
```

**4.spring-mvc.xml**

用于配置视图解析和注解驱动

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">


    <context:component-scan base-package="com.fang.log_reg" use-default-filters="false">
        <!--  只扫描控制器-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>


    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!-- 两个标准配置   -->
    <!--  将springmvc不能处理的请求交给tomcat  -->
    <mvc:default-servlet-handler/>
    <!--  能支持springmvc更高级的一些功能，JSR303校验，快捷的ajax...映射动态请求  -->
    <mvc:annotation-driven/>
</beans>
```

如果你已经完成上述内容，那么恭喜你，你已经完成了全部的配置环节

# 3.0项目业务配置-四个目录

![image-20220822212805390](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822212805390.png)

## 3.1bean目录User

```
package com.fang.log_reg.bean;


public class User {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

## 3.2controller目录UserController

```
package com.fang.log_reg.controller;


import com.fang.log_reg.service.UserService;
import com.fang.log_reg.bean.User;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class UserController {

    @Autowired //为我们注入一个定义好的 bean
    private UserService userService;

    @RequestMapping("/golog")
    public String log(User user, Model model) {
        User u = userService.findOne(user);
        model.addAttribute("user", u);
        if (u != null) {
            return "logok";
        } else
            return "logno";
    }

    @RequestMapping("/goreg")
    public String reg(User user) {
        User u = userService.checkReg(user.getUsername());
        if (u == null) {
            userService.addOne(user);
            return "regok";
        }
        return "regno";
    }

    @RequestMapping("/reg")
    public String reg(String string){
        return "reg";
    }
}
```

## 3.3dao目录UserMapper接口

```
package com.fang.log_reg.dao;


import com.fang.log_reg.bean.User;

public interface UserMapper {

    User findOne(User user);

    //注册时的重名校验
    User checkReg(String username);

    //用户注册
    int addOne(User user);
}

```

## 3.4service目录UserService

```
package com.fang.log_reg.service;

import com.fang.log_reg.bean.User;
import com.fang.log_reg.dao.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;



@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;


    public User findOne(User user) {
        return userMapper.findOne(user);
    }


    public int addOne(User user) {
        return userMapper.addOne(user);

    }

    public User checkReg(String username) {
        return userMapper.checkReg(username);
    }

}
```

## 3.4配置mapper文件



![image-20220822213053012](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822213053012.png)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.fang.log_reg.dao.UserMapper">

    <!--该方法为注册方法-->
    <select id="findOne" parameterType="User" resultType="User">
        SELECT * FROM  user where username=#{username} and password=#{password}
    </select>

    <insert id="addOne" parameterType="User" >
        insert into user (username,password) values(#{username},#{password})
    </insert>

    <select id="checkReg" resultType="User">
        select * from   user where username=#{username}
    </select>
</mapper>
```

# 4.0配置6个界面

![image-20220822213225316](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220822213225316.png)

## 4.1log.jsp

```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <style type="text/css">
        div {
            width: 300px;
            height: 200px;
            margin-left: auto;
            margin-right: auto;
        }
    </style>
</head>
<body>
<div>
    <form action="golog" method="post">
        <table border="1">
            <tr>
                <td>用户名:</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>密&emsp;码:</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td colspan="2" style="text-align: center;"><input
                        type="submit" value="登录"></td>
            </tr>
            <tr>
                <td colspan="2" style="text-align: center;"><a href="http://localhost:8081/login_registration_SSM_war/reg">还没有账号？点此注册</a></td>
            </tr>
        </table>
    </form>
</div>
</body>
</html>
```

## 4.2logno与logok

```
<!-- logno.jsp -->
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
</head>
<body>
<h1>用户名或者密码错误</h1>
</body>
</html>



<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
</head>
<body >
<h1>如果你能看见我，说明你成功了</h1>
<hr>
您的用户名是:${user.username}<br>
您的密码是:${user.password}<br>
</body>
</html>
```



## 4.3reg与regno、regok

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>用户注册</title>
    <style type="text/css">
        div {
            width: 300px;
            height: 200px;
            margin-left: auto;
            margin-right: auto;
        }
    </style>
</head>
<body>
<div>
    <form action="goreg" method="post">
        <table border="1">
            <tr>
                <td>用户名:</td>
                <td><input type="text" name="username" ></td>
            </tr>
            <tr>
                <td>密&emsp;码:</td>
                <td><input type="password" name="password" ></td>
            </tr>

            <tr>
                <td colspan="2" style="text-align: center;"><input type="submit" value="注册"></td>
            </tr>

        </table>
    </form>
</div>
</body>
</html>
```

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
</head>
<body>
<h1>该用户名已被注册</h1>
</body>
</html>

```

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
</head>
<body >
<h1>注册成功如果你能看见我，说明你成功了</h1><br>
<a href="http://localhost:8081/login_registration_SSM_war/">点此返回登录界面</a>
</body>
</html>

```

**恭喜你完成项目配置！！！**

