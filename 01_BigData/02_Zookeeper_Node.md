[TOC]

# Zookeeper学习笔记

# 第一章：Zookeeper入门

## 1.1概述

![1658826504321](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658826504321.png)

![1658826546066](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658826546066.png)

## 1.2 Zookeper的特点

1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。 

2）集群中只要有**半数以上**节点存活，Zookeeper集群就能正常服务。所 以Zookeeper适合安装奇数台服务器。 

3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。 

4）更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。 

5）数据更新原子性，一次数据更新要么成功，要么失败。 

6）实时性，在一定时间范围内，Client能读到最新数据。 



![1658826614299](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658826614299.png)

## 1.3数据结构

​		ZooKeeper 数据模型的结构与 Unix 文件系统很类似，整体上可以看作是一棵树，每个 节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过 其路径唯一标识。 

![1658839917642](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658839917642.png)

## 1.4应用场景

**提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。**

**统一命名服务：**

![1658841048673](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841048673.png)

**统一配置管理**：

![1658841060646](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841060646.png)

**统一集群管理：**

![1658841087017](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841087017.png)



**统一服务器动态上下线：**



![1658841131053](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841131053.png)

**软负载均衡：**

![1658841165810](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841165810.png)

## 1.5下载地址

https://zookeeper.apache.org/ 

下载 Linux 环境安装的 tar 包

![1658841358106](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658841358106.png)

# 第二章：Zookeeper的安装

## 2.1本地模式安装

**(1)安装前准备**

（1）安装 JDK

（2）拷贝 apache-zookeeper-3.5.7-bin.tar.gz 安装包到 Linux 系统下

（3）解压到指定目录

（4）修改名称

**(2)配置修改**

（1）将/opt/module/zookeeper-3.5.7/conf 这个路径下的 zoo_sample.cfg 修改为 zoo.cfg；

（2）打开 zoo.cfg 文件，修改 dataDir 路径： 

   修改如下内容：

（3）在/opt/module/zookeeper-3.5.7/这个目录上创建 zkData 文件夹 

**(3)操作Zookeeper**

（1）启动 Zookeeper

```
bin/zkServer.sh start
```

（2）查看进程是否启动 

```
jps 
```

（3）查看状态

```
 bin/zkServer.sh status
```

（4）启动客户端

```
bin/zkCli.sh
```

（5）退出客户端：

```
quit
```

（6）停止 Zookeeper

```
bin/zkServer.sh stop
```

## 2.2配置参数解读

1）tickTime = 2000：通信心跳时间，Zookeeper服务器与客户端心跳时间，单位毫秒

![1658927896017](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658927896017.png)

2）initLimit = 10：LF初始通信时限

![1658927908698](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658927908698.png)

​		**Leader和Follower初始连接时能容忍的最多心跳数（tickTime的数量）** 

3）syncLimit = 5：LF同步通信时限

![1658927940989](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658927940989.png)

​		**Leader和Follower之间通信时间如果超过syncLimit * tickTime，Leader认为Follwer死**
**掉，从服务器列表中删除Follwer。**

4）dataDir：保存Zookeeper中的数据

​		注意：默认的tmp目录，容易被Linux系统定期删除，所以一般不用默认的tmp目录

5）clientPort = 2181：客户端连接端口，通常不做修改。

# 第三章：集群操作

## 3.1集群操作

### 3.1.1集群安装

**1）集群规划**

在 hadoop102、hadoop103 和 hadoop104 三个节点上都部署 Zookeeper。
思考：如果是 10 台服务器，需要部署多少台 Zookeeper？

**2）解压安装**

**3）配置服务器编号**

（1）在/opt/module/zookeeper-3.5.7/这个目录下创建 zkData

（2）在/opt/module/zookeeper-3.5.7/zkData 目录下创建一个 myid 的文件

（3）拷贝配置好的 zookeeper 到其他机器上

**4）配置zoo.cfg文件**

（1）重命名/opt/module/zookeeper-3.5.7/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg

（2）打开 zoo.cfg 文件

​	修改参数

```
dataDir=/opt/module/zookeeper-3.5.7/zkData

#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

（3）配置参数解读

```
server.A=B:C:D。
```

​	**A** 是一个数字，表示这个是第几号服务器； 

​	集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据 

​	就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比 较从而判断到底是	哪个 server。 

​	**B** 是这个服务器的地址； 

​	**C** 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口； 

​	**D** 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 

​	Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

（4）同步 zoo.cfg 配置文件

**5）集群操作**

（1）分别启动 Zookeeper

```
bin/zkServer.sh start
```

（2）查看状态

```
bin/zkServer.sh status
```

### 3.1.2选举机制

![1658928322089](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658928322089.png)

![1658928340199](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658928340199.png)

### 3.1.3 ZK集群启停脚本

**1）在 hadoop102 的/home/atguigu/bin 目录下创建脚本**

在脚本中编写如下内容

```
#!/bin/bash
case $1 in
"start"){
for i in hadoop102 hadoop103 hadoop104
do
 echo ---------- zookeeper $i 启动 ------------
ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh 
start"
done
};;
"stop"){
for i in hadoop102 hadoop103 hadoop104
do
 echo ---------- zookeeper $i 停止 ------------ 
ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh 
stop"
done
};;
"status"){
for i in hadoop102 hadoop103 hadoop104
do
 echo ---------- zookeeper $i 状态 ------------ 
ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh 
status"
done
};;
esac
```

**2）增加脚本执行权限** 

```
[atguigu@hadoop102 bin]$ chmod u+x zk.sh
```

**3）Zookeeper 集群启动脚本** 

```
[atguigu@hadoop102 module]$ zk.sh start
```

**4）Zookeeper 集群停止脚本** 

```
[atguigu@hadoop102 module]$ zk.sh stop
```

## 3.2客户端命令行操作

### 3.2.1命令行语法

![1658928441090](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658928441090.png)

![1658928453064](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658928453064.png)

**1）启动客户端**

```
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkCli.sh -server
```

**2）显示所有操作命令**

```
[zk: hadoop102:2181(CONNECTED) 1] help 
```

### 3.2.2znode节点数据信息

```
[zk: hadoop102:2181(CONNECTED) 0] ls /    #正常查看
[zk: hadoop102:2181(CONNECTED) 5] ls -s / #详细查看
```

（1）czxid：创建节点的事务 zxid 

每次修改 ZooKeeper 状态都会产生一个 ZooKeeper 事务 ID。事务 ID 是 ZooKeeper 中所 

有修改总的次序。每次修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之 

前发生。 

（2）ctime：znode 被创建的毫秒数（从 1970 年开始） 

（3）mzxid：znode 最后更新的事务 zxid 

（4）mtime：znode 最后修改的毫秒数（从 1970 年开始） 

（5）pZxid：znode 最后更新的子节点 zxid **尚硅谷技术之 Zookeeper**

  

**—————————————————————————————**

  

更多 Java –大数据 –前端 –python 人工智能资料下载，可百度访问：尚硅谷官网 

（6）cversion：znode 子节点变化号，znode 子节点修改次数 

（7）dataversion：znode 数据变化号 

（8）aclVersion：znode 访问控制列表的变化号 

（9）ephemeralOwner：如果是临时节点，这个是 znode 拥有者的 session id。如果不是 

临时节点则是 0。 

（10）dataLength：znode 的数据长度 

（11）numChildren：znode 子节点数量

### 3.2.3节点类型

![1658928634311](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658928634311.png)

**1）分别创建2个普通节点（永久节点 + 不带序号）**

```
[zk: localhost:2181(CONNECTED) 3] create /sanguo "diaochan"
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo "liubei"
Created /sanguo/shuguo
```

**2）获得节点的值**

```
[zk: localhost:2181(CONNECTED) 5] get -s /sanguo
[zk: localhost:2181(CONNECTED) 6] get -s /sanguo/shuguo
```

**3）创建带序号的节点（永久节点 + 带序号）**

（1）先创建一个普通的根节点/sanguo/weiguo

```
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo "caocao"
Created /sanguo/weiguo
```

（2）创建带序号的节点 

```
[zk: localhost:2181(CONNECTED) 2] create -s /sanguo/weiguo/zhangliao "zhangliao" 

Created /sanguo/weiguo/zhangliao0000000000 

[zk: localhost:2181(CONNECTED) 3] create -s  /sanguo/weiguo/zhangliao "zhangliao" 
Created /sanguo/weiguo/zhangliao0000000001 

[zk: localhost:2181(CONNECTED) 4] create -s  /sanguo/weiguo/xuchu "xuchu" 
Created /sanguo/weiguo/xuchu0000000002 
```

**4）创建短暂节点（短暂节点 + 不带序号 or 带序号）**

（1）创建短暂的不带序号的节点

```
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo  "zhouyu"
```

（2）创建短暂的带序号的节点

```
[zk: localhost:2181(CONNECTED) 2] create -e -s /sanguo/wuguo "zhouyu"
```

（3）在当前客户端是能查看到的

```
[zk: localhost:2181(CONNECTED) 3] ls /sanguo
```

（4）退出当前客户端然后再重启客户端

```
[zk: localhost:2181(CONNECTED) 3] ls /sanguo
```

（5）再次查看根目录下短暂节点已经删除

```
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
```

**5）修改节点数据值**

```
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "simayi"
```

### 3.2.4监听器原理

​		客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、节点删除、子目 

录节点增加删除）时，ZooKeeper 会通知客户端。监听机制保证 ZooKeeper 保存的任何的数 

据的任何改变都能快速的响应到监听了该节点的应用程序。 

![1658929014231](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658929014231.png)

**1）节点的值变化监听**

（1）在 hadoop104 主机上注册监听/sanguo 节点数据变化 

```
[zk: localhost:2181(CONNECTED) 26] get -w /sanguo
```

（2）在 hadoop103 主机上修改/sanguo 节点的数据

```
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xisi"
```

（3）观察 hadoop104 主机收到数据变化的监听 

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged 
path:/sanguo
```

​		==注意：在hadoop103再多次修改/sanguo的值，hadoop104上不会再收到监听。因为注册 一次，只能监听一次。想再次监听，需要再次注册。== 

**2）节点的子节点变化监听（路径变化）**

（1）在 hadoop104 主机上注册监听/sanguo 节点的子节点变化

（2）在 hadoop103 主机/sanguo 节点上创建子节点

（3）观察 hadoop104 主机收到子节点变化的监听

==注意：节点的路径变化，也是注册一次，生效一次。想多次生效，就需要多次注册。==

### 3.2.5节点删除与查看

**1）删除节点**

```
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin 
```

**2）递归删除节点**

```
[zk: localhost:2181(CONNECTED) 15] deleteall /sanguo/shuguo
```

**3）查看节点状态**

```
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1
```

## 3.3客户端API操作

前提：保证 hadoop102、hadoop103、hadoop104 服务器上 Zookeeper 集群服务端启动。

### 3.3.1IDEA环境搭建

**1）创建一个工程：zookeeper**

**2）添加pom文件**

```
<dependencies>
<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>RELEASE</version>
</dependency>
<dependency>
<groupId>org.apache.logging.log4j</groupId>
<artifactId>log4j-core</artifactId>
<version>2.8.2</version>
</dependency>
<dependency>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
<version>3.5.7</version>
</dependency>
</dependencies>
```

**3）拷贝log4j.properties文件到项目根目录**

​		需要在项目的 src/main/resources 目录下，新建一个文件，命名为“log4j.properties”，在 

文件中填入。 

```
log4j.rootLogger=INFO, stdout 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] 
%m%n 
log4j.appender.logfile=org.apache.log4j.FileAppender 
log4j.appender.logfile.File=target/spring.log 
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout 
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] 
%m%n
```

**4）创建包名com.atguigu.zk**

**5）创建类名称zkClient**

**5)创建ZooKeeper客户端**

**6)创建子节点**

**7)获取子节点并监听节点变化**

**8)判断Znode是否存在**

**9)客户端向服务端写数据流程**

![1658929839019](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658929839019.png)

![1658929846364](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658929846364.png)

# 第四章：服务器动态上下线监听案例

## 4.1 需求

​		**某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知** 

**到主节点服务器的上下线。**



## 4.2 需求分析

![1658929869742](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658929869742.png)

## 4.3 具体实现

（1）先在集群上创建/servers 节点

（2）在 Idea 中创建包名：com.atguigu.zkcase1 

（3）服务器端向 Zookeeper 注册代码

（4）客户端代码 

## 4.4 测试

1）在 Linux 命令行上操作增加减少服务器 

（1）启动 DistributeClient 客户端 

（2）在 hadoop102 上 zk 的客户端/servers 目录上创建临时带序号节点 

（3）观察 Idea 控制台变化 

（4）执行删除操作

（5）观察 Idea 控制台变化

2）在 Idea 上操作增加减少服务器 

（1）启动 DistributeClient 客户端（如果已经启动过，不需要重启） 

（2）启动 DistributeServer 服务

# 第五章： ZooKeeper 分布式锁案例

## 5.1 原生 Zookeeper 实现分布式锁案例

![1658930010097](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658930010097.png)

## 5.2 Curator 框架实现分布式锁案例

**1）原生的 Java API 开发存在的问题**
（1）会话连接是异步的，需要自己去处理。比如使用 CountDownLatch
（2）Watch 需要重复注册，不然就不能生效
（3）开发的复杂性还是比较高的
（4）不支持多节点删除和创建。需要自己去递归
**2）Curator 是一个专门解决分布式锁的框架，解决了原生 JavaAPI 开发分布式遇到的问题。**
详情请查看官方文档：https://curator.apache.org/index.html
**3）Curator 案例实操**
**（1）添加依赖**

**（2）代码实现**

**（2）观察控制台变化：** 

​	线程 1 获取锁 

​	线程 1 再次获取锁 

​	线程 1 释放锁 

​	线程 1 再次释放锁 

​	线程 2 获取锁 

​	线程 2 再次获取锁 

​	线程 2 释放锁 

​	线程 2 再次释放锁

# 第六章：企业面试题

## 6.1 选举机制

半数机制，超过半数的投票通过，即通过。 

（1）第一次启动选举规则： 

投票过半数时，服务器 id 大的胜出 

（2）第二次启动选举规则： 

​	①EPOCH 大的直接胜出 

​	②EPOCH 相同，事务 id 大的胜出 

​	③事务 id 相同，服务器 id 大的胜出 

## 6.2 生产集群安装多少 zk 合适？

**安装奇数台。** 

生产经验： 

⚫ 10 台服务器：3 台 zk； 

⚫ 20 台服务器：5 台 zk； 

⚫ 100 台服务器：11 台 zk； 

⚫ 200 台服务器：11 台 zk 

服务器台数多：好处，提高可靠性；坏处：提高通信延时

## 6.3 常用命令

ls、get、create、delete