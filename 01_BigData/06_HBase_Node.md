[TOC]



# HBase学习笔记

# 第一章：HBase简介（1-6）

## 1.1 HBase 定义

**Apache HBase™ 是以 hdfs 为数据存储的，一种分布式、可扩展的 NoSQL 数据库。**

## 1.2 HBase 数据模型

**HBase 的设计理念依据 Google 的 BigTable 论文，论文中对于数据模型的首句介绍。**
		Bigtable 是一个==稀疏的、分布式的、持久的多维排序 map。==
		之后对于映射的解释如下：
**该映射由行键、列键和时间戳索引；映射中的每个值都是一个未解释的字节数组。**
		最终 HBase 关于数据模型和 BigTable 的对应关系如下：
HBase 使用与 Bigtable 非常相似的数据模型。用户将数据行存储在带标签的表中。数
据行具有可排序的键和任意数量的列。该表存储稀疏，因此如果用户喜欢，同一表中的行可
以具有疯狂变化的列。
		最终理解 HBase 数据模型的关键在于稀疏、分布式、多维、排序的映射。其中映射 map
指代非关系型数据库的 key-Value 结构。

### 1.2.1 HBase 逻辑结构

![1659187919513](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659187919513.png)

==先切分行再切分列==

### 1.2.2 HBase 物理存储结构

**物理存储结构即为数据映射关系，而在概念视图的空单元格，底层实际根本不存储。**

![1659187998595](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659187998595.png)



### 1.2.3 数据模型

**1）Name Space**

​		命名空间，类似于关系型数据库的 database 概念，每个命名空间下有多个表。HBase 两个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default表是用户默认使用的命名空间。 

**2）Table**

​		类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明列族即可，不需
要声明具体的列。因为数据存储时稀疏的，所有往 HBase 写入数据时，字段可以动态、按需
指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景。

**3）Row**

​		HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要。

**4）Column**

​		HBase 中的每个列都由 Column Family(列族)和 Column Qualifier（列限定符）进行限
定，例如 info：name，info：age。建表时，只需指明列族，而列限定符无需预先定义。

**5）Time Stamp**

​		用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，
其值为写入 HBase 的时间。

**6）Cell**

​		由{rowkey, column Family：column Qualifier, timestamp} 唯一确定的单元。cell 中的数据全部是字节码形式存贮。

## 1.3 HBase 基本架构

![1659188622783](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659188622783.png)

**架构角色： 1）Master** 
实现类为 HMaster，负责监控集群中所有的 RegionServer 实例。主要作用如下：
	（1）管理元数据表格 hbase:meta，接收用户对表格创建修改删除的命令并执行
	（2）监控 region 是否需要进行负载均衡，故障转移和 region 的拆分。
通过启动多个后台线程监控实现上述功能：
	①LoadBalancer 负载均衡器
	周期性监控 region 分布在 regionServer 上面是否均衡，由参数 hbase.balancer.period 控
	制周期时间，默认 5 分钟。
	②CatalogJanitor 元数据管理器
	定期检查和清理 hbase:meta 中的数据。meta 表内容在进阶中介绍。
	③MasterProcWAL master 预写日志处理器
	把 master 需要执行的任务记录到预写日志 WAL 中，如果 master 宕机，让backupMaster	读取日志继续干。
**2）Region Server** 
	 Region Server 实现类为 HRegionServer，主要作用如下: （1）负责数据 cell 的处理，例如写入数据 put，查询数据 get 等 （2）拆分合并 region 的实际执行者，有 master 监控，有 regionServer 执行。

**3）Zookeeper** 

​		HBase 通过 Zookeeper 来做 master 的高可用、记录 RegionServer 的部署信息、并且存储有 meta 表的位置信息。 HBase 对于数据的读写操作时直接访问 Zookeeper 的，在 2.3 版本推出 Master Registry模式，客户端可以直接访问 master。使用此功能，会加大对 master 的压力，减轻对 Zookeeper的压力。
**4）HDFS** 
​       HDFS 为 Hbase 提供最终的底层数据存储服务，同时为 HBase 提供高容错的支持。

# 第二章：HBase快速入门（7-13）

## 2.1安装部署

### 2.1.1 Zookeeper 正常部署

### 2.1.2 Hadoop 正常部署

### 2.1.3 HBase 的解压

**1）解压 Hbase 到指定目录**

```
[atguigu@hadoop102 software]$ tar -zxvf hbase-2.4.11-bin.tar.gz -C 
/opt/module/
[atguigu@hadoop102 software]$ mv /opt/module/hbase-2.4.11
/opt/module/hbase
```

**2）配置环境变量**

```
[atguigu@hadoop102 ~]$ sudo vim /etc/profile.d/my_env.sh
#HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

**3）使用 source 让配置的环境变量生效**

```
[atguigu@hadoop102 module]$ source /etc/profile.d/my_env.sh
```

### 2.1.4 HBase 的配置文件

1）hbase-env.sh 修改内容，可以添加到最后：

```
export HBASE_MANAGES_ZK=false
```

2）hbase-site.xml 修改内容：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
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
</configuration>
```

**3）regionservers**

```
hadoop102
hadoop103
hadoop104
```

**4）解决 HBase 和 Hadoop 的 log4j 兼容性问题，修改 HBase 的 jar 包，使用 Hadoop 的 jar 包**

```
[atguigu@hadoop102 hbase]$ mv /opt/module/hbase/lib/client-facingthirdparty/slf4j-reload4j-1.7.33.jar /opt/module/hbase/lib/clientfacing-thirdparty/slf4j-reload4j-1.7.33.jar.bak
```

### 2.1.5 HBase 远程发送到其他集群

```
[atguigu@hadoop102 module]$ xsync hbase/
```

### 2.1.6 HBase 服务的启动

```
[atguigu@hadoop102 hbase]$ bin/start-hbase.sh
```

```
[atguigu@hadoop102 hbase]$ bin/stop-hbase.sh
```

### 2.1.7 查看 HBase 页面

http://hadoop102:16010

### 2.1.8 高可用（可选）

​		在 HBase 中 HMaster 负责监控 HRegionServer 的生命周期，均衡 RegionServer 的负载，如果 HMaster 挂掉了，那么整个 HBase 集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以 HBase 支持对 HMaster 的高可用配置。

**1）关闭 HBase 集群（如果没有开启则跳过此步）**

```
[atguigu@hadoop102 hbase]$ bin/stop-hbase.sh
```

**2）在 conf 目录下创建 backup-masters 文件**

```
[atguigu@hadoop102 hbase]$ touch conf/backup-masters
```

**3）在 backup-masters 文件中配置高可用 HMaster 节点**

```
[atguigu@hadoop102 hbase]$ echo hadoop103 > conf/backup-masters
```

**4）将整个 conf 目录 scp 到其他节**

```
[atguigu@hadoop102 hbase]$ xsync conf
```

**5）重启 hbase,打开页面测试查看**

## 2.2 HBase Shell 操作

### 2.2.1 基本操作

**1）进入 HBase 客户端命令行**  

```
[atguigu@hadoop102 hbase]$ bin/hbase shell
```

**2）查看帮助命令** 
		能够展示 HBase 中所有能使用的命令，主要使用的命令有 namespace 命令空间相关，DDL 创建修改表格，DML 写入读取数据。

```
hbase:001:0> help
```

### 2.2.2 namespace

1）创建命名空间 
使用特定的 help 语法能够查看命令如何使用。

```
hbase:002:0> help 'create_namespace'
```

2）创建命名空间 bigdata 

```
hbase:003:0> create_namespace 'bigdata' 3）查看所有的命名空间 
hbase:004:0> list_namespace
```

### 2.2.3 DDL

**1）创建表** 

在 bigdata 命名空间中创建表格 student，两个列族。info 列族数据维护的版本数为 5 个，如果不写默认版本数为 1。
**hbase:005:0> create 'bigdata:student', {NAME => 'info', VERSIONS => 5}, {NAME => 'msg'}**
如果创建表格只有一个列族，没有列族属性，可以简写。
如果不写命名空间，使用默认的命名空间 default。
**hbase:009:0> create 'student1','info'**
**2）查看表** 
查看表有两个命令：list 和 describe
list：查看所有的表名
**hbase:013:0> list**
describe：查看一个表的详情
**hbase:014:0> describe 'student1'**

**3）修改表** 
表名创建时写的所有和列族相关的信息，都可以后续通过 alter 修改，包括增加删除列
族。

**（1）增加列族和修改信息都使用覆盖的方法**

```
hbase:015:0> alter 'student1', {NAME => 'f1', VERSIONS => 3}
```

**（2）删除信息使用特殊的语法**

**（2）删除信息使用特殊的语法**

```
hbase:015:0> alter 'student1', NAME => 'f1', METHOD => 'delete'
hbase:016:0> alter 'student1', 'delete' => 'f1'
```

**4）删除表** 
shell 中删除表格,需要先将表格状态设置为不可用。

```
hbase:017:0> disable 'student1'
hbase:018:0> drop 'student1'
```

### 2.2.4 DML

**1）写入数据** 

在 HBase 中如果想要写入数据，只能添加结构中最底层的 cell。可以手动写入时间戳指定 cell 的版本，推荐不写默认使用当前的系统时间。

```
hbase:019:0> put 'bigdata:student','1001','info:name','zhangsan'
hbase:020:0> put 'bigdata:student','1001','info:name','lisi'
hbase:021:0> put 'bigdata:student','1001','info:age','18'
```

如果重复写入相同 rowKey，相同列的数据，会写入多个版本进行覆盖。
**2）读取数据** 
读取数据的方法有两个：get 和 scan。
get 最大范围是一行数据，也可以进行列的过滤，读取数据的结果为多行 cell。

```
hbase:022:0> get 'bigdata:student','1001'
hbase:023:0> get 'bigdata:student','1001' , {COLUMN => 'info:name'}
```

也可以修改读取 cell 的版本数，默认读取一个。最多能够读取当前列族设置的维护版本数。
hbase:024:0>get 'bigdata:student','1001' , {COLUMN => 'info:name', VERSIONS => 6}

scan 是扫描数据，能够读取多行数据，不建议扫描过多的数据，推荐使用 startRow 和stopRow 来控制读取的数据，默认范围左闭右开。

```
hbase:025:0> scan 'bigdata:student',{STARTROW => '1001',STOPROW => '1002'}
```

实际开发中使用 shell 的机会不多，所有丰富的使用方法到 API 中介绍。
**3）删除数据**

删除数据的方法有两个：delete 和 deleteall。 

delete 表示删除一个版本的数据，即为 1 个 cell，不填写版本默认删除最新的一个版本。 

```
hbase:026:0> delete 'bigdata:student','1001','info:name' 
```

deleteall 表示删除所有版本的数据，即为当前行当前列的多个 cell。（执行命令会标记 

数据为要删除，不会直接将数据彻底删除，删除数据只在特定时期清理磁盘时进行） 

```
hbase:027:0> deleteall 'bigdata:student','1001','info:name'
```



# 第三章：HBase API（14-31）

## 3.1 环境准备

```
    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>2.4.11</version>
            <exclusions>
                <exclusion>
                    <groupId>org.glassfish</groupId>
                    <artifactId>javax.el</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.el</artifactId>
            <version>3.0.1-b06</version>
        </dependency>
    </dependencies>
```

## 3.2 创建连接



### 3.2.1 单线程创建连接

```
//1.创建连接配置对象
        Configuration conf=new Configuration();

        //2.添加配置参数
        conf.set("hbase.zookeeper.quorum","hadoop102,hadoop103,hadoop104");

        //3.创建连接
        //默认是要同步连接，也可以使用异步
        Connection connection= ConnectionFactory.createConnection(conf);

        //4.使用连接
        System.out.println(connection);

        //5.关闭连接
        connection.close();

        //直接使用创建好的连接，不要在main线程里面单独创建
        System.out.println(HbaseConnection.connection);

        //在main线程的最后记得关闭连接
        HbaseConnection.closeConnection();
```

### 3.2.2 多线程创建连接

```
//声明一个静态属性
    public static Connection connection=null;
    static{

        //1.创建连接
        try {
            //使用读去本地文件的形式添加参数
            connection= ConnectionFactory.createConnection();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void closeConnection() throws IOException {
        //判断连接是否为null
        if (connection!=null){
            connection.close();
        }
    }
```

## 3.3 DDL

### 3.3.1 创建命名空间

```
 //创建命名空间
    public static void createNamespace(String namespace) throws IOException {

        //1.获取admin
        //此处的异常先不要抛出
        //admin的连接时轻量级的不是线程安全的，不推荐池化或者缓存这个连接
        Admin admin = connection.getAdmin();

        //2.调用方法创建命名空间
        //代码相对shell更加底层，所以shell能够实现的功能代码一定能实现
        //所以需要填写完整的命名空间描述

        //2.1创建命名空间描述建造者=》设计师
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(namespace);

        //2.2给命名空间添加需求
        builder.addConfiguration("user","fang");

        //2.3使用builder构造出对应的添加玩参数的对象，完成创建
        //创建命名空间出现的问题都属于本身自身的问题
        try {
            admin.createNamespace(builder.build());
        } catch (IOException e) {
            System.out.println("命名空间已经存在");
            e.printStackTrace();
        }

        //3.关闭admin
        admin.close();

    }
```



### 3.3.2 判断表格是否存在

```
//判断表格是否存在
    public static boolean isTableExists(String namespace,String tableName) throws IOException {
        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.使用方法
        boolean b = false;
        try {
            b = admin.tableExists(TableName.valueOf(namespace, tableName));
        } catch (IOException e) {

            e.printStackTrace();
        }

        admin.close();

        //3.返回结果
        return b;

        //后面的代码不能生效
    }
```

### 3.3.3 创建表

```
//新建表格
    public static void createTable(String namespace,String tableName,String... columnFamilies) throws IOException {
        //判断是否至少有一个列祖
        if (columnFamilies.length==0){
            System.out.println("创建表格至少有一个列祖");
            return;
        }

        //判断表格是否存在
        if (isTableExists(namespace,tableName)){
            System.out.println("表格已经存在");
            return;
        }

        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.调用方法，创建表格

        //2.1创建表格描述
        TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf(namespace, tableName));

        //2.2添加参数

        for (String columnFamily:columnFamilies){

            //2.3创建列族描述
            ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder =
                    ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(columnFamily));

            //2.4对应当前的列族添加参数
            //添加版本参数
            columnFamilyDescriptorBuilder.setMaxVersions(5);

            //2.5创建添加完参数的列祖描述
            tableDescriptorBuilder.setColumnFamily(columnFamilyDescriptorBuilder.build());
        }

        //2.6创建对应的表格描述
        try {
            admin.createTable(tableDescriptorBuilder.build());
        } catch (IOException e) {
            e.printStackTrace();
        }

        //3.关闭admin
        admin.close();
    }
```

### 3.3.4 修改表

```
 //修改表格
    public static void modifyTable(String namespace ,String tableName,String columnFamily,int version) throws IOException {
        // 判断表格是否存在
        if (!isTableExists(namespace,tableName)){
            System.out.println("表格不存在无法修改");
            return;
        }
        // 1. 获取 admin
        Admin admin = connection.getAdmin();
        try {
            // 2. 调用方法修改表格
            // 2.0 获取之前的表格描述
            TableDescriptor descriptor =
                    admin.getDescriptor(TableName.valueOf(namespace, tableName));
            // 2.1 创建一个表格描述建造者
            // 如果使用填写 tableName 的方法 相当于创建了一个新的表格描述建造者没有之前的信息
            // 如果想要修改之前的信息 必须调用方法填写一个旧的表格描述
            TableDescriptorBuilder tableDescriptorBuilder =
                    TableDescriptorBuilder.newBuilder(descriptor);
            // 2.2 对应建造者进行表格数据的修改
            ColumnFamilyDescriptor columnFamily1 =
                    descriptor.getColumnFamily(Bytes.toBytes(columnFamily));
            // 创建列族描述建造者
            // 需要填写旧的列族描述
            ColumnFamilyDescriptorBuilder
                    columnFamilyDescriptorBuilder =
                    ColumnFamilyDescriptorBuilder.newBuilder(columnFamily1);
            // 修改对应的版本
            columnFamilyDescriptorBuilder.setMaxVersions(version);
            // 此处修改的时候 如果填写的新创建 那么别的参数会初始化

            tableDescriptorBuilder.modifyColumnFamily(columnFamilyDescriptorBuilder.build());
            admin.modifyTable(tableDescriptorBuilder.build());
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 3. 关闭 admin
        admin.close();
    }
```

### 3.3.5 删除表

```
    /**
     * 删除表格
     * @param namespace 命名空间名称
     * @param tableName 表格名称
     * @return true 表示删除成功
     */
    public static boolean deleteTable(String namespace ,String tableName) throws IOException {
        // 1. 判断表格是否存在
        if (!isTableExists(namespace,tableName)){
            System.out.println("表格不存在 无法删除");
            return false;
        }
        // 2. 获取 admin
        Admin admin = connection.getAdmin();
        // 3. 调用相关的方法删除表格
        try {
            // HBase 删除表格之前 一定要先标记表格为不可以
            TableName tableName1 = TableName.valueOf(namespace,
                    tableName);
            admin.disableTable(tableName1);
            admin.deleteTable(tableName1);
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 4. 关闭 admin
        admin.close();
        return true;
    }
```

## 3.4 DML

### 3.4.1 插入数据

```
// 添加静态属性 connection 指向单例连接
    public static Connection connection =HbaseConnection.connection;
    /**
     * 插入数据
     * @param namespace 命名空间名称
     * @param tableName 表格名称
     * @param rowKey 主键
     * @param columnFamily 列族名称
     * @param columnName 列名
     * @param value 值
     */
    public static void putCell(String namespace,String tableName,String rowKey, String columnFamily,String columnName,String value) throws IOException {
        // 1. 获取 table
        Table table = connection.getTable(TableName.valueOf(namespace, tableName));
        // 2. 调用相关方法插入数据
        // 2.1 创建 put 对象
        Put put = new Put(Bytes.toBytes(rowKey));
        // 2.2. 给 put 对象添加数据

        put.addColumn(Bytes.toBytes(columnFamily),Bytes.toBytes(columnName), Bytes.toBytes(value));
        // 2.3 将对象写入对应的方法
        try {
            table.put(put);
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
    }
```

### 3.4.2 读取数据

```
    /**
     * 读取数据 读取对应的一行中的某一列
     *
     * @param namespace 命名空间名称
     * @param tableName 表格名称
     * @param rowKey 主键
     * @param columnFamily 列族名称
     * @param columnName 列名
     */
    public static void getCells(String namespace, String tableName,String rowKey, String columnFamily, String columnName) throws IOException {
        // 1. 获取 table
        Table table = connection.getTable(TableName.valueOf(namespace, tableName));
        // 2. 创建 get 对象
        Get get = new Get(Bytes.toBytes(rowKey));
        // 如果直接调用 get 方法读取数据 此时读一整行数据
        // 如果想读取某一列的数据 需要添加对应的参数
        get.addColumn(Bytes.toBytes(columnFamily),
                Bytes.toBytes(columnName));
        // 设置读取数据的版本
        get.readAllVersions();
        try {
            // 读取数据 得到 result 对象
            Result result = table.get(get);
            // 处理数据
            Cell[] cells = result.rawCells();
            // 测试方法: 直接把读取的数据打印到控制台
            // 如果是实际开发 需要再额外写方法 对应处理数据
            for (Cell cell : cells) {
                // cell 存储数据比较底层
                String value = new String(CellUtil.cloneValue(cell));
                System.out.println(value);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 关闭 table
        table.close();
    }
```

### 3.4.3 扫描数据

```
    /**
     * 扫描数据
     *
     * @param namespace 命名空间
     * @param tableName 表格名称
     * @param startRow 开始的 row 包含的
     * @param stopRow 结束的 row 不包含
     */
    public static void scanRows(String namespace, String tableName,String startRow, String stopRow) throws IOException {
        // 1. 获取 table
        Table table =
                connection.getTable(TableName.valueOf(namespace, tableName));
        // 2. 创建 scan 对象
        Scan scan = new Scan();
        // 如果此时直接调用 会直接扫描整张表
        // 添加参数 来控制扫描的数据
        // 默认包含
        scan.withStartRow(Bytes.toBytes(startRow));
        // 默认不包含
        scan.withStopRow(Bytes.toBytes(stopRow));
        try {
            // 读取多行数据 获得 scanner
            ResultScanner scanner = table.getScanner(scan);
            // result 来记录一行数据 cell 数组
            // ResultScanner 来记录多行数据 result 的数组
            for (Result result : scanner) {
                Cell[] cells = result.rawCells();
                for (Cell cell : cells) {
                    System.out.print (new
                            String(CellUtil.cloneRow(cell)) + "-" + new
                            String(CellUtil.cloneFamily(cell)) + "-" + new
                            String(CellUtil.cloneQualifier(cell)) + "-" + new
                            String(CellUtil.cloneValue(cell)) + "\t");
                }
                System.out.println();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
    }
```

### 3.4.4 带过滤扫描

```
/**
     * 带过滤的扫描
     *
     * @param namespace 命名空间
     * @param tableName 表格名称
     * @param startRow 开始 row
     * @param stopRow 结束 row
     * @param columnFamily 列族名称
     * @param columnName 列名
     * @param value value 值
     *  @throws IOException
      */
    public static void filterScan(String namespace, String tableName, String startRow, String stopRow, String columnFamily, String columnName, String value) throws IOException {
        // 1. 获取 table
        Table table =
                connection.getTable(TableName.valueOf(namespace, tableName));
        // 2. 创建 scan 对象
        Scan scan = new Scan();
        // 如果此时直接调用 会直接扫描整张表
        // 添加参数 来控制扫描的数据
        // 默认包含
        scan.withStartRow(Bytes.toBytes(startRow));
        // 默认不包含
        scan.withStopRow(Bytes.toBytes(stopRow));
        // 可以添加多个过滤
        FilterList filterList = new FilterList();
        // 创建过滤器
        // (1) 结果只保留当前列的数据
        ColumnValueFilter columnValueFilter = new ColumnValueFilter(
                // 列族名称
                Bytes.toBytes(columnFamily),
                // 列名
                Bytes.toBytes(columnName),
                // 比较关系
                CompareOperator.EQUAL,
                // 值
                Bytes.toBytes(value)
        );
        // (2) 结果保留整行数据
        // 结果同时会保留没有当前列的数据
        SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter(
                // 列族名称
                Bytes.toBytes(columnFamily),
                // 列名
                Bytes.toBytes(columnName),
                // 比较关系
                CompareOperator.EQUAL,
                // 值
                Bytes.toBytes(value)
        );
        // 本身可以添加多个过滤器
        filterList.addFilter(singleColumnValueFilter);
        // 添加过滤
        scan.setFilter(filterList);
        try{
            // 读取多行数据 获得 scanner
            ResultScanner scanner = table.getScanner(scan);
            // result 来记录一行数据 cell 数组
            // ResultScanner 来记录多行数据 result 的数组
            for (Result result : scanner) {
                Cell[] cells = result.rawCells();
                for (Cell cell : cells) {
                    System.out.print(new
                            String(CellUtil.cloneRow(cell)) + "-" + new
                            String(CellUtil.cloneFamily(cell)) + "-" + new
                            String(CellUtil.cloneQualifier(cell)) + "-" + new
                            String(CellUtil.cloneValue(cell)) + "\t");
                }
                System.out.println();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 3. 关闭 table
        table.close();
    }
```

### 3.4.5 删除数据

```
    /**
     * 删除 column 数据
     *
     * @param nameSpace
     * @param tableName
     * @param rowKey
     * @param family
     * @param column
     * @throws IOException
     */
    public static void deleteColumn(String nameSpace, String tableName, String rowKey, String family, String column) throws IOException {
        // 1.获取 table
        Table table = connection.getTable(TableName.valueOf(nameSpace,
                tableName));
        // 2.创建 Delete 对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        // 3.添加删除信息
        // 3.1 删除单个版本
//
        delete.addColumn(Bytes.toBytes(family),Bytes.toBytes(column));
        // 3.2 删除所有版本
        delete.addColumns(Bytes.toBytes(family),
                Bytes.toBytes(column));
        // 3.3 删除列族
// delete.addFamily(Bytes.toBytes(family));
        // 3.删除数据
        table.delete(delete);
        // 5.关闭资源
        table.close();
    }
```

# 第四章：HBase进阶（32-47）

## 4.1 Master 架构

![1659258083887](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659258083887.png)

## 4.2 RegionServer 架构

![1659258537027](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659258537027.png)

## 4.3 写流程

![1659258873726](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659258873726.png)

## 4.4 MemStore Flush

**MemStore 刷写由多个线程控制，条件互相独立：**

主要的刷写规则是控制刷写文件的大小，在每一个刷写线程中都会进行监控
	（1）当某个 memstroe 的大小达到了 hbase.hregion.memstore.flush.size（默认值 128M）， 其所在 region 的所有 memstore 都会刷写。

​	（2）由 HRegionServer 中的属性 MemStoreFlusher 内部线程 FlushHandler 控制。标准LOWER_MARK（低水位线）和 HIGH_MARK（高水位线），意义在于避免写缓存使用过多的内存造成 OOM。

## 4.5 读流程

### 4.5.1 HFile 结构

在了解读流程之前，需要先知道读取的数据是什么样子的。 

​		HFile 是存储在 HDFS 上面每一个 store 文件夹下实际存储数据的文件。里面存储多种内 容。包括数据本身（keyValue 键值对）、元数据记录、文件信息、数据索引、元数据索引和 一个固定长度的尾部信息（记录文件的修改情况）。

​		 键值对按照块大小（默认 64K）保存在文件中，数据索引按照块创建，块越多，索引越 大。每一个 HFile 还会维护一个布隆过滤器（就像是一个很大的地图，文件中每有一种 key， 就在对应的位置标记，读取时可以大致判断要 get 的 key 是否存在 HFile 中）。 

### 4.5.2 读流程

![1659258907770](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659258907770.png)

### 4.5.3 合并读取数据优化



## 4.6 StoreFile Compaction

​		由于 memstore 每次刷写都会生成一个新的 HFile，文件过多读取不方便，所以会进行文件的合并，清理掉过期和删除的数据，会进行 StoreFile Compaction。
​		Compaction 分为两种，分别是 Minor Compaction 和 Major Compaction。Minor Compaction会将临近的若干个较小的 HFile 合并成一个较大的 HFile，并清理掉部分过期和删除的数据，有系统使用一组参数自动控制，Major Compaction 会将一个 Store 下的所有的 HFile 合并成一个大 HFile，并且会清理掉所有过期和删除的数据，由参数 hbase.hregion.majorcompaction控制，默认 7 天。

![1659258919702](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659258919702.png)

## 4.7 Region Split

​		Region 切分分为两种，创建表格时候的预分区即自定义分区，同时系统默认还会启动一 个切分规则，避免单个 Region 中的数据量太大。

### 4.7.1 预分区（自定义分区）

​		每一个 region 维护着 startRow 与 endRowKey，如果加入的数据符合某个 region 维护的 rowKey 范围，则该数据交给这个 region 维护。那么依照这个原则，我们可以将数据所要投 放的分区提前大致的规划好，以提高 HBase 性能。 

### 4.7.2 系统拆分

​		Region 的拆分是由 HRegionServer 完成的，在操作之前需要通过 ZK 汇报 master，修改对应的 Meta 表信息添加两列 info：splitA 和 info：splitB 信息。之后需要操作 HDFS 上面对应的文件，按照拆分后的 Region 范围进行标记区分，实际操作为创建文件引用，不会挪动数据。刚完成拆分的时候，两个 Region 都由原先的 RegionServer 管理。之后汇报给 Master， 由Master将修改后的信息写入到Meta表中。等待下一次触发负载均衡机制，才会修改Region 的管理服务者，而数据要等到下一次压缩时，才会实际进行移动。 