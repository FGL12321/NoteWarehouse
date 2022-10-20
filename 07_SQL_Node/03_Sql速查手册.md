[TOC]



# SQL速查手册-version1.0

# 1.0 SQL语句分类

## 1.1DDL

**DDL（Data Definition Languages、数据定义语言）**，这些语句定义了不同的数据库、表、视图、索引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911001620543.png" alt="image-20220911001620543" style="zoom:67%;" />



## 1.2DML

​		**DML（Data Manipulation Language、数据操作语言）**，用于添加、删除、更新和查询数据库记录，并检查数据完整性。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911001537283.png" alt="image-20220911001537283" style="zoom:67%;" />

## 1.3DCL

​		**DCL（Data Control Language、数据控制语言）**，用于定义数据库、表、字段、用户的访问权限和安全级别。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911001643759.png" alt="image-20220911001643759" style="zoom:67%;" />



# 2.0SQL语言规范

## 2.1基本规则

SQL 可以写在一行或者多行。为了提高可读性，各子句分行写，必要时使用缩进
每条命令以 ; 或 \g 或 \G 结束
关键字不能被缩写也不能分行
**关于标点符号**

* 必须保证所有的()、单引号、双引号是成对结束的
* 必须使用英文状态下的半角输入方式
* 字符串型和日期时间类型的数据可以使用单引号（’ '）表示
* 列的别名，尽量使用双引号（" "），而且不建议省略as


## 2.2SQL大小写规范

**MySQL 在 Windows 环境下是大小写不敏感的**

​       **MySQL 在 Linux 环境下是大小写敏感的**

​       数据库名、表名、表的别名、变量名是严格区分大小写的

​       关键字、函数名、列名(或字段名)、列的别名(字段的别名) 是忽略大小写的。

​       **推荐采用统一的书写规范：**

​		数据库名、表名、表别名、字段名、字段别名等都小写

​		SQL 关键字、函数名、绑定变量等都大写

## 2.3命名规则

数据库、表名不得超过30个字符，变量名限制为29个

必须只能包含 A–Z, a–z, 0–9, _共63个字符

数据库名、表名、字段名等对象名中间不要包含空格

同一个MySQL软件中，数据库不能同名；同一个库中，表不能重名同一个表中，字段不能重名

必须保证你的字段没有和保留字、数据库系统或常用方法冲突。如果坚持使用，请在SQL语句中使 用`（着重号）引起来

保持字段名和类型的一致性，在命名字段并为其指定数据类型的时候一定要保证一致性。

假如数据 类型在一个表里是整数，那在另一个表里可就别变成字符型了

## 2.4注释

```
单行注释：#注释文字(MySQL特有的方式)
单行注释：-- 注释文字(--后面必须包含一个空格。)
多行注释：/* 注释文字  */
```

# 3.0DDL语言（次重要内容）

## 3.1表操作

**1.创建表**

```sql
create table <表名>(
<列名1> <数据类型> <约束条件>
<列名2> <数据类型> <约束条件>
<列名3> <数据类型> <约束条件>
...
);
```

**2.更新表**

```sql
alter table <表名> add column <列名> <数据类型>;     //增加新列
alter table <表名> drop column <列名>;               //删除列
```

**3.重命名表**

```sql
rename table <旧表名> to <新表名>;
alter table <旧表名> rename to <新表名>;
```

**4.删除表**

```sql
drop table <表名>;
```

# 4.0DML语言（重要内容）

## 4.1查询

**1.检索单列**

```text
select <列名> from <表名>;
```

**2.检索多列**

```text
select <列名1>,<列名2>,... from <表名>;
```

**3.检索所有列**

```text
select * from <表名>;
```

**4.检索不同值**

```text
select distinct(<列名>) from <表名>;
```

**5.限制结果**

```
 在mysql中，如果我们需要限制结果输出的条数，可以使用limit <数字>来表示限制的行数。 
```

**6.注释**

```sql
# 注释               //单行注释

/*多行注释
嘻嘻嘻
*/                   //多行注释
```

## 4.2增加

**insert into:插入数据,如果主键重复，则报错**
**insert repalce:插入替换数据,如果存在主键或unique数据则替换数据**
**insert ignore:如果存在主键或unique数据,则忽略。**

## 4.3删除

 **删除表：** 

```
 drop table user; 
```

**清空表：**

```
truncate table user；
```

**删除行：**

```
DELETE FROM 表名称 WHERE 列名称 = 值
```

## 4.4修改

**新增一列**

```sql
alter table actor add column
create_date datetime not null default('2020-10-01 00:00:00')
```

# 5.0DCL语言（不重要内容）

**一、简介**
   **DCL 数据控制语言 (Data Control Language ) 在SQL语言中，是一种可对数据访问权进行控制的指令，它可以控制特定用户账户对数据表、查看表、存储程序、用户自定义函数等数据库对象的控制权，由 GRANT 和 REVOKE 两个指令组成。**

**二、用户**

## 5.1 创建用户

```
create user '用户名'@'IP地址' identified WITH mysql_native_password by '密码';
flush privileges;比如：
```

```
CREATE USER 'alian'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges;  MySQL8开始使用 caching_sha2_password ，如果你使用sqlyog工具无法连接，有两种方式处理：
```

**升级sqlyog到 SQLyog-13.1.6-0.x64 以上**
**修改mysql默认身份验证插件为 mysql_native_password**

## 5.2 修改用户名

```
rename user '用户名'@'IP地址' to '新用户名'@'IP地址';
```

## 5.3 修改密码

```
#切换到mysql库
use mysql;
#更新密码
UPDATE user SET password=password('新密码') WHERE user='用户名' AND host='IP地址';
#刷新权限
FLUSH PRIVILEGES;
```

**或者**

```
ALTER USER '用户名'@'IP地址' IDENTIFIED WITH mysql_native_password BY '新密码';
flush privileges;或者
```

```
#普通用户登录后
SET PASSWORD=password('新密码');
FLUSH PRIVILEGES;
```

## 5.4 删除用户

```
#注意这里的IP地址，一个用户可能会有多个
drop user '用户名'@'IP地址';
#比如
drop user 'Alian'@'192.168.0.100';
```

三、权限管理

## 5.5授权

**基本语法如下：**

```
grant 权限1, 权限2, 权限3,… ,权限n on 数据库名.表名 to 用户名@地址;
```


**关于 数据库名.表名 的说明：**

*.* 表示任意库的任意表（不建议）
mysql.* 表示mysql库的任意表
mysql.user 表示mysql库的user表
关于 用户名@地址 的说明(这里都是英文的单引号)：

’alian’@'localhost’ ：表示只允许本机登录
’alian’@’%' ：表示任意地址登录
’alian’@'192.168.0.100’ ：表示只允许ip为192.168.0.100的地址登录
’alian’@'192.168.*.*' ：表示只允许ip为192.168网段的地址登录

把数据库的所有库的所有权限都给alian,并且可以指定ip地址

```
#把数据库的所有库的所有权限都给alian,并且是任意ip地址都可以操作
grant all privileges on *.* to 'alian'@'%';
flush privileges;
```

把mysql数据库的所有权限都给alian

```
#把mysql数据库的所有权限都给alian,并且是任意ip地址都可以操作
grant all privileges on mysql.* to 'alian'@'%';
flush privileges;
```

把mysql数据库的user表的所有权限都给alian,并且是任意ip地址都可以操作

```
#把mysql数据库的user表的所有权限都给alian,并且是只能通过192.138.0.10才可以操作
grant all privileges on mysql.user to 'alian'@'192.138.0.10';
flush privileges;
```

把mysql数据库的user表的（查询，插入，更新，删除）的权限都给alian,并且是任意ip地址都可以操作

```
#把mysql数据库的user表的（查询，插入，更新，删除）的权限都给alian,并且是任意ip地址都可以操作
grant SELECT, INSERT, UPDATE, DELETE on mysql.user to 'alian'@'%';
flush privileges;
```

## 5.6查看权限

```
show grants for 'alian'@'%';
```

## 5.7 回收权限

基本语法如下：

```
revoke 权限1, 权限2…权限n on 数据库名.表名 from 用户名@地址;
比如把用户alian对mysql（默认的库）库的更新和删除权限收回
```

```
#回收用户的更新和删除mysql（默认的库）数据库的权限
revoke update,delete on mysql.user from 'alian'@'%';
```

## 5.8容器中中文显示

```
#交互模式设置语言并进入mysql容器（419413b9d276 是mysql的容器id）
docker exec -it 419413b9d276 env LANG=C.UTF-8 /bin/bash
```



# 6.0最常用关键字与函数

## 6.1排序

**1.对检索出的数据进行升序或者降序**

```text
order by <列名> asc/desc;
```

​       asc为升序排序，也是mysql中默认的排序方式；desc表示按照==降序==排序。此外根据需要我们也可以对多列进行排序，如下代码所示。 

```text
order by <列名1> <排序方法>,<列名2> <排序方式>...;
```

 值得注意的是，**order by语句通常位于查询语句的最后面**。后面会具体介绍查询语句的完整构成。 

## 6.2分组

**GROUP BY 语句**

GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

**SQL GROUP BY 语法**

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
```

## 6.3过滤

**1.使用where子句**

​        在sql中，我们使用where子句来对查询出的结果进行过滤，得到我们所需要的数据。它的格式如下。 

```text
select <列名> from <表名>
where <过滤条件>;
```

​         过滤条件有多个时，我们可以使用**and或or**来进行多个条件的罗列。值得注意的是，当有多个过滤条件是，**and的优先级会高于or**，这也许会导致一些逻辑错误，需要注意。 

**2.where子句操作符**

![img](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/v2-a8b698f93bf714ea6e8d7c3365342665_720w.jpg)

值得注意的是，在判断某个值是否为空值是，应该使用is null或is not null来判断，而不是使用"= null"或"!= null"。



## 6.4join关键字



![image-20220911191049869](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911191049869.png)

### 6.4.1INNER JOIN 关键字

![image-20220911191246838](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911191246838.png)

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name=table2.column_name;
```

==INNER JOIN 关键字在表中存在至少一个匹配时返回行==

### 6.4.2LEFT JOIN 关键字

![image-20220911191304330](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911191304330.png)



```sql
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.column_name=table2.column_name;
```

==LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL==

### 6.4.3RIGHT JOIN 关键字

![image-20220911191321010](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911191321010.png)

```sql
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.column_name=table2.column_name;
```

==RIGHT JOIN 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL==

### 6.4.4FULL OUTER JOIN 关键字

![image-20220911191334867](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220911191334867.png)

```sql
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.column_name=table2.column_name;
```

==FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.==

## 6.5通配符

在搜索数据库中的数据时，SQL 通配符可以替代一个或多个字符。

SQL 通配符必须与 LIKE 运算符一起使用。

在 SQL 中，可使用以下通配符：

| 通配符                     | 描述                       |
| :------------------------- | :------------------------- |
| %                          | 代表零个或多个字符         |
| _                          | 仅替代一个字符             |
| [charlist]                 | 字符列中的任何单一字符     |
| [^charlist]或者[!charlist] | 不在字符列中的任何单一字符 |

## 6.6SQL IN语法

**IN 操作符允许我们在 WHERE 子句中规定多个值。**

**SQL IN 语法**

```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```

## 6.7HAVING 子句

**在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。**

**SQL HAVING 语法**

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

# 7.0重要函数

## 7.1数学函数

- `ABS(x)`  返回x的绝对值
- `BIN(x)`  返回x的二进制（OCT返回八进制，HEX返回十六进制）
- `CEILING(x)`  返回大于x的最小整数值
- `EXP(x) ` 返回值e（自然对数的底）的x次方
- `FLOOR(x)`  返回小于x的最大整数值
- `GREATEST(x1,x2,...,xn)`返回集合中最大的值
- `LEAST(x1,x2,...,xn) `   返回集合中最小的值
- `LN(x)  `         返回x的自然对数
- `LOG(x,y)`返回x的以y为底的对数
- `MOD(x,y) `        返回x/y的模（余数）
- `PI()`返回pi的值（圆周率）
- `RAND()`返回０到１内的随机值,可以通过提供一个参数(种子)使RAND()随机数生成器生成一个指定的值。
- `ROUND(x,y)`返回参数x的四舍五入的有y位小数的值
- `SIGN(x) `返回代表数字x的符号的值
- `SQRT(x) `返回一个数的平方根
- `TRUNCATE(x,y) `      返回数字x截短为y位小数的结果

## 7.2聚合函数(常用于GROUP BY从句的SELECT查询中)

- `AVG(col)`返回指定列的平均值
- `COUNT(col)`返回指定列中非NULL值的个数
- `MIN(col)`返回指定列的最小值
- `MAX(col)`返回指定列的最大值
- `SUM(col)`返回指定列的所有值之和
- `GROUP_CONCAT(col) `返回由属于一组的列值连接组合而成的结果

## 7.3字符串函数

- `ASCII(char)`返回字符的ASCII码值
- `BIT_LENGTH(str)`返回字符串的比特长度
- `CONCAT(s1,s2...,sn)`将s1,s2...,sn连接成字符串
- `CONCAT_WS(sep,s1,s2...,sn)`将s1,s2...,sn连接成字符串，并用sep字符间隔
- `INSERT(str,x,y,instr) `将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
- `FIND_IN_SET(str,list)`分析逗号分隔的list列表，如果发现str，返回str在list中的位置
- `LCASE(str)或LOWER(str) `返回将字符串str中所有字符改变为小写后的结果
- `LEFT(str,x)`返回字符串str中最左边的x个字符
- `LENGTH(s)`返回字符串str中的字符数
- `LTRIM(str) `从字符串str中切掉开头的空格
- `POSITION(substr,str)` 返回子串substr在字符串str中第一次出现的位置
- `QUOTE(str) `用反斜杠转义str中的单引号
- `REPEAT(str,srchstr,rplcstr)`返回字符串str重复x次的结果
- `REVERSE(str) `返回颠倒字符串str的结果
- `RIGHT(str,x) `返回字符串str中最右边的x个字符
- `RTRIM(str)` 返回字符串str尾部的空格
- `STRCMP(s1,s2)`比较字符串s1和s2
- `TRIM(str)`去除字符串首部和尾部的所有空格
- `UCASE(str)`或`UPPER(str) `返回将字符串str中所有字符转变为大写后的结果
- `SUBSTRING_INDEX`(s, delimiter, number)

  > 返回从字符串 s 的第 number 个出现的分隔符 delimiter 之后的子串。
  > 如果 number 是正数，返回第 number 个字符左边的字符串。
  > 如果 number 是负数，返回第(number 的绝对值(从右边数))个字符右边的字符串。

## 7.4日期和时间函数

- `CURDATE()`或`CURRENT_DATE() `返回当前的日期
- `CURTIME()`或`CURRENT_TIME() `返回当前的时间
- `DATE_ADD(date,INTERVAL int keyword)`返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：`SELECTDATE_ADD(CURRENT_DATE,INTERVAL 6 MONTH);`
- `DATE_FORMAT(date,fmt) ` 依照指定的fmt格式格式化日期date值
- `DATE_SUB(date,INTERVAL int keyword)`返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：`SELECTDATE_SUB(CURRENT_DATE,INTERVAL 6 MONTH);`
- `DAYOFWEEK(date)`  返回date所代表的一星期中的第几天(1~7)
- `DAYOFMONTH(date) ` 返回date是一个月的第几天(1~31)
- `DAYOFYEAR(date)`  返回date是一年的第几天(1~366)
- `DAYNAME(date)`  返回date的星期名，如：`SELECT DAYNAME(CURRENT_DATE);`
- `FROM_UNIXTIME(ts,fmt) ` 根据指定的fmt格式，格式化UNIX时间戳ts
- `HOUR(time) ` 返回time的小时值(0~23)
- `MINUTE(time) ` 返回time的分钟值(0~59)
- `MONTH(date) ` 返回date的月份值(1~12)
- `MONTHNAME(date) ` 返回date的月份名，如：`SELECT MONTHNAME(CURRENT_DATE);`
- `NOW() `  返回当前的日期和时间
- `QUARTER(date) ` 返回date在一年中的季度(1~4)，如`SELECT QUARTER(CURRENT_DATE);`
- `WEEK(date) ` 返回日期date为一年中第几周(0~53)
- `YEAR(date) ` 返回日期date的年份(1000~9999)











