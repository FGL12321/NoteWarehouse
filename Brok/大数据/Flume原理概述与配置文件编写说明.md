[TOC]



# Flume原理概述与配置文件编写说明

# 1.0Flume定义

​		<font color="red"> **Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传 输的系统。Flume 基于流式架构，灵活简单。**</font>

![1658931452374](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658931452374.png)

# 2.0Flume架构

![1658931493708](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658931493708.png)

## 2.1Agent

​	Agent 是一个 JVM 进程，它以事件的形式将数据从源头送至目的。 

​	Agent 主要有 3 个部分组成，**Source、Channel、Sink**。

## 2.2 Source

​		Source 是负责接收数据到 Flume Agent 的组件。Source 组件可以处理各种类型、**各种 格式的日志数据**，包括 **avro、thrift、exec、jms、spooling directory、netcat、taildir、 sequence generator、syslog、http、legacy。**

- **Exec source：适用于监控一个实时追加的文件，不能实现断点续传**
- **Spooldir Source ：适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步**
- **Taildir Source：适合用于监听多个实时追加的文件，并且能够实现断点续传。**

## 2.3Sink

​		Sink **不断地轮询 Channel 中的事件且批量地移除它们**，并将这些事件**批量写入到存储** 或索引系统、或者被发送到另一个 Flume Agent。 

​		Sink 组件目的地包括 hdfs、logger、avro、thrift、ipc、file、HBase、solr、自定 义。 

## 2.4 Channel

​		Channel 是位于 **Source 和 Sink 之间的缓冲区**。因此，Channel 允许 Source 和 Sink 运 作在不同的速率上。Channel 是线程安全的，可以同时处理几个 Source 的写入操作和几个 Sink 的读取操作。 

**Memory Channel  ：**是内存中的队列Memory Channel 在不需要关心数据丢失的情景下适用。

​									==因为程序死亡、机器宕机或者重启都会导致数据丢失。==

**File Channel：**将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数 据。 

## 2.5 Event

​		传输单元，Flume 数据传输的基本单元，以 Event 的形式将数据从源头送至目的地。 Event 由 **Header** 和 **Body** 两部分组成，Header 用来存放该 event 的一些属性，为 K-V 结构， Body 用来存放该条数据，形式为字节数组。

![1658973601015](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658973601015.png)

# 3.0事物处理与拓扑结构简介

## 3.1Flume事物处理





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

- DefaultSinkProcessor 、 LoadBalancingSinkProcessor 和 FailoverSinkProcessor 

- DefaultSinkProcessor 对 应 的 是 单 个 的 Sink ， LoadBalancingSinkProcessor 和 

- FailoverSinkProcessor 对应的是 Sink Group，LoadBalancingSinkProcessor 可以实现负 


载均衡的功能，FailoverSinkProcessor 可以错误恢复的功能。

### 3.3.1简单串联

![1658993362015](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993362015.png)

### 3.3.1复制和多路复用

![1658993373518](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658993373518.png)



​		Flume 支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个 channel 中，或者将不同数据分发到不同的 channel 中，sink 可以选择传送到不同的目的地。

# 4.0配置文件的编写

## 4.1基本的配置文件编写

```python
添加内容如下：
# Name the components on this agent #a1：表示agent的名称
a1.sources = r1   #r1:表示a1的Source的名称
a1.sinks = k1     #k1：表示Sink的名称
a1.channels = c1  #c1：表示a1的Channel的名称

# Describe/configure the source  
a1.sources.r1.type = netcat  #表示a1的输入源类型为netcat端口名称
a1.sources.r1.bind = localhost #表示a1的监听的主机
a1.sources.r1.port = 44444     #表示a1的监听的端口号

# Describe the sink        
a1.sinks.k1.type = logger      #表示a1的输出目的是控制台logger类型

# Use a channel which buffers events in memory
a1.channels.c1.type = memory                #表示a1的channel类型是memory内存型
a1.channels.c1.capacity = 1000              #表示a1的channel总容量为1000个event
a1.channels.c1.transactionCapacity = 100    #表示a1的channel传输时收集100条再提交

# Bind the source and sink to the channel
a1.sources.r1.channels = c1      #表示将r1与c1连接起来
a1.sinks.k1.channel = c1         #表示k1和c1连接起来
```

## 4.2进阶编写

```python
#控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

#控制输出文件是原生文件。
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = lzop

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
```

