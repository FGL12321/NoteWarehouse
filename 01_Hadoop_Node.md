

[TOC]



# hadoop

## 1.hadoop的组成（前置知识了解）

### 1.1MapReduce（计算）

**一个完整的MapReduce程序在分布式运行时有三类实例进程：**

**（1）MrAppMaster：负责整个程序的过程调度及状态协调。**

**（2）MapTask：负责Map阶段的整个数据处理流程。**

**（3）ReduceTask：负责Reduce阶段的整个数据处理流程。**

### 1.2Yam（资源调度）

**1.ResourceManger：整个集群的所有资源**

**2.NodeManger：单个节点的服务器资源==可以有多个Container==**

**3.ApplicationMaster：单个任务线程**

**4.Containter：相当于一台独立的服务器**

### 1.3HDFS（数据存储）

**1.NameNode：相当于文件的目录**

**2.DataNode：存储具体的文件内容**

**3.NameNode：每隔一段时间对NameNode进行备份**

**4.Client客户端**

### 1.4Cormmon（辅助工具）

### 1.5HDFS，YARN，MapReduce三者之间的关系

![1657634333264](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657634333264.png)

## 2.hadoop集群搭建流程

#### 完全分布式运行模式配置

#### 虚拟机准备阶段

1. 安装VMware
2. 安装CenterOS并进行配置hadoop100
3. 克隆出3台客户机hadoop102 103 104（关闭防火墙，配置静态IP，配置主机名称） 
4. 安装远程终端访问工具
5. hadop102安装JDK 并配置环境变量
6. hadop102安装Hadoop 并配置环境变量

#### hadoop配置阶段

:one:配置xsync

:two:配置SSH

:three:配置HDFS和YARN

:four:群起并测试集群

:five:配置历史服务器

:six:配置日志聚集



#### 1.CentrtOS的安装与模板虚拟机的配置（准备阶段）

1. ​    配置电脑
2. ​    安装CentOS系统
3. ​    修改VMware的网络配置 
4. ​    修改Windows的网络配置
5. ​    修改CenterOS的网络配置
6. ​    修改主机名和hosts文件
7. ​    安装终端访问工具
8. ​    ==7步==对hadoop100的详细配置（模板虚拟机配置完成）
9. ​    克隆虚拟机hadoop102  hadoop103  hadoop104
10. ​    分别修改克隆机的网络配置
11. ​    在hadoop102上安装JDK和Hadoop 

```
第1章 VMware]
    [1.1 VMware安装]
        [1.1.1 开始安装]
        [1.2.2 欢迎界面]
        [1.2.3 同意许可证]
        [1.2.4 选择安装路径]
        [1.2.5 用户体检计划]
        [1.2.6 快捷方式]
        [1.2.7 开始安装]
        [1.2.8 等待安装完成]
        [1.2.9 安装完成]
        [1.2.10 输入许可证]
        [1.2.11 VMware安装完毕]

[第2章 CentOS]
    [2.1 配置电脑]
        [2.1.1 进入VMware]
        [2.1.2 自定义新的虚拟机]
        [2.1.3 解决虚拟机的兼容性]
        [2.1.4 选择当前虚拟机的操作系统	]
        [2.1.5 选择虚拟机将来需要安装的系统]
        [2.1.6 配置电脑]
        [2.1.7 选择CPU的个数]
        [2.1.8 设置虚拟机的内存]
        [2.1.9 选择虚拟机上网方式]
        [2.1.10 选择对应的文件系统的IO方式]
        [2.1.11 选择磁盘的类型]
        [2.1.12 选择磁盘的种类]
        [2.1.13 选择虚拟机的磁盘大小]
        [1.1.14 虚拟机文件的存放位置]
        [2.1.14 电脑配置完毕]

    [2.2 安装系统（CentOS7）]
        [2.2.1 选择cd/dvd的方式安装系统]
        [2.2.2 系统安装引导界面]
        [2.2.3 需要定制化的内容]
        [2.2.4 虚拟机的使用引导界面]
        [2.2.5 切换root用户]

    [2.3 网络配置]
        [2.3.1 编辑VMware的网络配置]
        [2.3.2 Windows的网络配置]

    [2.4 虚拟机网络IP修改地址配置]

    [2.5 修改主机名和hosts文件]

[第3章 远程终端工具安装]
    [3.1 Xshell5安装和配置]
    [3.2 Xftp传输工具]
```



#### 2.配置集群

**1.scp（secure copy）安全拷贝(准备阶段)**

​    rsync远程同步工具(准备阶段)

​    xsync集群分发脚本(准备阶段)==选择xsync==



**2.SSH无密登录配置(准备阶段)**



**3.配置HDFS和YARN**

![1657725907070](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657725907070.png)

​       3.1核心配置文件    core-site.xml

​       3.2HDFS配置文件  hdfs-site.xml

​       3.3YARN配置文件  yarn-site.xml

​       3.4MapReduce配置文件 mapred-site.xml

![完全分布式搭建流程](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83%E5%BC%8F%E6%90%AD%E5%BB%BA%E6%B5%81%E7%A8%8B.jpg)

**4.启动集群**

```
整体启动：
整体启动/停止HDFS：start-dfs.sh/stop-dfs.sh
整体启动/停止YARN：start-yarn.sh/stop-yarn.sh

各个服务组件逐一启动/停止：
hdfs --daemon start/stop namenode/datanode/secondarynamenode
yarn --daemon start/stop  resourcemanager/nodemanager

启动历史服务器hadoop102：mapred --daemon start historyserver
关闭历史服务器hadoop102: mapred --daemon stop historyserver

脚本启动：myhadoop.sh start
脚本关闭：myhadoop.sh stop
```

查看HDFS：http://hadoop102:9870

查看YARN：http://hadoop103:8088

查看JobHistory：http://hadoop102:19888/jobhistory



**5.测试集群**

```
1.上传文件到集群：源文件地址 目标地址
[atguigu@hadoop102 ~]$hadoop fs -mkdir /input
[atguigu@hadoop102 ~]$hadoop fs -put $HADOOP_HOME/wcinput/word.txt /input

2.上传大文件：
[atguigu@hadoop102 ~]$ hadoop fs -put  /opt/software/jdk-8u212-linux-x64.tar.gz  /

3.HDFS文件的存储路径:
[atguigu@hadoop102 subdir0]$ pwd
/opt/module/hadoop-3.1.3/data/dfs/data/current/BP-1436128598-192.168.10.102-1610603650062/current/finalized/subdir0/subdir0

4.执行wordcount程序：
[atguigu@hadoop102 hadoop-3.1.3]$hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output

5.下载程序：
[atguigu@hadoop104 software]$ hadoop fs -get /jdk-8u212-linux-x64.tar.gz ./

```



**6.配置历史服务器**

配置mapred-site.xml



**7.配置日志聚集**

配置yarn-site.xml



## 3.Linux基本操作

```
1.切换用户
    su 用户名


2.切换目录
    cd 目录名
    cd ..     返回上一级目录
    cd ../..  返回上两级目录
    cd /      返回根目录


3.查看当前目录下有哪些文件以及详细信息
    ls 
    ls 文件名

4.删除文件
    rm -f 
    rm -R dir dirname（删除dirname目录下的所有东西）


4.vim编辑器的使用
      1.分为命令模式：
         1. i 切换到输入模式，以输入字符。
         2. x 删除当前光标所在处的字符。
         3. : 切换到底线命令模式，以在最底一行输入命令
      2.输入模式
            字符按键以及Shift组合，输入字符
            ENTER，回车键，换行
            BACK SPACE，退格键，删除光标前一个字符
            DEL，删除键，删除光标后一个字符
            方向键，在文本中移动光标
            HOME/END，移动光标到行首/行尾
            Page Up/Page Down，上/下翻页
            Insert，切换光标为输入/替换模式，光标将变成竖线/下划线
            ESC，退出输入模式，切换到命令模式
      3.底线命令模式 
            1.:wq保存并退出
            2. q 退出程序
            3. w 保存文件
      4.新建一个文件
            vim test.txt
            
hadoop102 103 104            
root fgl123
fang fgl15706021741

hdaoop100
fang leilei.123
root fgl123

```

## 4.HDFS

### 1.概述

**1.产生背景**

- HDFS只是**分布式文件管理系统**中的一种

**2.优缺点**

- 高容错性
- 适合处理大数据
- 不适合低延时数据访问
- 无法高效对大量小文件进行存储
- 不支持并发写入，文件随机修改

**3.HDFS组成部分**

1. NameNode：相当于文件的目录
2. DataNode：存储具体的文件内容

3. NameNode：每隔一段时间对NameNode进行备份
4. Client客户端用于文件切分和对数据的交互，管理HDFS

**4.HDFS文件块大小**

 **1.默认值**

- ​	块的大小可以通过配置参数来规定dfs.blocksize
- ​	默认大小为128M（大厂256M）
- ​    如果有1kb文件存储，块的其他空间依然可以被分配
- ​    寻址时间：为传输时间的1%为最佳，从而确定块的大小
- ​    传输时间

### 2.HDFS的Shell相关操作（开发重点）

hadoop fs具体命令和hdfs dfs具体命令完成相同

#### **1.hdfs增删改查系列**

```
hdfs dfs -mkdir dir  创建文件夹

hdfs dfs -mkdir -p dir 递归创建文件夹

hdfs dfs -rmr dir  删除文件夹dir

hdfs dfs -ls  查看目录文件信息

hdfs dfs -lsr  递归查看文件目录信息

hdfs dfs -stat path 返回指定路径的信息
```

#### **2.hdfs上传**

```
1.-moveFromLocal：从本地剪切粘贴到HDFS
	hadoop fs  -moveFromLocal  ./shuguo.txt  /sanguo
2.-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去
	hadoop fs -copyFromLocal weiguo.txt /sanguo
3.-put：等同于copyFromLocal，生产环境更习惯用put
	hadoop fs -put ./wuguo.txt /sanguo
4.-appendToFile：追加一个文件到已经存在的文件末尾
	hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt
```

#### **3.hdfs下载**

```
1.-copyToLocal：从HDFS拷贝到本地
	hadoop fs -copyToLocal /sanguo/shuguo.txt ./
2.-get：等同于copyToLocal，生产环境更习惯用get
	hadoop fs -get /sanguo/shuguo.txt ./shuguo2.txt
```

#### **4.hdfs直接操作**

```
1.-ls: 显示目录信息
	hadoop fs -ls /sanguo
2.-cat：显示文件内容
	hadoop fs -cat /sanguo/shuguo.txt
3.-chgrp、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限
	hadoop fs  -chmod 666  /sanguo/shuguo.txt
	hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo.txt
4.-mkdir：创建路径
	hadoop fs -mkdir /jinguo
5.-cp：从HDFS的一个路径拷贝到HDFS的另一个路径
	hadoop fs -cp /sanguo/shuguo.txt /jinguo
6.-mv：在HDFS目录中移动文件
	 hadoop fs -mv /sanguo/wuguo.txt /jinguo
	 
7.-tail：显示一个文件的末尾1kb的数据
	hadoop fs -tail /jinguo/shuguo.txt
8.-rm：删除文件或文件夹
	hadoop fs -rm /sanguo/shuguo.txt
9.-rm -r：递归删除目录及目录里面内容
	hadoop fs -rm -r /sanguo
10.-du统计文件夹的大小信息
	hadoop fs -du -s -h /jinguo
```

### 3.HDFS的客户端API（Java代码实现，用处不多）

**使用Windows系统连接集群完成对HDFS的客户端代码操作**

**在IDEA中进行实现：**

* 客户端代码常用套路

* 1.获取同一个客户端对象

  - 连接集群地址

    - ```
      URI uri = new URI("hdfs://hadoop102:8020");
      ```

  - 创建一个配置文件用于设置副本个数

    - ```
      Configuration configuration = new Configuration();configuration.set("dfs.replication","3");
      ```

  - 指出用户

    - ```
      String user="fang";
      ```

  - 获取到客户端对象

    - ```
      fileSystem = FileSystem.get(uri,configuration,user);
      ```

* 2.执行相关操作命令

  * 创建目录
  * 上传操作
    * 副本个数 hads-default.xml=>hdfs-site.xml=>在项目配置文件下优先级高=>代码中的配置更高
  * 文件下载
  * 文件的更名和移动
  * 获取文件详细信息
  * 判断是文件夹还是文件

* 3.关闭资源

  * ```
    fileSystem.close();
    ```

### 4.HDFS的读写流程（面试重点）

#### 4.1HDFS写数据流程

**1.剖析文件写入**

![1657784921827](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657784921827.png)



![1657785106993](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657785106993.png)

```
（1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。

（2）NameNode返回是否可以上传。

（3）客户端请求第一个 Block上传到哪几个DataNode服务器上。

（4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。

（5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。

（6）dn1、dn2、dn3逐级应答客户端。

（7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。

（8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。
```

**2.网络拓扑-节点距离计算**

==节点距离：两个节点到达最近的共同祖先的距离总和。==

![1657785665135](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657785665135.png)



**3.机架感知（副本存储节点选择）**

选择副本所在的节点位置

![1657785859369](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657785859369.png)



#### 4.2HDFS读数据流程

![1657785941517](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657785941517.png)

```
（1）客户端通过DistributedFileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。

（2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。

（3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。

（4）客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。
```

### 5.NN和2NN（了解）

#### 5.1.工作机制

![1657865652984](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657865652984.png)



```
1）第一阶段：NameNode启动
    （1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
    （2）客户端对元数据进行增删改的请求。
    （3）NameNode记录操作日志，更新滚动日志。
    （4）NameNode在内存中对元数据进行增删改。

2）第二阶段：Secondary NameNode工作
    （1）Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。
    （2）Secondary NameNode请求执行CheckPoint。
    （3）NameNode滚动正在写的Edits日志。
    （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
    （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
    （6）生成新的镜像文件fsimage.chkpoint。
    （7）拷贝fsimage.chkpoint到NameNode。
    （8）NameNode将fsimage.chkpoint重新命名成fsimage。
```

#### 5.2Fsimage和Edits概念

![1657865926638](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657865926638.png)

#### 5.3oiv和oev分别查看Fsimage文件和Edits文件

**1.oiv**

```
[atguigu@hadoop102 current]$ hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml
[atguigu@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/fsimage.xml
```

**2.oev**

```
[atguigu@hadoop102 current]$ hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml
[atguigu@hadoop102 current]$ cat /opt/module/hadoop-3.1.3/edits.xml
```

#### 5.4CheckPoint时间设置

1）通常情况下，SecondaryNameNode每隔一小时执行一次。

​	[hdfs-default.xml]

```html
<property>
 <name>dfs.namenode.checkpoint.period</name>
 <value>3600s</value>
</property>
```

2.一分钟检查一次操作次数，当操作次数达到1百万执行，SecondaryNameNode执行一次

```html
<property>
 <name>dfs.namenode.checkpoint.txns</name>
 <value>1000000</value>
<description>操作动作次数</description>
</property>


<property>
 <name>dfs.namenode.checkpoint.check.period</name>
 <value>60s</value>
<description> 1分钟检查一次操作次数</description>
</property>
```

### 6.Datanode工作机制（了解）

#### 6.1DataNode工作机制

![1657866589063](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657866589063.png)

（1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

（2）DataNode启动后向NameNode注册，通过后，周期性（6小时）的向NameNode上报所有的块信息。

DN向NN汇报当前解读信息的时间间隔，默认6小时；

```html
<property>
	<name>dfs.blockreport.intervalMsec</name>
	<value>21600000</value>
	<description>Determines block reporting interval in milliseconds.</description>
</property>
```

DN扫描自己节点块信息列表的时间，默认6小时

```html
<property>
	<name>dfs.datanode.directoryscan.interval</name>
	<value>21600s</value>
	<description>Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk.
	Support multiple time unit suffix(case insensitive), as described
	in dfs.heartbeat.interval.
	</description>
</property>
```

（3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

（4）集群运行中可以安全加入和退出一些机器。

#### 6.2数据完整性

![1657867107812](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657867107812.png)

如下是DataNode节点保证数据完整性的方法。

（1）当DataNode读取Block的时候，它会计算CheckSum。

（2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。

（3）Client读取其他DataNode上的Block。

（4）常见的校验算法crc（32），md5（128），sha1（160）

（5）DataNode在其文件创建后周期验证CheckSum。

#### 6.3掉线时参数设置

![1657867325157](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657867325157.png)

​	需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒。

```
<property>
  <name>dfs.namenode.heartbeat.recheck-interval</name>
  <value>300000</value>
</property>

<property>
  <name>dfs.heartbeat.interval</name>
  <value>3</value>
</property>
```

## 5.MapREduce

### 1.MapReduce概述

#### 1.1定义

MapReduce是一个分布式运算程序的编程框架

#### 1.2优缺点

![1657868454526](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657868454526.png)

#### 1.3MapReduce核心思想

![1657868648015](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657868648015.png)

```
（1）分布式的运算程序往往需要分成至少2个阶段。
（2）第一个阶段的MapTask并发实例，完全并行运行，互不相干。
（3）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。
（4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。
总结：分析WordCount数据流走向深入理解MapReduce核心思想。
```

**一个完整的MapReduce程序在分布式运行时有三类实例进程：**

**（1）MrAppMaster：负责整个程序的过程调度及状态协调。**

**（2）MapTask：负责Map阶段的整个数据处理流程。**

**（3）ReduceTask：负责Reduce阶段的整个数据处理流程。**

**1.在hadoop上执行jar包**

```
hadoop jar  wc.jar  com.atguigu.mapreduce.wordcount.WordCountDriver /user/atguigu/input /user/atguigu/output
```



**2.程序打包**

```
<build>
  <plugins>
​    <plugin>
​      <artifactId>maven-compiler-plugin</artifactId>
​      <version>3.6.1</version>
​      <configuration>
​        <source>1.8</source>
​        <target>1.8</target>
​      </configuration>
​    </plugin>
​    <plugin>
​      <artifactId>maven-assembly-plugin</artifactId>
​      <configuration>
​        <descriptorRefs>
​          <descriptorRef>jar-with-dependencies</descriptorRef>
​        </descriptorRefs>
​      </configuration>
​      <executions>
​        <execution>
​          <id>make-assembly</id>
​          <phase>package</phase>
​          <goals>
​            <goal>single</goal>
​          </goals>
​        </execution>
​      </executions>
​    </plugin>
  </plugins>
</build>
```

### 2.序列化

![1658023300605](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658023300605.png)

![1658023422554](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658023422554.png)

**1.什么是序列化**

序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储到磁盘（持久化）和网络传输。 

反序列化就是将收到字节序列（或其他数据传输协议）或者是磁盘的持久化数据，转换成内存中的对象。



**2.问什么要序列化**

序列化可以存储“活的”对象，可以将“活的”对象发送到远程计算机。



**3.为什么不用Java序列化**

由于不便于在网络中高效传输。所以，Hadoop自己开发了一套序列化机制（Writable）。



**4.hadoop序列化特点**

紧凑 ：高效使用存储空间。

快速：读写数据的额外开销小。

互操作：支持多语言的交互



**5.序列化的7个步骤**

1. 必须实现Writable接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
3. 重写序列化方法
4. 重写反序列化方法
5. 注意反序列化的顺序和序列化的顺序完全一致
6. 要想把结果显示在文件中，需要重写toString()，可用"\t"分开，方便后续用。
7. 如果需要将自定义的bean放在key中传输，则还需要实现Comparable接口，因为MapReduce框中的Shuffle过程要求对key必须能排序。

### 3.核心框架原理（非常重要）

![1658026207355](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658026207355.png)

#### 3.1输入的数据InputFormat

##### 3.1.1切片与MapTask并行度决定机制

![1658026363964](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658026363964.png)



**数据块**：Block是HDFS物理上把数据分成一块一块。数据块是HDFS存储数据单位。

**数据切片**：数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。数据切片是MapReduce程序计算输入数据的单位，一个切片会对应启动一个MapTask。

**数据切片与MapTesk并行度决定机制**

![1658026706801](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658026706801.png)

##### 3.1.2job提交流程源码和切片源码详解

![1658027106228](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658027106228.png)

1. 建立连接
2. 提交job
   1. 创建给集群提交数据的Stag路径
   2. 获取jobid ，并创建Job路径
   3. 拷贝jar包到集群
   4. 计算切片，生成切片规划文件
   5. 向Stag路径写XML配置文件
   6. 提交Job,返回提交状态

**（1） FileInputFormat切片源码解析**

![1658027194307](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658027194307.png)

![1658027222245](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658027222245.png)

#####  3.1.3 TextInputFormat

**1）FileInputFormat实现类**

思考：在运行MapReduce程序时，输入的文件格式包括：基于行的日志文件、二进制格式文件、数据库表等。那么，针对不同的数据类型，MapReduce是如何读取这些数据的呢？

FileInputFormat常见的接口实现类包括：TextInputFormat、KeyValueTextInputFormat、NLineInputFormat、CombineTextInputFormat和自定义InputFormat等。

**2）TextInputFormat**

TextInputFormat是默认的FileInputFormat实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量， LongWritable类型。值是这行的内容，不包括任何行终止符（换行符和回车符），Text类型。

以下是一个示例，比如，一个分片包含了如下4条文本记录。

```
Rich learning form
Intelligent learning engine
Learning more convenient
From the real demand for more close to the enterprise
```

每条记录表示为以下键/值对：

```
(0,Rich learning form)
(20,Intelligent learning engine)
(49,Learning more convenient)
(74,From the real demand for more close to the enterprise)
```

CombineTextInputFormat切片机制

​        框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。



1）应用场景：

CombineTextInputFormat用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。



2）虚拟存储切片最大值设置

CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m

注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

3）切片机制

虚拟存储过程和切片过程二部分。

![1658027909629](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658027909629.png)

```
（a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。
（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。
（c）测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：
    1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）
    最终会形成3个切片，大小分别为：
   （1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M
```

#### 3.2MapReduce的框架原理

![1658029660544](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658029660544.png)

![1658029701878](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658029701878.png)

```
上面的流程是整个MapReduce最全工作流程，但是Shuffle过程只是从第7步开始到第16步结束，具体Shuffle过程详解，如下：
（1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中
（2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件
（3）多个溢出文件会被合并成大的溢出文件
（4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序
（5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据
（6）ReduceTask会抓取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）
（7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）
注意：
（1）Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。
（2）缓冲区的大小可以通过参数调整，参数：mapreduce.task.io.sort.mb默认100M。
```

#### 3.3Shuffle

##### 3.3.1Shuffle机制

![1658029829282](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658029829282.png)



##### 3.3.2Partition分区

##### 3.3.3Partion分区案例实操

**1.在原案例基础上加上ProvincePartitioner类**

**2.在驱动函数上增加**

```
//8 指定自定义分区器
job.setPartitionerClass(ProvincePartitioner.class);

//9 同时指定相应数量的ReduceTask
job.setNumReduceTasks(5);
```

**3.分区总结**

```
(l)如果ReduceTask的数量>getPartition的结果数，则会多产生几个空的输出文件part-r-O00xx;

(2)如果1<Reduce Task的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception;

(3)如果ReduceTask的数量-l,则不管MapTask端输出多少个分区文件，最终结果都交给这一个
ReduceTask,最终也就只会产生一个结果文件part-r-O0000;

(4)分区号必须从零开始，逐一累加。
```

##### 3.3.4WritableComparable排序

​        MapTask和ReduceTask均会对数据按照key进行排序。该操作属于Hadoop的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。

​        默认排序按照字典顺序排序，且实现该排序的方法是快速排序

​        对于MapTask,它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据谥写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。

​        对于Reduce Tas趾，它从每个Map Task上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则益写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据益写到磁盘上。当所有数据拷贝完毕后，Reduce Task统一对内存和磁盘上的所有数据进行一次归并排序。

**(1)部分排序**
        MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部有序。
**(2)全排序**
        最终输出结果只有一个文件，且文件内部有序。实现方式是只设置一个ReduceTask。但该方法在
处理大型文件时效率极低，因为一台机器处理所有文件，完全丧失了MapRe duce所提供的并行架构。
**(3)辅助排序：(Grouping Compar ator分组)**
        在Reducei端对ke进行分组。应用于：在接收的key为ben对象时，想让一个或几个字段相同（全部
字段比较不相同)的key进入到同一个reduce?方法时，可以采用分组排序。
**(4)二次排序**
        在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。

##### 3.3.5WritableComparable排序案例实操

1.FlowBean对象在在需求1基础上增加了比较功能

```
public class FlowBean implements WritableComparable<FlowBean> {

​ @Override
  public int compareTo(FlowBean o) {
   //按照总流量比较,倒序排列
​    if(this.sumFlow > o.sumFlow){
​      return -1;
​    }else if(this.sumFlow < o.sumFlow){
​      return 1;
​    }else {
​      return 0;
​    }
  }
}
```

2.编写Mapper类

3.编写Driver类

```
 //4 设置Map端输出数据的KV类型
​    job.setMapOutputKeyClass(FlowBean.class);
​    job.setMapOutputValueClass(Text.class);
```

##### 3.3.6WritableComparable分区排序案例操作

在原有已经排序的基础上

（1）增加自定义分区类

（2）在驱动类中添加分区类

##### 3.3.7Combiner合并

(l)Combiner是MR程序中Mapper7和Reducer之外的一种组件。
(2)Combiner?组件的父类就是Reducer。.
(3)Combiner和Reducer的区别在于运行的位置
			Combiner是在每一个MapTask所在的节点运行；
			Reducer是接收全局所有Mapper的输出结果；
(4)Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。
(5)Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

##### 3.3.8Combiner合并案例操作

![1658045062254](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658045062254.png)



```
// 指定需要使用combiner，以及用哪个类作为combiner的逻辑
job.setCombinerClass(WordCountCombiner.class);
```

```
// 指定需要使用Combiner，以及用哪个类作为Combiner的逻辑
job.setCombinerClass(WordCountReducer.class);
```



#### 3.4输出数据处理OutputEormat

（1）编写LogMapper类

（2）编写LogReducer类

（3）自定义一个LogOutputFormat类、

（4）自定义一个LogOutputFormat类

（5）编写LogDriver类

![1658109292160](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658109292160.png)

![1658109324839](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658109324839.png)

#### 3.5MapReduce内核源码解析

##### 3.5.1MapTask工作机制

![1658109499382](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658109499382.png)

```
（1）Read阶段：MapTask通过InputFormat获得的RecordReader，从输入InputSplit中解析出一个个key/value。
​	（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
​	（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
​	（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

​	溢写阶段详情：
​	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。
​	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
​	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

​	（5）Merge阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。
​	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。
​	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并mapreduce.task.io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
​	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。


```

##### 3.5.2 ReduceTask的工作机制

![1658109591411](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658109591411.png)

```
    （1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

​	（2）Sort阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

​	（3）Reduce阶段：reduce()函数将计算结果写到HDFS上。


```

##### 3.5.3 ReduceTask并行度决定机制

**回顾**：MapTask并行度由切片个数决定，切片个数由输入文件和切片规则决定。
**思考**：Reduce Task并行度由谁决定？
   1)设置ReduceTask并行度（个数）
           Reduce Task的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：

```
//默认值是1，手动设置为4
job.setNumReduceTasks (4);
```

![1658109960174](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658109960174.png)



**注意事项：**

(1)ReduceTask=0,表示没有Reducef阶段，输出文件个数和Map个数一致

(2)ReduceTask默认值就是1，肌以输出文件个数为一个。

(3)如果数鞍据分布不均匀，就有可能在Reduce阶段产生数据倾酴斜

(4)ReduceTask数量并不是任意设置，还要考虑务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。

(5)具体多少个ReduceTask,需要根据集群性能而定。

(6)如果分区数不是1，但是ReduceTask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断Reduce Num个数是否大于1。不大于1肯定不执行。



#### 3.6Join应用

##### 3.6.1 Reduce Join

​      Map端的主要工作：为来自不同表或文件的key/value对，打标签以区别不同来源的记录。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。

​      Reduce端的主要工作：在Reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录（在Map阶段已经打标志）分开，最后进行合并就ok了。

##### 3.6.2 Reduce Join案例实操

![1658111013877](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658111013877.png)

**（1）创建商品和订单合并后的TableBean类**

**（2）编写TableMapper类**

**（3）编写TableReducer类**

**（4）编写TableDriver类**

**总结**

​            缺点：这种方式中，合并的操作是在Reduce阶段完成，Reduce端的处理压力太大，Map节点的运算负载则很低，资源利用率不高，且在Reduce阶段极易产生数据倾斜。

解决方案：Map端实现数据合并。

##### 3.6.3Map Join

Map Join适用于一张表十分小、一张表很大的场景。

优点：



##### 3.6.4Map join实操

**思考：**在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？

**优点：**

在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。

**具体办法：**

  采用DistributedCache

（1）在Mapper的setup阶段，将文件读取到缓存集合中。

（2）在Driver驱动类中加载缓存。

#### 3.7ETL

​        “ETL，是英文Extract-Transform-Load的缩写，用来描述将数据从来源端经过抽取（Extract）、转换（Transform）、加载（Load）至目的端的过程。ETL一词较常用在数据仓库，但其对象并不限于数据仓库

​         在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。

​			**需要在Map阶段对输入的数据根据规则进行过滤清洗。**



#### 3.8总结

![1658112646369](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658112646369.png)

**1）输入数据接口：InputFormat**

​	（1）默认使用的实现类是：TextInputFormat

​	（2）TextInputFormat的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为value返回。

​	（3）CombineTextInputFormat可以把多个小文件合并成一个切片处理，提高处理效率。

**2）逻辑处理接口：Mapper** 

​		用户根据业务需求实现其中三个方法：**map()用户业务逻辑  setup()初始化  cleanup () 关闭资源**

**3）Partitioner分区**

​		（1）有默认实现 HashPartitioner，逻辑是根据key的哈希值和numReduces来返回一个分区号；		key.hashCode()&Integer.MAXVALUE % numReduces

​		（2）如果业务上有特别的需求，可以自定义分区。

**4）Comparable排序**

​		（1）当我们用自定义的对象作为key来输出时，就必须要实现WritableComparable接口，重写其中的   compareTo()方法。

​		（2）**部分排序：**对最终输出的每一个文件进行内部排序。

​		（3）**全排序：**对所有数据进行排序，通常只有一个Reduce。

​		（4）**二次排序：**排序的条件有两个。

**5）Combiner合并**

​      	Combiner合并可以提高程序执行效率，减少IO传输。但是使用时必须==不能影响原有的业务==处理结果。

​           提前聚合Map=》解决数据倾斜的一个方法

**6）逻辑处理接口：Reducer**

​     	用户根据业务需求实现其中三个方法：**map()用户业务逻辑  setup()初始化  cleanup () 关闭资源**

**7）输出数据接口：OutputFormat**

​		（1）默认实现类是TextOutputFormat，功能逻辑是：将每一个KV对，向目标文本文件输出一行。

​		（2）用户还可以自定义OutputFormat。

### 4.压缩

#### 4.1概述

**优缺点：**

压缩的优点：以减少磁盘IO、减少磁盘存储空间。

压缩的缺点：增加CPU开销。

**压缩原则：**

（1）运算密集型的Job，少用压缩

（2）IO密集型的Job，多用压缩

#### 4.2MR支持的压缩编码

**压缩算法对比：**

| 压缩格式 | Hadoop自带？ | 算法    | 文件扩展名 | 是否可切片 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 是，直接使用 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

**压缩性能比较：**

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

#### 4.3压缩方式的选择

压缩方式选择时重点考虑：压缩/解压缩速度、压缩率（压缩后存储大小）、压缩后是否可以支持切片。

**4.3.1 Gzip压缩**

优点：压缩率比较高； 

缺点：不支持Split；压缩/解压速度一般；

**4.3.2 Bzip2压缩**

优点：压缩率高；支持Split； 

缺点：压缩/解压速度慢。

**4.3.3 Lzo压缩**

优点：压缩/解压速度比较快；支持Split；

缺点：压缩率一般；想支持切片需要额外创建索引。

**4.3.4 Snappy压缩**

优点：压缩和解压缩速度快； 

缺点：不支持Split；压缩率一般； 

**4.3.5压缩后位置的选择**

![1658113203398](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658113203398.png)

#### 4.4压缩参数配置

**1）为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器**

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

**2）要在Hadoop中启用压缩，可以配置如下参数**

| 参数                                                         | 默认值                                         | 阶段        | 建议                                          |
| ------------------------------------------------------------ | ---------------------------------------------- | ----------- | --------------------------------------------- |
| io.compression.codecs  （在core-site.xml中配置）             | 无，这个需要在命令行输入hadoop checknative查看 | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器  |
| mapreduce.map.output.compress（在mapred-site.xml中配置）     | false                                          | mapper输出  | 这个参数设为true启用压缩                      |
| mapreduce.map.output.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | mapper输出  | 企业多使用LZO或Snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置） | false                                          | reducer输出 | 这个参数设为true启用压缩                      |
| mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec     | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2       |

#### 4.5压缩案例实操

##### 4.5.1Map输出端采用压缩

```
// 开启map端输出压缩
​		conf.setBoolean("mapreduce.map.output.compress", true);
​		// 设置map端输出压缩方式
​		conf.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class,CompressionCodec.class);
```

##### 4.5.2Reduce输出端采用压缩

```
	// 设置压缩的方式
​	  FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
//	  FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class); 
//	  FileOutputFormat.setOutputCompressorClass(job, DefaultCodec.class); 
```





## 6.YARN

### 1.Yarn资源调度器

 		Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

#### 1.1Yarn的基础架构

![1658114361666](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658114361666.png)

#### 1.2Yarn的工作机制

![1658114698367](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658114698367.png)

```
（1）MR程序提交到客户端所在的节点。
​	（2）YarnRunner向ResourceManager申请一个Application。
​	（3）RM将该应用程序的资源路径返回给YarnRunner。
​	（4）该程序将运行所需资源提交到HDFS上。
​	（5）程序资源提交完毕后，申请运行mrAppMaster。
​	（6）RM将用户的请求初始化成一个Task。
​	（7）其中一个NodeManager领取到Task任务。
​	（8）该NodeManager创建容器Container，并产生MRAppmaster。
​	（9）Container从HDFS上拷贝资源到本地。
​	（10）MRAppmaster向RM 申请运行MapTask资源。
​	（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。
​	（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
​	（14）ReduceTask向MapTask获取相应分区的数据。
​	（15）程序运行完毕后，MR会向RM申请注销自己。
```

#### 1.3作业提交全过程

![1658114809732](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658114809732.png)

![1658114878267](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658114878267.png)

**作业提交全过程详解**

**（1）作业提交**

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

**（2）作业初始化**

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

**（3）任务分配**

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

**（4）任务运行**

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

**（5）进度和状态更新**

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

**（6）作业完成**

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

#### 1.4Yarn调度器和调度算法

​		目前，Hadoop作业调度器主要有三种：FIFO、容量（Capacity Scheduler）和公平（Fair Scheduler）。Apache Hadoop3.1.3默认的资源调度器是Capacity Scheduler。

​		**CDH框架默认调度器是Fair Scheduler。**

​		**具体设置详见：yarn-default.xml文件**

```
<property>
  <description>The class to use as the resource scheduler.</description>
  <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

##### 	1.4.1先进先出调度器

FIFO调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。

优点：简单易懂；

缺点：不支持多队列，生产环境很少使用；

##### 	1.4.2容量调度器

![1658115209241](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658115209241.png)



![1658115279734](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658115279734.png)

##### 	1.4.3公平调度器

![1658115492134](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658115492134.png)

![1658115599726](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658115599726.png)

![1658115635271](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658115635271.png)



#### 1.5Yarn的常用命令

##### 	1.5.1yarn application查看任务

（1）列出所有Application：

```
yarn application -list
```

（2）根据Application状态过滤：yarn application -list -appStates （所有状态：ALL、NEW、NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED）

```
yarn application -list -appStates FINISHED
```

（3）Kill掉Application：

```
yarn application -kill application_1612577921195_0001
```

##### 	1.5.2yarn logs查看日志

（1）查询Application日志：yarn logs -applicationId <ApplicationId>

```
yarn logs -applicationId application_1612577921195_0001
```

（2）查询Container日志：yarn logs -applicationId <ApplicationId> -containerId <ContainerId> 

```
yarn logs -applicationId application_1612577921195_0001 -containerId container_1612577921195_0001_01_000001
```

##### 	1.5.3 yarn applicationattempt查看尝试运行的任务

（1）列出所有Application尝试的列表：yarn applicationattempt -list <ApplicationId>

```
yarn applicationattempt  -list application_1612577921195_0001
```

（2）打印ApplicationAttemp状态：yarn applicationattempt -status <ApplicationAttemptId>

```
yarn applicationattempt -status  appattempt_1612577921195_0001_000001
```

##### 	1.5.4 yarn container查看容器

​	（1）列出所有Container：yarn container -list <ApplicationAttemptId>

```
yarn container -list appattempt_1612577921195_0001_000001
```

​	（2）打印Container状态：	yarn container -status <ContainerId>

```
yarn container -status container_1612577921195_0001_01_000001
```

 ==注：只有在任务跑的途中才能看到container的状态==

##### 	1.5.5 yarn node查看节点状态

列出所有节点：yarn node -list -all

```
yarn node -list -all
```

##### 	1.5.6 yarn rmadmin更新配置

加载队列配置：yarn rmadmin -refreshQueues

```
yarn rmadmin -refreshQueues
```

##### 	1.5.7 yarn queue查看队列

​	打印队列信息：yarn queue -status <QueueName>

```
 yarn queue -status default
```

#### 1.6Yarn生产环境核心参数

![1658125694396](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658125694396.png)

### 2.Yarn案例实操



#### 2.1Yarn生产环境核心参数配置案例

1）需求：从1G数据中，统计每个单词出现次数。服务器3台，每台配置4G内存，4核CPU，4线程。

2）需求分析：

1G / 128m = 8个MapTask；1个ReduceTask；1个mrAppMaster

平均每个节点运行10个 / 3台 ≈ 3个任务（4	3	3）

3）修改yarn-site.xml配置参数如下：

4）分发配置

5）重启集群

6）执行WordCount程序

7）观察Yarn任务执行页面

#### 2.2容量调度器多队列提交案例

1）在生产环境怎么创建队列？

（1）调度器默认就1个default队列，不能满足生产要求。

  （2）按照框架：hive /spark/ flink 每个框架的任务放入指定的队列（企业用的不是特别多）

（3）按照业务模块：登录注册、购物车、下单、业务部门1、业务部门2

2）创建多队列的好处？

（1）因为担心员工不小心，写递归死循环代码，把所有资源全部耗尽。

（2）实现任务的降级使用，特殊时期保证重要的任务队列资源充足。11.11 6.18

业务部门1（重要）=》业务部门2（比较重要）=》下单（一般）=》购物车（一般）=》登录注册（次要）

#####     2.2.1需求

​	需求1：default队列占总内存的40%，最大资源容量占总资源60%，hive队列占总内存的60%，最大资源容量占总资源80%。

​	需求2：配置队列优先级

##### 	2.2.2配置多队列的容量调度器

1）在capacity-scheduler.xml中配置如下：

2）分发配置文件

3）重启Yarn或者执行yarn rmadmin -refreshQueues刷新队列，就可以看到两条队列：

##### 	2.2.3向Hive队列提交任务

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -D mapreduce.job.queuename=hive /input /output
```

##### 	2.2.4任务优先级

​		容量调度器，支持任务优先级的配置，在资源紧张时，优先级高的任务将优先获取资源。默认情况，Yarn将所有任务的优先级限制为0，若想使用任务的优先级功能，须开放该限制。

1）修改yarn-site.xml文件，增加以下参数

2）分发配置，并重启Yarn

3）模拟资源紧张环境，可连续提交以下任务，直到新提交的任务申请不到资源为止。

4）再次重新提交优先级高的任务

5）也可以通过以下命令修改正在执行的任务的优先级。

```
[atguigu@hadoop102 hadoop-3.1.3]$ yarn application -appID application_1611133087930_0009 -updatePriority 5
```



#### 2.3公平调度器案例

##### 	2.3.1需求

​		创建两个队列，分别是test和atguigu（以用户所属组命名）。期望实现以下效果：若用户提交任务时指定队列，则任务提交到指定队列运行；若未指定队列，test用户提交的任务到root.group.test队列运行，atguigu提交的任务到root.group.atguigu队列运行（注：group为用户所属组）。

公平调度器的配置涉及到两个文件，一个是yarn-site.xml，另一个是公平调度器队列分配文件fair-scheduler.xml（文件名可自定义）。

（1）配置文件参考资料：

1）修改yarn-site.xml文件，加入以下参数

（2）任务队列放置规则参考资料：

##### 	2.3.2配置多队列的公平调度器

1）修改yarn-site.xml文件，加入以下参数

1）修改yarn-site.xml文件，加入以下参数

3）分发配置并重启Yarn

##### 	2.3.3测试提交任务

1）提交任务时指定队列，按照配置规则，任务会到指定的root.test队列 

2）提交任务时不指定队列，按照配置规则，任务会到root.atguigu.atguigu队列

#### 2.4Yarn的Tool接口案例

1）需求：自己写的程序也可以动态修改参数。编写Yarn的Tool接口。

2）具体步骤：

（1）新建Maven项目YarnDemo，pom如下：

（2）新建com.atguigu.yarn报名

（3）创建类WordCount并实现Tool接口：

（4）新建WordCountDriver

3）在HDFS上准备输入文件，假设为/input目录，向集群提交该Jar包

```
[atguigu@hadoop102 hadoop-3.1.3]$ yarn jar YarnDemo.jar com.atguigu.yarn.WordCountDriver wordcount /input /output
```

