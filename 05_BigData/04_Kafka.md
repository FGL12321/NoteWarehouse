[TOC]



# Kafka学习笔记

# 第一章：Kafka概述

## 1.1定义

​		**Kafka传 统定义：**Kafka是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。

![1658998577076](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658998577076.png)

​		Kafka最 新定义 ： Kafka是 一个开源的 分 布式事件流平台 （Event Streaming Platform），被数千家公司用于高性能数据管道、流分析、数据集成和关键任务应用。



## 1.2消息队列

​		目 前企 业中比 较常 见的 消息 队列产 品主 要有 Kafka、ActiveMQ 、RabbitMQ 、 RocketMQ 等。 

​		在大数据场景主要采用 Kafka 作为消息队列。在 JavaEE 开发中主要采用 ActiveMQ、 RabbitMQ、RocketMQ。 

### 1.2.1传统消息队列的应用场景

传统的消息队列的主要应用场景包括：缓存/消峰、解耦和异步通信。

![1658998592217](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658998592217.png)

![1658998603835](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658998603835.png)

![1658998638265](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658998638265.png)

### 1.2.2消息队列的两种模式

![1658998689856](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658998689856.png)

## 1.3Kafka基础架构

![1658999102007](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658999102007.png)

（1）Producer：消息生产者，就是向 Kafka broker 发消息的客户端。 

（2）Consumer：消息消费者，向 Kafka broker 取消息的客户端。

 （3）Consumer Group（CG）：消费者组，由多个 consumer 组成。消费者组内每个消
费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不
影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。 

（4）Broker：一台 Kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个
broker 可以容纳多个 topic。 

（5）Topic：可以理解为一个队列，生产者和消费者面向的都是一个 topic。

 （6）Partition：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服
务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列。 

（7）Replica：副本。一个 topic 的每个分区都有若干个副本，一个 Leader 和若干个
Follower。

 （8）Leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数
据的对象都是 Leader。 

（9）Follower：每个分区多个副本中的“从”，实时从 Leader 中同步数据，保持和
Leader 数据的同步。Leader 发生故障时，某个 Follower 会成为新的 Leader。

# 第二章：Kafka快速入门

## 2.1安装部署

### 2.1.1集群规划

![1659077784684](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659077784684.png)

### 2.1.2集群部署

0）官方下载地址：http://kafka.apache.org/downloads.html

1）解压安装包并修改名称

2）进入到/opt/module/kafka 目录，修改配置文件

```
#broker 的全局唯一编号，不能重复，只能是数字。
broker.id=0

#kafka 运行日志(数据)存放的路径，路径不需要提前创建，kafka 自动帮你创建，可以
配置多个磁盘路径，路径与路径之间可以用"，"分隔
log.dirs=/opt/module/kafka/datas

#配置连接 Zookeeper 集群地址（在 zk 根目录下创建/kafka，方便管理）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/ka
fka
```

3）分发安装包

4）分别在 hadoop103 和 hadoop104 上修改配置文件/opt/module/kafka/config/server.properties
中的 broker.id=1、broker.id=2

5）配置环境变量并进行分发和刷新

```
[atguigu@hadoop102 module]$ sudo vim /etc/profile.d/my_env.sh

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin

[atguigu@hadoop102 module]$ source /etc/profile
```

（6）启动集群

先启动Zookeeper集群再启动Kafka集群

依次在个虚拟机上启动

```
[atguigu@hadoop102 kafka]$ zk.sh start
```

### 2.1.3集群启停脚本

1）在/home/atguigu/bin 目录下创建文件 kf.sh 脚本文件

```
[atguigu@hadoop102 bin]$ vim kf.sh
```

```
#! /bin/bash
case $1 in
"start"){
 for i in hadoop102 hadoop103 hadoop104
 do
 echo " --------启动 $i Kafka-------"
 ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -
daemon /opt/module/kafka/config/server.properties"
 done
};;
"stop"){
 for i in hadoop102 hadoop103 hadoop104
 do
 echo " --------停止 $i Kafka-------"
 ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh "
 done
};;
esac
```

（2）添加执行权限

```
[atguigu@hadoop102 bin]$ chmod +x kf.sh
```

（3）启动集群命令

```
[atguigu@hadoop102 ~]$ kf.sh start
```

（4）停止

```
[atguigu@hadoop102 ~]$ kf.sh stop
```

==**注意：**停止 Kafka 集群时，一定要等 Kafka 所有节点进程全部停止后再停止 Zookeeper 集群。因为 Zookeeper 集群当中记录着 Kafka 集群相关信息，Zookeeper 集群一旦先停止，Kafka 集群就没有办法再获取停止进程的信息，只能手动杀死 Kafka 进程了。==

## 2.2Kafka命令行操作

### 2.2.1主题命令行操作

（1）查看操作主题命令参数

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh
```

![1659078301393](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659078301393.png)

（2）查看当前服务器中的所有topic

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --list
```

（3）创建first topic

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 1 --replication-factor 3 --topic first 
```

选项说明：
--topic 定义 topic 名
--replication-factor 定义副本数
--partitions 定义分区数

### 2.2.2生成者命令行操作

1）查看操作生产者命令参数

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-producer.sh
```

![1659079228585](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659079228585.png)

2）发送消息

>```
>[atguigu@hadoop102 kafka]$ bin/kafka-console-producer.sh --
>bootstrap-server hadoop102:9092 --topic first
>>hello world
>>atguigu atguigu
>```

### 2.2.3消费者命令行操作

**1）查看操作消费者命令参数**

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh
```

![1659079273484](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659079273484.png)

![1659079333121](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659079333121.png)

**2）消费消息**

（1）消费 first 主题中的数据

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first
```

（2）把主题中所有的数据都读取出来（包括历史数据）

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh -- bootstrap-server hadoop102:9092 --from-beginning --topic first
```

# 第三章：Kafka生产者

## 3.1生产者消息发送流程

### 3.1.1 发送原理

​		在消息发送的过程中，涉及到了两个线程——main 线程和 Sender 线程。在 main 线程中创建了一个双端队列 RecordAccumulator。main 线程将消息发送给 RecordAccumulator，Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker。

![1659099842289](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659099842289.png)

###  3.1.2 生产者重要参数列表

1）需求：创建 Kafka 生产者，采用异步的方式发送到 Kafka Broker

2）代码编写

```
<dependencies>
 <dependency>
 <groupId>org.apache.kafka</groupId>
 <artifactId>kafka-clients</artifactId>
 <version>3.0.0</version>
 </dependency>
</dependencies>
```

```java
package com.atguigu.kafka.producer;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;
public class CustomProducer {
 public static void main(String[] args) throws 
InterruptedException {
 // 1. 创建 kafka 生产者的配置对象
 Properties properties = new Properties();
 // 2. 给 kafka 配置对象添加配置信息：bootstrap.servers
 properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, 
"hadoop102:9092");
 
 // key,value 序列化（必须）：key.serializer，value.serializer
 properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
 
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
"org.apache.kafka.common.serialization.StringSerializer");
 // 3. 创建 kafka 生产者对象
 KafkaProducer<String, String> kafkaProducer = new 
KafkaProducer<String, String>(properties);
 // 4. 调用 send 方法,发送消息
 for (int i = 0; i < 5; i++) {
 kafkaProducer.send(new 
ProducerRecord<>("first","atguigu " + i));
 }
 // 5. 关闭资源
 kafkaProducer.close();
 }
} 
```



## 3.2 异步发送 API



### 3.2.2 带回调函数的异步发送

​		回调函数会在 producer 收到 ack 时调用，为异步调用，该方法有两个参数，分别是元数据信息（RecordMetadata）和异常信息（Exception），如果 Exception 为 null，说明消息发送成功，如果 Exception 不为 null，说明消息发送失败。

## 3.3 同步发送 API

==只需在异步发送的基础上，再调用一下 get()方法即可。==

## 3.4 生产者分区

### 3.4.1 分区好处

![1659100007214](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100007214.png)

### 3.4.2 生产者发送消息的分区策略

![1659100027666](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100027666.png)

### 3.4.3 自定义分区器



## 3.5 生产经验——生产者如何提高吞吐量

![1659100108083](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100108083.png)

```java
// batch.size：批次大小，默认 16K
 properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
 // linger.ms：等待时间，默认 0
 properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);
 // RecordAccumulator：缓冲区大小，默认 32M：buffer.memory
 properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG,
 33554432);
 // compression.type：压缩，默认 none，可配置值 gzip、snappy、lz4 和 zstd
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"snappy");
```

## 3.6 生产经验——数据可靠性

![1659100166739](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100166739.png)

![1659100177069](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100177069.png)

![1659100186501](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100186501.png)

```
// 设置 acks
 properties.put(ProducerConfig.ACKS_CONFIG, "all");
 // 重试次数 retries，默认是 int 最大值，2147483647
 properties.put(ProducerConfig.RETRIES_CONFIG, 3);
```



## 3.7 生产经验——数据去重

### 3.7.1 数据传递语义

![1659100236005](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100236005.png)

### 3.7.2 幂等性

![1659100244840](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100244840.png)

开启参数 enable.idempotence 默认为 true，false 关闭。

### 3.7.3 生产者事务

```java
package com.fang.kafka.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class CustomProducerTrana {
    public static void main(String[] args) {

        //0 配置
        Properties properties = new Properties();

        //连接集群
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"hadoop102:9092,hadoop103:9092");

        //指定对应的value和key
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());

        //指定事物id
        properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG,"tranactionnal_01");

        //1.创建Kafka生产者对象
        KafkaProducer<String, String> KafkaProducer = new KafkaProducer<String, String>(properties);

        KafkaProducer.initTransactions();

        KafkaProducer.beginTransaction();

        try {
            //2.发送数据
            for (int i = 0; i <5 ; i++) {
                KafkaProducer.send(new ProducerRecord<String, String>("first","fang"+i));
            }
            KafkaProducer.commitTransaction();

        }catch (Exception e){
          KafkaProducer.abortTransaction();
        }finally {
            //3.关闭资源
            KafkaProducer.close();
        }
    }
}

```



## 3.8 生产经验——数据有序

![1659100278206](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100278206.png)

## 3.9 生产经验——数据乱序

**1）kafka在1.x版本之前保证数据单分区有序，条件如下：**
max.in.flight.requests.per.connection=1（不需要考虑是否开启幂等性）。 

**2）kafka在1.x及以后版本保证数据单分区有序，条件如下：**
（2）未启幂等性
max.in.flight.requests.per.connection需要设置小于等于5。 

（1）开启幂等性
max.in.flight.requests.per.connection需要设置为1。
原因说明：因为在kafka1.x以后，启用幂等后，kafka服务端会缓存producer发来的最近5个request的元数据，
故无论如何，都可以保证最近5个request的数据都是有序的。

![1659100961486](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659100961486.png)

# 第四章：Kafka Broker

## 4.1 Kafka Broker 工作流程

### 4.1.1 Zookeeper 存储的 Kafka 信息

![1659181547659](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181547659.png)

### 4.1.2 Kafka Broker 总体工作流程

![1659181564783](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181564783.png)

### 4.1.3 Broker 重要参数

![1659181595186](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181595186.png)



![1659181606376](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181606376.png)

## 4.2 生产经验——节点服役和退役

### 4.2.1 服役新节点

### 4.2.2 退役旧节点

## 4.3 Kafka 副本

### 4.3.1 副本基本信息

（1）Kafka 副本作用：提高数据可靠性。 （2）Kafka 默认副本 1 个，生产环境一般配置为 2 个，保证数据可靠性；太多副本会
增加磁盘存储空间，增加网络上数据传输，降低效率。
（3）Kafka 中副本分为：Leader 和 Follower。Kafka 生产者只会把数据发往 Leader，
然后 Follower 找 Leader 进行同步数据。
（4）Kafka 分区中的所有副本统称为 AR（Assigned Repllicas）。
AR = ISR + OSR
ISR，表示和 Leader 保持同步的 Follower 集合。如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值由 **replica.lag.time.max.ms** 参数设定，默认 30s。Leader 发生故障之后，就会从 ISR 中选举新的 Leader。 OSR，表示 Follower 与 Leader 副本同步时，延迟过多的副本。 

### 4.3.2 Leader 选举流程

​		Kafka 集群中有一个 broker 的 Controller 会被选举为 Controller Leader，负责管理集群broker 的上下线，所有 topic 的分区副本分配和 Leader 选举等工作。

![1659181678765](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181678765.png)

### 4.3.3 Leader 和 Follower 故障处理细节

![1659181695782](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181695782.png)

![1659181706416](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181706416.png)

### 4.3.4 分区副本分配

![1659181739446](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181739446.png)

### 4.3.5 生产经验——手动调整分区副本存储

![1659181752978](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181752978.png)

### 4.3.6 生产经验——Leader Partition 负载平衡

![1659181770110](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181770110.png)

## 4.4 文件存储

### 4.4.1 文件存储机制

![1659181795586](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181795586.png)

### 4.4.2 文件清理策略

![1659181813058](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181813058.png)

## 4.5 高效读写数据

![1659181829810](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181829810.png)

4）页缓存 + 零拷贝技术

![1659181850549](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181850549.png)

# 第五章：Kafka消费者

## 5.1 Kafka 消费方式

![1659181883032](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181883032.png)



## 5.2 Kafka 消费者工作流程

### 5.2.1 消费者总体工作流程

![1659181896541](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181896541.png)



### 5.2.2 消费者组原理

![1659181905969](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181905969.png)

![1659181950252](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181950252.png)

![1659181958806](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659181958806.png)

## 5.3消费者API

### 5.3.1 独立消费者案例（订阅主题）

// 1.创建消费者的配置对象

// 2.给消费者配置对象添加参数 

// 配置序列化 必须

// 配置消费者组（组名任意起名） 必须

// 创建消费者对象 

// 注册要消费的主题（可以消费多个主题） 

// 拉取数据打印 

### 5.3.2 独立消费者案例（订阅分区）

// 1.创建消费者的配置对象

// 2.给消费者配置对象添加参数 

// 配置序列化 必须

// 配置消费者组（组名任意起名） 必须

==// 消费某个主题的某个分区数据==

// 注册要消费的主题（可以消费多个主题） 

// 拉取数据打印 

### 5.3.3 消费者组案例

![1659182369849](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182369849.png)

## 5.4 生产经验——分区的分配以及再平衡

![1659182388332](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182388332.png)

### 5.4.1 Range 以及再平衡

![1659182415589](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182415589.png)

### 5.4.2 RoundRobin 以及再平衡

![1659182431110](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182431110.png)

### 5.4.3 Sticky 以及再平衡

​		**粘性分区定义：**可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前， 考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。 

​		粘性分区是 Kafka 从 0.11.x 版本开始引入这种分配策略，首先会尽量均衡的放置分区 到消费者上面，在出现同一消费者组内消费者出现问题的时候，会尽量保持原有分配的分 区不变化。 

## 5.5 offset 位移

### 5.5.1 offset 的默认维护位置

![1659182469747](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182469747.png)

### 5.5.2 自动提交 offset

![1659182484991](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182484991.png)

### 5.5.3 手动提交 offset

![1659182496221](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182496221.png)

### 5.5.4 指定 Offset 消费

![1659182517233](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182517233.png)

### 5.5.5 指定时间消费



### 5.5.6 漏消费和重复消费

![1659182532917](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182532917.png)

## 5.6 生产经验——消费者事务

![1659182543133](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182543133.png)

## 5.7 生产经验——数据积压（消费者如何提高吞吐量）

![1659182555180](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182555180.png)

# 第六章：Kafka-Kraft模式

![1659182171414](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659182171414.png)

​		左图为 Kafka 现有架构，元数据在 zookeeper 中，运行时动态选举 controller，由controller 进行 Kafka 集群管理。右图为 kraft 模式架构（实验性），不再依赖 zookeeper 集群，而是用三台 controller 节点代替 zookeeper，元数据保存在 controller 中，由 controller 直接进行 Kafka 集群管理。
这样做的好处有以下几个：

 ⚫ Kafka 不再依赖外部框架，而是能够独立运行； 

⚫ controller 管理集群时，不再需要从 zookeeper 中先读取数据，集群性能上升；

 ⚫ 由于不依赖 zookeeper，集群扩展时不再受到 zookeeper 读写能力限制；

 ⚫ controller 不再动态选举，而是由配置文件规定。这样我们可以有针对性的加强
		controller 节点的配置，而不是像以前一样对随机 controller 节点的高负载束手无策。