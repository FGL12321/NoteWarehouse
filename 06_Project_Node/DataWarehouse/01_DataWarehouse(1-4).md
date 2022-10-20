[TOC]



# 第一部分：用户行为采集平台

# 1.0 数据仓库概念

![image-20220831234500440](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220831234500440.png)

​		**数据仓库（ Data Warehouse ），是为企业制定决策，提供数据支持的。可以帮助企业，改进业务流程、提高产品质量等。**
​		数据仓库的输入数据通常包括：业务数据、用户行为数据和爬虫数据等
​		**业务数据**：就是各行业在处理事务过程中产生的数据。比如用户在电商网站中登录、下单、支付等 过程中，需要和网站后台数据库进行增删改查交互，产生的数据就是业务数据。业务数据通常存储在MySQL、Oracle等数据库中。

​		**用户行为数据**：用户在使用产品过程中，通过埋点收集与客户端产品交互过程中产生的数据，并发往日志服务器进行保存。比如页面浏览、点击、停留、评论、点赞、收藏等。用户行为数据通常存储在日志文件中。

​		**爬虫数据**：通常事通过技术手段获取其他公司网站的数据。

<font color="red">		数据仓库不是数据的最终的目的地，而是为数据最终的目的地做好准备。这些准备包括对数据的：备份，清洗，聚合，统计等</font>

**ODS：备份**

**DWD：清洗**

**DWS：按天存储**

**DWT：累积性存储**

**ADS：报表**

# 2.0 项目 需求及架构设计

## 2.1项目需求

**1、用户行为数据采集平台搭建**

**2、业务数据采集平台搭建**

**<font color="red">3、数据仓库维度建模(核心)</font>**

**4、分析，设备、会员、商品、地区、活动等电商核心主题，统计的报表指标近100个。**

**5、采用即席查询工具，随时进行指标分析**

**6、对集群性能进行监控，发生异常需要报警。**

**7、元数据管理**

**8、质量监控**

**9、权限管理**

## 2.2项目框架

### 2.2.1技术选型

​		**技术选型主要考虑因素：数据量大小、业务需求、行业内经验、技术成熟度、开发维护成本、总成本预算。**

>数据采集传输：Flume,.Kafka,Sqoop;Logstash,DataX
>
>数据存储：MySQL,HDFS,HBase,Redis,MongoDB
>
>数据计算：Hive,Tez,Spark,Flink,Storm
>
>数据查询：Presto,Kylin,Impala,Duid,ClickHouse,Doris
>
>数据可视化：Echats,Superset,QuickBI,DataV
>
>任务调度：Azkaban,Oozie.,DolphinScheduler,Airflow
>
>集群监控：Zabbi这，Prometheus
>
>元数据管理：Atlas
>
>权限管理：Ranger,Sentry

### 2.2.2 系统数据流程设计

![image-20220901111526317](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901111526317.png)

### 2.2.3 框架版本选型



**(1)Apache:运维麻烦，组件间兼容性需要自己调研。（一般大厂使用，技术实力雄厚，有专业的运维人员)（建议使用）**

**(2)CDH:国内使用最多的版本，但CM不开源，今年开始收费，一个节点1万美金/年。**

**(3)HDP:开源，可以进行二次开发，但是没有CDH稳定，国内使用较少**

**2)云服务选择**

> (l)阿里云的EMR、MaxCompute、Data Works

> (2)亚马逊云EMR

> (3)腾讯云MR

> (4)华为云EMR

![image-20220901111846543](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901111846543.png)

### 2.2.4服务器选型


**1)物理机：**

>以128G内存，20核物理CPU,40线程，8THDD和2TSSD硬盘，戴尔品牌
>单台报价4W出头。一般物理机寿命5年左右。
>需要有专业的运维人员，平均一个月1万。电费也是不少的开销。

**2)云主机：**

>云主机：以阿里云为例，差不多相同配置，每年5W。
>很多运维工作都由阿里云完成，运维相对较轻松

**3)企业选择**

>金融有钱公司和阿里没有直接冲突的公司选择阿里云
>中小公司、为了融资上市，选择阿里云，拉倒融资后买物理机。
>有长期打算，资金比较足，选释物理机。
>让天下漫有推学的技

### 2.2.5 集群规模

**1)如何确认集群规模？（假设：每台服务器8T磁盘，128G内存）**

(1)每天日活跃用户100万，每人一天平均100条：100万*100条=1亿条
(2)每条日志1K左右，每天1亿条：100000000/1024/1024=约100G
(3)半年内不扩容服务器来算：100G*180天=约18T
(4)保存3副本：18T*3=54T
(5)预留20%30%Buf=54T0.7=77T
(6)算到这：约8T*10台服务器

**2)如果考虑数仓分层？数据采用压縮？需要重新再计算**



### 2.2.6 集群资源规划设计

​		在企业中通常会搭建一套生产集群和一套测试集群。生产集群运行生产任务，测试集群用于上线前代码编写和测试。

**1）生产集群**

​	（1）消耗内存的分开

​	（2）数据传输数据比较紧密的放在一起（Kafka 、Zookeeper）

​	（3）客户端尽量放在一到两台服务器上，方便外部访问

​	（4）有依赖关系的尽量放到同一台服务器（例如：Hive和Azkaban Executor）

| 1       | 2       | 3     | 4     | 5     | 6    | 7    | 8     | 9     | 10    |
| ------- | ------- | ----- | ----- | ----- | ---- | ---- | ----- | ----- | ----- |
| nn      | nn      | dn    | dn    | dn    | dn   | dn   | dn    | dn    | dn    |
|         |         | rm    | rm    | nm    | nm   | nm   | nm    | nm    | nm    |
|         |         | nm    | nm    |       |      |      |       |       |       |
|         |         |       |       |       |      |      | zk    | zk    | zk    |
|         |         |       |       |       |      |      | kafka | kafka | kafka |
|         |         |       |       |       |      |      | Flume | Flume | flume |
|         |         | Hbase | Hbase | Hbase |      |      |       |       |       |
| hive    | hive    |       |       |       |      |      |       |       |       |
| mysql   | mysql   |       |       |       |      |      |       |       |       |
| spark   | spark   |       |       |       |      |      |       |       |       |
| Azkaban | Azkaban |       |       |       | ES   | ES   |       |       |       |

**2）测试集群服务器规划**

| 服务名称              | 子服务           | 服务器hadoop102 | 服务器hadoop103 | 服务器hadoop104 |
| --------------------- | ---------------- | --------------- | --------------- | --------------- |
| HDFS                  | NameNode         | √               |                 |                 |
| DataNode              | √                | √               | √               |                 |
| SecondaryNameNode     |                  |                 | √               |                 |
| Yarn                  | NodeManager      | √               | √               | √               |
| Resourcemanager       |                  | √               |                 |                 |
| Zookeeper             | Zookeeper Server | √               | √               | √               |
| Flume（采集日志）     | Flume            | √               | √               |                 |
| Kafka                 | Kafka            | √               | √               | √               |
| Flume（消费Kafka）    | Flume            |                 |                 | √               |
| Hive                  | Hive             | √               |                 |                 |
| MySQL                 | MySQL            | √               |                 |                 |
| Sqoop                 | Sqoop            | √               |                 |                 |
| Presto                | Coordinator      | √               |                 |                 |
| Worker                |                  | √               | √               |                 |
| Azkaban               | AzkabanWebServer | √               |                 |                 |
| AzkabanExecutorServer | √                |                 |                 |                 |
| Spark                 |                  | √               |                 |                 |
| Kylin                 |                  | √               |                 |                 |
| HBase                 | HMaster          | √               |                 |                 |
| HRegionServer         | √                | √               | √               |                 |
| Superset              |                  | √               |                 |                 |
| Atlas                 |                  | √               |                 |                 |
| Solr                  | Jar              | √               |                 |                 |
| 服务数总计            |                  | 19              | 8               | 8               |

# 3.0 数据生成模块

## 3.1目标数据

我们要收集和分析的数据主要包括 **页面数据、事件数据、曝光数据、启动数据和错误数据。**

### 3.1.1页面

页面数据主要记录一个页面的用户访问情况，包括访问时间、停留时间、页面路径等信息。

| 字段名称       | 字段描述                                                     |
| -------------- | ------------------------------------------------------------ |
| **page_id**    | 页面id<br/>home（"首页"）,category（"分类页"）,discovery（"发现页"）,<br />top_n（"热门排行"）,favor（"收藏页"）,search（"搜索页"）,<br />good_list（"商品列表页"）,good_detail（"商品详情"）,<br />good_spec（"商品规格"）,comment（"评价"）,<br />comment_done（"评价完成"）,comment_list（"评价列表"）,<br />cart（"购物车"）,trade（"下单结算"）,payment（"支付页面"）,<br />payment_done（"支付完成"）,orders_all（"全部订单"）,<br />orders_unpaid（"订单待支付"）,orders_undelivered<br />（"订单待发货"）,orders_unreceipted（"订单待收货"）,<br />orders_wait_comment（"订单待评价"）,mine（"我的"）,<br />activity（"活动"）,login（"登录"）,register（"注册"）; |
| last_page_id   | 上页id                                                       |
| page_item_type | 页面对象类型<br />sku_id（"商品skuId"）,<br />keyword（"搜索关键词"）,<br />sku_ids（"多个商品skuId"）,<br />activity_id（"活动id"）,<br />coupon_id（"购物券id"）; |
| page_item      | 页面对象id                                                   |
| sourceType     | 页面来源类型<br />promotion（"商品推广"）,<br />recommend（"算法推荐商品"）,<br />query（"查询结果商品"）,<br />activity（"促销活动"）; |
| during_time    | 停留时间（毫秒）                                             |
| ts             | 跳入时间                                                     |

### 3.1.2事件（动作）

事件数据主要记录应用内一个具体操作行为，包括操作类型、操作对象、操作对象描述等信息。

![image-20220902141147836](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902141147836.png)

| 字段名称  | 字段描述                                                     |
| --------- | ------------------------------------------------------------ |
| action_id | 动作idfavor_add（"添加收藏"）,favor_canel（"取消收藏"）,<br />cart_add（"添加购物车"）,cart_remove（"删除购物车"）,<br />cart_add_num（"增加购物车商品数量"）,<br />cart_minus_num（"减少购物车商品数量"）,<br />trade_add_address（"增加收货地址"）,<br />get_coupon（"领取优惠券"）;<br />注：对于下单、支付等业务数据，可从业务数据库获取。 |
| item_type | 动作目标类型sku_id（"商品"）,coupon_id（"购物券"）;          |
| item      | 动作目标id                                                   |
| ts        | 动作时间                                                     |

### 3.1.3曝光

曝光数据主要记录页面所曝光的内容，包括曝光对象，曝光类型等信息。

![image-20220902141202500](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902141202500.png)

| 字段名称    | 字段描述                                                     |
| ----------- | ------------------------------------------------------------ |
| displayType | 曝光类型promotion（"商品推广"）,<br>recommend（"算法推荐商品"）,<br/>query（"查询结果商品"）,<br/>activity（"促销活动"）; |
| item_type   | 曝光对象类型sku_id（"商品skuId"）,activity_id（"活动id"）;   |
| item        | 曝光对象id                                                   |
| order       | 曝光顺序                                                     |

### 3.1.4启动

启动数据记录应用的启动信息。

![image-20220902141228067](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902141228067.png)

| **字段名称**        | **字段描述**                                                 |
| ------------------- | ------------------------------------------------------------ |
| **entry**           | **启动入口icon（"图标"）<br>,notification（"通知"）<br>,install（"安装后启动"）;** |
| **loading_time**    | **启动加载时间**                                             |
| **open_ad_id**      | **开屏广告id**                                               |
| **open_ad_ms**      | **广告播放时间**                                             |
| **open_ad_skip_ms** | **用户跳过广告时间**                                         |
| **ts**              | **启动时间**                                                 |

### 3.1.5错误

**错误数据记录应用使用**

**过程中的错误信息，包括错误编号及错误信息。**

| **字段名称**   | **字段描述** |
| -------------- | ------------ |
| **error_code** | **错误码**   |
| **msg**        | **错误信息** |



## 3.2 数据埋点

![image-20220902142957756](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902142957756.png)

### 3.2.1 主流埋点方式（了解）

**目前主流的埋点方式，有代码埋点（前端/后端）、可视化埋点、全埋点三种。**
		**代码埋点**是通过调用埋点SDK函数，在需要埋点的业务逻辑功能位置调用接口，上报埋点数据。例如，我们对页面中的某个按钮埋点后，当这个按钮被点击时，可以在这个按对应的 OnClick 函数里面调用SDK提供的数据发送接口，来发送数据。

​		**可视化埋点**只需要研发人员集成采集 SDK，不需要写埋点代码，业务人员就可以通过访问分析平台的“圈选”功能，来“圈”出需要对用户行为进行捕捉的控件，并对该事件进行命名。圈选完毕后，这些配置会同步到各个用户的终端上，由采集 SDK 按照圈选的配置自动进行用户行为数据的采集和发送。

​		**全埋点**是通过在产品中嵌入SDK，前端自动采集页面上的全部用户行为事件，上报埋点数据，相当于做了一个统一的埋点。然后再通过界面配置哪些数据需要在系统里面进行分析。

### 3.2.2 埋点数据上报时机

**埋点数据上报时机包括两种方式。**

​		**方式一，在离开该页面时**，上传在这个页面产生的所有数据（页面、事件、曝光、错误等）。优点，批处理，减少了服务器接收数据压力。缺点，不是特别及时。

​		**方式二，每个事件、动作、错误等，产生后，立即发送。优点，响应及时。**缺点，对服务器接收数据压力比较大。

​		本次项目采用方式一埋点。

### 3.2.3 埋点数据日志结构

**我们的日志结构大致可分为两类，一是普通页面埋点日志，二是启动日志。**

​		**普通页面日志结构如下，每条日志包含了，当前页面的页面信息，所有事件（动作）、所有曝光信息以及错误信息。除此之外，还包含了一系列公共信息，包括设备信息，地理位置，应用信息等，即下边的common字段。**

**（1）普通页面埋点日志格式**

```
{
  "common": {                  -- 公共信息
    "ar": "230000",              -- 地区编码
    "ba": "iPhone",              -- 手机品牌
    "ch": "Appstore",            -- 渠道
    "is_new": "1",--是否首日使用，首次使用的当日，该字段值为1，过了24:00，该字段置为0。
	"md": "iPhone 8",            -- 手机型号
    "mid": "YXfhjAYH6As2z9Iq", -- 设备id
    "os": "iOS 13.2.9",          -- 操作系统
    "uid": "485",                 -- 会员id
    "vc": "v2.1.134"             -- app版本号
  },
"actions": [                     --动作(事件)  
    {
      "action_id": "favor_add",   --动作id
      "item": "3",                   --目标id
      "item_type": "sku_id",       --目标类型
      "ts": 1585744376605           --动作时间戳
    }
  ],
  "displays": [
    {
      "displayType": "query",        -- 曝光类型
      "item": "3",                     -- 曝光对象id
      "item_type": "sku_id",         -- 曝光对象类型
      "order": 1,                      --出现顺序
      "pos_id": 2                      --曝光位置
    },
    {
      "displayType": "promotion",
      "item": "6",
      "item_type": "sku_id",
      "order": 2, 
      "pos_id": 1
    },
    {
      "displayType": "promotion",
      "item": "9",
      "item_type": "sku_id",
      "order": 3, 
      "pos_id": 3
    },
    {
      "displayType": "recommend",
      "item": "6",
      "item_type": "sku_id",
      "order": 4, 
      "pos_id": 2
    },
    {
      "displayType": "query ",
      "item": "6",
      "item_type": "sku_id",
      "order": 5, 
      "pos_id": 1
    }
  ],
  "page": {                       --页面信息
    "during_time": 7648,        -- 持续时间毫秒
    "item": "3",                  -- 目标id
    "item_type": "sku_id",      -- 目标类型
    "last_page_id": "login",    -- 上页类型
    "page_id": "good_detail",   -- 页面ID
    "sourceType": "promotion"   -- 来源类型
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744374423  --跳入时间戳
}
```

**（2）启动日志格式**

```
{
  "common": {
    "ar": "370000",
    "ba": "Honor",
    "ch": "wandoujia",
    "is_new": "1",
    "md": "Honor 20s",
    "mid": "eQF5boERMJFOujcp",
    "os": "Android 11.0",
    "uid": "76",
    "vc": "v2.1.134"
  },
  "start": {   
    "entry": "icon",         --icon手机图标  notice 通知   install 安装后启动
    "loading_time": 18803,  --启动加载时间
    "open_ad_id": 7,        --广告页ID
    "open_ad_ms": 3449,    -- 广告总共播放时间
    "open_ad_skip_ms": 1989   --  用户跳过广告时点
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744304000
}
```

**启动日志结构相对简单，主要包含公共信息，启动信息和错误信息。**

## 3.3 服务器和JDK准备

1) **安装模板虚拟机，IP地址192.168.10.100、主机名称hadoop100、内存4G、硬盘50G**

2) **克隆3台虚拟机**
3)  **集群分发脚本xsync**
4) **配置SSH免密登录**
5) **安装JDK**

**以上内容为Hadoop集群配置相关文件，可参考链接：**https://blog.csdn.net/m0_58022371/article/details/126440789

### 3.3.6 环境变量配置说明

​		Linux的环境变量可在多个文件中配置，如/etc/profile，/etc/profile.d/*.sh，/.bash_profile等，下面说明上述几个文件之间的关系和区别。

​		bash的运行模式可分为login shell和non-login shell。

​		例如，我们通过终端，输入用户名、密码，登录系统之后，得到就是一个login shell。而当我们执行以下命令ssh hadoop103 command，在hadoop103执行command的就是一个non-login shell。

![image-20220902144959215](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902144959215.png)

这两种shell的主要区别在于，它们启动时会加载不同的配置文件，login shell启动时会加载/etc/profile，/.bash_profile，~/.bashrc。non-login shell启动时会加载~/.bashrc。
而在加载/.bashrc（实际是~/.bashrc中加载的/etc/bashrc）或/etc/profile时，都会执行如下代码片段

![image-20220902145053443](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220902145053443.png)

<font color="red">**因此不管是login shell还是non-login shell，启动时都会加载/etc/profile.d/*.sh中的环境变量。**</font>

## 3.4 模拟数据

### 3.4.1 使用说明

**1）将application.yml、gmall2020-mock-log-2021-01-22.jar、path.json、logback.xml上传hadoop102的/opt/module/applog目录下**
（1）创建applog路径

```
[atguigu@hadoop102 module]$ mkdir /opt/module/applog
```

（2）上传文件application.yml到/opt/module/applog目录

**2）配置文件**
**（1）application.yml文件**
**可以根据需求生成对应日期的用户行为日志。**

```
[atguigu@hadoop102 applog]$ vim application.yml
```

**修改如下内容**

```
# 外部配置打开
# 外部配置打开
logging.config: "./logback.xml"
#业务日期  注意：并不是Linux系统生成日志的日期，而是生成数据中的时间
mock.date: "2020-06-14"

#模拟数据发送模式
#mock.type: "http"
#mock.type: "kafka"
mock.type: "log"

#http模式下，发送的地址
mock.url: "http://hdp1/applog"

#kafka模式下，发送的地址
mock:
  kafka-server: "hdp1:9092,hdp2:9092,hdp3:9092"
  kafka-topic: "ODS_BASE_LOG"
#启动次数
mock.startup.count: 200
#设备最大值
mock.max.mid: 500000
#会员最大值
mock.max.uid: 100
#商品最大值
mock.max.sku-id: 35
#页面平均访问时间
mock.page.during-time-ms: 20000
#错误概率 百分比
mock.error.rate: 3
#每条日志发送延迟 ms
mock.log.sleep: 10
#商品详情来源  用户查询，商品推广，智能推荐, 促销活动
mock.detail.source-type-rate: "40:25:15:20"
#领取购物券概率
mock.if_get_coupon_rate: 75
#购物券最大id
mock.max.coupon-id: 3
#搜索关键词  
mock.search.keyword: "图书,小米,iphone11,电视,口红,ps5,苹果手机,小米盒子"
```

**（2）path.json，该文件用来配置访问路径**
**根据需求，可以灵活配置用户点击路径。**

```
[
	{"path":["home","good_list","good_detail","cart","trade","payment"],"rate":20 },
	{"path":["home","search","good_list","good_detail","login","good_detail","cart","trade","payment"],"rate":40 },
	{"path":["home","mine","orders_unpaid","trade","payment"],"rate":10 },
	{"path":["home","mine","orders_unpaid","good_detail","good_spec","comment","trade","payment"],"rate":5 },
	{"path":["home","mine","orders_unpaid","good_detail","good_spec","comment","home"],"rate":5 },
	{"path":["home","good_detail"],"rate":10 },
	{"path":["home"  ],"rate":10 }
]
```

**（3）logback配置文件**
**可配置日志生成路径，修改内容如下**

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_HOME" value="/opt/module/applog/log" />
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>

<appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_HOME}/app.%d{yyyy-MM-dd}.log</fileNamePattern>
    </rollingPolicy>
    <encoder>
        <pattern>%msg%n</pattern>
    </encoder>
</appender>

<!-- 将某一个包下日志单独打印日志 -->
<logger name="com.atgugu.gmall2020.mock.log.util.LogUtil"
        level="INFO" additivity="false">
    <appender-ref ref="rollingFile" />
    <appender-ref ref="console" />
</logger>

<root level="error"  >
    <appender-ref ref="console" />
</root>
</configuration>
```

**3）生成日志**
**（1）进入到/opt/module/applog路径，执行以下命令**

```
[atguigu@hadoop102 applog]$ java -jar gmall2020-mock-log-2021-01-22.jar
```

**（2）在/opt/module/applog/log目录下查看生成日志**

```
[atguigu@hadoop102 log]$ ll
```

### 3.4.2集群日志生成脚本

**在hadoop102的/home/atguigu目录下创建bin目录，这样脚本可以在服务器的任何目录执行。**

​	（1）在/home/atguigu/bin目录下创建脚本lg.sh

```
[atguigu@hadoop102 bin]$ vim lg.sh
```

​	（2）在脚本中编写如下内容

```
#!/bin/bash
for i in hadoop102 hadoop103; do
    echo "========== $i =========="
    ssh $i "cd /opt/module/applog/; java -jar gmall2020-mock-log-2021-01-22.jar >/dev/null 2>&1 &"
done 
```

```
注：
①/opt/module/applog/为jar包及配置文件所在路径
②/dev/null代表Linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。
标准输入0：从键盘获得输入 /proc/self/fd/0 
标准输出1：输出到屏幕（即控制台） /proc/self/fd/1 
错误输出2：输出到屏幕（即控制台） /proc/self/fd/2
（3）修改脚本执行权限
[atguigu@hadoop102 bin]$ chmod u+x lg.sh
（4）将jar包及配置文件上传至hadoop103的/opt/module/applog/路径
（5）启动脚本
```

```
[atguigu@hadoop102 module]$ lg.sh 
（6）分别在hadoop102、hadoop103的/opt/module/applog/log目录上查看生成的数据
[atguigu@hadoop102 logs]$ ls
app.2020-06-14.log
[atguigu@hadoop103 logs]$ ls
app.2020-06-14.log
```

# 4.0 数据采集模块（P30-P60）

## 4.1 集群所有进程查看脚本

**（1）在/home/atguigu/bin目录下创建脚本xcall.sh**

```
[atguigu@hadoop102 bin]$ vim xcall.sh
```

​	**（2）在脚本中编写如下内容**

```
#! /bin/bash

for i in hadoop102 hadoop103 hadoop104
do
    echo --------- $i ----------
    ssh $i "$*"
done
```

**（3）修改脚本执行权限**

```
[atguigu@hadoop102 bin]$ chmod 777 xcall.sh
```

**（4）启动脚本**

```
[atguigu@hadoop102 bin]$ xcall.sh jps
```



## <font color="red">**4.2 Hadoop安装（P30-P47）**</font>

<font color="red">**关于Hadoop的安装这里不做详细叙述：如需获取文档教程可访问：**</font>

https://blog.csdn.net/m0_58022371/article/details/126440789

![image-20220903105925352](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903105925352.png)

![image-20220903110817339](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903110817339.png)

![image-20220903110332455](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903110332455.png)

![image-20220903110428214](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903110428214.png)

### 4.2.1 项目经验之HDFS存储多目录（了解）

（1）给Linux系统新增加一块硬盘

参考：https://www.cnblogs.com/yujianadu/p/10750698.html

（2）生产环境服务器磁盘情况

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903110859213.png" alt="image-20220903110859213" style="zoom:150%;" />

（3）在hdfs-site.xml文件中配置多目录，注意新挂载磁盘的访问权限问题

HDFS的DataNode节点保存数据的路径由dfs.datanode.data.dir参数决定，其默认值为file://${hadoop.tmp.dir}/dfs/data，若服务器有多个磁盘，必须对该参数进行修改。如服务器磁盘如上图所示，则该参数应修改为如下的值。

```
<property>
  <name>dfs.datanode.data.dir</name>
<value>file:///dfs/data1,file:///hd2/dfs/data2,file:///hd3/dfs/data3,file:///hd4/dfs/data4</value>
</property>
```

**注意：因为每台服务器节点的磁盘情况不同，所以这个配置配完之后，不需要分发**

### 4.2.2 集群数据均衡

**1）节点间数据均衡**
**（1）开启数据均衡命令**

```
start-balancer.sh -threshold 10
```

对于参数10，代表的是集群中各个节点的磁盘空间利用率相差不超过10%，可根据实际情况进行调整。
**（2）停止数据均衡命令**

```
stop-balancer.sh
```

==注意：于HDFS需要启动单独的Rebalance Server来执行Rebalance操作，所以尽量不要在NameNode上执==

```
行start-balancer.sh
```

所以尽量不要在NameNode上执行start-balancer.sh，而是找一台比较空闲的机器。

**2）磁盘间数据均衡**
**（1）生成均衡计划（我们只有一块磁盘，不会生成计划）**
hdfs diskbalancer -plan hadoop103
**（2）执行均衡计划**
hdfs diskbalancer -execute hadoop103.plan.json
**（3）查看当前均衡任务的执行情况**
hdfs diskbalancer -query hadoop103
**（4）取消均衡任务**
hdfs diskbalancer -cancel hadoop103.plan.json

### 4.2.3 项目经验之支持LZO压缩配置

**1）hadoop-lzo编译**
		hadoop本身并不支持lzo压缩，故需要使用twitter提供的hadoop-lzo开源组件。hadoop-lzo需依赖hadoop和lzo进行编译，编译步骤如下。

**2）将编译好后的hadoop-lzo-0.4.20.jar 放入hadoop-3.1.3/share/hadoop/common/**

```
[atguigu@hadoop102 common]$ pwd
/opt/module/hadoop-3.1.3/share/hadoop/common
[atguigu@hadoop102 common]$ ls
hadoop-lzo-0.4.20.jar
```

**3）同步hadoop-lzo-0.4.20.jar到hadoop103、hadoop104**

```
[atguigu@hadoop102 common]$ xsync hadoop-lzo-0.4.20.jar
```

**4）core-site.xml增加配置支持LZO压缩**

```
<configuration>
    <property>
        <name>io.compression.codecs</name>
        <value>
            org.apache.hadoop.io.compress.GzipCodec,
            org.apache.hadoop.io.compress.DefaultCodec,
            org.apache.hadoop.io.compress.BZip2Codec,
            org.apache.hadoop.io.compress.SnappyCodec,
            com.hadoop.compression.lzo.LzoCodec,
            com.hadoop.compression.lzo.LzopCodec
        </value>
    </property>
	
	<property>
   		 <name>io.compression.codec.lzo.class</name>
   		 <value>com.hadoop.compression.lzo.LzoCodec</value>
	</property>

</configuration>
```

**5）同步core-site.xml到hadoop103、hadoop104**

```
[atguigu@hadoop102 hadoop]$ xsync core-site.xml
```

**6）启动及查看集群**

```
[atguigu@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
```

**7）测试-数据准备**

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir /input
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -put README.txt /input
```

**8）测试-压缩**

```
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.output.fileoutputformat.compress=true -Dmapreduce.output.fileoutputformat.compress.codec=com.hadoop.compression.lzo.LzopCodec  /input /output
```

### 4.2.4 项目经验之LZO创建索引

​		<font color="red">**创建索引的原因是，由于LZO是压缩格式，HDFS在传输运算过程过程中无法 进行切片，创建索引后，则会进行切片。**</font>

**测试**

**（1）将bigtable.lzo（200M）上传到集群的根目录**

```
[atguigu@hadoop102 module]$ hadoop fs -mkdir /input
[atguigu@hadoop102 module]$ hadoop fs -put bigtable.lzo /input
```

**（2）执行wordcount程序**

```
[atguigu@hadoop102 module]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat /input /output1
```

![image-20220903175406127](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175406127.png)

**（3）对上传的LZO文件建索引**

```
[atguigu@hadoop102 module]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar  com.hadoop.compression.lzo.DistributedLzoIndexer /input/bigtable.lzo
```

**（4）再次执行WordCount程序**

```
[atguigu@hadoop102 module]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -Dmapreduce.job.inputformat.class=com.hadoop.mapreduce.LzoTextInputFormat /input /output2
```

![image-20220903175425515](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175425515.png)

**3）注意：如果以上任务，在运行过程中报如下异常**

```
Container [pid=8468,containerID=container_1594198338753_0001_01_000002] is running 318740992B beyond the 'VIRTUAL' memory limit. Current usage: 111.5 MB of 1 GB physical memory used; 2.4 GB of 2.1 GB virtual memory used. Killing container.
Dump of the process-tree for container_1594198338753_0001_01_000002 :
```

​		解决办法：在hadoop102的/opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml文件中增加如下配置，然后分发到hadoop103、hadoop104服务器上，并重新启动集群。

```
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
</property>
```

### 4.2.5 项目经验之基准测试

​		**在企业中非常关心每天从Java后台拉取过来的数据，需要多久能上传到集群？消费者关心多久能从HDFS上拉取需要的数据？**

为了搞清楚HDFS的读写性能，生产环境上非常需要对集群进行压测

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175610282.png" alt="image-20220903175610282" style="zoom:150%;" />

​		HDFS的读写性能主要受**网络和磁盘影响比较大**。为了方便测试，将hadoop102、hadoop103、hadoop104虚拟机网络都设置为100mbps。
​		100Mbps单位是bit；10M/s单位是byte ; 1byte=8bit，100Mbps/8=12.5M/s。

![image-20220903175703339](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175703339.png)

测试网速：

**（1）来到hadoop102的/opt/module目录，创建一个**

[atguigu@hadoop102 software]$ python -m SimpleHTTPServer

**（2）在Web页面上访问**

hadoop102:8000

**1）测试HDFS写性能**
（1）写测试底层原理

![image-20220903175728958](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175728958.png)

**（2）测试内容：向HDFS集群写10个128M的文件**

```
[atguigu@hadoop102 mapreduce]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

2021-02-09 10:43:16,853 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:             Date & time: Tue Feb 09 10:43:16 CST 2021
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:         Number of files: 10
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:  Total MBytes processed: 1280
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:       Throughput mb/sec: 1.61
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:  Average IO rate mb/sec: 1.9
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:   IO rate std deviation: 0.76
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:      Test exec time sec: 133.05
2021-02-09 10:43:16,854 INFO fs.TestDFSIO:
```

​		**注意：nrFiles n为生成mapTask的数量，生产环境一般可通过hadoop103:8088查看CPU核数，设置为（CPU核数 - 1）**

Ø Number of files：生成mapTask数量，一般是集群中（CPU核数 - 1），我们测试虚拟机就按照实际的物理内存-1分配即可。（目标，让每个节点都参与测试）

Ø Total MBytes processed：单个map处理的文件大小

Ø Throughput mb/sec:单个mapTak的吞吐量 

计算方式：处理的总文件大小/每一个mapTask写数据的时间累加

集群整体吞吐量：生成mapTask数量*单个mapTak的吞吐量

Ø Average IO rate mb/sec::平均mapTak的吞吐量

计算方式：每个mapTask处理文件大小/每一个mapTask写数据的时间 

   全部相加除以task数量

Ø IO rate std deviation:方差、反映各个mapTask处理的差值，越小越均衡

注意：如果测试过程中，出现异常

①可以在yarn-site.xml中设置虚拟内存检测为false

```
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
   <name>yarn.nodemanager.vmem-check-enabled</name>
   <value>false</value>
</property>
```

②分发配置并重启Yarn集z群

（3）测试结果分析

​	①由于副本1就在本地，所以该副本不参与测试

![image-20220903175921560](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175921560.png)

一共参与测试的文件：10个文件 * 2个副本 = 20个
压测后的速度：1.61
实测速度：1.61M/s * 20个文件 ≈ 32M/s
三台服务器的带宽：12.5 + 12.5 + 12.5 ≈ 30m/s
所有网络资源都已经用满。
如果实测速度远远小于网络，并且实测速度不能满足工作需求，可以考虑采用固态硬盘或者增加磁盘个数。
	②如果客户端不在集群节点，那就三个副本都参与计算

![image-20220903175943628](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903175943628.png)

**2）测试HDFS读性能**
（1）测试内容：读取HDFS集群10个128M的文件

```
[atguigu@hadoop102 mapreduce]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128MB

2021-02-09 11:34:15,847 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
2021-02-09 11:34:15,847 INFO fs.TestDFSIO:             Date & time: Tue Feb 09 11:34:15 CST 2021
2021-02-09 11:34:15,847 INFO fs.TestDFSIO:         Number of files: 10
2021-02-09 11:34:15,847 INFO fs.TestDFSIO:  Total MBytes processed: 1280
2021-02-09 11:34:15,848 INFO fs.TestDFSIO:       Throughput mb/sec: 200.28
2021-02-09 11:34:15,848 INFO fs.TestDFSIO:  Average IO rate mb/sec: 266.74
2021-02-09 11:34:15,848 INFO fs.TestDFSIO:   IO rate std deviation: 143.12
2021-02-09 11:34:15,848 INFO fs.TestDFSIO:      Test exec time sec: 20.83
```

（2）删除测试生成数据

```
[atguigu@hadoop102 mapreduce]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -clean
```

（3）测试结果分析：为什么读取文件速度大于网络带宽？由于目前只有三台服务器，且有三个副本，数据读取就近原则，相当于都是读取的本地磁盘数据，没有走网络。z

![image-20220903180030164](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903180030164.png)

**3）使用Sort程序评测MapReduce**
（1）使用RandomWriter来产生随机数，每个节点运行10个Map任务，每个Map产生大约1G大小的二进制随机数

```
[atguigu@hadoop102 mapreduce]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar randomwriter random-data
```

（2）执行Sort程序

```
[atguigu@hadoop102 mapreduce]$ hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar sort random-data sorted-data
```


（3）验证数据是否真正排好序了

```
[atguigu@hadoop102 mapreduce]$ 
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar testmapredsort -sortInput random-data -sortOutput sorted-data
```

### 4.2.6 项目经验之Hadoop参数调优

**1）HDFS参数调优hdfs-site.xml**

```
The number of Namenode RPC server threads that listen to requests from clients. If dfs.namenode.servicerpc-address is not configured then Namenode RPC server threads listen to requests from all nodes.
NameNode有一个工作线程池，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作。
对于大集群或者有大量客户端的集群来说，通常需要增大参数dfs.namenode.handler.count的默认值10。
<property>
    <name>dfs.namenode.handler.count</name>
    <value>10</value>
</property>
```

dfs.namenode.handler.count=![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps2.jpg)，**比如集群规模为8台时，此参数设置为41。可通过简单的python代码计算该值，代码如下。**

```
[atguigu@hadoop102 ~]$ python
Python 2.7.5 (default, Apr 11 2018, 07:36:10) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import math
>>> print int(20*math.log(8))
41
>>> quit()
```

**2）YARN参数调优yarn-site.xml**

**（1）情景描述：**

​		总共7台机器，每天几亿条数据，数据源->Flume->Kafka->HDFS->Hive面临问题：数据统计主要用HiveSQL，没有数据倾斜，小文件已经做了合并处理，开启的JVM重用，而且IO没有阻塞，内存用了不到50%。但是还是跑的非常慢，而且数据量洪峰过来时，整个集群都会宕掉。基于这种情况有没有优化方案。

**（2）解决办法：**

NodeManager内存和服务器实际内存配置尽量接近，如服务器有128g内存，但是NodeManager默认内存8G，不修改该参数最多只能用8G内存。NodeManager使用的CPU核数和服务器CPU核数尽量接近。

①yarn.nodemanager.resource.memory-mb	NodeManager使用内存数

②yarn.nodemanager.resource.cpu-vcores	NodeManager使用CPU核数

## <font color="red">4.3 Zookeeper安装（48-49）</font>

### 4.3.1 安装ZK

**内容请参考：[小憨皮全套大数据集群环境配置](https://blog.csdn.net/m0_58022371/article/details/126440789)**

### 4.3.2 ZK集群启动停止脚本

```shell
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
```

## <font color="red">4.4 Kafka安装（50-52）</font>

### 4.4.1 Kafka集群安装

<font color="red">**关于Kafka的安装这里不做详细叙述：如需获取文档教程可访问：**</font>

https://blog.csdn.net/m0_58022371/article/details/126440789

- 1.解压安装包
- 2.在kafka下创建logs文件夹
- 3.修改config/server.properties文件中的相关内容
- 4.配置环境变量，分发Kafka

### 4.4.2 Kafka集群启动停止脚本

**（1）在/home/atguigu/bin目录下创建脚本kf.sh**

```
[atguigu@hadoop102 bin]$ vim kf.sh
```

​	**在脚本中填写如下内容**

```
#! /bin/bash

case $1 in
"start"){
    for i in hadoop102 hadoop103 hadoop104
    do
        echo " --------启动 $i Kafka-------"
        ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
    done
};;
"stop"){
    for i in hadoop102 hadoop103 hadoop104
    do
        echo " --------停止 $i Kafka-------"
        ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh stop"
    done
};;
esac
```

**（2）增加脚本执行权限**

```
[atguigu@hadoop102 bin]$ chmod u+x kf.sh
```

**（3）kf集群启动脚本**

```
[atguigu@hadoop102 module]$ kf.sh start
```

**（4）kf集群停止脚本**

```
[atguigu@hadoop102 module]$ kf.sh stop
```

### 4.4.3 Kafka常用命令

**1）查看Kafka Topic列表**

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka --list
```

**2）创建Kafka Topic**
==进入到/opt/module/kafka/目录下创建日志主题==

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka  --create --replication-factor 1 --partitions 1 --topic topic_log
```

**3）删除Kafka Topic**

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --delete --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka --topic topic_log
```

**4）Kafka生产消息**

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server hadoop102:9092 --from-beginning --topic topic_log
```

**5）Kafka消费消息**

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server hadoop102:9092 --from-beginning --topic topic_log
```

​		==--from-beginning：会把主题中以往所有的数据都读取出来。根据业务场景选择是否增加该配置。==

**6）查看Kafka Topic详情**

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181/kafka \ 
--describe --topic topic_log
```

### 4.4.4 项目经验之Kafka机器数量计算

​		Kafka机器数量（经验公式）= 2 *（峰值生产速度 * 副本数 / 100）+ 1

​		先拿到峰值生产速度，再根据设定的副本数，就能预估出需要部署Kafka的数量。

**1）峰值生产速度**

​		峰值生产速度可以压测得到。

**2）副本数**

​		副本数默认是1个，在企业里面2-3个都有，2个居多。

​		副本多可以提高可靠性，但是会降低网络传输效率。

​		比如我们的峰值生产速度是50M/s。副本数为2。

​		Kafka机器数量 = 2 *（50 * 2 / 100）+ 1 = 3台

### 4.4.5 项目经验之Kafka压力测试

**1）Kafka压测**
		用Kafka官方自带的脚本，对Kafka进行压测。
		kafka-consumer-perf-test.sh
		kafka-producer-perf-test.sh
		Kafka压测时，在硬盘读写速度一定的情况下，可以查看到哪些地方出现了瓶颈（CPU，内存，网络IO）。一般都是网络IO达到瓶颈。 
**2）Kafka Producer压力测试**

![image-20220904102755637](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220904102755637.png)

**（0）压测环境准备**
	①hadoop102、hadoop103、hadoop104的网络带宽都设置为100mbps。
	②关闭hadoop102主机，并根据hadoop102克隆出hadoop105（修改IP和主机名称）
	③hadoop105的带宽不设限
	④创建一个test topic，设置为3个分区2个副本

```
[atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka --create --replication-factor 2 --partitions 3 --topic test
```

**（1）在/opt/module/kafka/bin目录下面有这两个文件。我们来测试一下**

```
[atguigu@hadoop102 kafka]$ bin/kafka-producer-perf-test.sh  --topic test --record-size 100 --num-records 10000000 --throughput -1 --producer-props bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
```

**说明：**
		record-size是一条信息有多大，单位是字节。
		num-records是总共发送多少条信息。
		throughput 是每秒多少条信息，设成-1，表示不限流，尽可能快的生产数据，可测出生产者最大吞吐量。
**（2）Kafka会打印下面的信息**

```
699884 records sent, 139976.8 records/sec (13.35 MB/sec), 1345.6 ms avg latency, 2210.0 ms max latency.
713247 records sent, 141545.3 records/sec (13.50 MB/sec), 1577.4 ms avg latency, 3596.0 ms max latency.
773619 records sent, 153862.2 records/sec (14.67 MB/sec), 2326.8 ms avg latency, 4051.0 ms max latency.
773961 records sent, 154206.2 records/sec (15.71 MB/sec), 1964.1 ms avg latency, 2917.0 ms max latency.
776970 records sent, 154559.4 records/sec (15.74 MB/sec), 1960.2 ms avg latency, 2922.0 ms max latency.
776421 records sent, 154727.2 records/sec (15.76 MB/sec), 1960.4 ms avg latency, 2954.0 ms max latency.
```

参数解析：Kafka的吞吐量15m/s左右是否符合预期呢？
		hadoop102、hadoop103、hadoop104三台集群的网络总带宽30m/s左右，由于是两个副本，所以Kafka的吞吐量30m/s ➗ 2（副本） = 15m/s 
		==结论：网络带宽和副本都会影响吞吐量。==
**（4）调整batch.size**
		batch.size默认值是16k。
		batch.size较小，会降低吞吐量。比如说，批次大小为0则完全禁用批处理，会一条一条发送消息）；
		batch.size过大，会增加消息发送延迟。比如说，Batch设置为64k，但是要等待5秒钟Batch才凑满了64k，才能发送出去。那这条消息的延迟就是5秒钟。

```
[atguigu@hadoop102 kafka]$ bin/kafka-producer-perf-test.sh  --topic test --record-size 100 --num-records 10000000 --throughput -1 --producer-props bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092 batch.size=500
```

输出结果

```
69169 records sent, 13833.8 records/sec (1.32 MB/sec), 2517.6 ms avg latency, 4299.0 ms max latency.
105372 records sent, 21074.4 records/sec (2.01 MB/sec), 6748.4 ms avg latency, 9016.0 ms max latency.
113188 records sent, 22637.6 records/sec (2.16 MB/sec), 11348.0 ms avg latency, 13196.0 ms max latency.
108896 records sent, 21779.2 records/sec (2.08 MB/sec), 12272.6 ms avg latency, 12870.0 ms max latency.
```

**（5）linger.ms**
		如果设置batch size为64k，但是比如过了10分钟也没有凑够64k，怎么办？
		可以设置，linger.ms。比如linger.ms=5ms，那么就是要发送的数据没有到64k，5ms后，数据也会发出去。
**（6）总结**
		同时设置batch.size和 linger.ms，就是哪个条件先满足就都会将消息发送出去
Kafka需要考虑高吞吐量与延时的平衡。

**3）Kafka Consumer压力测试**

**（1）Consumer的测试，如果这四个指标（IO，CPU，内存，网络）都不能改变，考虑增加分区数来提升性能。**

```
[atguigu@hadoop102 kafka]$ bin/kafka-consumer-perf-test.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic test --fetch-size 10000 --messages 10000000 --threads 1
```

①参数说明：
--broker-list指定Kafka集群地址
--topic 指定topic的名称
--fetch-size 指定每次fetch的数据的大小
--messages 总共要消费的消息个数
②测试结果说明：

```
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
2021-08-03 21:17:21:778, 2021-08-03 21:18:19:775, 514.7169, 8.8749, 5397198, 93059.9514
```

​		开始测试时间，测试结束数据，共消费数据514.7169MB，吞吐量8.8749MB/s
**（2）调整fetch-size**
①增加fetch-size值，观察消费吞吐量。

```
[atguigu@hadoop102 kafka]$ bin/kafka-consumer-perf-test.sh --broker-list hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic test --fetch-size 100000 --messages 10000000 --threads 1
```

②测试结果说明：

```
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec
2021-08-03 21:22:57:671, 2021-08-03 21:23:41:938, 514.7169, 11.6276, 5397198, 121923.7355
```

**（3）总结**
==吞吐量受网络带宽和fetch-size的影响==

### 4.4.6 项目经验值Kafka分区数计算

​	（1）创建一个只有1个分区的topic

​	（2）测试这个topic的producer吞吐量和consumer吞吐量。

​	（3）假设他们的值分别是Tp和Tc，单位可以是MB/s。

​	（4）然后假设总的目标吞吐量是Tt，那么分区数 = Tt / min（Tp，Tc）

例如：producer吞吐量 = 20m/s；consumer吞吐量 = 50m/s，期望吞吐量100m/s；

分区数 = 100 / 20 = 5分区

https://blog.csdn.net/weixin_42641909/article/details/89294698

分区数一般设置为：3-10个

## <font color="red">4.5 采集日志Flume（53-56）</font>

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps3.png" alt="img" style="zoom:150%;" />

### 4.5.1 日志采集Flume安装

<font color="red">**关于Flume的安装这里不做详细叙述：如需获取文档教程可访问：**</font>

https://blog.csdn.net/m0_58022371/article/details/126440789

![image-20220904103958951](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220904103958951.png)

- 

### 4.5.2 项目经验之Flume组件选型

**1）Source**
**（1）Taildir Source相比Exec Source、Spooling Directory Source的优势**
		TailDir Source：断点续传、多目录。Flume1.6以前需要自己自定义Source记录每次读取文件位置，实现断点续传。不会丢数据，但是有可能会导致数据重复。
Exec Source可以实时搜集数据，但是在Flume不运行或者Shell命令出错的情况下，数据将会丢失。
		Spooling Directory Source监控目录，支持断点续传。
**（2）batchSize大小如何设置？**
==答：Event 1K左右时，500-1000合适（默认为100）==

- **Exec source：适用于监控一个实时追加的文件，不能实现断点续传**
- **Spooldir Source ：适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步**
- **Taildir Source：适合用于监听多个实时追加的文件，并且能够实现断点续传。**

**2）Channel**
		采用Kafka Channel，省去了Sink，提高了效率。==KafkaChannel数据存储在Kafka里面==，所以数据是存储在磁盘中。
		注意在Flume1.7以前，Kafka Channel很少有人使用，因为发现parseAsFlumeEvent这个配置起不了作用。也就是无论parseAsFlumeEvent配置为true还是false，都会转为Flume Event。这样的话，造成的结果是，会始终都把Flume的headers中的信息混合着内容一起写入Kafka的消息中，这显然不是我所需要的，我只是需要把内容写入即可。

### 4.5.3 日志采集Flume配置

**1）Flume配置分析**



![image-20220904191204599](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220904191204599.png)

​	Flume直接读log日志的数据，log日志的格式是app.yyyy-mm-dd.log。

**2）Flume的具体配置如下：**

**（1）在/opt/module/flume/conf目录下创建file-flume-kafka.conf文件**

```
[atguigu@hadoop102 conf]$ vim file-flume-kafka.conf
```

**在文件配置如下内容**

```
#为各组件命名
a1.sources = r1
a1.channels = c1

#描述source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/applog/log/app.* #监控文件名称
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json #断点续传
a1.sources.r1.interceptors =  i1 #配置拦截器
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.ETLInterceptor$Builder# 配置拦截器类型

#描述channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel #类型
a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092 #连接
a1.channels.c1.kafka.topic = topic_log  #数据发送到kafkatopic_log
a1.channels.c1.parseAsFlumeEvent = false #将传输的数据截去头留下身体

#绑定source和channel以及sink和channel的关系
a1.sources.r1.channels = c1
```

### 4.5.4 Flume拦截器

**1）创建Maven工程flume-interceptor**

**2）创建包名：com.atguigu.flume.interceptor**

**3）在pom.xml文件中添加如下配置**

```
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.9.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
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

​		==注意：scope中provided的含义是编译时用该jar包。打包时不用。因为集群上已经存在flume的jar包。只是本地编译时用一下。==

**4）在com.atguigu.flume.interceptor包下创建JSONUtils类**

```java
package com.atguigu.flume.interceptor;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONException;

public class JSONUtils {
    public static boolean isJSONValidate(String log){
        try {
            JSON.parse(log);
            return true;
        }catch (JSONException e){
            return false;
        }
    }
}
```

5）在com.atguigu.flume.interceptor包下创建LogInterceptor类

```java
package com.atguigu.flume.interceptor;

import com.alibaba.fastjson.JSON;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.StandardCharsets;
import java.util.Iterator;
import java.util.List;

public class ETLInterceptor implements Interceptor {

    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {

        byte[] body = event.getBody();
        String log = new String(body, StandardCharsets.UTF_8);

        if (JSONUtils.isJSONValidate(log)) {
            return event;
        } else {
            return null;
        }
    }

    @Override
    public List<Event> intercept(List<Event> list) {

        Iterator<Event> iterator = list.iterator();

        while (iterator.hasNext()){
            Event next = iterator.next();
            if(intercept(next)==null){
                iterator.remove();
            }
        }

        return list;
    }

    public static class Builder implements Interceptor.Builder{

        @Override
        public Interceptor build() {
            return new ETLInterceptor();
        }
        @Override
        public void configure(Context context) {

        }

    }

    @Override
    public void close() {

    }
}
```

**6）打包**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps1.jpg) 

**7）需要先将打好的包放入到hadoop102的/opt/module/flume/lib文件夹下面。**

```
[atguigu@hadoop102 lib]$ ls | grep interceptor

flume-interceptor-1.0-SNAPSHOT-jar-with-dependencies.jar
```

**8）分发Flume到hadoop103、hadoop104**

```
[atguigu@hadoop102 module]$ xsync flume/
```

**9）分别在hadoop102、hadoop103上启动Flume**

```
[atguigu@hadoop102 flume]$ bin/flume-ng agent --name a1 --conf-file conf/file-flume-kafka.conf &

[atguigu@hadoop103 flume]$ bin/flume-ng agent --name a1 --conf-file conf/file-flume-kafka.conf &
```

### 4.5.5 测试Flume-Kafka通道

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps3.png" alt="img" style="zoom:150%;" />

**（1）生成日志**

```
[atguigu@hadoop102 ~]$ lg.sh
```

**（2）消费Kafka数据，观察控制台是否有数据获取到**

```
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server hadoop102:9092 --from-beginning --topic topic_log
```

​		**说明：如果获取不到数据，先检查Kafka、Flume、Zookeeper是否都正确启动。再检查Flume的拦截器代码是否正常。**

### 4.5.6 日志采集Flume启动停止脚本

**（1）在/home/atguigu/bin目录下创建脚本f1.sh**

```
[atguigu@hadoop102 bin]$ vim f1.sh
```

​	在脚本中填写如下内容

```shell
#! /bin/bash

case $1 in
"start"){
        for i in hadoop102 hadoop103
        do
                echo " --------启动 $i 采集flume-------"
                ssh $i "nohup /opt/module/flume/bin/flume-ng agent --conf-file /opt/module/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/opt/module/flume/log1.txt 2>&1  &"
        done
};;	
"stop"){
        for i in hadoop102 hadoop103
        do
                echo " --------停止 $i 采集flume-------"
                ssh $i "ps -ef | grep file-flume-kafka | grep -v grep |awk  '{print \$2}' | xargs -n1 kill -9 "
        done

};;
esac
```

说明1：nohup，该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。nohup就是不挂起的意思，不挂断地运行命令。

说明2：awk 默认分隔符为空格

说明3：$2是在“”双引号内部会被解析为脚本的第二个参数，但是这里面想表达的含义是awk的第二个值，所以需要将他转义，用\$2表示。

说明4：xargs 表示取出前面命令运行的结果，作为后面命令的输入参数。

**（2）增加脚本执行权限**

```
[atguigu@hadoop102 bin]$ chmod u+x f1.sh
```

**（3）f1集群启动脚本**

```
[atguigu@hadoop102 module]$ f1.sh start
```

**（4）f1集群停止脚本**

```
[atguigu@hadoop102 module]$ f1.sh stop
```



## <font color="red">4.6 消费Kafka数据Flume（57-60）</font>

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps5.png" alt="img" style="zoom:150%;" />

### 4.6.1 项目经验之Flume组件选型

**1）FileChannel和MemoryChannel区别**
		MemoryChannel传输数据速度更快，但因为数据保存在JVM的堆内存中，Agent进程挂掉会导致数据丢失，适用于对数据质量要求不高的需求。
		FileChannel传输速度相对于Memory慢，但数据安全保障高，Agent进程挂掉也可以从失败中恢复数据。
选型：
		金融类公司、对钱要求非常准确的公司通常会选择FileChannel
		传输的是普通日志信息（京东内部一天丢100万-200万条，这是非常正常的），通常选择MemoryChannel。
**2）FileChannel优化**
		通过配置dataDirs指向多个路径，每个路径对应不同的硬盘，增大Flume吞吐量。

官方说明如下：

​		Comma separated list of directories for storing log files. Using multiple directories on separate disks can improve file channel peformance

​		checkpointDir和backupCheckpointDir也尽量配置在不同硬盘对应的目录中，保证checkpoint坏掉后，可以快速使用backupCheckpointDir恢复数据。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps6.png" alt="img" style="zoom:150%;" />

**3）Sink：HDFS Sink**
**（1）HDFS存入大量小文件，有什么影响？**
		元数据层面：每个小文件都有一份元数据，其中包括文件路径，文件名，所有者，所属组，权限，创建时间等，这些信息都保存在Namenode内存中。所以小文件过多，会占用Namenode服务器大量内存，影响Namenode性能和使用寿命
计算层面：默认情况下MR会对每个小文件启用一个Map任务计算，非常影响计算性能。同时也影响磁盘寻址时间。
**（2）HDFS小文件处理**
		官方默认的这三个参数配置写入HDFS后会产生小文件，hdfs.rollInterval、hdfs.rollSize、hdfs.rollCount基于以上hdfs.rollInterval=3600，hdfs.rollSize=134217728，hdfs.rollCount =0几个参数综合作用，效果如下：
①文件在达到128M时会滚动生成新文件
②文件创建超3600秒时会滚动生成新文件

### 4.6.2 消费者Flume配置

**1）Flume配置分析**

![image-20220904192143305](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220904192143305.png)

**2）Flume的具体配置如下：**

（1）在hadoop104的/opt/module/flume/conf目录下创建kafka-flume-hdfs.conf文件

```
[atguigu@hadoop104 conf]$ vim kafka-flume-hdfs.conf
```

在文件配置如下内容：

```
## 组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics=topic_log
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.TimeStampInterceptor$Builder

## channel1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1/


## sink1
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log-
a1.sinks.k1.hdfs.round = false

#控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

## 控制输出文件是原生文件。
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = lzop

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1
```

### 4.6.3 Flume时间戳拦截器

​		由于**Flume默认会用Linux系统时间，作为输出到HDFS路径的时间。如果数据是23:59分产生的。Flume消费Kafka里面的数据时，有可能已经是第二天了，那么这部门数据会被发往第二天的HDFS路径。我们希望的是根据日志里面的实际时间，发往HDFS的路径，所以下面拦截器作用是获取日志中的实际时间。**
​		解决的思路：拦截json日志，通过fastjson框架解析json，获取实际时间ts。将获取的ts时间写入拦截器header头，header的key必须是timestamp，因为Flume框架会根据这个key的值识别为时间，写入到HDFS。
1）在com.atguigu.flume.interceptor包下创建TimeStampInterceptor类

```java
package com.atguigu.flume.interceptor;

import com.alibaba.fastjson.JSONObject;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class TimeStampInterceptor implements Interceptor {

    private ArrayList<Event> events = new ArrayList<>();

    @Override
    public void initialize() {

    }

    @Override
    public Event intercept(Event event) {

        Map<String, String> headers = event.getHeaders();
        String log = new String(event.getBody(), StandardCharsets.UTF_8);

        JSONObject jsonObject = JSONObject.parseObject(log);

        String ts = jsonObject.getString("ts");
        headers.put("timestamp", ts);

        return event;
    }

    @Override
    public List<Event> intercept(List<Event> list) {
        events.clear();
        for (Event event : list) {
            events.add(intercept(event));
        }

        return events;
    }

    @Override
    public void close() {

    }

    public static class Builder implements Interceptor.Builder {
        @Override
        public Interceptor build() {
            return new TimeStampInterceptor();
        }

        @Override
        public void configure(Context context) {
        }
    }
}
```

2）重新打包

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps7.jpg) 

3）需要先将打好的包放入到hadoop102的/opt/module/flume/lib文件夹下面。

```
[atguigu@hadoop102 lib]$ ls | grep interceptor
flume-interceptor-1.0-SNAPSHOT-jar-with-dependencies.jar
```

4）分发Flume到hadoop103、hadoop104

```
[atguigu@hadoop102 module]$ xsync flume/
```

### 4.6.4 消费者Flume启动停止脚本

（1）在/home/atguigu/bin目录下创建脚本f2.sh

```
[atguigu@hadoop102 bin]$ vim f2.sh
```

（1）在/home/atguigu/bin目录下创建脚本f2.sh

```shell
#! /bin/bash

case $1 in
"start"){
        for i in hadoop104
        do
                echo " --------启动 $i 消费flume-------"
                ssh $i "nohup /opt/module/flume/bin/flume-ng agent --conf-file /opt/module/flume/conf/kafka-flume-hdfs.conf --name a1 -Dflume.root.logger=INFO,LOGFILE >/opt/module/flume/log2.txt   2>&1 &"
        done
};;
"stop"){
        for i in hadoop104
        do
                echo " --------停止 $i 消费flume-------"
                ssh $i "ps -ef | grep kafka-flume-hdfs | grep -v grep |awk '{print \$2}' | xargs -n1 kill"
        done

};;
esac
```

（2）增加脚本执行权限

```
[atguigu@hadoop102 bin]$ chmod u+x f2.sh
```

（3）f2集群启动脚本

```
[atguigu@hadoop102 module]$ f2.sh start
```

（4）f2集群停止脚本

```
[atguigu@hadoop102 module]$ f2.sh stop
```

### 4.6.5 项目经验之Flume内存优化

**1）问题描述：如果启动消费Flume抛出如下异常**
ERROR hdfs.HDFSEventSink: process failed
java.lang.OutOfMemoryError: GC overhead limit exceeded

**2）解决方案步骤**
（1）在hadoop102服务器的/opt/module/flume/conf/flume-env.sh文件中增加如下配置

```
export JAVA_OPTS="-Xms100m -Xmx2000m -Dcom.sun.management.jmxremote"
```

（2）同步配置到hadoop103、hadoop104服务器

```
[atguigu@hadoop102 conf]$ xsync flume-env.sh
```

**3）Flume内存参数设置及优化**
	JVM heap一般设置为4G或更高
	-Xmx与-Xms最好设置一致，减少内存抖动带来的性能影响，如果设置不一致容易导致频繁fullgc。
	-Xms表示JVM Heap（堆内存）最小尺寸，初始分配；-Xmx 表示JVM Heap(堆内存)最大允许的尺寸，按	需分配。如果不设置一致，容易在初始化时，由于内存不够，频繁触发fullgc。



## 4.7 采集通道启动/停止脚本

（1）在/home/atguigu/bin目录下创建脚本cluster.sh

```
[atguigu@hadoop102 bin]$ vim cluster.sh
```

在脚本中填写如下内容

```shell
#!/bin/bash

case $1 in
"start"){
        echo ================== 启动 集群 ==================

        #启动 Zookeeper集群
        zk.sh start

        #启动 Hadoop集群
        hdp.sh start

        #启动 Kafka采集集群
        kf.sh start

        #启动 Flume采集集群
        f1.sh start

        #启动 Flume消费集群
        f2.sh start

        };;
"stop"){
        echo ================== 停止 集群 ==================

        #停止 Flume消费集群
        f2.sh stop

        #停止 Flume采集集群
        f1.sh stop

        #停止 Kafka采集集群
        kf.sh stop

        #停止 Hadoop集群
        hdp.sh stop

        #停止 Zookeeper集群
        zk.sh stop

};;
esac
```

（2）增加脚本执行权限

```
[atguigu@hadoop102 bin]$ chmod u+x cluster.sh	
```

（3）cluster集群启动脚本

```
[atguigu@hadoop102 module]$ cluster.sh start
```

（4）cluster集群停止脚本

```
[atguigu@hadoop102 module]$ cluster.sh stop
```



==常见问题与解决方案：==

**1）问题描述**

访问2NN页面[http://hadoop104:9868](http://hadoop104:9868/)，看不到详细信息

**2）解决办法**

 （1）在浏览器上按F12，查看问题原因。定位bug在61行

 （2）找到要修改的文件

```
[atguigu@hadoop102 static]$ pwd
/opt/module/hadoop-3.1.3/share/hadoop/hdfs/webapps/static

[atguigu@hadoop102 static]$ vim dfs-dust.js
:set nu
修改61行 
return new Date(Number(v)).toLocaleString();
```

 （3）分发dfs-dust.js

```
[atguigu@hadoop102 static]$ xsync dfs-dust.js
```

 （4）在http://hadoop104:9868/status.html 页面强制刷新

