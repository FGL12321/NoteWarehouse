[TOC]

# 16.0 数仓搭建-ADS层

## 16.1 建表说明	

ADS层不涉及建模，建表根据具体需求而定。

## 16.2 访客主题	

### 16.2.1 访客统计	

该需求为访客综合统计，其中包含若干指标，以下为对每个指标的解释说明。

| 指标             | 说明                                   | 对应字段         |
| ---------------- | -------------------------------------- | ---------------- |
| 访客数           | 统计访问人数                           | uv_count         |
| 页面停留时长     | 统计所有页面访问记录总时长，以秒为单位 | duration_sec     |
| 平均页面停留时长 | 统计每个会话平均停留时长，以秒为单位   | avg_duration_sec |
| 页面浏览总数     | 统计所有页面访问记录总数               | page_count       |
| 平均页面浏览数   | 统计每个会话平均浏览页面数             | avg_page_count   |
| 会话总数         | 统计会话总数                           | sv_count         |
| 跳出数           | 统计只浏览一个页面的会话个数           | bounce_count     |
| 跳出率           | 只有一个页面的会话的比例               | bounce_rate      |

1.建表语句

```sql
DROP TABLE IF EXISTS ads_visit_stats;
CREATE EXTERNAL TABLE ads_visit_stats (
  `dt` STRING COMMENT '统计日期',
  `is_new` STRING COMMENT '新老标识,1:新,0:老',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `channel` STRING COMMENT '渠道',
  `uv_count` BIGINT COMMENT '日活(访问人数)',
  `duration_sec` BIGINT COMMENT '页面停留总时长',
  `avg_duration_sec` BIGINT COMMENT '一次会话，页面停留平均时长,单位为描述',
  `page_count` BIGINT COMMENT '页面总浏览数',
  `avg_page_count` BIGINT COMMENT '一次会话，页面平均浏览数',
  `sv_count` BIGINT COMMENT '会话次数',
  `bounce_count` BIGINT COMMENT '跳出数',
  `bounce_rate` DECIMAL(16,2) COMMENT '跳出率'
) COMMENT '访客统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_visit_stats/';
```

2.数据装载
思路分析：该需求的关键点为会话的划分，总体实现思路可分为以下几步：
第一步：对所有页面访问记录进行会话的划分。
第二步：统计每个会话的浏览时长和浏览页面数。
第三步：统计上述各指标。

```sql
insert overwrite table ads_visit_stats
select * from ads_visit_stats
union
select
    '2020-06-14' dt,
    is_new,
    recent_days,
    channel,
    count(distinct(mid_id)) uv_count,
    cast(sum(duration)/1000 as bigint) duration_sec,
    cast(avg(duration)/1000 as bigint) avg_duration_sec,
    sum(page_count) page_count,
    cast(avg(page_count) as bigint) avg_page_count,
    count(*) sv_count,
    sum(if(page_count=1,1,0)) bounce_count,
    cast(sum(if(page_count=1,1,0))/count(*)*100 as decimal(16,2)) bounce_rate
from
(
    select
        session_id,
        mid_id,
        is_new,
        recent_days,
        channel,
        count(*) page_count,
        sum(during_time) duration
    from
    (
        select
            mid_id,
            channel,
            recent_days,
            is_new,
            last_page_id,
            page_id,
            during_time,
            concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by recent_days,mid_id order by ts)) session_id
        from
        (
            select
                mid_id,
                channel,
                last_page_id,
                page_id,
                during_time,
                ts,
                recent_days,
                if(visit_date_first>=date_add('2020-06-14',-recent_days+1),'1','0') is_new
            from
            (
                select
                    t1.mid_id,
                    t1.channel,
                    t1.last_page_id,
                    t1.page_id,
                    t1.during_time,
                    t1.dt,
                    t1.ts,
                    t2.visit_date_first
                from
                (
                    select
                        mid_id,
                        channel,
                        last_page_id,
                        page_id,
                        during_time,
                        dt,
                        ts
                    from dwd_page_log
                    where dt>=date_add('2020-06-14',-30)
                )t1
                left join
                (
                    select
                        mid_id,
                        visit_date_first
                    from dwt_visitor_topic
                    where dt='2020-06-14'
                )t2
                on t1.mid_id=t2.mid_id
            )t3 lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('2020-06-14',-recent_days+1)
        )t4
    )t5
    group by session_id,mid_id,is_new,recent_days,channel
)t6
group by is_new,recent_days,channel;
```

### 16.2.2 路径分析	

​		用户路径分析，顾名思义，就是指用户在APP或网站中的访问路径。为了衡量网站优化的效果或营销推广的效果，以及了解用户行为偏好，时常要对访问路径进行分析。

​		用户访问路径的可视化通常使用桑基图。如下图所示，该图可真实还原用户的访问路径，包括页面跳转和页面访问次序。

​		桑基图需要我们提供每种页面跳转的次数，每个跳转由source/target表示，source指跳转起始页面，target表示跳转终到页面。

![image-20220928142911244](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220928142911244.png)

1.建表语句

```sql
DROP TABLE IF EXISTS ads_page_path;
CREATE EXTERNAL TABLE ads_page_path
(
    `dt` STRING COMMENT '统计日期',
    `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `source` STRING COMMENT '跳转起始页面ID',
    `target` STRING COMMENT '跳转终到页面ID',
    `path_count` BIGINT COMMENT '跳转次数'
)  COMMENT '页面浏览路径'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_page_path/';
```

2.数据装载
思路分析：该需求要统计的就是每种跳转的次数，故理论上对source/target进行分组count()即可。统计时需注意以下两点：
第一点：桑基图的source不允许为空，但target可为空。
第二点：桑基图所展示的流程不允许存在环。

```sql
insert overwrite table ads_page_path
select * from ads_page_path
union
select
    '2020-06-14',
    recent_days,
    source,
    target,
    count(*)
from
(
    select
        recent_days,
        concat('step-',step,':',source) source,
        concat('step-',step+1,':',target) target
    from
    (
        select
            recent_days,
            page_id source,
            lead(page_id,1,null) over (partition by recent_days,session_id order by ts) target,
            row_number() over (partition by recent_days,session_id order by ts) step
        from
        (
            select
                recent_days,
                last_page_id,
                page_id,
                ts,
                concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by mid_id,recent_days order by ts)) session_id
            from dwd_page_log lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('2020-06-14',-30)
            and dt>=date_add('2020-06-14',-recent_days+1)
        )t2
    )t3
)t4
group by recent_days,source,target;
```

## 16.3 用户主题	

### 16.3.1 用户统计	

该需求为用户综合统计，其中包含若干指标，以下为对每个指标的解释说明。

| 指标           | 说明                   | 对应字段             |
| -------------- | ---------------------- | -------------------- |
| 新增用户数     | 统计新增注册用户人数   | new_user_count       |
| 新增下单用户数 | 统计新增下单用户人数   | new_order_user_count |
| 下单总金额     | 统计所有订单总额       | order_final_amount   |
| 下单用户数     | 统计下单用户总数       | order_user_count     |
| 未下单用户数   | 统计活跃但未下单用户数 | no_order_user_count  |

1.建表语句

```sql
DROP TABLE IF EXISTS ads_user_total;
CREATE EXTERNAL TABLE `ads_user_total` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,0:累积值,1:最近1天,7:最近7天,30:最近30天',
  `new_user_count` BIGINT COMMENT '新注册用户数',
  `new_order_user_count` BIGINT COMMENT '新增下单用户数',
  `order_final_amount` DECIMAL(16,2) COMMENT '下单总金额',
  `order_user_count` BIGINT COMMENT '下单用户数',
  `no_order_user_count` BIGINT COMMENT '未下单用户数(具体指活跃用户中未下单用户)'
) COMMENT '用户统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_total/';
```

2.数据装载

```sql
insert overwrite table ads_user_total
select * from ads_user_total
union
select
    '2020-06-14',
    recent_days,
    sum(if(login_date_first>=recent_days_ago,1,0)) new_user_count,
    sum(if(order_date_first>=recent_days_ago,1,0)) new_order_user_count,
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count,
    sum(if(login_date_last>=recent_days_ago and order_final_amount=0,1,0)) no_order_user_count
from
(
    select
        recent_days,
        user_id,
        login_date_first,
        login_date_last,
        order_date_first,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount,
        if(recent_days=0,'1970-01-01',date_add('2020-06-14',-recent_days+1)) recent_days_ago
    from dwt_user_topic lateral view explode(Array(0,1,7,30)) tmp as recent_days
    where dt='2020-06-14'
)t1
group by recent_days;
```

### 16.3.2 用户变动统计	

该需求包括两个指标，分别为流失用户数和回流用户数，以下为对两个指标的解释说明。

| 指标       | 说明                                                         | 对应字段             |
| ---------- | ------------------------------------------------------------ | -------------------- |
| 流失用户数 | 之前活跃过的用户，最近一段时间未活跃，就称为流失用户。此处要求统计7日前（只包含7日前当天）活跃，但最近7日未活跃的用户总数。 | user_churn_count     |
| 回流用户数 | 之前的活跃用户，一段时间未活跃（流失），今日又活跃了，就称为回流用户。此处要求统计回流用户总数。 | new_order_user_count |

1.建表语句

```sql
DROP TABLE IF EXISTS ads_user_change;
CREATE EXTERNAL TABLE `ads_user_change` (
  `dt` STRING COMMENT '统计日期',
  `user_churn_count` BIGINT COMMENT '流失用户数',
  `user_back_count` BIGINT COMMENT '回流用户数'
) COMMENT '用户变动统计'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_change/';
```

2.数据装载
思路分析：
流失用户：末次活跃时间为7日前的用户即为流失用户。
回流用户：末次活跃时间为今日，上次活跃时间在8日前的用户即为回流用户。

```sql
insert overwrite table ads_user_change
select * from ads_user_change
union
select
    churn.dt,
    user_churn_count,
    user_back_count
from
(
    select
        '2020-06-14' dt,
        count(*) user_churn_count
    from dwt_user_topic
    where dt='2020-06-14'
    and login_date_last=date_add('2020-06-14',-7)
)churn
join
(
    select
        '2020-06-14' dt,
        count(*) user_back_count
    from
    (
        select
            user_id,
            login_date_last
        from dwt_user_topic
        where dt='2020-06-14'
        and login_date_last='2020-06-14'
    )t1
    join
    (
        select
            user_id,
            login_date_last login_date_previous
        from dwt_user_topic
        where dt=date_add('2020-06-14',-1)
    )t2
    on t1.user_id=t2.user_id
    where datediff(login_date_last,login_date_previous)>=8
)back
on churn.dt=back.dt;
```

### 16.3.3 用户行为漏斗分析	

​		漏斗分析是一个数据分析模型，它能够科学反映一个业务过程从起点到终点各阶段用户转化情况。由于其能将各阶段环节都展示出来，故哪个阶段存在问题，就能一目了然。

![image-20220928143220470](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220928143220470.png)

该需求要求统计一个完整的购物流程各个阶段的人数。

1.建表语句

```
DROP TABLE IF EXISTS ads_user_action;
CREATE EXTERNAL TABLE `ads_user_action` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `home_count` BIGINT COMMENT '浏览首页人数',
  `good_detail_count` BIGINT COMMENT '浏览商品详情页人数',
  `cart_count` BIGINT COMMENT '加入购物车人数',
  `order_count` BIGINT COMMENT '下单人数',
  `payment_count` BIGINT COMMENT '支付人数'
) COMMENT '漏斗分析'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_action/';
```

2.数据装载

```sql
with
tmp_page as
(
    select
        '2020-06-14' dt,
        recent_days,
        sum(if(array_contains(pages,'home'),1,0)) home_count,
        sum(if(array_contains(pages,'good_detail'),1,0)) good_detail_count
    from
    (
        select
            recent_days,
            mid_id,
            collect_set(page_id) pages
        from
        (
            select
                dt,
                mid_id,
                page.page_id
            from dws_visitor_action_daycount lateral view explode(page_stats) tmp as page
            where dt>=date_add('2020-06-14',-29)
            and page.page_id in('home','good_detail')
        )t1 lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('2020-06-14',-recent_days+1)
        group by recent_days,mid_id
    )t2
    group by recent_days
),
tmp_cop as
(
    select
        '2020-06-14' dt,
        recent_days,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from
    (
        select
            recent_days,
            user_id,
            case
                when recent_days=1 then cart_last_1d_count
                when recent_days=7 then cart_last_7d_count
                when recent_days=30 then cart_last_30d_count
            end cart_count,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then payment_last_1d_count
                when recent_days=7 then payment_last_7d_count
                when recent_days=30 then payment_last_30d_count
            end payment_count
        from dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='2020-06-14'
    )t1
    group by recent_days
)
insert overwrite table ads_user_action
select * from ads_user_action
union
select
    tmp_page.dt,
    tmp_page.recent_days,
    home_count,
    good_detail_count,
    cart_count,
    order_count,
    payment_count
from tmp_page
join tmp_cop
on tmp_page.recent_days=tmp_cop.recent_days;
```

### 16.3.4 用户留存率	

​		留存分析一般包含新增留存和活跃留存分析。

​		新增留存分析是分析某天的新增用户中，有多少人有后续的活跃行为。活跃留存分析是分析某天的活跃用户中，有多少人有后续的活跃行为。

​		留存分析是衡量产品对用户价值高低的重要指标。

​		此处要求统计新增留存率，新增留存率具体是指留存用户数与新增用户数的比值，例如2020-06-14新增100个用户，1日之后（2020-06-15）这100人中有80个人活跃了，那2020-06-14的1日留存数则为80，2020-06-14的1日留存率则为80%。

​		要求统计每天的1至7日留存率，如下图所示。

![image-20220928143336113](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220928143336113.png)

1.建表语句

```sql
DROP TABLE IF EXISTS ads_user_retention;
CREATE EXTERNAL TABLE ads_user_retention (
  `dt` STRING COMMENT '统计日期',
  `create_date` STRING COMMENT '用户新增日期',
  `retention_day` BIGINT COMMENT '截至当前日期留存天数',
  `retention_count` BIGINT COMMENT '留存用户数量',
  `new_user_count` BIGINT COMMENT '新增用户数量',
  `retention_rate` DECIMAL(16,2) COMMENT '留存率'
) COMMENT '用户留存率'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_user_retention/';
```

2.数据装载

```sql
insert overwrite table ads_user_retention
select * from ads_user_retention
union
select
    '2020-06-14',
    login_date_first create_date,
    datediff('2020-06-14',login_date_first) retention_day,
    sum(if(login_date_last='2020-06-14',1,0)) retention_count,
    count(*) new_user_count,
    cast(sum(if(login_date_last='2020-06-14',1,0))/count(*)*100 as decimal(16,2)) retention_rate
from dwt_user_topic
where dt='2020-06-14'
and login_date_first>=date_add('2020-06-14',-7)
and login_date_first<'2020-06-14'
group by login_date_first;
```

## 16.4 商品主题

### 16.4.1 商品统计	

该指标为商品综合统计，包含每个spu被下单总次数和被下单总金额。

1.建表语句

```sql
DROP TABLE IF EXISTS ads_order_spu_stats;
CREATE EXTERNAL TABLE `ads_order_spu_stats` (
    `dt` STRING COMMENT '统计日期',
    `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `spu_id` STRING COMMENT '商品ID',
    `spu_name` STRING COMMENT '商品名称',
    `tm_id` STRING COMMENT '品牌ID',
    `tm_name` STRING COMMENT '品牌名称',
    `category3_id` STRING COMMENT '三级品类ID',
    `category3_name` STRING COMMENT '三级品类名称',
    `category2_id` STRING COMMENT '二级品类ID',
    `category2_name` STRING COMMENT '二级品类名称',
    `category1_id` STRING COMMENT '一级品类ID',
    `category1_name` STRING COMMENT '一级品类名称',
    `order_count` BIGINT COMMENT '订单数',
    `order_amount` DECIMAL(16,2) COMMENT '订单金额'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_spu_stats/';
```

2.数据装载

```sql
insert overwrite table ads_order_spu_stats
select * from ads_order_spu_stats
union
select
    '2020-06-14' dt,
    recent_days,
    spu_id,
    spu_name,
    tm_id,
    tm_name,
    category3_id,
    category3_name,
    category2_id,
    category2_name,
    category1_id,
    category1_name,
    sum(order_count),
    sum(order_amount)
from
(
    select
        recent_days,
        sku_id,
        case
            when recent_days=1 then order_last_1d_count
            when recent_days=7 then order_last_7d_count
            when recent_days=30 then order_last_30d_count
        end order_count,
        case
            when recent_days=1 then order_last_1d_final_amount
            when recent_days=7 then order_last_7d_final_amount
            when recent_days=30 then order_last_30d_final_amount
        end order_amount
    from dwt_sku_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='2020-06-14'
)t1
left join
(
    select
        id,
        spu_id,
        spu_name,
        tm_id,
        tm_name,
        category3_id,
        category3_name,
        category2_id,
        category2_name,
        category1_id,
        category1_name
    from dim_sku_info
    where dt='2020-06-14'
)t2
on t1.sku_id=t2.id
group by recent_days,spu_id,spu_name,tm_id,tm_name,category3_id,category3_name,category2_id,category2_name,category1_id,category1_name;
```

### 16.4.2 品牌复购率	

​		品牌复购率是指一段时间内重复购买某品牌的人数与购买过该品牌的人数的比值。重复购买即购买次数大于等于2，购买过即购买次数大于1。

​		此处要求统计最近1,7,30天的各品牌复购率。

1.建表语句

```sql
DROP TABLE IF EXISTS ads_repeat_purchase;
CREATE EXTERNAL TABLE `ads_repeat_purchase` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `tm_id` STRING COMMENT '品牌ID',
  `tm_name` STRING COMMENT '品牌名称',
  `order_repeat_rate` DECIMAL(16,2) COMMENT '复购率'
) COMMENT '品牌复购率'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_repeat_purchase/';
```

2.数据装载
思路分析：该需求可分两步实现：
第一步：统计每个用户购买每个品牌的次数。
第二步：分别统计购买次数大于1的人数和大于2的人数。

```sql
insert overwrite table ads_repeat_purchase
select * from ads_repeat_purchase
union
select
    '2020-06-14' dt,
    recent_days,
    tm_id,
    tm_name,
    cast(sum(if(order_count>=2,1,0))/sum(if(order_count>=1,1,0))*100 as decimal(16,2))
from
(
    select
        recent_days,
        user_id,
        tm_id,
        tm_name,
        sum(order_count) order_count
    from
    (
        select
            recent_days,
            user_id,
            sku_id,
            count(*) order_count
        from dwd_order_detail lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('2020-06-14',-29)
        and dt>=date_add('2020-06-14',-recent_days+1)
        group by recent_days, user_id,sku_id
    )t1
    left join
    (
        select
            id,
            tm_id,
            tm_name
        from dim_sku_info
        where dt='2020-06-14'
    )t2
    on t1.sku_id=t2.id
    group by recent_days,user_id,tm_id,tm_name
)t3
group by recent_days,tm_id,tm_name;
```

## 16.5 订单主题	

### 16.5.1 订单统计	

该需求包含订单总数，订单总金额和下单总人数。

1.建表语句

```sql
DROP TABLE IF EXISTS ads_order_total;
CREATE EXTERNAL TABLE `ads_order_total` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `order_count` BIGINT COMMENT '订单数',
  `order_amount` DECIMAL(16,2) COMMENT '订单金额',
  `order_user_count` BIGINT COMMENT '下单人数'
) COMMENT '订单统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_total/';
```

2.数据装载

```sql
insert overwrite table ads_order_total
select * from ads_order_total
union
select
    '2020-06-14',
    recent_days,
    sum(order_count),
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count
from
(
    select
        recent_days,
        user_id,
        case when recent_days=0 then order_count
             when recent_days=1 then order_last_1d_count
             when recent_days=7 then order_last_7d_count
             when recent_days=30 then order_last_30d_count
        end order_count,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount
    from dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='2020-06-14'
)t1
group by recent_days;
```

### 16.5.2 各地区订单统计	

该需求包含各省份订单总数和订单总金额。
1.建表语句 

```sql
DROP TABLE IF EXISTS ads_order_by_province;
CREATE EXTERNAL TABLE `ads_order_by_province` (
  `dt` STRING COMMENT '统计日期',
  `recent_days` BIGINT COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
  `province_id` STRING COMMENT '省份ID',
  `province_name` STRING COMMENT '省份名称',
  `area_code` STRING COMMENT '地区编码',
  `iso_code` STRING COMMENT '国际标准地区编码',
  `iso_code_3166_2` STRING COMMENT '国际标准地区编码',
  `order_count` BIGINT COMMENT '订单数',
  `order_amount` DECIMAL(16,2) COMMENT '订单金额'
) COMMENT '各地区订单统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_order_by_province/';
```

2.数据装载

```sql
insert overwrite table ads_order_by_province
select * from ads_order_by_province
union
select
    dt,
    recent_days,
    province_id,
    province_name,
    area_code,
    iso_code,
    iso_3166_2,
    order_count,
    order_amount
from
(
    select
        '2020-06-14' dt,
        recent_days,
        province_id,
        sum(order_count) order_count,
        sum(order_amount) order_amount
    from
    (
        select
            recent_days,
            province_id,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then order_last_1d_final_amount
                when recent_days=7 then order_last_7d_final_amount
                when recent_days=30 then order_last_30d_final_amount
            end order_amount
        from dwt_area_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='2020-06-14'
    )t1
    group by recent_days,province_id
)t2
join dim_base_province t3
on t2.province_id=t3.id;
```

## 16.6 优惠券主题	

​		该需求要求统计最近30日发布的所有优惠券的领用情况和补贴率，补贴率是指，优惠金额与使用优惠券的订单的原价金额的比值。

### 16.6.1 优惠券统计	

1.建表语句

```sql
DROP TABLE IF EXISTS ads_coupon_stats;
CREATE EXTERNAL TABLE ads_coupon_stats (
  `dt` STRING COMMENT '统计日期',
  `coupon_id` STRING COMMENT '优惠券ID',
  `coupon_name` STRING COMMENT '优惠券名称',
  `start_date` STRING COMMENT '发布日期',
  `rule_name` STRING COMMENT '优惠规则，例如满100元减10元',
  `get_count`  BIGINT COMMENT '领取次数',
  `order_count` BIGINT COMMENT '使用(下单)次数',
  `expire_count`  BIGINT COMMENT '过期次数',
  `order_original_amount` DECIMAL(16,2) COMMENT '使用优惠券订单原始金额',
  `order_final_amount` DECIMAL(16,2) COMMENT '使用优惠券订单最终金额',
  `reduce_amount` DECIMAL(16,2) COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) COMMENT '补贴率'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_coupon_stats/';

```

2.数据装载

```sql
insert overwrite table ads_coupon_stats
select * from ads_coupon_stats
union
select
    '2020-06-14' dt,
    t1.id,
    coupon_name,
    start_date,
    rule_name,
    get_count,
    order_count,
    expire_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        id,
        coupon_name,
        date_format(start_time,'yyyy-MM-dd') start_date,
        case
            when coupon_type='3201' then concat('满',condition_amount,'元减',benefit_amount,'元')
            when coupon_type='3202' then concat('满',condition_num,'件打', (1-benefit_discount)*10,'折')
            when coupon_type='3203' then concat('减',benefit_amount,'元')
        end rule_name
    from dim_coupon_info
    where dt='2020-06-14'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('2020-06-14',-29)
)t1
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        expire_count,
        order_original_amount,
        order_final_amount,
        order_reduce_amount reduce_amount,
        cast(order_reduce_amount/order_original_amount as decimal(16,2)) reduce_rate
    from dwt_coupon_topic
    where dt='2020-06-14'
)t2
on t1.id=t2.coupon_id;
```

## 16.7 活动主题	

### 16.7.1 活动统计	

​		该需求要求统计最近30日发布的所有活动的参与情况和补贴率，补贴率是指，优惠金额与参与活动的订单原价金额的比值。

1.建表语句

```sql
DROP TABLE IF EXISTS ads_activity_stats;
CREATE EXTERNAL TABLE `ads_activity_stats` (
  `dt` STRING COMMENT '统计日期',
  `activity_id` STRING COMMENT '活动ID',
  `activity_name` STRING COMMENT '活动名称',
  `start_date` STRING COMMENT '活动开始日期',
  `order_count` BIGINT COMMENT '参与活动订单数',
  `order_original_amount` DECIMAL(16,2) COMMENT '参与活动订单原始金额',
  `order_final_amount` DECIMAL(16,2) COMMENT '参与活动订单最终金额',
  `reduce_amount` DECIMAL(16,2) COMMENT '优惠金额',
  `reduce_rate` DECIMAL(16,2) COMMENT '补贴率'
) COMMENT '商品销售统计'
ROW FORMAT DELIMITED  FIELDS TERMINATED BY '\t'
LOCATION '/warehouse/gmall/ads/ads_activity_stats/';

```

2.数据装载

```sql
insert overwrite table ads_activity_stats
select * from ads_activity_stats
union
select
    '2020-06-14' dt,
    t4.activity_id,
    activity_name,
    start_date,
    order_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        activity_id,
        activity_name,
        date_format(start_time,'yyyy-MM-dd') start_date
    from dim_activity_rule_info
    where dt='2020-06-14'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('2020-06-14',-29)
    group by activity_id,activity_name,start_time
)t4
left join
(
    select
        activity_id,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(order_reduce_amount) reduce_amount,
        cast(sum(order_reduce_amount)/sum(order_original_amount)*100 as decimal(16,2)) reduce_rate
    from dwt_activity_topic
    where dt='2020-06-14'
    group by activity_id
)t5
on t4.activity_id=t5.activity_id;
```

## 16.8 ADS层业务数据导入脚本

1）编写脚本

（1）在/home/atguigu/bin目录下创建脚本dwt_to_ads.sh

```
[atguigu@hadoop102 bin]$ vim dwt_to_ads.sh
```

在脚本中填写如下内容

```sql
#!/bin/bash

APP=gmall

# 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

ads_activity_stats="
insert overwrite table ${APP}.ads_activity_stats
select * from ${APP}.ads_activity_stats
union
select
    '$do_date' dt,
    t4.activity_id,
    activity_name,
    start_date,
    order_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        activity_id,
        activity_name,
        date_format(start_time,'yyyy-MM-dd') start_date
    from ${APP}.dim_activity_rule_info
    where dt='$do_date'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('$do_date',-29)
    group by activity_id,activity_name,start_time
)t4
left join
(
    select
        activity_id,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(order_reduce_amount) reduce_amount,
        cast(sum(order_reduce_amount)/sum(order_original_amount)*100 as decimal(16,2)) reduce_rate
    from ${APP}.dwt_activity_topic
    where dt='$do_date'
    group by activity_id
)t5
on t4.activity_id=t5.activity_id;
"
ads_coupon_stats="
insert overwrite table ${APP}.ads_coupon_stats
select * from ${APP}.ads_coupon_stats
union
select
    '$do_date' dt,
    t1.id,
    coupon_name,
    start_date,
    rule_name,
    get_count,
    order_count,
    expire_count,
    order_original_amount,
    order_final_amount,
    reduce_amount,
    reduce_rate
from
(
    select
        id,
        coupon_name,
        date_format(start_time,'yyyy-MM-dd') start_date,
        case
            when coupon_type='3201' then concat('满',condition_amount,'元减',benefit_amount,'元')
            when coupon_type='3202' then concat('满',condition_num,'件打', (1-benefit_discount)*10,'折')
            when coupon_type='3203' then concat('减',benefit_amount,'元')
        end rule_name
    from ${APP}.dim_coupon_info
    where dt='$do_date'
    and date_format(start_time,'yyyy-MM-dd')>=date_add('$do_date',-29)
)t1
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        expire_count,
        order_original_amount,
        order_final_amount,
        order_reduce_amount reduce_amount,
        cast(order_reduce_amount/order_original_amount as decimal(16,2)) reduce_rate
    from ${APP}.dwt_coupon_topic
    where dt='$do_date'
)t2
on t1.id=t2.coupon_id;
"

ads_order_by_province="
insert overwrite table ${APP}.ads_order_by_province
select * from ${APP}.ads_order_by_province
union
select
    dt,
    recent_days,
    province_id,
    province_name,
    area_code,
    iso_code,
    iso_3166_2,
    order_count,
    order_amount
from
(
    select
        '$do_date' dt,
        recent_days,
        province_id,
        sum(order_count) order_count,
        sum(order_amount) order_amount
    from
    (
        select
            recent_days,
            province_id,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then order_last_1d_final_amount
                when recent_days=7 then order_last_7d_final_amount
                when recent_days=30 then order_last_30d_final_amount
            end order_amount
        from ${APP}.dwt_area_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='$do_date'
    )t1
    group by recent_days,province_id
)t2
join ${APP}.dim_base_province t3
on t2.province_id=t3.id;
"

ads_order_spu_stats="
insert overwrite table ${APP}.ads_order_spu_stats
select * from ${APP}.ads_order_spu_stats
union
select
    '$do_date' dt,
    recent_days,
    spu_id,
    spu_name,
    tm_id,
    tm_name,
    category3_id,
    category3_name,
    category2_id,
    category2_name,
    category1_id,
    category1_name,
    sum(order_count),
    sum(order_amount)
from
(
    select
        recent_days,
        sku_id,
        case
            when recent_days=1 then order_last_1d_count
            when recent_days=7 then order_last_7d_count
            when recent_days=30 then order_last_30d_count
        end order_count,
        case
            when recent_days=1 then order_last_1d_final_amount
            when recent_days=7 then order_last_7d_final_amount
            when recent_days=30 then order_last_30d_final_amount
        end order_amount
    from ${APP}.dwt_sku_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
left join
(
    select
        id,
        spu_id,
        spu_name,
        tm_id,
        tm_name,
        category3_id,
        category3_name,
        category2_id,
        category2_name,
        category1_id,
        category1_name
    from ${APP}.dim_sku_info
    where dt='$do_date'
)t2
on t1.sku_id=t2.id
group by recent_days,spu_id,spu_name,tm_id,tm_name,category3_id,category3_name,category2_id,category2_name,category1_id,category1_name;
"

ads_order_total="
insert overwrite table ${APP}.ads_order_total
select * from ${APP}.ads_order_total
union
select
    '$do_date',
    recent_days,
    sum(order_count),
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count
from
(
    select
        recent_days,
        user_id,
        case when recent_days=0 then order_count
             when recent_days=1 then order_last_1d_count
             when recent_days=7 then order_last_7d_count
             when recent_days=30 then order_last_30d_count
        end order_count,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount
    from ${APP}.dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
group by recent_days;
"

ads_page_path="
insert overwrite table ${APP}.ads_page_path
select * from ${APP}.ads_page_path
union
select
    '$do_date',
    recent_days,
    source,
    target,
    count(*)
from
(
    select
        recent_days,
        concat('step-',step,':',source) source,
        concat('step-',step+1,':',target) target
    from
    (
        select
            recent_days,
            page_id source,
            lead(page_id,1,null) over (partition by recent_days,session_id order by ts) target,
            row_number() over (partition by recent_days,session_id order by ts) step
        from
        (
            select
                recent_days,
                last_page_id,
                page_id,
                ts,
                concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by mid_id,recent_days order by ts)) session_id
            from ${APP}.dwd_page_log lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('$do_date',-30)
            and dt>=date_add('$do_date',-recent_days+1)
        )t2
    )t3
)t4
group by recent_days,source,target;
"

ads_repeat_purchase="
insert overwrite table ${APP}.ads_repeat_purchase
select * from ${APP}.ads_repeat_purchase
union
select
    '$do_date' dt,
    recent_days,
    tm_id,
    tm_name,
    cast(sum(if(order_count>=2,1,0))/sum(if(order_count>=1,1,0))*100 as decimal(16,2))
from
(
    select
        recent_days,
        user_id,
        tm_id,
        tm_name,
        sum(order_count) order_count
    from
    (
        select
            recent_days,
            user_id,
            sku_id,
            count(*) order_count
        from ${APP}.dwd_order_detail lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('$do_date',-29)
        and dt>=date_add('$do_date',-recent_days+1)
        group by recent_days, user_id,sku_id
    )t1
    left join
    (
        select
            id,
            tm_id,
            tm_name
        from ${APP}.dim_sku_info
        where dt='$do_date'
    )t2
    on t1.sku_id=t2.id
    group by recent_days,user_id,tm_id,tm_name
)t3
group by recent_days,tm_id,tm_name;
"

ads_user_action="
with
tmp_page as
(
    select
        '$do_date' dt,
        recent_days,
        sum(if(array_contains(pages,'home'),1,0)) home_count,
        sum(if(array_contains(pages,'good_detail'),1,0)) good_detail_count
    from
    (
        select
            recent_days,
            mid_id,
            collect_set(page_id) pages
        from
        (
            select
                dt,
                mid_id,
                page.page_id
            from ${APP}.dws_visitor_action_daycount lateral view explode(page_stats) tmp as page
            where dt>=date_add('$do_date',-29)
            and page.page_id in('home','good_detail')
        )t1 lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt>=date_add('$do_date',-recent_days+1)
        group by recent_days,mid_id
    )t2
    group by recent_days
),
tmp_cop as
(
    select
        '$do_date' dt,
        recent_days,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from
    (
        select
            recent_days,
            user_id,
            case
                when recent_days=1 then cart_last_1d_count
                when recent_days=7 then cart_last_7d_count
                when recent_days=30 then cart_last_30d_count
            end cart_count,
            case
                when recent_days=1 then order_last_1d_count
                when recent_days=7 then order_last_7d_count
                when recent_days=30 then order_last_30d_count
            end order_count,
            case
                when recent_days=1 then payment_last_1d_count
                when recent_days=7 then payment_last_7d_count
                when recent_days=30 then payment_last_30d_count
            end payment_count
        from ${APP}.dwt_user_topic lateral view explode(Array(1,7,30)) tmp as recent_days
        where dt='$do_date'
    )t1
    group by recent_days
)
insert overwrite table ${APP}.ads_user_action
select * from ${APP}.ads_user_action
union
select
    tmp_page.dt,
    tmp_page.recent_days,
    home_count,
    good_detail_count,
    cart_count,
    order_count,
    payment_count
from tmp_page
join tmp_cop
on tmp_page.recent_days=tmp_cop.recent_days;
"

ads_user_change="
insert overwrite table ${APP}.ads_user_change
select * from ${APP}.ads_user_change
union
select
    churn.dt,
    user_churn_count,
    user_back_count
from
(
    select
        '$do_date' dt,
        count(*) user_churn_count
    from ${APP}.dwt_user_topic
    where dt='$do_date'
    and login_date_last=date_add('$do_date',-7)
)churn
join
(
    select
        '$do_date' dt,
        count(*) user_back_count
    from
    (
        select
            user_id,
            login_date_last
        from ${APP}.dwt_user_topic
        where dt='$do_date'
        and login_date_last='$do_date'
    )t1
    join
    (
        select
            user_id,
            login_date_last login_date_previous
        from ${APP}.dwt_user_topic
        where dt=date_add('$do_date',-1)
    )t2
    on t1.user_id=t2.user_id
    where datediff(login_date_last,login_date_previous)>=8
)back
on churn.dt=back.dt;
"

ads_user_retention="
insert overwrite table ${APP}.ads_user_retention
select * from ${APP}.ads_user_retention
union
select
    '$do_date',
    login_date_first create_date,
    datediff('$do_date',login_date_first) retention_day,
    sum(if(login_date_last='$do_date',1,0)) retention_count,
    count(*) new_user_count,
    cast(sum(if(login_date_last='$do_date',1,0))/count(*)*100 as decimal(16,2)) retention_rate
from ${APP}.dwt_user_topic
where dt='$do_date'
and login_date_first>=date_add('$do_date',-7)
and login_date_first<'$do_date'
group by login_date_first;
"

ads_user_total="
insert overwrite table ${APP}.ads_user_total
select * from ${APP}.ads_user_total
union
select
    '$do_date',
    recent_days,
    sum(if(login_date_first>=recent_days_ago,1,0)) new_user_count,
    sum(if(order_date_first>=recent_days_ago,1,0)) new_order_user_count,
    sum(order_final_amount) order_final_amount,
    sum(if(order_final_amount>0,1,0)) order_user_count,
    sum(if(login_date_last>=recent_days_ago and order_final_amount=0,1,0)) no_order_user_count
from
(
    select
        recent_days,
        user_id,
        login_date_first,
        login_date_last,
        order_date_first,
        case when recent_days=0 then order_final_amount
             when recent_days=1 then order_last_1d_final_amount
             when recent_days=7 then order_last_7d_final_amount
             when recent_days=30 then order_last_30d_final_amount
        end order_final_amount,
        if(recent_days=0,'1970-01-01',date_add('$do_date',-recent_days+1)) recent_days_ago
    from ${APP}.dwt_user_topic lateral view explode(Array(0,1,7,30)) tmp as recent_days
    where dt='$do_date'
)t1
group by recent_days;
"

ads_visit_stats="
insert overwrite table ${APP}.ads_visit_stats
select * from ${APP}.ads_visit_stats
union
select
    '$do_date' dt,
    is_new,
    recent_days,
    channel,
    count(distinct(mid_id)) uv_count,
    cast(sum(duration)/1000 as bigint) duration_sec,
    cast(avg(duration)/1000 as bigint) avg_duration_sec,
    sum(page_count) page_count,
    cast(avg(page_count) as bigint) avg_page_count,
    count(*) sv_count,
    sum(if(page_count=1,1,0)) bounce_count,
    cast(sum(if(page_count=1,1,0))/count(*)*100 as decimal(16,2)) bounce_rate
from
(
    select
        session_id,
        mid_id,
        is_new,
        recent_days,
        channel,
        count(*) page_count,
        sum(during_time) duration
    from
    (
        select
            mid_id,
            channel,
            recent_days,
            is_new,
            last_page_id,
            page_id,
            during_time,
            concat(mid_id,'-',last_value(if(last_page_id is null,ts,null),true) over (partition by recent_days,mid_id order by ts)) session_id
        from
        (
            select
                mid_id,
                channel,
                last_page_id,
                page_id,
                during_time,
                ts,
                recent_days,
                if(visit_date_first>=date_add('$do_date',-recent_days+1),'1','0') is_new
            from
            (
                select
                    t1.mid_id,
                    t1.channel,
                    t1.last_page_id,
                    t1.page_id,
                    t1.during_time,
                    t1.dt,
                    t1.ts,
                    t2.visit_date_first
                from
                (
                    select
                        mid_id,
                        channel,
                        last_page_id,
                        page_id,
                        during_time,
                        dt,
                        ts
                    from ${APP}.dwd_page_log
                    where dt>=date_add('$do_date',-30)
                )t1
                left join
                (
                    select
                        mid_id,
                        visit_date_first
                    from ${APP}.dwt_visitor_topic
                    where dt='$do_date'
                )t2
                on t1.mid_id=t2.mid_id
            )t3 lateral view explode(Array(1,7,30)) tmp as recent_days
            where dt>=date_add('$do_date',-recent_days+1)
        )t4
    )t5
    group by session_id,mid_id,is_new,recent_days,channel
)t6
group by is_new,recent_days,channel;
"

case $1 in
    "ads_activity_stats" )
        hive -e "$ads_activity_stats" 
    ;;
    "ads_coupon_stats" )
        hive -e "$ads_coupon_stats"
    ;;
    "ads_order_by_province" )
        hive -e "$ads_order_by_province" 
    ;;
    "ads_order_spu_stats" )
        hive -e "$ads_order_spu_stats" 
    ;;
    "ads_order_total" )
        hive -e "$ads_order_total" 
    ;;
    "ads_page_path" )
        hive -e "$ads_page_path" 
    ;;
    "ads_repeat_purchase" )
        hive -e "$ads_repeat_purchase" 
    ;;
    "ads_user_action" )
        hive -e "$ads_user_action" 
    ;;
    "ads_user_change" )
        hive -e "$ads_user_change" 
    ;;
    "ads_user_retention" )
        hive -e "$ads_user_retention" 
    ;;
    "ads_user_total" )
        hive -e "$ads_user_total" 
    ;;
    "ads_visit_stats" )
        hive -e "$ads_visit_stats" 
    ;;
    "all" )
        hive -e "$ads_activity_stats$ads_coupon_stats$ads_order_by_province$ads_order_spu_stats$ads_order_total$ads_page_path$ads_repeat_purchase$ads_user_action$ads_user_change$ads_user_retention$ads_user_total$ads_visit_stats"
    ;;
esac
```

（2）增加脚本执行权限

```
[atguigu@hadoop102 bin]$ chmod 777 dwt_to_ads.sh
```

**2）脚本使用**
（1）执行脚本

```
[atguigu@hadoop102 bin]$ dwt_to_ads.sh all 2020-06-14           
```

（2）查看数据是否导入



