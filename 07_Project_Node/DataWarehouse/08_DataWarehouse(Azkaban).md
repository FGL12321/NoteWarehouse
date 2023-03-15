[TOC]

# 17.0 全流程调度

## 17.1 Azkaban部署

**详情见本人博客：**https://blog.csdn.net/m0_58022371/article/details/127110533

## 17.2 创建MySQL数据库和表	

![image-20220929224233998](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220929224233998.png)

**注:SQL语句：**

```
CREATE DATABASE `gmall_report` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
```

**2）创建表**

**（1）访客统计**

```sql
DROP TABLE IF EXISTS ads_visit_stats;
CREATE TABLE `ads_visit_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `is_new` VARCHAR(255) NOT NULL COMMENT '新老标识,1:新,0:老',
  `recent_days` INT NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `channel` VARCHAR(255) NOT NULL COMMENT '渠道',
  `uv_count` BIGINT(20) DEFAULT NULL COMMENT '日活(访问人数)',
  `duration_sec` BIGINT(20) DEFAULT NULL COMMENT '页面停留总时长',
  `avg_duration_sec` BIGINT(20)  DEFAULT NULL COMMENT '一次会话，页面停留平均时长',
  `page_count` BIGINT(20) DEFAULT NULL COMMENT '页面总浏览数',
  `avg_page_count` BIGINT(20) DEFAULT NULL COMMENT '一次会话，页面平均浏览数',
  `sv_count` BIGINT(20) DEFAULT NULL COMMENT '会话次数',
  `bounce_count` BIGINT(20) DEFAULT NULL COMMENT '跳出数',
  `bounce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '跳出率',
  PRIMARY KEY (`dt`,`recent_days`,`is_new`,`channel`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

**（2）页面路径分析**

```sql
DROP TABLE IF EXISTS ads_page_path;
CREATE TABLE `ads_page_path` (      
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `source` VARCHAR(255) DEFAULT NULL COMMENT '跳转起始页面',
  `target` VARCHAR(255) DEFAULT NULL COMMENT '跳转终到页面',
  `path_count` BIGINT(255) DEFAULT NULL COMMENT '跳转次数',
  UNIQUE KEY (`dt`,`recent_days`,`source`,`target`) USING BTREE     
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（3）用户统计**

```sql
DROP TABLE IF EXISTS ads_user_total;
CREATE TABLE `ads_user_total` (          
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,0:累积值,1:最近1天,7:最近7天,30:最近30天',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新注册用户数',
  `new_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增下单用户数',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '下单总金额',
  `order_user_count` BIGINT(20) DEFAULT NULL COMMENT '下单用户数',
  `no_order_user_count` BIGINT(20) DEFAULT NULL COMMENT '未下单用户数(具体指活跃用户中未下单用户)',
  PRIMARY KEY (`dt`,`recent_days`)           
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

**（4）用户变动统计**

```sql
DROP TABLE IF EXISTS ads_user_change;
CREATE TABLE `ads_user_change` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `user_churn_count` BIGINT(20) DEFAULT NULL  COMMENT '流失用户数',
  `user_back_count` BIGINT(20) DEFAULT NULL  COMMENT '回流用户数',
  PRIMARY KEY (`dt`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

**（5）用户行为漏斗分析**

```sql
DROP TABLE IF EXISTS ads_user_action;
CREATE TABLE `ads_user_action` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `home_count` BIGINT(20) DEFAULT NULL COMMENT '浏览首页人数',
  `good_detail_count` BIGINT(20) DEFAULT NULL COMMENT '浏览商品详情页人数',
  `cart_count` BIGINT(20) DEFAULT NULL COMMENT '加入购物车人数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '下单人数',
  `payment_count` BIGINT(20) DEFAULT NULL COMMENT '支付人数',
  PRIMARY KEY (`dt`,`recent_days`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（6）用户留存率分析**

```sql
DROP TABLE IF EXISTS ads_user_retention;
CREATE TABLE `ads_user_retention` (      
  `dt` DATE DEFAULT NULL COMMENT '统计日期',
  `create_date` VARCHAR(255) NOT NULL COMMENT '用户新增日期',
  `retention_day` BIGINT(20) NOT NULL COMMENT '截至当前日期留存天数',
  `retention_count` BIGINT(20) DEFAULT NULL COMMENT '留存用户数量',
  `new_user_count` BIGINT(20) DEFAULT NULL COMMENT '新增用户数量',
  `retention_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '留存率',
  PRIMARY KEY (`create_date`,`retention_day`) USING BTREE        
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（7）订单统计**

```sql
DROP TABLE IF EXISTS ads_order_total;
 CREATE TABLE `ads_order_total` (   
  `dt` DATE NOT NULL COMMENT '统计日期', 
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `order_count` BIGINT(255) DEFAULT NULL COMMENT '订单数', 
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  `order_user_count` BIGINT(255) DEFAULT NULL COMMENT '下单人数',
  PRIMARY KEY (`dt`,`recent_days`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（8）各省份订单统计**

```sql
DROP TABLE IF EXISTS ads_order_by_province;
CREATE TABLE `ads_order_by_province` (
  `dt` DATE NOT NULL,
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `province_id` VARCHAR(255) NOT NULL COMMENT '统计日期',
  `province_name` VARCHAR(255) DEFAULT NULL COMMENT '省份名称',
  `area_code` VARCHAR(255) DEFAULT NULL COMMENT '地区编码',
  `iso_code` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `iso_code_3166_2` VARCHAR(255) DEFAULT NULL COMMENT '国际标准地区编码',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额',
  PRIMARY KEY (`dt`, `recent_days` ,`province_id`) USING BTREE       
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（9）品牌复购率**

```sql
DROP TABLE IF EXISTS ads_repeat_purchase;
CREATE TABLE `ads_repeat_purchase` (         
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `order_repeat_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '复购率',
  PRIMARY KEY (`dt` ,`recent_days`,`tm_id`)          
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（10）商品统计**

```sql
DROP TABLE IF EXISTS ads_order_spu_stats;
CREATE TABLE `ads_order_spu_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `recent_days` BIGINT(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `spu_id` VARCHAR(255) NOT NULL COMMENT '商品ID',
  `spu_name` VARCHAR(255) DEFAULT NULL COMMENT '商品名称',
  `tm_id` VARCHAR(255) NOT NULL COMMENT '品牌ID',
  `tm_name` VARCHAR(255) DEFAULT NULL COMMENT '品牌名称',
  `category3_id` VARCHAR(255) NOT NULL COMMENT '三级品类ID',
  `category3_name` VARCHAR(255) DEFAULT NULL COMMENT '三级品类名称',
  `category2_id` VARCHAR(255) NOT NULL COMMENT '二级品类ID',
  `category2_name` VARCHAR(255) DEFAULT NULL COMMENT '二级品类名称',
  `category1_id` VARCHAR(255) NOT NULL COMMENT '一级品类ID',
  `category1_name` VARCHAR(255) NOT NULL COMMENT '一级品类名称',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '订单数',
  `order_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '订单金额', 
  PRIMARY KEY (`dt`,`recent_days`,`spu_id`)  
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

**（11）活动统计**

```sql
DROP TABLE IF EXISTS ads_activity_stats;
CREATE TABLE `ads_activity_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `activity_id` VARCHAR(255) NOT NULL COMMENT '活动ID',
  `activity_name` VARCHAR(255) DEFAULT NULL COMMENT '活动名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',
  `order_count` BIGINT(11) DEFAULT NULL COMMENT '参与活动订单数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '参与活动订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`activity_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

**（12）优惠券统计**

```sql
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE TABLE `ads_coupon_stats` (
  `dt` DATE NOT NULL COMMENT '统计日期',
  `coupon_id` VARCHAR(255) NOT NULL COMMENT '优惠券ID',
  `coupon_name` VARCHAR(255) DEFAULT NULL COMMENT '优惠券名称',
  `start_date` DATE DEFAULT NULL COMMENT '开始日期',  
  `rule_name`  VARCHAR(200) DEFAULT NULL COMMENT '优惠规则',
  `get_count`  BIGINT(20) DEFAULT NULL COMMENT '领取次数',
  `order_count` BIGINT(20) DEFAULT NULL COMMENT '使用(下单)次数',
  `expire_count`  BIGINT(20) DEFAULT NULL COMMENT '过期次数',
  `order_original_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单原始金额',
  `order_final_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '使用优惠券订单最终金额',
  `reduce_amount` DECIMAL(16,2) DEFAULT NULL COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) DEFAULT NULL COMMENT '补贴率',
  PRIMARY KEY (`dt`,`coupon_id` )
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

## 17.3 Sqoop导出脚本	

1）编写Sqoop导出脚本

在/home/atguigu/bin目录下创建脚本hdfs_to_mysql.sh

```
[atguigu@hadoop102 bin]$ vim hdfs_to_mysql.sh
```

​    在脚本中填写如下内容

```sql
#!/bin/bash

hive_db_name=gmall
mysql_db_name=gmall_report

export_data() {
/opt/module/sqoop/bin/sqoop export \
--connect "jdbc:mysql://hadoop102:3306/${mysql_db_name}?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password 000000 \
--table $1 \
--num-mappers 1 \
--export-dir /warehouse/$hive_db_name/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $2 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}

case $1 in
  "ads_activity_stats" )
    export_data "ads_activity_stats" "dt,activity_id"
  ;;

  "ads_coupon_stats" )
    export_data "ads_coupon_stats" "dt,coupon_id"
  ;;

  "ads_order_by_province" )
    export_data "ads_order_by_province" "dt,recent_days,province_id"
  ;;

  "ads_order_spu_stats" )
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
  ;;

  "ads_order_total" )
    export_data "ads_order_total" "dt,recent_days"
  ;;

  "ads_page_path" )
    export_data "ads_page_path" "dt,recent_days,source,target"
  ;;

  "ads_repeat_purchase" )
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
  ;;

  "ads_user_action" )
    export_data "ads_user_action" "dt,recent_days"
  ;;

  "ads_user_change" )
    export_data "ads_user_change" "dt"
  ;;

  "ads_user_retention" )
    export_data "ads_user_retention" "create_date,retention_day"
  ;;

  "ads_user_total" )
    export_data "ads_user_total" "dt,recent_days"
  ;;

  "ads_visit_stats" )
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
  "all" )
    export_data "ads_activity_stats" "dt,activity_id"
    export_data "ads_coupon_stats" "dt,coupon_id"
    export_data "ads_order_by_province" "dt,recent_days,province_id"
    export_data "ads_order_spu_stats" "dt,recent_days,spu_id"
    export_data "ads_order_total" "dt,recent_days"
    export_data "ads_page_path" "dt,recent_days,source,target"
    export_data "ads_repeat_purchase" "dt,recent_days,tm_id"
    export_data "ads_user_action" "dt,recent_days"
    export_data "ads_user_change" "dt"
    export_data "ads_user_retention" "create_date,retention_day"
    export_data "ads_user_total" "dt,recent_days"
    export_data "ads_visit_stats" "dt,recent_days,is_new,channel"
  ;;
esac
```

 **关于导出update还是insert的问题**

 **--update-mode：**

  	  updateonly  只更新，无法插入新数据

  	  allowinsert  允许新增 

**--update-key：**

​		允许更新的情况下，指定哪些字段匹配视为同一条数据，进行更新而不增加。多个字段用逗号分隔。

**--input-null-string和--input-null-non-string：**

​		分别表示，将字符串列和非字符串列的空串和“null”转义。

## 17.4 全调度流程	

### 17.4.1 数据准备	

1）用户行为数据准备

（1）修改/opt/module/applog下的application.properties

```
#业务日期
mock.date=2020-06-15
```

==注意：分发至其他需要生成数据的节点==

```
[atguigu@hadoop102 applog]$ xsync application.properties
```

（2）生成数据

```
[atguigu@hadoop102 bin]$ lg.sh
```

==注意：生成数据之后，记得查看HDFS数据是否存在！==

（3）观察HDFS的/origin_data/gmall/log/topic_log/2020-06-15路径是否有数据

2）业务数据准备

（1）修改/opt/module/db_log下的application.properties

```
[atguigu@hadoop102 db_log]$ vim application.properties
#业务日期
mock.date=2020-06-15
```

（2）生成数据

```
[atguigu@hadoop102 db_log]$ java -jar gmall2020-mock-db-2020-04-01.jar
```

（3）观察SQLyog中order_infor表中operate_time中有2020-06-15日期的数据

![image-20220929225453519](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220929225453519.png)

### 17.4.2 编写Azkaban工作流程配置文件	

1）编写azkaban.project文件，内容如下

```
azkaban-flow-version: 2.0
```

2）编写gmall.flow文件，内容如下

```shell
nodes:
  - name: mysql_to_hdfs
    type: command
    config:
     command: /home/atguigu/bin/mysql_to_hdfs.sh all ${dt}
    
  - name: hdfs_to_ods_log
    type: command
    config:
     command: /home/atguigu/bin/hdfs_to_ods_log.sh ${dt}
     
  - name: hdfs_to_ods_db
    type: command
    dependsOn: 
     - mysql_to_hdfs
    config: 
     command: /home/atguigu/bin/hdfs_to_ods_db.sh all ${dt}
  
  - name: ods_to_dim_db
    type: command
    dependsOn: 
     - hdfs_to_ods_db
    config: 
     command: /home/atguigu/bin/ods_to_dim_db.sh all ${dt}

  - name: ods_to_dwd_log
    type: command
    dependsOn: 
     - hdfs_to_ods_log
    config: 
     command: /home/atguigu/bin/ods_to_dwd_log.sh all ${dt}
    
  - name: ods_to_dwd_db
    type: command
    dependsOn: 
     - hdfs_to_ods_db
    config: 
     command: /home/atguigu/bin/ods_to_dwd_db.sh all ${dt}
    
  - name: dwd_to_dws
    type: command
    dependsOn:
     - ods_to_dim_db
     - ods_to_dwd_log
     - ods_to_dwd_db
    config:
     command: /home/atguigu/bin/dwd_to_dws.sh all ${dt}
    
  - name: dws_to_dwt
    type: command
    dependsOn:
     - dwd_to_dws
    config:
     command: /home/atguigu/bin/dws_to_dwt.sh all ${dt}
    
  - name: dwt_to_ads
    type: command
    dependsOn: 
     - dws_to_dwt
    config:
     command: /home/atguigu/bin/dwt_to_ads.sh all ${dt}
     
  - name: hdfs_to_mysql
    type: command
    dependsOn:
     - dwt_to_ads
    config:
      command: /home/atguigu/bin/hdfs_to_mysql.sh all
```

3）将azkaban.project、gmall.flow文件压缩到一个zip文件，文件名称必须是英文。

4）在WebServer新建项目：http://hadoop102:8081/index

5）给项目名称命名和添加项目描述

6）gmall.zip文件上传

7）选择上传的文件

8）查看任务流

9）详细任务流展示

10）配置输入dt时间参数

11）执行成功

12）在SQLyog上查看结果



### 17.4.3 Azkaban多Executor模式下注意事项	

​		Azkaban多Executor模式是指，在集群中多个节点部署Executor。在这种模式下， Azkaban web Server会根据策略，选取其中一个Executor去执行任务。

​		由于我们需要交给Azkaban调度的脚本，以及脚本需要的Hive，Sqoop等应用只在hadoop102部署了，为保证任务顺利执行，我们须在以下两种方案任选其一，推荐使用方案二。

方案一：指定特定的Executor（hadoop102）去执行任务。

1）在MySQL中azkaban数据库executors表中，查询hadoop102上的Executor的id。

```
mysql> use azkaban;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from executors;
+----+-----------+-------+--------+
| id | host          | port  | active |
+----+-----------+-------+--------+
|  1   | hadoop103 | 35985 |      1 |
|  2   | hadoop104 | 36363 |      1 |
|  3   | hadoop102 | 12321 |      1 |
+----+-----------+-------+--------+
3 rows in set (0.00 sec)
```

2）在执行工作流程时加入useExecutor属性，如下

![image-20220929225705543](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220929225705543.png)

方案二：在Executor所在所有节点部署任务所需脚本和应用。

1）分发脚本、sqoop、spark、my_env.sh

```
[atguigu@hadoop102 ~]$ xsync /home/atguigu/bin/
[atguigu@hadoop102 ~]$ xsync /opt/module/hive
[atguigu@hadoop102 ~]$ xsync /opt/module/sqoop
[atguigu@hadoop102 ~]$ xsync /opt/module/spark
[atguigu@hadoop102 ~]$ sudo /home/atguigu/bin/xsync /etc/profile.d/my_env.sh
```

2）分发之后，在hadoop103，hadoop104重新加载环境变量配置文件，并重启Azkaban

![image-20220929225746323](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220929225746323.png)

![image-20220929225759801](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220929225759801.png)