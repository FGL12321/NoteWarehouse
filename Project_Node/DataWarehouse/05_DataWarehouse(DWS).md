[TOC]



# 14.0 数仓搭建-DWS层

## 14.1 系统函数

### 14.1.1 nvl函数

1）基本语法

NVL（表达式1，表达式2）

如果表达式1为空值，NVL返回值为表达式2的值，否则返回表达式1的值。

该函数的目的是把一个空值（null）转换成一个实际的值。其表达式的值可以是数字型、字符型和日期型。但是表达式1和表达式2的数据类型必须为同一个类型。

2）案例实操

```
hive (gmall)> select nvl(1,0);
1
hive (gmall)> select nvl(null,"hello");
hello
```

### 14.1.2 日期处理函数

1）date_format函数（根据格式整理日期）

```
hive (gmall)> select date_format('2020-06-14','yyyy-MM');

2020-06
```

2）date_add函数（加减日期）

```sql
hive (gmall)> select date_add('2020-06-14',-1);
2020-06-13

hive (gmall)> select date_add('2020-06-14',1);
2020-06-15
```

3）date_add函数（加减日期）

（1）取当前天的下一个周一

```
hive (gmall)> select next_day('2020-06-14','MO');
2020-06-15
```

说明：星期一到星期日的英文（Monday，Tuesday、Wednesday、Thursday、Friday、Saturday、Sunday）

（2）取当前周的周一

```
hive (gmall)> select date_add(next_day('2020-06-14','MO'),-7);
2020-06-8
```

4）last_day函数（求当月最后一天日期）

```
hive (gmall)> select last_day('2020-06-14');
2020-06-30
```

### 14.1.3 复杂数据类型定义	

1）map结构数据定义

```
map<string,string>
```

2）array结构数据定义

```
array<string>
```

3）struct结构数据定义

```
struct<id:int,name:string,age:int>
```

4）struct和array嵌套定义

```
array<struct<id:int,name:string,age:int>>
```

## 14.2 DWS层

![image-20220927141551119](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220927141551119.png)

![image-20220927141540817](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220927141540817.png)

### 14.2.1 访客主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_visitor_action_daycount;
CREATE EXTERNAL TABLE dws_visitor_action_daycount
(
    `mid_id` STRING COMMENT '设备id',
    `brand` STRING COMMENT '设备品牌',
    `model` STRING COMMENT '设备型号',
    `is_new` STRING COMMENT '是否首次访问',
    `channel` ARRAY<STRING> COMMENT '渠道',
    `os` ARRAY<STRING> COMMENT '操作系统',
    `area_code` ARRAY<STRING> COMMENT '地区ID',
    `version_code` ARRAY<STRING> COMMENT '应用版本',
    `visit_count` BIGINT COMMENT '访问次数',
    `page_stats` ARRAY<STRUCT<page_id:STRING,page_count:BIGINT,during_time:BIGINT>> COMMENT '页面访问统计'
) COMMENT '每日设备行为表'
PARTITIONED BY(`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_visitor_action_daycount'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

```sql
insert overwrite table dws_visitor_action_daycount partition(dt='2020-06-14')
select
    t1.mid_id,
    t1.brand,
    t1.model,
    t1.is_new,
    t1.channel,
    t1.os,
    t1.area_code,
    t1.version_code,
    t1.visit_count,
    t3.page_stats
from
(
    select
        mid_id,
        brand,
        model,
        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,--ods_page_log中，同一天内，同一设备的is_new字段，可能全部为1，可能全部为0，也可能部分为0，部分为1(卸载重装),故做该处理
        collect_set(channel) channel,
        collect_set(os) os,
        collect_set(area_code) area_code,
        collect_set(version_code) version_code,
        sum(if(last_page_id is null,1,0)) visit_count
    from dwd_page_log
    where dt='2020-06-14'
    and last_page_id is null
    group by mid_id,model,brand
)t1
join
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count,'during_time',during_time)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count,
            sum(during_time) during_time
        from dwd_page_log
        where dt='2020-06-14'
        group by mid_id,model,brand,page_id
    )t2
    group by mid_id,model,brand
)t3
on t1.mid_id=t3.mid_id
and t1.brand=t3.brand
and t1.model=t3.model;
```

3）查询加载结果

### 14.2.2 用户主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_user_action_daycount;
CREATE EXTERNAL TABLE dws_user_action_daycount
(
    `user_id` STRING COMMENT '用户id',
    `login_count` BIGINT COMMENT '登录次数',
    `cart_count` BIGINT COMMENT '加入购物车次数',
    `favor_count` BIGINT COMMENT '收藏次数',
    `order_count` BIGINT COMMENT '下单次数',
    `order_activity_count` BIGINT COMMENT '订单参与活动次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '订单减免金额(活动)',
    `order_coupon_count` BIGINT COMMENT '订单用券次数',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '订单减免金额(优惠券)',
    `order_original_amount` DECIMAL(16,2)  COMMENT '订单单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '订单总金额',
    `payment_count` BIGINT COMMENT '支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `refund_order_count` BIGINT COMMENT '退单次数',
    `refund_order_num` BIGINT COMMENT '退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_payment_count` BIGINT COMMENT '退款次数',
    `refund_payment_num` BIGINT COMMENT '退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '退款金额',
    `coupon_get_count` BIGINT COMMENT '优惠券领取次数',
    `coupon_using_count` BIGINT COMMENT '优惠券使用(下单)次数',
    `coupon_used_count` BIGINT COMMENT '优惠券使用(支付)次数',
    `appraise_good_count` BIGINT COMMENT '好评数',
    `appraise_mid_count` BIGINT COMMENT '中评数',
    `appraise_bad_count` BIGINT COMMENT '差评数',
    `appraise_default_count` BIGINT COMMENT '默认评价数',
    `order_detail_stats` array<struct<sku_id:string,sku_num:bigint,order_count:bigint,activity_reduce_amount:decimal(16,2),coupon_reduce_amount:decimal(16,2),original_amount:decimal(16,2),final_amount:decimal(16,2)>> COMMENT '下单明细统计'
) COMMENT '每日用户行为'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_user_action_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```sql
with
tmp_login as
(
    select
        dt,
        user_id,
        count(*) login_count
    from dwd_page_log
    where user_id is not null
    and last_page_id is null
    group by dt,user_id
),
tmp_cf as
(
    select
        dt,
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from dwd_action_log
    where user_id is not null
    and action_id in ('cart_add','favor_add')
    group by dt,user_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),user_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from dwd_refund_payment
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.user_id
),
tmp_coupon as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt) dt,
        coalesce(coupon_get.user_id,coupon_using.user_id,coupon_used.user_id) user_id,
        nvl(coupon_get_count,0) coupon_get_count,
        nvl(coupon_using_count,0) coupon_using_count,
        nvl(coupon_used_count,0) coupon_used_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_get_count
        from dwd_coupon_use
        where get_time is not null
        group by user_id,date_format(get_time,'yyyy-MM-dd')
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_using_count
        from dwd_coupon_use
        where using_time is not null
        group by user_id,date_format(using_time,'yyyy-MM-dd')
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.user_id=coupon_using.user_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_used_count
        from dwd_coupon_use
        where used_time is not null
        group by user_id,date_format(used_time,'yyyy-MM-dd')
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.user_id,coupon_using.user_id)=coupon_used.user_id
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_od as
(
    select
        dt,
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            date_format(create_time,'yyyy-MM-dd') dt,
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from dwd_order_detail
        group by date_format(create_time,'yyyy-MM-dd'),user_id,sku_id
    )t1
    group by dt,user_id
)
insert overwrite table dws_user_action_daycount partition(dt)
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats,
    coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt,tmp_od.dt)
from tmp_login
full outer join tmp_cf
on tmp_login.user_id=tmp_cf.user_id
and tmp_login.dt=tmp_cf.dt
full outer join tmp_order
on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
and coalesce(tmp_login.dt,tmp_cf.dt)=tmp_order.dt
full outer join tmp_pay
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt)=tmp_pay.dt
full outer join tmp_ri
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt)=tmp_ri.dt
full outer join tmp_rp
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt)=tmp_rp.dt
full outer join tmp_comment
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt)=tmp_comment.dt
full outer join tmp_coupon
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt)=tmp_coupon.dt
full outer join tmp_od
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt)=tmp_od.dt;
```

（2）每日装载

```sql
with
tmp_login as
(
    select
        user_id,
        count(*) login_count
    from dwd_page_log
    where dt='2020-06-15'
    and user_id is not null
    and last_page_id is null
    group by user_id
),
tmp_cf as
(
    select
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from dwd_action_log
    where dt='2020-06-15'
    and user_id is not null
    and action_id in ('cart_add','favor_add')
    group by user_id
),
tmp_order as
(
    select
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from dwd_order_info
    where (dt='2020-06-15'
    or dt='9999-99-99')
    and date_format(create_time,'yyyy-MM-dd')='2020-06-15'
    group by user_id
),
tmp_pay as
(
    select
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from dwd_payment_info
    where dt='2020-06-15'
    group by user_id
),
tmp_ri as
(
    select
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    where dt='2020-06-15'
    group by user_id
),
tmp_rp as
(
    select
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount
        from dwd_refund_payment
        where dt='2020-06-15'
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from dwd_order_refund_info
        where dt>=date_add('2020-06-15',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by rp.user_id
),
tmp_coupon as
(
    select
        user_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='2020-06-15',1,0)) coupon_get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='2020-06-15',1,0)) coupon_using_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='2020-06-15',1,0)) coupon_used_count
    from dwd_coupon_use
    where (dt='2020-06-15' or dt='9999-99-99')
    and (date_format(get_time, 'yyyy-MM-dd') = '2020-06-15'
    or date_format(using_time,'yyyy-MM-dd')='2020-06-15'
    or date_format(used_time,'yyyy-MM-dd')='2020-06-15')
    group by user_id
),
tmp_comment as
(
    select
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from dwd_comment_info
    where dt='2020-06-15'
    group by user_id
),
tmp_od as
(
    select
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from dwd_order_detail
        where dt='2020-06-15'
        group by user_id,sku_id
    )t1
    group by user_id
)
insert overwrite table dws_user_action_daycount partition(dt='2020-06-15')
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats
from tmp_login
full outer join tmp_cf on tmp_login.user_id=tmp_cf.user_id
full outer join tmp_order on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
full outer join tmp_pay on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
full outer join tmp_ri on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
full outer join tmp_rp on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
full outer join tmp_comment on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
full outer join tmp_coupon on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
full outer join tmp_od on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id;
```

3）查询加载结果

### 14.2.3 商品主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_sku_action_daycount;
CREATE EXTERNAL TABLE dws_sku_action_daycount
(
    `sku_id` STRING COMMENT 'sku_id',
    `order_count` BIGINT COMMENT '被下单次数',
    `order_num` BIGINT COMMENT '被下单件数',
    `order_activity_count` BIGINT COMMENT '参与活动被下单次数',
    `order_coupon_count` BIGINT COMMENT '使用优惠券被下单次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '优惠金额(活动)',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '优惠金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '被下单原价金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '被下单最终金额',
    `payment_count` BIGINT COMMENT '被支付次数',
    `payment_num` BIGINT COMMENT '被支付件数',
    `payment_amount` DECIMAL(16,2) COMMENT '被支付金额',
    `refund_order_count` BIGINT  COMMENT '被退单次数',
    `refund_order_num` BIGINT COMMENT '被退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '被退单金额',
    `refund_payment_count` BIGINT  COMMENT '被退款次数',
    `refund_payment_num` BIGINT COMMENT '被退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '被退款金额',
    `cart_count` BIGINT COMMENT '被加入购物车次数',
    `favor_count` BIGINT COMMENT '被收藏次数',
    `appraise_good_count` BIGINT COMMENT '好评数',
    `appraise_mid_count` BIGINT COMMENT '中评数',
    `appraise_bad_count` BIGINT COMMENT '差评数',
    `appraise_default_count` BIGINT COMMENT '默认评价数'
) COMMENT '每日商品行为'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_sku_action_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```sql
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from dwd_order_detail od
    join
    (
        select
            order_id,
            callback_time
        from dwd_payment_info
        where callback_time is not null
    )pi on pi.order_id=od.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),sku_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from dwd_refund_payment
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.sku_id
),
tmp_cf as
(
    select
        dt,
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from dwd_action_log
    where action_id in ('cart_add','favor_add')
    group by dt,item
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
)
insert overwrite table dws_sku_action_daycount partition(dt)
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count),
    dt
from
(
    select
        dt,
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by dt,sku_id;
```

（2）每日装载

```sql
with
tmp_order as
(
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    where dt='2020-06-15'
    group by sku_id
),
tmp_pay as
(
    select
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from dwd_order_detail
    where (dt='2020-06-15'
    or dt=date_add('2020-06-15',-1))
    and order_id in
    (
        select order_id from dwd_payment_info where dt='2020-06-15'
    )
    group by sku_id
),
tmp_ri as
(
    select
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    where dt='2020-06-15'
    group by sku_id
),
tmp_rp as
(
    select
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount
        from dwd_refund_payment
        where dt='2020-06-15'
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from dwd_order_refund_info
        where dt>=date_add('2020-06-15',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by rp.sku_id
),
tmp_cf as
(
    select
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from dwd_action_log
    where dt='2020-06-15'
    and action_id in ('cart_add','favor_add')
    group by item
),
tmp_comment as
(
    select
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from dwd_comment_info
    where dt='2020-06-15'
    group by sku_id
)
insert overwrite table dws_sku_action_daycount partition(dt='2020-06-15')
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count)
from
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by sku_id;
```

3）查询加载结果

### 14.2.4 优惠券主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_coupon_info_daycount;
CREATE EXTERNAL TABLE dws_coupon_info_daycount(
    `coupon_id` STRING COMMENT '优惠券ID',
    `get_count` BIGINT COMMENT '被领取次数',
    `order_count` BIGINT COMMENT '被使用(下单)次数', 
    `order_reduce_amount` DECIMAL(16,2) COMMENT '用券下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '用券订单原价金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '用券下单最终金额',
    `payment_count` BIGINT COMMENT '被使用(支付)次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '用券支付优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '用券支付总金额',
    `expire_count` BIGINT COMMENT '过期次数'
) COMMENT '每日活动统计'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_coupon_info_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```sql
with
tmp_cu as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt,coupon_exprie.dt) dt,
        coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id,coupon_exprie.coupon_id) coupon_id,
        nvl(get_count,0) get_count,
        nvl(order_count,0) order_count,
        nvl(payment_count,0) payment_count,
        nvl(expire_count,0) expire_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) get_count
        from dwd_coupon_use
        group by date_format(get_time,'yyyy-MM-dd'),coupon_id
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) order_count
        from dwd_coupon_use
        where using_time is not null
        group by date_format(using_time,'yyyy-MM-dd'),coupon_id
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.coupon_id=coupon_using.coupon_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) payment_count
        from dwd_coupon_use
        where used_time is not null
        group by date_format(used_time,'yyyy-MM-dd'),coupon_id
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.coupon_id,coupon_using.coupon_id)=coupon_used.coupon_id
    full outer join
    (
        select
            date_format(expire_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) expire_count
        from dwd_coupon_use
        where expire_time is not null
        group by date_format(expire_time,'yyyy-MM-dd'),coupon_id
    )coupon_exprie
    on coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt)=coupon_exprie.dt
    and coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id)=coupon_exprie.coupon_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    where coupon_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),coupon_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            order_id,
            coupon_id,
            split_coupon_amount,
            split_final_amount
        from dwd_order_detail
        where coupon_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),coupon_id
)
insert overwrite table dws_coupon_info_daycount partition(dt)
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count),
    dt
from
(
    select
        dt,
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by dt,coupon_id;
```

（2）每日装载

```sql
with
tmp_cu as
(
    select
        coupon_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='2020-06-15',1,0)) get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='2020-06-15',1,0)) order_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='2020-06-15',1,0)) payment_count,
        sum(if(date_format(expire_time,'yyyy-MM-dd')='2020-06-15',1,0)) expire_count
    from dwd_coupon_use
    where dt='9999-99-99'
    or dt='2020-06-15'
    group by coupon_id
),
tmp_order as
(
    select
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    where dt='2020-06-15'
    and coupon_id is not null
    group by coupon_id
),
tmp_pay as
(
    select
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from dwd_order_detail
    where (dt='2020-06-15'
    or dt=date_add('2020-06-15',-1))
    and coupon_id is not null
    and order_id in
    (
        select order_id from dwd_payment_info where dt='2020-06-15'
    )
    group by coupon_id
)
insert overwrite table dws_coupon_info_daycount partition(dt='2020-06-15')
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count)
from
(
    select
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by coupon_id;
```

3）查询加载结果

### 14.2.5 活动主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_activity_info_daycount;
CREATE EXTERNAL TABLE dws_activity_info_daycount(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING COMMENT '活动ID',
    `order_count` BIGINT COMMENT '参与某活动某规则下单次数',    `order_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单减免金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '参与某活动某规则下单最终金额',
    `payment_count` BIGINT COMMENT '参与某活动某规则支付次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则支付减免金额',
    `payment_amount` DECIMAL(16,2) COMMENT '参与某活动某规则支付金额'
) COMMENT '每日活动统计'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_activity_info_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```sql
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    where activity_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),activity_rule_id,activity_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            activity_rule_id,
            activity_id,
            order_id,
            split_activity_amount,
            split_final_amount
        from dwd_order_detail
        where activity_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),activity_rule_id,activity_id
)
insert overwrite table dws_activity_info_daycount partition(dt)
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    dt
from
(
    select
        dt,
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        dt,
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by dt,activity_rule_id,activity_id;
```

（2）每日装载

```sql
with
tmp_order as
(
    select
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    where dt='2020-06-15'
    and activity_id is not null
    group by activity_rule_id,activity_id
),
tmp_pay as
(
    select
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from dwd_order_detail
    where (dt='2020-06-15'
    or dt=date_add('2020-06-15',-1))
    and activity_id is not null
    and order_id in
    (
        select order_id from dwd_payment_info where dt='2020-06-15'
    )
    group by activity_rule_id,activity_id
)
insert overwrite table dws_activity_info_daycount partition(dt='2020-06-15')
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by activity_rule_id,activity_id;
```

3）查询加载结果

### 14.2.6 地区主题

1）建表语句

```sql
DROP TABLE IF EXISTS dws_area_stats_daycount;
CREATE EXTERNAL TABLE dws_area_stats_daycount(
    `province_id` STRING COMMENT '地区编号',
    `visit_count` BIGINT COMMENT '访问次数',
    `login_count` BIGINT COMMENT '登录次数',
    `visitor_count` BIGINT COMMENT '访客人数',
    `user_count` BIGINT COMMENT '用户人数',
    `order_count` BIGINT COMMENT '下单次数',
    `order_original_amount` DECIMAL(16,2) COMMENT '下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '下单最终金额',
    `payment_count` BIGINT COMMENT '支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `refund_order_count` BIGINT COMMENT '退单次数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_payment_count` BIGINT COMMENT '退款次数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '退款金额'
) COMMENT '每日地区统计表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dws/dws_area_stats_daycount/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```sql
with
tmp_vu as
(
    select
        dt,
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            dt,
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from dwd_page_log
        where last_page_id is null
        group by dt,area_code
    )tmp
    left join dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
),
tmp_ro as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from dwd_refund_payment
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
)
insert overwrite table dws_area_stats_daycount partition(dt)
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount),
    dt
from
(
    select
        dt,
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by dt,province_id;
```

（2）每日装载

```sql
with
tmp_vu as
(
    select
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from dwd_page_log
        where dt='2020-06-15'
        and last_page_id is null
        group by area_code
    )tmp
    left join dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from dwd_order_info
    where dt='2020-06-15'
    or dt='9999-99-99'
    and date_format(create_time,'yyyy-MM-dd')='2020-06-15'
    group by province_id
),
tmp_pay as
(
    select
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from dwd_payment_info
    where dt='2020-06-15'
    group by province_id
),
tmp_ro as
(
    select
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    where dt='2020-06-15'
    group by province_id
),
tmp_rp as
(
    select
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from dwd_refund_payment
    where dt='2020-06-15'
    group by province_id
)
insert overwrite table dws_area_stats_daycount partition(dt='2020-06-15')
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount)
from
(
    select
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by province_id;
```

### 14.2.7 DWS层首日数据装载脚本

1）编写脚本
（1）在/home/atguigu/bin目录下创建脚本dwd_to_dws_init.sh

```sql
#!/bin/bash

APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi

dws_visitor_action_daycount="
insert overwrite table ${APP}.dws_visitor_action_daycount partition(dt='$do_date')
select
    t1.mid_id,
    t1.brand,
    t1.model,
    t1.is_new,
    t1.channel,
    t1.os,
    t1.area_code,
    t1.version_code,
    t1.visit_count,
    t3.page_stats
from
(
    select
        mid_id,
        brand,
        model,
        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,--ods_page_log中，同一天内，同一设备的is_new字段，可能全部为1，可能全部为0，也可能部分为0，部分为1(卸载重装),故做该处理
        collect_set(channel) channel,
        collect_set(os) os,
        collect_set(area_code) area_code,
        collect_set(version_code) version_code,
        sum(if(last_page_id is null,1,0)) visit_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and last_page_id is null
    group by mid_id,model,brand
)t1
join
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count,'during_time',during_time)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count,
            sum(during_time) during_time
        from ${APP}.dwd_page_log
        where dt='$do_date'
        group by mid_id,model,brand,page_id
    )t2
    group by mid_id,model,brand
)t3
on t1.mid_id=t3.mid_id
and t1.brand=t3.brand
and t1.model=t3.model;
"

dws_area_stats_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_vu as
(
    select
        dt,
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            dt,
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from ${APP}.dwd_page_log
        where last_page_id is null
        group by dt,area_code
    )tmp
    left join ${APP}.dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
),
tmp_ro as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),province_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from ${APP}.dwd_refund_payment
    group by date_format(callback_time,'yyyy-MM-dd'),province_id
)
insert overwrite table ${APP}.dws_area_stats_daycount partition(dt)
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount),
    dt
from
(
    select
        dt,
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        dt,
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by dt,province_id;
"

dws_user_action_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_login as
(
    select
        dt,
        user_id,
        count(*) login_count
    from ${APP}.dwd_page_log
    where user_id is not null
    and last_page_id is null
    group by dt,user_id
),
tmp_cf as
(
    select
        dt,
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where user_id is not null
    and action_id in ('cart_add','favor_add')
    group by dt,user_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    group by date_format(callback_time,'yyyy-MM-dd'),user_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from ${APP}.dwd_refund_payment
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.user_id
),
tmp_coupon as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt) dt,
        coalesce(coupon_get.user_id,coupon_using.user_id,coupon_used.user_id) user_id,
        nvl(coupon_get_count,0) coupon_get_count,
        nvl(coupon_using_count,0) coupon_using_count,
        nvl(coupon_used_count,0) coupon_used_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_get_count
        from ${APP}.dwd_coupon_use
        where get_time is not null
        group by user_id,date_format(get_time,'yyyy-MM-dd')
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_using_count
        from ${APP}.dwd_coupon_use
        where using_time is not null
        group by user_id,date_format(using_time,'yyyy-MM-dd')
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.user_id=coupon_using.user_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            user_id,
            count(*) coupon_used_count
        from ${APP}.dwd_coupon_use
        where used_time is not null
        group by user_id,date_format(used_time,'yyyy-MM-dd')
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.user_id,coupon_using.user_id)=coupon_used.user_id
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),user_id
),
tmp_od as
(
    select
        dt,
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            date_format(create_time,'yyyy-MM-dd') dt,
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from ${APP}.dwd_order_detail
        group by date_format(create_time,'yyyy-MM-dd'),user_id,sku_id
    )t1
    group by dt,user_id
)
insert overwrite table ${APP}.dws_user_action_daycount partition(dt)
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats,
    coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt,tmp_od.dt)
from tmp_login
full outer join tmp_cf
on tmp_login.user_id=tmp_cf.user_id
and tmp_login.dt=tmp_cf.dt
full outer join tmp_order
on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
and coalesce(tmp_login.dt,tmp_cf.dt)=tmp_order.dt
full outer join tmp_pay
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt)=tmp_pay.dt
full outer join tmp_ri
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt)=tmp_ri.dt
full outer join tmp_rp
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt)=tmp_rp.dt
full outer join tmp_comment
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt)=tmp_comment.dt
full outer join tmp_coupon
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt)=tmp_coupon.dt
full outer join tmp_od
on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id
and coalesce(tmp_login.dt,tmp_cf.dt,tmp_order.dt,tmp_pay.dt,tmp_ri.dt,tmp_rp.dt,tmp_comment.dt,tmp_coupon.dt)=tmp_od.dt;
"

dws_activity_info_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where activity_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),activity_rule_id,activity_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            activity_rule_id,
            activity_id,
            order_id,
            split_activity_amount,
            split_final_amount
        from ${APP}.dwd_order_detail
        where activity_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),activity_rule_id,activity_id
)
insert overwrite table ${APP}.dws_activity_info_daycount partition(dt)
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    dt
from
(
    select
        dt,
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        dt,
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by dt,activity_rule_id,activity_id;"

dws_sku_action_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
        where callback_time is not null
    )pi on pi.order_id=od.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),sku_id
),
tmp_ri as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
),
tmp_rp as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from ${APP}.dwd_refund_payment
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.sku_id
),
tmp_cf as
(
    select
        dt,
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where action_id in ('cart_add','favor_add')
    group by dt,item
),
tmp_comment as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
)
insert overwrite table ${APP}.dws_sku_action_daycount partition(dt)
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count),
    dt
from
(
    select
        dt,
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        dt,
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by dt,sku_id;"

dws_coupon_info_daycount="
set hive.exec.dynamic.partition.mode=nonstrict;
with
tmp_cu as
(
    select
        coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt,coupon_exprie.dt) dt,
        coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id,coupon_exprie.coupon_id) coupon_id,
        nvl(get_count,0) get_count,
        nvl(order_count,0) order_count,
        nvl(payment_count,0) payment_count,
        nvl(expire_count,0) expire_count
    from
    (
        select
            date_format(get_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) get_count
        from ${APP}.dwd_coupon_use
        group by date_format(get_time,'yyyy-MM-dd'),coupon_id
    )coupon_get
    full outer join
    (
        select
            date_format(using_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) order_count
        from ${APP}.dwd_coupon_use
        where using_time is not null
        group by date_format(using_time,'yyyy-MM-dd'),coupon_id
    )coupon_using
    on coupon_get.dt=coupon_using.dt
    and coupon_get.coupon_id=coupon_using.coupon_id
    full outer join
    (
        select
            date_format(used_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) payment_count
        from ${APP}.dwd_coupon_use
        where used_time is not null
        group by date_format(used_time,'yyyy-MM-dd'),coupon_id
    )coupon_used
    on nvl(coupon_get.dt,coupon_using.dt)=coupon_used.dt
    and nvl(coupon_get.coupon_id,coupon_using.coupon_id)=coupon_used.coupon_id
    full outer join
    (
        select
            date_format(expire_time,'yyyy-MM-dd') dt,
            coupon_id,
            count(*) expire_count
        from ${APP}.dwd_coupon_use
        where expire_time is not null
        group by date_format(expire_time,'yyyy-MM-dd'),coupon_id
    )coupon_exprie
    on coalesce(coupon_get.dt,coupon_using.dt,coupon_used.dt)=coupon_exprie.dt
    and coalesce(coupon_get.coupon_id,coupon_using.coupon_id,coupon_used.coupon_id)=coupon_exprie.coupon_id
),
tmp_order as
(
    select
        date_format(create_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where coupon_id is not null
    group by date_format(create_time,'yyyy-MM-dd'),coupon_id
),
tmp_pay as
(
    select
        date_format(callback_time,'yyyy-MM-dd') dt,
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from
    (
        select
            order_id,
            coupon_id,
            split_coupon_amount,
            split_final_amount
        from ${APP}.dwd_order_detail
        where coupon_id is not null
    )od
    join
    (
        select
            order_id,
            callback_time
        from ${APP}.dwd_payment_info
    )pi
    on od.order_id=pi.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),coupon_id
)
insert overwrite table ${APP}.dws_coupon_info_daycount partition(dt)
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count),
    dt
from
(
    select
        dt,
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        dt,
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by dt,coupon_id;
"

case $1 in
    "dws_visitor_action_daycount" )
        hive -e "$dws_visitor_action_daycount"
    ;;
    "dws_user_action_daycount" )
        hive -e "$dws_user_action_daycount"
    ;;
    "dws_activity_info_daycount" )
        hive -e "$dws_activity_info_daycount"
    ;;
    "dws_area_stats_daycount" )
        hive -e "$dws_area_stats_daycount"
    ;;
    "dws_sku_action_daycount" )
        hive -e "$dws_sku_action_daycount"
    ;;
    "dws_coupon_info_daycount" )
        hive -e "$dws_coupon_info_daycount"
    ;;
    "all" )
        hive -e "$dws_visitor_action_daycount$dws_user_action_daycount$dws_activity_info_daycount$dws_area_stats_daycount$dws_sku_action_daycount$dws_coupon_info_daycount"
    ;;
esac
```

（2）增加执行权限

```
[atguigu@hadoop102 bin]$ chmod +x dwd_to_dws_init.sh
```

2）脚本使用
（1）执行脚本

```
[atguigu@hadoop102 bin]$ dwd_to_dws_init.sh all 2020-06-14
```

（2）查看数据是否导入成功

### 14.2.8 DWS层每日数据装载脚本

1）编写脚本

（1）在/home/atguigu/bin目录下创建脚本dwd_to_dws.sh

```sql
#!/bin/bash

APP=gmall
# 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

dws_visitor_action_daycount="insert overwrite table ${APP}.dws_visitor_action_daycount partition(dt='$do_date')
select
    t1.mid_id,
    t1.brand,
    t1.model,
    t1.is_new,
    t1.channel,
    t1.os,
    t1.area_code,
    t1.version_code,
    t1.visit_count,
    t3.page_stats
from
(
    select
        mid_id,
        brand,
        model,
        if(array_contains(collect_set(is_new),'0'),'0','1') is_new,--ods_page_log中，同一天内，同一设备的is_new字段，可能全部为1，可能全部为0，也可能部分为0，部分为1(卸载重装),故做该处理
        collect_set(channel) channel,
        collect_set(os) os,
        collect_set(area_code) area_code,
        collect_set(version_code) version_code,
        sum(if(last_page_id is null,1,0)) visit_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and last_page_id is null
    group by mid_id,model,brand
)t1
join
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count,'during_time',during_time)) page_stats
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count,
            sum(during_time) during_time
        from ${APP}.dwd_page_log
        where dt='$do_date'
        group by mid_id,model,brand,page_id
    )t2
    group by mid_id,model,brand
)t3
on t1.mid_id=t3.mid_id
and t1.brand=t3.brand
and t1.model=t3.model;"

dws_user_action_daycount="
with
tmp_login as
(
    select
        user_id,
        count(*) login_count
    from ${APP}.dwd_page_log
    where dt='$do_date'
    and user_id is not null
    and last_page_id is null
    group by user_id
),
tmp_cf as
(
    select
        user_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and user_id is not null
    and action_id in ('cart_add','favor_add')
    group by user_id
),
tmp_order as
(
    select
        user_id,
        count(*) order_count,
        sum(if(activity_reduce_amount>0,1,0)) order_activity_count,
        sum(if(coupon_reduce_amount>0,1,0)) order_coupon_count,
        sum(activity_reduce_amount) order_activity_reduce_amount,
        sum(coupon_reduce_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    where (dt='$do_date'
    or dt='9999-99-99')
    and date_format(create_time,'yyyy-MM-dd')='$do_date'
    group by user_id
),
tmp_pay as
(
    select
        user_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    where dt='$do_date'
    group by user_id
),
tmp_ri as
(
    select
        user_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by user_id
),
tmp_rp as
(
    select
        rp.user_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(rp.refund_amount) refund_payment_amount
    from
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_amount
        from ${APP}.dwd_refund_payment
        where dt='$do_date'
    )rp
    left join
    (
        select
            user_id,
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
        where dt>=date_add('$do_date',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=rp.sku_id
    group by rp.user_id
),
tmp_coupon as
(
    select
        user_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_using_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='$do_date',1,0)) coupon_used_count
    from ${APP}.dwd_coupon_use
    where (dt='$do_date' or dt='9999-99-99')
    and (date_format(get_time, 'yyyy-MM-dd') = '$do_date'
    or date_format(using_time,'yyyy-MM-dd')='$do_date'
    or date_format(used_time,'yyyy-MM-dd')='$do_date')
    group by user_id
),
tmp_comment as
(
    select
        user_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    where dt='$do_date'
    group by user_id
),
tmp_od as
(
    select
        user_id,
        collect_set(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'activity_reduce_amount',activity_reduce_amount,'coupon_reduce_amount',coupon_reduce_amount,'original_amount',original_amount,'final_amount',final_amount)) order_detail_stats
    from
    (
        select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            cast(sum(split_activity_amount) as decimal(16,2)) activity_reduce_amount,
            cast(sum(split_coupon_amount) as decimal(16,2)) coupon_reduce_amount,
            cast(sum(original_amount) as decimal(16,2)) original_amount,
            cast(sum(split_final_amount) as decimal(16,2)) final_amount
        from ${APP}.dwd_order_detail
        where dt='$do_date'
        group by user_id,sku_id
    )t1
    group by user_id
)
insert overwrite table ${APP}.dws_user_action_daycount partition(dt='$do_date')
select
    coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id,tmp_od.user_id),
    nvl(login_count,0),
    nvl(cart_count,0),
    nvl(favor_count,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0),
    order_detail_stats
from tmp_login
full outer join tmp_cf on tmp_login.user_id=tmp_cf.user_id
full outer join tmp_order on coalesce(tmp_login.user_id,tmp_cf.user_id)=tmp_order.user_id
full outer join tmp_pay on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id)=tmp_pay.user_id
full outer join tmp_ri on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id)=tmp_ri.user_id
full outer join tmp_rp on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id)=tmp_rp.user_id
full outer join tmp_comment on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id)=tmp_comment.user_id
full outer join tmp_coupon on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id)=tmp_coupon.user_id
full outer join tmp_od on coalesce(tmp_login.user_id,tmp_cf.user_id,tmp_order.user_id,tmp_pay.user_id,tmp_ri.user_id,tmp_rp.user_id,tmp_comment.user_id,tmp_coupon.user_id)=tmp_od.user_id;
"


dws_activity_info_daycount="
with
tmp_order as
(
    select
        activity_rule_id,
        activity_id,
        count(*) order_count,
        sum(split_activity_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    and activity_id is not null
    group by activity_rule_id,activity_id
),
tmp_pay as
(
    select
        activity_rule_id,
        activity_id,
        count(*) payment_count,
        sum(split_activity_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and activity_id is not null
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by activity_rule_id,activity_id
)
insert overwrite table ${APP}.dws_activity_info_daycount partition(dt='$do_date')
select
    activity_rule_id,
    activity_id,
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount
    from tmp_order
    union all
    select
        activity_rule_id,
        activity_id,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from tmp_pay
)t1
group by activity_rule_id,activity_id;"


dws_sku_action_daycount="
with
tmp_order as
(
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    group by sku_id
),
tmp_pay as
(
    select
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by sku_id
),
tmp_ri as
(
    select
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by sku_id
),
tmp_rp as
(
    select
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount
        from ${APP}.dwd_refund_payment
        where dt='$do_date'
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from ${APP}.dwd_order_refund_info
        where dt>=date_add('$do_date',-15)
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by rp.sku_id
),
tmp_cf as
(
    select
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from ${APP}.dwd_action_log
    where dt='$do_date'
    and action_id in ('cart_add','favor_add')
    group by item
),
tmp_comment as
(
    select
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from ${APP}.dwd_comment_info
    where dt='$do_date'
    group by sku_id
)
insert overwrite table ${APP}.dws_sku_action_daycount partition(dt='$do_date')
select
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_activity_count),
    sum(order_coupon_count),
    sum(order_activity_reduce_amount),
    sum(order_coupon_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_num),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_num),
    sum(refund_payment_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count)
from
(
    select
        sku_id,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_pay
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_ri
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_rp
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cf
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_activity_count,
        0 order_coupon_count,
        0 order_activity_reduce_amount,
        0 order_coupon_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_num,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_num,
        0 refund_payment_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_comment
)t1
group by sku_id;"

dws_coupon_info_daycount="
with
tmp_cu as
(
    select
        coupon_id,
        sum(if(date_format(get_time,'yyyy-MM-dd')='$do_date',1,0)) get_count,
        sum(if(date_format(using_time,'yyyy-MM-dd')='$do_date',1,0)) order_count,
        sum(if(date_format(used_time,'yyyy-MM-dd')='$do_date',1,0)) payment_count,
        sum(if(date_format(expire_time,'yyyy-MM-dd')='$do_date',1,0)) expire_count
    from ${APP}.dwd_coupon_use
    where dt='9999-99-99'
    or dt='$do_date'
    group by coupon_id
),
tmp_order as
(
    select
        coupon_id,
        sum(split_coupon_amount) order_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from ${APP}.dwd_order_detail
    where dt='$do_date'
    and coupon_id is not null
    group by coupon_id
),
tmp_pay as
(
    select
        coupon_id,
        sum(split_coupon_amount) payment_reduce_amount,
        sum(split_final_amount) payment_amount
    from ${APP}.dwd_order_detail
    where (dt='$do_date'
    or dt=date_add('$do_date',-1))
    and coupon_id is not null
    and order_id in
    (
        select order_id from ${APP}.dwd_payment_info where dt='$do_date'
    )
    group by coupon_id
)
insert overwrite table ${APP}.dws_coupon_info_daycount partition(dt='$do_date')
select
    coupon_id,
    sum(get_count),
    sum(order_count),
    sum(order_reduce_amount),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_reduce_amount),
    sum(payment_amount),
    sum(expire_count)
from
(
    select
        coupon_id,
        get_count,
        order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        expire_count
    from tmp_cu
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_reduce_amount,
        0 payment_amount,
        0 expire_count
    from tmp_order
    union all
    select
        coupon_id,
        0 get_count,
        0 order_count,
        0 order_reduce_amount,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        payment_reduce_amount,
        payment_amount,
        0 expire_count
    from tmp_pay
)t1
group by coupon_id;"


dws_area_stats_daycount="
with
tmp_vu as
(
    select
        id province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count
    from
    (
        select
            area_code,
            count(*) visit_count,--访客访问次数
            count(user_id) login_count,--用户访问次数,等价于sum(if(user_id is not null,1,0))
            count(distinct(mid_id)) visitor_count,--访客人数
            count(distinct(user_id)) user_count--用户人数
        from ${APP}.dwd_page_log
        where dt='$do_date'
        and last_page_id is null
        group by area_code
    )tmp
    left join ${APP}.dim_base_province area
    on tmp.area_code=area.area_code
),
tmp_order as
(
    select
        province_id,
        count(*) order_count,
        sum(original_amount) order_original_amount,
        sum(final_amount) order_final_amount
    from ${APP}.dwd_order_info
    where dt='$do_date'
    or dt='9999-99-99'
    and date_format(create_time,'yyyy-MM-dd')='$do_date'
    group by province_id
),
tmp_pay as
(
    select
        province_id,
        count(*) payment_count,
        sum(payment_amount) payment_amount
    from ${APP}.dwd_payment_info
    where dt='$do_date'
    group by province_id
),
tmp_ro as
(
    select
        province_id,
        count(*) refund_order_count,
        sum(refund_amount) refund_order_amount
    from ${APP}.dwd_order_refund_info
    where dt='$do_date'
    group by province_id
),
tmp_rp as
(
    select
        province_id,
        count(*) refund_payment_count,
        sum(refund_amount) refund_payment_amount
    from ${APP}.dwd_refund_payment
    where dt='$do_date'
    group by province_id
)
insert overwrite table ${APP}.dws_area_stats_daycount partition(dt='$do_date')
select
    province_id,
    sum(visit_count),
    sum(login_count),
    sum(visitor_count),
    sum(user_count),
    sum(order_count),
    sum(order_original_amount),
    sum(order_final_amount),
    sum(payment_count),
    sum(payment_amount),
    sum(refund_order_count),
    sum(refund_order_amount),
    sum(refund_payment_count),
    sum(refund_payment_amount)
from
(
    select
        province_id,
        visit_count,
        login_count,
        visitor_count,
        user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_vu
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        order_count,
        order_original_amount,
        order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_order
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        payment_count,
        payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_pay
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        refund_order_count,
        refund_order_amount,
        0 refund_payment_count,
        0 refund_payment_amount
    from tmp_ro
    union all
    select
        province_id,
        0 visit_count,
        0 login_count,
        0 visitor_count,
        0 user_count,
        0 order_count,
        0 order_original_amount,
        0 order_final_amount,
        0 payment_count,
        0 payment_amount,
        0 refund_order_count,
        0 refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from tmp_rp
)t1
group by province_id;"

case $1 in
    "dws_visitor_action_daycount" )
        hive -e "$dws_visitor_action_daycount"
    ;;
    "dws_user_action_daycount" )
        hive -e "$dws_user_action_daycount"
    ;;
    "dws_activity_info_daycount" )
        hive -e "$dws_activity_info_daycount"
    ;;
    "dws_area_stats_daycount" )
        hive -e "$dws_area_stats_daycount"
    ;;
    "dws_sku_action_daycount" )
        hive -e "$dws_sku_action_daycount"
    ;;
    "dws_coupon_info_daycount" )
        hive -e "$dws_coupon_info_daycount"
    ;;
    "all" )
        hive -e "$dws_visitor_action_daycount$dws_user_action_daycount$dws_activity_info_daycount$dws_area_stats_daycount$dws_sku_action_daycount$dws_coupon_info_daycount"
    ;;
esac
```

（2）增加执行权限

```sql
[atguigu@hadoop102 bin]$ chmod +x dwd_to_dws.sh
```

**2）脚本使用**
（1）执行脚本

```sql
[atguigu@hadoop102 bin]$ dwd_to_dws.sh all 2020-06-14
```

（2）查看数据是否导入成功

