[TOC]



# Hive学习笔记

# [第 1 章 Hive 基本概念	](#_Toc1180 )（1-5）

## 1.1 **什么是** Hive

**Hive：由 Facebook 开源用于解决海量结构化日志的数据统计工具。**

**Hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类 SQL 查询功能。**



**1）** **Hive** **本质**：**将 HQL 转化成 MapReduce 程序**

**（1） Hive 处理的数据存储在 HDFS**

**（2） Hive 分析数据底层的实现是 MapReduce**

**（3） 执行程序运行在 Yarn 上**



## 1.2Hive的优缺点

**优点：**

- 采用SQL语法，提供快速开发的能力
- 避免写MR
- 执行延迟高，常用语数据分析
- 优势在于处理大数据
- 支持自定义函数

**缺点：**

-  迭代式算法无法表达
-  数据挖掘方面不擅长，由于 MapReduce 数据处理流程的限制，效率更高的算法却无法实现。
-  Hive 自动生成的 MapReduce 作业，通常情况下不够智能化
-  Hive 调优比较困难，粒度较粗

## 1.3Hive的原理

![1658367115977](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658367115977.png)

**1）用户接口：Client**

​		CLI（command-line interface）、JDBC/ODBC(jdbc 访问 hive)、WEBUI（浏览器访问 hive）

**2）元数据：Metastore**

​		元数据包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；

**3）Hadoop**

使用 HDFS 进行存储，使用 MapReduce 进行计算。

**4）驱动器：Driver**

（1）**解析器（**SQL Parser）：将 SQL 字符串转换成抽象语法树 AST，这一步一般都用第三方工具库完成，比如 antlr；对 AST 进行语法分析，比如表是否存在、字段是否存在、SQL 语义是否有误。

（2）**编译器**（Physical Plan）：将 AST 编译生成逻辑执行计划。

（3）**优化器**（Query Optimizer）：对逻辑执行计划进行优化。

（4）**执行器**（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于 Hive 来说，就是 MR/Spark。



## 1.4Hive与数据库的比较

**Hive 和数据库除了拥有类似的查询语言，再无类似之处。**

### 1.4.1查询语言

​		由于 SQL 被广泛的应用在数据仓库中，因此，专门针对 Hive 的特性设计了**类 SQL 的查询语言 HQL**。熟悉 SQL 开发的开发者可以很方便的使用 Hive 进行开发

### 1.4.2数据更新

​		由于 Hive 是针对数据仓库应用设计的，而数据仓库的内容是读多写少的。因此，**Hive 中不建议对数据的改写**，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用INSERT INTO … VALUES 添加数据，使用UPDATE … SET 修改数据。

### 1.4.3执行延迟

​		Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致 Hive 执行延迟高的因素是MapReduce 框架。数据库的执行延迟较低。

### 1.4.4数据规模

​		由于 Hive 建立在集群上并可以利用 MapReduce 进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

# [第 2 章 Hive 安装	](#_Toc13557 )（6-17）

## 2.1Hive的安装

**1）把 apache-hive-3.1.2-bin.tar.gz 上传到 linux 的/opt/software 目录下**

**2）解压 apache-hive-3.1.2-bin.tar.gz 到/opt/module/目录下面**

```
[atguigu@hadoop102 software]$ tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

**3）修改 apache-hive-3.1.2-bin.tar.gz 的名称为 hive**

```
mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
```

**4）修改/etc/profile.d/my_env.sh，添加环境变量**

```
sudo vim /etc/profile.d/my_env.sh
```

**5）添加内容**

```java
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin


source /etc/profile  //记得让环境变量生效
```

**6）解决日志 Jar 包冲突**

```
[atguigu@hadoop102 software]$ mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
```

**7）初始化元数据库**

```
[atguigu@hadoop102 hive]$ bin/schematool -dbType derby -initSchema
```

## 2.2启动并使用Hive

**1）启动 Hive**

```
[atguigu@hadoop102 hive]$ bin/hive
```

**2）使用 Hive**

```
hive> show databases;
hive> show tables;
hive> create table test(id int);
hive> insert into test values(1);
hive> select * from test;
```

**3）在 CRT 窗口中开启另一个窗口开启 Hive，在/tmp/<font color="red">自己的用户名</font> 目录下监控 hive.log 文件**

我们启动两个hive客户端

监控日志

```
tail -f hive.log
```

```python
#derby默认只能单用户使用，因此开启两个客户端会报错
Caused by: ERROR XSDB6: Another instance of Derby may have already booted 
the database /opt/module/hive/metastore_db.
 at 
org.apache.derby.iapi.error.StandardException.newException(Unknown 
Source)
 at 
org.apache.derby.iapi.error.StandardException.newException(Unknown
Source)
 at 
org.apache.derby.impl.store.raw.data.BaseDataFileFactory.privGetJBMSLockO
nDB(Unknown Source)
 at 
org.apache.derby.impl.store.raw.data.BaseDataFileFactory.run(Unknown 
Source)
...
```

​		<font color="red">原因在于 Hive 默认使用的元数据库为 derby，开启 Hive 之后就会占用元数据库，且不与其他客户端共享数据，所以我们需要将 Hive 的元数据地址改为 MySQL。</font>

## 2.3hive的元数据与表数据

​		大家都知道使用Hive的时候，Hive的元信息和数据是分开存储的。

​		数据是存储在HDFS里面的，元信息是存储在结构化数据库中的，类似于mysql,,postgresqli这种。

​		原因就是元信息常被修改，而且HDFS基本属性是不支持修改的，所以需要将元信息存储到结构化（支持update)的数据库中。



**元数据的说明**

> ​		元数据（Meta Date），主要记录数据仓库中模型的定义、各层级间的映射关系、监控数据仓库的数据状态及 ETL 的任务运行状态。一般会通过元数据资料库（Metadata Repository）来统一地存储和管理元数据，其主要目的是使数据仓库的设计、部署、操作和管理能达成协同和一致。

> ​		**元数据包括表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等。**

> ​		元数据包含用Hive创建的database、table等的元信息。元数据存储在关系型数据库中。如Derby、MySQL等。



- 客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。



​		**为了解决这一问题，我们就需要采用hive与Mysql相结合的方式进行hive数据元数据的存储与表中数据存储的分离！**



<font color="red">**解决方案：本地安装mysql 替代derby存储元数据**</font>>

​		不再使用内嵌的Derby作为元数据的存储介质，而是使用其他数据库比如MySQL来存储元数据。hive服务和metastore服务运行在同一个进程中，mysql是单独的进程，可以同一台机器，也可以在远程机器上。
这种方式是一个多用户的模式，运行多个用户client连接到一个数据库中。这种方式一般作为公司内部同时使用Hive。每一个用户必须要有对MySQL的访问权利，即每一个客户端使用者需要知道MySQL的用户名和密码才行。

## 2.4 Mysql的安装

**1）检查当前系统是否安装过 MySQL**

```
[atguigu@hadoop102 ~]$ rpm -qa|grep mariadb
mariadb-libs-5.5.56-2.el7.x86_64 
//如果存在通过如下命令卸载
[atguigu @hadoop102 ~]$ sudo rpm -e --nodeps mariadb-libs
```

**2）将 MySQL 安装包拷贝到/opt/software 目录下**

```
[atguigu @hadoop102 software]# ll
总用量 528384
-rw-r--r--. 1 root root 609556480 3 月 21 15:41 mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**3）解压 MySQL 安装包**

```
[atguigu @hadoop102 software]# tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
```

**4）在安装目录下执行 rpm 安装**

```
[atguigu @hadoop102 software]$ 
sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

​		==过程中出现报错则说明需要安装依赖，通过 yum 安装缺少的依赖,然后重新安装 mysql-community-server-5.7.28-1.el7.x86_64 即可==

```
[atguigu@hadoop102 software] yum install -y libaio
```

**5）删除/etc/my.cnf 文件中 datadir 指向的目录下的所有内容,如果有内容的情况下:**（一般省略）

 **查看 datadir 的值：** 

```
[mysqld]
datadir=/var/lib/mysql
```

**删除/var/lib/mysql 目录下的所有内容（一般没有任何东西）:**

```
[atguigu @hadoop102 mysql]# cd /var/lib/mysql
[atguigu @hadoop102 mysql]# sudo rm -rf ./* //注意执行命令的位置
```

**6）初始化数据库**

```
[atguigu @hadoop102 opt]$ sudo mysqld --initialize --user=mysql
```

**7）查看临时生成的 root 用户的密码**

```
[atguigu @hadoop102 opt]$ sudo cat /var/log/mysqld.log       jZu%i>4hD
```

![1660702382426](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660702382426.png)

**8）启动 MySQL 服务**

```
[atguigu @hadoop102 opt]$ sudo systemctl start mysqld
```

**9）登录 MySQL 数据库**

```
[atguigu @hadoop102 opt]$ mysql -uroot -p
Enter password: 输入临时生成的密码
```

**10）必须先修改 root 用户的密码,否则执行其他的操作会报错**

```
mysql> set password = password("fgl123");
```

**11）修改 mysql 库下的 user 表中的 root 用户允许任意 ip 连接**

```
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;
```

**12）使用本机Navicat连接远程数据库**

![1660702611954](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660702611954.png)

## 2.5配置Hive与Mysql

**1）将mysql驱动拷贝到hive的lib包下**

```
[atguigu@hadoop102 software]$ cp /opt/software/mysql-connector-java-5.1.37.jar $HIVE_HOME/lib
```

**2）配置Metastore到Mysql**

​	**（1）在$HIVE_HOME/conf 目录下新建 hive-site.xml 文件**

```
[atguigu@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
```

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

 <!-- jdbc 连接的 URL -->
 <property>
 <name>javax.jdo.option.ConnectionURL</name>
 <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
</property>

 <!-- jdbc 连接的 Driver-->
 <property>
 <name>javax.jdo.option.ConnectionDriverName</name>
 <value>com.mysql.jdbc.Driver</value>
</property>

<!-- jdbc 连接的 username-->
 <property>
 <name>javax.jdo.option.ConnectionUserName</name>
 <value>root</value>
 </property>
 
 <!-- jdbc 连接的 password -->
 <property>
 <name>javax.jdo.option.ConnectionPassword</name>
 <value>fgl123</value>
</property>

 <!-- Hive 元数据存储版本的验证 -->
 <property>
 <name>hive.metastore.schema.verification</name>
 <value>false</value>
</property>

 <!--元数据存储授权-->
 <property>
 <name>hive.metastore.event.db.notification.api.auth</name>
 <value>false</value>
 </property>
 
 <!-- Hive 默认在 HDFS 的工作目录 -->
 <property>
 <name>hive.metastore.warehouse.dir</name>
 <value>/user/hive/warehouse</value>
 </property>
</configuration>
```

   **（2）登陆 MySQL**

```
[atguigu@hadoop102 software]$ mysql -uroot -pfgl123
```

   **（3）新建 Hive 元数据库**

```
mysql> create database metastore;
mysql> quit;
```

   **（4）初始化 Hive 元数据库**

```
[atguigu@hadoop102 software]$ schematool -initSchema -dbType mysql -verbose
```



****

<font color="red">**使用元数据服务的方式访问 Hive**</font>



**1）在 hive-site.xml 文件中添加如下配置信息**

```
 <!-- 指定存储元数据要连接的地址 -->
 <property>
 <name>hive.metastore.uris</name>
 <value>thrift://hadoop102:8020</value>
 </property>
```

**2）启动 metastore**

```
[atguigu@hadoop202 hive]$ hive --service metastore
2020-04-24 16:58:08: Starting Hive Metastore Server
注意: 启动后窗口不能再操作，需打开一个新的 shell 窗口做别的操作
```

**3）启动 hive**

```
[atguigu@hadoop202 hive]$ bin/hive
```



---



<font color="red">**使用 JDBC 方式访问 Hive**</font>



**1）在 hive-site.xml 文件中添加如下配置信息**

```
<!-- 指定 hiveserver2 连接的 host -->
 <property>
 <name>hive.server2.thrift.bind.host</name>
 <value>hadoop102</value>
 </property>
 
 <!-- 指定 hiveserver2 连接的端口号 -->
 <property>
 <name>hive.server2.thrift.port</name>
 <value>10000</value>
 </property>
```

**2）启动 hiveserver2**

```
[atguigu@hadoop102 hive]$ bin/hive --service hiveserver2
```

**3）启动 beeline 客户端（需要多等待一会）**

```
[atguigu@hadoop102 hive]$ bin/beeline -u jdbc:hive2://hadoop102:10000 -n fang
```

**4）看到如下界面**

```
Connecting to jdbc:hive2://hadoop102:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
0: jdbc:hive2://hadoop102:10000>
```



```xml
<property>
                <name>hadoop.proxyuser.fang.hosts</name>
                <value>*</value>
</property>
<property>
               <name>hadoop.proxyuser.fang.groups</name>
               <value>*</value>
</property>
```



# [第 3 章 Hive 数据类型	](#_Toc6324 )（18-20）

## 3.1基本数据类型

| Hive 数据类型 | Java 数据类型 | 长度                                                 | 例子                                   |
| ------------- | ------------- | ---------------------------------------------------- | -------------------------------------- |
| TINYINT       | byte          | 1byte 有符号整数                                     | 20                                     |
| SMALINT       | short         | 2byte 有符号整数                                     | 20                                     |
| ==INT==       | int           | 4byte 有符号整数                                     | 20                                     |
| ==BIGINT==    | long          | 8byte 有符号整数                                     | 20                                     |
| BOOLEAN       | boolean       | 布尔类型，true 或者false                             | TRUE FALSE                             |
| FLOAT         | float         | 单精度浮点数                                         | 3.14159                                |
| ==DOUBLE==    | double        | 双精度浮点数                                         | 3.14159                                |
| ==STRING==    | string        | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | ‘ now is the time ’ “for all good men” |
| TIMESTAMP     |               | 时间类型                                             |                                        |
| BINARY        |               | 字节数组                                             |                                        |

​		**对于 Hive 的 String 类型相当于数据库的varchar 类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储 2GB 的字符数。**

## 3.2集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| STRUCT   | 和c 语言中的 struct 类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是 STRUCT{first STRING, last STRING},那么第 1 个元素可以通过字段.first 来引用。 | struct()例	如	struct<street:string, city:string> |
| MAP      | MAP 是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是 MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()例如 map<string, int>                             |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第 2 个元素可以通过数组名[1]进行引用。 | Array()例如 array<string>                              |

​		**Hive 有三种复杂数据类型 ARRAY、MAP 和 STRUCT。ARRAY 和 MAP 与 Java 中的 Array 和 Map 类似，而 STRUCT 与 C 语言中的 Struct 类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。**

**字段解释：** 

```
row format delimited fields terminated by ',' -- 列分隔符 
collection items terminated by '_' --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号) 
map keys terminated by ':'            -- MAP 中的 key 与 value 的分隔符 
lines terminated by '\n';                  -- 行分隔符 
```

**访问三种集合列里的数据，以下分别是 ARRAY，MAP，STRUCT 的访问方式**

```
hive (default)> select friends[1],children['xiao song'],address.city from  
test 
where name="songsong"; 
OK 
_c0 _c1 city 
lili 18 beijing 
Time taken: 0.076 seconds, Fetched: 1 row(s) 
```

## 3.3类型转化

​		**Hive 的原子数据类型是可以进行隐式转换的，类似于 Java 的类型转换，**例如某表达式 使用 INT 类型，TINYINT 会自动转换为 INT 类型，但是 **Hive 不会进行反向转化**，例如，某表 达式使用 TINYINT 类型，INT 不会自动转换为 TINYINT 类型，它会返回错误，除非使用 CAST 操作。 

**1）隐式类型转换规则如下**

（1）任何整数类型都可以隐式地转换为一个范围更广的类型，如 TINYINT 可以转换成
           INT，INT 可以转换成 BIGINT。 

（2）所有整数类型、FLOAT 和 STRING 类型都可以隐式地转换成 DOUBLE。 

（3）TINYINT、SMALLINT、INT 都可以转换为 FLOAT。 

（4）BOOLEAN 类型不可以转换为任何其它的类型。

**2）可以使用 CAST 操作显示进行数据类型转换**

​	例如 CAST('1' AS INT)将把字符串'1' 转换成整数 1；如果强制类型转换失败，如执行 

​    CAST('X' AS INT)，表达式返回空值 NULL。

# [第 4 章 DDL 数据定义	](#_Toc30153 )（21-28）

## 4.1创建数据库

1)创建一个数据库，数据库在 HDFS 上的默认存储路径是/user/hive/warehouse/*.db、

```
hive (default)> create database db_hive;
```

2）避免要创建的数据库已经存在错误，增加 if not exists 判断。（标准写法）、

```
create database if not exists db_hive;
```

3）创建一个数据库，指定数据库在 HDFS 上存放的位置

```
hive (default)> create database db_hive2 location '/db_hive2.db';
```

## 4.2查询数据库

### 4.2.1显示数据库

**1）显示数据库**

```
hive> show databases;
```

**2）过滤显示查询的数据库**

```
hive> show databases like 'db_hive*'; 
db_hive 
db_hive_1
```

### 4.2.2查看数据库详情

**1）显示数据库信息**

```
hive> desc database db_hive;
```

**2）显示数据库详细信息**

```
hive> desc database extended db_hive; 
```

### 4.2.3切换当前数据库

```
hive (default)> use db_hive;
```

## 4.3修改数据库

1）用户可以使用ALTER DATABASE 命令为某个数据库的DBPROPERTIES 设置键-值对属性值， 来描述这个数据库的属性信息。

```
hive (default)> alter database db_hive
set dbproperties('createtime'='20170830');
```

2）在 hive 中查看修改结果

```
hive> desc database extended db_hive;
```

## 4.4删除数据库

1）删除数据库

```
hive>drop database db_hive2;
```

2）如果删除的数据库不存在，最好采用if exists判断数据库是否存在

```
hive> drop database if exists db_hive2;
```

3）如果数据库不为空，可以采用cascade命令，强制进行删除

```
hive> drop database db_hive cascade;
```

## 4.5创建表

1）建表语法

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_nam
```

### 4.5.1管理表

当我们删除一个管理表时，Hive 也会删除这个表中数据

![image-20221106110457444](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221106110457444.png)

### 4.5.2外部表

 Hive 并非认为其完全拥有这份数据。删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

![image-20221106110538415](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221106110538415.png)

![image-20221106110701277](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221106110701277.png)

![image-20221106140330057](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221106140330057.png)

### 4.5.3管理表与外部表的相互转换

1）查询表的类型

```
hive (default)> desc formatted student2; 
```

2）修改内部表为外部表

```
alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

3）查询表的类型

```
hive (default)> desc formatted student2; 
```

4）修改外部表

```
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

5）查询表的类型

```
hive (default)> desc formatted student2; 
```

## 4.6修改表

### 4.6.1重命名表

```
hive (default)> alter table dept_partition2 rename to dept_partition3;
```

### 4.6.2增加，修改，删除表分区

详见 7.1 章分区表基本操作。

### 4.6.3增加，修改，替换列信息

**（1）案例实操**

（1）查询表结构

```
hive> desc dept;
```

（2）添加列

```
hive (default)> alter table dept add columns(deptdesc string);
```

（3）查询表结构

```
hive> desc dept;
```

（4）更新列

```
hive (default)> alter table dept change column deptdesc desc string;
```

（5）查询表结构

```
hive> desc dept;
```

（6）替换列

```
hive (default)> alter table dept replace columns(deptno string, dname string, loc string);
```

（7）查询表结构

```
hive> desc dept;
```

## 4.7删除表

```
hive (default)> drop table dept;
```

# [第 5 章 DML 数据操作	](#_Toc20867 )（29-44）

## 5.1数据导入

### 5.1.1向表中装载数据

1. **创建一张表**

   ```
   hive (default)> create table student(id string, name string) row format delimited fields terminated by '\t';
   ```

2. **加载本地文件到 hive**

   ```
   hive (default)> load data local inpath '/opt/module/hive/datas/student.txt' into table default.student;
   ```

3. **加载 HDFS 文件到 hive 中**

   上传文件到HDFS

   ```
    hive (default)> dfs -put /opt/module/hive/data/student.txt
   /user/atguigu/hive;
   ```

    加载HDFS上的数据

   ```
   hive (default)> load data inpath '/user/atguigu/hive/student.txt' into table default.student;
   ```

4. **加载数据覆盖表中已有的数据**

   上传文件到 HDFS

    加载数据覆盖表中已有的数据

   ```
   hive (default)> dfs -put /opt/module/data/student.txt /user/atguigu/hive;
   
   hive (default)> load data inpath '/user/atguigu/hive/student.txt' overwrite into table default.student;
   ```

### 5.1.2通过查询语句向表中插入数据

**1）创建一张表**

```
hive (default)> create table student_par(id int, name string) row format delimited fields terminated by '\t';
```

**2）基本插入数据**

```
hive (default)> insert into table student_par values(1,'wangwu'),(2,'zhaoliu');
```

**3）基本模式插入（根据单张表查询结果）**

```
hive (default)> insert overwrite table student_par
select id, name from student where month='201709';
```

**4）多表（多分区）插入模式（根据多张表查询结果）**

```
hive (default)> from student
		insert overwrite table student partition(month='201707') 		 select id, name where month='201709'
		insert overwrite table student partition(month='201706') 		 select id, name where month='201709';
```

### 5.1.3查询语句中创建表并加载数据

**根据查询结果创建表（查询的结果会添加到新创建的表中）**

```
create table if not exists student3 as select id, name from student;
```

### 5.1.4创建表时通过Location指定加载数据路径

**1）上传数据到 hdfs 上**

```
hive (default)> dfs -mkdir /student;
hive (default)> dfs -put /opt/module/datas/student.txt /student;
```

**2）创建表，并指定在 hdfs 上的位置**

```
hive (default)> create external table if not exists student5( id int, name string
)
row format delimited fields terminated by '\t' 
location '/student;
```

**3）查询数据**

```
hive (default)> select * from student5;
```

### 5.1.5Import数据到指定Hive表中

```
hive (default)> import table student2
from '/user/hive/warehouse/export/student';
```

## 5.2数据导出

### 5.2.1Insert导出

**1）将查询的结果导出到本地**

```
hive (default)> insert overwrite local directory '/opt/module/hive/data/export/student'
select * from student;
```

**2）将查询的结果格式化导出到本地**

```
hive(default)>insert overwrite local directory '/opt/module/hive/data/export/student1'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
```

**3）将查询的结果导出到 HDFS 上(没有 local)**

```
hive (default)> insert overwrite directory '/user/atguigu/student2' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
select * from student;
```

### 5.2.2Hadoop命令导出到本地

```
hive (default)> dfs -get /user/hive/warehouse/student/student.txt
/opt/module/data/export/student3.txt
```

### 5.2.3Hive Shell命令导出

基本语法：（hive -f/-e 执行语句或者脚本 > file）

```
[atguigu@hadoop102 hive]$ bin/hive -e 'select * from default.student;' >/opt/module/hive/data/export/student4.txt;
```

### 5.2.4Export导出到HDFS上

```
(defahiveult)> export table default.student to '/user/hive/warehouse/export/student';
```

### 5.2.5Sqoop导出

后续课程专门讲。

### 5.2.6清楚表中的数据

注意：Truncate 只能删除管理表，不能删除外部表中数据

```
hive (default)> truncate table dept_partition;
```

# [第 6 章 查询	](#_Toc408 )（45-57）

## 6.1基本查询

**(1)运算符的使用**

| 运算符 | 描述              |
| ------ | ----------------- |
| A+B    | A 和 B 相加       |
| A-B    | A 减去 B          |
| A*B    | A 和 B 相乘       |
| A/B    | A 除以 B          |
| A%B    | A 对 B 取余       |
| A&B    | A 和 B 按位取与   |
| A\|B   | A 和 B 按位取或   |
| A^B    | A 和 B 按位取异或 |
| ~A     | A 按位取反        |

**（2）常用函数**

​		**1）求总行数（count）**
​		hive (default)> select count(*) cnt from emp;	

​		**2）求工资的最大值（max）**
​		hive (default)> select max(sal) max_sal from emp;	

​		**3）求工资的最小值（min）**
​		hive (default)> select min(sal) min_sal from emp;	

​		**4）求工资的总和（sum）**
​		hive (default)> select sum(sal) sum_sal from emp;	

​		**5）求工资的平均值（avg）**
​		hive (default)> select avg(sal) avg_sal from emp;

**（3）Limit语句**

​		典型的查询会返回多行数据。LIMIT 子句用于限制返回的行数。

 **（4）比较运算符**

| 操作符                  | 支持的数据类型 | 描述                                                         |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| A=B                     | 基本数据类型   | 如果A 等于 B 则返回TRUE，反之返回 FALSE                      |
| A<=>B                   | 基本数据类型   | 如果A 和B 都为 NULL，则返回 TRUE，如果一边为NULL，返回 False |
| A<>B, A!=B              | 基本数据类型   | A 或者 B 为 NULL 则返回 NULL；如果 A 不等于 B，则返回TRUE，反之返回 FALSE |
| A<B                     | 基本数据类型   | A 或者 B 为 NULL，则返回 NULL；如果 A 小于 B，则返回TRUE，反之返回 FALSE |
| A<=B                    | 基本数据类型   | A 或者 B 为 NULL，则返回 NULL；如果 A 小于等于 B，则返回 TRUE，反之返回 FALSE |
| A>B                     | 基本数据类型   | A 或者 B 为 NULL，则返回 NULL；如果 A 大于 B，则返回TRUE，反之返回 FALSE |
| A>=B                    | 基本数据类型   | A 或者 B 为 NULL，则返回 NULL；如果 A 大于等于 B，则返回 TRUE，反之返回 FALSE |
| A [NOT] BETWEEN B AND C | 基本数据类型   | 如果 A，B 或者 C 任一为NULL，则结果为 NULL。如果 A 的值大于等于 B 而且小于或等于C，则结果为 TRUE，反之为 FALSE。如果使用NOT 关键字则可达到相反的效果。 |
| A IS NULL               | 所有数据类型   | 如果A 等于 NULL，则返回TRUE，反之返回 FALSE                  |
| A IS NOT NULL           | 所有数据类型   | 如果A 不等于NULL，则返回 TRUE，反之返回 FALSE                |
| IN(数值 1, 数值 2)      | 所有数据类型   | 使用 IN 运算显示列表中的值                                   |
| A [NOT] LIKE B          | STRING 类型    | B 是一个 SQL 下的简单正则表达式，也叫通配符模式，如果 A 与其匹配的话，则返回TRUE；反之返回 FALSE。B 的表达式说明如下：‘x%’表示 A 必须以字母‘x’开头，‘%x’表示 A 必须以字母’x’结尾，而‘%x%’表示 A 包含有字母’x’,可以位于开头，结尾或者字符串中间。如果使用 NOT 关键字则可达到相反的效果。 |
| A RLIKE B, A REGEXP B   | STRING 类型    | B 是基于 java 的正则表达式，如果 A 与其匹配，则返回TRUE；反之返回 FALSE。匹配使用的是 JDK 中的正则表达式接口实现的，因为正则也依据其中的规则。例如，正则表达式必须和整个字符串 A 相匹配，而不是只需与其字符串匹配。 |

**（5）Like与RLike**

​	**1）** **使用** **LIKE** **运算选择类似的值**

​	**2）** **选择条件可以包含字符或数字**

​		% 代表零个或多个字符(任意个字符)。

​		_ 代表一个字符。

​	**3）** **RLIKE** **子句**

​	RLIKE 子句是 Hive 中这个功能的一个扩展，其可以通过 Java 的正则表达式这个更强大的语言来指定匹配条件。

​	（1）查找名字以A 开头的员工信息

```
hive (default)> select * from emp where ename LIKE 'A%';
```

​	（2）查找名字中第二个字母为A 的员工信息

```
hive (default)> select * from emp where ename LIKE '_A%';
```

​	（3）查找名字中带有A 的员工信息

```
hive (default)> select * from emp where ename RLIKE '[A]';
```

**（6）逻辑运算符**

![1658665519809](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658665519809.png)

## 6.2分组

**（1）Group By语句**

​		GROUP BY 语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然
后对每个组执行聚合操作。

**（2）Having 语句**

​	**having 与 where 不同点**
​	（1）where 后面不能写分组函数，而 having 后面可以使用分组函数。
​	（2）having 只用于 group by 分组统计语句。

## 6.3Join语句

**内连接：**只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来。

**左外连接：**JOIN 操作符左边表中符合 WHERE 子句的所有记录将会被返回。 

**右外连接：**JOIN 操作符右边表中符合 WHERE 子句的所有记录将会被返回。 

**满外连接：**将会返回所有表中符合 WHERE 语句条件的所有记录。如果任一表的指定字 段没有符合条件的值的话，那么就使用 NULL 值替代。 

**多表连接：**连接 n 个表，至少需要 n-1 个连接条件。例如：连接三个表，至少需要两个连接 条件

## 6.4排序

**1.全局排序（Order By）**

Order By：全局排序，只有一个 Reducer 

​		ASC（ascend）: 升序（默认） 

​		DESC（descend）: 降序

**2.每个 Reduce 内部排序（Sort By）**

Sort By：对于大规模的数据集 order by 的效率非常低。在很多情况下，并不需要全局排 序，此时可以使用 **sort by**。 Sort by 为每个 reducer 产生一个排序文件。每个 Reducer 内部进行排序，对全局结果集来说不是排序。

1）设置reduce个数

```
hive (default)> set mapreduce.job.reduces=3;
```

2）查看设置的个数

```
hive (default)> set mapreduce.job.reduces; 
```

3）根据部门编号降序查看员工信息

```
hive (default)> select * from emp sort by deptno desc;
```

4）将查询结果导入到文件中（按照部门编号降序排序） 

```
hive (default)> insert overwrite local directory  
'/opt/module/data/sortby-result' 
select * from emp sort by deptno desc; 
```

**3.分区（Distribute By）**

Distribute By： 在有些情况下，我们需要控制某个特定行应该到哪个 reducer，通常是为了进行后续的聚集操作。distribute by 子句可以做这件事。distribute by 类似 MR 中 partition（自定义分区），进行分区，结合 sort by 使用。对于 distribute by 进行测试，一定要分配多 reduce 进行处理，否则无法看到 distribute by 的效果。

```
hive (default)> set mapreduce.job.reduces=3; 

hive (default)> insert overwrite local directory  

'/opt/module/data/distribute-result' select * from emp distribute by  

deptno sort by empno desc; 
```

==➢distribute by 的分区规则是根据分区字段的 hash 码与 reduce 的个数进行模除后，
余数相同的分到一个区==

==➢ Hive 要求 DISTRIBUTE BY 语句要写在 SORT BY 语句之前。==

**4.Cluster By**

当 distribute by 和 sorts by 字段相同时，可以使用 cluster by 方式。 cluster by 除了具有 distribute by 的功能外还兼具 sort by 的功能。但是排序只能是升序 排序，不能指定排序规则为 ASC 或者 DESC。 

```
hive (default)> select * from emp cluster by deptno; 
hive (default)> select * from emp distribute by deptno sort by deptno;
```

注意：按照部门编号分区，不一定就是固定死的数值，可以是 20 号和 30 号部门分到一 

个分区里面去。 

# [第 7 章 分区表和分桶表	](#_Toc4861 )（58-66）

## 7.1分区表

​		把一个大的数据集根据业务需要分割成小的数据集。在查询时通过 WHERE 子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多

### 7.1.1分区表基本操作

(1)建表：指明分区字段

```
hive (default)> create table 
dept_partition( deptno int, dname string,
)
partitioned by (day string)
row format delimited fields terminated by '\t';
```

（2）加载数据：指明分区字段名

```
hive (default)> load data local inpath '/opt/module/data/dept3.txt' into table dept_partition partition(day='20200401');
```

（3）查询分区表中数据

```
hive (default)> select * from dept_partition where day='20200401';
```

（4）增加分区

```sql
--新建单个分区
hive (default)> alter table dept_partition add partition(day='20200404');
--新建多个分区
hive (default)> alter table dept_partition add partition(day='20200405') partition(day='20200406');--用空格隔开
```

（5）删除分区

```sql
--删除单个分区
hive (default)> alter table dept_partition drop partition (day='20200406');
--删除多个分区用逗号隔开
hive (default)> alter table dept_partition drop partition (day='20200404'), partition(day='20200405');
```

### 7.1.2二级分区

（1）创建二级分区表

```
hive (default)> create table dept_partition2( deptno int, dname string, loc string
)
partitioned by (day string, hour string)
```

（2）正常的加载数据

​			加载数据到二级分区中	

```sql
hive (default)> load data local inpath '/opt/module/data/dept1.txt' into table dept_partition2 partition(day='20200401', hour='14');
```

​			查询数据分区

```sql
hive (default)> select * from dept_partition2 where day='20200401' and hour='12';
```

（3）把数据上传到分区目录上，让分区表和数据产生关联

**3.方式一：上传数据后修复**

（1）加载数据后到二级分区表中

```
hive (default)> dfs -mkdir -p
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13; hive (default)> dfs -put /opt/module/datas/dept_20200401.log
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=13;
```

（2）查询数据（查询不到刚上传的数据）

```
hive (default)> select * from dept_partition2 where day='20200401' and hour='13';
```

（3）执行修复命令

```
hive> msck repair table dept_partition2;
```

（4）再次查询数据

```
hive (default)> select * from dept_partition2 where day='20200401' and hour='13';
```

**2.方式二：上传数据后添加分区**

```sql
--上传分区
hive (default)> dfs -mkdir -p
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14; hive (default)> dfs -put /opt/module/hive/datas/dept_20200401.log
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=14;

--执行添加分区
hive (default)> alter table dept_partition2 add partition(day='201709',hour='14');

--查询数据
hive (default)> select * from dept_partition2 where day='20200401' and hour='14';
```

**3.方式三：创建文件夹后load数据到分区**

```sql
--创建目录
hive (default)> dfs -mkdir -p
/user/hive/warehouse/mydb.db/dept_partition2/day=20200401/hour=15;

--上传数据
hive (default)> load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition2 partition(day='20200401',hour='15');

--查询数据
hive (default)> select * from dept_partition2 where day='20200401' and hour='15';
```

### 7.1.3动态分区调整

​		关系型数据库中，对分区表 Insert 数据时候，数据库自动会根据分区字段的值，将数据插入到相应的分区中，Hive 中也提供了类似的机制，即动态分区(Dynamic Partition)，只不过，使用 Hive 的动态分区，需要进行相应的配置。

（1）开启动态分区功能（默认true，开启）

```
hive.exec.dynamic.partition=true
```

（2）设置为非严格模式（动态分区的模式，默认 strict，表示必须指定至少一个分区为

静态分区，nonstrict 模式表示允许所有的分区字段都可以使用动态分区。）

```
hive.exec.dynamic.partition.mode=nonstrict
```

（3）在所有执行 MR 的节点上，最大一共可以创建多少个动态分区。默认 1000

```
hive.exec.max.dynamic.partitions=1000
```

（4）在每个执行 MR 的节点上，最大可以创建多少个动态分区。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即 day 字段有 365 个值，那么该参数就需要设置成大于 365，如果使用默认值 100，则会报错。

```
hive.exec.max.dynamic.partitions.pernode=100
```

（5）整个 MR Job 中，最大可以创建多少个 HDFS 文件。默认 100000

```
hive.exec.max.created.files=100000
```

（6）当有空分区生成时，是否抛出异常。一般不需要设置。默认 false

```
hive.error.on.empty.partition=false
```

**案例实操：**

```sql
--需求：将 dept 表中的数据按照地区（loc 字段），插入到目标表 dept_partition 的相应分区中。
--（1）创建目标分区表
hive (default)> create table dept_partition_dy(id int, name string) partitioned by (loc int) row format delimited fields terminated by '\t';

--（2）设置动态分区
set hive.exec.dynamic.partition.mode = nonstrict;
hive (default)> insert into table dept_partition_dy partition(loc) select deptno, dname, loc from dept;

--（3）查看目标分区表的分区情况
hive (default)> show partitions dept_partition;	
--思考：目标分区表是如何匹配到分区字段的？
```

## 7.2分桶表

​		分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区。对于一张表或者分区，Hive 可以进一步组织成桶，也就是更为细粒度的数据范围划分。

​		分桶是将数据集分解成更容易管理的若干部分的另一个技术。分区针对的是数据的存储路径；分桶针对的是数据文件。

**（1）创建分桶表**

```
create table stu_buck(id int, name string) clustered by(id)
into 4 buckets
row format delimited fields terminated by '\t';
```

**（2）查看表结构**

```
hive (default)> desc formatted stu_buck; 
Num Buckets:	4
```

**（3）导入数据到分桶表中，load 的方式**

```sql
hive (default)> load data inpath '/student.txt' into table stu_buck;
```

**（4）查询分桶的数据**

```
hive(default)> select * from stu_buck;
```

**==分桶表的注意事项：==**

（1） reduce 的个数设置为-1,让 Job 自行决定需要用多少个 reduce 或者将 reduce 的个数设置为大于等于分桶表的桶数

（2） 从 hdfs 中 load 数据到分桶表中，避免本地文件找不到问题

（3） 不要使用本地模式

## 7.3抽样查询

​		对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive 可以通过对表进行抽样来满足这个需求。

**语法: TABLESAMPLE(BUCKET x OUT OF y)**

```sql
--查询表 stu_buck 中的数据。
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);

--注意：x 的值必须小于等于 y 的值，否则
FAILED: SemanticException [Error 10061]: Numerator should not be bigger than denominator in sample clause for table stu_buck
```

# [第 8 章 函数	](#_Toc30611 )（67-94）

## 8.1系统内置函数

**(1)查看系统自带的函数**

```
show functions;
```

**(2)显示自带函数的用法**

```
desc function upper;
```

**(3)详细显示自带的函数的语法**

```
desc function extended upper;
```

## 8.2常用内置函数

### 8.2.1 空字段赋值

**(1)函数说明**

**NVL：**给值为 NULL 的数据赋值，它的格式是 NVL( value，default_value)。

替换空值

```
hive (default)> select comm,nvl(comm, -1) from emp; 
```

### 8.2.2 CASE WHEN THEN ELSE END

相当于if...else...

```
select
 dept_id,
 sum(case sex when '男' then 1 else 0 end) male_count,
 sum(case sex when '女' then 1 else 0 end) female_count
from emp_sex
group by dept_id;
```

### 8.2.3 行转列

CONCAT(string A/col, string B/col…)：返回字符串连接后的结果，支持任意字符。

CONCAT_WS(separator, str1, str2,...)：第一个设置为分割符，连接后面的字段

==只接受string or array<string>==

COLLECT_SET(col)：只接受基本数据类型，将某些字段进行去重汇总，产生Array类型

### 8.2.4 列转行

EXPLODE(col)：将 hive 一列中复杂的 Array 或者 Map 结构拆分成多行。 

LATERAL VIEW：用法：LATERAL VIEW udtf(expression) tableAlias AS columnAlias 

​							解释：用于和 split, explode 等 UDTF 一起使用，它能够将一列数据拆成							多行数据，在此基础上可以对拆分后的数据进行聚合。

### 8.2.5 窗口函数（开窗函数）

OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变

CURRENT ROW：当前行 

n PRECEDING：往前 n 行数据 

n FOLLOWING：往后 n 行数据 

UNBOUNDED：起点， 

​			UNBOUNDED PRECEDING 表示从前面的起点， 

 		   UNBOUNDED FOLLOWING 表示到后面的终点 

LAG(col,n,default_val)：往前第 n 行数据 

LEAD(col,n, default_val)：往后第 n 行数据 

NTILE(n)：把有序窗口的行分发到指定数据的组中，各个组有编号，编号从 1 开始，对 

于每一行，NTILE 返回此行所属的组的编号。注意：n 必须为 int 类型。 

### 8.2.6 Rank

RANK() 排序相同时会重复，总数不会变

DENSE_RANK() 排序相同时会重复，总数会减少 

ROW_NUMBER() 会根据顺序计算

## 8.3自定义函数

1）Hive 自带了一些函数，比如：max/min 等，但是数量有限，自己可以通过自定义 UDF 来 方便的扩展。 

2）当 Hive 提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义 函数（UDF：user-defined function）。 

（1）UDF（User-Defined-Function） 一进一出 

（2）UDAF（User-Defined Aggregation Function） 

聚集函数，多进一出   类似于：count/max/min 

（3）UDTF（User-Defined Table-Generating Functions） 

一进多出   如 lateral view explode()

## 8.4自定义UDF函数（一进一出）

**1）创建一个 Maven 工程 Hive**

**2）导入依赖**

**3）创建一个类**

**4）打成 jar 包上传到服务器/opt/module/data/myudf.jar**

**5）将 jar 包添加到 hive 的 classpath**

**6）创建临时函数与开发好的 java class 关联**

**7）即可在 hql 中使用自定义的函数**

## 8.5自定义UDTF函数（一进多出）

**1）代码实现**

**2）打成 jar 包上传到服务器/opt/module/hive/data/myudtf.jar、**

**3）将 jar 包添加到 hive 的 classpath 下**

**4）创建临时函数与开发好的 java class 关联**

**5）使用自定义的函数**

# [第 9 章 压缩和存储	](#_Toc18035 )（95-100）

## 9.1Hadoop压缩配置

### 9.1.1**MR** **支持的压缩编码** 

![1658737826827](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658737826827.png)

![1658737838192](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658737838192.png)

![1658737858306](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658737858306.png)

### 9.1.2 **压缩参数配置** 

| 参数                                              | 默认值                                                       | 阶段         | 建议                                            |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------ | ----------------------------------------------- |
| io.compression.codecs（在 core-site.xml 中配置）  | org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.Lz4Codec | 输入压缩     | Hadoop 使用文件扩展名判断是否支持某种编解码器   |
| mapreduce.map.output.com press                    | false                                                        | mapper 输出  | 这个参数设为 true 启用压缩                      |
| mapreduce.map.output.com press.codec              | org.apache.hadoop.io.compress.DefaultCodec                   | mapper 输出  | 使用 LZO、LZ4 或snappy 编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutput format.compress       | false                                                        | reducer 输出 | 这个参数设为 true 启用压缩                      |
| mapreduce.output.fileoutput format.compress.codec | org.apache.hadoop.io.compress. DefaultCodec                  | reducer 输出 | 使用标准工具或者编解码器，如 gzip 和bzip2       |
| mapreduce.output.fileoutput format.compress.type  | RECORD                                                       | reducer 输出 | SequenceFile 输出使用的压缩类型：NONE和 BLOCK   |

##  9.2开启Map输出阶段压缩（MR引擎）

**开启 map 输出阶段压缩可以减少 job 中 map 和 Reduce task 间数据传输量。具体配置如下：**

**（1）开启 hive 中间传输数据压缩功能**

```
hive (default)>set hive.exec.compress.intermediate=true;
```

**（2）开启 mapreduce 中 map 输出压缩功能**

```
hive (default)>set mapreduce.map.output.compress=true;
```

**（3）设置 mapreduce 中 map 输出数据的压缩方式**

```
hive (default)>set mapreduce.map.output.compress.codec= org.apache.hadoop.io.compress.SnappyCodec;
```

**（4）执行查询语句**

```
hive (default)> select count(ename) name from emp;
```



## 9.3 开启 Reduce 输出阶段压缩

当 Hive 将 输 出 写 入 到 表 中 时 ， 输出内容同样可以 进行压缩。属 性hive.exec.compress.output 控制着这个功能。用户可能需要保持默认设置文件中的默认值false， 这样默认的输出就是非压缩的纯文本文件了。用户可以通过在查询语句或执行脚本中设置这 个值为 true，来开启输出结果压缩功能。

（1）开启 hive 最终输出数据压缩功能

```
hive (default)>set hive.exec.compress.output=true;
```

（2）开启 mapreduce 最终输出数据压缩

```
hive (default)>set mapreduce.output.fileoutputformat.compress=true;
```

（3）设置 mapreduce 最终数据输出压缩方式

```
hive (default)> set mapreduce.output.fileoutputformat.compress.codec = org.apache.hadoop.io.compress.SnappyCodec;
```

（4）设置 mapreduce 最终数据输出压缩为块压缩

```
hive (default)> set mapreduce.output.fileoutputformat.compress.type=BLOCK;
```

（5）测试一下输出结果是否是压缩文件

```
hive (default)> insert overwrite local directory '/opt/module/data/distribute-result' select * from emp distribute by deptno sort by empno desc;
```



## 9.4 文件存储格式

**Hive 支持的存储数据的格式主要有：TEXTFILE 、SEQUENCEFILE、ORC、PARQUET。**

### 9.4.1列式存储和行式存储

![1658738431036](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658738431036.png)

**1）行存储的特点**

​		查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。
**2）列存储的特点**

​		因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。

### 9.4.2TextFile 格式

​			默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合 Gzip、Bzip2 使用， 但使用 Gzip 这种方式，hive 不会对数据进行切分，从而无法对数据进行并行操作。

### 9.4.3Orc 格式

**列式存储**

**分段进行列式存储**

### 9.4.4Parquet 格式

​		Parquet 文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和元数据，因此 Parquet 格式文件是自解析的。

### 9.4.5主流文件存储格式对比实验

1）TextFile

18.13 M	/user/hive/warehouse/log_text/log.data

2）ORC

7.7 M	/user/hive/warehouse/log_orc/000000_0

3）Parquet

13.1 M	/user/hive/warehouse/log_parquet/000000_0

==存储文件的对比总结： ORC >	Parquet >	textFile==

==存储文件的查询速度总结：查询速度相近。==

## 9.5 存储和压缩结合

### 9.5.1测试存储和压缩

**1）创建一个 ZLIB 压缩的 ORC 存储方式**

```sql
create table log_orc_zlib( track_time string,
url string, session_id string, referer string,
ip string, end_user_id string, city_id string
)
row format delimited fields terminated by '\t' stored as orc 
```

**2）创建一个 SNAPPY 压缩的 ORC 存储方式**

referer string, ip string,
end_user_id string, city_id string
)
row format delimited fields terminated by '\t' stored as orc tblproperties("orc.compress"="SNAPPY");

**3）创建一个 SNAPPY 压缩的 parquet 存储方式**

```
create table log_parquet_snappy( track_time string,
url string, session_id string, referer string,
ip string, end_user_id string, city_id string
)
row format delimited fields terminated by '\t' stored as parquet 
```

**4）存储方式和压缩总结**

​		**在实际的项目开发当中，hive 表的数据存储格式一般选择：orc 或 parquet。压缩方式一般选择 snappy，lzo。**

# [第 10 章 TEZ的安装	](#_Toc21409 )（116-125）

![1658802249003](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658802249003.png)

​		**用 Hive 直接编写 MR 程序，假设有四个有依赖关系的 MR 作业，上图中，绿色是 Reduce Task，云状表示写屏蔽，需要将中间结果持久化写到 HDFS。**

​		**Tez 可以将多个有依赖的作业转换为一个作业，这样只需写一次 HDFS，且中间节点较少，从而大大提升作业的计算性能。**

**1）将 tez 安装包拷贝到集群，并解压 tar 包**

```
[atguigu@hadoop102 software]$ mkdir /opt/module/tez
[atguigu@hadoop102 software]$ tar -zxvf /opt/software/tez-0.10.1- SNAPSHOT-minimal.tar.gz -C /opt/module/tez
```

**2）上传 tez 依赖到 HDFS**

```
[atguigu@hadoop102 software]$ hadoop fs -mkdir /tez
[atguigu@hadoop102 software]$ hadoop fs -put /opt/software/tez-0.10.1- SNAPSHOT.tar.gz /tez
```

**3）新建 tez-site.xml**

```
[atguigu@hadoop102 software]$ vim $HADOOP_HOME/etc/hadoop/tez-site.xml
```



```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
<name>tez.lib.uris</name>
<value>${fs.defaultFS}/tez/tez-0.10.1-SNAPSHOT.tar.gz</value>
</property>
<property>
<name>tez.use.cluster.hadoop-libs</name>
<value>true</value>
</property>
<property>
<name>tez.am.resource.memory.mb</name>
<value>1024</value>

</property>
<property>
<name>tez.am.resource.cpu.vcores</name>
<value>1</value>
</property>
<property>
<name>tez.container.max.java.heap.fraction</name>
<value>0.4</value>
</property>
<property>
<name>tez.task.resource.memory.mb</name>
<value>1024</value>
</property>
<property>
<name>tez.task.resource.cpu.vcores</name>
<value>1</value>
</property>
</configuration>
```

**5）修改 Hive 的计算引擎**

```
[atguigu@hadoop102 software]$ vim $HIVE_HOME/conf/hive-site.xml
```

添加

```
<property>
<name>hive.execution.engine</name>
<value>tez</value>
</property>
<property>
<name>hive.tez.container.size</name>
<value>1024</value>
</property>
```

**6）解决日志 Jar 包冲突**

```
[atguigu@hadoop102 software]$ rm /opt/module/tez/lib/slf4j-log4j12- 1.7.10.jar
```

