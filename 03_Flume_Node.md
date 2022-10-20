[TOC]



# Flume学习笔记

# 第一章Flume概述

## 1.1Flume定义

​		Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传 输的系统。Flume 基于流式架构，灵活简单。 

![1658931452374](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658931452374.png)

## 1.2Flume 基础架构

![1658931493708](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658931493708.png)

### 1.2.1 Agent

​	Agent 是一个 JVM 进程，它以事件的形式将数据从源头送至目的。 

​	Agent 主要有 3 个部分组成，Source、Channel、Sink。

### 1.2.2 Source

​		Source 是负责接收数据到 Flume Agent 的组件。Source 组件可以处理各种类型、各种 格式的日志数据，包括 avro、thrift、exec、jms、spooling directory、netcat、taildir、 sequence generator、syslog、http、legacy。



- **Exec source：适用于监控一个实时追加的文件，不能实现断点续传**
- **Spooldir Source ：适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步**
- **Taildir Source：适合用于监听多个实时追加的文件，并且能够实现断点续传。**

### 1.2.3 Sink

​		Sink 不断地轮询 Channel 中的事件且批量地移除它们，并将这些事件批量写入到存储 或索引系统、或者被发送到另一个 Flume Agent。 

​		Sink 组件目的地包括 hdfs、logger、avro、thrift、ipc、file、HBase、solr、自定 义。 

### 1.2.4 Channel

​		Channel 是位于 Source 和 Sink 之间的缓冲区。因此，Channel 允许 Source 和 Sink 运 作在不同的速率上。Channel 是线程安全的，可以同时处理几个 Source 的写入操作和几个 Sink 的读取操作。 



**Memory Channel  ：**是内存中的队列Memory Channel 在不需要关心数据丢失的情景下适用。

​									==因为程序死亡、机器宕机或者重启都会导致数据丢失。==

**File Channel：**将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数 据。 

​		

​	

### 1.2.5 Event

​		传输单元，Flume 数据传输的基本单元，以 Event 的形式将数据从源头送至目的地。 Event 由 **Header** 和 **Body** 两部分组成，Header 用来存放该 event 的一些属性，为 K-V 结构， Body 用来存放该条数据，形式为字节数组。

![1658973601015](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658973601015.png)

# 第二章Flume入门

## 2.1 Flume 安装部署

### 2.1.1 安装地址

（1）Flume 官网地址：http://flume.apache.org/ 

（2）文档查看地址：http://flume.apache.org/FlumeUserGuide.html 

（3）下载地址：http://archive.apache.org/dist/flume/ 

### 2.1.2 安装部署

（1）将 apache-flume-1.9.0-bin.tar.gz 上传到 linux 的/opt/software 目录下

（2）解压 apache-flume-1.9.0-bin.tar.gz 到/opt/module/目录下

```
[atguigu@hadoop102 software]$ tar -zxf /opt/software/apacheflume-1.9.0-bin.tar.gz -C /opt/module/
```

（3）修改 apache-flume-1.9.0-bin 的名称为 flume

```
[atguigu@hadoop102 module]$ mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume
```

（4）将 lib 文件夹下的 guava-11.0.2.jar 删除以兼容 Hadoop 3.1.3

```
[atguigu@hadoop102 lib]$ rm /opt/module/flume/lib/guava- 11.0.2.jar 
```

## 2.2 Flume 入门案例

### 2.2.1 监控端口数据官方案例

**1）案例需求：** 使用 Flume 监听一个端口，收集该端口数据，并打印到控制台。 

**2）需求分析：** 

![1658975293051](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658975293051.png)

**3）实现步骤：**

（1）安装 netcat 工具

```
[atguigu@hadoop102 software]$ sudo yum install -y nc
```

（2）判断 44444 端口是否被占用

```
[atguigu@hadoop102 flume-telnet]$ sudo netstat -nlp | grep 44444 
```

（3）创建 Flume Agent 配置文件 flume-netcat-logger.conf

（4）在 flume 目录下创建 job 文件夹并进入 job 文件夹。

```
[atguigu@hadoop102 flume]$ mkdir job
[atguigu@hadoop102 flume]$ cd job/
```

（5）在 job 文件夹下创建 Flume Agent 配置文件 flume-netcat-logger.conf

```
[atguigu@hadoop102 job]$ vim flume-netcat-logger.conf
```

（6）在 flume-netcat-logger.conf 文件中添加如下内容

```
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

![1658990209413](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658990209413.png)

（7）先开启 flume 监听端口

第一种写法：

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
a1 --conf-file job/flume-netcat-logger.conf -
Dflume.root.logger=INFO,console
```

第二种写法：

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent -c conf/ -n a1 -f  
job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console 
```

（8）使用 netcat 工具向本机的 44444 端口发送内容 

```
[atguigu@hadoop102 ~]$ nc localhost 44444
hello 
atguigu
```

（9）在 Flume 监听页面观察接收数据情况

![1658990347004](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658990347004.png)

### 2.2.2 实时监控单个追加文件

**1）案例需求：** **实时监控 Hive 日志，并上传到 HDFS 中** 

**2）需求分析：** 

![1658988087080](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658988087080.png)

**3）实现步骤：**

**（1）Flume 要想将数据输出到 HDFS，依赖 Hadoop 相关 jar 包**

**（2）创建 flume-file-hdfs.conf 文件**

​		vim flume-file-hdfs.conf

```
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2
# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop102:9820/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs- #是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k2.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 60
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a2.sinks.k2.hdfs.rollCount = 0
# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100
# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name  
a2 --conf-file job/flume-file-hdfs.conf 
```

```
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh 
[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh 
[atguigu@hadoop102 hive]$ bin/hive 
hive (default)> 
```

（3）运行 Flume

（4）开启 Hadoop 和 Hive 并操作 Hive 产生日志

（5）在 HDFS 上查看文件。

### 2.2.3 实时监控目录下多个新文件

**1）案例需求：** **使用 Flume 监听整个目录的文件，并上传至 HDFS** 

**2）需求分析：** 

![1658991916134](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658991916134.png)

**3）实现步骤：**

```
[atguigu@hadoop102 job]$ vim flume-dir-hdfs.conf 
```

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3
# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /opt/module/flume/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
#忽略所有以.tmp 结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)
# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = 
hdfs://hadoop102:9820/flume/upload/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload- #是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是 128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a3.sinks.k3.hdfs.rollCount = 0
# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100
# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

（2）启动监控文件夹命令

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
a3 --conf-file job/flume-dir-hdfs.conf
```

（3）向 upload 文件夹中添加文件

​		在/opt/module/flume 目录下创建 upload 文件夹 

```
[atguigu@hadoop102 flume]$ mkdir upload 
```

​		向 upload 文件夹中添加文件 

```
[atguigu@hadoop102 upload]$ touch atguigu.txt 

[atguigu@hadoop102 upload]$ touch atguigu.tmp 

[atguigu@hadoop102 upload]$ touch atguigu.log 
```

（4）查看 HDFS 上的数据

### 2.2.4 实时监控目录下的多个追加文件

**1）案例需求：** **使用 Flume 监听整个目录的实时追加文件，并上传至 HDFS** 

​		Exec source 适用于监控一个实时追加的文件，不能实现断点续传；Spooldir Source 适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步；而 Taildir Source 适合用于监听多个实时追加的文件，并且能够实现断点续传。 

**2）需求分析：** 

![1658977623362](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658977623362.png)

**3）实现步骤：**

（1）创建配置文件 flume-taildir-hdfs.conf 

​	创建一个文件 

```
[atguigu@hadoop102 job]$ vim flume-taildir-hdfs.conf
```

​	添加如下内容 

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3
# Describe/configure the source
a3.sources.r3.type = TAILDIR
a3.sources.r3.positionFile = /opt/module/flume/tail_dir.json
a3.sources.r3.filegroups = f1 f2
a3.sources.r3.filegroups.f1 = /opt/module/flume/files/.*file.*
a3.sources.r3.filegroups.f2 = /opt/module/flume/files2/.*log.*
# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = 
hdfs://hadoop102:9820/flume/upload2/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a3.sinks.k3.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是 128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a3.sinks.k3.hdfs.rollCount = 0
# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100
# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

(2)启动监控文件夹命令

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
a3 --conf-file job/flume-taildir-hdfs.conf
```

（3）向 files 文件夹中追加内容

在/opt/module/flume 目录下创建 files 文件夹 

```
[atguigu@hadoop102 flume]$ mkdir files
```

向 upload 文件夹中添加文件

```
[atguigu@hadoop102 files]$ echo hello >> file1.txt 
[atguigu@hadoop102 files]$ echo atguigu >> file2.txt 
```

（4）查看 HDFS 上的数据

**Taildir 说明：**  

​		Taildir Source 维护了一个 json 格式的 position File，其会定期的往 position File 中更新每个文件读取到的最新的位置，因此能够实现断点续传。Position File 的格式如下：

```
{"inode":2496272,"pos":12,"file":"/opt/module/flume/files/file1.t xt"} 
{"inode":2496275,"pos":12,"file":"/opt/module/flume/files/file2.t xt"} 
```

# 第三章：调优

## 3.1Flume事物

![1658993294838](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993294838.png)

## 3.2Flume Agent内部原理

![1658993306404](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993306404.png)

**重要组件：** 

**1）ChannelSelector**  

ChannelSelector 的作用就是选出 Event 将要被发往哪个 Channel。其共有两种类型， 

分别是 Replicating（复制）和 Multiplexing（多路复用）。 

ReplicatingSelector 会将同一个 Event 发往所有的 Channel，Multiplexing 会根据相 

应的原则，将不同的 Event 发往不同的 Channel。 

**2）SinkProcessor**  

SinkProcessor 共 有 三 种 类 型 ， 分 别 是 

DefaultSinkProcessor 、 LoadBalancingSinkProcessor 和 FailoverSinkProcessor 

DefaultSinkProcessor 对 应 的 是 单 个 的 Sink ， LoadBalancingSinkProcessor 和 

FailoverSinkProcessor 对应的是 Sink Group，LoadBalancingSinkProcessor 可以实现负 

载均衡的功能，FailoverSinkProcessor 可以错误恢复的功能。

## 3.3Flume拓扑结构

### 3.3.1简单串联

![1658993362015](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993362015.png)

### 3.3.1复制和多路复用

![1658993373518](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993373518.png)



​		Flume 支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个 

channel 中，或者将不同数据分发到不同的 channel 中，sink 可以选择传送到不同的目的 

地。

## 3.4Flume企业开发案例

 



# 第四章Flume面试题

## 4.1 你是如何实现 Flume 数据传输的监控的

使用第三方框架 Ganglia 实时监控 Flume。 

## 4.2 Flume 的 Source，Sink，Channel 的作用？你们 Source 是什么类型？

（1）Source 组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据， 

包括 avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、 http、legacy 

（2）Channel 组件对采集到的数据进行缓存，可以存放在 Memory 或 File 中。 

（3）Sink 组件是用于把数据发送到目的地的组件，目的地包括 Hdfs、Logger、avro、 thrift、ipc、file、Hbase、solr、自定义。 

**2）我公司采用的 Source 类型为：** 

（1）监控后台日志：exec 

（2）监控后台产生日志的端口：netcat 

## 4.3 Flume 的 Channel Selectors

![1658992508443](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658992508443.png)

## 4.4 Flume 参数调优

**1）Source**  

增加 Source 个（使用 Tair Dir Source 时可增加 FileGroups 个数）可以增大 Source 

的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个 

文件目录，同时配置好多个 Source 以保证 Source 有足够的能力获取到新产生的数据。 

batchSize 参数决定 Source 一次批量运输到 Channel 的 event 条数，适当调大这个参 

数可以提高 Source 搬运 Event 到 Channel 时的性能。 

**2）Channel**  

type 选择 memory 时 Channel 的性能最好，但是如果 Flume 进程意外挂掉可能会丢失 

数据。type 选择 file 时 Channel 的容错性更好，但是性能上会比 memory channel 差。 

使用 file Channel 时 dataDirs 配置多个不同盘下的目录可以提高性能。 

Capacity 参数决定 Channel 可容纳最大的 event 条数。transactionCapacity 参数决 

定每次 Source 往 channel 里面写的最大 event 条数和每次 Sink 从 channel 里面读的最大 

event 条数。**transactionCapacity 需要大于 Source 和 Sink 的 batchSize 参数。** 

**3）Sink**  

增加 Sink 的个数可以增加 Sink 消费 event 的能力。Sink 也不是越多越好够用就行，   

过多的 Sink 会占用系统资源，造成系统资源不必要的浪费。 

batchSize 参数决定 Sink 一次批量从 Channel 读取的 event 条数，适当调大这个参数 

可以提高 Sink 从 Channel 搬出 event 的性能。 

## 4.5 Flume 的事务机制

Flume 的事务机制（类似数据库的事务机制）：Flume 使用两个独立的事务分别负责从 

Soucrce 到 Channel，以及从 Channel 到 Sink 的事件传递。 

比如 spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的 

事件全部传递到 Channel 且提交成功，那么 Soucrce 就将该文件标记为完成。 

同理，事务以类似的方式处理从 Channel 到 Sink 的传递过程，如果因为某种原因使得 

事件无法记录，那么事务将会回滚。且所有的事件都会保持到 Channel 中，等待重新传递。

## 4.6 Flume 采集数据会丢失吗?

根据 Flume 的架构原理，Flume 是不可能丢失数据的，其内部有完善的事务机制， 

Source 到 Channel 是事务性的，Channel 到 Sink 是事务性的，因此这两个环节不会出现数 

据的丢失，唯一可能丢失数据的情况是 Channel 采用 memoryChannel，agent 宕机导致数据 

丢失，或者 Channel 存储数据已满，导致 Source 不再写入，未写入的数据丢失。 

Flume 不会丢失数据，但是有可能造成数据的重复，例如数据已经成功由 Sink 发出， 

但是没有接收到响应，Sink 会再次发送数据，此时可能会导致数据的重复。