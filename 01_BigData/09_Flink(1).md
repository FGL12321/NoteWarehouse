[TOC]

# 第1章:F link简介

## 1.1Flink是什么

![image-20221112161958512](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112161958512.png)

**Apache Flink 是一个框架和分布式处理引擎，用于对无界和有界数据流进行状态计算**

​        Flink 起源于 Stratosphere 项目，Stratosphere 是在 2010~2014 年由 3 所地处柏林的大学和欧洲的一些其他的大学共同进行的研究项目，2014 年 4 月 Stratosphere 的代码被复制并捐赠 给了 Apache 软件基 金会，参加 这个 孵化项 目的 初始 成员 是Stratosphere 系统的核心开发人员，2014 年12 月,Flin	k 一跃成为 Apache 软件基金会的顶级项目。
​        在德语中，Flink 一词表示快速和灵巧，项目采用一只松鼠的彩色图案作为 logo，这不仅是因为松鼠具有快速和灵巧的特点，还因为柏林的松鼠有一种迷人的红棕色，而 Flink 的松鼠 logo 拥有可爱的尾巴，尾巴的颜色与 Apache 软件基金会的 logo 颜色相呼应，也就是说，这是一只 Apache 风格的松鼠。

​        Flink 项目的理念是：==“Apache Flink 是为分布式、高性能、随时可用以及准确的流处理应用程序打造的开源流处理框架”。== Apache Flink 是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。Flink 被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

![image-20221112163233844](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112163233844.png)

## 1.2Flink 的重要特点

### 1.2.1 事件驱动型(Event-driven)

​		事件驱动型应用是一类具有状态的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作。比较典型的就是以 kafka 为代表的消息队列几乎都是事件驱动型应用。与之不同的就SparkStreaming 微批次，如图：

![image-20221112164716347](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112164716347.png)

事件驱动型：

![image-20221112164841765](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112164841765.png)

### 1.2.2 流与批的世界观

​		**批处理的特点是有界、持久、大量，非常适合需要访问全套记录才能完成的计算工作，一般用于离线统计。**

​		**流处理的特点是无界、实时, 无需针对整个数据集执行操作，而是对通过系统传输的每个数据项执行操作，一般用于实时统计。**		

​		在 spark 的世界观中，一切都是由批次组成的，离线数据是一个大批次，而实时数据是由一个一个无限的小批次组成的。
​		而在 flink 的世界观中，一切都是由流组成的，离线数据是有界限的流，实时数据是一个没有界限的流，这就是所谓的有界流和无界流。

​		**无界数据流：** 无界数据流有一个开始但是没有结束，它们不会在生成时终止并提供数据，必须连续处理无界流，也就是说必须在获取后立即处理 event。对于无界数据流我们无法等待所有数据都到达，因为输入是无界的，并且在任何时间点都不会完成。处理无界数据通常要求以特定顺序（例如事件发生的顺序）获取 event，以便能够推断结果完整性。

​		**有界数据流：** 有界数据流有明确定义的开始和结束，可以在执行任何计算之前通过获取所有数据来处理有界流，处理有界流不需要有序获取，因为可以始终对有界数据集进行排序，有界流的处理也称为批处理。

![image-20221112164412995](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112164412995.png)

这种以流为世界观的架构，获得的最大好处就是具有极低的延迟。

### 1.2.3 分层 api

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112165803609.png" alt="image-20221112165803609" style="zoom:50%;" />



![image-20221112165843682](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112165843682.png)

​		最底层级的抽象仅仅提供了有状态流，它将通过过程函数（Process Function）被嵌入到 DataStream API 中。底层过程函数（Process Function） 与 DataStream API 相集成，使其可以对某些特定的操作进行底层的抽象，它允许用户可以自由地处理来自一个或多个数据流的事件，并使用一致的容错的状态。除此之外，用户可以注
册事件时间并处理时间回调，从而使程序可以处理复杂的计算。

​		实际上，大多数应用并不需要上述的底层抽象，而是针对核心 API（Core APIs）进行编程，比如 DataStream API（有界或无界流数据）以及 DataSet API（有界数据集）。这些 API 为数据处理提供了通用的构建模块，比如由用户定义的多种形式的转换（transformations），连接（joins），聚合（aggregations），窗口操作（windows）等等。DataSet API 为有界数据集提供了额外的支持，例如循环与迭代。这些 API处理的数据类型以类（classes）的形式由各自的编程语言所表示。

​		Table API 是以表为中心的声明式编程，其中表可能会动态变化（在表达流数据时）。Table API 遵循（扩展的）关系模型：表有二维数据结构（schema）（类似于关系数据库中的表），同时 API 提供可比较的操作，例如 select、project、join、group-by、aggregate 等。Table API 程序声明式地定义了什么逻辑操作应该执行，而不是准确地确定这些操作代码的看上去如何。

​		尽管 Table API 可以通过多种类型的用户自定义函数（UDF）进行扩展，其仍不如核心 API 更具表达能力，但是使用起来却更加简洁（代码量更少）。除此之外，Table API 程序在执行之前会经过内置优化器进行优化。你可以在表与 DataStream/DataSet 之间无缝切换，以允许程序将 Table API 与DataStream 以及 DataSet 混合使用。

​		Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与Table API 类似，但是是以 SQL 查询表达式的形式表现程序。SQL 抽象与 Table API交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

​		目前 Flink 作为批处理还不是主流，不如 Spark 成熟，所以 DataSet 使用的并不是很多。Flink Table API 和 Flink SQL 也并不完善，大多都由各大厂商自己定制。所以我们主要学习 DataStream API 的使用。实际上 Flink 作为最接近 Google DataFlow模型的实现，是流批统一的观点，所以基本上使用 DataStream 就可以了。

Flink 几大模块

⚫ Flink Table & SQL(还没开发完)

⚫ Flink Gelly(图计算)

⚫ Flink CEP(复杂事件处理)

### 1.2.4Flink vs Spark

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221112170149937.png" alt="image-20221112170149937" style="zoom:50%;" />



### 1.2.5Flink的特性总结

⚫ 高吞吐和低延迟。每秒处理数百万个事件，毫秒级延迟。
⚫ 结果的准确性。Flink 提供了事件时间（event-time）和处理时间（processing-time）语义。对于乱序事件流，事件时间语义仍然能提供一致且准确的结果。
⚫ 精确一次（exactly-once）的状态一致性保证。
⚫ 可以连接到最常用的存储系统，如 Apache Kafka、Apache Cassandra、Elasticsearch、JDBC、Kinesis 和（分布式）文件系统，如 HDFS 和 S3。
⚫ 高可用。本身高可用的设置，加上与 K8s，YARN 和 Mesos 的紧密集成，再加上从故障中快速恢复和动态扩展任务的能力，Flink 能做到以极少的停机时间 7×24 全天候运行。
⚫ 能够更新应用程序代码并将作业（jobs）迁移到不同的 Flink 集群，而不会丢失应用程序的状态。

# 第2章：Flink快速上手

## 2.1创建项目

**1.创建Maven项目**

**2.导包**

```
<properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencies>
        <!-- 引入 Flink 相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!-- 引入日志管理相关依赖-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.14.0</version>
        </dependency>
    </dependencies>
```

3.配置日志管理

在目录 src/main/resources 下添加文件:log4j.properties，内容配置如下：

```
log4j.rootLogger=error, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```

## 2.2代码编写

### 2.2.1批处理

（1）在工程根目录下新建一个 input 文件夹，并在下面创建文本文件 words.txt

（2）在 words.txt 中输入一些文字，例如：

```
hello world
hello flink
hello java
```

（3）在 com.atguigu.chapter02 包下新建 Java 类 BatchWordCount，在静态 main 方法中编写测试代码。

&emsp;&emsp;我们进行单词频次统计的基本思路是：先逐行读入文件数据，然后将每一行文字拆分成单词；接着按照单词分组，统计每组数据的个数，就是对应单词的频次。

```java
package com.fang.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.FlatMapOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

public class BatchWordCount {
    public static void main(String[] args) throws Exception {
        //1.创建一个执行环境
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //2.从文件中读去数据
        DataSource<String> lineDataSource = env.readTextFile("input/word.txt");

        //3.将每行数据进行分词，然后装换成二元组类型
        FlatMapOperator<String, Tuple2<String, Long>> wordAndOneTuple = lineDataSource.flatMap((String line, Collector<Tuple2<String, Long>> out) -> {
            //将一行文本进行才分，
            String[] words = line.split(" ");
            //将每个单词转化成二元组输出
            for (String word : words) {
                out.collect(Tuple2.of(word, 1L));
            }
        })
                .returns(Types.TUPLE(Types.STRING, Types.LONG));

        //4.按照world进行分组
        UnsortedGrouping<Tuple2<String, Long>> wordAndOneGroup = wordAndOneTuple.groupBy(0);

        //5.分组内进行聚合统计
        AggregateOperator<Tuple2<String, Long>> sum = wordAndOneGroup.sum(1);

        //6.打印输出
        sum.print();
    }
}
```

### 2.2.2流处理

&emsp;&emsp;用 DataSet API 可以很容易地实现批处理；与之对应，流处理当然可以用DataStream API 来实现。对于 Flink 而言，流才是整个处理逻辑的底层核心，所以流批统一之后的 DataStream API 更加强大，可以直接处理批处理和流处理的所有场景。

1. 读取文件

```java
package com.fang.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

import java.util.Arrays;


public class StreamWordCount {
    public static void main(String[] args) throws Exception {

        // 1. 创建流式执行环境
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
        // 2. 读取文件
        DataStreamSource<String> lineDSS = env.readTextFile("input/word.txt");
        // 3. 转换数据格式
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = lineDSS
                .flatMap((String line, Collector<String> words) -> {
                    Arrays.stream(line.split(" ")).forEach(words::collect);
                })
                .returns(Types.STRING)
                .map(word -> Tuple2.of(word, 1L))
                .returns(Types.TUPLE(Types.STRING, Types.LONG));


        // 4. 分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne.keyBy(t -> t.f0);

        // 5. 求和
        SingleOutputStreamOperator<Tuple2<String, Long>> result = wordAndOneKS.sum(1);

        // 6. 打印
        result.print();

        // 7. 执行
        env.execute();
    }
}

```

2. 读取文本流

```java
package com.fang.wc;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

import java.util.Arrays;

public class StreamWordCount_2 {
    public static void main(String[] args) throws Exception {
        // 1. 创建流式执行环境
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
        // 2. 读取文本流
        DataStreamSource<String> lineDSS = env.socketTextStream("hadoop102", 7777);
        // 3. 转换数据格式
        SingleOutputStreamOperator<Tuple2<String, Long>> wordAndOne = lineDSS.flatMap((String line, Collector<String> words) -> {
                    Arrays.stream(line.split(" ")).forEach(words::collect);
                })
                .returns(Types.STRING)
                .map(word -> Tuple2.of(word, 1L))
                .returns(Types.TUPLE(Types.STRING, Types.LONG));
        // 4. 分组
        KeyedStream<Tuple2<String, Long>, String> wordAndOneKS = wordAndOne
                .keyBy(t -> t.f0);
        // 5. 求和
        SingleOutputStreamOperator<Tuple2<String, Long>> result = wordAndOneKS
                .sum(1);
        // 6. 打印
        result.print();
        // 7. 执行
        env.execute();
    }
}
```

运行结果：

```
3> (world,1)
2> (hello,1)
4> (flink,1)
2> (hello,2)
2> (hello,3)
1> (java,1)
```

&emsp;&emsp;Flink 是一个分布式处理引擎，所以我们的程序应该也是分布式运行的。在开发环境里，会通过多线程来模拟 Flink 集群运行。所以这里结果前的数字，其实就**指示了本地执行的不同线程**，对应着 Flink 运行时不同的并行资源。这样第一个乱序的问题也就解决了：既然是并行执行，不同线程的输出结果，自然也就无法保持输入的顺序了。

# 第 3 章 Flink 部署

![image-20221113190234763](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221113190234763.png)

**Flink 中的几个关键组件：** 客户端（Client）、作业管理器（JobManager）和任务管理器（TaskManager）

## 3.1 快速启动一个 Flink 集群

### 3.1.1 环境配置

⚫ 系统环境为 CentOS 7.5 版本

⚫ 安装 Java 8

⚫ 安装 Hadoop 集群，Hadoop 建议选择 Hadoop 2.7.5 以上版本

⚫ 配置集群节点服务器间时间同步以及免密登录，关闭防火墙

**三台服务器的具体设置如下：**

⚫ 节点服务器 1，IP 地址为 192.168.10.102，主机名为 hadoop102

⚫ 节点服务器 2，IP 地址为 192.168.10.103，主机名为 hadoop103

⚫ 节点服务器 3，IP 地址为 192.168.10.104，主机名为 hadoop104

### 3.1.2 本地启动

**1.解压**

&emsp;&emsp;在 hadoop102 节点服务器上创建安装目录/opt/module，将 flink 安装包放在该目录下，并执行解压命令，解压至当前目录。

```
$ tar -zxvf flink-1.13.0-bin-scala_2.12.tgz -C /opt/module/
```

**2.启动**

进入解压后的目录，执行启动命令，并查看进程。

```
$ cd flink-1.13.0/
$ bin/start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host hadoop102.
Starting taskexecutor daemon on host hadoop102.
$ jps
10369 StandaloneSessionClusterEntrypoint
10680 TaskManagerRunner
10717 Jps
```

**3.访问 Web UI**

启动成功后，访问 http://hadoop102:8081

**4.关闭集群**

```
$ bin/stop-cluster.sh 
Stopping taskexecutor daemon (pid: 10680) on host hadoop102.
Stopping standalonesession daemon (pid: 10369) on host hadoop102.
```

### 3.1.3 集群启动

![image-20221114112910044](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221114112910044.png)

**1.下载并解压安装包**

```
$ tar -zxvf flink-1.13.0-bin-scala_2.12.tgz -C /opt/module/
```

**2.修改集群配置**

（1）进入 conf 目录下，修改 flink-conf.yaml 文件，修改 jobmanager.rpc.address 参数为

hadoop102，如下所示：

```
$ cd conf/
$ vim flink-conf.yaml
# JobManager 节点地址.
jobmanager.rpc.address: hadoop102
```

这就指定了 hadoop102 节点服务器为 JobManager 节点。

（2）修改 workers 文件，将另外两台节点服务器添加为本 Flink 集群的 TaskManager 节点，

具体修改如下：

```
$ vim workers 
hadoop103
hadoop104
```

（3）另外，在 flink-conf.yaml 文件中还可以对集群中的 JobManager 和 TaskManager 组件进行优化配置，主要配置项如下：

jobmanager.memory.process.size：对 JobManager 进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整。

⚫ taskmanager.memory.process.size：对 TaskManager 进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整。

⚫ taskmanager.numberOfTaskSlots：对每个 TaskManager 能够分配的 Slot 数量进行配置，默认为 1，可根据 TaskManager 所在的机器能够提供给 Flink 的 CPU 数量决定。所谓Slot 就是 TaskManager 中具体运行一个任务所分配的计算资源。

⚫ parallelism.default：Flink 任务执行的默认并行度，优先级低于代码中进行的并行度配置和任务提交时使用参数指定的并行度数量。

**3.分发安装目录**

配置修改完毕后，将 Flink 安装目录发给另外两个节点服务器。

```
$ scp -r ./flink-1.13.0 atguigu@hadoop103:/opt/module
$ scp -r ./flink-1.13.0 atguigu@hadoop104:/opt/module
```

**4.启动集群**

（1）在 hadoop102 节点服务器上执行 start-cluster.sh 启动 Flink 集群：

```
$ bin/start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host hadoop102.
Starting taskexecutor daemon on host hadoop103.
Starting taskexecutor daemon on host hadoop104.
```

（2）查看进程情况：

```
[atguigu@hadoop102 flink-1.13.0]$ jps
13859 Jps
13782 StandaloneSessionClusterEntrypoint
[atguigu@hadoop103 flink-1.13.0]$ jps
12215 Jps
12124 TaskManagerRunner
[atguigu@hadoop104 flink-1.13.0]$ jps
11602 TaskManagerRunner
11694 Jps
```

**5.访问 Web UI**

启动成功后，访问 http://hadoop102:8081

### 3.1.4 向集群提交作业

1.程序打包

2.提交作业

## 3.2 部署模式

### 3.2.1 会话模式（Session Mode）

![image-20221113205903575](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221113205903575.png)

​		**集群启动时所有资源就都已经确定，所以所有提交的作业会竞争集群中的资源**

​		这样的好处很明显，我们只需要一个集群，就像一个大箱子，所有的作业提交之后都塞进去；集群的生命周期是超越于作业之上的，铁打的营盘流水的兵，作业结束了就释放资源，集群依然正常运行。

​		当然缺点也是显而易见的：因为资源是共享的，所以资源不够了，提交新的作业就会失败。另外，同一个TaskManager 上可能运行了很多作业，如果其中一个发生故障导致 TaskManager 宕机，那么所有作业都会受到影响。

**会话模式比较适合于单个规模小、执行时间短的大量作业。**

### 3.2.2 单作业模式（Per-Job Mode）

![image-20221114113643491](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221114113643491.png)



​		单作业模式也很好理解，就是严格的一对一，集群只为这个作业而生。同样由客户端运行应用程序，然后启动集群，作业被提交给 JobManager，进而分发给 TaskManager 执行。作业作业完成后，集群就会关闭，所有资源也会释放。

​		这样一来，每个作业都有它自己的 JobManager管理，占用独享的资源，即使发生故障，它的TaskManager 宕机也不会影响其他作业。这些特性使得单作业模式在生产环境运行更加稳定，所以是实际应用的首选模式。需要注意的是，Flink 本身无法直接这样运行，所以单作业模式一般需要借助一些资源管理框架来启动集群，比如 YARN、Kubernetes。

### 3.2.3 应用模式（Application Mode）

![image-20221114113651460](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221114113651460.png)

​		要为每一个提交的应用单独启动一个 JobManager，也就是创建一个集群。这个 JobManager 只为执行这一个应用而存在，执行结束之后 JobManager 也就关闭了，这就是所谓的应用模式，

## 3.3 独立模式（Standalone）

​		独立模式（Standalone）是部署 Flink 最基本也是最简单的方式：所需要的所有 Flink 组件，都只是操作系统上运行的一个 JVM 进程。

​		独立模式是独立运行的，不依赖任何外部的资源管理平台；当然独立也是有代价的：如果资源不足，或者出现故障，没有自动扩展或重分配资源的保证，必须手动处理。所以独立模式一般只用在开发测试或作业非常少的场景下。

​		另外，我们也可以将独立模式的集群放在容器中运行。Flink 提供了独立模式的容器化部署方式，可以在 Docker 或者 Kubernetes 上进行部署。

### 3.3.1 会话模式部署

​		可以发现，独立模式的特点是不依赖外部资源管理平台，而会话模式的特点是先启动集群、后提交作业。所以，我们在第 3.1 节用的就是独立模式（Standalone）的会话模式部署

### 3.3.2 单作业模式部署

​		Flink 本身无法直接以单作业方式启动集群，一般需要借助一些资源管理平台。所以 Flink 的独立（Standalone）集群并不支持单作业模式部署。

### 3.3.3 应用模式部署

应用模式下不会提前创建集群，所以不能调用 start-cluster.sh 脚本。我们可以使用同样在

bin 目录下的 standalone-job.sh 来创建一个 JobManager。

具体步骤如下：

（1）进入到 Flink 的安装路径下，将应用程序的 jar 包放到 lib/目录下。

```
$ cp ./FlinkTutorial-1.0-SNAPSHOT.jar lib/
```

（2）执行以下命令，启动 JobManager。

```
$ ./bin/standalone-job.sh start --job-classname com.atguigu.wc.StreamWordCount
```

这里我们直接指定作业入口类，脚本会到 lib 目录扫描所有的 jar 包。

（3）同样是使用 bin 目录下的脚本，启动 TaskManager。

```
$ ./bin/taskmanager.sh start
```

（4）如果希望停掉集群，同样可以使用脚本，命令如下。

```
$ ./bin/standalone-job.sh stop
$ ./bin/taskmanager.sh stop
```

## 3.4 YARN 模式

**（1）按照 3.1 节所述，下载并解压安装包，并将解压后的安装包重命名为 flink-1.13.0-yarn，本节的相关操作都将默认在此安装路径下执行。**

**（2）配置环境变量，增加环境变量配置如下：**

```
$ sudo vim /etc/profile.d/my_env.sh
HADOOP_HOME=/opt/module/hadoop-2.7.5
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
42export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_CLASSPATH=`hadoop classpath`
```

这里必须保证设置了环境变量 HADOOP_CLASSPATH。

**（3）启动 Hadoop 集群，包括 HDFS 和 YARN。**

```
[atguigu@hadoop102 ~]$ start-dfs.sh
[atguigu@hadoop103 ~]$ start-yarn.sh
```

分别在 3 台节点服务器查看进程启动情况。

```
[atguigu@hadoop102 ~]$ jps
5190 Jps
5062 NodeManager
4408 NameNode
4589 DataNode
[atguigu@hadoop103 ~]$ jps
5425 Jps
4680 ResourceManager
5241 NodeManager
4447 DataNode
[atguigu@hadoop104 ~]$ jps
4731 NodeManager
4333 DataNode
4861 Jps
4478 SecondaryNameNode
```

**（4）进入 conf 目录，修改 flink-conf.yaml 文件，修改以下配置，这些配置项的含义在进行 Standalone 模式配置的时候进行过讲解，若在提交命令中不特定指明，这些配置将作为默认配置。**

```
$ cd /opt/module/flink-1.13.0-yarn/conf/
$ vim flink-conf.yaml
jobmanager.memory.process.size: 1600m
taskmanager.memory.process.size: 1728m
taskmanager.numberOfTaskSlots: 8
parallelism.default: 1
```

## 3.5作业提交

### 3.5.1界面提交

![image-20221114134333031](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221114134333031.png)

### 3.5.2命令行提交

```
./bin/flink run -m hadoop102:8081 -c com.fang.wc.StreamWordCount_2 -p 2  ./Flink_project-1.0-SNAPSHOT.jar 
```

# 第 4 章 Flink 运行时架构

## 4.1 系统架构

**三大部分  四个组件 四张图**

![image-20221115160643884](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115160643884.png)

### 4.1.1 整体构成

![image-20221114134455380](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221114134455380.png)

&emsp;&emsp;这里首先要说明一下“客户端”。其实客户端并不是处理系统的一部分，==它只负责作业的提交。具体来说，就是调用程序的 main 方法，将代码转换成“数据流图”（Dataflow Graph），并最终生成作业图（JobGraph），一并发送给 JobManager。== 提交之后，任务的执行其实就跟客户端没有关系了；我们可以在客户端选择断开与 JobManager 的连接, 也可以继续保持连接。之前我们在命令提交作业时，加上的-d 参数，就是表示分离模式（detached mode)，也就是断开连接。

&emsp;&emsp;当然，客户端可以随时连接到 JobManager，获取当前作业的状态和执行结果，也可以发送请求取消作业。我们在上一章中不论通过 Web UI 还是命令行执行“flink run”的相关操作，都是通过客户端实现的。

&emsp;&emsp;Flink 的运行时架构中，==最重要的就是两大组件：作业管理器（JobManger）和任务管理器（TaskManager）。== 对于一个提交执行的作业，JobManager 是真正意义上的“管理者”（Master），负责管理调度，所以在不考虑高可用的情况下只能有一个；而 TaskManager 是“工作者”（Worker、Slave），负责执行任务处理数据，所以可以有一个或多个。

### 4.1.2 作业管理器（JobManager）

**1.JobMaster**

​		JobMaster 是 JobManager 中最核心的组件，负责处理单独的作业（Job）。==所以 JobMaster和具体的 Job 是一一对应的==，多个 Job 可以同时运行在一个 Flink 集群中, 每个 Job 都有一个自己的 JobMaster。需要注意在早期版本的 Flink 中，没有 JobMaster 的概念；而 JobManager的概念范围较小，实际指的就是现在所说的 JobMaster。

​		在作业提交时，JobMaster 会先接收到要执行的应用。这里所说“应用”一般是客户端提交来的，包括：Jar 包，数据流图（dataflow graph），和作业图（JobGraph）。JobMaster 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫作“执行图”（ExecutionGraph），它包含了所有可以并发执行的任务。

​		 JobMaster 会向资源管理器（ResourceManager）发出请求，申请执行任务必要的资源。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的 TaskManager 上。

​		而在运行过程中，JobMaster 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调。

**2.资源管理器（ResourceManager）**

​		==ResourceManager 主要负责资源的分配和管理==，在 Flink 集群中只有一个。所谓“资源”，主要是指 TaskManager 的任务槽（task slots）。任务槽就是 Flink 集群中的资源调配单元，包含了机器用来执行计算的一组 CPU 和内存资源。每一个任务（Task）都需要分配到一个 slot 上执行。

​		这里注意要把 Flink 内置的 ResourceManager 和其他资源管理平台（比如 YARN）的ResourceManager 区分开。

​		Flink 的 ResourceManager，针对不同的环境和资源管理平台（比如 Standalone 部署，或者YARN），有不同的具体实现。在 Standalone 部署时，因为 TaskManager 是单独启动的（没有Per-Job 模式），所以 ResourceManager 只能分发可用 TaskManager 的任务槽，不能单独启动新TaskManager。

​		而在有资源管理平台时，就不受此限制。当新的作业申请资源时，ResourceManager 会将有空闲槽位的 TaskManager 分配给 JobMaster。如果 ResourceManager 没有足够的任务槽，它还可以向资源提供平台发起会话，请求提供启动 TaskManager 进程的容器。另外，ResourceManager 还负责停掉空闲的 TaskManager，释放计算资源。

**3.分发器（Dispatcher）**

​		==Dispatcher 主要负责提供一个 REST 接口，用来提交应用，并且负责为每一个新提交的作业启动一个新的 JobMaster 组件==。Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作业执行的信息。Dispatcher 在架构中并不是必需的，在不同的部署模式下可能会被忽略掉。

### 4.1.3 任务管理器（TaskManager）

​		TaskManager 是 Flink 中的工作进程，数据流的具体计算就是它来做的，所以也被称为“Worker”。Flink 集群中必须至少有一个 TaskManager；当然由于分布式计算的考虑，通常会有多个 TaskManager 运行，==**每一个 TaskManager 都包含了一定数量的任务槽（task slots）。**== Slot是资源调度的最小单位，slot 的数量限制了 TaskManager 能够并行处理的任务数量。启动之后，TaskManager 会向资源管理器注册它的 slots；收到资源管理器的指令后，TaskManager 就会将一个或者多个槽位提供给 JobMaster 调用，JobMaster 就可以分配任务来执行了。在执行过程中，TaskManager 可以缓冲数据，还可以跟其他运行同一应用的 TaskManager交换数据。

## 4.2 作业提交流程

### 4.2.1 高层级抽象视角

![image-20221115133903731](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115133903731.png)



（1） 一般情况下，由客户端（App）通过分发器提供的 REST 接口，将作业提交给JobManager。

（2）由分发器启动 JobMaster，并将作业（包含 JobGraph）提交给 JobMaster。

（3）JobMaster 将 JobGraph 解析为可执行的 ExecutionGraph，得到所需的资源数量，然后向资源管理器请求资源（slots）。

（4）资源管理器判断当前是否由足够的可用资源；如果没有，启动新的 TaskManager。

（5）TaskManager 启动之后，向 ResourceManager 注册自己的可用任务槽（slots）。

（6）资源管理器通知 TaskManager 为新的作业提供 slots。

（7）TaskManager 连接到对应的 JobMaster，提供 slots。

（8）JobMaster 将需要执行的任务分发给 TaskManager。

（9）TaskManager 执行任务，互相之间可以交换数据。

​		如果部署模式不同，或者集群环境不同（例如 Standalone、YARN、K8S 等），其中一些步骤可能会不同或被省略，也可能有些组件会运行在同一个 JVM 进程中。比如我们在上一章实践过的独立集群环境的会话模式，就是需要先启动集群，如果资源不够，只能等待资源释放，而不会直接启动新的 TaskManager。接下来我们就具体介绍一下不同部署环境下的提交流程。

### 4.2.2 独立模式（Standalone）

![image-20221115134123876](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115134123876.png)

（1）客户端通过 REST 接口，将作业提交给分发器。
（2）分发器启动 JobMaster，并将作业（包含 JobGraph）提交给 JobMaster。
（3）JobMaster 向资源管理器请求资源（slots）。
（4）资源管理器向 YARN 的资源管理器请求 container 资源。
（5）YARN 启动新的 TaskManager 容器。
（6）TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。
（7）资源管理器通知 TaskManager 为新的作业提供 slots。
（8）TaskManager 连接到对应的 JobMaster，提供 slots。
（9）JobMaster 将需要执行的任务分发给 TaskManager，执行任务。
		可见，整个流程除了请求资源时要“上报”YARN 的资源管理器，其他与 4.2.1 节所述抽象流程几乎完全一样。

### 4.2.3 YARN 集群

**会话模式：**

![image-20221115134246908](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115134246908.png)

**单作业模式：**

![image-20221115134135670](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115134135670.png)

（1）客户端将作业提交给 YARN 的资源管理器，这一步中会同时将 Flink 的 Jar 包和配置上传到 HDFS，以便后续启动 Flink 相关组件的容器。

（2）YARN 的资源管理器分配 Container 资源，启动 Flink JobManager，并将作业提交给JobMaster。这里省略了 Dispatcher 组件。

（3）JobMaster 向资源管理器请求资源（slots）。

（4）资源管理器向 YARN 的资源管理器请求 container 资源。

（5）YARN 启动新的 TaskManager 容器。

（6）TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。

（7）资源管理器通知 TaskManager 为新的作业提供 slots。

（8）TaskManager 连接到对应的 JobMaster，提供 slots。

（9）JobMaster 将需要执行的任务分发给 TaskManager，执行任务。

​		可见，区别只在于 JobManager 的启动方式，以及省去了分发器。当第 2 步作业提交给JobMaster，之后的流程就与会话模式完全一样了。

## 4.3 一些重要概念

### 4.3.1 数据流图（Dataflow Graph）

所有的 Flink 程序都可以归纳为由三部分构成：Source、Transformation 和 Sink。

⚫ Source 表示“源算子”，负责读取数据源。

⚫ Transformation 表示“转换算子”，利用各种算子进行处理加工。

⚫ Sink 表示“下沉算子”，负责数据的输出。

![image-20221115142636901](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115142636901.png)

### 4.3.2 并行度（Parallelism）

![image-20221115140909206](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115140909206.png)

​		当前数据流中有 source、map、window、sink 四个算子，除最后 sink，其他算子的并行度都为 2。整个程序包含了 7 个子任务，至少需要 2 个分区来并行执行。我们可以说，这段流处理程序的并行度就是 2

### 4.3.3 算子链（Operator Chain）

&emsp;&emsp;在 Flink 中，==并行度相同的一对一（one to one）算子操作，可以直接链接在一起形成一个“大”的任务（task）==，这样原来的算子就成为了真正任务里的一部分，如图 4-11 所示。每个 task会被一个线程执行。==这样的技术被称为“算子链”（Operator Chain）。==

![image-20221115142036619](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115142036619.png)

合并算子链

![image-20221115142128151](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115142128151.png)

​		Source 和 map 之间满足了算子链的要求，所以可以直接合并在一起，形成了一个任务；因为并行度为 2，所以合并后的任务也有两个并行子任务。这样，这个数据流图所表示的作业最终会有 5 个任务，由 5 个线程并行执行。

**Flink 为什么要有算子链这样一个设计呢？**

​		这是因为将算子链接成 task 是非常有效的优化：==可以减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。==



### 4.3.4 作业图（JobGraph）与执行图（ExecutionGraph）

![image-20221115161020657](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115161020657.png)

![image-20221115161033765](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115161033765.png)

**1.逻辑流图（StreamGraph）**

​		这是根据用户通过 DataStream API 编写的代码生成的最初的 DAG 图，用来表示程序的拓扑结构。这一步一般在客户端完成。

**我们可以看到，逻辑流图中的节点，完全对应着代码中的四步算子操作：**

源算子 Source（socketTextStream()）→扁平映射算子 Flat Map(flatMap()) →分组聚合算子

Keyed Aggregation(keyBy/sum()) →输出算子 Sink(print())。

**2.作业图（JobGraph）**

​		StreamGraph 经过优化后生成的就是作业图（JobGraph），这是提交给 JobManager 的数据结构，确定了当前作业中所有任务的划分。主要的优化为:==将多个符合条件的节点链接在一起合并成一个任务节点，形成算子链==，这样可以减少数据交换的消耗。JobGraph 一般也是在客户端生成的，在作业提交时传递给 JobMaster。

​		分组聚合算子（Keyed Aggregation）和输出算子 Sink(print)并行度都为 2，而且是一对一的关系，满足算子链的要求，所以会合并在一起，成为一个任务节点。

**3.执行图（ExecutionGraph）**

​		JobMaster 收到 JobGraph 后，会根据它来生成执行图（ExecutionGraph）。ExecutionGraph是 JobGraph 的并行化版本，是调度层最核心的数据结构。

​		从图 4-12 中可以看到，与 JobGraph 最大的区别就是按照并行度对并行子任务进行了拆分，并明确了任务间数据传输的方式。

**4.物理图（Physical Graph）**

​		JobMaster 生成执行图后， 会将它分发给 TaskManager；各个 TaskManager 会根据执行图部署任务，最终的物理执行过程也会形成一张“图”，一般就叫作物理图（Physical Graph）。这只是具体执行层面的图，并不是一个具体的数据结构。

​		物理图主要就是在执行图的基础上，进一步确定数据存放的位置和收发的具体方式。有了物理图，TaskManager 就可以对传递来的数据进行处理计算了。

​		所以我们可以看到，程序里定义了四个算子操作：源（Source）->转换（flatMap）->分组聚合（keyBy/sum）->输出（print）；合并算子链进行优化之后，就只有三个任务节点了；再考虑并行度后，一共有 5 个并行子任务，最终需要 5 个线程来执行。

### 4.3.5 任务（Tasks）和任务槽（Task Slots）

**1.任务槽**

​		之前已经提到过，Flink 中每一个 worker(也就是 TaskManager)都是一个 JVM 进程，它可以启动多个独立的线程，来并行执行多个子任务（subtask）。

​		所以如果想要执行 5 个任务，并不一定非要 5 个 TaskManager，我们可以让 TaskManager多线程执行任务。如果可以同时运行 5 个线程，那么只要一个 TaskManager 就可以满足我们之前程序的运行需求了。

​		很显然，TaskManager 的计算资源是有限的，并不是所有任务都可以放在一个 TaskManager上并行执行。并行的任务越多，每个线程的资源就会越少。那一个 TaskManager 到底能并行处理多少个任务呢？为了控制并发量，我们需要在 TaskManager 上对每个任务运行所占用的资源做出明确的划分，这就是所谓的任务槽（task slots）。



![image-20221115142553751](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115142553751.png)

**2.任务对任务槽的共享**

​		我们可以基于之前的例子继续扩展。如果我们保持 sink 任务并行度为 1 不变，而作业提交时设置全局并行度为 6，那么前两个任务节点就会各自有 6 个并行子任务，整个流处理程序则有 13 个子任务。那对于 2 个 TaskManager、每个有 3 个 slot 的集群配置来说，还能否正常运行呢？

![image-20221115162158021](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115162158021.png)

**3.任务槽和并行度的关系**

​		**Slot 和并行度确实都跟程序的并行执行有关**，但两者是完全不同的概念。简单来说，==task slot 是 静 态 的 概 念 ， 是 指 TaskManager 具 有 的 并 发 执 行 能 力==， 可 以 通 过 参 数taskmanager.numberOfTaskSlots 进行配置；而==并行度（parallelism）是动态概念，也就是TaskManager 运行程序时实际使用的并发能力==，可以通过参数 parallelism.default 进行配置。换句话说，并行度如果小于等于集群中可用 slot 的总数，程序是可以正常执行的，因为 slot 不一定要全部占用，有十分力气可以只用八分；**而如果并行度大于可用 slot 总数，导致超出了并行能力上限，那么心有余力不足，程序就只好等待资源管理器分配更多的资源了。**

![image-20221115162521689](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115162521689.png)

![image-20221115162536373](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115162536373.png)

![image-20221115162548638](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221115162548638.png)

# 第 5 章 DataStream API（基础篇）

⚫ 获取执行环境（execution environment）

⚫ 读取数据源（source）

⚫ 定义基于数据的转换操作（transformations）

⚫ 定义计算结果的输出位置（sink）

⚫ 触发程序执行（execute）

![image-20221116103659422](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221116103659422.png)

## 5.1 执行环境（Execution Environment）

### 5.1.1 创建执行环境

**1. getExecutionEnvironment**

直接根据当前运行上下文得到正确的结果，也就是本地的结果，没有用到远程的集群资源。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

**2. createLocalEnvironment**

和上面一样同样是本地的执行环境，同时可以设置并行度

```
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

**3. createRemoteEnvironment**

这个方法返回集群执行环境。需要在调用时指定 JobManager 的主机名和端口号，并指定要在集群中运行的 Jar 包。

```java
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment
 .createRemoteEnvironment(
 "host", // JobManager 主机名
 1234, // JobManager 进程端口号
 "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
);
```

### 5.1.2 执行模式(Execution Mode)

批处理和流处理执行环境的使用

```java
// 批处理环境
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment();
// 流处理环境
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
```

&emsp;&emsp;基于 ExecutionEnvironment 读入数据创建的数据集合，就是 DataSet；对应的调用的一整套转换方法，就是 DataSet API。

⚫ **流执行模式（STREAMING）**

​		这是 DataStream API 最经典的模式，一般用于需要持续实时处理的无界数据流。默认情况下，程序使用的就是 STREAMING 执行模式。

⚫ **批执行模式（BATCH）**

&emsp;&emsp;专门用于批处理的执行模式, 这种模式下，Flink 处理作业的方式类似于 MapReduce 框架。对于不会持续计算的有界数据，我们用这种模式处理会更方便。

⚫ **自动模式（AUTOMATIC）**

  在这种模式下，将由程序根据输入数据源是否有界，来自动选择执行模式。

**1. BATCH 模式的配置方法**

要有两种方式：

（1）通过命令行配置

```
bin/flink run -Dexecution.runtime-mode=BATCH ...
```

在提交作业时，增加 execution.runtime-mode 参数，指定值为 BATCH。

（2）通过代码配置

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```

在代码中，直接基于执行环境调用 setRuntimeMode 方法，传入 BATCH 模式。

建议: 不要在代码中配置，而是使用命令行。这同设置并行度是类似的：在提交作业时指定参数可以更加灵活，同一段应用程序写好之后，既可以用于批处理也可以用于流处理。而在代码中硬编码（hard code）的方式可扩展性比较差，一般都不推荐。

**2. 什么时候选择 BATCH 模式**

  用 BATCH 模式处理批量数据，用 STREAMING模式处理流式数据。因为数据有界的时候，直接输出结果会更加高效；而当数据无界的时候, 我们没得选择——只有 STREAMING 模式才能处理持续的数据流

### 5.1.3 触发程序执行

  有了执行环境，我们就可以构建程序的处理流程了：基于环境读取数据源，进而进行各种转换操作，最后输出结果到外部系统。

  需要注意的是，写完输出（sink）操作并不代表程序已经结束。因为当 main()方法被调用时，其实只是定义了作业的每个执行操作，然后添加到数据流图中；这时并没有真正处理数据——因为数据可能还没来。Flink 是由事件驱动的，只有等到数据到来，才会触发真正的计算，

  这也被称为“延迟执行”或“懒执行”（lazy execution）。所以我们需要显式地调用执行环境的 execute()方法，来触发程序执行。execute()方法将一直等待作业完成，然后返回一个执行结果（JobExecutionResult）。

## 5.2 源算子（Source）

![image-20221116163119071](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221116163119071.png)

### 5.2.1 准备工作

  为了更好地理解，我们先构建一个实际应用场景。比如网站的访问操作，可以抽象成一个三元组（用户名，用户访问的 urrl，用户访问 url 的时间戳），所以在这里，我们可以创建一个类 Event，将用户行为包装成它的一个对象。Event 包含了以下一些字段

```java
package com.fang.chapter05;

import java.sql.Timestamp;

public class Event {

    public String user;
    public String url;
    public Long timestamp;

    public Event() {

    }
    public Event(String user, String url, Long timestamp) {
        this.user = user;
        this.url = url;
        this.timestamp = timestamp;
    }
    @Override
    public String toString() {
        return "Event{" +
                "user='" + user + '\'' +
                ", url='" + url + '\'' +
                ", timestamp=" + new Timestamp(timestamp) +
                '}';
    }

}
```

这里需要注意，我们定义的 Event，有这样几个特点：

⚫ 类是公有（public）的74

⚫ 有一个无参的构造方法

⚫ 所有属性都是公有（public）的

⚫ 所有属性的类型都是可以序列化的

### 5.2.2 从集合中读取数据

  最简单的读取数据的方式，就是在代码中直接创建一个 Java 集合，然后调用执行环境的fromCollection 方法进行读取。这相当于将数据临时存储到内存中，形成特殊的数据结构后，作为数据源使用，一般用于测试。

### 5.2.3 从文件读取数据

  真正的实际应用中，自然不会直接将数据写在代码中。通常情况下，我们会从存储介质中获取数据，个比较常见的方式就是读取日志文件。这也是批处理中最常见的读取方式。

### 5.2.4 从 Socket 读取数据

  不论从集合还是文件，我们读取的其实都是有界数据。在流处理的场景中，数据往往是无界的。这时又从哪里读取呢？一个简单的方式，就是我们之前用到的读取 socket 文本流。这种方式由于吞吐量小、稳定性较差，一般也是用于测试

### 5.2.5 从 Kafka 读取数据

  Kafka 作为分布式消息传输队列，是一个高吞吐、易于扩展的消息系统。而消息队列的传输方式，恰恰和流处理是完全一致的。所以可以说 Kafka 和 Flink 天生一对，是当前处理流式数据的双子星。在如今的实时流处理应用中，由 Kafka 进行数据的收集和传输，Flink 进行分析计算，这样的架构已经成为众多企业的首选。

**引入kafka依赖：**

```java
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
```

**代码实现：**

```java
package com.fang.chapter05;

import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.kafka.common.protocol.types.Field;

import java.util.ArrayList;
import java.util.Properties;

public class SourceTest {
    public static void main(String[] args) throws Exception {
        //1.创建新的执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //1.从文件中读取数据
        DataStreamSource<String> stream1 = env.readTextFile("input/clicks.csv");

        //2.从集合中读取数据
        ArrayList<Integer> nums =new ArrayList();
        nums.add(2);
        nums.add(5);
        DataStreamSource<Integer> numStream= env.fromCollection(nums);

        ArrayList<Event> events = new ArrayList<>();
        events.add(new Event("Marry","./home",1000L));
        events.add(new Event("Bob","./home",1000L));
        DataStreamSource<Event> stream2 = env.fromCollection(events);

        //3.从元素读取数据
        DataStreamSource<Event> stream3 = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./home", 1000L)
        );

        //4.从socket文本流中读取
        DataStreamSource<String> streamSource = env.socketTextStream("hadoop102", 7777);


//        stream1.print("1");
//        numStream.print("nums");
//        stream2.print("2");
//        stream3.print("3");

//        streamSource.print("4");

        //5.从kafka中读取数据
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "hadoop102:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");



        DataStreamSource<String> kafkaStream= env.addSource(new FlinkKafkaConsumer<String>(
                "clicks",
                new SimpleStringSchema(),
                properties));
        kafkaStream.print();
        env.execute();

    }
}
```

### 5.2.6 自定义 Source

大多数情况下，前面的数据源已经能够满足需要。但是凡事总有例外，如果遇到特殊情况，我们想要读取的数据源来自某个外部系统，而 flink 既没有预实现的方法、也没有提供连接器，又该怎么办呢？

那就只好自定义实现 SourceFunction 了。

接下来我们创建一个自定义的数据源，实现 SourceFunction 接口。主要重写两个关键方法：

run()和 cancel()。

⚫ run()方法：使用运行时上下文对象（SourceContext）向下游发送数据；

⚫ cancel()方法：通过标识位控制退出循环，来达到中断数据源的效果。

代码如下：

我们先来自定义一下数据源：

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Calendar;
import java.util.Random;

public class ClinkSource implements SourceFunction<Event> {

    //声明一个标志位
    private Boolean running=true;

    @Override
    public void run(SourceContext<Event> ctx) throws Exception {

        //随机生成数据
        Random random = new Random();
        //自定义生成数组
        String[] users = {"Mary", "Alice", "Bob", "Cary"};
        String[] urls = {"./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2"};


        //循环生成数据
        while (running){
            String user=users[random.nextInt(users.length)];
            String url=urls[random.nextInt(urls.length)];
            Long timestamp=Calendar.getInstance().getTimeInMillis();
            ctx.collect(new Event(user,url,timestamp));
        }
    }

    @Override
    public void cancel() {
        running=false;

    }
}
```

实现ParallelSourceFunction从而可以设置并行度

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;

import java.util.Random;

public class SourceCustomTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(16);

        //DataStreamSource<Event> customStream=env.addSource(new ClinkSource());
        DataStreamSource<Integer> objectDataStreamSource = env.addSource(new ParallelCustomSource());

        //customStream.print();
        objectDataStreamSource.print();

        env.execute();
    }

    public static class ParallelCustomSource implements ParallelSourceFunction<Integer>{

        private Boolean running=true;
        public Random random=new Random();

        @Override
        public void run(SourceContext<Integer> sourceContext) throws Exception {
            while (running)
                sourceContext.collect(random.nextInt());

        }

        @Override
        public void cancel() {
            running=false;
        }
    }
}
```

### 5.2.7 Flink 支持的数据类型

**1. Flink** 支持的数据类型**

（1）基本类型

所有 Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。81

（2）数组类型

包括基本类型数组（PRIMITIVE_ARRAY）和对象数组(OBJECT_ARRAY)

（3）复合数据类型

⚫ Java 元组类型（TUPLE）：这是 Flink 内置的元组类型，是 Java API 的一部分。最多25 个字段，也就是从 Tuple0~Tuple25，不支持空字段

⚫ Scala 样例类及 Scala 元组：不支持空字段

⚫ 行类型（ROW）：可以认为是具有任意个字段的元组,并支持空字段

⚫ POJO：Flink 自定义的类似于 Java bean 模式的类

（4）辅助类型

Option、Either、List、Map 等

（5）泛型类型（GENERIC）

  Flink 支持所有的 Java 类和 Scala 类。不过如果没有按照上面 POJO 类型的要求来定义，就会被 Flink 当作泛型类来处理。Flink 会把泛型类型当作黑盒，无法获取它们内部的属性；它们也不是由 Flink 本身序列化的，而是由 Kryo 序列化的。

  在这些类型中，元组类型和 POJO 类型最为灵活，因为它们支持创建复杂类型。而相比之下，POJO 还支持在键（key）的定义中直接使用字段名，这会让我们的代码可读性大大增加。所以，在项目实践中，往往会将流处理程序中的元素类型定为 Flink 的 POJO 类型。

Flink 对 POJO 类型的要求如下：

⚫ 类是公共的（public）和独立的（standalone，也就是说没有非静态的内部类）；

⚫ 类有一个公共的无参构造方法；

⚫ 类中的所有字段是 public 且非 final 的；或者有一个公共的 getter 和 setter 方法，这些方法需要符合 Java bean 的命名规范。所以我们看到，之前的 UserBehavior，就是我们创建的符合 Flink POJO 定义的数据类型。

**2. 类型提示（Type Hints）**

  Flink 还具有一个类型提取系统，可以分析函数的输入和返回类型，自动获取类型信息，从而获得对应的序列化器和反序列化器。但是，由于 Java 中泛型擦除的存在，在某些特殊情况下（比如 Lambda 表达式中），自动提取的信息是不够精细的——只告诉 Flink 当前的元素由“船头、船身、船尾”构成，根本无法重建出“大船”的模样；这时就需要显式地提供类型信息，才能使应用程序正常工作或提高其性能。

  为了解决这类问题，Java API 提供了专门的“类型提示”（type hints）。

  回忆一下之前的 word count 流处理程序，我们在将 String 类型的每个词转换成（word，count）二元组后，就明确地用 returns 指定了返回的类型。因为对于 map 里传入的 Lambda 表达式，系统只能推断出返回的是 Tuple2 类型，而无法得到 Tuple2<String, Long>。只有显式地告诉系统当前的返回类型，才能正确地解析出完整数据。

```
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));
```

  这是一种比较简单的场景，二元组的两个元素都是基本数据类型。那如果元组中的一个元素又有泛型，该怎么处理呢？

  Flink 专门提供了 TypeHint 类，它可以捕获泛型的类型信息，并且一直记录下来，为运行时提供足够的信息。我们同样可以通过.returns()方法，明确地指定转换之后的 DataStream 里元素的类型。

```
returns(new TypeHint<Tuple2<Integer, SomeType>>(){})
```

## 5.3 转换算子（Transformation）

![image-20221116164847350](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221116164847350.png)

### 5.3.1 基本转换算子

**1.映射（map）**

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransformMapTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //从元素中读取数据
        //3.从元素读取数据
        DataStreamSource<Event> stream3 = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./home", 1000L));
        //1.使用自定义类，实现mapFunction接口
        SingleOutputStreamOperator<String> map = stream3.map(new MyMapper());

        //2.使用匿名了实现MapFunction接口
        SingleOutputStreamOperator<String> map1 = stream3.map(new MapFunction<Event, String>() {
            public String map(Event value) throws Exception{
                return value.user;
            }
        });
        
        //3.传入lambda表达式
        SingleOutputStreamOperator<String> map2 = stream3.map(data -> data.user);


        map.print();
        map1.print();
        map2.print();
        env.execute();
    }
    public static class MyMapper implements MapFunction<Event,String>{

        @Override
        public String map(Event event) throws Exception {
            return event.user;
        }
    }
}
```

![image-20221215125247502](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221215125247502.png)

**2. 过滤（filter）**

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransformFilterTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //从元素中读取数据
        //3.从元素读取数据
        DataStreamSource<Event> stream3 = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./home", 1000L));


        //1.传入一个实现了FilterFunction的类的对象
        SingleOutputStreamOperator<Event> result1 = stream3.filter(new MyFilterTest());

        //2.入一个匿名类
        SingleOutputStreamOperator<Event> mary = stream3.filter(new FilterFunction<Event>() {
            @Override
            public boolean filter(Event e) throws Exception {
                return e.user.equals("Marry");
            }
        });

        //3.传入Lambda表达式
        stream3.filter(data->data.user.equals("Marry")).print();

        result1.print();
        mary.print();
        env.execute();
    }
    public static class MyFilterTest implements FilterFunction<Event> {
        @Override
        public boolean filter(Event e) throws Exception {
            return e.user.equals("Marry");
        }
    }
}
```

![image-20221215125602332](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221215125602332.png)

**3. 扁平映射（flatMap）**

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.typeinfo.TypeHint;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class TransFlatmapTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );

        //1.实现FlatMapFunction接口
        stream.flatMap(new MyFlatMap()).print("1");


        //2.传入一个lambda表达式
        stream.flatMap((Event value, Collector<String> out) -> {
            if (value.user.equals("Mary")) {
                out.collect(value.user);
            } else if (value.user.equals("Bob")) {
                out.collect(value.user);
                out.collect(value.url);
            }
        }).returns(new TypeHint<String>() {}).print("2");
        
        env.execute();


    }

    public static class MyFlatMap implements FlatMapFunction<Event, String> {
        @Override
        public void flatMap(Event value, Collector<String> out) throws Exception
        {
            if (value.user.equals("Mary")) {
                out.collect(value.user);
            } else if (value.user.equals("Bob")) {
                out.collect(value.user);
                out.collect(value.url);
            }
        }
    }

}
```

![image-20221215125822572](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221215125822572.png)

### 5.3.2 聚合算子（Aggregation）

**1.按键分区（keyBy）**

  **keyBy 是聚合前必须要用到的一个算子**。keyBy 通过指定键（key），可以将一条流从逻辑上划分成不同的分区（partitions）。这里所说的分区，其实就是并行处理的子任务，也就对应着任务槽（task slot）。

  基于不同的 key，流中的数据将被分配到不同的分区中去，这样一来，所有具有相同的 key 的数据，都将被发往同一个分区，那么下一步算子操作就将会在同一个 slot中进行处理了。

```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
public class TransKeyByTest {
     public static void main(String[] args) throws Exception {
         StreamExecutionEnvironment env = 
        StreamExecutionEnvironment.getExecutionEnvironment();
             env.setParallelism(1);
             DataStreamSource<Event> stream = env.fromElements(
             new Event("Mary", "./home", 1000L),
             new Event("Bob", "./cart", 2000L)
         );
         
         // 使用 Lambda 表达式进行分组
         KeyedStream<Event, String> keyedStream = stream.keyBy(e -> e.user);
         
         // 使用匿名类实现 KeySelector
         KeyedStream<Event, String> keyedStream1 = stream.keyBy(new 
            KeySelector<Event, String>() {
                 @Override
                 public String getKey(Event e) throws Exception {
                 return e.user;
            }
         });
         env.execute();
     }
}
```

**2. 简单聚合**

⚫ sum()：在输入流上，对指定的字段做叠加求和的操作。

⚫ min()：在输入流上，对指定的字段求最小值。

⚫ max()：在输入流上，对指定的字段求最大值。

⚫ minBy()：与 min()类似，在输入流上针对指定字段求最小值。不同的是，min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值；而 minBy()则会返回包含字段最小值的整条数据。

⚫ maxBy()：与 max()类似，在输入流上针对指定字段求最大值。两者区别与min()/minBy()完全一致

```java
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransTupleAggreationTest {
     public static void main(String[] args) throws Exception {
         StreamExecutionEnvironment env = 
        StreamExecutionEnvironment.getExecutionEnvironment();
        
         env.setParallelism(1);
         DataStreamSource<Tuple2<String, Integer>> stream = env.fromElements(
             Tuple2.of("a", 1),
             Tuple2.of("a", 3),
             Tuple2.of("b", 3),
             Tuple2.of("b", 4)
         );
         
         stream.keyBy(r -> r.f0).sum(1).print();
         stream.keyBy(r -> r.f0).sum("f1").print();
         
         stream.keyBy(r -> r.f0).max(1).print();
         stream.keyBy(r -> r.f0).max("f1").print();
         
         stream.keyBy(r -> r.f0).min(1).print();
         stream.keyBy(r -> r.f0).min("f1").print();
         
         stream.keyBy(r -> r.f0).maxBy(1).print();
         stream.keyBy(r -> r.f0).maxBy("f1").print();
         
         stream.keyBy(r -> r.f0).minBy(1).print();
         stream.keyBy(r -> r.f0).minBy("f1").print();
         
         env.execute();
     }
}
```

而如果数据流的类型是 POJO 类，那么就只能通过字段名称来指定，不能通过位置来指定了。

```java
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransPojoAggregationTest {
 public static void main(String[] args) throws Exception {
     StreamExecutionEnvironment env = 
      StreamExecutionEnvironment.getExecutionEnvironment();
     env.setParallelism(1);
     
     DataStreamSource<Event> stream = env.fromElements(
         new Event("Mary", "./home", 1000L),
         new Event("Bob", "./cart", 2000L)
     );
     stream.keyBy(e -> e.user).max("timestamp").print(); // 指定字段名称
     
     env.execute();
 }
```

**3.归约聚合**

  如果说简单聚合是对一些特定统计需求的实现，那么 reduce 算子就是一个一般化的聚合统计操作了。从大名鼎鼎的 MapReduce 开始，我们对 reduce 操作就不陌生：==它可以对已有的数据进行归约处理，把每一个新输入的数据和当前已经归约出来的值，再做一个聚合计算。==

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransformReduceTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //从元素读取数据
        DataStreamSource<Event> stream3 = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Marry", "./home", 1200L),
                new Event("Bob", "./prod?id=1", 1000L),
                new Event("Bob", "./home", 3500L),
                new Event("Bob", "./prod?id=2", 3200L)
        );


        //1.统计每个用户的访问频次
        SingleOutputStreamOperator<Tuple2<String, Long>> clicksByUser = stream3.map(new MapFunction<Event, Tuple2<String, Long>>() {

            @Override
            public Tuple2<String, Long> map(Event value) throws Exception {
                return Tuple2.of(value.user, 1L);
            }
        }).keyBy(data -> data.f0).reduce(new ReduceFunction<Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> reduce(Tuple2<String, Long> value1, Tuple2<String, Long> value2) throws Exception {
                return Tuple2.of(value1.f0, value1.f1 + value2.f1);
            }
        });

        //2.选取当前最活跃的用户
        SingleOutputStreamOperator<Tuple2<String, Long>> result = clicksByUser.keyBy(data -> "key").reduce(new ReduceFunction<Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> reduce(Tuple2<String, Long> value1, Tuple2<String, Long> value2) throws Exception {
                return value1.f1 > value2.f1 ? value1 : value2;
            }
        });

         clicksByUser.print("1");
         result.print("2");
         env.execute();
    }
}

```

![image-20221215131152255](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221215131152255.png)

### 5.3.3 用户自定义函数（UDF）

**1.函数类（Function Classes）**

```java
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;


public class TransFunctionUDFTest {
 public static void main(String[] args) throws Exception {
     StreamExecutionEnvironment env = 
    StreamExecutionEnvironment.getExecutionEnvironment();
     
     env.setParallelism(1);
  
     DataStreamSource<Event> clicks = env.fromElements(
         new Event("Mary", "./home", 1000L),
         new Event("Bob", "./cart", 2000L)
     );
     
     DataStream<Event> stream = clicks.filter(new FlinkFilter());
     stream.print();
     env.execute();
   }
  public static class FlinkFilter implements FilterFunction<Event> {
   
     public boolean filter(Event value) throws Exception {
         return value.url.contains("home");
     }
  }
}
```

**2.匿名函数（Lambda）**

```java
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransFunctionLambdaTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        DataStreamSource<Event> clicks = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L)
        );

        //map 函数使用 Lambda 表达式，返回简单类型，不需要进行类型声明
        DataStream<String> stream1 = clicks.map(event -> event.url);
        stream1.print();

        env.execute();
    }
}
```

![image-20221215131431053](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221215131431053.png)

**3.富函数类（Rich Function Classes）**

  “富函数类”也是 DataStream API 提供的一个函数类的接口，所有的 Flink 函数类都有其Rich 版本。富函数类一般是以抽象类的形式出现的。例如：RichMapFunction、RichFilterFunction、RichReduceFunction 等。

  富函数类可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能

  **Rich Function 有生命周期的概念。典型的生命周期方法有：**

⚫ open()方法，是 Rich Function 的初始化方法，也就是会开启一个算子的生命周期。当一个算子的实际工作方法例如 map()或者 filter()方法被调用之前，open()会首先被调用。所以像文件 IO 的创建，数据库连接的创建，配置文件的读取等等这样一次性的工作，都适合在 open()方法中完成。。

⚫ close()方法，是生命周期中的最后一个调用的方法，类似于解构方法。一般用来做一些清理工作。

  **需要注意的是，这里的生命周期方法，对于一个并行子任务来说只会调用一次；而对应的，实际工作方法，例如 RichMapFunction 中的 map()，在每条数据到来后都会触发一次调用。**

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class RichFunctionTest {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(3);

        DataStreamSource<Event> clicks = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=1", 5 * 1000L),
                new Event("Cary", "./home", 60 * 1000L)
        );

        clicks.map(new MyRichMapper()).print();
        env.execute();

    }

    //实现一个自定义的复函数类
    public static class MyRichMapper extends RichMapFunction<Event,Integer>{

        @Override
        public void open(Configuration parameters) throws Exception {
            super.open(parameters);
            System.out.println("open声明周期被调用"+getRuntimeContext().getIndexOfThisSubtask()+"号任务启动");
        }

        @Override
        public Integer map(Event value) throws Exception {
            return value.url.length();
        }

        @Override
        public void close() throws Exception {
            super.close();
            System.out.println("close声明周期被调用"+getRuntimeContext().getIndexOfThisSubtask()+"号任务结束");
        }
    }

}
```

![image-20221117100708249](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117100708249.png)

### 5.3.4 物理分区（Physical Partitioning）

  为了同keyBy相区别，我们把这些操作统称为“物理分区”操作。物理分区与keyBy另一大区别在于，**keyBy之后得到的是一个KeyedStream，而物理分区之后结果仍是DataStream**，且流中元素数据类型保持不变。从这一点也可以看出，分区算子并不对数据进行转换处理，只是定义了数据的传输方式。

**1.随机分区（shuffle）**

  最简单的重分区方式就是直接“洗牌”。通过调用DataStream的.shuffle()方法，将数据随机地分配到下游算子的并行任务中去。

​		随机分区服从均匀分布（uniform distribution），所以可以把流中的数据随机打乱，均匀地传递到下游任务分区。因为是完全随机的，所以对于同样的输入数据, 每次执行得到的结果也不会相同。

![image-20221117105745301](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117105745301.png)

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ShuffleTest {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Event> stream = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./prod?id=1", 1000L),
                new Event("Li", "./home", 3500L),
                new Event("Bob", "./prod?id=2", 3200L),
                new Event("Marry", "./home", 1200L),
                new Event("Bob", "./prod?id=3", 110L),
                new Event("Anna", "./home", 3550L),
                new Event("Li", "./prod?id=4", 3210L)
        );

        //经洗牌后打印输出，并行度为 4
        stream.shuffle().print("shuffle").setParallelism(4);

        env.execute();
    }
}
```

![image-20221117104400843](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104400843.png)

**2.询分区（Round-Robin）**

  轮询也是一种常见的重分区方式。简单来说就是“发牌”，按照先后顺序将数据做依次分发，。通过调用DataStream的.rebalance()方法，就可以实现轮询重分区。rebalance 使用的是Round-Robin负载均衡算法，可以将输入流数据平均分配到下游的并行任务中去。

![image-20221117105732840](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117105732840.png)

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ShuffleTest {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Event> stream = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./prod?id=1", 1000L),
                new Event("Li", "./home", 3500L),
                new Event("Bob", "./prod?id=2", 3200L),
                new Event("Marry", "./home", 1200L),
                new Event("Bob", "./prod?id=3", 110L),
                new Event("Anna", "./home", 3550L),
                new Event("Li", "./prod?id=4", 3210L)
        );



        // 经轮询重分区后打印输出，并行度为 4
        stream.rebalance().print("rebalance").setParallelism(4);
        //stream.print("rebalance").setParallelism(4);//默认使用
        env.execute();
    }
}

```

![image-20221117104328853](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104328853.png)

**3. 重缩放分区（rescale）**

  重缩放分区和轮询分区非常相似。当调用rescale()方法时，其实底层也是使用Round-Robin 算法进行轮询，但是只会将数据轮询发送到下游并行任务的一部分中，。也就是说，“发牌人”如果有多个，那么rebalance的方式是每个发牌人都面向所有人发牌；而rescale 的做法是分成小团体，发牌人只给自己团体内的所有人轮流发牌。



![image-20221117105759339](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117105759339.png)

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

public class ShuffleTest2 {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        
        env.addSource(new RichParallelSourceFunction<Integer>() {

            @Override
            public void run(SourceContext<Integer> sourceContext) throws Exception {
                for (int i = 0; i < 8; i++) {
                    // 将奇数发送到索引为 1 的并行子任务
                    // 将偶数发送到索引为 0 的并行子任务
                    if ((i + 1) % 2 == getRuntimeContext().getIndexOfThisSubtask()) {
                        sourceContext.collect(i + 1);
                    }
                }
            }

            @Override
            public void cancel() {

            }
        }).setParallelism(2).rescale().print().setParallelism(4);

        env.execute();
    }
}
```

![image-20221117104953515](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104953515.png)

**4.广播（broadcast）**

  这种方式其实不应该叫做“重分区”，因为经过==广播之后，数据会在不同的分区都保留一份，==可能进行重复处理。可以通过调用DataStream的broadcast()方法，将输入数据复制并发送到下游算子的所有并行任务中去。

```java
package com.fang.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ShuffleTest {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 读取数据源，并行度为 1
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=1", 5 * 1000L),
                new Event("Cary", "./home", 60 * 1000L)
        );
        // 经广播后打印输出，并行度为 4
        stream.broadcast().print("broadcast").setParallelism(4);
        env.execute();

    }
}
```

![image-20221117104723132](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104723132.png)

**5.全局分区**

  全局分区也是一种特殊的分区方式。这种做法非常极端，通过调用.global()方法，会==将所有的输入流数据都发送到下游算子的第一个并行子任务中去==。这就相当于强行让下游任务并行度变成了 1，所以使用这个操作需要非常谨慎，可能对程序造成很大的压力。

```
package com.fang.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ShuffleTest {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 读取数据源，并行度为 1
        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=1", 5 * 1000L),
                new Event("Cary", "./home", 60 * 1000L)
        );
        // 经广播后打印输出，并行度为 4
        stream.global().print().setParallelism(4);
        env.execute();

    }
}
```

![image-20221117104857452](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104857452.png)

**6.自定义分区**

  当 Flink 提 供 的 所 有 分 区 策 略 都 不 能 满 足 用 户 的 需 求 时 ， 我 们 可 以 通 过 使 用partitionCustom()方法来自定义分区策略。
  在调用时，方法需要传入两个参数，第一个是自定义分区器（Partitioner）对象，第二个是应用分区器的字段，它的指定方式与 keyBy 指定 key 基本一样：可以通过字段名称指定，也可以通过字段位置索引来指定，还可以实现一个 KeySelector

```java
package com.fang.chapter05;

import org.apache.flink.api.common.functions.Partitioner;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class ShuffleTest {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.fromElements(1, 2, 3, 4, 5, 6, 7, 8)
                .partitionCustom(new Partitioner<Integer>() {
                    @Override
                    public int partition(Integer key, int numPartitions) {
                        return key % 2;
                    }
                }, new KeySelector<Integer, Integer>() {
                    @Override
                    public Integer getKey(Integer value) throws Exception {
                        return value;
                    }
                })
                .print().setParallelism(2);

        env.execute();

    }
}
```

![image-20221117104038383](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117104038383.png)

## 5.4 输出算子（Sink）

![image-20221117140235397](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117140235397.png)

### 5.4.1 连接到外部系统

### 5.4.2 输出到文件

```java
package com.fang.chapter05;

import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;
import java.util.concurrent.TimeUnit;

//并行写入的效果
public class SinkToFileTest {
    public static void main(String[] args) throws Exception {



        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(4);

        DataStreamSource<Event> stream3 = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Marry", "./home", 1200L),
                new Event("Bob", "./prod?id=1", 1000L),
                new Event("Bob", "./home", 3500L),
                new Event("Bob", "./prod?id=2", 3200L)
        );


        StreamingFileSink<String> fileSink = StreamingFileSink.<String>forRowFormat(new Path("./output"),
                new SimpleStringEncoder<>("UTF-8"))
                //参数一：输出路径
                //参数二：输出字符编码格式
                .withRollingPolicy(
                //指定一个滚动策略
                        DefaultRollingPolicy.builder()
                                .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))//时间间隔毫秒数
                                .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))//当前不活跃的时间 5分钟没有数据就归档保存
                                .withMaxPartSize(1024 * 1024 * 1024)   //文件最大值
                                .build()
                ).build();
        // 将 Event 转换成 String 写入文件
        stream3.map(Event::toString).addSink(fileSink);

        env.execute();
    }
}
```

![image-20221117153723928](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117153723928.png)

### 5.4.3 输出到 Kafka

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;

public class SinkToKafkaTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();


        Properties properties = new Properties();
        properties.put("bootstrap.servers", "hadoop102:9092");


        DataStreamSource<String> stream = env.readTextFile("input/clicks.csv");

        stream.addSink(new FlinkKafkaProducer<String>("clicks", new SimpleStringSchema(), properties));

        env.execute();
    }
}
```

运行代码，在 Linux 主机启动一个消费者, 查看是否收到数据

```
 bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic clicks
```

![image-20221117154943602](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117154943602.png)

### 5.4.4 输出到 Redis

```
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.11</artifactId>
    <version>1.0</version>
</dependency>
```

```java
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.redis.RedisSink;
import org.apache.flink.streaming.connectors.redis.common.config.FlinkJedisPoolConfig;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommand;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommandDescription;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisMapper;

public class SinkToRedisTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Event> stream = env.addSource(new ClickSource());

        // 创建一个到redis连接的配置
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder()
                .setHost("hadoop102")
                .build();

        stream.addSink(new RedisSink<Event>(conf, new MyRedisMapper()));

        env.execute();
    }
    public static class MyRedisMapper implements RedisMapper<Event> {
        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "clicks");
        }

        @Override
        public String getKeyFromData(Event data) {
            return data.user;
        }

        @Override
        public String getValueFromData(Event data) {
            return data.url;
        }
    }

}
```

### 5.4.5 输出到 Elasticsearch

```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-elasticsearch7_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```

```java
import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink;
import org.apache.http.HttpHost;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

import java.util.ArrayList;
import java.util.HashMap;

public class SinkToEsTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=100", 3000L),
                new Event("Alice", "./prod?id=200", 3500L),
                new Event("Bob", "./prod?id=2", 2500L),
                new Event("Alice", "./prod?id=300", 3600L),
                new Event("Bob", "./home", 3000L),
                new Event("Bob", "./prod?id=1", 2300L),
                new Event("Bob", "./prod?id=3", 3300L));

        ArrayList<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("hadoop102", 9200, "http"));

        // 创建一个ElasticsearchSinkFunction
        ElasticsearchSinkFunction<Event> elasticsearchSinkFunction = new ElasticsearchSinkFunction<Event>() {
            @Override
            
            public void process(Event element, RuntimeContext ctx, RequestIndexer indexer) {
                HashMap<String, String> data = new HashMap<>();
                data.put(element.user, element.url);

                IndexRequest request = Requests.indexRequest()
                        .index("clicks")
                        .type("type")    // Es 6 必须定义 type
                        .source(data);

                indexer.add(request);
            }
        };

        stream.addSink(new ElasticsearchSink.Builder<Event>(httpHosts, elasticsearchSinkFunction).build());

        env.execute();
    }
}
```

### 5.4.6 输出到 MySQL（JDBC）

（1）添加依赖

```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
<dependency>
 <groupId>mysql</groupId>
 <artifactId>mysql-connector-java</artifactId>
 <version>5.1.47</version>
</dependency>
```

（2）启动 MySQL，在 database 库下建表 clicks

```
mysql> create table clicks(
 -> user varchar(20) not null,
 -> url varchar(100) not null);
```

（3）编写输出到 MySQL 的示例代码

```java
package com.atguigu.chapter05;

/**
 * Copyright (c) 2020-2030 尚硅谷 All Rights Reserved
 * <p>
 * Project:  FlinkTutorial
 * <p>
 * Created by  wushengran
 */

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.connector.jdbc.JdbcConnectionOptions;
import org.apache.flink.connector.jdbc.JdbcExecutionOptions;
import org.apache.flink.connector.jdbc.JdbcSink;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SinkToMySQL {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);


        DataStreamSource<Event> stream = env.fromElements(
                new Event("Mary", "./home", 1000L),
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=100", 3000L),
                new Event("Alice", "./prod?id=200", 3500L),
                new Event("Bob", "./prod?id=2", 2500L),
                new Event("Alice", "./prod?id=300", 3600L),
                new Event("Bob", "./home", 3000L),
                new Event("Bob", "./prod?id=1", 2300L),
                new Event("Bob", "./prod?id=3", 3300L));

        stream.addSink(
                JdbcSink.sink(
                        "INSERT INTO clicks (user, url) VALUES (?, ?)",
                        (statement, r) -> {
                            statement.setString(1, r.user);
                            statement.setString(2, r.url);
                        },
                        new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                                .withUrl("jdbc:mysql://localhost:3306/test")
                                .withDriverName("com.mysql.jdbc.Driver")
                                .withUsername("root")
                                .withPassword("root")
                                .build()
                )
        );
        env.execute();
    }
}
```

（4）运行代码，用客户端连接 MySQL，查看是否成功写入数据。

```
mysql> select * from clicks;
+------+--------------+
| user | url |
+------+--------------+
| Mary | ./home |
| Alice| ./prod?id=300 |
| Bob | ./prod?id=3 |
+------+---------------+
3 rows in set (0.00 sec)
```

# 第 6 章 Flink 中的时间和窗口

## 6.1 时间语义

### 6.1.1 Flink 中的时间语义

![image-20221117163355711](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117163355711.png)

**1.处理时间（Processing Time）**

  处理时间的概念非常简单，就是指执行处理操作的机器的系统时间。

**2.事件时间（Event Time）**

  事件时间，是指每个事件在对应的设备上发生的时间，也就是数据生成的时间。

### 6.1.2 哪种时间语义更重要

  实际应用中，数据==产生的时间==和==处理的时间==可能是完全不同的。很长时间收集起来的数据，处理或许只要一瞬间；也有可能数据量过大、处理能力不足，短时间堆了大量数据处理不完，==产生“背压”（back pressure）。==

  ==通常来说，处理时间是我们计算效率的衡量标准==，而==事件时间会更符合我们的业务计算逻辑==。所以==更多时候我们使用事件时间==；不过处理时间也不是一无是处。对于处理时间而言，由于没有任何附加考虑，数据一来就直接处理，因此这种方式可以让我们的流处理延迟降到最低，效率达到最高。

## 6.2 水位线（Watermark）

### 6.2.1 事件时间和窗口

![image-20221117165859376](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221117165859376.png)

### 6.2.2 什么是水位线

  在Flink中，**水位线是一种衡量Event Time进展的机制**，用来处理实时数据中的乱序问题的，通常是水位线和窗口结合使用来实现。 从设备生成实时流事件，到Flink的source，再到多个oparator处理数据，过程中会受到网络延迟、背压等多种因素影响造成数据乱序。

​		具体实现上，==水位线可以看作一条特殊的数据记录，它是插入到数据流中的一个标记点，主要内容就是一个时间戳，用来指示当前的事件时间。==而它插入流中的位置，就应该是在某个数据到来之后；这样就可以从这个数据中提取时间戳，作为当前水位线的时间戳了。

![image-20221119172614899](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119172614899.png)

**1.有序流中的水位线**

![image-20221118170851128](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118170851128.png)





**2.乱序流中的水位线**

  这里所说的“乱序”（out-of-order），是指数据的先后顺序不一致，主要就是基于数据的产生时间而言的。

![image-20221118171008781](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118171008781.png)

  我们插入新的水位线时，要先判断一下时间戳是否比之前的大，否则就不再生成新的水位线

![image-20221118171221753](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118171221753.png)

  如果考虑到大量数据同时到来的处理效率，我们同样可以周期性地生成水位线。这时只需要保存一下之前所有数据中的最大时间戳，需要插入水位线时，就直接以它作为时间戳生成新的水位线

![image-20221118171234685](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118171234685.png)

  为了让窗口能够正确收集到迟到的数据，我们也可以等上 2 秒；也就是用当前已有数据的最大时间戳减去 2 秒，就是要插入的水位线的时间戳

![image-20221118171246223](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118171246223.png)

**3.水位线的特性**

⚫ 水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据

⚫ 水位线主要的内容是一个时间戳，用来表示当前事件时间的进展

⚫ 水位线是基于数据的时间戳生成的

⚫ 水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进

⚫ 水位线可以通过设置延迟，来保证正确处理乱序数据

⚫ 一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t 的数据

### 6.2.3 如何生成水位线

**1.生成水位线的总体原则**

  我们知道，完美的水位线是“绝对正确”的，也就是一个水位线一旦出现，就表示这个时间之前的数据已经全部到齐、之后再也不会出现了。而完美的东西总是可望不可即，==我们只能尽量去保证水位线的正确。如果对结果正确性要求很高、想要让窗口收集到所有数据，==我们该怎么做呢？

  ==一个字，等==。由于网络传输的延迟不确定，为了获取所有迟到数据，我们只能等待更长的时间。作为筹划全局的程序员，我们当然不会傻傻地一直等下去。那到底等多久呢？这就需要对相关领域有一定的了解了。比如，如果我们知道当前业务中事件的迟到时间不会超过 5 秒，那就可以将水位线的时间戳设为当前已有数据的最大时间戳减去 5 秒，相当于设置了 5 秒的延迟等待。

**2.水位线生成策略（Watermark Strategies）**

```java
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.time.Duration;

public class WatermarkTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.getConfig().setAutoWatermarkInterval(100);

        //从元素中读取数据
         SingleOutputStreamOperator<Event> stream = env.fromElements(
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./home", 1100L),
                new Event("Marry", "./home", 1000L),
                new Event("Bob", "./prod?id=1", 1000L),
                new Event("Bob", "./home", 3500L),
                new Event("Bob", "./prod?id=2", 3200L)
                //有序流的watermark生成
//        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
//        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
//            @Override
//            public long extractTimestamp(Event element, long recordTimestamp) {
//                return element.timestamp;
//            }
//        })  //提取时间戳，生成水位线
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2))
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event element, long recordTimestamp) {
                        return element.timestamp;
                    }
                })
        );
        env.execute();
    }
}
```

它主要用来为流中的数据分配时间戳，并生成水位线来指示事件时间

```java
.assignTimestampsAndWatermarks()
```

  .assignTimestampsAndWatermarks()方法需要传入一个 WatermarkStrategy 作为参数，这就
是 所 谓 的 “ 水 位 线 生 成 策 略 ” 

  WatermarkStrategy 中 包 含 了 一 个 “ 时 间 戳 分 配器”TimestampAssigner 和一个“水位线生成器”WatermarkGenerator

**3.Flink 内置水位线生成器**

**（1）有序流**

```java
stream.assignTimestampsAndWatermarks(
 WatermarkStrategy.<Event>forMonotonousTimestamps()
 .withTimestampAssigner(new SerializableTimestampAssigner<Event>() 
{
 		@Override
		public long extractTimestamp(Event element, long recordTimestamp) 
         {
 			return element.timestamp;
 		   }
 		 })
);
```

**（2）乱序流**

  由于乱序流中需要等待迟到数据到齐，所以必须设置一个固定量的延迟时间（Fixed Amount of Lateness）

```java
.assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2))
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event element, long recordTimestamp) {
                        return element.timestamp;
                    }
                })
        );
```

### 6.2.4 水位线的传递

  水位线定义的本质了：它表示的是“当前时间之前的数据，都已经到齐了”。这是一种保证，告诉下游任务“只要你接到这个水位线，就代表之后我不会再给你发更早的数据了，你可以放心做统计计算而不会遗漏数据”。所以如果一个任务收到了来自上游并行任务的不同的水位线，说明上游各个分区处理得有快有慢，进度各不相同比如上游有两个并行子任务都发来了水位线，一个是 5 秒，一个是 7 秒；这代表第一个并行任务已经处理完 5 秒之前的所有数据，而第二个并行任务处理到了 7 秒。那这时自己的时钟怎么确定呢？

  当然也要以“这之前的数据全部到齐”为标准。如果我们以较大的水位线 7 秒作为当前时间，那就表示“7 秒前的数据都已经处理完”，这显然不是事实——第一个上游分区才处理到 5 秒，5~7 秒的数据还会不停地发来；而如果以最小的水位线 5 秒作为当前时钟就不会有这个问题了，因为确实所有上游分区都已经处理完，不会再发 5 秒前的数据了。这让我们想到“木桶原理”：==所有的上游并行任务就像围成木桶的一块块木板，它们中最短的那一块，决定了我们桶中的水位。==

![image-20221118174258154](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118174258154.png)

### 6.2.5 水位线的计算

水位线的默认计算公式：水位线 = 观察到的最大事件时间 – 最大延迟时间 – 1 毫秒

## 6.3 窗口（Window）

### 6.3.1 窗口的概念

  Flink 是一种流式计算引擎，主要是来处理无界数据流的，数据源源不断、无穷无尽。想要更加方便高效地处理无界流，一种方式就是将无限数据切割成有限的“数据块”进行处理，这就是所谓的“窗口”（Window）。

![image-20221118174543309](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118174543309.png)

  所以在 Flink 中，窗口其实并不是一个“框”，流进来的数据被框住了就只能进这一个窗口。相比之下，我们应该把窗口理解成一个“桶”。在 Flink 中，窗口可以把流切割成有限大小的多个“存储桶”（bucket)；每个数据都会分发到对应的桶中，当到达窗口结束时间时，就对每个桶中收集的数据进行计算处理。

![image-20221118174647125](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118174647125.png)

### 6.3.2 窗口的分类

**1.按照驱动类型分类**

**（1）时间窗口（Time Window）**

  时间窗口以时间点来定义窗口的开始（start）和结束（end），所以截取出的就是某一时间段的数据。到达结束时间时，窗口不再收集数据，触发计算输出结果，并将窗口关闭销毁。所以可以说基本思路就是“定点发车”。

**（2）计数窗口（Count Window）**

  计数窗口基于元素的个数来截取数据，到达固定的个数时就触发计算并关闭窗口。这相当于座位有限、“人满就发车”，是否发车与时间无关。每个窗口截取数据的个数，就是窗口的大小。

![image-20221118175314162](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118175314162.png)

**2.按照窗口分配数据的规则分类**

**（1）滚动窗口（Tumbling Windows）**

  滚动窗口有固定的大小，是一种对数据进行“均匀切片”的划分方式。窗口之间没有重叠，也不会有间隔，是“首尾相接”的状态。

![image-20221118175539307](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118175539307.png)

**（2）滑动窗口（Sliding Windows）**

  与滚动窗口类似，滑动窗口的大小也是固定的。区别在于，窗口之间并不是首尾相接的，而是可以“错开”一定的位置。如果看作一个窗口的运动，那么就像是向前小步“滑动”一样。

![image-20221118175551399](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118175551399.png)

**（3）会话窗口（Session Windows）**

  会话窗口顾名思义，是基于“会话”（session）来来对数据进行分组的。这里的会话类似Web 应用中 session 的概念，不过并不表示两端的通讯过程，而是借用会话超时失效的机制来描述窗口。简单来说，就是数据来了之后就开启一个会话窗口，如果接下来还有数据陆续到来，那么就一直保持会话；如果一段时间一直没收到数据，那就认为会话超时失效，窗口自动关闭。



![image-20221118175604219](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118175604219.png)

**（4）全局窗口（Global Windows）**

  还有一类比较通用的窗口，就是“全局窗口”。这种窗口全局有效，会把相同 key 的所有数据都分配到同一个窗口中；说直白一点，就跟没分窗口一样。无界流的数据永无止尽，所以这种窗口也没有结束的时候，默认是不会做触发计算的。

![image-20221118175650018](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221118175650018.png)

### 6.3.3 窗口 API 概览

**1.按键分区（Keyed）和非按键分区（Non-Keyed）**

**（1）按键分区窗口（Keyed Windows）**

  在调用窗口算子之前，是否有 keyBy 操作。

```
stream.keyBy(...)
 .window(...)
```

**（2）非按键分区（Non-Keyed Windows）**

  推荐KeyBy之后再开窗

> 这时窗口逻辑只能在一个任务（task）上执行，就相当于并行度变成了 1。所以在实际应用中一般不推荐使用这种方式。

```
stream.windowAll(...)
```

**2.代码中窗口 API 的调用**

```
stream.keyBy(<key selector>)
 	.window(<window assigner>)  //窗口分配器
 	.aggregate(<window function>)  //窗口函数
```

### 6.3.4 窗口分配器（Window Assigners）

**1.时间窗口**

**（1）滚动处理时间窗口**

  窗口分配器由类 TumblingProcessingTimeWindows 提供，需要调用它的静态方法.of()

```
stream.keyBy(...)
.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
.aggregate(...)
```

  这里.of()方法需要传入一个 Time 类型的参数 size，表示滚动窗口的大小，我们这里创建了一个长度为 5 秒的滚动窗口。

**（2）滑动处理时间窗口**

窗口分配器由类 SlidingProcessingTimeWindows 提供，同样需要调用它的静态方法.of()

```
stream.keyBy(...)
  .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
  .aggregate(...)
```

  这里.of()方法需要传入两个 Time 类型的参数：size 和 slide，前者表示滑动窗口的大小，后者表示滑动窗口的滑动步长。我们这里创建了一个长度为 10 秒、滑动步长为 5 秒的滑动窗口。

  滑动窗口同样可以追加第三个参数，用于指定窗口起始点的偏移量，用法与滚动窗口完全一致。

**（3）处理时间会话窗口**

  窗口分配器由类 ProcessingTimeSessionWindows 提供，需要调用它的静态方法.withGap()或者.withDynamicGap()。

```
stream.keyBy(...)
  .window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
  .aggregate(...)
```

  这里.withGap()方法需要传入一个 Time 类型的参数 size，表示会话的超时时间，也就是最小间隔 session gap。我们这里创建了静态会话超时时间为 10 秒的会话窗口。

**（4）滚动事件时间窗口**

  窗口分配器由类 TumblingEventTimeWindows 提供，用法与滚动处理事件窗口完全一致。

```
stream.keyBy(...)
	.window(TumblingEventTimeWindows.of(Time.seconds(5)))
	.aggregate(...)
```

  这里.of()方法也可以传入第二个参数 offset，用于设置窗口起始点的偏移量。

**（5）滑动事件时间窗口**

  窗口分配器由类 SlidingEventTimeWindows 提供，用法与滑动处理事件窗口完全一致。

```
stream.keyBy(...)
	.window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
	.aggregate(...)
```

**（6）事件时间会话窗口**

  窗口分配器由类 EventTimeSessionWindows 提供，用法与处理事件会话窗口完全一致

```
stream.keyBy(...)
	.window(EventTimeSessionWindows.withGap(Time.seconds(10)))
	.aggregate(...)
```



**2.计数窗口**

**（1）滚动计数窗口**

  滚动计数窗口只需要传入一个长整型的参数 size，表示窗口的大小

```
stream.keyBy(...)
	.countWindow(10)
```

**（2）滑动计数窗口**

  与滚动计数窗口类似，不过需要在.countWindow()调用时传入两个参数：size 和 slide，前者表示窗口大小，后者表示滑动步长。

```
stream.keyBy(...)
	.countWindow(10，3)
```

  我们定义了一个长度为 10、滑动步长为 3 的滑动计数窗口。每个窗口统计 10 个数据，每隔 3 个数据就统计输出一次结果。

**3.全局窗口**

  全局窗口是计数窗口的底层实现，一般在需要自定义窗口时使用。它的定义同样是直接调用.window()，分配器由 GlobalWindows 类提供。

```
stream.keyBy(...)
	.window(GlobalWindows.create());
```

  需要注意使用全局窗口，必须自行定义触发器才能实现窗口计算，否则起不到任何作用。

### 6.3.5 窗口函数（Window Functions）

![image-20221119163752972](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119163752972.png)

**1.增量聚合函数（incremental aggregation functions）**

  每来一条数据就立即进行计算，中间只要保持一个简单的聚合状态就可以了

**（1）归约函数（ReduceFunction）**

  最基本的聚合方式就是归约（reduce）。我们在基本转换的聚合算子中介绍过 reduce 的用法，窗口的归约聚合也非常类似，就是将窗口中收集到的数据两两进行归约。

```java
package com.fang.chapter06;


import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;


import java.time.Duration;

public class WindowTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.getConfig().setAutoWatermarkInterval(100);

        //从元素中读取数据
        SingleOutputStreamOperator<Event> stream = env.addSource(new ClinkSource())
            
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long
                                    recordTimestamp) {
                                return element.timestamp;
                            }}));


        stream.map(new MapFunction<Event, Tuple2<String,Long>>() {
            public Tuple2<String, Long> map(Event value) throws Exception {
                return Tuple2.of(value.user,1L);
            }
        })
                .keyBy(data->data.f0)
                .window(TumblingProcessingTimeWindows.of(Time.seconds(10)))//滚动事件时间窗口
                .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                    @Override
                    public Tuple2<String, Long> reduce(Tuple2<String, Long> value1, Tuple2<String, Long> value2) throws Exception {
                        return Tuple2.of(value1.f0, value1.f1+value2.f1);
                    }
                })
                .print();

        env.execute();
    }
}

```

![image-20221119163607311](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119163607311.png)

**（2）聚合函数（AggregateFunction）**

  ReduceFunction 可以解决大多数归约聚合的问题，但是这个接口有一个限制，就是聚合状态的类型、输出结果的类型都必须和输入数据类型一样。这就迫使我们必须在==聚合前，先将数据转换（map）成预期结果类型；==而在有些情况下，还需要对状态进行进一步处理才能得到输出结果，这时它们的类型可能不同，使用 ReduceFunction 就会非常麻烦。

  Flink 的 Window API 中的 aggregate 就提供了这样的操作。==直接基于 WindowedStream 调用.aggregate()方法，就可以定义更加灵活的窗口聚合操作。这个方法需要传入一个AggregateFunction 的实现类作为参数。==AggregateFunction 在源码中的定义如下：

```java
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable 
{
	 ACC createAccumulator();
	 ACC add(IN value, ACC accumulator);
	 OUT getResult(ACC accumulator);
	 ACC merge(ACC a, ACC b);
}
```

**接口中有四个方法：**

⚫ createAccumulator()：创建一个累加器，这就是为聚合创建了一个初始状态，每个聚合任务只会调用一次。

⚫ add()：将输入的元素添加到累加器中。这就是基于聚合状态，对新来的数据进行进一步聚合的过程。方法传入两个参数：当前新到的数据 value，和当前的累加器accumulator；返回一个新的累加器值，也就是对聚合状态进行更新。每条数据到来之后都会调用这个方法。

⚫ getResult()：从累加器中提取聚合的输出结果。也就是说，我们可以定义多个状态，然后再基于这些聚合的状态计算出一个结果进行输出。比如之前我们提到的计算平均值，就可以把 sum 和 count 作为状态放入累加器，而在调用这个方法时相除得到最终结果。这个方法只在窗口要输出结果时调用。

⚫ merge()：合并两个累加器，并将合并后的状态作为一个累加器返回。这个方法只在需要合并窗口的场景下才会被调用；最常见的合并窗口（Merging Window）的场景就是会话窗口（Session Windows）。

```java
import java.util.HashSet;

import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;


public class WindowAggregateTest2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //PV(访问量)： 即Page View, 即页面浏览量或点击量，用户每次刷新即被计算一次。
        //UV(独立访客)：即Unique Visitor,访问您网站的一台电脑客户端为一个访客

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));

        // 所有数据设置相同的key，发送到同一个分区统计PV和UV，再相除
        stream.keyBy(data -> true)
                .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(2)))
                .aggregate(new AvgPv())
                .print();
        env.execute();
    }

    public static class AvgPv implements AggregateFunction<Event, Tuple2<HashSet<String>, Long>, Double> {
        @Override
        public Tuple2<HashSet<String>, Long> createAccumulator() {
            // 创建累加器
            return Tuple2.of(new HashSet<String>(), 0L);
        }

        @Override
        public Tuple2<HashSet<String>, Long> add(Event value, Tuple2<HashSet<String>, Long> accumulator) {
            // 属于本窗口的数据来一条累加一次，并返回累加器
            accumulator.f0.add(value.user);
            return Tuple2.of(accumulator.f0, accumulator.f1 + 1L);
        }

        @Override
        public Double getResult(Tuple2<HashSet<String>, Long> accumulator) {
            // 窗口闭合时，增量聚合结束，将计算结果发送到下游
            return (double) accumulator.f1 / accumulator.f0.size();  //平均每个用户活跃度
        }

        @Override
        public Tuple2<HashSet<String>, Long> merge(Tuple2<HashSet<String>, Long> a, Tuple2<HashSet<String>, Long> b) {
            return null;
        }
    }
}
```

![image-20221119163637406](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119163637406.png)

**2.全窗口函数（full window functions）**

  窗口操作中的另一大类就是全窗口函数。与增量聚合函数不同，全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。

**（1）窗口函数（WindowFunction）**

  WindowFunction 字面上就是“窗口函数”，它其实是老版本的通用窗口函数接口。我们可以基于 WindowedStream 调用.apply()方法，传入一个 WindowFunction 的实现类。

```java
stream
 .keyBy(<key selector>)
 .window(<window assigner>)
 .apply(new MyWindowFunction());
```

  这个类中可以获取到包含窗口所有数据的可迭代集合（Iterable），还可以拿到窗口（Window）本身的信息。WindowFunction 接口在源码中实现如下：

```java
public interface WindowFunction<IN, OUT, KEY, W extends Window> extends Function, 
Serializable {
	void apply(KEY key, W window, Iterable<IN> input, Collector<OUT> out) throws 
	Exception;
}
```

  当窗口到达结束时间需要触发计算时，就会调用这里的 apply 方法。我们可以从 input 集合中取出窗口收集的数据，结合 key 和 window 信息，通过收集器（Collector）输出结果。这里 Collector 的用法，与 FlatMapFunction 中相同。

  不过我们也看到了，WindowFunction 能提供的上下文信息较少，也没有更高级的功能。事实上，它的作用可以被 ProcessWindowFunction 全覆盖，所以之后可能会逐渐弃用。一般在实际应用，直接使用 ProcessWindowFunction 就可以了

**（2）处理窗口函数（ProcessWindowFunction）**

```java
import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;
import java.time.Duration;
import java.util.HashSet;

public class UvCountByWindowExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));

        // 将数据全部发往同一分区，按窗口统计UV
        stream.keyBy(data -> true)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .process(new UvCountByWindow())
                .print();

        env.execute();
    }

    // 自定义窗口处理函数
    public static class UvCountByWindow extends ProcessWindowFunction<Event, String, Boolean, TimeWindow> {
        @Override
        public void process(Boolean aBoolean, Context context, Iterable<Event> elements, Collector<String> out) throws Exception {
            HashSet<String> userSet = new HashSet<>();
            // 遍历所有数据，放到Set里去重
            for (Event event: elements){
                userSet.add(event.user);
            }
            // 结合窗口信息，包装输出内容
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            out.collect("窗口: " + new Timestamp(start) + " ~ " + new Timestamp(end)
                    + " 的独立访客数量是：" + userSet.size());
        }
    }
}
```

![image-20221119163418234](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119163418234.png)



**3.增量聚合和全窗口函数的结合使用**

```java
package com.fang.chapter06;

import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;


public class UrlViewCountExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));

        // 需要按照url分组，开滑动窗口统计
        stream.keyBy(data -> data.url)
                .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                // 同时传入增量聚合函数和全窗口函数
                .aggregate(new UrlViewCountAgg(), new UrlViewCountResult())
                .print();

        env.execute();
    }

    // 自定义增量聚合函数，来一条数据就加一
    public static class UrlViewCountAgg implements AggregateFunction<Event, Long, Long> {
        @Override
        public Long createAccumulator() {
            return 0L;
        }

        @Override
        public Long add(Event value, Long accumulator) {
            return accumulator + 1;
        }

        @Override
        public Long getResult(Long accumulator) {
            return accumulator;
        }

        @Override
        public Long merge(Long a, Long b) {
            return null;
        }
    }

    // 自定义窗口处理函数，只需要包装窗口信息
    public static class UrlViewCountResult extends ProcessWindowFunction<Long, UrlViewCount, String, TimeWindow> {

        @Override
        public void process(String url, Context context, Iterable<Long> elements, Collector<UrlViewCount> out) throws Exception {
            // 结合窗口信息，包装输出内容
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            // 迭代器中只有一个元素，就是增量聚合函数的计算结果
            out.collect(new UrlViewCount(url, elements.iterator().next(), start, end));
        }
    }
}
```

![image-20221119163454925](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221119163454925.png)

### 6.3.6 测试水位线和窗口的使用

```java
package com.fang.chapter06;

import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.time.Duration;

public class WatermarkTest2 {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 将数据源改为 socket 文本流，并转换成 Event 类型
        env.socketTextStream("hadoop102", 7777)
                .map(new MapFunction<String, Event>() {
                    @Override
                    public Event map(String value) throws Exception {
                        String[] fields = value.split(",");
                        return new Event(fields[0].trim(), fields[1].trim(),
                                Long.valueOf(fields[2].trim()));
                    }
                })
                // 插入水位线的逻辑
                .assignTimestampsAndWatermarks(
                        // 针对乱序流插入水位线，延迟时间设置为 5s
                        WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))
                                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                    // 抽取时间戳的逻辑
                                    @Override
                                    public long extractTimestamp(Event element, long
                                            recordTimestamp) {
                                        return element.timestamp;
                                    }
                                })
                )
                // 根据 user 分组，开窗统计
                .keyBy(data -> data.user)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))//窗口大小为10s
                .process(new WatermarkTestResult())
                .print();

        env.execute();
    }
    // 自定义处理窗口函数，输出当前的水位线和窗口信息
    public static class WatermarkTestResult extends ProcessWindowFunction<Event,
            String, String, TimeWindow>{
        @Override
        public void process(String s, Context context, Iterable<Event> elements,
                            Collector<String> out) throws Exception {
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            Long currentWatermark = context.currentWatermark();
            Long count = elements.spliterator().getExactSizeIfKnown();
            out.collect("窗口" + start + " ~ " + end + "中共有" + count + "个元素，窗口闭合计算时，水位线处于：" + currentWatermark);

        }
    }

}

```

我们这里设置的最大延迟时间是 5 秒，所以当我们在终端启动 nc 程序，也就是 nc –lk 7777

**然后输入如下数据时：**

```
Alice, ./home, 1000
Alice, ./cart, 2000
Alice, ./prod?id=100, 10000
Alice, ./prod?id=200, 8000
Alice, ./prod?id=300, 15000
```

**我们会看到如下结果：**

```
  窗口 0 ~ 10000 中共有 3 个元素，窗口闭合计算时，水位线处于：9999
```

  我们就会发现，当最后输入[Alice, ./prod?id=300, 15000]时，流中会周期性地（默认 200毫秒）插入一个时间戳为 15000L – 5 * 1000L – 1L = 9999 毫秒的水位线，已经到达了窗口[0,10000)的结束时间，所以会触发窗口的闭合计算。而后面再输入一条[Alice, ./prod?id=200, 9000]时，将不会有任何结果；因为这是一条迟到数据，它所属于的窗口已经触发计算然后销毁了（窗口默认被销毁），所以无法再进入到窗口中，自然也就无法更新计算结果了。窗口中的迟到数据默认会被丢弃，这会导致计算结果不够准确。Flink 提供了有效处理迟到数据的手段。

### 6.3.7 其他 API

**1.触发器（Trigger）**

  触发器主要是用来控制窗口什么时候触发计算。所谓的“触发计算”，本质上就是执行窗口函数，所以可以认为是计算得到结果并输出的过程。

  基于 WindowedStream 调用.trigger()方法，就可以传入一个自定义的窗口触发器（Trigger）。

```java
stream.keyBy(...)
	 .window(...)
	 .trigger(new MyTrigger())
```

==Trigger 是一个抽象类，自定义时必须实现下面四个抽象方法：==

⚫ onElement()：窗口中每到来一个元素，都会调用这个方法。

⚫ onEventTime()：当注册的事件时间定时器触发时，将调用这个方法。

⚫ onProcessingTime ()：当注册的处理时间定时器触发时，将调用这个方法。

⚫ clear()：当窗口关闭销毁时，调用这个方法。一般用来清除自定义的状态。

  ==以上三个方法返回类型都是 TriggerResult，这是一个枚举类型（enum），其中定义了对窗口进行操作的四种类型。==

⚫ CONTINUE（继续）：什么都不做

⚫ FIRE（触发）：触发计算，输出结果

⚫ PURGE（清除）：清空窗口中的所有数据，销毁窗口

⚫ FIRE_AND_PURGE（触发并清除）：触发计算输出结果，并清除窗口

**2.移除器（Evictor）**

  移除器主要用来定义移除某些数据的逻辑。基于 WindowedStream 调用.evictor()方法，就可以传入一个自定义的移除器（Evictor）。Evictor 是一个接口，不同的窗口类型都有各自预实现的移除器。

```java
stream.keyBy(...)
	 .window(...)
	 .evictor(new MyEvictor())
```

Evictor 接口定义了两个方法：

⚫ evictBefore()：定义执行窗口函数之前的移除数据操作

⚫ evictAfter()：定义执行窗口函数之后的以处数据操作

默认情况下，预实现的移除器都是在执行窗口函数（window fucntions）之前移除数据的。

**3.允许延迟（Allowed Lateness）**

  基于 WindowedStream 调用.allowedLateness()方法，传入一个 Time 类型的延迟时间，就可以表示允许这段时间内的延迟数据。

  我们可以设定允许延迟一段时间，在这段时间内，窗口不会销毁，继续到来的数据依然可以进入窗口中并触发计算。直到水位线推进到了 窗口结束时间 + 延迟时间，才真正将窗口的内容清空，正式关闭窗口。

```java
stream.keyBy(...)
 	.window(TumblingEventTimeWindows.of(Time.hours(1)))
 	.allowedLateness(Time.minutes(1))
```

**4.将迟到的数据放入侧输出流**

  将未收入窗口的迟到数据，放入“侧输出流”（side output）进行另外的处理。所谓的侧输出流，相当于是数据流的一个“分支”，这个流中单独放置那些错过了该上的车、本该被丢弃的数据。

  基于 WindowedStream 调用.sideOutputLateData() 方法，就可以实现这个功能。方法需要传入一个“输出标签”（OutputTag），用来标记分支的迟到数据流。因为保存的就是流中的原始数据，所以 OutputTag 的类型与流中数据类型相同。

```
DataStream<Event> stream = env.addSource(...);
OutputTag<Event> outputTag = new OutputTag<Event>("late") {};
stream.keyBy(...)
 .window(TumblingEventTimeWindows.of(Time.hours(1)))
 .sideOutputLateData(outputTag)
```

  将迟到数据放入侧输出流之后，还应该可以将它提取出来。基于窗口处理完成之后的DataStream，调用.getSideOutput()方法，传入对应的输出标签，就可以获取到迟到数据所在的流了。

```java
SingleOutputStreamOperator<AggResult> winAggStream = stream.keyBy(...)
	 .window(TumblingEventTimeWindows.of(Time.hours(1)))
 	.sideOutputLateData(outputTag)
	 .aggregate(new MyAggregateFunction())
	 DataStream<Event> lateStream =    
 	 winAggStream.getSideOutput(outputTag);
```

  这里注意，getSideOutput()是 SingleOutputStreamOperator 的方法，获取到的侧输出流数据类型应该和 OutputTag 指定的类型一致，与窗口聚合之后流中的数据类型可以不同。

### 6.3.8 窗口的生命周期

**1.窗口的创建**

  窗口的类型和基本信息由窗口分配器（window assigners）指定，但窗口不会预先创建好，而是由数据驱动创建。<font color='red'>当第一个应该属于这个窗口的数据元素到达时，就会创建对应的窗口。</font>

**2.窗口计算的触发**

  除了窗口分配器，每个窗口还会有自己的窗口函数（window functions）和触发器（trigger）。窗口函数可以分为增量聚合函数和全窗口函数，主要定义了窗口中计算的逻辑；而触发器则是指定调用窗口函数的条件。对于不同的窗口类型，触发计算的条件也会不同。例如，一个滚动事件时间窗口，应该在水位线到达窗口结束时间的时候触发计算，属于“定点发车”；而一个计数窗口，会在窗口中元素数量达到定义大小时触发计算，属于“人满就发车”。所以 Flink 预定义的窗口类型都有对应内置的触发器。

  对于事件时间窗口而言，除去到达结束时间的“定点发车”，还有另一种情形。当我们设置了允许延迟，那么如果水位线超过了窗口结束时间、但还没有到达设定的最大延迟时间，这期间内到达的迟到数据也会触发窗口计算。这类似于没有准时赶上班车的人又追上了车，这时车要再次停靠、开门，将新的数据整合统计进来。

**3.窗口的销毁**

  一般情况下，当时间达到了结束点，就会直接触发计算输出结果、进而清除状态销毁窗口。这时窗口的销毁可以认为和触发计算是同一时刻。这里需要注意，Flink 中只对时间窗口（TimeWindow）有销毁机制；由于计数窗口（CountWindow）是基于全局窗口（GlobalWindw）实现的，而全局窗口不会清除状态，所以就不会被销毁。在特殊的场景下，窗口的销毁和触发计算会有所不同。事件时间语义下，如果设置了允许延迟，那么在水位线到达窗口结束时间时，仍然不会销毁窗口；窗口真正被完全删除的时间点，是窗口的结束时间加上用户指定的允许延迟时间。

**4.窗口 API 调用总结**

  到目前为止，我们已经彻底明白了 Flink 中窗口的概念和 Window API 的调用，我们再用一张图做一个完整总结。

![image-20221120105631721](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120105631721.png)

## 6.4 迟到数据的处理

### 6.4.1 设置水位线延迟时间

  水位线是事件时间的进展，它是我们整个应用的全局逻辑时钟。水位线生成之后，会随着数据在任务间流动，从而给每个任务指明当前的事件时间。所以从这个意义上讲，==水位线是一个覆盖万物的存在，它并不只针对事件时间窗口有效。==

  之前我们讲到触发器时曾提到过“定时器”，时间窗口的操作底层就是靠定时器来控制触发的。既然是底层机制，定时器自然就不可能是窗口的专利了；事实上它是 Flink 底层 API——处理函数（process function）的重要部分。

  所以水位线其实是所有事件时间定时器触发的判断标准。那么==水位线的延迟，当然也就是全局时钟的滞后==，相当于是上帝拨动了琴弦，所有人的表都变慢了

### 6.4.2 允许窗口处理迟到数据

  由于大部分乱序数据已经被水位线的延迟等到了，所以往往迟到的数据不会太多。这样，我们会在水位线到达窗口结束时间时，先快速地输出一个近似正确的计算结果；然后保持窗口继续等到延迟数据，每来一条数据，窗口就会再次计算，并将更新后的结果输出。这样就可以逐步修正计算结果，最终得到准确的统计值了。

  大多数人是在发车时刻前后到达的，所以我们只要把表调慢，稍微等一会儿，绝大部分人就都上车了，这个把表调慢的时间就是水位线的延迟；到点之后，班车就准时出发了，不过可能还有该来的人没赶上。于是我们就先慢慢往前开，这段时间内，如果迟到的人抓点紧还是可以追上的；如果有人追上来了，就停车开门让他上来，然后车继续向前开。

### 6.4.3 将迟到数据放入窗口侧输出流

  用窗口的侧输出流来收集关窗以后的迟到数据。这种方式是最后“兜底”的方法，只能保证数据不丢失；因为窗口已经真正关闭，所以是无法基于之前窗口的结果直接做更新的。我们只能将之前的窗口计算结果保存下来，然后获取侧输出流中的迟到数据，判断数据所属的窗口，手动对结果进行合并更新。尽管有些烦琐，实时性也不够强，但能够保证最终结果一定是正确的。

### 6.4.4综合案例

```java
package com.fang.chapter06;

import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

import java.time.Duration;

public class ProcessLateDataExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);
        // 读取 socket 文本流
        SingleOutputStreamOperator<Event> stream =
                env.socketTextStream("hadoop102", 7777)
                        .map(new MapFunction<String, Event>() {
                            @Override
                            public Event map(String value) throws Exception {
                                String[] fields = value.split(" ");
                                return new Event(fields[0].trim(), fields[1].trim(),
                                        Long.valueOf(fields[2].trim()));
                            }
                        })

                        // 方式一：设置 watermark 延迟时间，2 秒钟
                        .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2))
                                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                                                           @Override
                                                                           public long extractTimestamp(Event element, long
                                                                                   recordTimestamp) {
                                                                               return element.timestamp;
                                                                           }
                                                                       }));
        // 定义侧输出流标签
        OutputTag<Event> outputTag = new OutputTag<Event>("late"){};
        SingleOutputStreamOperator<UrlViewCount> result = stream.keyBy(data ->
                data.url)
                .window(TumblingEventTimeWindows.of(Time.seconds(10)))
                // 方式二：允许窗口处理迟到数据，设置 1 分钟的等待时间
                .allowedLateness(Time.minutes(1))

        // 方式三：将最后的迟到数据输出到侧输出流
                .sideOutputLateData(outputTag)
                .aggregate(new UrlViewCountAgg(), new UrlViewCountResult());
        result.print("result");
        result.getSideOutput(outputTag).print("late");
        // 为方便观察，可以将原始数据也输出
        stream.print("input");
        env.execute();
    }


    public static class UrlViewCountAgg implements AggregateFunction<Event, Long,
            Long> {
        @Override
        public Long createAccumulator() {
            return 0L;
        }
        @Override
        public Long add(Event value, Long accumulator) {
            return accumulator + 1;
        }
        @Override
        public Long getResult(Long accumulator) {
            return accumulator;
        }
        @Override
        public Long merge(Long a, Long b) {
            return null;
        }

    }


    public static class UrlViewCountResult extends ProcessWindowFunction<Long,
            UrlViewCount, String, TimeWindow> {
        @Override
        public void process(String url, Context context, Iterable<Long> elements,
                            Collector<UrlViewCount> out) throws Exception {
            // 结合窗口信息，包装输出内容
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            out.collect(new UrlViewCount(url, elements.iterator().next(), start,
                    end));
        }
    }
}
```

**输入以下数据：**

```
Alice, ./home, 1000
Alice, ./home, 2000
Alice, ./home, 10000
Alice, ./home, 9000
Alice, ./cart, 12000
Alice, ./prod?id=100, 15000
Alice, ./home, 9000
Alice, ./home, 8000
Alice, ./prod?id=200, 70000
Alice, ./home, 8000
Alice, ./prod?id=300, 72000
Alice, ./home, 8000
```

**运行结果：**

![image-20221120164214773](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120164214773.png)

  下面我们来分析一下程序的运行过程。当输入数据[Alice, ./home, 10000]时，时间戳为10000，由于设置了 2 秒钟的水位线延迟时间，所以此时水位线到达了 8 秒（事实上是 7999毫秒，这里不再追究减 1 的细节），并没有触发 [0, 10s) 窗口的计算；所以接下来时间戳为 9000的数据到来，同样可以直接进入窗口做增量聚合。当时间戳为 12000 的数据到来时（无所谓url 是什么，所有数据都可以推动水位线前进），==水位线到达了 12000 – 2 * 1000 = 10000，所以触发了[0, 10s) 窗口的计算==，第一次输出了窗口统计结果，如下所示：

```
result> UrlViewCount{url='./home,', count=3, windowStart=1970-01-01 08:00:00.0, windowEnd=1970-01-01 08:00:10.0}
```

  这里 count 值为 3，就包括了之前输入的时间戳为 1000、2000、9000 的三条数据。不过窗口触发计算之后并没有关闭销毁，而是继续等待迟到数据。之后时间戳为 15000的数据继续推进水位线，此时时钟已经进展到了 13000ms；此时再来一条时间戳为 9000 的数据，我们会发现立即输出了一条统计结果：

```
result> UrlViewCount{url='./home,', count=4, windowStart=1970-01-01 
08:00:00.0, windowEnd=1970-01-01 08:00:10.0}
```

  很明显，这仍然是[0, 10s) 的窗口，在之前计数值 3 的基础上继续叠加，更新统计结果为4。所以允许窗口处理迟到数据之后，相当于窗口有了一段等待时间，在这期间所有的迟到数据都会立即触发窗口计算，更新之前的结果。

因此，之后时间戳为 8000 的数据到来，同样会立即输出：

```
result> UrlViewCount{url='./home,', count=5, windowStart=1970-01-01 
08:00:00.0, windowEnd=1970-01-01 08:00:10.0}
```

  我们设置窗口等待的时间为 1 分钟，所以当时间推进到 10000 + 60 * 1000 = 70000 时，窗口就会真正被销毁。此前的所有迟到数据可以直接更新窗口的计算结果，而之后的迟到数据已经无法整合进窗口，就只能用侧输出流来捕获了。需要注意的是，这里的“时间”依然是由水位线来指示的，所以时间戳为 70000 的数据到来，并不会触发窗口的销毁；当时间戳为 72000的数据到来，水位线推进到了 72000 – 2 * 1000 = 70000，此时窗口真正销毁关闭，之后再来的迟到数据就会输出到侧输出流了：

```
late> Event{user='Alice,', url='./home,', timestamp=1970-01-01 08:00:08.0}
```

