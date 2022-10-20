[TOC]



# 15.0 数仓搭建-DWT层

## 15.1 访客主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_visitor_topic;
CREATE EXTERNAL TABLE dwt_visitor_topic
(
    `mid_id` STRING COMMENT '设备id',
    `brand` STRING COMMENT '手机品牌',
    `model` STRING COMMENT '手机型号',
    `channel` ARRAY<STRING> COMMENT '渠道',
    `os` ARRAY<STRING> COMMENT '操作系统',
    `area_code` ARRAY<STRING> COMMENT '地区ID',
    `version_code` ARRAY<STRING> COMMENT '应用版本',
    `visit_date_first` STRING  COMMENT '首次访问时间',
    `visit_date_last` STRING  COMMENT '末次访问时间',
    `visit_last_1d_count` BIGINT COMMENT '最近1日访问次数',
    `visit_last_1d_day_count` BIGINT COMMENT '最近1日访问天数',
    `visit_last_7d_count` BIGINT COMMENT '最近7日访问次数',
    `visit_last_7d_day_count` BIGINT COMMENT '最近7日访问天数',
    `visit_last_30d_count` BIGINT COMMENT '最近30日访问次数',
    `visit_last_30d_day_count` BIGINT COMMENT '最近30日访问天数',
    `visit_count` BIGINT COMMENT '累积访问次数',
    `visit_day_count` BIGINT COMMENT '累积访问天数'
) COMMENT '设备主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_visitor_topic'
TBLPROPERTIES ("parquet.compression"="lzo");

```

2）数据装载

```
insert overwrite table dwt_visitor_topic partition(dt='2020-06-14')
select
    nvl(1d_ago.mid_id,old.mid_id),
    nvl(1d_ago.brand,old.brand),
    nvl(1d_ago.model,old.model),
    nvl(1d_ago.channel,old.channel),
    nvl(1d_ago.os,old.os),
    nvl(1d_ago.area_code,old.area_code),
    nvl(1d_ago.version_code,old.version_code),
    case when old.mid_id is null and 1d_ago.is_new=1 then '2020-06-14'
         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-13'--无法获取准确的首次登录日期，给定一个数仓搭建日之前的日期
         else old.visit_date_first end,
    if(1d_ago.mid_id is not null,'2020-06-14',old.visit_date_last),
    nvl(1d_ago.visit_count,0),
    if(1d_ago.mid_id is null,0,1),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.visit_last_7d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(7d_ago.mid_id is null,0,1),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.visit_last_30d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(30d_ago.mid_id is null,0,1),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.visit_day_count,0)+if(1d_ago.mid_id is null,0,1)
from
(
    select
        mid_id,
        brand,
        model,
        channel,
        os,
        area_code,
        version_code,
        visit_date_first,
        visit_date_last,
        visit_last_1d_count,
        visit_last_1d_day_count,
        visit_last_7d_count,
        visit_last_7d_day_count,
        visit_last_30d_count,
        visit_last_30d_day_count,
        visit_count,
        visit_day_count
    from dwt_visitor_topic
    where dt=date_add('2020-06-14',-1)
)old
full outer join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from dws_visitor_action_daycount
    where dt='2020-06-14'
)1d_ago
on old.mid_id=1d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from dws_visitor_action_daycount
    where dt=date_add('2020-06-14',-7)
)7d_ago
on old.mid_id=7d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from dws_visitor_action_daycount
    where dt=date_add('2020-06-14',-30)
)30d_ago
on old.mid_id=30d_ago.mid_id;

```

3）查询加载结果

## 15.2 用户主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_user_topic;
CREATE EXTERNAL TABLE dwt_user_topic
(
    `user_id` STRING  COMMENT '用户id',
    `login_date_first` STRING COMMENT '首次活跃日期',
    `login_date_last` STRING COMMENT '末次活跃日期',
    `login_date_1d_count` STRING COMMENT '最近1日登录次数',
    `login_last_1d_day_count` BIGINT COMMENT '最近1日登录天数',
    `login_last_7d_count` BIGINT COMMENT '最近7日登录次数',
    `login_last_7d_day_count` BIGINT COMMENT '最近7日登录天数',
    `login_last_30d_count` BIGINT COMMENT '最近30日登录次数',
    `login_last_30d_day_count` BIGINT COMMENT '最近30日登录天数',
    `login_count` BIGINT COMMENT '累积登录次数',
    `login_day_count` BIGINT COMMENT '累积登录天数',
    `order_date_first` STRING COMMENT '首次下单时间',
    `order_date_last` STRING COMMENT '末次下单时间',
    `order_last_1d_count` BIGINT COMMENT '最近1日下单次数',
    `order_activity_last_1d_count` BIGINT COMMENT '最近1日订单参与活动次数',
    `order_activity_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日订单减免金额(活动)',
    `order_coupon_last_1d_count` BIGINT COMMENT '最近1日下单用券次数',
    `order_coupon_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日订单减免金额(优惠券)',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日原始下单金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日最终下单金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日下单次数',
    `order_activity_last_7d_count` BIGINT COMMENT '最近7日订单参与活动次数',
    `order_activity_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日订单减免金额(活动)',
    `order_coupon_last_7d_count` BIGINT COMMENT '最近7日下单用券次数',
    `order_coupon_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日订单减免金额(优惠券)',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日原始下单金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日最终下单金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日下单次数',
    `order_activity_last_30d_count` BIGINT COMMENT '最近30日订单参与活动次数',
    `order_activity_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日订单减免金额(活动)',
    `order_coupon_last_30d_count` BIGINT COMMENT '最近30日下单用券次数',
    `order_coupon_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日订单减免金额(优惠券)',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日原始下单金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日最终下单金额',
    `order_count` BIGINT COMMENT '累积下单次数',
    `order_activity_count` BIGINT COMMENT '累积订单参与活动次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '累积订单减免金额(活动)',
    `order_coupon_count` BIGINT COMMENT '累积下单用券次数',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '累积订单减免金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积原始下单金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积最终下单金额',
    `payment_date_first` STRING COMMENT '首次支付时间',
    `payment_date_last` STRING COMMENT '末次支付时间',
    `payment_last_1d_count` BIGINT COMMENT '最近1日支付次数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日支付次数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日支付次数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日支付金额',
    `payment_count` BIGINT COMMENT '累积支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1日退单次数',
    `refund_order_last_1d_num` BIGINT COMMENT '最近1日退单件数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7日退单次数',
    `refund_order_last_7d_num` BIGINT COMMENT '最近7日退单件数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30日退单次数',
    `refund_order_last_30d_num` BIGINT COMMENT '最近30日退单件数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_num` BIGINT COMMENT '累积退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1日退款次数',
    `refund_payment_last_1d_num` BIGINT COMMENT '最近1日退款件数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7日退款次数',
    `refund_payment_last_7d_num` BIGINT COMMENT '最近7日退款件数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30日退款次数',
    `refund_payment_last_30d_num` BIGINT COMMENT '最近30日退款件数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_num` BIGINT COMMENT '累积退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额',
    `cart_last_1d_count` BIGINT COMMENT '最近1日加入购物车次数',
    `cart_last_7d_count` BIGINT COMMENT '最近7日加入购物车次数',
    `cart_last_30d_count` BIGINT COMMENT '最近30日加入购物车次数',
    `cart_count` BIGINT COMMENT '累积加入购物车次数',
    `favor_last_1d_count` BIGINT COMMENT '最近1日收藏次数',
    `favor_last_7d_count` BIGINT COMMENT '最近7日收藏次数',
    `favor_last_30d_count` BIGINT COMMENT '最近30日收藏次数',
    `favor_count` BIGINT COMMENT '累积收藏次数',
    `coupon_last_1d_get_count` BIGINT COMMENT '最近1日领券次数',
    `coupon_last_1d_using_count` BIGINT COMMENT '最近1日用券(下单)次数',
    `coupon_last_1d_used_count` BIGINT COMMENT '最近1日用券(支付)次数',
    `coupon_last_7d_get_count` BIGINT COMMENT '最近7日领券次数',
    `coupon_last_7d_using_count` BIGINT COMMENT '最近7日用券(下单)次数',
    `coupon_last_7d_used_count` BIGINT COMMENT '最近7日用券(支付)次数',
    `coupon_last_30d_get_count` BIGINT COMMENT '最近30日领券次数',
    `coupon_last_30d_using_count` BIGINT COMMENT '最近30日用券(下单)次数',
    `coupon_last_30d_used_count` BIGINT COMMENT '最近30日用券(支付)次数',
    `coupon_get_count` BIGINT COMMENT '累积领券次数',
    `coupon_using_count` BIGINT COMMENT '累积用券(下单)次数',
    `coupon_used_count` BIGINT COMMENT '累积用券(支付)次数',
    `appraise_last_1d_good_count` BIGINT COMMENT '最近1日好评次数',
    `appraise_last_1d_mid_count` BIGINT COMMENT '最近1日中评次数',
    `appraise_last_1d_bad_count` BIGINT COMMENT '最近1日差评次数',
    `appraise_last_1d_default_count` BIGINT COMMENT '最近1日默认评价次数',
    `appraise_last_7d_good_count` BIGINT COMMENT '最近7日好评次数',
    `appraise_last_7d_mid_count` BIGINT COMMENT '最近7日中评次数',
    `appraise_last_7d_bad_count` BIGINT COMMENT '最近7日差评次数',
    `appraise_last_7d_default_count` BIGINT COMMENT '最近7日默认评价次数',
    `appraise_last_30d_good_count` BIGINT COMMENT '最近30日好评次数',
    `appraise_last_30d_mid_count` BIGINT COMMENT '最近30日中评次数',
    `appraise_last_30d_bad_count` BIGINT COMMENT '最近30日差评次数',
    `appraise_last_30d_default_count` BIGINT COMMENT '最近30日默认评价次数',
    `appraise_good_count` BIGINT COMMENT '累积好评次数',
    `appraise_mid_count` BIGINT COMMENT '累积中评次数',
    `appraise_bad_count` BIGINT COMMENT '累积差评次数',
    `appraise_default_count` BIGINT COMMENT '累积默认评价次数'
)COMMENT '会员主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_user_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

```

2）数据装载

![image-20220928140108503](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220928140108503.png)

（1）首日装载

```
insert overwrite table dwt_user_topic partition(dt='2020-06-14')
select
    id,
    login_date_first,--以用户的创建日期作为首次登录日期
    nvl(login_date_last,date_add('2020-06-14',-1)),--若有历史登录记录，则根据历史记录获取末次登录日期，否则统一指定一个日期
    nvl(login_last_1d_count,0),
    nvl(login_last_1d_day_count,0),
    nvl(login_last_7d_count,0),
    nvl(login_last_7d_day_count,0),
    nvl(login_last_30d_count,0),
    nvl(login_last_30d_day_count,0),
    nvl(login_count,0),
    nvl(login_day_count,0),
    order_date_first,
    order_date_last,
    nvl(order_last_1d_count,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    payment_date_first,
    payment_date_last,
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(coupon_last_1d_get_count,0),
    nvl(coupon_last_1d_using_count,0),
    nvl(coupon_last_1d_used_count,0),
    nvl(coupon_last_7d_get_count,0),
    nvl(coupon_last_7d_using_count,0),
    nvl(coupon_last_7d_used_count,0),
    nvl(coupon_last_30d_get_count,0),
    nvl(coupon_last_30d_using_count,0),
    nvl(coupon_last_30d_used_count,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id,
        date_format(create_time,'yyyy-MM-dd') login_date_first
    from dim_user_info
    where dt='9999-99-99'
)t1
left join
(
    select
        user_id user_id,
        max(dt) login_date_last,
        sum(if(dt='2020-06-14',login_count,0)) login_last_1d_count,
        sum(if(dt='2020-06-14' and login_count>0,1,0)) login_last_1d_day_count,
        sum(if(dt>=date_add('2020-06-14',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6) and login_count>0,1,0)) login_last_7d_day_count,
        sum(if(dt>=date_add('2020-06-14',-29),login_count,0)) login_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29) and login_count>0,1,0)) login_last_30d_day_count,
        sum(login_count) login_count,
        sum(if(login_count>0,1,0)) login_day_count,
        min(if(order_count>0,dt,null)) order_date_first,
        max(if(order_count>0,dt,null)) order_date_last,
        sum(if(dt='2020-06-14',order_count,0)) order_last_1d_count,
        sum(if(dt='2020-06-14',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='2020-06-14',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='2020-06-14',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='2020-06-14',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='2020-06-14',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='2020-06-14',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_activity_count) order_activity_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_count) order_coupon_count,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        min(if(payment_count>0,dt,null)) payment_date_first,
        max(if(payment_count>0,dt,null)) payment_date_last,
        sum(if(dt='2020-06-14',payment_count,0)) payment_last_1d_count,
        sum(if(dt='2020-06-14',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='2020-06-14',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='2020-06-14',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='2020-06-14',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='2020-06-14',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='2020-06-14',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='2020-06-14',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='2020-06-14',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='2020-06-14',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='2020-06-14',coupon_get_count,0)) coupon_last_1d_get_count,
        sum(if(dt='2020-06-14',coupon_using_count,0)) coupon_last_1d_using_count,
        sum(if(dt='2020-06-14',coupon_used_count,0)) coupon_last_1d_used_count,
        sum(if(dt>=date_add('2020-06-14',-6),coupon_get_count,0)) coupon_last_7d_get_count,
        sum(if(dt>=date_add('2020-06-14',-6),coupon_using_count,0)) coupon_last_7d_using_count,
        sum(if(dt>=date_add('2020-06-14',-6),coupon_used_count,0)) coupon_last_7d_used_count,
        sum(if(dt>=date_add('2020-06-14',-29),coupon_get_count,0)) coupon_last_30d_get_count,
        sum(if(dt>=date_add('2020-06-14',-29),coupon_using_count,0)) coupon_last_30d_using_count,
        sum(if(dt>=date_add('2020-06-14',-29),coupon_used_count,0)) coupon_last_30d_used_count,
        sum(coupon_get_count) coupon_get_count,
        sum(coupon_using_count) coupon_using_count,
        sum(coupon_used_count) coupon_used_count,
        sum(if(dt='2020-06-14',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='2020-06-14',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='2020-06-14',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='2020-06-14',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from dws_user_action_daycount
    group by user_id
)t2
on t1.id=t2.user_id;

```

（2）每日装载

![image-20220928140157634](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220928140157634.png)

```
insert overwrite table dwt_user_topic partition(dt='2020-06-15')
select
    nvl(1d_ago.user_id,old.user_id),
    nvl(old.login_date_first,'2020-06-15'),
    if(1d_ago.user_id is not null,'2020-06-15',old.login_date_last),
    nvl(1d_ago.login_count,0),
    if(1d_ago.user_id is not null,1,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.login_last_7d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(7d_ago.user_id is null,0,1),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.login_last_30d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(30d_ago.user_id is null,0,1),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(old.login_day_count,0)+if(1d_ago.user_id is not null,1,0),
    if(old.order_date_first is null and 1d_ago.order_count>0, '2020-06-15', old.order_date_first),
    if(1d_ago.order_count>0,'2020-06-15',old.order_date_last),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    if(old.payment_date_first is null and 1d_ago.payment_count>0, '2020-06-15', old.payment_date_first),
    if(1d_ago.payment_count>0,'2020-06-15',old.payment_date_last),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)-nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)-nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)-nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_num,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)-nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)-nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.coupon_get_count,0),
    nvl(1d_ago.coupon_using_count,0),
    nvl(1d_ago.coupon_used_count,0),
    nvl(old.coupon_last_7d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(7d_ago.coupon_get_count,0),
    nvl(old.coupon_last_7d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(7d_ago.coupon_using_count,0),
    nvl(old.coupon_last_7d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(7d_ago.coupon_used_count,0),
    nvl(old.coupon_last_30d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(30d_ago.coupon_get_count,0),
    nvl(old.coupon_last_30d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(30d_ago.coupon_using_count,0),
    nvl(old.coupon_last_30d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(30d_ago.coupon_used_count,0),
    nvl(old.coupon_get_count,0)+nvl(1d_ago.coupon_get_count,0),
    nvl(old.coupon_using_count,0)+nvl(1d_ago.coupon_using_count,0),
    nvl(old.coupon_used_count,0)+nvl(1d_ago.coupon_used_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(1d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count, 0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        user_id,
        login_date_first,
        login_date_last,
        login_date_1d_count,
        login_last_1d_day_count,
        login_last_7d_count,
        login_last_7d_day_count,
        login_last_30d_count,
        login_last_30d_day_count,
        login_count,
        login_day_count,
        order_date_first,
        order_date_last,
        order_last_1d_count,
        order_activity_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_last_1d_count,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_activity_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_last_7d_count,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_activity_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_last_30d_count,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_date_first,
        payment_date_last,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        coupon_last_1d_get_count,
        coupon_last_1d_using_count,
        coupon_last_1d_used_count,
        coupon_last_7d_get_count,
        coupon_last_7d_using_count,
        coupon_last_7d_used_count,
        coupon_last_30d_get_count,
        coupon_last_30d_using_count,
        coupon_last_30d_used_count,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dwt_user_topic
    where dt=date_add('2020-06-15',-1)
)old
full outer join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_user_action_daycount
    where dt='2020-06-15'
)1d_ago
on old.user_id=1d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_user_action_daycount
    where dt=date_add('2020-06-15',-7)
)7d_ago
on old.user_id=7d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_user_action_daycount
    where dt=date_add('2020-06-15',-30)
)30d_ago
on old.user_id=30d_ago.user_id;

```

3）查询加载结果

## 15.3 商品主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_sku_topic;
CREATE EXTERNAL TABLE dwt_sku_topic
(
    `sku_id` STRING COMMENT 'sku_id',
    `order_last_1d_count` BIGINT COMMENT '最近1日被下单次数',
    `order_last_1d_num` BIGINT COMMENT '最近1日被下单件数',
    `order_activity_last_1d_count` BIGINT COMMENT '最近1日参与活动被下单次数',
    `order_coupon_last_1d_count` BIGINT COMMENT '最近1日使用优惠券被下单次数',
    `order_activity_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日优惠金额(活动)',
    `order_coupon_reduce_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日优惠金额(优惠券)',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日被下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日被下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日被下单次数',
    `order_last_7d_num` BIGINT COMMENT '最近7日被下单件数',
    `order_activity_last_7d_count` BIGINT COMMENT '最近7日参与活动被下单次数',
    `order_coupon_last_7d_count` BIGINT COMMENT '最近7日使用优惠券被下单次数',
    `order_activity_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日优惠金额(活动)',
    `order_coupon_reduce_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日优惠金额(优惠券)',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日被下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日被下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日被下单次数',
    `order_last_30d_num` BIGINT COMMENT '最近30日被下单件数',
    `order_activity_last_30d_count` BIGINT COMMENT '最近30日参与活动被下单次数',
    `order_coupon_last_30d_count` BIGINT COMMENT '最近30日使用优惠券被下单次数',
    `order_activity_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日优惠金额(活动)',
    `order_coupon_reduce_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日优惠金额(优惠券)',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日被下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日被下单最终金额',
    `order_count` BIGINT COMMENT '累积被下单次数',
    `order_num` BIGINT COMMENT '累积被下单件数',
    `order_activity_count` BIGINT COMMENT '累积参与活动被下单次数',
    `order_coupon_count` BIGINT COMMENT '累积使用优惠券被下单次数',
    `order_activity_reduce_amount` DECIMAL(16,2) COMMENT '累积优惠金额(活动)',
    `order_coupon_reduce_amount` DECIMAL(16,2) COMMENT '累积优惠金额(优惠券)',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积被下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积被下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日被支付次数',
    `payment_last_1d_num` BIGINT COMMENT '最近1日被支付件数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日被支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日被支付次数',
    `payment_last_7d_num` BIGINT COMMENT '最近7日被支付件数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日被支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日被支付次数',
    `payment_last_30d_num` BIGINT COMMENT '最近30日被支付件数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日被支付金额',
    `payment_count` BIGINT COMMENT '累积被支付次数',
    `payment_num` BIGINT COMMENT '累积被支付件数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积被支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1日退单次数',
    `refund_order_last_1d_num` BIGINT COMMENT '最近1日退单件数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7日退单次数',
    `refund_order_last_7d_num` BIGINT COMMENT '最近7日退单件数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30日退单次数',
    `refund_order_last_30d_num` BIGINT COMMENT '最近30日退单件数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_num` BIGINT COMMENT '累积退单件数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1日退款次数',
    `refund_payment_last_1d_num` BIGINT COMMENT '最近1日退款件数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7日退款次数',
    `refund_payment_last_7d_num` BIGINT COMMENT '最近7日退款件数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30日退款次数',
    `refund_payment_last_30d_num` BIGINT COMMENT '最近30日退款件数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_num` BIGINT COMMENT '累积退款件数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额',
    `cart_last_1d_count` BIGINT COMMENT '最近1日被加入购物车次数',
    `cart_last_7d_count` BIGINT COMMENT '最近7日被加入购物车次数',
    `cart_last_30d_count` BIGINT COMMENT '最近30日被加入购物车次数',
    `cart_count` BIGINT COMMENT '累积被加入购物车次数',
    `favor_last_1d_count` BIGINT COMMENT '最近1日被收藏次数',
    `favor_last_7d_count` BIGINT COMMENT '最近7日被收藏次数',
    `favor_last_30d_count` BIGINT COMMENT '最近30日被收藏次数',
    `favor_count` BIGINT COMMENT '累积被收藏次数',
    `appraise_last_1d_good_count` BIGINT COMMENT '最近1日好评数',
    `appraise_last_1d_mid_count` BIGINT COMMENT '最近1日中评数',
    `appraise_last_1d_bad_count` BIGINT COMMENT '最近1日差评数',
    `appraise_last_1d_default_count` BIGINT COMMENT '最近1日默认评价数',
    `appraise_last_7d_good_count` BIGINT COMMENT '最近7日好评数',
    `appraise_last_7d_mid_count` BIGINT COMMENT '最近7日中评数',
    `appraise_last_7d_bad_count` BIGINT COMMENT '最近7日差评数',
    `appraise_last_7d_default_count` BIGINT COMMENT '最近7日默认评价数',
    `appraise_last_30d_good_count` BIGINT COMMENT '最近30日好评数',
    `appraise_last_30d_mid_count` BIGINT COMMENT '最近30日中评数',
    `appraise_last_30d_bad_count` BIGINT COMMENT '最近30日差评数',
    `appraise_last_30d_default_count` BIGINT COMMENT '最近30日默认评价数',
    `appraise_good_count` BIGINT COMMENT '累积好评数',
    `appraise_mid_count` BIGINT COMMENT '累积中评数',
    `appraise_bad_count` BIGINT COMMENT '累积差评数',
    `appraise_default_count` BIGINT COMMENT '累积默认评价数'
 )COMMENT '商品主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_sku_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```
insert overwrite table dwt_sku_topic partition(dt='2020-06-14')
select
    id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_num,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_num,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_num,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_num,0),
    nvl(order_activity_count,0),
    nvl(order_coupon_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_num,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_num,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_num,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_num,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id
    from dim_sku_info
    where dt='2020-06-14'
)t1
left join
(
    select
        sku_id,
        sum(if(dt='2020-06-14',order_count,0)) order_last_1d_count,
        sum(if(dt='2020-06-14',order_num,0)) order_last_1d_num,
        sum(if(dt='2020-06-14',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='2020-06-14',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='2020-06-14',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='2020-06-14',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='2020-06-14',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='2020-06-14',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_num,0)) order_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_num,0)) order_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_num) order_num,
        sum(order_activity_count) order_activity_count,
        sum(order_coupon_count) order_coupon_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='2020-06-14',payment_count,0)) payment_last_1d_count,
        sum(if(dt='2020-06-14',payment_num,0)) payment_last_1d_num,
        sum(if(dt='2020-06-14',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),payment_num,0)) payment_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),payment_num,0)) payment_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_num) payment_num,
        sum(payment_amount) payment_amount,
        sum(if(dt='2020-06-14',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='2020-06-14',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='2020-06-14',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='2020-06-14',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='2020-06-14',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='2020-06-14',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='2020-06-14',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='2020-06-14',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='2020-06-14',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='2020-06-14',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='2020-06-14',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='2020-06-14',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('2020-06-14',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('2020-06-14',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from dws_sku_action_daycount
    group by sku_id
)t2
on t1.id=t2.sku_id;
```

（2）每日装载

```
insert overwrite table dwt_sku_topic partition(dt='2020-06-15')
select
    nvl(1d_ago.sku_id,old.sku_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_num,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_num,0)+nvl(1d_ago.order_num,0)- nvl(7d_ago.order_num,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_num,0)+nvl(1d_ago.order_num,0)- nvl(30d_ago.order_num,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_num,0)+nvl(1d_ago.order_num,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_num,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_num,0)+nvl(1d_ago.payment_num,0)- nvl(7d_ago.payment_num,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_num,0)+nvl(1d_ago.payment_num,0)- nvl(30d_ago.payment_num,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_num,0)+nvl(1d_ago.payment_num,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(old.refund_order_last_1d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_last_1d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_last_1d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)- nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)- nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(1d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count,0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        sku_id,
        order_last_1d_count,
        order_last_1d_num,
        order_activity_last_1d_count,
        order_coupon_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_num,
        order_activity_last_7d_count,
        order_coupon_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_num,
        order_activity_last_30d_count,
        order_coupon_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_num,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_num,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_num,
        payment_last_30d_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dwt_sku_topic
    where dt=date_add('2020-06-15',-1)
)old
full outer join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_sku_action_daycount
    where dt='2020-06-15'
)1d_ago
on old.sku_id=1d_ago.sku_id
left join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_sku_action_daycount
    where dt=date_add('2020-06-15',-7)
)7d_ago
on old.sku_id=7d_ago.sku_id
left join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from dws_sku_action_daycount
    where dt=date_add('2020-06-15',-30)
)30d_ago
on old.sku_id=30d_ago.sku_id;
```

3）查询加载结果

## 15.4 优惠券主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_coupon_topic;
CREATE EXTERNAL TABLE dwt_coupon_topic(
    `coupon_id` STRING COMMENT '优惠券ID',
    `get_last_1d_count` BIGINT COMMENT '最近1日领取次数',
    `get_last_7d_count` BIGINT COMMENT '最近7日领取次数',
    `get_last_30d_count` BIGINT COMMENT '最近30日领取次数',
    `get_count` BIGINT COMMENT '累积领取次数',
    `order_last_1d_count` BIGINT COMMENT '最近1日使用某券下单次数',
    `order_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单优惠金额',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日使用某券下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7日使用某券下单次数',
    `order_last_7d_reduce_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单优惠金额',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7日使用某券下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30日使用某券下单次数',
    `order_last_30d_reduce_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单优惠金额',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30日使用某券下单最终金额',
    `order_count` BIGINT COMMENT '累积使用(下单)次数',
    `order_reduce_amount` DECIMAL(16,2) COMMENT '使用某券累积下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '使用某券累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '使用某券累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日使用某券支付次数',
    `payment_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日使用某券优惠金额',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日使用某券支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7日使用某券支付次数',
    `payment_last_7d_reduce_amount` DECIMAL(16,2) COMMENT '最近7日使用某券优惠金额',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7日使用某券支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30日使用某券支付次数',
    `payment_last_30d_reduce_amount` DECIMAL(16,2) COMMENT '最近30日使用某券优惠金额',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30日使用某券支付金额',
    `payment_count` BIGINT COMMENT '累积使用(支付)次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '使用某券累积优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '使用某券累积支付金额',
    `expire_last_1d_count` BIGINT COMMENT '最近1日过期次数',
    `expire_last_7d_count` BIGINT COMMENT '最近7日过期次数',
    `expire_last_30d_count` BIGINT COMMENT '最近30日过期次数',
    `expire_count` BIGINT COMMENT '累积过期次数'
)comment '优惠券主题表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_coupon_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```
insert overwrite table dwt_coupon_topic partition(dt='2020-06-14')
select
    id,
    nvl(get_last_1d_count,0),
    nvl(get_last_7d_count,0),
    nvl(get_last_30d_count,0),
    nvl(get_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_reduce_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_reduce_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_reduce_amount,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_reduce_amount,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0),
    nvl(expire_last_1d_count,0),
    nvl(expire_last_7d_count,0),
    nvl(expire_last_30d_count,0),
    nvl(expire_count,0)
from
(
    select
        id
    from dim_coupon_info
    where dt='2020-06-14'
)t1
left join
(
    select
        coupon_id coupon_id,
        sum(if(dt='2020-06-14',get_count,0)) get_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),get_count,0)) get_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),get_count,0)) get_last_30d_count,
        sum(get_count) get_count,
        sum(if(dt='2020-06-14',order_count,0)) order_last_1d_count,
        sum(if(dt='2020-06-14',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='2020-06-14',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='2020-06-14',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_reduce_amount,0)) order_last_7d_reduce_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_reduce_amount,0)) order_last_30d_reduce_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='2020-06-14',payment_count,0)) payment_last_1d_count,
        sum(if(dt='2020-06-14',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='2020-06-14',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),payment_reduce_amount,0)) payment_last_7d_reduce_amount,
        sum(if(dt>=date_add('2020-06-14',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),payment_reduce_amount,0)) payment_last_30d_reduce_amount,
        sum(if(dt>=date_add('2020-06-14',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount,
        sum(if(dt='2020-06-14',expire_count,0)) expire_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),expire_count,0)) expire_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),expire_count,0)) expire_last_30d_count,
        sum(expire_count) expire_count
    from dws_coupon_info_daycount
    group by coupon_id
)t2
on t1.id=t2.coupon_id;
```

（2）每日装载

```
insert overwrite table dwt_coupon_topic partition(dt='2020-06-15')
select
    nvl(1d_ago.coupon_id,old.coupon_id),
    nvl(1d_ago.get_count,0),
    nvl(old.get_last_7d_count,0)+nvl(1d_ago.get_count,0)- nvl(7d_ago.get_count,0),
    nvl(old.get_last_30d_count,0)+nvl(1d_ago.get_count,0)- nvl(30d_ago.get_count,0),
    nvl(old.get_count,0)+nvl(1d_ago.get_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(7d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(30d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(old.payment_last_1d_count,0)+nvl(1d_ago.payment_count,0)- nvl(1d_ago.payment_count,0),
    nvl(old.payment_last_1d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_1d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(7d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(30d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.expire_count,0),
    nvl(old.expire_last_7d_count,0)+nvl(1d_ago.expire_count,0)- nvl(7d_ago.expire_count,0),
    nvl(old.expire_last_30d_count,0)+nvl(1d_ago.expire_count,0)- nvl(30d_ago.expire_count,0),
    nvl(old.expire_count,0)+nvl(1d_ago.expire_count,0)
from
(
    select
        coupon_id,
        get_last_1d_count,
        get_last_7d_count,
        get_last_30d_count,
        get_count,
        order_last_1d_count,
        order_last_1d_reduce_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_reduce_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_reduce_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_reduce_amount,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_reduce_amount,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_reduce_amount,
        payment_last_30d_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_last_1d_count,
        expire_last_7d_count,
        expire_last_30d_count,
        expire_count
    from dwt_coupon_topic
    where dt=date_add('2020-06-15',-1)
)old
full outer join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from dws_coupon_info_daycount
    where dt='2020-06-15'
)1d_ago
on old.coupon_id=1d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from dws_coupon_info_daycount
    where dt=date_add('2020-06-15',-7)
)7d_ago
on old.coupon_id=7d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from dws_coupon_info_daycount
    where dt=date_add('2020-06-15',-30)
)30d_ago
on old.coupon_id=30d_ago.coupon_id;
```

3）查询加载结果

## 15.5 活动主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_activity_topic;
CREATE EXTERNAL TABLE dwt_activity_topic(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING  COMMENT '活动ID',
    `order_last_1d_count` BIGINT COMMENT '最近1日参与某活动某规则下单次数',
    `order_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单优惠金额',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则下单最终金额',
    `order_count` BIGINT COMMENT '参与某活动某规则累积下单次数',
    `order_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单优惠金额',
    `order_original_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1日参与某活动某规则支付次数',
    `payment_last_1d_reduce_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则支付优惠金额',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1日参与某活动某规则支付金额',
    `payment_count` BIGINT COMMENT '参与某活动某规则累积支付次数',
    `payment_reduce_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积支付优惠金额',
    `payment_amount` DECIMAL(16,2) COMMENT '参与某活动某规则累积支付金额'
) COMMENT '活动主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_activity_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");

```

2）数据装载

（1）首日装载

```
insert overwrite table dwt_activity_topic partition(dt='2020-06-14')
select
    t1.activity_rule_id,
    t1.activity_id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0)
from
(
    select
        activity_rule_id,
        activity_id
    from dim_activity_rule_info
    where dt='2020-06-14'
)t1
left join
(
    select
        activity_rule_id,
        activity_id,

        sum(if(dt='2020-06-14',order_count,0)) order_last_1d_count,
        sum(if(dt='2020-06-14',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='2020-06-14',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='2020-06-14',order_final_amount,0)) order_last_1d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='2020-06-14',payment_count,0)) payment_last_1d_count,
        sum(if(dt='2020-06-14',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='2020-06-14',payment_amount,0)) payment_last_1d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount
    from dws_activity_info_daycount
    group by activity_rule_id,activity_id
)t2
on t1.activity_rule_id=t2.activity_rule_id
and t1.activity_id=t2.activity_id;
```

（2）每日装载

```
insert overwrite table dwt_activity_topic partition(dt='2020-
insert overwrite table dwt_activity_topic partition(dt='2020-06-15')
select
    nvl(1d_ago.activity_rule_id,old.activity_rule_id),
    nvl(1d_ago.activity_id,old.activity_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from dwt_activity_topic
    where dt=date_add('2020-06-15',-1)
)old
full outer join
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from dws_activity_info_daycount
    where dt='2020-06-15'
)1d_ago
on old.activity_rule_id=1d_ago.activity_rule_id;
```

3）查询加载结果

## 15.6 地区主题

1）建表语句

```
DROP TABLE IF EXISTS dwt_area_topic;
CREATE EXTERNAL TABLE dwt_area_topic(
    `province_id` STRING COMMENT '编号',
    `visit_last_1d_count` BIGINT COMMENT '最近1日访客访问次数',
    `login_last_1d_count` BIGINT COMMENT '最近1日用户访问次数',
    `visit_last_7d_count` BIGINT COMMENT '最近7访客访问次数',
    `login_last_7d_count` BIGINT COMMENT '最近7日用户访问次数',
    `visit_last_30d_count` BIGINT COMMENT '最近30日访客访问次数',
    `login_last_30d_count` BIGINT COMMENT '最近30日用户访问次数',
    `visit_count` BIGINT COMMENT '累积访客访问次数',
    `login_count` BIGINT COMMENT '累积用户访问次数',
    `order_last_1d_count` BIGINT COMMENT '最近1天下单次数',
    `order_last_1d_original_amount` DECIMAL(16,2) COMMENT '最近1天下单原始金额',
    `order_last_1d_final_amount` DECIMAL(16,2) COMMENT '最近1天下单最终金额',
    `order_last_7d_count` BIGINT COMMENT '最近7天下单次数',
    `order_last_7d_original_amount` DECIMAL(16,2) COMMENT '最近7天下单原始金额',
    `order_last_7d_final_amount` DECIMAL(16,2) COMMENT '最近7天下单最终金额',
    `order_last_30d_count` BIGINT COMMENT '最近30天下单次数',
    `order_last_30d_original_amount` DECIMAL(16,2) COMMENT '最近30天下单原始金额',
    `order_last_30d_final_amount` DECIMAL(16,2) COMMENT '最近30天下单最终金额',
    `order_count` BIGINT COMMENT '累积下单次数',
    `order_original_amount` DECIMAL(16,2) COMMENT '累积下单原始金额',
    `order_final_amount` DECIMAL(16,2) COMMENT '累积下单最终金额',
    `payment_last_1d_count` BIGINT COMMENT '最近1天支付次数',
    `payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天支付金额',
    `payment_last_7d_count` BIGINT COMMENT '最近7天支付次数',
    `payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天支付金额',
    `payment_last_30d_count` BIGINT COMMENT '最近30天支付次数',
    `payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天支付金额',
    `payment_count` BIGINT COMMENT '累积支付次数',
    `payment_amount` DECIMAL(16,2) COMMENT '累积支付金额',
    `refund_order_last_1d_count` BIGINT COMMENT '最近1天退单次数',
    `refund_order_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天退单金额',
    `refund_order_last_7d_count` BIGINT COMMENT '最近7天退单次数',
    `refund_order_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天退单金额',
    `refund_order_last_30d_count` BIGINT COMMENT '最近30天退单次数',
    `refund_order_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天退单金额',
    `refund_order_count` BIGINT COMMENT '累积退单次数',
    `refund_order_amount` DECIMAL(16,2) COMMENT '累积退单金额',
    `refund_payment_last_1d_count` BIGINT COMMENT '最近1天退款次数',
    `refund_payment_last_1d_amount` DECIMAL(16,2) COMMENT '最近1天退款金额',
    `refund_payment_last_7d_count` BIGINT COMMENT '最近7天退款次数',
    `refund_payment_last_7d_amount` DECIMAL(16,2) COMMENT '最近7天退款金额',
    `refund_payment_last_30d_count` BIGINT COMMENT '最近30天退款次数',
    `refund_payment_last_30d_amount` DECIMAL(16,2) COMMENT '最近30天退款金额',
    `refund_payment_count` BIGINT COMMENT '累积退款次数',
    `refund_payment_amount` DECIMAL(16,2) COMMENT '累积退款金额'
) COMMENT '地区主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_area_topic/'
TBLPROPERTIES ("parquet.compression"="lzo");
```

2）数据装载

（1）首日装载

```
insert overwrite table dwt_area_topic partition(dt='2020-06-14')
select
    id,
    nvl(visit_last_1d_count,0),
    nvl(login_last_1d_count,0),
    nvl(visit_last_7d_count,0),
    nvl(login_last_7d_count,0),
    nvl(visit_last_30d_count,0),
    nvl(login_last_30d_count,0),
    nvl(visit_count,0),
    nvl(login_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_amount,0)
from
(
    select
        id
    from dim_base_province
)t1
left join
(
    select
        province_id province_id,
        sum(if(dt='2020-06-14',visit_count,0)) visit_last_1d_count,
        sum(if(dt='2020-06-14',login_count,0)) login_last_1d_count,
        sum(if(dt>=date_add('2020-06-14',-6),visit_count,0)) visit_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-29),visit_count,0)) visit_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),login_count,0)) login_last_30d_count,
        sum(visit_count) visit_count,
        sum(login_count) login_count,
        sum(if(dt='2020-06-14',order_count,0)) order_last_1d_count,
        sum(if(dt='2020-06-14',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='2020-06-14',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('2020-06-14',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='2020-06-14',payment_count,0)) payment_last_1d_count,
        sum(if(dt='2020-06-14',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='2020-06-14',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='2020-06-14',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='2020-06-14',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='2020-06-14',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('2020-06-14',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('2020-06-14',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_amount) refund_payment_amount
    from dws_area_stats_daycount
    group by province_id
)t2
on t1.id=t2.province_id;
```

（2）每日装载

```
insert overwrite table dwt_area_topic partition(dt='2020-06-15')
select
    nvl(old.province_id, 1d_ago.province_id),
    nvl(1d_ago.visit_count,0),
    nvl(1d_ago.login_count,0),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)

from
(
    select
        province_id,
        visit_last_1d_count,
        login_last_1d_count,
        visit_last_7d_count,
        login_last_7d_count,
        visit_last_30d_count,
        login_last_30d_count,
        visit_count,
        login_count,
        order_last_1d_count,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_amount
    from dwt_area_topic
    where dt=date_add('2020-06-15',-1)
)old
full outer join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from dws_area_stats_daycount
    where dt='2020-06-15'
)1d_ago
on old.province_id=1d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from dws_area_stats_daycount
    where dt=date_add('2020-06-15',-7)
)7d_ago
on old.province_id= 7d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from dws_area_stats_daycount
    where dt=date_add('2020-06-15',-30)
)30d_ago
on old.province_id= 30d_ago.province_id;
```

3）查询加载结果

## 15.7 DWT层首日数据导入脚本

1）编写脚本

（1）在/home/atguigu/bin目录下创建脚本dws_to_dwt_init.sh

```
[atguigu@hadoop102 bin]$ vim dws_to_dwt_init.sh
```

在脚本中填写如下内容

```
#!/bin/bash

APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

dwt_visitor_topic="
insert overwrite table ${APP}.dwt_visitor_topic partition(dt='$do_date')
select
    nvl(1d_ago.mid_id,old.mid_id),
    nvl(1d_ago.brand,old.brand),
    nvl(1d_ago.model,old.model),
    nvl(1d_ago.channel,old.channel),
    nvl(1d_ago.os,old.os),
    nvl(1d_ago.area_code,old.area_code),
    nvl(1d_ago.version_code,old.version_code),
    case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-13'--无法获取准确的首次登录日期，给定一个数仓搭建日之前的日期
         else old.visit_date_first end,
    if(1d_ago.mid_id is not null,'$do_date',old.visit_date_last),
    nvl(1d_ago.visit_count,0),
    if(1d_ago.mid_id is null,0,1),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.visit_last_7d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(7d_ago.mid_id is null,0,1),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.visit_last_30d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(30d_ago.mid_id is null,0,1),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.visit_day_count,0)+if(1d_ago.mid_id is null,0,1)
from
(
    select
        mid_id,
        brand,
        model,
        channel,
        os,
        area_code,
        version_code,
        visit_date_first,
        visit_date_last,
        visit_last_1d_count,
        visit_last_1d_day_count,
        visit_last_7d_count,
        visit_last_7d_day_count,
        visit_last_30d_count,
        visit_last_30d_day_count,
        visit_count,
        visit_day_count
    from ${APP}.dwt_visitor_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt='$do_date'
)1d_ago
on old.mid_id=1d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.mid_id=7d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.mid_id=30d_ago.mid_id;
"

dwt_user_topic="
insert overwrite table ${APP}.dwt_user_topic partition(dt='$do_date')
select
    id,
    login_date_first,--以用户的创建日期作为首次登录日期
    nvl(login_date_last,date_add('$do_date',-1)),--若有历史登录记录，则根据历史记录获取末次登录日期，否则统一指定一个日期
    nvl(login_last_1d_count,0),
    nvl(login_last_1d_day_count,0),
    nvl(login_last_7d_count,0),
    nvl(login_last_7d_day_count,0),
    nvl(login_last_30d_count,0),
    nvl(login_last_30d_day_count,0),
    nvl(login_count,0),
    nvl(login_day_count,0),
    order_date_first,
    order_date_last,
    nvl(order_last_1d_count,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_activity_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_count,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    payment_date_first,
    payment_date_last,
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(coupon_last_1d_get_count,0),
    nvl(coupon_last_1d_using_count,0),
    nvl(coupon_last_1d_used_count,0),
    nvl(coupon_last_7d_get_count,0),
    nvl(coupon_last_7d_using_count,0),
    nvl(coupon_last_7d_used_count,0),
    nvl(coupon_last_30d_get_count,0),
    nvl(coupon_last_30d_using_count,0),
    nvl(coupon_last_30d_used_count,0),
    nvl(coupon_get_count,0),
    nvl(coupon_using_count,0),
    nvl(coupon_used_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id,
        date_format(create_time,'yyyy-MM-dd') login_date_first
    from ${APP}.dim_user_info
    where dt='9999-99-99'
)t1
left join
(
    select
        user_id user_id,
        max(dt) login_date_last,
        sum(if(dt='$do_date',login_count,0)) login_last_1d_count,
        sum(if(dt='$do_date' and login_count>0,1,0)) login_last_1d_day_count,
        sum(if(dt>=date_add('$do_date',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6) and login_count>0,1,0)) login_last_7d_day_count,
        sum(if(dt>=date_add('$do_date',-29),login_count,0)) login_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29) and login_count>0,1,0)) login_last_30d_day_count,
        sum(login_count) login_count,
        sum(if(login_count>0,1,0)) login_day_count,
        min(if(order_count>0,dt,null)) order_date_first,
        max(if(order_count>0,dt,null)) order_date_last,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='$do_date',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='$do_date',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_activity_count) order_activity_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_count) order_coupon_count,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        min(if(payment_count>0,dt,null)) payment_date_first,
        max(if(payment_count>0,dt,null)) payment_date_last,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='$do_date',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='$do_date',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='$do_date',coupon_get_count,0)) coupon_last_1d_get_count,
        sum(if(dt='$do_date',coupon_using_count,0)) coupon_last_1d_using_count,
        sum(if(dt='$do_date',coupon_used_count,0)) coupon_last_1d_used_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_get_count,0)) coupon_last_7d_get_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_using_count,0)) coupon_last_7d_using_count,
        sum(if(dt>=date_add('$do_date',-6),coupon_used_count,0)) coupon_last_7d_used_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_get_count,0)) coupon_last_30d_get_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_using_count,0)) coupon_last_30d_using_count,
        sum(if(dt>=date_add('$do_date',-29),coupon_used_count,0)) coupon_last_30d_used_count,
        sum(coupon_get_count) coupon_get_count,
        sum(coupon_using_count) coupon_using_count,
        sum(coupon_used_count) coupon_used_count,
        sum(if(dt='$do_date',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='$do_date',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='$do_date',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='$do_date',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from ${APP}.dws_user_action_daycount
    group by user_id
)t2
on t1.id=t2.user_id;
"

dwt_sku_topic="
insert overwrite table ${APP}.dwt_sku_topic partition(dt='$do_date')
select
    id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_num,0),
    nvl(order_activity_last_1d_count,0),
    nvl(order_coupon_last_1d_count,0),
    nvl(order_activity_reduce_last_1d_amount,0),
    nvl(order_coupon_reduce_last_1d_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_num,0),
    nvl(order_activity_last_7d_count,0),
    nvl(order_coupon_last_7d_count,0),
    nvl(order_activity_reduce_last_7d_amount,0),
    nvl(order_coupon_reduce_last_7d_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_num,0),
    nvl(order_activity_last_30d_count,0),
    nvl(order_coupon_last_30d_count,0),
    nvl(order_activity_reduce_last_30d_amount,0),
    nvl(order_coupon_reduce_last_30d_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_num,0),
    nvl(order_activity_count,0),
    nvl(order_coupon_count,0),
    nvl(order_activity_reduce_amount,0),
    nvl(order_coupon_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_num,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_num,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_num,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_num,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_num,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_num,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_num,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_num,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_num,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_num,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_num,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_num,0),
    nvl(refund_payment_amount,0),
    nvl(cart_last_1d_count,0),
    nvl(cart_last_7d_count,0),
    nvl(cart_last_30d_count,0),
    nvl(cart_count,0),
    nvl(favor_last_1d_count,0),
    nvl(favor_last_7d_count,0),
    nvl(favor_last_30d_count,0),
    nvl(favor_count,0),
    nvl(appraise_last_1d_good_count,0),
    nvl(appraise_last_1d_mid_count,0),
    nvl(appraise_last_1d_bad_count,0),
    nvl(appraise_last_1d_default_count,0),
    nvl(appraise_last_7d_good_count,0),
    nvl(appraise_last_7d_mid_count,0),
    nvl(appraise_last_7d_bad_count,0),
    nvl(appraise_last_7d_default_count,0),
    nvl(appraise_last_30d_good_count,0),
    nvl(appraise_last_30d_mid_count,0),
    nvl(appraise_last_30d_bad_count,0),
    nvl(appraise_last_30d_default_count,0),
    nvl(appraise_good_count,0),
    nvl(appraise_mid_count,0),
    nvl(appraise_bad_count,0),
    nvl(appraise_default_count,0)
from
(
    select
        id
    from ${APP}.dim_sku_info
    where dt='$do_date'
)t1
left join
(
    select
        sku_id,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_num,0)) order_last_1d_num,
        sum(if(dt='$do_date',order_activity_count,0)) order_activity_last_1d_count,
        sum(if(dt='$do_date',order_coupon_count,0)) order_coupon_last_1d_count,
        sum(if(dt='$do_date',order_activity_reduce_amount,0)) order_activity_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_coupon_reduce_amount,0)) order_coupon_reduce_last_1d_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_num,0)) order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),order_activity_count,0)) order_activity_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_count,0)) order_coupon_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_activity_reduce_amount,0)) order_activity_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_coupon_reduce_amount,0)) order_coupon_reduce_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_num,0)) order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),order_activity_count,0)) order_activity_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_count,0)) order_coupon_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_activity_reduce_amount,0)) order_activity_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_coupon_reduce_amount,0)) order_coupon_reduce_last_30d_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_num) order_num,
        sum(order_activity_count) order_activity_count,
        sum(order_coupon_count) order_coupon_count,
        sum(order_activity_reduce_amount) order_activity_reduce_amount,
        sum(order_coupon_reduce_amount) order_coupon_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_num,0)) payment_last_1d_num,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_num,0)) payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_num,0)) payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_num) payment_num,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_num,0)) refund_order_last_1d_num,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_num,0)) refund_order_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_num,0)) refund_order_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_num) refund_order_num,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_num,0)) refund_payment_last_1d_num,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_num,0)) refund_payment_last_7d_num,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_num,0)) refund_payment_last_30d_num,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_num) refund_payment_num,
        sum(refund_payment_amount) refund_payment_amount,
        sum(if(dt='$do_date',cart_count,0)) cart_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),cart_count,0)) cart_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),cart_count,0)) cart_last_30d_count,
        sum(cart_count) cart_count,
        sum(if(dt='$do_date',favor_count,0)) favor_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),favor_count,0)) favor_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),favor_count,0)) favor_last_30d_count,
        sum(favor_count) favor_count,
        sum(if(dt='$do_date',appraise_good_count,0)) appraise_last_1d_good_count,
        sum(if(dt='$do_date',appraise_mid_count,0)) appraise_last_1d_mid_count,
        sum(if(dt='$do_date',appraise_bad_count,0)) appraise_last_1d_bad_count,
        sum(if(dt='$do_date',appraise_default_count,0)) appraise_last_1d_default_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_good_count,0)) appraise_last_7d_good_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_mid_count,0)) appraise_last_7d_mid_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_bad_count,0)) appraise_last_7d_bad_count,
        sum(if(dt>=date_add('$do_date',-6),appraise_default_count,0)) appraise_last_7d_default_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_good_count,0)) appraise_last_30d_good_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_mid_count,0)) appraise_last_30d_mid_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_bad_count,0)) appraise_last_30d_bad_count,
        sum(if(dt>=date_add('$do_date',-29),appraise_default_count,0)) appraise_last_30d_default_count,
        sum(appraise_good_count) appraise_good_count,
        sum(appraise_mid_count) appraise_mid_count,
        sum(appraise_bad_count) appraise_bad_count,
        sum(appraise_default_count) appraise_default_count
    from ${APP}.dws_sku_action_daycount
    group by sku_id
)t2
on t1.id=t2.sku_id;
"

dwt_coupon_topic="
insert overwrite table ${APP}.dwt_coupon_topic partition(dt='$do_date')
select
    id,
    nvl(get_last_1d_count,0),
    nvl(get_last_7d_count,0),
    nvl(get_last_30d_count,0),
    nvl(get_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_reduce_amount,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_reduce_amount,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_reduce_amount,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_reduce_amount,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0),
    nvl(expire_last_1d_count,0),
    nvl(expire_last_7d_count,0),
    nvl(expire_last_30d_count,0),
    nvl(expire_count,0)
from
(
    select
        id
    from ${APP}.dim_coupon_info
    where dt='$do_date'
)t1
left join
(
    select
        coupon_id coupon_id,
        sum(if(dt='$do_date',get_count,0)) get_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),get_count,0)) get_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),get_count,0)) get_last_30d_count,
        sum(get_count) get_count,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_reduce_amount,0)) order_last_7d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_reduce_amount,0)) order_last_30d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_reduce_amount,0)) payment_last_7d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_reduce_amount,0)) payment_last_30d_reduce_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',expire_count,0)) expire_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),expire_count,0)) expire_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),expire_count,0)) expire_last_30d_count,
        sum(expire_count) expire_count
    from ${APP}.dws_coupon_info_daycount
    group by coupon_id
)t2
on t1.id=t2.coupon_id;
"

dwt_activity_topic="
insert overwrite table ${APP}.dwt_activity_topic partition(dt='$do_date')
select
    t1.activity_rule_id,
    t1.activity_id,
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_reduce_amount,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_count,0),
    nvl(order_reduce_amount,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_reduce_amount,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_count,0),
    nvl(payment_reduce_amount,0),
    nvl(payment_amount,0)
from
(
    select
        activity_rule_id,
        activity_id
    from ${APP}.dim_activity_rule_info
    where dt='$do_date'
)t1
left join
(
    select
        activity_rule_id,
        activity_id,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_reduce_amount,0)) order_last_1d_reduce_amount,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(order_count) order_count,
        sum(order_reduce_amount) order_reduce_amount,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_reduce_amount,0)) payment_last_1d_reduce_amount,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(payment_count) payment_count,
        sum(payment_reduce_amount) payment_reduce_amount,
        sum(payment_amount) payment_amount
    from ${APP}.dws_activity_info_daycount
    group by activity_rule_id,activity_id
)t2
on t1.activity_rule_id=t2.activity_rule_id
and t1.activity_id=t2.activity_id;
"

dwt_area_topic="
insert overwrite table ${APP}.dwt_area_topic partition(dt='$do_date')
select
    id,
    nvl(visit_last_1d_count,0),
    nvl(login_last_1d_count,0),
    nvl(visit_last_7d_count,0),
    nvl(login_last_7d_count,0),
    nvl(visit_last_30d_count,0),
    nvl(login_last_30d_count,0),
    nvl(visit_count,0),
    nvl(login_count,0),
    nvl(order_last_1d_count,0),
    nvl(order_last_1d_original_amount,0),
    nvl(order_last_1d_final_amount,0),
    nvl(order_last_7d_count,0),
    nvl(order_last_7d_original_amount,0),
    nvl(order_last_7d_final_amount,0),
    nvl(order_last_30d_count,0),
    nvl(order_last_30d_original_amount,0),
    nvl(order_last_30d_final_amount,0),
    nvl(order_count,0),
    nvl(order_original_amount,0),
    nvl(order_final_amount,0),
    nvl(payment_last_1d_count,0),
    nvl(payment_last_1d_amount,0),
    nvl(payment_last_7d_count,0),
    nvl(payment_last_7d_amount,0),
    nvl(payment_last_30d_count,0),
    nvl(payment_last_30d_amount,0),
    nvl(payment_count,0),
    nvl(payment_amount,0),
    nvl(refund_order_last_1d_count,0),
    nvl(refund_order_last_1d_amount,0),
    nvl(refund_order_last_7d_count,0),
    nvl(refund_order_last_7d_amount,0),
    nvl(refund_order_last_30d_count,0),
    nvl(refund_order_last_30d_amount,0),
    nvl(refund_order_count,0),
    nvl(refund_order_amount,0),
    nvl(refund_payment_last_1d_count,0),
    nvl(refund_payment_last_1d_amount,0),
    nvl(refund_payment_last_7d_count,0),
    nvl(refund_payment_last_7d_amount,0),
    nvl(refund_payment_last_30d_count,0),
    nvl(refund_payment_last_30d_amount,0),
    nvl(refund_payment_count,0),
    nvl(refund_payment_amount,0)
from
(
    select
        id
    from ${APP}.dim_base_province
)t1
left join
(
    select
        province_id province_id,
        sum(if(dt='$do_date',visit_count,0)) visit_last_1d_count,
        sum(if(dt='$do_date',login_count,0)) login_last_1d_count,
        sum(if(dt>=date_add('$do_date',-6),visit_count,0)) visit_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),login_count,0)) login_last_7d_count,
        sum(if(dt>=date_add('$do_date',-29),visit_count,0)) visit_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),login_count,0)) login_last_30d_count,
        sum(visit_count) visit_count,
        sum(login_count) login_count,
        sum(if(dt='$do_date',order_count,0)) order_last_1d_count,
        sum(if(dt='$do_date',order_original_amount,0)) order_last_1d_original_amount,
        sum(if(dt='$do_date',order_final_amount,0)) order_last_1d_final_amount,
        sum(if(dt>=date_add('$do_date',-6),order_count,0)) order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),order_original_amount,0)) order_last_7d_original_amount,
        sum(if(dt>=date_add('$do_date',-6),order_final_amount,0)) order_last_7d_final_amount,
        sum(if(dt>=date_add('$do_date',-29),order_count,0)) order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),order_original_amount,0)) order_last_30d_original_amount,
        sum(if(dt>=date_add('$do_date',-29),order_final_amount,0)) order_last_30d_final_amount,
        sum(order_count) order_count,
        sum(order_original_amount) order_original_amount,
        sum(order_final_amount) order_final_amount,
        sum(if(dt='$do_date',payment_count,0)) payment_last_1d_count,
        sum(if(dt='$do_date',payment_amount,0)) payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),payment_count,0)) payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),payment_amount,0)) payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),payment_count,0)) payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),payment_amount,0)) payment_last_30d_amount,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(dt='$do_date',refund_order_count,0)) refund_order_last_1d_count,
        sum(if(dt='$do_date',refund_order_amount,0)) refund_order_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_order_count,0)) refund_order_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_order_amount,0)) refund_order_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_order_count,0)) refund_order_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_order_amount,0)) refund_order_last_30d_amount,
        sum(refund_order_count) refund_order_count,
        sum(refund_order_amount) refund_order_amount,
        sum(if(dt='$do_date',refund_payment_count,0)) refund_payment_last_1d_count,
        sum(if(dt='$do_date',refund_payment_amount,0)) refund_payment_last_1d_amount,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_count,0)) refund_payment_last_7d_count,
        sum(if(dt>=date_add('$do_date',-6),refund_payment_amount,0)) refund_payment_last_7d_amount,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_count,0)) refund_payment_last_30d_count,
        sum(if(dt>=date_add('$do_date',-29),refund_payment_amount,0)) refund_payment_last_30d_amount,
        sum(refund_payment_count) refund_payment_count,
        sum(refund_payment_amount) refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    group by province_id
)t2
on t1.id=t2.province_id;
"


case $1 in
    "dwt_visitor_topic" )
        hive -e "$dwt_visitor_topic"
    ;;
    "dwt_user_topic" )
        hive -e "$dwt_user_topic"
    ;;
    "dwt_sku_topic" )
        hive -e "$dwt_sku_topic"
    ;;
    "dwt_activity_topic" )
        hive -e "$dwt_activity_topic"
    ;;
    "dwt_coupon_topic" )
        hive -e "$dwt_coupon_topic"
    ;;
    "dwt_area_topic" )
        hive -e "$dwt_area_topic"
    ;;
    "all" )
        hive -e "$dwt_visitor_topic$dwt_user_topic$dwt_sku_topic$dwt_activity_topic$dwt_coupon_topic$dwt_area_topic"
    ;;
esac
```

（2）增加执行权限

```
[atguigu@hadoop102 bin]$ chmod +x dws_to_dwt_init.sh
```

2）脚本使用
（1）执行脚本

```
[atguigu@hadoop102 bin]$ dws_to_dwt_init.sh all 2020-06-14
```

（2）查看数据是否导入成功

## 15.8 DWT层每日数据导入脚本

1）编写脚本
（1）在/home/atguigu/bin目录下创建脚本dws_to_dwt.sh

```
[atguigu@hadoop102 bin]$ vim dws_to_dwt.sh
```

在脚本中填写如下内容

```
#!/bin/bash

APP=gmall
# 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
if [ -n "$2" ] ;then
    do_date=$2
else 
    do_date=`date -d "-1 day" +%F`
fi

clear_date=`date -d "$do_date -2 day" +%F`

dwt_visitor_topic="
insert overwrite table ${APP}.dwt_visitor_topic partition(dt='$do_date')
select
    nvl(1d_ago.mid_id,old.mid_id),
    nvl(1d_ago.brand,old.brand),
    nvl(1d_ago.model,old.model),
    nvl(1d_ago.channel,old.channel),
    nvl(1d_ago.os,old.os),
    nvl(1d_ago.area_code,old.area_code),
    nvl(1d_ago.version_code,old.version_code),
    case when old.mid_id is null and 1d_ago.is_new=1 then '$do_date'
         when old.mid_id is null and 1d_ago.is_new=0 then '2020-06-13'--无法获取准确的首次登录日期，给定一个数仓搭建日之前的日期
         else old.visit_date_first end,
    if(1d_ago.mid_id is not null,'$do_date',old.visit_date_last),
    nvl(1d_ago.visit_count,0),
    if(1d_ago.mid_id is null,0,1),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.visit_last_7d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(7d_ago.mid_id is null,0,1),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.visit_last_30d_day_count,0)+if(1d_ago.mid_id is null,0,1)- if(30d_ago.mid_id is null,0,1),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.visit_day_count,0)+if(1d_ago.mid_id is null,0,1)
from
(
    select
        mid_id,
        brand,
        model,
        channel,
        os,
        area_code,
        version_code,
        visit_date_first,
        visit_date_last,
        visit_last_1d_count,
        visit_last_1d_day_count,
        visit_last_7d_count,
        visit_last_7d_day_count,
        visit_last_30d_count,
        visit_last_30d_day_count,
        visit_count,
        visit_day_count
    from ${APP}.dwt_visitor_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt='$do_date'
)1d_ago
on old.mid_id=1d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.mid_id=7d_ago.mid_id
left join
(
    select
        mid_id,
        brand,
        model,
        is_new,
        channel,
        os,
        area_code,
        version_code,
        visit_count
    from ${APP}.dws_visitor_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.mid_id=30d_ago.mid_id;
alter table ${APP}.dwt_visitor_topic drop partition(dt='$clear_date');
"

dwt_user_topic="
insert overwrite table ${APP}.dwt_user_topic partition(dt='$do_date')
select
    nvl(1d_ago.user_id,old.user_id),
    nvl(old.login_date_first,'$do_date'),
    if(1d_ago.user_id is not null,'$do_date',old.login_date_last),
    nvl(1d_ago.login_count,0),
    if(1d_ago.user_id is not null,1,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.login_last_7d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(7d_ago.user_id is null,0,1),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.login_last_30d_day_count,0)+if(1d_ago.user_id is null,0,1)- if(30d_ago.user_id is null,0,1),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(old.login_day_count,0)+if(1d_ago.user_id is not null,1,0),
    if(old.order_date_first is null and 1d_ago.order_count>0, '$do_date', old.order_date_first),
    if(1d_ago.order_count>0,'$do_date',old.order_date_last),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    if(old.payment_date_first is null and 1d_ago.payment_count>0, '$do_date', old.payment_date_first),
    if(1d_ago.payment_count>0,'$do_date',old.payment_date_last),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)-nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)-nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)-nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_num,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num, 0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+ nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)-nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+ nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)-nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)-nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.coupon_get_count,0),
    nvl(1d_ago.coupon_using_count,0),
    nvl(1d_ago.coupon_used_count,0),
    nvl(old.coupon_last_7d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(7d_ago.coupon_get_count,0),
    nvl(old.coupon_last_7d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(7d_ago.coupon_using_count,0),
    nvl(old.coupon_last_7d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(7d_ago.coupon_used_count,0),
    nvl(old.coupon_last_30d_get_count,0)+nvl(1d_ago.coupon_get_count,0)- nvl(30d_ago.coupon_get_count,0),
    nvl(old.coupon_last_30d_using_count,0)+nvl(1d_ago.coupon_using_count,0)- nvl(30d_ago.coupon_using_count,0),
    nvl(old.coupon_last_30d_used_count,0)+ nvl(1d_ago.coupon_used_count,0)- nvl(30d_ago.coupon_used_count,0),
    nvl(old.coupon_get_count,0)+nvl(1d_ago.coupon_get_count,0),
    nvl(old.coupon_using_count,0)+nvl(1d_ago.coupon_using_count,0),
    nvl(old.coupon_used_count,0)+nvl(1d_ago.coupon_used_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)-nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)-nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)-nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count, 0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        user_id,
        login_date_first,
        login_date_last,
        login_date_1d_count,
        login_last_1d_day_count,
        login_last_7d_count,
        login_last_7d_day_count,
        login_last_30d_count,
        login_last_30d_day_count,
        login_count,
        login_day_count,
        order_date_first,
        order_date_last,
        order_last_1d_count,
        order_activity_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_last_1d_count,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_activity_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_last_7d_count,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_activity_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_last_30d_count,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_date_first,
        payment_date_last,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        coupon_last_1d_get_count,
        coupon_last_1d_using_count,
        coupon_last_1d_used_count,
        coupon_last_7d_get_count,
        coupon_last_7d_using_count,
        coupon_last_7d_used_count,
        coupon_last_30d_get_count,
        coupon_last_30d_using_count,
        coupon_last_30d_used_count,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dwt_user_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt='$do_date'
)1d_ago
on old.user_id=1d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.user_id=7d_ago.user_id
left join
(
    select
        user_id,
        login_count,
        cart_count,
        favor_count,
        order_count,
        order_activity_count,
        order_activity_reduce_amount,
        order_coupon_count,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        coupon_get_count,
        coupon_using_count,
        coupon_used_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_user_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.user_id=30d_ago.user_id;
alter table ${APP}.dwt_user_topic drop partition(dt='$clear_date');
"

dwt_sku_topic="
insert overwrite table ${APP}.dwt_sku_topic partition(dt='$do_date')
select
    nvl(1d_ago.sku_id,old.sku_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_num,0),
    nvl(1d_ago.order_activity_count,0),
    nvl(1d_ago.order_coupon_count,0),
    nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_num,0)+nvl(1d_ago.order_num,0)- nvl(7d_ago.order_num,0),
    nvl(old.order_activity_last_7d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(7d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_7d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(7d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(7d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_7d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(7d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_num,0)+nvl(1d_ago.order_num,0)- nvl(30d_ago.order_num,0),
    nvl(old.order_activity_last_30d_count,0)+nvl(1d_ago.order_activity_count,0)- nvl(30d_ago.order_activity_count,0),
    nvl(old.order_coupon_last_30d_count,0)+nvl(1d_ago.order_coupon_count,0)- nvl(30d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0)- nvl(30d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_last_30d_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0)- nvl(30d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_num,0)+nvl(1d_ago.order_num,0),
    nvl(old.order_activity_count,0)+nvl(1d_ago.order_activity_count,0),
    nvl(old.order_coupon_count,0)+nvl(1d_ago.order_coupon_count,0),
    nvl(old.order_activity_reduce_amount,0.0)+nvl(1d_ago.order_activity_reduce_amount,0.0),
    nvl(old.order_coupon_reduce_amount,0.0)+nvl(1d_ago.order_coupon_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_num,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_num,0)+nvl(1d_ago.payment_num,0)- nvl(7d_ago.payment_num,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_num,0)+nvl(1d_ago.payment_num,0)- nvl(30d_ago.payment_num,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_num,0)+nvl(1d_ago.payment_num,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(old.refund_order_last_1d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_last_1d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_last_1d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(7d_ago.refund_order_num,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_num,0)+nvl(1d_ago.refund_order_num,0)- nvl(30d_ago.refund_order_num,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_num,0)+nvl(1d_ago.refund_order_num,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_num,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(7d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_num,0)+nvl(1d_ago.refund_payment_num,0)- nvl(30d_ago.refund_payment_num,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_num,0)+nvl(1d_ago.refund_payment_num,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0),
    nvl(1d_ago.cart_count,0),
    nvl(old.cart_last_7d_count,0)+nvl(1d_ago.cart_count,0)- nvl(7d_ago.cart_count,0),
    nvl(old.cart_last_30d_count,0)+nvl(1d_ago.cart_count,0)- nvl(30d_ago.cart_count,0),
    nvl(old.cart_count,0)+nvl(1d_ago.cart_count,0),
    nvl(1d_ago.favor_count,0),
    nvl(old.favor_last_7d_count,0)+nvl(1d_ago.favor_count,0)- nvl(7d_ago.favor_count,0),
    nvl(old.favor_last_30d_count,0)+nvl(1d_ago.favor_count,0)- nvl(30d_ago.favor_count,0),
    nvl(old.favor_count,0)+nvl(1d_ago.favor_count,0),
    nvl(1d_ago.appraise_good_count,0),
    nvl(1d_ago.appraise_mid_count,0),
    nvl(1d_ago.appraise_bad_count,0),
    nvl(1d_ago.appraise_default_count,0),
    nvl(old.appraise_last_7d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(7d_ago.appraise_good_count,0),
    nvl(old.appraise_last_7d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(7d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_7d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(7d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_7d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(7d_ago.appraise_default_count,0),
    nvl(old.appraise_last_30d_good_count,0)+nvl(1d_ago.appraise_good_count,0)- nvl(30d_ago.appraise_good_count,0),
    nvl(old.appraise_last_30d_mid_count,0)+nvl(1d_ago.appraise_mid_count,0)- nvl(30d_ago.appraise_mid_count,0),
    nvl(old.appraise_last_30d_bad_count,0)+nvl(1d_ago.appraise_bad_count,0)- nvl(30d_ago.appraise_bad_count,0),
    nvl(old.appraise_last_30d_default_count,0)+nvl(1d_ago.appraise_default_count,0)- nvl(30d_ago.appraise_default_count,0),
    nvl(old.appraise_good_count,0)+nvl(1d_ago.appraise_good_count,0),
    nvl(old.appraise_mid_count,0)+nvl(1d_ago.appraise_mid_count,0),
    nvl(old.appraise_bad_count,0)+nvl(1d_ago.appraise_bad_count,0),
    nvl(old.appraise_default_count,0)+nvl(1d_ago.appraise_default_count,0)
from
(
    select
        sku_id,
        order_last_1d_count,
        order_last_1d_num,
        order_activity_last_1d_count,
        order_coupon_last_1d_count,
        order_activity_reduce_last_1d_amount,
        order_coupon_reduce_last_1d_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_num,
        order_activity_last_7d_count,
        order_coupon_last_7d_count,
        order_activity_reduce_last_7d_amount,
        order_coupon_reduce_last_7d_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_num,
        order_activity_last_30d_count,
        order_coupon_last_30d_count,
        order_activity_reduce_last_30d_amount,
        order_coupon_reduce_last_30d_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_num,
        order_activity_count,
        order_coupon_count,
        order_activity_reduce_amount,
        order_coupon_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_num,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_num,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_num,
        payment_last_30d_amount,
        payment_count,
        payment_num,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_num,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_num,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_num,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_num,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_num,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_num,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_last_1d_count,
        cart_last_7d_count,
        cart_last_30d_count,
        cart_count,
        favor_last_1d_count,
        favor_last_7d_count,
        favor_last_30d_count,
        favor_count,
        appraise_last_1d_good_count,
        appraise_last_1d_mid_count,
        appraise_last_1d_bad_count,
        appraise_last_1d_default_count,
        appraise_last_7d_good_count,
        appraise_last_7d_mid_count,
        appraise_last_7d_bad_count,
        appraise_last_7d_default_count,
        appraise_last_30d_good_count,
        appraise_last_30d_mid_count,
        appraise_last_30d_bad_count,
        appraise_last_30d_default_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dwt_sku_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt='$do_date'
)1d_ago
on old.sku_id=1d_ago.sku_id
left join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.sku_id=7d_ago.sku_id
left join
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
        payment_count,
        payment_num,
        payment_amount,
        refund_order_count,
        refund_order_num,
        refund_order_amount,
        refund_payment_count,
        refund_payment_num,
        refund_payment_amount,
        cart_count,
        favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from ${APP}.dws_sku_action_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.sku_id=30d_ago.sku_id;
alter table ${APP}.dwt_sku_topic drop partition(dt='$clear_date');
"

dwt_activity_topic="
insert overwrite table ${APP}.dwt_activity_topic partition(dt='$do_date')
select
    nvl(1d_ago.activity_rule_id,old.activity_rule_id),
    nvl(1d_ago.activity_id,old.activity_id),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0)
from
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from ${APP}.dwt_activity_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        activity_rule_id,
        activity_id,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount
    from ${APP}.dws_activity_info_daycount
    where dt='$do_date'
)1d_ago
on old.activity_rule_id=1d_ago.activity_rule_id;
alter table ${APP}.dwt_activity_topic drop partition(dt='$clear_date');
"

dwt_coupon_topic="
insert overwrite table ${APP}.dwt_coupon_topic partition(dt='$do_date')
select
    nvl(1d_ago.coupon_id,old.coupon_id),
    nvl(1d_ago.get_count,0),
    nvl(old.get_last_7d_count,0)+nvl(1d_ago.get_count,0)- nvl(7d_ago.get_count,0),
    nvl(old.get_last_30d_count,0)+nvl(1d_ago.get_count,0)- nvl(30d_ago.get_count,0),
    nvl(old.get_count,0)+nvl(1d_ago.get_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_reduce_amount,0.0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(7d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0)- nvl(30d_ago.order_reduce_amount,0.0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_reduce_amount,0.0)+nvl(1d_ago.order_reduce_amount,0.0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(old.payment_last_1d_count,0)+nvl(1d_ago.payment_count,0)- nvl(1d_ago.payment_count,0),
    nvl(old.payment_last_1d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_1d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(7d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0)- nvl(30d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_reduce_amount,0.0)+nvl(1d_ago.payment_reduce_amount,0.0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.expire_count,0),
    nvl(old.expire_last_7d_count,0)+nvl(1d_ago.expire_count,0)- nvl(7d_ago.expire_count,0),
    nvl(old.expire_last_30d_count,0)+nvl(1d_ago.expire_count,0)- nvl(30d_ago.expire_count,0),
    nvl(old.expire_count,0)+nvl(1d_ago.expire_count,0)
from
(
    select
        coupon_id,
        get_last_1d_count,
        get_last_7d_count,
        get_last_30d_count,
        get_count,
        order_last_1d_count,
        order_last_1d_reduce_amount,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_reduce_amount,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_reduce_amount,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_reduce_amount,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_reduce_amount,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_reduce_amount,
        payment_last_30d_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_last_1d_count,
        expire_last_7d_count,
        expire_last_30d_count,
        expire_count
    from ${APP}.dwt_coupon_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt='$do_date'
)1d_ago
on old.coupon_id=1d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.coupon_id=7d_ago.coupon_id
left join
(
    select
        coupon_id,
        get_count,
        order_count,
        order_reduce_amount,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_reduce_amount,
        payment_amount,
        expire_count
    from ${APP}.dws_coupon_info_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.coupon_id=30d_ago.coupon_id;
alter table ${APP}.dwt_coupon_topic drop partition(dt='$clear_date');
"

dwt_area_topic="
insert overwrite table ${APP}.dwt_area_topic partition(dt='$do_date')
select
    nvl(old.province_id, 1d_ago.province_id),
    nvl(1d_ago.visit_count,0),
    nvl(1d_ago.login_count,0),
    nvl(old.visit_last_7d_count,0)+nvl(1d_ago.visit_count,0)- nvl(7d_ago.visit_count,0),
    nvl(old.login_last_7d_count,0)+nvl(1d_ago.login_count,0)- nvl(7d_ago.login_count,0),
    nvl(old.visit_last_30d_count,0)+nvl(1d_ago.visit_count,0)- nvl(30d_ago.visit_count,0),
    nvl(old.login_last_30d_count,0)+nvl(1d_ago.login_count,0)- nvl(30d_ago.login_count,0),
    nvl(old.visit_count,0)+nvl(1d_ago.visit_count,0),
    nvl(old.login_count,0)+nvl(1d_ago.login_count,0),
    nvl(1d_ago.order_count,0),
    nvl(1d_ago.order_original_amount,0.0),
    nvl(1d_ago.order_final_amount,0.0),
    nvl(old.order_last_7d_count,0)+nvl(1d_ago.order_count,0)- nvl(7d_ago.order_count,0),
    nvl(old.order_last_7d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(7d_ago.order_original_amount,0.0),
    nvl(old.order_last_7d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(7d_ago.order_final_amount,0.0),
    nvl(old.order_last_30d_count,0)+nvl(1d_ago.order_count,0)- nvl(30d_ago.order_count,0),
    nvl(old.order_last_30d_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0)- nvl(30d_ago.order_original_amount,0.0),
    nvl(old.order_last_30d_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0)- nvl(30d_ago.order_final_amount,0.0),
    nvl(old.order_count,0)+nvl(1d_ago.order_count,0),
    nvl(old.order_original_amount,0.0)+nvl(1d_ago.order_original_amount,0.0),
    nvl(old.order_final_amount,0.0)+nvl(1d_ago.order_final_amount,0.0),
    nvl(1d_ago.payment_count,0),
    nvl(1d_ago.payment_amount,0.0),
    nvl(old.payment_last_7d_count,0)+nvl(1d_ago.payment_count,0)- nvl(7d_ago.payment_count,0),
    nvl(old.payment_last_7d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(7d_ago.payment_amount,0.0),
    nvl(old.payment_last_30d_count,0)+nvl(1d_ago.payment_count,0)- nvl(30d_ago.payment_count,0),
    nvl(old.payment_last_30d_amount,0.0)+nvl(1d_ago.payment_amount,0.0)- nvl(30d_ago.payment_amount,0.0),
    nvl(old.payment_count,0)+nvl(1d_ago.payment_count,0),
    nvl(old.payment_amount,0.0)+nvl(1d_ago.payment_amount,0.0),
    nvl(1d_ago.refund_order_count,0),
    nvl(1d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_7d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(7d_ago.refund_order_count,0),
    nvl(old.refund_order_last_7d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(7d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_last_30d_count,0)+nvl(1d_ago.refund_order_count,0)- nvl(30d_ago.refund_order_count,0),
    nvl(old.refund_order_last_30d_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0)- nvl(30d_ago.refund_order_amount,0.0),
    nvl(old.refund_order_count,0)+nvl(1d_ago.refund_order_count,0),
    nvl(old.refund_order_amount,0.0)+nvl(1d_ago.refund_order_amount,0.0),
    nvl(1d_ago.refund_payment_count,0),
    nvl(1d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_7d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(7d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_7d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(7d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_last_30d_count,0)+nvl(1d_ago.refund_payment_count,0)- nvl(30d_ago.refund_payment_count,0),
    nvl(old.refund_payment_last_30d_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)- nvl(30d_ago.refund_payment_amount,0.0),
    nvl(old.refund_payment_count,0)+nvl(1d_ago.refund_payment_count,0),
    nvl(old.refund_payment_amount,0.0)+nvl(1d_ago.refund_payment_amount,0.0)

from
(
    select
        province_id,
        visit_last_1d_count,
        login_last_1d_count,
        visit_last_7d_count,
        login_last_7d_count,
        visit_last_30d_count,
        login_last_30d_count,
        visit_count,
        login_count,
        order_last_1d_count,
        order_last_1d_original_amount,
        order_last_1d_final_amount,
        order_last_7d_count,
        order_last_7d_original_amount,
        order_last_7d_final_amount,
        order_last_30d_count,
        order_last_30d_original_amount,
        order_last_30d_final_amount,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_last_1d_count,
        payment_last_1d_amount,
        payment_last_7d_count,
        payment_last_7d_amount,
        payment_last_30d_count,
        payment_last_30d_amount,
        payment_count,
        payment_amount,
        refund_order_last_1d_count,
        refund_order_last_1d_amount,
        refund_order_last_7d_count,
        refund_order_last_7d_amount,
        refund_order_last_30d_count,
        refund_order_last_30d_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_last_1d_count,
        refund_payment_last_1d_amount,
        refund_payment_last_7d_count,
        refund_payment_last_7d_amount,
        refund_payment_last_30d_count,
        refund_payment_last_30d_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dwt_area_topic
    where dt=date_add('$do_date',-1)
)old
full outer join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt='$do_date'
)1d_ago
on old.province_id=1d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt=date_add('$do_date',-7)
)7d_ago
on old.province_id= 7d_ago.province_id
left join
(
    select
        province_id,
        visit_count,
        login_count,
        order_count,
        order_original_amount,
        order_final_amount,
        payment_count,
        payment_amount,
        refund_order_count,
        refund_order_amount,
        refund_payment_count,
        refund_payment_amount
    from ${APP}.dws_area_stats_daycount
    where dt=date_add('$do_date',-30)
)30d_ago
on old.province_id= 30d_ago.province_id;
alter table ${APP}.dwt_area_topic drop partition(dt='$clear_date');
"


case $1 in
    "dwt_visitor_topic" )
        hive -e "$dwt_visitor_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_visitor_topic/dt=$clear_date
    ;;
    "dwt_user_topic" )
        hive -e "$dwt_user_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_user_topic/dt=$clear_date
    ;;
    "dwt_sku_topic" )
        hive -e "$dwt_sku_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_sku_topic/dt=$clear_date
    ;;
    "dwt_activity_topic" )
        hive -e "$dwt_activity_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_activity_topic/dt=$clear_date
    ;;
    "dwt_coupon_topic" )
        hive -e "$dwt_coupon_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_coupon_topic/dt=$clear_date
    ;;
    "dwt_area_topic" )
        hive -e "$dwt_area_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_area_topic/dt=$clear_date
    ;;
    "all" )
        hive -e "$dwt_visitor_topic$dwt_user_topic$dwt_sku_topic$dwt_activity_topic$dwt_coupon_topic$dwt_area_topic"
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_visitor_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_user_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_sku_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_activity_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_coupon_topic/dt=$clear_date
        hadoop fs -rm -r -f /warehouse/gmall/dwt/dwt_area_topic/dt=$clear_date
    ;;
esac
```

（2）增加脚本执行权限

```
[atguigu@hadoop102 bin]$ chmod 777 dws_to_dwt.sh
```

2）脚本使用

（1）执行脚本

```
[atguigu@hadoop102 bin]$ dws_to_dwt.sh 2020-06-14
```

（2）查看导入数据
