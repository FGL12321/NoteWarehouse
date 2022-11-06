# 大数据集群配置

[TOC]



# 1.0 Hadoop安装

![1659765478218](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659765478218.png)

![1659766068823](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659766068823.png)

## 1.1克隆虚拟机102,103,104

**（1）修改克隆虚拟机的静态IP**

```
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

**（2）修改主机名**

```
vim /etc/hostname
```

**（3）连接xshell和XFTP**

## 1.2伪分布式的测试

![1659766135778](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659766135778.png)

**（1）将下面六个文件放入/opt/software目录下**

```
 apache-flume-1.9.0-bin.tar.gz
 
 apache-zookeeper-3.5.7-bin.tar.gz
 
 hadoop-3.1.3.tar.gz
 
 hbase-2.4.11-bin.tar.gz
 
 jdk-8u212-linux-x64.tar.gz
 
 kafka_2.12-3.0.0.tgz
```



**（2）将JDK和Hadoop两个文件分别解压**

```
tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
```

```
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
```



**（3）添加JDK和Hadoop的环境变量**

```
sudo vim /etc/profile.d/my_env.sh
```

```
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin

#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

最后让文件生效：
source /etc/profile
```

**（4）对虚拟机进行重启并验证hadoop和JDK是否安装成功**

```
java -version
hadoop version Hadoop 3.1.3
```

**可以看到**

```
Hadoop 3.1.3  
Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
Compiled by ztang on 2019-09-12T02:47Z
Compiled with protoc 2.5.0
From source with checksum ec785077c385118ac91aadde5ec9799
This command was run using /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-common-3.1.3.jar

java version "1.8.0_212"
Java(TM) SE Runtime Environment (build 1.8.0_212-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.212-b10, mixed mode)
```

**==跳过伪分布式启动==**

## 1.3完全分布式搭建

![1659767783263](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659767783263.png)





**（1）首先进行ssh免密登录**

```
生成公钥和私钥
[atguigu@hadoop102 .ssh]$ pwd
/home/atguigu/.ssh

[atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa
然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

将公钥拷贝到要免密登录的目标机器上（每个机器上均执行以下三个命令）
[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102
[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103
[atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104

```

**(2)测试成功后进行xsync配置**

```
（a）在/home/atguigu/bin目录下创建xsync文件
[atguigu@hadoop102 opt]$ cd /home/atguigu
[atguigu@hadoop102 ~]$ mkdir bin
[atguigu@hadoop102 ~]$ cd bin
[atguigu@hadoop102 bin]$ vim xsync
在该文件中编写如下代码

#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done

（b）修改脚本 xsync 具有执行权限
[atguigu@hadoop102 bin]$ chmod +x xsync

（c）测试脚本
[atguigu@hadoop102 ~]$ xsync /home/fang/bin

（d）将脚本复制到/bin中，以便全局调用
[atguigu@hadoop102 bin]$ sudo cp xsync /bin/

（e）同步环境变量配置（root所有者）
[atguigu@hadoop102 ~]$ sudo ./bin/xsync /etc/profile.d/my_env.sh
注意：如果用了sudo，那么xsync一定要给它的路径补全。

让环境变量生效
[atguigu@hadoop103 bin]$ source /etc/profile
[atguigu@hadoop104 opt]$ source /etc/profile
```

**（3）检查103 104 上的环境变量是否生效，若果没有生效可以重启虚拟机再次检查**

```
java -version
hadoop version Hadoop 3.1.3
```

## 1.4对配置文件进行配置

![1659770217317](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659770217317.png)



**（1）核心配置文件**

```
[atguigu@hadoop102 ~]$ cd $HADOOP_HOME/etc/hadoop
[atguigu@hadoop102 hadoop]$ vim core-site.xml

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
    </property>

    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
    </property>

    <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>fang</value>  
    </property>
</configuration>
```

**（2）HDFS 配置文件**

```
[atguigu@hadoop102 hadoop]$ vim hdfs-site.xml

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
    </property>
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
</configuration>
```

**（3）YARN 配置文件**

```
[atguigu@hadoop102 hadoop]$ vim yarn-site.xml

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>

    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
<!-- 开启日志聚集功能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property>  
    <name>yarn.log.server.url</name>  
    <value>http://hadoop102:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为7天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>
</configuration>
```

**（4）MapReduce 配置文件**

```
[atguigu@hadoop102 hadoop]$ vim mapred-site.xml

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定MapReduce程序运行在Yarn上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>
</configuration>
```

**（5）配置 workers**

```
[atguigu@hadoop102 hadoop]$ vim /opt/module/hadoop-3.1.3/etc/hadoop/workers

hadoop102
hadoop103
hadoop104

同步所有配置
[atguigu@hadoop102 hadoop]$ xsync /opt/module/hadoop-3.1.3/etc
```

**（6）初始化NameNode**

```
[atguigu@hadoop102 hadoop-3.1.3]$ hdfs namenode -format
```

**（7）编写myhadoop.sh脚本**

```
[atguigu@hadoop102 ~]$ cd /home/fang/bin
[atguigu@hadoop102 bin]$ vim myhadoop.sh

#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac

记得添加执行权限
[atguigu@hadoop102 bin]$ chmod +x myhadoop.sh 
```

**（8）编写jpsall脚本**

```
[atguigu@hadoop102 ~]$ cd /home/atguigu/bin
[atguigu@hadoop102 bin]$ vim jpsall

#!/bin/bash

for host in hadoop102 hadoop103 hadoop104
do
        echo =============== $host ===============
        ssh $host jps 
done

添加脚本并分发脚本
[atguigu@hadoop102 bin]$ chmod +x jpsall
[atguigu@hadoop102 ~]$ xsync /home/fang/bin/
在103 104上添加执行权限
```

（9）启动集群

```
myhadoop.sh start  启动集群

jpsall      查看所有节点是否启动
```

(10)查看3个网址是否可以正常访问

```
http://hadoop102:9870
http://hadoop103:8088/
http://hadoop102:19888/jobhistory
```

==如果节点都正常启动，页面无法访问，检查hosts文件是否修改，如果无误，可以重写启动，并进行测试==

# 2.0 zookeeper安装

## 2.1解压安装

```
解压
[atguigu@hadoop102 software]$ tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/

更名
[atguigu@hadoop102 module]$ mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7
```

## 2.2文件配置

**配置服务器编号**

（1）在/opt/module/zookeeper-3.5.7/这个目录下创建 zkData

```
[atguigu@hadoop102 zookeeper-3.5.7]$ mkdir zkData
```

（2）在/opt/module/zookeeper-3.5.7/zkData 目录下创建一个 myid 的文件

```
[atguigu@hadoop102 zkData]$ vi myid

在文件中添加与 server 对应的编号（注意：上下不要有空行，左右不要有空格）

2

[atguigu@hadoop102 module ]$ xsync zookeeper-3.5.7

vim /opt/module/zookeeper-3.5.7/zkData/myid
并分别在 hadoop103、hadoop104 上修改 myid 文件中内容为 3、4
```

**（3）配置zoo.cfg文件**

```
（1）重命名/opt/module/zookeeper-3.5.7/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg
[atguigu@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg

（2）打开 zoo.cfg 文件
[atguigu@hadoop102 conf]$ vim zoo.cfg

#修改数据存储路径配置
dataDir=/opt/module/zookeeper-3.5.7/zkData
#增加如下配置
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888

（3）同步 zoo.cfg 配置文件
[atguigu@hadoop102 conf]$ xsync zoo.cfg
```

**（4）编写启停脚本**

```
1）在 hadoop102 的/home/atguigu/bin 目录下创建脚本
[atguigu@hadoop102 bin]$ vim zk.sh

#!/bin/bash
case $1 in
"start"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 启动 ------------
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 停止 ------------    
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in hadoop102 hadoop103 hadoop104
	do
        echo ---------- zookeeper $i 状态 ------------    
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh status"
	done
};;
esac

2）增加脚本执行权限
[atguigu@hadoop102 bin]$ chmod u+x zk.sh

3）Zookeeper 集群启动脚本
[atguigu@hadoop102 module]$ zk.sh start

4）Zookeeper 集群停止脚本
[atguigu@hadoop102 module]$ zk.sh stop

输入jpsall正常启动即可
```

**（5）zookeeper的操作**

```
启动zookeeper客户端
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkCli.sh -server hadoop102:2181

查看zookeeper节点信息
[zk: hadoop102:2181(CONNECTED) 0] ls /
```

# 3.0HBase安装

​		HBase 通过 Zookeeper 来做 master 的高可用、记录 RegionServer 的部署信息、并且存储有 meta 表的位置信息。 

​		HBase 对于数据的读写操作时直接访问 Zookeeper 的，在 2.3 版本推出 Master Registry 模式，客户端可以直接访问 master。使用此功能，会加大对 master 的压力，减轻对 Zookeeper的压力。



## 3.1解压安装、环境变量

```
1）解压 Hbase 到指定目录
[atguigu@hadoop102 software]$ tar -zxvf hbase-2.4.11-bin.tar.gz -C /opt/module/
[atguigu@hadoop102 software]$ mv /opt/module/hbase-2.4.11 /opt/module/hbase

2）配置环境变量
[atguigu@hadoop102 ~]$ sudo vim /etc/profile.d/my_env.sh

添加
：

3）使用 source 让配置的环境变量生效
[atguigu@hadoop102 module]$ source /etc/profile.d/my_env.sh

[fang@hadoop102 hbase]$ sudo /home/fang/bin/xsync /etc/profile.d/my_env.sh 

```



## 3.2文件配置

1）hbase-env.sh 修改内容，可以添加到最后：

在hbase/conf目录下

```
[fang@hadoop102 conf]$ vim hbase-env.sh

最后面添加一句
export HBASE_MANAGES_ZK=false

```

2）hbase-site.xml 修改内容： ==把原先标签内的内容删掉，直接替换成下方的即可==

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>

<property>
 <name>hbase.zookeeper.quorum</name>
 <value>hadoop102,hadoop103,hadoop104</value>
 <description>The directory shared by RegionServers.
 </description>
 </property>
 
<!-- <property>-->
<!-- <name>hbase.zookeeper.property.dataDir</name>-->
<!-- <value>/export/zookeeper</value>-->
<!-- <description> 记得修改 ZK 的配置文件 -->
<!-- ZK 的信息不能保存到临时文件夹-->
<!-- </description>-->
<!-- </property>-->

 <property>
 <name>hbase.rootdir</name>
 <value>hdfs://hadoop102:8020/hbase</value>
 <description>The directory shared by RegionServers.
 </description>
 </property>
 
 <property>
 <name>hbase.cluster.distributed</name>
 <value>true</value>
 </property>
```

3）修改regionservers

```
hadoop102 
hadoop103 
hadoop104 
```

4）解决 HBase 和 Hadoop 的 log4j 兼容性问题，修改 HBase 的 jar 包，使用 Hadoop 的 jar 包

```
[atguigu@hadoop102hbase]$mv /opt/module/hbase/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar /opt/module/hbase/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar.bak




发送到其他
xsync hbase/
```

5）Hbase启停

```
[atguigu@hadoop102 hbase]$ bin/start-hbase.sh
[atguigu@hadoop102 hbase]$ bin/stop-hbase.sh
```



# 4.0 Flume安装

## 4.1解压安装

```
解压
[atguigu@hadoop102 software]$ tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/

改名
[atguigu@hadoop102 module]$ mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume

将 lib 文件夹下的 guava-11.0.2.jar 删除以兼容 Hadoop 3.1.3
[atguigu@hadoop102 lib]$ rm /opt/module/flume/lib/guava-11.0.2.jar
```

## 4.2 配置

```
（1）安装 netcat 工具
[atguigu@hadoop102 software]$ sudo yum install -y nc

（2）判断 44444 端口是否被占用
[atguigu@hadoop102 flume-telnet]$ sudo netstat -nlp | grep 44444

（3）创建 Flume Agent 配置文件 flume-netcat-logger.conf
（4）在 flume 目录下创建 job 文件夹并进入 job 文件夹。
[atguigu@hadoop102 flume]$ mkdir job
[atguigu@hadoop102 flume]$ cd job/

（5）在 job 文件夹下创建 Flume Agent 配置文件 flume-netcat-logger.conf。
[atguigu@hadoop102 job]$ vim flume-netcat-logger.conf

（6）在 flume-netcat-logger.conf 文件中添加如下内容。
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

（7）先开启 flume 监听端口
[atguigu@hadoop102 flume]$ bin/flume-ng agent -c conf/ -n a1 -f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console

复制102会话

（8）使用 netcat 工具向本机的 44444 端口发送内容
[atguigu@hadoop102 ~]$ nc localhost 44444

```

==到这就可以了，项目中会继续进行配置==

# 5.0 Kafka安装

## 5.1解压安装配置环境变量

```
解压
[atguigu@hadoop102 software]$ tar -zxvf kafka_2.12-3.0.0.tgz -C /opt/module/

改名
[atguigu@hadoop102 module]$ mv kafka_2.12-3.0.0/ kafka

进入到/opt/module/kafka 目录，修改配置文件
[atguigu@hadoop102 kafka]$ cd config/
[atguigu@hadoop102 config]$ vim server.properties

修改三个地方：记得找对相应的位置

#broker 的全局唯一编号，不能重复，只能是数字。
broker.id=0

#kafka 运行日志(数据)存放的路径，路径不需要提前创建，kafka 自动帮你创建，可以
配置多个磁盘路径，路径与路径之间可以用"，"分隔
log.dirs=/opt/module/kafka/datas


# 检查过期数据的时间，默认 5 分钟检查一次是否数据过期
log.retention.check.interval.ms=300000
#配置连接 Zookeeper 集群地址（在 zk 根目录下创建/kafka，方便管理）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka

分发
[atguigu@hadoop102 module]$ xsync kafka/

分别在 hadoop103 和 hadoop104 上修改配置文件
[fang@hadoop102 flume]$ vim /opt/module/kafka/config/server.properties 
中的 broker.id=1、broker.id=2

在/etc/profile.d/my_env.sh 文件中增加 kafka 环境变量配置
[atguigu@hadoop102 module]$ sudo vim /etc/profile.d/my_env.sh

#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin
刷新
[atguigu@hadoop102 module]$ source /etc/profile

分发并刷新
[atguigu@hadoop102 module]$ sudo /home/atguigu/bin/xsync /etc/profile.d/my_env.sh
[atguigu@hadoop103 module]$ source /etc/profile
[atguigu@hadoop104 module]$ source /etc/profile
```

## 5.2集群启动

```
（1）先启动 Zookeeper 集群，然后启动 Kafka。
[atguigu@hadoop102 kafka]$ zk.sh start

（2）集群脚本
1）在/home/atguigu/bin 目录下创建文件 kf.sh 脚本文件
[atguigu@hadoop102 bin]$ vim kf.sh


#! /bin/bash
case $1 in
        "start" ){
        for i in hadoop102 hadoop103 hadoop104;
        do
                echo "==============$i start=============="
                ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
        done
};;
        "stop" ){
        for i in hadoop102 hadoop103 hadoop104;
        do
                echo "==============$i stop=============="
                ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh"
        done
};;
esac


2）添加执行权限
[atguigu@hadoop102 bin]$ chmod +x kf.sh
3）启动集群命令
[atguigu@hadoop102 ~]$ kf.sh start
4）停止集群命令
[atguigu@hadoop102 ~]$ kf.sh stop


```

# 6.0Redis环境配置

**1.将软件包上传到/opt/softwere目录下，解压到/opt/modul下**

```
[root@linux softwere]# tar -zxvf redis-3.0.0.tar.gz -C /opt/module/
```

**2.进入redis目录进行启动**

```
[root@linux redis-3.0.0]# make
```

==**出现如下报错：没有找到gcc，此时需要安装gcc**==

```
[root@linux module]# yum -y install gcc
```

![1660718220098](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660718220098.png)

**安装完成后验证是否安装成功**

```
[root@linux module]# gcc -v
```

![1660721470107](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660721470107.png)

**3.再次进行make出现如下报错**

![1660721564091](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660721564091.png)

此时执行

```
#清理残余文件
make distclean

#再次进行make
make
```

执行成功

![1660721704391](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660721704391.png)

**4.执行**

```
make install
```

![1660721859955](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660721859955.png)

**5.查看安装目录**

```
[root@linux redis-3.0.0]# cd /usr/local/bin/
```

![1660722008875](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660722008875.png)

6.回到Redis目录备份文件

```
#在根目录创建文件夹
[root@linux /]# mkdir myredis

#将redis.conf移动到文件夹汇总
[root@linux redis-3.0.0]# cp redis.conf /myredis/

#在myredis目录中编辑文件
[root@linux myredis]# vi redis.conf 
```

![1660722373407](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660722373407.png)

```
#切换目录
[root@linux myredis]# cd /usr/local/bin/

#启动redis
[root@linux bin]# redis-server /myredis/redis.conf 

#查看客户端
[root@linux bin]# redis-cli -p 6379

```

![1660722559207](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660722559207.png)

**测试**

![1660722595954](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660722595954.png)

查看redis在后台是否启动

```
[root@linux bin]# ps -ef|grep redis
```

![1660722694207](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660722694207.png)

# 7.0MySql安装

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

​		==过程中出现报错则说明需要安装依赖，通过 yum 安装缺少的依赖,然后重新安装 mysql-community-server-5.7.28-1.el7.x86_64 即可==cd

```
[atguigu@hadoop102 software] yum install -y libaio
```

**5）删除/etc/my.cnf 文件中 datadir 指向的目录下的所有内容,如果有内容的情况下:**

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

# 8.0Tomcat安装

**（1）上传Tomcat文件包到/opt/softwere**

```
apache-tomcat-8.5.75 
```

**（2）解压Tomacat**

```
tar -zxvf apache-tomcat-8.5.75 
mv  apache-tomcat-8.5.75 /opt/module
```

### 8.8.1如何将项目打war包

**1.在IDEA中打开Project Structure**

![1660705598516](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705598516.png)

**2.选择**

![1660705637100](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705637100.png)

**3.点击+号**

![1660705666763](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705666763.png)

**4.选择**

![1660705700030](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705700030.png)

**5.配置并打包**

![1660705778126](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705778126.png)

![1660705807161](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705807161.png)

![1660705831419](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660705831419.png)

### 8.8.2如何在本地Tomcat运行war包

**1.将war包放入本地Tomcat目录中的webapp下**

![1660706036288](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706036288.png)

**2.在tomcat  bin目录下打开终端输入.\startup.bat**

![1660706169703](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706169703.png)

**3.弹出运行框正常运行即可访问web项目**

![1660706234298](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706234298.png)

**4.关闭Tomcat**

![1660706297352](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660706297352.png)



**集群环境查看**

此时输入：jpsall  可查看到一下节点相应信息

|       软件       |    hadoop102     |    hadoop103    |     hadoop104     |
| :--------------: | :--------------: | :-------------: | :---------------: |
|   Hadoop  HDFS   |     NameNode     |                 | SecondaryNameNode |
|                  |     DataNode     |    DataNode     |     DataNode      |
|   Hadoop  YARN   |                  | ResourceManager |                   |
|                  |   NodeManager    |   NodeManager   |    NodeManager    |
| Hadoop历史服务器 | JobHistoryServer |                 |                   |
|    Zookeeper     |  QuorumPeerMain  | QuorumPeerMain  |  QuorumPeerMain   |
|      Kafka       |      Kafka       |      Kafka      |       Kafka       |
|      HBase       |     HMaster      |                 |                   |
|                  |  HRegionServer   |  HRegionServer  |   HRegionServer   |
|                  |       jsp        |       jsp       |        jsp        |

![1660900727767](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1660900727767.png)

集群停止脚本：**

```
Hadoop启停：
myhadoop.sh start
myhadoop.sh stop

HBase启停：
bin/start-hbase.sh
bin/stop-hbase.sh

Zookeeper启停： 先启停Zookeeper再启停Kafka
zk.sh start
zk.sh stop

KafKa启停：
kf.sh start
kf.sh stop

各大网址：
http://hadoop102:9870
http://hadoop103:8088/
http://hadoop102:19888/jobhistory
http://hadoop102:16010/master-status
```

==注意：==

==Hbase启动前先启动zookeeper再启动Hadoop==

==必须先关闭KafKa再关闭Zookeeper==

