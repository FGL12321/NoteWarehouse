[TOC]



#  Spark学习笔记

![1659276211445](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276211445.png)

# 第一章：Spark概述（1-3）

## 1.1 Spark 是什么

**Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。**

## 1.2 Spark and Hadoop

![1659276346305](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276346305.png)

**hadoop：**

![1659276561905](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276561905.png)

**Spark：**

![1659276587368](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276587368.png)

​		由上面的信息可以获知，Spark 出现的时间相对较晚，并且主要功能主要是用于数据计算，
所以其实 Spark 一直被认为是 Hadoop 框架的升级版。

### 1.3 Spark or Hadoop

**Hadoop MapReduce**

![1659276786447](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276786447.png)

**Spark**：满足迭代式开发

![1659276981336](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659276981336.png)

Spark 和Hadoop 的根本差异是多个作业之间的数据通信问题 : Spark 多个作业之间数据 通信是基于内存，而 Hadoop 是基于磁盘。

## 1.4 Spark 核心模块

![1659277079622](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659277079622.png)

➢ **Spark Core** 

**Spark Core 中提供了 Spark 最基础与最核心的功能**，Spark 其他的功能如：Spark SQL， 

Spark Streaming，GraphX, MLlib 都是在 Spark Core 的基础上进行扩展的 

➢ **Spark SQL** 

**Spark SQL 是 Spark 用来操作结构化数据的组件**。通过 Spark SQL，用户可以使用 SQL或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。 

➢ **Spark Streaming** 

**Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件**，提供了丰富的处理 数据流的 API。 

➢ **Spark MLlib** 

MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等 额外的功能，还提供了一些更底层的机器学习原语。 

➢ **Spark GraphX** 

GraphX 是 Spark 面向图计算提供的框架与算法库。

# 第二章 Spark快速上手（4-10）

## 2.1 创建 Maven 项目 

### 2.1.1 增加 Scala 插件 

​       Spark 由 Scala 语言开发的，所以本课件接下来的开发所使用的语言也为 Scala，咱们当前使用的 Spark 版本为 3.0.0，默认采用的 Scala 编译版本为 2.12，所以后续开发时。我们依然采用这个版本。开发前请保证 IDEA 开发工具中含有 Scala 开发插件。

### 2.1.2 增加依赖关系

```
 <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>
    <build>
    <plugins>
    <!-- 该插件用于将 Scala 代码编译成 class 文件 -->
    <plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>scala-maven-plugin</artifactId>
    <version>3.2.2</version>
    <executions>
    <execution>
        <!-- 声明绑定到 maven 的 compile 阶段 -->
        <goals>
            <goal>testCompile</goal>
        </goals>
    </execution>
    </executions>
    </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.1.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
    </build>
```

### 2.1.3 WordCount

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Spark01_WordCount3 {

  def main(args: Array[String]): Unit = {


    //TODO 建立和spark框架连接

    val sparkConf=new SparkConf().setMaster("local").setAppName("WordCount")
    val context = new SparkContext(sparkConf)

    //TODO 执行业务操作

    val value: RDD[String] = context.textFile("datas")

    val word: RDD[String] = value.flatMap(_.split(" "))

    val wordToOne=word.map(
      word=>(word,1)
    )

    //Spark框架提供了更多的功能，可以将分组和聚合使用一个方法实现
    //reduceByKey:相同的key的数据可以对value进行reduce聚合
    val wordTOCount = wordToOne.reduceByKey(_ + _)

    val tuples: Array[(String, Int)] = wordTOCount.collect()
    tuples.foreach(println)

    //TODO 关闭连接
    context.stop()

  }

}

```

执行过程中，会产生大量的执行日志，如果为了能够更好的查看程序的执行结果，可以在项
目的 resources 目录中创建 log4j.properties 文件，并添加日志配置信息：

```
log4j.rootCategory=ERROR, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/ddHH:mm:ss} %p %c{1}: %m%n
# Set the default spark-shell log level to ERROR. When running the spark-shell,the
# log level for this class is used to overwrite the root logger's log level, sothat
# the user can have different defaults for the shell and regular Spark apps.
log4j.logger.org.apache.spark.repl.Main=ERROR
# Settings to quiet third party logs that are too verbose
log4j.logger.org.spark_project.jetty=ERROR
log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR
log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR
# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
```

# 第三章：Spark运行环境（5-18）

​		Spark 作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行, 在国 内工作中主流的环境为 Yarn，不过逐渐容器式环境也慢慢流行起来。接下来，我们就分别 看看不同环境下 Spark 的运行

![1659429892691](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659429892691.png)

## 3.1 Local 模式

### 3.1.1 解压缩文件

​		将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到 Linux 并解压缩，放置在指定位置，路径中 不要包含中文或空格，课件后续如果涉及到解压缩操作，不再强调。 

```
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
cd /opt/module 
mv spark-3.0.0-bin-hadoop3.2 spark-local
```

### 3.1.2 启动 Local 环境

```
bin/spark-shell
```

启动成功后，可以输入网址进行 Web UI 监控页面访问

```
http://虚拟机地址:4040
```

### 3.1.3 命令行工具

​		在解压缩文件夹下的 data 目录中，添加 word.txt 文件。在命令行工具中执行如下代码指令（和 IDEA 中代码简化版一致）

```
sc.textFile("data/word.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

### 3.1.4 退出本地模式

```
按键 Ctrl+C 或输入 Scala 指令
```

### 3.1.5 提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

1) --class 表示要执行程序的主类，此处可以更换为咱们自己写的应用程序 

2) --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟 CPU 核数量 

3) spark-examples_2.12-3.0.0.jar 运行的应用类所在的 jar 包，实际使用时，可以设定为咱 

们自己打的 jar 包 

4) 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

## 3.2 Standalone 模式

local 本地模式毕竟只是用来进行练习演示的，真实工作中还是要将应用提交到对应的 集群中去执行，这里我们来看看只使用 Spark 自身节点运行的集群模式，也就是我们所谓的 ==独立部署（Standalone）模式==。Spark 的 Standalone 模式体现了经典的 master-slave 模式。 集群规划:

![1659443395749](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443395749.png)

### 3.2.1 解压缩文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到 Linux 并解压缩在指定位置

```
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
cd /opt/module 
mv spark-3.0.0-bin-hadoop3.2 spark-standalone
```

### 3.2.2 修改配置文件

1) 进入解压缩后路径的 conf 目录，修改 slaves.template 文件名为 slaves

```
mv slaves.template slaves
```

2) 修改 slaves 文件，添加 work 节点

```
linux1
linux2
linux3
```

3) 修改 spark-env.sh.template 文件名为 spark-env.sh

```
mv spark-env.sh.template spark-env.sh
```

4) 修改 spark-env.sh 文件，添加 JAVA_HOME 环境变量和集群对应的 master 节点

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
SPARK_MASTER_HOST=linux1
SPARK_MASTER_PORT=7077
```

==注意：7077 端口，相当于 hadoop3 内部通信的 8020 端口，此处的端口需要确认自己的 Hadoop 配置==

5) 分发 spark-standalone 目录 

```
xsync spark-standalone 
```

### 3.2.3 启动集群

**1) 执行脚本命令：** 

```
sbin/start-all.sh
```

**2) 查看三台服务器运行进程**

```
================linux1================
3330 Jps
3238 Worker
3163 Master
================linux2================
2966 Jps
2908 Worker
================linux3================
2978 Worker
3036 Jps
```

3) 查看 Master 资源监控 Web UI 界面: http://linux1:8080

![1659443581917](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443581917.png)

### 3.2.4 提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://linux1:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

1) --class 表示要执行程序的主类
2) --master spark://linux1:7077 独立部署模式，连接到 Spark 集群
3) spark-examples_2.12-3.0.0.jar 运行类所在的 jar 包
4) 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

### 3.2.5 提交参数说明

```
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \
[application-arguments]
```

![1659443623726](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443623726.png)

![1659443633686](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443633686.png)

### 3.2.6 配置历史服务

​		**由于 spark-shell 停止掉后，集群监控 linux1:4040 页面就看不到历史任务的运行情况，所以 开发时都配置历史服务器记录任务运行情况。** 

**1) 修改 spark-defaults.conf.template 文件名为 spark-defaults.conf** 

```
mv spark-defaults.conf.template spark-defaults.conf
```

**2) 修改 spark-default.conf 文件，配置日志存储路径** 

```
spark.eventLog.enabled true 
spark.eventLog.dir hdfs://linux1:8020/directory 
```

注意：需要启动 hadoop 集群，HDFS 上的 directory 目录需要提前存在。 

```
sbin/start-dfs.sh 
hadoop fs -mkdir /directory 
```

**3) 修改 spark-env.sh 文件, 添加日志配置** 

```
export SPARK_HISTORY_OPTS=" 
-Dspark.history.ui.port=18080  
-Dspark.history.fs.logDirectory=hdfs://linux1:8020/directory  
-Dspark.history.retainedApplications=30" 
```

⚫ 参数 1 含义：WEB UI 访问的端口号为 18080 

⚫ 参数 2 含义：指定历史服务器日志存储路径 

⚫ 参数 3 含义：指定保存 Application 历史记录的个数，如果超过这个值，旧的应用程序 

信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。 

**4) 分发配置文件** 

```
xsync conf 
```

**5) 重新启动集群和历史服务** 

```
sbin/start-all.sh 
sbin/start-history-server.sh 
```

**6) 重新执行任务** 

```
bin/spark-submit \ 
--class org.apache.spark.examples.SparkPi \ 
--master spark://linux1:7077 \ 
./examples/jars/spark-examples_2.12-3.0.0.jar \ 
10
```

7) 查看历史服务：http://linux1:18080 

![1659443759493](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443759493.png)

### 3.2.7 配置高可用（HA）

​		所谓的高可用是因为当前集群中的 Master 节点只有一个，所以会存在单点故障问题。所以为了解决单点故障问题，需要在集群中配置多个 Master 节点，一旦处于活动状态的 Master发生故障时，由备用 Master 提供服务，保证作业可以继续执行。这里的高可用一般采用Zookeeper 设置

![1659443777612](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443777612.png)

1) 停止集群

```
sbin/stop-all.sh
```

2) 启动 Zookeeper

```
xstart zk
```

3) 修改 spark-env.sh 文件添加如下配置

注释如下内容：

```
#SPARK_MASTER_HOST=linux1
#SPARK_MASTER_PORT=7077
添加如下内容:
```

```
#Master 监控页面默认访问端口为 8080，但是可能会和 Zookeeper 冲突，所以改成 8989，也可以自
定义，访问 UI 监控页面时请注意
SPARK_MASTER_WEBUI_PORT=8989
export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=linux1,linux2,linux3
-Dspark.deploy.zookeeper.dir=/spark"
```

4) 分发配置文件

```
xsync conf/
```

5) 启动集群

```
sbin/start-all.sh
```

![1659443888948](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443888948.png)

6) 启动 linux2 的单独 Master 节点，此时 linux2 节点 Master 状态处于备用状态

```
[root@linux2 spark-standalone]# sbin/start-master.sh
```

![1659443899890](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443899890.png)

7) 提交应用到高可用集群

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://linux1:7077,linux2:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

8) 停止 linux1 的 Master 资源监控进程

![1659443934564](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443934564.png)

9) 查看 linux2 的 Master 资源监控 Web UI，稍等一段时间后，linux2 节点的 Master 状态提升为活动状态

![1659443948180](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659443948180.png)

## 3.3 Yarn 模式

​		独立部署（Standalone）模式由 Spark 自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的 Yarn 环境下 Spark 是如何工作的（其实是因为在国内工作中，Yarn 使用的非常多）。

### 3.3.1 解压缩文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到 linux 并解压缩，放置在指定位置。

```
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module
cd /opt/module 
mv spark-3.0.0-bin-hadoop3.2 spark-yarn
```

### 3.3.2 修改配置文件

1) 修改 hadoop 配置文件/opt/module/hadoop/etc/hadoop/yarn-site.xml, 并分发

```
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
 <name>yarn.nodemanager.pmem-check-enabled</name>
 <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
 <name>yarn.nodemanager.vmem-check-enabled</name>
 <value>false</value>
</property>
```

2) 修改 conf/spark-env.sh，添加 JAVA_HOME 和 YARN_CONF_DIR 配置

```
 spark-env.sh.template spark-env.sh
。。。
export JAVA_HOME=/opt/module/jdk1.8.0_144
YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop
```

### 3.3.3 启动 HDFS 以及 YARN 集群

### 3.3.4 提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

查看 http://linux2:8088 页面，点击 History，查看历史页面

![1659444089612](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659444089612.png)

### 3.3.5 配置历史服务器

3.3.5 配置历史服务器 

1) 修改 spark-defaults.conf.template 文件名为 spark-defaults.conf

```
mv spark-defaults.conf.template spark-defaults.conf
```

2) 修改 spark-default.conf 文件，配置日志存储路径

```
spark.eventLog.enabled true
spark.eventLog.dir hdfs://linux1:8020/directory
```

注意：需要启动 hadoop 集群，HDFS 上的目录需要提前存在。

```
[root@linux1 hadoop]# sbin/start-dfs.sh
[root@linux1 hadoop]# hadoop fs -mkdir /directory
```

3) 修改 spark-env.sh 文件, 添加日志配置

```
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080 
-Dspark.history.fs.logDirectory=hdfs://linux1:8020/directory 
-Dspark.history.retainedApplications=30"
```

⚫ 参数 1 含义：WEB UI 访问的端口号为 18080
⚫ 参数 2 含义：指定历史服务器日志存储路径
⚫ 参数 3 含义：指定保存 Application 历史记录的个数，如果超过这个值，旧的应用程序
信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4) 修改 spark-defaults.conf

```
spark.yarn.historyServer.address=linux1:18080
spark.history.ui.port=18080
```

5) 启动历史服务

```
sbin/start-history-server.sh 
```

6) 重新提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.12-3.0.0.jar \
10
```

7) Web 页面查看日志：http://linux2:8088

![1659444177150](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659444177150.png)

## 3.5 Windows 模式

### 3.5.1 解压缩文件

将文件 spark-3.0.0-bin-hadoop3.2.tgz 解压缩到无中文无空格的路径中

### 3.5.2 启动本地环境

1) 执行解压缩文件路径下 bin 目录中的 spark-shell.cmd 文件，启动 Spark 本地环境

2) 在 bin 目录中创建 input 目录，并添加 word.txt 文件, 在命令行中输入脚本代码

### 3.5.3 命令行提交应用

在 DOS 命令行窗口中执行提交指令

```
spark-submit --class org.apache.spark.examples.SparkPi --master 
local[2] ../examples/jars/spark-examples_2.12-3.0.0.jar 10
```

## 3.6 部署模式对比

![1659444274354](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659444274354.png)

## 3.7 端口号

➢ Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算） 

➢ Spark Master 内部通信服务端口号：7077

➢ Standalone 模式下，Spark Master Web 端口号：8080（资源）

➢ Spark 历史服务器端口号：18080

➢ Hadoop YARN 任务运行情况查看端口号：8088

# 第4章Spark运行架构（19-24）

## 4.1运行架构

Spark 框架的核心是一个计算引擎，整体来说，它采用了标准 master-slave 的结构。 如下图所示，它展示了一个 

Spark 执行时的基本结构。图形中的 Driver 表示 master，

负责管理整个集群中的作业任务调度。图形中的 Executor 则是 slave，负责实际执行任务。

![1659495351850](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659495351850.png)

## 4.2核心组件

### 4.2.1Driver

Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作。 

Driver 在 Spark 作业执行时主要负责： 

➢ 将用户程序转化为作业（job） 

➢ 在 Executor 之间调度任务(task) 

➢ 跟踪 Executor 的执行情况 

➢ 通过 UI 展示查询运行情况 

实际上，我们无法准确地描述 Driver 的定义，因为在整个的编程过程中没有看到任何有关 Driver 的字眼。所以简单理解，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为Driver 类。

### 4.2.2Executor

Spark Executor 是集群中工作节点（Worker）中的一个 JVM 进程，负责在 Spark 作业 中运行具体任务（Task），任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了 故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点 上继续运行。 

**Executor 有两个核心功能：** 

➢ 负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程 

➢ 它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存 式存储。

​		RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算。 

### 4.2.3Master Worker

​		Spark 集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调 度的功能，所以环境中还有其他两个核心组件：Master 和 Worker，这里的 Master 是一个进 程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM, 而 Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对数据进行并行的处理和计算，==类似于 Yarn 环境中 NM==。 

### 4.2.4ApplicationMaster

​		Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含 ApplicationMaster，用 于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整 个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。 说的简单点就是，==ResourceManager（资源）和 Driver（计算）==之间的解耦合靠的就是 ApplicationMaster。

## 4.3核心概念

### 4.3.1 Executor与Core

​		Spark Executor 是集群中运行在工作节点（Worker）中的一个 JVM 进程，是整个集群中 的专门用于计算的节点。在提交应用中，可以提供参数指定计算节点的个数，以及对应的资 源。这里的资源一般指的是工作节点 Executor 的内存大小和使用的虚拟 CPU 核（Core）数量

![1659495653583](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659495653583.png)

### 4.3.2 并行度（Parallelism）

整个集群并行执行任务的数量称之为并行度

### 4.3.3 有向无环图（DAG）

![1659495817346](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659495817346.png)



其中第一类就是 Hadoop 所承载的 MapReduce,它将计算分为两个阶段，分别为 Map 阶段 和 Reduce 阶段。

支持 DAG 的框架被划分为第二代计算引擎。如 Tez 以及更上层的

第三代计 算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及实时计算。 



这里所谓的有向无环图，并不是真正意义的图形，而是由 Spark 程序直接映射成的数据 流的高级抽象模型。简单理解就是将整个程序计算的执行过程用图形表示出来,这样更直观，更便于理解，可以用于表示程序的拓扑结构



DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方 

向，不会闭环。 

## 4.4提交流程

将 Spark 引用部署到 Yarn 环境中会更多一些，所以本课程中的提交流程是基于 Yarn 环境的

![1659495983270](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659495983270.png)

Spark 应用程序提交到 Yarn 环境中执行的时候，一般会有两种部署执行的方式：Client 和 Cluster。两种模式主要区别在于：Driver 程序的运行节点位置。 

### 4.2.1 Yarn Client模式

Client 模式将用于监控和调度的 Driver 模块在客户端执行，而不是在 Yarn 中，所以一 般用于测试。 

### **4.2.2** Yarn Cluster 模式

Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于 实际生产环境。

# 第5章Spark核心编程（25-109）

## 5.1RDD

**Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，**用于 

处理不同的应用场景。三大数据结构分别是： 

➢ RDD : 弹性分布式数据集 

➢ 累加器：分布式共享只写变量 

➢ 广播变量：分布式共享只读变量 

接下来我们一起看看这三大数据结构是如何在数据处理中使用的。 

### 5.1.1什么是RDD

RDD（Resilient Distributed Dataset）叫做**弹性分布式数据集**，是 Spark 中最基本的==数据处理模型==。代码中是一个抽象类，**它代表一个弹性的、不可变、可分区、里面的元素可并行 计算的集合。** 

➢ 弹性 

⚫ 存储的弹性：内存与磁盘的自动切换； 

⚫ 容错的弹性：数据丢失可以自动恢复； 

⚫ 计算的弹性：计算出错重试机制； 

⚫ 分片的弹性：可根据需要重新分片。 

➢ 分布式：数据存储在大数据集群不同节点上 

➢ 数据集：RDD 封装了计算逻辑，并不保存数据 

➢ 数据抽象：RDD 是一个抽象类，需要子类具体实现 

➢ 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在新的 RDD 里面封装计算逻辑 

➢ 可分区、并行计算

![1659512958037](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659512958037.png)

![1659512978122](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659512978122.png)

### 5.1.2核心属性

**➢ 分区列表**

RDD 数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性。

**➢ 分区计算函数**

Spark 在计算时，是使用分区函数对每一个分区进行计算

**➢ RDD 之间的依赖关系**
RDD 是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个 RDD 建立依赖关系

**➢ 分区器（可选）**
当数据为 KV 类型数据时，可以通过设定分区器自定义数据的分区

**➢ 首选位置（可选）**

计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算

### 5.1.3执行原理

​		从计算的角度来讲，数据处理过程中需要计算资源（内存 & CPU）和计算模型（逻辑）。 执行时，需要将计算资源和计算模型进行协调和整合。 

​		Spark 框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的 计算任务。然后将任务发到已经分配资源的计算节点上, 按照指定的计算模型进行数据计 算。最后得到计算结果。 

​		RDD 是 Spark 框架中用于数据处理的核心模型，接下来我们看看，在 Yarn 环境中，RDD 的工作原理

1) 启动 Yarn 集群环境

![1659513501890](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659513501890.png)



2) Spark 通过申请资源创建调度节点和计算节点 



![1659513517988](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659513517988.png)

3) Spark 框架根据需求将计算逻辑根据分区划分成不同的任务

![1659513546892](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659513546892.png)

4) 调度节点将任务根据计算节点状态发送到对应的计算节点进行计算 

![1659513560256](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659513560256.png)

​		从以上流程可以看出 RDD 在整个流程中主要用于将逻辑进行封装，并生成 Task 发送给 Executor 节点执行计算，接下来我们就一起看看 Spark 框架中 RDD 是具体是如何进行数据 处理的。

### 5.1.4基础编程

#### 5.1.4.1RDD创建

**1)从集合（内存）中创建RDD（常用）**

从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法 

**2)从外部存储（文件）创建RDD（常用）**

由外部存储系统的数据集创建 RDD 包括：本地的文件系统，所有 Hadoop 支持的数据集， 比如 HDFS、HBase 等

**3)从其他RDD创建**

主要是通过一个 RDD 运算完后，再产生新的 RDD。详情请参考后续章节 

**4)直接创建RDD(new)**

使用 new 的方式直接构造 RDD，一般由 Spark 框架自身使用。 

#### 5.1.4.2RDD并行度与分区

默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能 够并行计算的任务数量我们称之为并行度。这个数量可以在构建 RDD 时指定。记住，这里的并行执行的任务数量，并不是指的切分任务的数量，不要混淆了。

⚫ 读取内存数据时，数据可以按照并行度的设定进行数据的分区操作，数据分区规则的

⚫ 读取文件数据时，数据是按照 Hadoop 文件读取的规则进行切片分区，而切片规则和数据读取的规则有些差异

#### 5.1.4.3RDD转换算子（41-77）

![1659581127248](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659581127248.png)

RDD 根据数据处理方式的不同将算子整体上分为 Value 类型、双 Value 类型和 Key-Value 类型

==value类型==

**1)map**

➢ 函数说明 

将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。 

❖ 小功能：从服务器日志数据 apache.log 中获取用户请求 URL 资源路径 

```
    //TODO 算子 -Map
    val rdd = sc.makeRDD(List(1,2,3,4))

    //转换函数
    def mapFunction(num:Int):Int={num*2}
    
    //进行转化
    val mapadd = rdd.map(mapFunction)
    mapadd.collect().foreach(println)
```

分区操作

```
    //TODO 算子 -Map
    // 1. rdd的计算一个分区内的数据是一个一个执行逻辑
    //    只有前面一个数据全部的逻辑执行完毕后，才会执行下一个数据。
    //    分区内数据的执行是有序的。
    // 2. 不同分区数据计算是无序的。
    val rdd = sc.makeRDD(List(1,2,3,4),2)

    val mapRDD = rdd.map(
      num => {
        println(">>>>>>>" + num)
        num
      }
    )
    val mapADD2 = mapRDD.map(
      num => {
        println("######" + num)
        num
      }
    )
```

**2)mapPartitions**

​		将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。 

❖ 小功能：获取每个数据分区的最大值

➢ 数据处理角度 

Map 算子是分区内一个数据一个数据的执行，类似于串行操作。而 mapPartitions 算子 是以分区为单位进行批处理操作。 

➢ 功能的角度 

Map 算子主要目的将数据源中的数据进行转换和改变。但是不会减少或增多数据。 

MapPartitions 算子需要传递一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变， 

所以可以增加或减少数据 

➢ 性能的角度 

Map 算子因为类似于串行操作，所以性能比较低，而是 mapPartitions 算子类似于批处 

理，所以性能较高。但是 mapPartitions 算子会长时间占用内存，那么这样会导致内存可能 

不够用，出现内存溢出的错误。所以在内存有限的情况下，不推荐使用。使用 map 操作。

**3) mapPartitionsWithIndex**

➢ 函数说明 

将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处 理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。 

**4)flatMap**

将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射

```
def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("operator")
    val sc=new SparkContext(sparkConf)
    //TODO 算子-flatMap
    val rdd =sc.makeRDD(List(List(1,2),3,List(4,5)))

    val mapRDD = rdd.flatMap(
      data => {
        data match {
          case list: List[_]=>list
          case dat=>List(dat)
        }
      }
    )

    mapRDD.collect().foreach(println)
    sc.stop()
  }
```

**5)glom**

将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

❖ 小功能：计算所有分区最大值求和（分区内取最大值，分区间最大值求和） 

```
object Spark05_RDD_Operator_Transform_Test {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("operator")
    val sc=new SparkContext(sparkConf)
    //TODO 算子-flatMap
    val rdd =sc.makeRDD(List(1,2,3,3),2)

    val glomRDD:RDD[Array[Int]] = rdd.glom()

    val value = glomRDD.map(
      array => {
        array.max
      }
    )
    println(value.collect().sum)

    sc.stop()
  }
}
```

**6)groupBy**

将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样 的操作称之为shuffle。极限情况下，数据可能被分在同一个分区中 一个组的数据在一个分区中，但是并不是说一个分区中只有一个组 

❖ 小功能：将 List("Hello", "hive", "hbase", "Hadoop")根据单词首写字母进行分组。 

❖ 小功能：从服务器日志数据 apache.log 中获取每个时间段访问量。 

❖ 小功能：WordCount。

```
object Spark06_RDD_Operator_Transform_Test {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("operator")
    val sc=new SparkContext(sparkConf)

    //TODO 算子 -Map
    val rdd = sc.textFile("datas/apache.log")

    val timeRDD:RDD[(String, Iterable[(String, Int)])] = rdd.map(
      line => {
        val datas = line.split(" ")
        val time = datas(3)
        val sdf = new SimpleDateFormat("dd/MM/yyy:HH:mm:ss")
        val date: Date = sdf.parse(time)
        val sdf1 = new SimpleDateFormat("HH")
        val hour: String = sdf1.format(date)
        (hour, 1)
      }
    ).groupBy(_._1)
    timeRDD.map{
      case (hour,iter)=>{
        (hour,iter.size)
      }
    }.collect.foreach(println)
  }
}

```

**7)filter**

➢ 函数说明 

将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。 

当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出 现数据倾斜。 

❖ 小功能：从服务器日志数据 apache.log 中获取 2015 年 5 月 17 日的请求路径

```
  def main(args: Array[String]): Unit = {

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
    val sc = new SparkContext(sparkConf)

    // TODO 算子 - filter
    val rdd = sc.textFile("datas/apache.log")

    rdd.filter(
      line => {
        val datas = line.split(" ")
        val time = datas(3)
        time.startsWith("17/05/2015")
      }
    ).collect().foreach(println)
    sc.stop()
  }
```

**8)sample**

根据指定的规则从数据集中抽取数据

```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - filter
        val rdd = sc.makeRDD(List(1,2,3,4,5,6,7,8,9,10))

        // sample算子需要传递三个参数
        // 1. 第一个参数表示，抽取数 据后是否将数据返回 true（放回），false（丢弃）
        // 2. 第二个参数表示，
        //         如果抽取不放回的场合：数据源中每条数据被抽取的概率，基准值的概念
        //         如果抽取放回的场合：表示数据源中的每条数据被抽取的可能次数
        // 3. 第三个参数表示，抽取数据时随机算法的种子
        //                    如果不传递第三个参数，那么使用的是当前系统时间
//        println(rdd.sample(
//            false,
//            0.4
//            //1
//        ).collect().mkString(","))

        println(rdd.sample(
            true,
            2
            //1
        ).collect().mkString(","))


        sc.stop()

    }
```



**9)distinct**

将数据集中重复的数据去重

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - filter
        val rdd = sc.makeRDD(List(1,2,3,4,1,2,3,4))

        // map(x => (x, null)).reduceByKey((x, _) => x, numPartitions).map(_._1)

        // (1, null),(2, null),(3, null),(4, null),(1, null),(2, null),(3, null),(4, null)
        // (1, null)(1, null)(1, null)
        // (null, null) => null
        // (1, null) => 1
        val rdd1: RDD[Int] = rdd.distinct()

        rdd1.collect().foreach(println)



        sc.stop()

    }
```



**10)coalesce**

➢ 函数说明 

根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率 

当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，

减少 分区的个数，减小任务调度成本

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - filter
        val rdd = sc.makeRDD(List(1,2,3,4,5,6), 3)

        // coalesce方法默认情况下不会将分区的数据打乱重新组合
        // 这种情况下的缩减分区可能会导致数据不均衡，出现数据倾斜
        // 如果想要让数据均衡，可以进行shuffle处理
        //val newRDD: RDD[Int] = rdd.coalesce(2)
        val newRDD: RDD[Int] = rdd.coalesce(2,true)

        newRDD.saveAsTextFile("output")

        sc.stop()

    }
```



**11)repartition**

该操作内部其实执行的是 coalesce 操作，参数 shuffle 的默认值为 true。无论是将分区数多的 

RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD，repartition 

操作都可以完成，因为无论如何都会经 shuffle 过程。 .

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - filter
        val rdd = sc.makeRDD(List(1,2,3,4,5,6), 2)

        // coalesce算子可以扩大分区的，但是如果不进行shuffle操作，是没有意义，不起作用。
        // 所以如果想要实现扩大分区的效果，需要使用shuffle操作
        // spark提供了一个简化的操作
        // 缩减分区：coalesce，如果想要数据均衡，可以采用shuffle
        // 扩大分区：repartition, 底层代码调用的就是coalesce，而且肯定采用shuffle
        //val newRDD: RDD[Int] = rdd.coalesce(3, true)
        val newRDD: RDD[Int] = rdd.repartition(3)

        newRDD.saveAsTextFile("output")
        
        sc.stop()

    }
```

**12)sortBy**

该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理 的结果进行排序，默认为升序排列。排序后新产生的 RDD 的分区数与原 RDD 的分区数一 致。中间存在 shuffle 的过程 

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - sortBy
        val rdd = sc.makeRDD(List(6,2,4,5,3,1), 2)

        val newRDD: RDD[Int] = rdd.sortBy(num=>num)

        newRDD.saveAsTextFile("output")
        
        sc.stop()

    }
```

==双Value类型==

**13) intersection**

对源 RDD 和参数 RDD 求交集后返回一个新的 RDD



**14)union**

对源 RDD 和参数 RDD 求并集后返回一个新的 RDD



**15)subtract**

以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集



**16)zip**

将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD 

中的元素，Value 为第 2 个 RDD 中的相同位置的元素。 

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - 双Value类型

        // 交集，并集和差集要求两个数据源数据类型保持一致
        // 拉链操作两个数据源的类型可以不一致

        val rdd1 = sc.makeRDD(List(1,2,3,4))
        val rdd2 = sc.makeRDD(List(3,4,5,6))
        val rdd7 = sc.makeRDD(List("3","4","5","6"))

        // 交集 : 【3，4】
        val rdd3: RDD[Int] = rdd1.intersection(rdd2)
        //val rdd8 = rdd1.intersection(rdd7)
        println(rdd3.collect().mkString(","))

        // 并集 : 【1，2，3，4，3，4，5，6】
        val rdd4: RDD[Int] = rdd1.union(rdd2)
        println(rdd4.collect().mkString(","))

        // 差集 : 【1，2】
        val rdd5: RDD[Int] = rdd1.subtract(rdd2)
        println(rdd5.collect().mkString(","))
        
        // Can't zip RDDs with unequal numbers of partitions: List(2, 4)
        // 两个数据源要求分区数量要保持一致
        // Can only zip RDDs with same number of elements in each partition
        // 两个数据源要求分区中数据数量保持一致
        // 拉链 : 【1-3，2-4，3-5，4-6】
        val rdd6: RDD[(Int, Int)] = rdd1.zip(rdd2)
        val rdd8 = rdd1.zip(rdd7)
        println(rdd6.collect().mkString(","))
        sc.stop()

    }
```

==Key-Value类型==

**17) partitionBy**

将数据按照指定 Partitioner 重新进行分区。Spark 默认的分区器是 HashPartitioner

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)
        val rdd = sc.makeRDD(List(1,2,3,4),2)

        val mapRDD:RDD[(Int, Int)] = rdd.map((_,1))
        // RDD => PairRDDFunctions
        // 隐式转换（二次编译）

        // partitionBy根据指定的分区规则对数据进行重分区
        val newRDD = mapRDD.partitionBy(new HashPartitioner(2))
        newRDD.partitionBy(new HashPartitioner(2))

        newRDD.saveAsTextFile("output")
        sc.stop()
    }
```

**18)reduceByKey**

可以将数据按照相同的 Key 对 Value 进行聚合

```
 def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("a", 3), ("b", 4)
        ))

        // reduceByKey : 相同的key的数据进行value数据的聚合操作
        // scala语言中一般的聚合操作都是两两聚合，spark基于scala开发的，所以它的聚合也是两两聚合
        // 【1，2，3】
        // 【3，3】
        // 【6】
        // reduceByKey中如果key的数据只有一个，是不会参与运算的。
        val reduceRDD: RDD[(String, Int)] = rdd.reduceByKey( (x:Int, y:Int) => {
            x + y
        } )
        reduceRDD.collect().foreach(println)
        sc.stop()

    }
```

**19)groupByKey**

将数据源的数据根据 key 对 value 进行分组 

```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("a", 3), ("b", 4)
        ))

        // groupByKey : 将数据源中的数据，相同key的数据分在一个组中，形成一个对偶元组
        //              元组中的第一个元素就是key，
        //              元组中的第二个元素就是相同key的value的集合
        val groupRDD: RDD[(String, Iterable[Int])] = rdd.groupByKey()

        groupRDD.collect().foreach(println)

        val groupRDD1: RDD[(String, Iterable[(String, Int)])] = rdd.groupBy(_._1)

        
        sc.stop()

    }
```

**20)aggregateByKey**

将数据根据不同的规则进行分区内计算和分区间计算

```

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("a", 3), ("a", 4)
        ),2)
        // (a,【1,2】), (a, 【3，4】)
        // (a, 2), (a, 4)
        // (a, 6)

        // aggregateByKey存在函数柯里化，有两个参数列表
        // 第一个参数列表,需要传递一个参数，表示为初始值
        //       主要用于当碰见第一个key的时候，和value进行分区内计算
        // 第二个参数列表需要传递2个参数
        //      第一个参数表示分区内计算规则
        //      第二个参数表示分区间计算规则

        // math.min(x, y)
        // math.max(x, y)
        rdd.aggregateByKey(0)(
            (x, y) => math.max(x, y),
            (x, y) => x + y
        ).collect.foreach(println)
        
        sc.stop()
    }
```

**21)foldByKey**

当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey

```
    def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("b", 3),
            ("b", 4), ("b", 5), ("a", 6)
        ),2)

        //rdd.aggregateByKey(0)(_+_, _+_).collect.foreach(println)

        // 如果聚合计算时，分区内和分区间计算规则相同，spark提供了简化的方法
        rdd.foldByKey(0)(_+_).collect.foreach(println)
        
        sc.stop()
    }
```



```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("b", 3),
            ("b", 4), ("b", 5), ("a", 6)
        ),2)

        // aggregateByKey最终的返回数据结果应该和初始值的类型保持一致
        //val aggRDD: RDD[(String, String)] = rdd.aggregateByKey("")(_ + _, _ + _)
        //aggRDD.collect.foreach(println)

        // 获取相同key的数据的平均值 => (a, 3),(b, 4)
        val newRDD : RDD[(String, (Int, Int))] = rdd.aggregateByKey( (0,0) )(
            ( t, v ) => {
                (t._1 + v, t._2 + 1)
            },
            (t1, t2) => {
                (t1._1 + t2._1, t1._2 + t2._2)
            }
        )

        val resultRDD: RDD[(String, Int)] = newRDD.mapValues {
            case (num, cnt) => {
                num / cnt
            }
        }
        resultRDD.collect().foreach(println)
        sc.stop()
    }
```

**22)combineByKey**

最通用的对 key-value 型 rdd 进行聚集操作的聚集函数（aggregation function）。类似于 

aggregate()，combineByKey()允许用户返回值的类型与输入不一致。 

小练习：将数据 List(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98))求每个 key 的平均值

```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("b", 3),
            ("b", 4), ("b", 5), ("a", 6)
        ),2)

        // combineByKey : 方法需要三个参数
        // 第一个参数表示：将相同key的第一个数据进行结构的转换，实现操作
        // 第二个参数表示：分区内的计算规则
        // 第三个参数表示：分区间的计算规则
        val newRDD : RDD[(String, (Int, Int))] = rdd.combineByKey(
            v => (v, 1),
            ( t:(Int, Int), v ) => {
                (t._1 + v, t._2 + 1)
            },
            (t1:(Int, Int), t2:(Int, Int)) => {
                (t1._1 + t2._1, t1._2 + t2._2)
            }
        )

        val resultRDD: RDD[(String, Int)] = newRDD.mapValues {
            case (num, cnt) => {
                num / cnt
            }
        }
        resultRDD.collect().foreach(println)
        
        sc.stop()
    }
```

**23)sortByKey**

在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序的 



**24)join**

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的 (K,(V,W))的 RDD 

```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd1 = sc.makeRDD(List(
            ("a", 1), ("a", 2), ("c", 3)
        ))

        val rdd2 = sc.makeRDD(List(
            ("a", 5), ("c", 6),("a", 4)
        ))

        // join : 两个不同数据源的数据，相同的key的value会连接在一起，形成元组
        //        如果两个数据源中key没有匹配上，那么数据不会出现在结果中
        //        如果两个数据源中key有多个相同的，会依次匹配，可能会出现笛卡尔乘积，数据量会几何性增长，会导致性能降低。
        val joinRDD: RDD[(String, (Int, Int))] = rdd1.join(rdd2)

        joinRDD.collect().foreach(println)
        
        sc.stop()

    }
```

**25)leftOuterJoin**

类似于 SQL 语句的左外连接

```
def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd1 = sc.makeRDD(List(
            ("a", 1), ("b", 2)//, ("c", 3)
        ))

        val rdd2 = sc.makeRDD(List(
            ("a", 4), ("b", 5),("c", 6)
        ))
        //val leftJoinRDD = rdd1.leftOuterJoin(rdd2)
        val rightJoinRDD = rdd1.rightOuterJoin(rdd2)

        //leftJoinRDD.collect().foreach(println)
        rightJoinRDD.collect().foreach(println)
        
        sc.stop()

    }
```

**26)cogroup**

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD

```
 def main(args: Array[String]): Unit = {

        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
        val sc = new SparkContext(sparkConf)

        // TODO 算子 - (Key - Value类型)

        val rdd1 = sc.makeRDD(List(
            ("a", 1), ("b", 2)//, ("c", 3)
        ))

        val rdd2 = sc.makeRDD(List(
            ("a", 4), ("b", 5),("c", 6),("c", 7)
        ))

        // cogroup : connect + group (分组，连接)
        val cgRDD: RDD[(String, (Iterable[Int], Iterable[Int]))] = rdd1.cogroup(rdd2)

        cgRDD.collect().foreach(println)
        sc.stop()

    }
```

#### 5.1.4.4案例实操

![1659673563499](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659673563499.png)

#### 5.1.4.5RDD行动算子

**1)reduce**

聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据



**2)collect**

在驱动程序中，以数组 Array 的形式返回数据集的所有元素

```scala
def main(args: Array[String]): Unit = {

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
    val sc=new SparkContext(sparkConf)

    val rdd=sc.makeRDD(List(1,2,3,4))

    //ToDo-行动算子
    //所谓的行动算子，其实就是触发作业（JOb）执行的方法
    //底层代码调用的是环境对象的runJob
    //底层代码中会创建ActionJob，并提交执行
    rdd.collect()
    sc.stop()
  }
```

**3)count**

返回 RDD 中元素的个数 

```
 //count:数据源中数据的个数
    val cnt = rdd.count()
    println(cnt)
```

**4)first**

返回 RDD 中的第一个元素 

```
    //first:获取数据源中数据的第一个
    val first = rdd.first()
    println(first)
```

**5)take**

返回一个由 RDD 的前 n 个元素组成的数组

```
   //take:获取前N个数据
    val ints1 = rdd.take(3)
    println(ints.mkString(","))
```

**6)takeOrdered**

返回该 RDD 排序后的前 n 个元素组成的数组

```
    //takeOrdered:数据排序后，取N个数据
    val rdd1 = sc.makeRDD(List(4, 2, 3, 1))
    val ints2:Array[Int]=rdd1.takeOrdered(3)
    println(ints1.mkString(","))
```

**7)aggregate**

分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

```scala
  def main(args: Array[String]): Unit = {

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
    val sc=new SparkContext(sparkConf)

    val rdd=sc.makeRDD(List(1,2,3,4),2)

    //TODO-行动算子
    //aggregateByKey:初始值只会参与分区内计算
    //aggregate：初始值会参与分区内计算，并且和参与分区间计算
    //val result=rdd.aggregate(10)(_+_,_+_) //设置分区内计算和分区间计算
    val result=rdd.fold(10)(_+_)
    println(result)
    sc.stop()
  }
```

**8)fold**

折叠操作，aggregate 的简化版操作 

**9)countByKey**

统计每种 key 的个数

```scala
    //TODO-行动算子
    //val stringToLong = rdd.countByValue()
    val stringToLong = rdd.countByKey()
    println(stringToLong)
```

**10)save相关算子**

将数据保存到不同格式的文件中

```scala
    //TODO-行动算子
    //val stringToLong = rdd.countByValue()
    rdd.saveAsTextFile("output")
    rdd.saveAsObjectFile("output1")
    //saveAsSequenceFile方法要求数据的格式必须为K-V类型
    rdd.saveAsSequenceFile("output2")
```

**11)foreach**

分布式遍历 RDD 中的每一个元素，调用指定函数

```
    //TODO-行动算子
    //foreach其实是Dirver端内存集合的循环遍历方法
    rdd.foreach(println)
    println("**************")

    //算子：Operator(操作)
    //      RDD的方法和Scala集合对象的方法不一样
    //      集合对象的方法都是在同一个节点的内存中完成的
    //      RDD的方法可以将计算逻辑发送到Excutoe端（分布式节点）执行
    //      为了区分不同的处理效果，所以将RDD的方法称之为算子
    //      RDD的方法外部的操作都是Driver端执行的，而方法内部的逻辑代码是在Excutor端执行的

    sc.stop()
```



#### 5.1.4.6RDD序列化

**1) 闭包检查** 

​		从计算的角度, ==算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor 端执行==。那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就 形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor 端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列 化，这个操作我们称之为闭包检测。Scala2.12 版本后闭包编译方式发生了改变 。

**2) 序列化方法和属性** 

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行，看如下代码：

```scala
def main(args: Array[String]): Unit = {

    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
    val sc=new SparkContext(sparkConf)

    val rdd=sc.makeRDD(List(1,2,3))

    val user=new User()

    //RDD算子中传递的函数是会包含闭包操作，name就会进行检测功能
    //闭包检测

    rdd.foreach(
      num=>{
        println("age="+(user.age+num))
      }
    )

    //TODO-行动算子
    sc.stop()
  }
  //样例类在编译时，会自动混入序列化特质（实现可序列化接口）
  class User extends Serializable {
    var age:Int=30
  }
```

**3) Kryo 序列化框架** 

参考地址: https://github.com/EsotericSoftware/kryo 

​		Java 的序列化能够序列化任何的类。但是比较重（字节多），序列化后，对象的提交也 比较大。Spark 出于性能的考虑，Spark2.0 开始支持另外一种 Kryo 序列化机制。Kryo 速度 是 Serializable 的 10 倍。当 RDD 在 Shuffle 数据的时候，简单数据类型、数组和字符串类型 已经在 Spark 内部使用 Kryo 来序列化。 

注意：即使使用 Kryo 序列化，也要继承 Serializable 接口

#### 5.1.4.7RDD依赖关系

![1659750543197](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750543197.png)

![1659750590371](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750590371.png)

![1659750611831](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750611831.png)

**1) RDD** **血缘关系** 

​			RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage （血统）记录下来，以便恢复丢失的分区。RDD 的 Lineage 会记录 RDD 的元数据信息和转 换行为，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区

```
  def main(args: Array[String]): Unit = {

    val sparConf = new SparkConf().setMaster("local").setAppName("WordCount")
    val sc = new SparkContext(sparConf)

    val lines: RDD[String] = sc.textFile("datas/1.txt")
    println(lines.toDebugString)
    println("*******************")

    val words = lines.flatMap(_.split(" "))
    println(lines.toDebugString)
    println("*******************")

    val wordToOne = words.map(word => (word, 1))
    println(lines.toDebugString)
    println("*******************")

    val wordToSum: RDD[(String, Int)] = wordToOne.reduceByKey(_ + _)
    println(lines.toDebugString)
    println("*******************")
    val array = wordToSum.collect()
    array.foreach(println)

  }
```

**2)RDD依赖关系**

​		这里所谓的依赖关系，其实就是两个相邻 RDD 之间的关系 





**3)RDD窄依赖**

​		窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游）RDD 的一个 Partition 使用， 窄依赖我们形象的比喻为独生子女





**4)RDD宽依赖**

​		宽依赖表示同一个父（上游）RDD 的 Partition 被多个子（下游）RDD 的 Partition 依赖，会 引起 Shuffle，总结：宽依赖我们形象的比喻为多生



**5)RDD阶段划分**

​		DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向， 不会闭环。例如，DAG 记录了 RDD 的转换过程和任务的阶段。



**6)RDD阶段划分源码**

**7) RDD 任务划分**

**RDD 任务切分中间分为：Application、Job、Stage 和 Task** 

⚫ Application：初始化一个 SparkContext 即生成一个 Application； 

⚫ Job：一个 Action 算子就会生成一个 Job； 

⚫ Stage：Stage 等于宽依赖(ShuffleDependency)的个数加 1； 

⚫ Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。 

==注意：Application->Job->Stage->Task 每一层都是 1 对 n 的关系==

![1659752186768](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659752186768.png)

**8)RDD任务划分源码**

#### 5.1.4.8RDD持久化

![1659750700921](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750700921.png)



**1) RDD Cache 缓存** 

​		RDD 通过 Cache 或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存 在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。 

```scala
    def main(args: Array[String]): Unit = {
        val sparConf = new SparkConf().setMaster("local").setAppName("Persist")
        val sc = new SparkContext(sparConf)

        val list = List("Hello Scala", "Hello Spark")

        val rdd = sc.makeRDD(list)

        val flatRDD = rdd.flatMap(_.split(" "))

        val mapRDD = flatRDD.map(word=>{
            println("@@@@@@@@@@@@")
            (word,1)
        })
        // cache默认持久化的操作，只能将数据保存到内存中，如果想要保存到磁盘文件，需要更改存储级别
        //mapRDD.cache()

        // 持久化操作必须在行动算子执行时完成的。
        mapRDD.persist(StorageLevel.DISK_ONLY)

        val reduceRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)
        reduceRDD.collect().foreach(println)
        println("**************************************")
        val groupRDD = mapRDD.groupByKey()
        groupRDD.collect().foreach(println)
        
        sc.stop()
    }
```



**2)RDD CheckPoint检查点**

​		所谓的检查点其实就是通过将 RDD 中间结果写入磁盘 由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点 之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。 对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发。 

```scala
def main(args: Array[String]): Unit = {
        val sparConf = new SparkConf().setMaster("local").setAppName("Persist")
        val sc = new SparkContext(sparConf)
        sc.setCheckpointDir("cp")

        val list = List("Hello Scala", "Hello Spark")

        val rdd = sc.makeRDD(list)

        val flatRDD = rdd.flatMap(_.split(" "))

        val mapRDD = flatRDD.map(word=>{
            println("@@@@@@@@@@@@")
            (word,1)
        })
        // checkpoint 需要落盘，需要指定检查点保存路径
        // 检查点路径保存的文件，当作业执行完毕后，不会被删除
        // 一般保存路径都是在分布式存储系统：HDFS
        mapRDD.checkpoint()

        val reduceRDD: RDD[(String, Int)] = mapRDD.reduceByKey(_+_)
        reduceRDD.collect().foreach(println)
        println("**************************************")
        val groupRDD = mapRDD.groupByKey()
        groupRDD.collect().foreach(println)


        sc.stop()
    }
```



**3)缓存和检查点区别**

1）Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。 

2）Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存 储在 HDFS 等容错、高可用的文件系统，可靠性高。 

3）建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存 中读取数据即可，否则需要再从头计算一次 RDD。



#### 5.1.4.9RDD分区器

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认 分区。分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了 Reduce 的个数。 

➢ 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None 

➢ 每个 RDD 的分区 ID 范围：0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。 

1) Hash 分区：对于给定的 key，计算其 hashCode,并除以分区个数取余

2) Range 分区：将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而
   且分区间有序

```scala
def main(args: Array[String]): Unit = {
        val sparConf = new SparkConf().setMaster("local").setAppName("WordCount")
        val sc = new SparkContext(sparConf)

        val rdd = sc.makeRDD(List(
            ("nba", "xxxxxxxxx"),
            ("cba", "xxxxxxxxx"),
            ("wnba", "xxxxxxxxx"),
            ("nba", "xxxxxxxxx"),
        ),3)
        val partRDD: RDD[(String, String)] = rdd.partitionBy( new MyPartitioner )

        partRDD.saveAsTextFile("output")

        sc.stop()
    }

    /**
      * 自定义分区器
      * 1. 继承Partitioner
      * 2. 重写方法
      */
    class MyPartitioner extends Partitioner{
        // 分区数量
        override def numPartitions: Int = 3

        // 根据数据的key值返回数据所在的分区索引（从0开始）
        override def getPartition(key: Any): Int = {
            key match {
                case "nba" => 0
                case "wnba" => 1
                case _ => 2
            }
        }
    }
```



#### 5.1.4.10RDD文件读取与保存

Spark 的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。 

文件格式分为：text 文件、csv 文件、sequence 文件以及 Object 文件； 

文件系统分为：本地文件系统、HDFS、HBASE 以及数据库。

➢ sequence 文件
SequenceFile 文件是 Hadoop 用来存储二进制形式的 key-value 对而设计的一种平面文件(Flat 
File)。在 SparkContext 中，可以调用 sequenceFile[keyClass, valueClass](path)。

➢ object 对象文件
对象文件是将对象序列化后保存的文件，采用 Java 的序列化机制。可以通过 objectFile[T: 
ClassTag](path)函数接收一个路径，读取对象文件，返回对应的 RDD，也可以通过调用
saveAsObjectFile()实现对对象文件的输出。因为是序列化所以要指定类型



## 5.2累加器

![1659750674138](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750674138.png)

### 5.2.1实现原理

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行 merge。

### 5.2.2基础编程

```scala
  def main(args: Array[String]): Unit = {

        val sparConf = new SparkConf().setMaster("local").setAppName("Acc")
        val sc = new SparkContext(sparConf)

        val rdd = sc.makeRDD(List(1,2,3,4))

        // 获取系统累加器
        // Spark默认就提供了简单数据聚合的累加器
        val sumAcc = sc.longAccumulator("sum")

        //sc.doubleAccumulator
        //sc.collectionAccumulator

        val mapRDD = rdd.map(
            num => {
                // 使用累加器
                sumAcc.add(num)
                num
            }
        )
        // 获取累加器的值
        // 少加：转换算子中调用累加器，如果没有行动算子的话，那么不会执行
        // 多加：转换算子中调用累加器，如果没有行动算子的话，那么不会执行
        // 一般情况下，累加器会放置在行动算子进行操作
        mapRDD.collect()
        mapRDD.collect()
        println(sumAcc.value)

        sc.stop()
    }
```



## 5.3 广播变量

![1659750659638](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659750659638.png)

### 5.3.1 实现原理

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个 或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表， 广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送

### 5.3.2基础编程

```scala
 def main(args: Array[String]): Unit = {

        val sparConf = new SparkConf().setMaster("local").setAppName("Acc")
        val sc = new SparkContext(sparConf)

        val rdd1 = sc.makeRDD(List(
            ("a", 1),("b", 2),("c", 3)
        ))
        val map = mutable.Map(("a", 4),("b", 5),("c", 6))

        // 封装广播变量
        val bc: Broadcast[mutable.Map[String, Int]] = sc.broadcast(map)

        rdd1.map {
            case (w, c) => {
                // 方法广播变量
                val l: Int = bc.value.getOrElse(w, 0)
                (w, (c, l))
            }
        }.collect().foreach(println)



        sc.stop()

    }
```

# 第六章：案例实操（110-127）

​		在之前的学习中，我们已经学习了 Spark 的基础编程方式，接下来，我们看看在实际的工作中如何使用这些 API 实现具体的需求。这些需求是电商网站的真实需求，所以在实现功能前，咱们必须先将数据准备好。

![1659753958166](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659753958166.png)

上面的数据图是从数据文件中截取的一部分内容，表示为电商网站的用户行为数据，主 

要包含用户的 4 种行为：搜索，点击，下单，支付。数据规则如下： 

➢ 数据文件中每行数据采用下划线分隔数据 

➢ 每一行数据表示用户的一次行为，这个行为只能是 4 种行为的一种 

➢ 如果搜索关键字为 null,表示数据不是搜索数据 

➢ 如果点击的品类 ID 和产品 ID 为-1，表示数据不是点击数据 

➢ 针对于下单行为，一次可以下单多个商品，所以品类 ID 和产品 ID 可以是多个，id 之 

间采用逗号分隔，如果本次不是下单行为，则数据采用 null 表示 

➢ 支付行为和下单行为类似

## 6.1 需求 1：Top10 热门品类

### 6.1.1 需求说明



### 6.1.2 实现方案一

#### 6.1.2.1 需求分析

分别统计每个品类点击的次数，下单的次数和支付的次数：（品类，点击总数）（品类，下单总数）（品类，支付总数）

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Spark01_Req1_HotCategoryTop10Analysis {

    def main(args: Array[String]): Unit = {

        // TODO : Top10热门品类
        val sparConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
        val sc = new SparkContext(sparConf)

        // 1. 读取原始日志数据
        val actionRDD = sc.textFile("datas/user_visit_action.txt")

        // 2. 统计品类的点击数量：（品类ID，点击数量）
        val clickActionRDD = actionRDD.filter(
            action => {
                val datas = action.split("_")
                datas(6) != "-1"
            }
        )

        val clickCountRDD: RDD[(String, Int)] = clickActionRDD.map(
            action => {
                val datas = action.split("_")
                (datas(6), 1)
            }
        ).reduceByKey(_ + _)

        // 3. 统计品类的下单数量：（品类ID，下单数量）
        val orderActionRDD = actionRDD.filter(
            action => {
                val datas = action.split("_")
                datas(8) != "null"
            }
        )

        // orderid => 1,2,3
        // 【(1,1)，(2,1)，(3,1)】
        val orderCountRDD = orderActionRDD.flatMap(
            action => {
                val datas = action.split("_")
                val cid = datas(8)
                val cids = cid.split(",")
                cids.map(id=>(id, 1))
            }
        ).reduceByKey(_+_)

        // 4. 统计品类的支付数量：（品类ID，支付数量）
        val payActionRDD = actionRDD.filter(
            action => {
                val datas = action.split("_")
                datas(10) != "null"
            }
        )

        // orderid => 1,2,3
        // 【(1,1)，(2,1)，(3,1)】
        val payCountRDD = payActionRDD.flatMap(
            action => {
                val datas = action.split("_")
                val cid = datas(10)
                val cids = cid.split(",")
                cids.map(id=>(id, 1))
            }
        ).reduceByKey(_+_)

        // 5. 将品类进行排序，并且取前10名
        //    点击数量排序，下单数量排序，支付数量排序
        //    元组排序：先比较第一个，再比较第二个，再比较第三个，依此类推
        //    ( 品类ID, ( 点击数量, 下单数量, 支付数量 ) )
        //
        //  cogroup = connect + group
        val cogroupRDD: RDD[(String, (Iterable[Int], Iterable[Int], Iterable[Int]))] =
            clickCountRDD.cogroup(orderCountRDD, payCountRDD)
        val analysisRDD = cogroupRDD.mapValues{
            case ( clickIter, orderIter, payIter ) => {

                var clickCnt = 0
                val iter1 = clickIter.iterator
                if ( iter1.hasNext ) {
                    clickCnt = iter1.next()
                }
                var orderCnt = 0
                val iter2 = orderIter.iterator
                if ( iter2.hasNext ) {
                    orderCnt = iter2.next()
                }
                var payCnt = 0
                val iter3 = payIter.iterator
                if ( iter3.hasNext ) {
                    payCnt = iter3.next()
                }

                ( clickCnt, orderCnt, payCnt )
            }
        }

        val resultRDD = analysisRDD.sortBy(_._2, false).take(10)

        // 6. 将结果采集到控制台打印出来
        resultRDD.foreach(println)

        sc.stop()
    }
}
```



### 6.1.3 实现方案二

#### 6.1.3.1 需求分析

一次性统计每个品类点击的次数，下单的次数和支付的次数：（品类，（点击总数，下单总数，支付总数））

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Spark05_Req2_HotCategoryTop10SessionAnalysis {

    def main(args: Array[String]): Unit = {

        // TODO : Top10热门品类
        val sparConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
        val sc = new SparkContext(sparConf)

        val actionRDD = sc.textFile("datas/user_visit_action.txt")
        actionRDD.cache()
        val top10Ids: Array[String] = top10Category(actionRDD)

        // 1. 过滤原始数据,保留点击和前10品类ID
        val filterActionRDD = actionRDD.filter(
            action => {
                val datas = action.split("_")
                if ( datas(6) != "-1" ) {
                    top10Ids.contains(datas(6))
                } else {
                    false
                }
            }
        )

        // 2. 根据品类ID和sessionid进行点击量的统计
        val reduceRDD: RDD[((String, String), Int)] = filterActionRDD.map(
            action => {
                val datas = action.split("_")
                ((datas(6), datas(2)), 1)
            }
        ).reduceByKey(_ + _)

        // 3. 将统计的结果进行结构的转换
        //  (（ 品类ID，sessionId ）,sum) => ( 品类ID，（sessionId, sum） )
        val mapRDD = reduceRDD.map{
            case ( (cid, sid), sum ) => {
                ( cid, (sid, sum) )
            }
        }

        // 4. 相同的品类进行分组
        val groupRDD: RDD[(String, Iterable[(String, Int)])] = mapRDD.groupByKey()

        // 5. 将分组后的数据进行点击量的排序，取前10名
        val resultRDD = groupRDD.mapValues(
            iter => {
                iter.toList.sortBy(_._2)(Ordering.Int.reverse).take(10)
            }
        )

        resultRDD.collect().foreach(println)


        sc.stop()
    }
    def top10Category(actionRDD:RDD[String]) = {
        val flatRDD: RDD[(String, (Int, Int, Int))] = actionRDD.flatMap(
            action => {
                val datas = action.split("_")
                if (datas(6) != "-1") {
                    // 点击的场合
                    List((datas(6), (1, 0, 0)))
                } else if (datas(8) != "null") {
                    // 下单的场合
                    val ids = datas(8).split(",")
                    ids.map(id => (id, (0, 1, 0)))
                } else if (datas(10) != "null") {
                    // 支付的场合
                    val ids = datas(10).split(",")
                    ids.map(id => (id, (0, 0, 1)))
                } else {
                    Nil
                }
            }
        )

        val analysisRDD = flatRDD.reduceByKey(
            (t1, t2) => {
                ( t1._1+t2._1, t1._2 + t2._2, t1._3 + t2._3 )
            }
        )

        analysisRDD.sortBy(_._2, false).take(10).map(_._1)
    }
}

```



### 6.1.4 实现方案三

#### 6.1.4.1 需求分析

使用累加器的方式聚合数据



## 6.2 需求 2：Top10 热门品类中每个品类的 Top10 活跃统计

### 6.2.1 需求说明  

在需求一的基础上，增加每个品类用户 session 的点击统计





## 6.3 需求 3：页面单跳转换率统计

### 6.3.1 需求说明

![1659754258289](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659754258289.png)

```
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object Spark06_Req3_PageflowAnalysis {

    def main(args: Array[String]): Unit = {

        // TODO : Top10热门品类
        val sparConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis")
        val sc = new SparkContext(sparConf)

        val actionRDD = sc.textFile("datas/user_visit_action.txt")

        val actionDataRDD = actionRDD.map(
            action => {
                val datas = action.split("_")
                UserVisitAction(
                    datas(0),
                    datas(1).toLong,
                    datas(2),
                    datas(3).toLong,
                    datas(4),
                    datas(5),
                    datas(6).toLong,
                    datas(7).toLong,
                    datas(8),
                    datas(9),
                    datas(10),
                    datas(11),
                    datas(12).toLong
                )
            }
        )
        actionDataRDD.cache()

        // TODO 对指定的页面连续跳转进行统计
        // 1-2,2-3,3-4,4-5,5-6,6-7
        val ids = List[Long](1,2,3,4,5,6,7)
        val okflowIds: List[(Long, Long)] = ids.zip(ids.tail)

        // TODO 计算分母
        val pageidToCountMap: Map[Long, Long] = actionDataRDD.filter(
            action => {
                ids.init.contains(action.page_id)
            }
        ).map(
            action => {
                (action.page_id, 1L)
            }
        ).reduceByKey(_ + _).collect().toMap

        // TODO 计算分子

        // 根据session进行分组
        val sessionRDD: RDD[(String, Iterable[UserVisitAction])] = actionDataRDD.groupBy(_.session_id)

        // 分组后，根据访问时间进行排序（升序）
        val mvRDD: RDD[(String, List[((Long, Long), Int)])] = sessionRDD.mapValues(
            iter => {
                val sortList: List[UserVisitAction] = iter.toList.sortBy(_.action_time)

                // 【1，2，3，4】
                // 【1，2】，【2，3】，【3，4】
                // 【1-2，2-3，3-4】
                // Sliding : 滑窗
                // 【1，2，3，4】
                // 【2，3，4】
                // zip : 拉链
                val flowIds: List[Long] = sortList.map(_.page_id)
                val pageflowIds: List[(Long, Long)] = flowIds.zip(flowIds.tail)

                // 将不合法的页面跳转进行过滤
                pageflowIds.filter(
                    t => {
                        okflowIds.contains(t)
                    }
                ).map(
                    t => {
                        (t, 1)
                    }
                )
            }
        )
        // ((1,2),1)
        val flatRDD: RDD[((Long, Long), Int)] = mvRDD.map(_._2).flatMap(list=>list)
        // ((1,2),1) => ((1,2),sum)
        val dataRDD = flatRDD.reduceByKey(_+_)

        // TODO 计算单跳转换率
        // 分子除以分母
        dataRDD.foreach{
            case ( (pageid1, pageid2), sum ) => {
                val lon: Long = pageidToCountMap.getOrElse(pageid1, 0L)

                println(s"页面${pageid1}跳转到页面${pageid2}单跳转换率为:" + ( sum.toDouble/lon ))
            }
        }
        sc.stop()
    }
    //用户访问动作表
    case class UserVisitAction(
              date: String,//用户点击行为的日期
              user_id: Long,//用户的ID
              session_id: String,//Session的ID
              page_id: Long,//某个页面的ID
              action_time: String,//动作的时间点
              search_keyword: String,//用户搜索的关键词
              click_category_id: Long,//某一个商品品类的ID
              click_product_id: Long,//某一个商品的ID
              order_category_ids: String,//一次订单中所有品类的ID集合
              order_product_ids: String,//一次订单中所有商品的ID集合
              pay_category_ids: String,//一次支付中所有品类的ID集合
              pay_product_ids: String,//一次支付中所有商品的ID集合
              city_id: Long
      )//城市 id
}
```

#  第七章：架构理解

![1659761605716](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659761605716.png)





# ********************************************************************************************************************

# 第一章:SparkSQL(153-155)

## 1.1SparkSQL 是什么



## 1.2 Hive and SparkSQL



## 1.3 SparkSQL 特点



## 1.4 DataFrame 是什么



## 1.5 DataSet 是什么



# 第2章:SparkSQL 核心编程(155-180)

## 2.1 新的起点

## 2.2 DataFrame

### 2.2.1 创建 DataFrame

### 2.2.2 SQL 语法

### 2.2.3 DSL 语法

### 2.2.4 RDD 转换为 DataFrame

### 2.2.5 DataFrame 转换为 RDD

## 2.3 DataSet

### 2.3.1 创建 DataSet

### 2.3.2 RDD 转换为 DataSet

### 2.3.3 DataSet 转换为 RDD

## 2.4 DataFrame 和 DataSet 转换

## 2.5 RDD、DataFrame、DataSet 三者的关系

### 2.5.1 三者的共性

### 2.5.2 三者的区别

### 2.5.3 三者的互相转换

## 2.6 IDEA 开发 SparkSQL

### 2.6.1 添加依赖

### 2.6.2 代码实现

## 2.7 用户自定义函数

### 2.7.1 UDF

### 2.7.2 UDAF

## 2.8 数据的加载和保存

### 2.8.1 通用的加载和保存方式

### 2.8.2 Parquet

### 2.8.3 JSON

### 2.8.4 CSV

### 2.8.5 MySQL

### 2.8.6 Hive







# 第3章:SparkSQL 项目实战(181-184)

## 3.1 数据准备

## 3.2 需求：各区域热门商品 Top3

### 3.2.1 需求简介

### 3.2.2 需求分析

### 3.2.3 功能实现







# ********************************************************************************************************************

# 第 1 章 SparkStreaming 概述(185-186)



# 第 2 章 Dstream 入门(187-788)



# 第 3 章 DStream 创建(189)



# 第 4 章 DStream 转换



# 第 5 章 DStream 输出(198)



# 第 6 章 优雅关闭(199)



# 第 7 章 SparkStreaming 案例实操(201-210)