# Hbase Java API的简单使用+原理介绍

​		**本文主要通过Hbase JavaAPI在Hbase中进行建表及插入数据及HBase的DDL和DML，让你更加了解关于Hbase的相关原理。**

如需获取更多源码，笔记，教程，请访问本人仓库，你的关注是我创作的动力！

链接：https://gitee.com/fanggaolei/learning-notes-warehouse



# 1.0 Hbase原理及基本说明

​		==HBase 数据模型的关键在于稀疏、分布式、多维、排序的映射。其中映射 map指代非关系型数据库的 key-Value 结构。==

**Hbase存储数据的原貌：**

```
"office_info":{         #列族
"tel":"010-1111111",    #tel列
"address":"atguigu"     #address列
}
```

## 数据模型介绍

**（1）Name Spase数据模型：相当于Mysql中的database，下面可以存放多张表**



**（2）Table：相当于数据库中的表**



**（3）Row：HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey**

​          **的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要。**



**（4）Column：每个列都由 Column Family(列族)和 Column Qualifier（列限定符）进行限定，例如 info：              			name，info：age。建表时，只需指明列族，而列限定符无需预先定义。**



**（5）Time Stamp：用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，**

​          **其值为写入 HBase 的时间。**



**（6）Cell：由{rowkey, column Family：column Qualifier, timestamp} 唯一确定的单元。cell 中的数**

​           **据全部是字节码形式存贮。**

**架构说明：**

![image-20220823193150880](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220823193151064.png)

**1）Master** 

**实现类为 HMaster，负责监控集群中所有的 RegionServer 实例。主要作用如下：**

（1）管理元数据表格 hbase:meta，接收用户对表格创建修改删除的命令并执行

（2）监控 region 是否需要进行负载均衡，故障转移和 region 的拆分。

  **通过启动多个后台线程监控实现上述功能：**

**①LoadBalancer 负载均衡器**

​		周期性监控 region 分布在 regionServer 上面是否均衡，由参数 hbase.balancer.period 控制周期时间，默认 5 分钟。

**②CatalogJanitor 元数据管理器**

​		定期检查和清理 hbase:meta 中的数据。meta 表内容在进阶中介绍。

**③MasterProcWAL master 预写日志处理器**

​		把 master 需要执行的任务记录到预写日志 WAL 中，如果 master 宕机，让 backupMaster读取日志继续干。

**2）Region Server** 

 **Region Server 实现类为 HRegionServer，主要作用如下:** 

（1）负责数据 cell 的处理，例如写入数据 put，查询数据 get 等 

（2）拆分合并 region 的实际执行者，有 master 监控，有 regionServer 执行。

**3）Zookeeper** 

​		HBase 通过 Zookeeper 来做 master 的高可用、记录 RegionServer 的部署信息、并且存储有 meta 表的位置信息。 HBase 对于数据的读写操作时直接访问 Zookeeper 的，在 2.3 版本推出 Master Registry模式，客户端可以直接访问 master。使用此功能，会加大对 master 的压力，减轻对 Zookeeper的压力。

**4）HDFS** 

​		HDFS 为 Hbase 提供最终的底层数据存储服务，同时为 HBase 提供高容错的支持。

# 2.0HbaseAPI的使用

## 2.1创建连接

​		HBase 的客户端连接由 ConnectionFactory 类来创建，用户使用完成之后需要**手动关闭连接**。同时连接是一个重量级的，推荐**一个进程使用一个连接**，对 **HBase的命令通过连接中的两个属性 Admin 和 Table 来实现**。使用类**单例模式**,确保使用一个连接，**可以同时用于多个线程**。

```java
package com.fang.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import java.io.IOException;


public class HbaseConnection{
    public static Connection connection=null;  //声明一个静态属性
    static {

        //1.创建连接
        //默认使用同步连接

        try {
            //使用读取本地文件的方式添加参数
            connection=ConnectionFactory.createConnection();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void closeConnection() throws IOException{
        //判断连接是否为null
        if (connection!=null){
            connection.close();
        }
    }


    public static void main(String[] args) throws IOException {
        //直接使用创建好的连接，不要在main线程里面单独创建
        System.out.println(HbaseConnection.connection);

        //在main线程的最后记得关闭连接
        HbaseConnection.closeConnection();
    }
}
```

​		==还记得静态代码块的特点吗？**随着类的加载而执行，而且只执行一次**==

​		**静态代码块**：执行优先级高于非静态的初始化块，它会在类初始化的时候执行一次，执行完成便销毁，它仅能初始化类变量

## 2.2创建命名空间

在同一个包下创建HbaseDDL类，执行创建命名空间的功能，使用建造者模式

```java
package com.fang.hbase;

import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;

import java.io.IOException;


public class HBaseDDL {

    //声明一个静态属性，获取连接
    public static Connection connection = HbaseConnection.connection;

    /**
     * 创建命名空间，命名空间名称
     * @param namespase
     * admin的连接是轻量级的，不是线程安全的，不推荐池化或者缓存连接
     */
    public static void createNamespace(String namespace) throws IOException {
        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.调用方法创建命名空间,给一个描述
        //代码相对shell更加底层，shell实现的功能，代码一定能实现


        //2.1创建命名空间描述建造者=》设计师
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(namespace);

        //2.2给命名空间添加需求
        builder.addConfiguration("user","fang");

        //2.3使用builder构造出对饮的创建一个命名空间描述
        admin.createNamespace(builder.build());

        //3.关闭admin
        admin.close();


    }

    public static void main(String[] args) throws IOException {
        //测试创建命名空间
        createNamespace("fang");

        System.out.println("创建成功");

        //关闭Hbase连接
        HbaseConnection.closeConnection();

    }

}
```

执行情况：

![image-20220824181255436](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220824181255436.png)

![image-20220824181330234](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220824181330234.png)

在命令行中查看，创建过程，表示创建成功！！！

==看这里，前面的代码你会发现，如果你再次执行代码就会发现会出现问题，原因是，创建的命名空间已经存在理论来说这个问题应该自己解决，并不是将异常直接抛出，可参考如下代码==

```java
package com.fang.hbase;

import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;

import java.io.IOException;


public class HBaseDDL {

    //声明一个静态属性，获取连接
    public static Connection connection = HbaseConnection.connection;

    /**
     * 创建命名空间，命名空间名称
     * @param namespase
     * admin的连接是轻量级的，不是线程安全的，不推荐池化或者缓存连接
     */
    public static void createNamespace(String namespace) throws IOException {
        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.调用方法创建命名空间,给一个描述
        //代码相对shell更加底层，shell实现的功能，代码一定能实现


        //2.1创建命名空间描述建造者=》设计师
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(namespace);

        //2.2给命名空间添加需求
        builder.addConfiguration("user","fang");

        //2.3使用builder构造出对饮的创建一个命名空间描述
        //创建命名空间出现的问题，都属于
        try {
            admin.createNamespace(builder.build());
        } catch (IOException e) {
            System.out.println("命名空间已经存在");
            e.printStackTrace();
        }

        //3.关闭admin
        admin.close();
    }

    public static void main(String[] args) throws IOException {
        //测试创建命名空间
        createNamespace("fang");

        System.out.println("执行完成");

        //关闭Hbase连接
        HbaseConnection.closeConnection();

    }

}
package com.fang.hbase;

import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;

import java.io.IOException;


public class HBaseDDL {

    //声明一个静态属性，获取连接
    public static Connection connection = HbaseConnection.connection;

    /**
     * 创建命名空间，命名空间名称
     * @param namespase
     * admin的连接是轻量级的，不是线程安全的，不推荐池化或者缓存连接
     */
    public static void createNamespace(String namespace) throws IOException {
        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.调用方法创建命名空间,给一个描述
        //代码相对shell更加底层，shell实现的功能，代码一定能实现


        //2.1创建命名空间描述建造者=》设计师
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(namespace);

        //2.2给命名空间添加需求
        builder.addConfiguration("user","fang");

        //2.3使用builder构造出对饮的创建一个命名空间描述
        //创建命名空间出现的问题，都属于
        try {
            admin.createNamespace(builder.build());
        } catch (IOException e) {
            System.out.println("命名空间已经存在");
            e.printStackTrace();
        }

        //3.关闭admin
        admin.close();
    }

    public static void main(String[] args) throws IOException {
        //测试创建命名空间
        createNamespace("fang");

        System.out.println("执行完成");

        //关闭Hbase连接
        HbaseConnection.closeConnection();

    }

}

```

## 2.3判断表格是否存在

**自行将本代码块，放入上方代码中进行测试即可**

```
   /**
     * 创建表格是否存在
     * @param namespace
     * @return
     * @throws IOException
     * trur表示存在 false表示不存在
     */
    public static boolean isTableExists(String namespace,String tableName) throws IOException {
        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.使用方法判断表格是否存在
        boolean b = false;
        try {
            b = admin.tableExists(TableName.valueOf(namespace,tableName));
        } catch (IOException e) {
            e.printStackTrace();
        }

        admin.close();

        //3.返回结果
        return b;
    }
```

执行结果：

![image-20220824204953468](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220824204953468.png)

## 2.5创建表格（使用HBase1.3.1版本和2.4版本有所区别）

```java
 /**
     * @param tableName 表格名称
     * @param families 列族名称，可以有多个
     */
    public static void createTable(String tableName,String... families) throws IOException {

        //1.获取admin
        Admin admin = connection.getAdmin();

        //2.调用方法创建表格
        //2.1创建表格描述的建造者
        TableName tableName1 = TableName.valueOf(tableName);

        HTableDescriptor tableDescriptor=new HTableDescriptor(tableName); //表格描述器


        if (families==null||families.length==0){ //如果没有填写列族，给一个info
            families=new String[1];
            families[0]="info";

        }

        for (String family : families) {
            HColumnDescriptor columnDescriptor= new HColumnDescriptor(family);//列族描述器

            tableDescriptor.addFamily(columnDescriptor);
        }
        try {
            admin.createTable(tableDescriptor);
        } catch (IOException e) {
            System.out.println("表格已存在");
            e.printStackTrace();
        }

        //3.关闭admin
        admin.close();
    }
```

## 2.6插入数据

```java
package com.fang.hbase;

import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBaseDML {

    public static Connection connection=HbaseConnection.connection;  //声明一个静态属性


    /**
     * @param tableName 表名
     * @param rowKey 主键
     * @param family 列族名
     * @param columnName 列名
     * @param value 值
     */
    public static void putCell(String tableName,String rowKey,String family,String columnName,String value) throws IOException {
        //1.获取table
        Table table = connection.getTable(TableName.valueOf(tableName));

        //2.调用相关方法添加数据
        Put put = new Put(Bytes.toBytes(rowKey));

        //3.给put对象添加属性
        put.addColumn(Bytes.toBytes(family),Bytes.toBytes(columnName),Bytes.toBytes(value));

        //4.将对象写入方法
        table.put(put);

        //5.关闭table
        try {
            table.close();
        } catch (IOException e) {
            System.out.println("表格是否存在");
            e.printStackTrace();
        }

    }

    public static void main(String[] args) throws IOException {

        //测试添加数据
        putCell("shuju:huahua","1000","age","agga","43");
        System.out.println("其他代码");

        //关闭连接
        HbaseConnection.closeConnection();

    }
}

```

测试成功！

![image-20220824230428295](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220824230428295.png)
