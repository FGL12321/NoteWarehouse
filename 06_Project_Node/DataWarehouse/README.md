# 电商离线数据仓库项目

#### 介绍

![image-20220831234500440](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220831234500440.png)

​		

&emsp;&emsp;**数据仓库（ Data Warehouse ），是为企业制定决策，提供数据支持的。可以帮助企业，改进业务流程、提高产品质量等。**
		数据仓库的输入数据通常包括：业务数据、用户行为数据和爬虫数据等
		**业务数据**：就是各行业在处理事务过程中产生的数据。比如用户在电商网站中登录、下单、支付等 过程中，需要和网站后台数据库进行增删改查交互，产生的数据就是业务数据。业务数据通常存储在MySQL、Oracle等数据库中。

​		**用户行为数据**：用户在使用产品过程中，通过埋点收集与客户端产品交互过程中产生的数据，并发往日志服务器进行保存。比如页面浏览、点击、停留、评论、点赞、收藏等。用户行为数据通常存储在日志文件中。

​		**爬虫数据**：通常事通过技术手段获取其他公司网站的数据。

#### 软件架构

![image-20220901111526317](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901111526317.png)

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
>任务调度：Azkaban,Oozie,DolphinScheduler,Airflow
>
>集群监控：Zabbi，Prometheus
>
>元数据管理：Atlas
>
>权限管理：Ranger,Sentry

