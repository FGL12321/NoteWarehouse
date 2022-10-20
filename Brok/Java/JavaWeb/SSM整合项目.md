# SSM整合项目—员工信息管理系统

[TOC]

**项目源码获取：**https://gitee.com/fanggaolei/SSM



### 项目基本信息

**功能点：**

1. 登录注册功能

2. 可执行对员工个人信息的增删改查功能

**项目开发工具：**

1. IDEA：java集成开发工具

2. Tomcat8.5.75：项目运行服务器

3. MySQL5.7：项目数据库

**主要技术点：**

4. SSM：javaWeb项目开发框架

5. BootStrap：响应式前端框架

6. Html+css+js：前端视图编写

7. pageHelper：分页插件

8. c3p0：数据校验
9. MyBatis Generator：逆向工程

### **编写说明**

### 一、问题描述及分析

   	 本项目是基于SSM框架采用MVC架构的员工信息管理系统，SSM（Spring+SpringMVC+MyBatis）框架集由Spring、MyBatis两个开源框架整合而成（SpringMVC是Spring中的部分内容）。常作为数据源较简单的web项目的框架。使用本开发框架进一步简化了繁琐的配置工作，SSM框架提高了开发人员的工作效率，及时的开发出具有良好的时效性的优秀的软件。

### 二、功能模块

分析及数据结构描述

**项目数据结构说明：**

  **本项目共有三张数据表：相关内容如下图所示**

  **①：tbl_admin:用于存储系统管理员信息**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps1.jpg) 

  ②：tbl_dept:用于存储部门信息

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps2.jpg) 

  ③：tbl_emp:用于存储员工信息

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps3.jpg) 

**功能模块一：登录注册功能**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps4.jpg" alt="img" style="zoom:150%;" /> 

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps5.jpg" alt="img" style="zoom:150%;" /> 

 

**功能模块二：查询功能**

**• 1、index.jsp页面直接发送ajax请求进行员工分页数据的查询**

**• 2、服务器将查出的数据，以json字符串的形式返回给浏览器**

**• 3、浏览器收到js字符串。可以使用js对json进行解析，使用js通过**

**dom增删改改变页面。**

**• 4、返回json。实现客户端的无关性。**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps6.jpg) 

**功能模块三：员工添加功能**

**1、在index.jsp页面点击”新增” • 2、弹出新增对话框**

**• 3、去数据库查询部门列表，显示在对话框中**

**• 4、用户输入数据，并进行校验**

**• jquery前端校验，ajax用户名重复校验，重要数据（后端校验(JSR303)，唯一约束）；**

**• 5、完成保存**

**• URI:**

**• /emp/{id} GET 查询员工**

**• /emp POST 保存员工**

**• /emp/{id} PUT 修改员工**

**• /emp/{id} DELETE 删除员工**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps7.jpg" alt="img" style="zoom:150%;" /> 

 

**功能模块四：员工修改功能**

**1、点击编辑**

**• 2、弹出用户修改的模态框（显示用户信息）**

**• 3、点击更新，完成用户修改**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps8.jpg) 

**功能模块五：员工删除功能**

**• 1、单个删除**

**• URI:/emp/{id} DELETE**

**• 2、批量删除**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps9.jpg) 

 

### 三、主要算法或流程描述

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps10.jpg) 

 

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps11.jpg) 

页面发送请求给控制器（Controller)，控制器调用业务层（Service）处理逻辑，逻辑层向持久层（Dao）发送请求，持久层与数据库交互，后将结果返回给业务层，业务层将处理逻辑发送给控制器，控制器再调用视图展现数据。

调用顺序：web页面发出请求 → web.xml（配置前端控制器）→ applicationContext.xml (开启注解扫描）→ Controller（@Resource注入service，调用Service） → Service（注入dao,调用dao) →DAO（Mapper接口映射器） → 完成具体的数据库操作

其中Service层应该既调用DAO层的接口，又要提供接口给Controller层的类来进行调用，它刚好处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。

### 四、系统使用说明

用户在使用本系统时，首先进行注册，注册成功后返回登录界面进行登录操作，登录成功后，用户可点击相关的按钮，对项目数据表进行增删改查操作。

 

### 五、问题及解决办法

1.在配置Tomcat时无法正常运行，经检查错误原因是配置的tomcat版本不一致，更换为tomcat8.5后可正常运行

2.在进行删除操作时，发现无法进行正常的删除，经检查，BookMapper.xml文件与BookMapper.java删除方法名不一致，将方法名修改一致后，可以进行正常的删除操作。

3.在查看日志的过程中，发现日志信息无法输出，经检查pom.xml过程中，日志相关的依赖没有正常导入，重新导入后，日志信息正常输出。

4.在进行登录操作的过程中，报错为Mapper.xml出现错误，经检查后，发现字段名与数据库中的字段名不符合，导致数据无法进行校验，修改后，可正常执行登录操作。

5.在使用Mybatis的逆向工程的时候，发现mapper.xml文件中的sql语句出现重复的现象，经过排查发现，错误原因在于两次运行了逆向工程，导致所有sql语句出现重复，将所有生成文件删除后，重写进行逆向工程，数据正确。

 

### 六、项目总结

  	 通过本项目的编写，我从本项目中学到了相关的知识，虽然对SSM框架有一定的了解，但真正动手写代码的时候还是会出现很多的问题，例如对SSM进行整合的过程中20多项相关配置，6个配置文件，很容易出现差错，这个时候就要及时回忆所学习的每一个框架中所包含的jar包，以及每一项配置的作用，这样在配置的过程中，才不会出现遗漏的情况，在实际开发的过程中也会遇到很多棘手的问题，在进行第一个模块开发的过程中，由于对ssm框架代码编写流程的不熟悉，以及对spring运行机制的不理解，导致会出现很多的配置错误，这个时候需要及时复盘相关内容以及知识点，查漏补缺之后，这样才能真正的去理解所写代码的含义。经过各种各样的报错信息，以及开发后本项目终于可以正常完成，同时在今后的学习过程中应该注意复习，更加深入的去理解开发框架的底层原理，这样在使用框架进行开发的时候，才能游刃有余。