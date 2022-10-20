[TOC]



# SQL学习笔记

### 1.简单（19题）

****

#### 1.查找最晚入员工信息

![1657713833699](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657713833699.png)

**涉及知识点：**

​       **order by排序  desc为降序   asc为升序**

​       **limit<数字>表示限定行数**

​              limit 0,1;表示输出从第0条向后读取一个 

**解答**

​     **采用先排序再输出的思路**

```sql
select * from employees
   order by hire_date desc
   limit 1;
```

​     **使用子查询：输出字符最大的员工的所有信息**

```sql
select * from employees
   where hire_date=(
       select max(hire_date) 
       from employees);
```

****



#### 2.如上图，查询入职员工中时间排名倒数第三的员工的所有信息

```sql
select * 
from 
employees
where hire_date=(
    select distinct hire_date 
    from employees 
    order by hire_date desc 
    limit 2,1);
```

**涉及新知识点：**distinct去重

****



#### 3. 请你查找所有已经分配部门的员工的last_name和first_name以及dept_no

![1657787070921](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657787070921.png)

**涉及知识点**

  JOIN 子句用于把来自两个或多个表的行结合起来，基于这些表之间的共同字段。 

表名  inner join  表名  on 连接条件 

- **INNER JOIN**：如果表中有至少一个匹配，则返回行

- **LEFT JOIN**：即使右表中没有匹配，也从左表返回所有的行

- **RIGHT JOIN**：即使左表中没有匹配，也从右表返回所有的行

- **FULL JOIN**：只要其中一个表中存在匹配，则返回行

  

**解答**

```sql
select last_name,first_name,dept_no
from employees e,dept_emp d
where  e.emp_no==d.emp_no;
```

```sql
select e.last_name,e.first_name,d.dept_no
from dept_emp d inner join employees e on d.emp_no=e.emp_no
```

****



#### 4. 查找薪水记录超过15条的员工号emp_no以及其对应的记录次数t

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657787898118.png" alt="1657787898118" style="zoom:150%;" />

**涉及知识点：**

Group By 分组

HAVING：可以让我们筛选分组后的各组数据。 

  				在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用。 

**解答：**

```sql
SELECT emp_no, COUNT(emp_no) AS t 
FROM salaries 
GROUP BY emp_no 
HAVING t > 15; 
```

****



#### 5. 请你找出所有员工具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示 

![1657788664173](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657788664173.png)

**涉及知识点：**

先分组，再排序

**解答：**

```sql
select salary
from salaries
group by salary
order by salary desc;
```

****

#### 6.  获取所有非manager的员工emp_no

![1657788961524](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657788961524.png)

**知识点：**

使用子查询+not in

**解答**：

```sql
select emp_no
from employees 
where emp_no not in (select emp_no from dept_manager)
```

****



#### 7. 查找employees表emp_no与last_name的员工信息

![1657789412944](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657789412944.png)

**涉及知识点：**

使用计算符进行对数字的运算

and的使用

**题解：**

```sql
select * 
from employees 
where emp_no % 2 == 1 and last_name != 'Mary'
order by hire_date desc
```

****



#### 8、 获取当前薪水第二多的员工的emp_no以及其对应的薪水salary

![1657790302417](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657790302417.png)

**涉及知识点**

排序 去重  limit的使用

**题解**

```sql
select emp_no ,salary
from salaries
where salary=(
      select distinct salary
      from salaries order by salary desc
      limit 1,1
)
```

#### 9. 将employees表的所有员工的last_name和first_name拼接起来

![1657872615821](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657872615821.png)

**涉及知识点**

字符串的拼接(last_name || ' '||first_name) as name

**题解**

```
select (last_name || ' '||first_name) as name
from employees
```

#### 10.批量插入数据

![1657872863319](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657872863319.png)

**涉及知识点：**

 INSERT INTO  [表名]([列名],[列名])   ([列名],[列名])  

VALUES ([列值],[列值])), ([列值],[列值])), ([列值],[列值])); 

**题解：**

```sql
insert into actor (actor_id,first_name,last_name,last_update)
values("1","PENELOPE","GUINESS","2006-02-15 12:34:33"),
      ("2","NICK","WAHLBERG","2006-02-15 12:34:33")
```

#### 11.删除emp_no重复的记录，只保留最小的id对应的记录。

![1657875014875](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657875014875.png)

涉及知识点：

分组 排序 子查询 min()函数

解答：

首先使用分组和min筛选出每个分组中不是最小值

然后删除不是这些值的内容

```
delete
from titles_test
where id not in(
   select *
   from (select min(id) from titles_test group by emp_no) as a);
```

#### 12. 将所有to_date为9999-01-01的全部更新为NULL

![1657875743408](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1657875743408.png)

**涉及知识点：**

更新字段： update 表名 set 字段 = 新值[,字段 = 新值] [where条件筛选]; 

**解答：**

```sql
update titles_test set to_date=null,from_data="2001-01-01"
where to_date="9999-01-01"
```

#### 13. 将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005, 

![1658048000875](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658048000875.png)

**涉及知识点：**

replace的使用

**解答：**

```sql
update titles_test set emp_no=replace(emp_no,10001,10005)
where id=5
```

#### 14  ：将titles_test表名修改为titles_2017



![1658048222084](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658048222084.png)

**涉及知识点：**

修改表名

**解答：**

####  alter table titles_test rename to titles_2017; 

####  15： 在牛客刷题的小伙伴们都有着牛客积分，积分(grade)表简化可以如下: 

![1658048401732](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658048401732.png)

**涉及知识点：**

 HAVING子句已添加到SQL中，因为WHERE关键字不能用于聚合函数。 

**解答：**

```sql
SELECT number 
FROM grade
GROUP BY number 
HAVING COUNT(number) >=3;
```

#### 16. 找到每个人的任务

![1658048609983](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658048609983.png)

**涉及知识点：**

​		 LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。 

**解答：**

```sql
select T.id, name, content
from person T left join task T2 on T2.person_id = T.id
order by T.id
```

#### 17.牛客每个人最近的登录日期(一)

![1658049112977](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658049112977.png)

**涉及知识点：**

分组求最大

**题解：**

```sql
select user_id,max(date)
from login 
group by user_id
```

#### 18.考试分数

![1658049503888](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658049503888.png)

**涉及知识点：**

小数点位数的设置

**题解：**

```sql
select job, round(avg(score),3) #round设置小数点的个数
from grade
group by job
order by avg(score) desc
```

#### 19. 牛客的课程订单分析(一)

![1658049751473](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658049751473.png)

**解答：**

```sql
select * 
from order_info
where date > '2025-10-15' and status = 'completed' and (product_name = 'C++' or product_name = 'Java' or product_name = 'Python')
order by id;
```

### 2.中等（12题）

#### 1.查找当前薪水详情以及部门编号dept_no

![1658453207426](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658453207426.png)

**题解：**

```sql
select s.*,d.dept_no
from salaries as s,dept_manager as d 
where s.emp_no = d.emp_no
order by emp_no asc
```

#### 2. 查找所有员工的last_name和first_name以 

![1658454756375](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658454756375.png)

**涉及知识点：**

left join关键字的使用

**题解：**

```sql
select e.last_name,e.first_name,d.dept_no
from employees e 
left join dept_emp d on d.emp_no=e.emp_no 
```

#### 3. 获取所有员工当前的manager 

![1658455460275](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658455460275.png)

**涉及知识点：**

on和where的区别

**题解：**

```sql
select e.emp_no,d.emp_no manager
from dept_emp e left join dept_manager d on e.dept_no=d.dept_no
where a.emp_no != b.emp_no
```

#### 4.统计平均工资

![1658457034097](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658457034097.png)



**涉及知识点：**

avg函数的使用 先分组，分组之后才能求平均

**解答：**

```sql
select t.title,avg(s.salary)
from titles t,salaries s
where t.emp_no=s.emp_no
group by t.title
order by avg(s.salary);
```

#### 5. 查找所有员工的last_name和first_name以及对应的dept_name 

![1658458148097](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658458148097.png)

**涉及知识点：**

三表联查，两次left join的使用

**题解：**

```sql
select e.last_name,e.first_name,departments.dept_name
from  employees e 
left join dept_emp p on e.emp_no=p.emp_no 
left join departments on  p.dept_no=departments.dept_no
```

####  6.统计各个部门的工资记录数 

![1658459371623](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658459371623.png)

**涉及知识点：**

三表联查   join的使用  count函数的使用  分组和排序的使用

**题解：**

```sql
select d.dept_no,d.dept_name,count(s.salary) sum 
from departments d 
inner join dept_emp e on d.dept_no=e.dept_no
inner join salaries s on e.emp_no=s.emp_no
group by d.dept_no
order by d.dept_no
```

#### 7. 使用join查询方式找出没有分类的电影id以及名称 

![1658469461477](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658469461477.png)



**涉及知识点：**

思路的转变，先两表查询，在进行筛选 where category_id is null

**解答：**

```sql
select d1.film_id,title,category_id
from  film d1 left join  film_category d3 on d1.film_id=d3.film_id
where category_id is null
```

#### 8. 使用子查询的方式找出属于Action分类的所有电影对应的title,description 

![1658469461477](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658469461477.png)

**涉及知识点：**

子查询

**解答：**

```sql
select f.title,f.description 
from film f left join film_category fc on f.film_id=fc.film_id 
where fc.category_id=(select category_id 
                      from category 
                      where name='Action')
```

#### 9.创建一个表设置主键和不为空等情况

![1658472217472](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658472217472.png)

**涉及知识点：**

新建一个表

**解答：**

```sql
CREATE TABLE actor
(
actor_id smallint(5) primary key not null,
first_name varchar(45) not null,
last_name varchar(45) not null,
last_update date not null
)
```

#### 10.批量插入数据，不使用replace操作

![1658820144022](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658820144022.png)

**涉及知识点：**

insert into:插入数据,如果主键重复，则报错
insert repalce:插入替换数据,如果存在主键或unique数据则替换数据
insert ignore:如果存在主键或unique数据,则忽略。 

**题解：**

```sql
insert ignore into actor values('3', 'ED', 'CHASE', '2006-02-15 12:34:33')
```

####  11.创建一个actor_name表 

![1658823845420](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658823845420.png)

**涉及知识点：**

从一个表向另一个表导入数据

**题解：**

```sql
create table  actor_name(
first_name varchar(45) not null,
last_name varchar(45) not null);
insert into actor_name
select first_name,last_name from actor;
```

#### 12.新增一列

![1658824182924](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1658824182924.png)

**涉及知识点：**

新增一列

**题解：**

```sql
ALTER TABLE actor ADD COLUMN
create_date datetime not null default('2020-10-01 00:00:00')
```

