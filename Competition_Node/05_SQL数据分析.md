

## 一，SQL概述



![3](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/3.png)

#### 1.1 SQL概述

![4](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/4.png)

![7](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/7.png)

#### 1.2 SQL数据库版本管理

可以采用sql 自带命令穿 cmd或者navicat

#### 1.3 SQL管理工具：

![8](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/8.png)

推荐使用以下的Navit:

Navict:支持MySQL、MongoDB、SQLServer、OracIe、SQLite、MariaDB和PostgreSQL的数据库管理。界面友好可读性更高，简单易用。

## 二，SQL简单查询

注意：在Navicat导入表，导入导向，选中要导入的表。

#### 2.1 查询关键字

![10](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/10.png)

#### 2.2 SQL核心框架

![11](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/11.png)

#### 2.3 SQL简单查询

##### 2.3.1基本查询

查询所有数据： select * from 表名;
查询某一列内容：select 列名f rom 表名;
查询指定多列内容:select 列名1,列名2,...from 表名;

示例·查看成绩信息

```
查看所有数据: select * from score;
查看数学成绩： select math from score;
查看学生姓名及数学成绩：select name,math from score;
```



##### 2.3.2去重（distinct）

检索不同的值：select distinct 列名 from 表名;

##### 2.3.3排序（orderhy）

对一列或多列排序，默认升序。
排序检索数据：select 列名 from 表名 order by 列名1，列名2 asc/desc;
asc：升序
desc：降序

##### 2.3.4 限制结果（limit）

limit 关键词可限制查询行数。
返回表的前n行：select 列名 from 表名 limit n;
返回表的特定行：select 列名 from 表名 limit n,m;    (从n+1行开始，返回m行）

```
示例：查看指定行信息
select * from score limit 5；
select * from score limit5,3;
```



##### 2.3.5 过滤查询(where）

过滤检索数据: select 列名 from 表名 where 条件语句
比较运算符    空值
逻辑运算符    模糊匹配
指定范围	   正则表达式
指定值			多条件

#### 2.4 条件查询

##### 2.4.1比较运算（>	 < 	>= 	<=）

```
示例：查看语文成绩大于90的数据信息
select * from score where chinese > 90;
```

##### 2.4.2 空值（is / is not null）

null值为遗漏的未知数据，用于判断是否为空;
示例·查找语文成绩为空的数据

```
 select * from score where chinese is null;
```

示例查找语文成绩不为空的数据

```
 select * from score where chinese is not null;
```



## 三，SQL复杂查询

#### 3.1 聚合函数

![21](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/21.png)

​		解析：

-示例：计算各班级学生数学平均成绩
--第一步：对班级分组

```
--select class from score group by class;
```

--第二步：求数学平均成绩avg(math)

```
select class,avg(math) from score group by class;
```



#### 3.2 分组函数

分组（groupby）
GROUP BY关键词可对数据进行分组，HAVING可对分组后的数据进行过滤。
示例1：计算各班级学生数学平均成绩

```
select class,avg(math) from score group by class;
```

示例2：计算数学平均分80以上的班级

```
select class,avg(math) as m from score group by class having m > 80;
```



#### 3.3 字段处理

计算字段：字段之间可以进行"加减乘除“
示例·统计每个学生的总分。

```
select name，(chinese+math+english) from score;
select name，(chinese+math+english) a s总分 from score;
```

注意针对NULL，对其进行任何算术运算，结果都为NULL

#### 3.4 视图(View)

视图是基于SQL语句结果集的可视化的表，视图包含行和列，就像一个真实的表。
语法结构：

```
create view 视图名称（<视图列名1>，<视图列名2>,......）as<select查询语句>；
```

示例：创建视图名为'班级人数'，按照班级分组，进行各班人数计数。

```
create view 班级人数（班级，人数）as select class,count(class)
			from score
			group by class;
```

![26](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/26.png)

汪意事顶:
视图是一张虚拟表，视图的字段由用户自定义
创建的视图会跟随着原始数据的变动实时更新
视图只供查询， 数据不可更改
使用select语句选择要查询的数据表也可以选择对已生成的视图查询
若要查询的数据表为某种经堂要查询的数据可建立为SQL视图，避免每次重复查询
避免在SQL视图的基础上再次新建SQL视图，会降低SQL的效能。

## 四，SQL多表

#### 4.1多表连接

一张表往往难以查询到需要的所有字段，多表连接可以同时查询多个表的字段，把不同表中的字段进行横向拼接，以满足在多表中各个字段的查询。

多表连接主要分为内连接、左连接、右连接、全连接、交叉连接。

#### 4.2内连接

![29](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/29.png)

![30](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/30.png)

#### 4.3 左连接



![444](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/444.png)



![32](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/32-166255323881112.png)

#### 4.4 右连接

![33](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/33.png)

![34](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/34.png)

#### 4.5 全连接

![35](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/35.png)

![36](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/36.png)

![37](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/37.png)

注意：union all不去重，union去重

#### 4.6 交叉连接

![38](https://yuchen-1313787112.cos.ap-nanjing.myqcloud.com/38.png)