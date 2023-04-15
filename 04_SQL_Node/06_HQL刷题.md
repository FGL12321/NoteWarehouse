# HQL刷题总结

# 1.基本语法

```
rank() over(): 排名

distinct：去重

limit：限制行数

as：重新命名

order by：排序 也可多列排序  desc降序

BETWEEN and：表示范围

or：或者

not in：不在什么中

like '%北京%'：模糊匹配

Max()
avg()
count()
sum()
```

# 2.难点题目

## 第一题

```sql
SELECT
  nvl (sku_id, null) as sku_id  --如果不存在排名第二的商品就 返回null
FROM

  (
    SELECT
      sku_id,
      rank() over (order by sum(sku_num) desc) rk --根据销售商品件数进行降序排序 并标明其排序顺序
    FROM
      order_detail
    GROUP BY
      sku_id --根据商品id进行分组
  ) t1
WHERE
  rk = 2;
```

NVL函数的功能是实现空值的转换，根据第一个表达式的值是否为空值来返回相应的列名或表达式，主要用于对数据列上的空值进行处理，语法格式如：NVL( string1, replace_with)。

RANK()函数是一个Window函数，它为结果集的分区中的每一行分配一个排名

## 第二题

```sql
SELECT
  user_id
FROM
  (
    SELECT
      user_id,
      count(*) as cnt
    FROM
      (
        SELECT
          user_id,
          date_sub(create_date, rk) as ds--日期相减
        FROM
          (
            SELECT
              user_id,
              create_date,
              row_number() over(partition by user_id order by create_date) as rk --对数据根据user_id分区 根据创建时间进行排序
            FROM
              order_info
          ) t1
      ) t2
    GROUP by
      user_id,
      ds
    HAVING
      cnt >= 3
  ) t3;
```

## 第四题

**题目描述：**

从订单信息表(order_info)中统计每个用户截止其每个下单日期的累积消费金额，以及每个用户在其每个下单日期的VIP等级。
用户vip等级根据累积消费金额计算，计算规则如下：

设累积消费总额为X，
若0=<X<10000,则vip等级为普通会员
若10000<=X<30000,则vip等级为青铜会员
若30000<=X<50000,则vip等级为白银会员
若50000<=X<80000,则vip为黄金会员
若80000<=X<100000,则vip等级为白金会员
若X>=100000,则vip等级为钻石会员



![image-20230223190005521](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230223190005521.png)

![image-20230223185942783](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230223185942783.png)

### 思路

1.首先求出：每个用户截止其每个下单日期的累积消费金额

2.根据累计金额赋予其对应的VIP等级

### 语法了解

sum(...) over( )：对所有行求和

sum(...) over( order by ... )，和 = 第一行 到 与当前行同序号行的最后一行的所有值求和，

OVER(PARTITION BY... ORDER BY...) 先分割再排序



### 第一步：求出截止每个下单日期的累计消费金额

```
SELECT DISTINCT
  user_id,
  create_date,
  sum(total_amount) over (partition byuser_idorder bycreate_date) as sum_so_far
FROM
  order_info
```

![image-20230223191012074](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20230223191012074.png)

### 第二步：根据累计金额赋予其对应的VIP等级

```
SELECT
   user_id,
   create_date,
   t1.sum_so_far,
   (
     CASE
       when t1.sum_so_far >= 0 and sum_so_far<10000 THEN '普通会员'
       when t1.sum_so_far >= 10000 and sum_so_far<30000 THEN '青铜会员'
       when t1.sum_so_far >= 30000 and sum_so_far<50000 THEN '白银会员'
       when t1.sum_so_far >= 50000 and sum_so_far<80000 THEN '黄金会员'
       when t1.sum_so_far >= 80000 and sum_so_far<100000 THEN '白金会员'
     else '钻石会员'
     END
   ) vip_level
   FROM
     (
     SELECT
       DISTINCT
     user_id,
     create_date,
     sum(total_amount) over(partition by user_id order by create_date) as sum_so_far
     FROM
     order_info
     ) t1
```

以上就是本题解析过程！

# 第五题

1.排除用户一天购买多次的情况

2.取出每个用户按天排序的序号

3.取出每个用户下单日期与按天排序的序号之间的差值（若差值即首次下单日期的前一天的个数大于等于2，则说明此用户有首次下单后第二天仍然下单的情况）

4.取出连续两天下单的用户

5.排除用户有多次连续两天下单的可能

6.left join获取到首次下单后第二天仍然下单的用户占所有下单用户的比例



```sql
select
  concat(
    100 * sum(if(s5.user_id is not null, 1, 0)) / count(1),
    '%'
  ) as percentage
from
  (
    select
      user_id
    from
      order_info
    group by
      user_id
  ) s4
  left join (
    select
      user_id
    from
      (
        select
          user_id,
          date_diff,
          count(1) as cnt
        from
          (
            select
              user_id,
              create_date,
              rnk,
              date_sub(create_date, rnk) as date_diff
            from
              (
                select
                  user_id,
                  create_date,
                  row_number() over(
                    partition by user_id
                    order by
                      create_date
                  ) as rnk
                from
                  (
                    select
                      user_id,
                      create_date
                    from
                      order_info
                    group by
                      user_id,
                      create_date
                  ) s1
              ) s2
          ) s3
        group by
          user_id,
          date_diff
        having
          cnt >= 2
      ) s3
    group by
      user_id
  ) s5 on s4.user_id = s5.user_id
```

