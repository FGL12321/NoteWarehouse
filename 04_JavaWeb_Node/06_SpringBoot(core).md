## SpringBoot

### 1.快速入门

```
<!-- springboot工程要继承的父工程 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

<dependencies>
    <!--web开发的起步依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 2.运行文件编写

运行类

```
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public  String hello(){
        return "hello Spring!";
    }
}
```

启动类

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

访问地址：http://localhost:8080/hello

### 3.配置文件编写

1.maven工程的resource文件夹中创建application.properties文件（高）

```
server.port=8081
```

2.创建application.yml（中）

```
server:
  port: 8082
```

3.创建application.yaml（低）

```
server:
  port: 8083
```

### 4.注解的使用

@SpringBootApplication：标注SprinBoot启动类

@RequestMapping("/hello4")：用于将任意HTTP 请求映射到控制器方法上

@RestController：作用等同于@Controller + @ResponseBody

@Value("${person.age}")：等同于赋值操作

@Autowired：自动装配

@ConfigurationProperties(prefix = "person")：对类注入数据



@Mapper：目的就是为了不再写mapper映射文件

>@Select("select * from t_user")   #配合使用

@controller：  controller控制器层（注入服务）

@service ：   service服务层（注入dao）

@repository ： dao持久层（实现dao访问）

@component： 标注一个类为Spring容器的Bean，把普通pojo实例化到spring容器中

@Bean：注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理



### 5.YAML基本语法

```yaml
server:
  port: 8082

name: abc
person:
  name: ${name} #参数引用
  age: 20

#对象的行内写法
person2: {name: zhangsan,age: 20}

#数组
address:
  - beijing
  - shanghai
address2:[beijing,shanghai]

#纯量
msg1: 'hello \n world' #不会识别转义字符
msg2: "hello \n world" #会识别转义字符
```

### 6.yml内容注入方式

**1） @Value：将一个值赋值给当前属性**

```
@Value("${person.age}")
    private int age;
```

**2） Environment：将数据注入到环境对象中**

```
@Autowired
private Environment env;
```

**3） @ConfigurationProperties ：将数据注入到对象类中**

```
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private int age;..........................
```

### 7.profile的使用和激活

profile是用来完成不同环境下，配置动态切换功能的

**1.多文件方式**

```
spring.profiles.active=dev #和文件后缀相同即可
```

**2.yml多文档方式**

```
---
server:
  port: 8081

spring:
  profiles: dev
---
server:
  port: 8082

spring:
  profiles: test
---
server:
  port: 8083

spring:
  profiles: pro
---
spring:
  profiles:
    active: dev
```

**激活方式：**

1.在application.yml中进行设置

2.虚拟机参数：虚拟机参数：在VM options 指定：-Dspring.profiles.active=dev

3.命令行参数：命令行参数：java –jar xxx.jar --spring.profiles.active=dev

![image-20230308213813034](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230308213813034.png)



### 8.内部配置加载顺序

Springboot程序启动时，会从以下位置加载配置文件：

1. **file:./config/：当前项目下的/config目录下**

2. **file:./ ：当前项目的根目录**

3. **classpath:/config/：classpath的/config目录**

​		resources的config目录下

4. **classpath:/ ：classpath的根目录**

​		resouces目录下

**加载顺序为上文的排列顺序，高优先级配置的属性会生效**

### 9.外部配置的加载顺序

命令行的方式进行配置

### 10.SpringBoot整合Junit

① 搭建SpringBoot工程

② 引入starter-test起步依赖

③ 编写测试类

④ 添加测试相关注解

​			@RunWith(SpringRunner.class)

​			@SpringBootTest(classes = 启动类.class)

⑤ 编写测试方法

```
<dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.0</version>
</dependency>
```

```
@RunWith(SpringRunner.class) 
@SpringBootTest(classes = SpringbootTestApplication.class) #启动类文件
```

### 11.SpringBoot整合Redis

① 搭建SpringBoot工程

② 引入redis起步依赖

③ 配置redis相关属性

④ 注入RedisTemplate模板

⑤ 编写测试方法，测试

```
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 12.SpringBoot整合MyBatis

① 搭建SpringBoot工程

② 引入mybatis起步依赖，添加mysql驱动

③ 编写DataSource和MyBatis相关配置

④ 定义表和实体类

⑤ 编写dao和mapper文件/纯注解开发

⑥ 测试

**引入相关依赖**

```
       <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <!--<scope>runtime</scope>-->
        </dependency>
```

**指定mybatis映射文件和内容**

```
# datasource
spring:
  datasource:
    url: jdbc:mysql:///springboot?serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

# mybatis
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径
  type-aliases-package: com.itheima.springbootmybatis.domain
  # config-location:  # 指定mybatis的核心配置文件
```

### 13.SpringBoot自动配置Condition（条件功能）

**Condition 是在Spring 4.0 增加的条件判断功能，通过这个可以功能可以实现选择性的创建 Bean 操作。**

**SpringBoot是如何知道要创建哪个Bean的？比如SpringBoot是如何知道要创建RedisTemplate的？**

==**@Conditional**：需要传入字节码文件==

用于判断是否创建bean对象



 **自定义条件：**

① 定义条件类：自定义类实现Condition接口，重写 matches 方法，在 matches 方法中进行逻辑判断，返回

boolean值 。 matches 方法两个参数：

• context：上下文对象，可以获取属性值，获取类加载器，获取BeanFactory等。

• metadata：元数据对象，用于获取注解属性。

② 判断条件： 在初始化Bean时，使用 @Conditional(条件类.**class**)注解



 **SpringBoot 提供的常用条件注解：**

• ConditionalOnProperty：判断配置文件中是否有对应属性和值才初始化Bean

• ConditionalOnClass：判断环境中是否有对应字节码文件才初始化Bean

• ConditionalOnMissingBean：判断环境中没有对应Bean才初始化Bean





==创建bean的几种方式==

**1.Spring XML方式配置，该方式用于在纯Spring 应用中**

```
@Conditional(ClassCondition.class)
```



**2.@Component,@Service,@Controler,@Repository注解标注后悔创建bean**

- `@Componet`一般的组件
- `@Service`是Service层组件
- `@Bean`这个要和@Configuration一块使用
- `@Controller`是用在SpringMVC控制层
- `@Repository`是数据访问层

@Configuration 标识这是一个Spring Boot 配置类，其将会扫描该类中是否存在@Bean 注解的方法**



### 14.如何切换内置服务器

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!--排除tomcat依赖-->
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!--引入jetty的依赖-->
        <dependency>
            <artifactId>spring-boot-starter-jetty</artifactId>
            <groupId>org.springframework.boot</groupId>
</dependency>
```

