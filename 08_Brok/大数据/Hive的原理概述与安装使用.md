# Hive概述与Mysql元数据配置

[TOC]

## 1.0Hive概述

### 1.1什么是hive

​		Hive：由 Facebook 开源用于解决海量结构化日志的数据统计工具。

​		Hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类 SQL 查询功能。

### 1.2Hive的本质是发什么？

​		**将HSQL转化为MapReduce程序**

![image-20220830184504180](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220830184504180.png)

**（1） Hive 处理的数据存储在 HDFS**

**（2） Hive 分析数据底层的实现是 MapReduce**

**（3） 执行程序运行在 Yarn 上**

### 1.3Hive的优缺点

**1）** **Hive** **的** **HQL** **表达能力有限**

（1） 迭代式算法无法表达

（2） 数据挖掘方面不擅长，由于 MapReduce 数据处理流程的限制，效率更高的算法却无法实现。

**2）** **Hive** **的效率比较低**

（1） Hive 自动生成的 MapReduce 作业，通常情况下不够智能化

（2） Hive 调优比较困难，粒度较粗

### 1.4Hive的架构原理

![image-20220830184919854](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220830184919854.png)



**1） 用户接口：Client**

​		CLI（command-line interface）、JDBC/ODBC(jdbc 访问 hive)、WEBUI（浏览器访问 hive）

**1） 元数据：Metastore**

​		元数据包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；默认存储在自带的 derby 数据库中，推荐使用 MySQL 存储Metastore

**2）** **Hadoop**

​		使用 HDFS 进行存储，使用 MapReduce 进行计算。

**3） 驱动器：Driver**

​		（1） 解析器（SQL Parser）：将 SQL 字符串转换成抽象语法树 AST，这一步一般都用第三方工具库完成，比如 antlr；对 AST 进行语法分析，比如表是否存在、字段是否存在、SQL 语义是否有误。

​		（2） 编译器（Physical Plan）：将 AST 编译生成逻辑执行计划。

​		（3） 优化器（Query Optimizer）：对逻辑执行计划进行优化。

​		（4） 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于 Hive 来说，就是 MR/Spark。

​	<font color="red">Hive 通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的 Driver， 结合元数据(MetaStore)，将这些指令翻译成 MapReduce，提交到 Hadoop 中执行，最后，将执行返回的结果输出到用户交互接口。</font>	

![image-20220830185600823](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220830185600823.png)

## 2.0Hive与数据库的比较

​		由于 Hive 采用了类似 SQL 的查询语言 HQL(Hive Query Language)，因此很容易将 Hive 理

​		解为数据库。其实从结构上来看，<font color="red">Hive 和数据库除了拥有类似的查询语言，再无类似之处。</font>本文将从多个方面来阐述 Hive 和数据库的差异。数据库可以用在 Online 的应用中，但是Hive 是为数据仓库而设计的，清楚这一点，有助于从应用角度理解 Hive 的特性。

**1.1.1** **查询语言**

​		由于 SQL 被广泛的应用在数据仓库中，因此，专门针对 Hive 的特性设计了类 SQL 的查询语言 HQL。熟悉 SQL 开发的开发者可以很方便的使用 Hive 进行开发。

**1.1.2** **数据更新**

​		由于 Hive 是针对数据仓库应用设计的，而<font color ="red">数据仓库的内容是读多写少的</font>。因此，Hive 中不建议对数据的改写，所有的数据都是在加载的时候确定好的。而数据库中的数据通常是需要经常进行修改的，因此可以使用INSERT INTO … VALUES 添加数据，使用UPDATE … SET 修改数据。

**1.1.3** **执行延迟**

​		<font color="red">Hive 在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。</font>另外一个导致 Hive 执行延迟高的因素是MapReduce 框架。由于 **MapReduce 本身具有较高的延迟**，因此在利用 MapReduce 执行 Hive 查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模大到超过数据库的处理能力的时候， Hive 的并行计算显然能体现出优势。

**1.1.4** **数据规模**

​		由于 Hive 建立在集群上并可以利用 MapReduce 进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

## 3.0Hive API的安装

### 3.1Hive的安装

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

### 3.2hive的元数据与表数据

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

## 4.0 Mysql的安装

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

## 5.0 配置Hive与Mysql

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
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>fgl123</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
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
[atguigu@hadoop102 conf]$ schematool -initSchema -dbType mysql -verbose
```



<font color="red">**使用 JDBC 方式访问 Hive**</font>

**1）启动 hiveserver2**

```
[atguigu@hadoop102 hive]$ bin/hive --service hiveserver2
```

**2）启动 beeline 客户端（需要多等待一会）**

```
[atguigu@hadoop102 hive]$ bin/beeline -u jdbc:hive2://hadoop102:10000 -n 
atguigu
```

**3）看到如下界面**

```
Connecting to jdbc:hive2://hadoop102:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
0: jdbc:hive2://hadoop102:10000>
```

