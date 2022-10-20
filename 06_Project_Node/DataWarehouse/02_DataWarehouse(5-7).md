[TOC]

# 第二部分：业务数据的采集（P60-P70）

# 5.0电商业务简介

## 5.1 电商业务流程

​		电商的业务流程可以以一个普通用户的浏览足迹为例进行说明，用户点开电商首页开始浏览，可能会通过分类查询也可能通过全文搜索寻找自己中意的商品，这些商品无疑都是存储在后台的管理系统中的。

​		当用户寻找到自己中意的商品，可能会想要购买，将商品添加到购物车后发现需要登录，登录后对商品进行结算，这时候购物车的管理和商品订单信息的生成都会对业务数据库产生影响，会生成相应的订单数据和支付数据。

​		订单正式生成之后，还会对订单进行跟踪处理，直到订单全部完成。

​		电商的主要业务流程包括用户前台浏览商品时的商品详情的管理，用户商品加入购物车进行支付时用户个人中心&支付服务的管理，用户支付完成后订单后台服务的管理，这些流程涉及到了十几个甚至几十个业务数据表，甚至更多。

![image-20220903200655874](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903200655874.png)

## 5.2 电商常识

### 5.2.1 SKU和SPU

​		 **SKU** = Stock Keeping Unit（库存量基本单位）。现在已经被引申为产品统一编号的简称，每种产品均对应有唯一的SKU号。

​		 **SPU**（Standard Product Unit）：是商品信息聚合的最小单位，是一组可复用、易检索的标准化信息集合。

==例如：iPhoneX手机就是SPU。一台银色、128G内存的、支持联通网络的iPhoneX，就是SKU。==

==同一SPU的商品可以共用商品图片、海报、销售属性等。==

### 5.2.2 平台属性和销售属性

**1）平台属性**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps1.jpg" alt="img" style="zoom:150%;" /> 

**2）销售属性**

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps2.jpg" alt="img" style="zoom:150%;" />

## 5.3 电商系统表结构

​		以下为本电商数仓系统涉及到的业务数据表结构关系。**这34个表以订单表、用户表、SKU商品表、活动表和优惠券表为中心**，延伸出了优惠券领用表、支付流水表、活动订单表、订单详情表、订单状态表、商品评论表、编码字典表退单表、SPU商品表等，用户表提供用户的详细信息，支付流水表提供该订单的支付详情，订单详情表提供订单的商品数量等情况，商品表给订单详情表提供商品的详细信息。本次讲解以此34个表为例，实际项目中，业务数据库中表格远远不止这些。

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps3.png" alt="img" style="zoom:150%;" />

### 5.3.1 活动信息表（activity_info）

<img src="https://pic-1313413291.cos.ap-nanjing.myqcloud.com/wps4.png" alt="img" style="zoom:150%;" />

| 字段名        |           字段说明           |
| ------------- | :--------------------------: |
| id            |            活动id            |
| activity_name |           活动名称           |
| activity_type | 活动类型（1：满减，2：折扣） |
| activity_desc |           活动描述           |
| start_time    |           开始时间           |
| end_time      |           结束时间           |
| create_time   |           创建时间           |

### 5.3.2 活动规则表（activity_rule）

| id               | 编号     |
| ---------------- | -------- |
| activity_id      | 活动ID   |
| activity_type    | 活动类型 |
| condition_amount | 满减金额 |
| condition_num    | 满减件数 |
| benefit_amount   | 优惠金额 |
| benefit_discount | 优惠折扣 |
| benefit_level    | 优惠级别 |

### 5.3.3 活动商品关联表（activity_sku）

| 字段名      | 字段说明 |
| ----------- | -------- |
| id          | 编号     |
| activity_id | 活动id   |
| sku_id      | sku_id   |
| create_time | 创建时间 |

==哪些商品能够参与这个活动==

### 5.3.4 平台属性表（base_attr_info）

| 字段名         | 字段说明 |
| -------------- | -------- |
| id             | 编号     |
| attr_name      | 属性名称 |
| category_id    | 分类id   |
| category_level | 分类层级 |



### 5.3.5 平台属性值表（base_attr_value）

| 字段名     | 字段说明   |
| ---------- | ---------- |
| id         | 编号       |
| value_name | 属性值名称 |
| attr_id    | 属性id     |



### 5.3.6 一级分类表（base_category1）

| 字段名 | 字段说明 |
| ------ | -------- |
| id     | 编号     |
| name   | 分类名称 |

### 5.3.7 二级分类表（base_category2）

| 字段名       | 字段说明     |
| ------------ | ------------ |
| id           | 编号         |
| name         | 二级分类名称 |
| category1_id | 一级分类编号 |

### 5.3.8 三级分类表（base_category3）

| 字段名       | 字段说明     |
| ------------ | ------------ |
| id           | 编号         |
| name         | 三级分类名称 |
| category2_id | 二级分类编号 |

### 5.3.9 字典表（base_dic）

| 字段名       | 字段说明 |
| ------------ | -------- |
| dic_code     | 编号     |
| dic_name     | 编码名称 |
| parent_code  | 父编号   |
| create_time  | 创建日期 |
| operate_time | 修改日期 |



### 5.3.10 省份表（base_province）

| 字段名     | 字段说明    |
| ---------- | ----------- |
| id         | id          |
| name       | 省名称      |
| region_id  | 大区id      |
| area_code  | 行政区位码  |
| iso_code   | 国际编码    |
| iso_3166_2 | ISO3166编码 |

### 5.3.11 地区表（base_region）

| 字段名      | 字段说明 |
| ----------- | -------- |
| id          | 大区id   |
| region_name | 大区名称 |

### 5.3.12 品牌表（base_trademark）

| 字段名   | 字段说明           |
| -------- | ------------------ |
| id       | 编号               |
| tm_name  | 属性值             |
| logo_url | 品牌logo的图片路径 |

### 5.3.13 购物车表（cart_info）

| 字段名       | 字段说明         |
| ------------ | ---------------- |
| id           | 编号             |
| user_id      | 用户id           |
| sku_id       | skuid            |
| cart_price   | 放入购物车时价格 |
| sku_num      | 数量             |
| img_url      | 图片文件         |
| sku_name     | sku名称 (冗余)   |
| is_checked   | 是否已经下单     |
| create_time  | 创建时间         |
| operate_time | 修改时间         |
| is_ordered   | 是否已经下单     |
| order_time   | 下单时间         |
| source_type  | 来源类型         |
| source_id    | 来源编号         |

### 5.3.14 评价表（comment_info）

| 字段名       | 字段说明                  |
| ------------ | ------------------------- |
| id           | 编号                      |
| user_id      | 用户id                    |
| nick_name    | 用户昵称                  |
| head_img     | 图片                      |
| sku_id       | 商品sku_id                |
| spu_id       | 商品spu_id                |
| order_id     | 订单编号                  |
| appraise     | 评价 1 好评 2 中评 3 差评 |
| comment_txt  | 评价内容                  |
| create_time  | 创建时间                  |
| operate_time | 修改时间                  |

### 5.3.15 优惠券信息表（coupon_info）

| 字段名           | 字段说明                                            |
| ---------------- | --------------------------------------------------- |
| id               | 购物券编号                                          |
| coupon_name      | 购物券名称                                          |
| coupon_type      | 购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券  |
| condition_amount | 满额数（3）                                         |
| condition_num    | 满件数（4）                                         |
| activity_id      | 活动编号                                            |
| benefit_amount   | 减金额（1 3）                                       |
| benefit_discount | 折扣（2 4）                                         |
| create_time      | 创建时间                                            |
| range_type       | 范围类型 1、商品(spuid) 2、品类(三级分类id) 3、品牌 |
| limit_num        | 最多领用次数                                        |
| taken_count      | 已领用次数                                          |
| start_time       | 可以领取的开始日期                                  |
| end_time         | 可以领取的结束日期                                  |
| operate_time     | 修改时间                                            |
| expire_time      | 过期时间                                            |
| range_desc       | 范围描述                                            |

### 5.3.16 优惠券优惠范围表（coupon_range）

| 字段名     | 字段说明                                            |
| ---------- | --------------------------------------------------- |
| id         | 购物券编号                                          |
| coupon_id  | 优惠券id                                            |
| range_type | 范围类型 1、商品(spuid) 2、品类(三级分类id) 3、品牌 |
| range_id   | 范围id                                              |

### 5.3.17 优惠券领用表（coupon_use）

| 字段名        | 字段说明                          |
| ------------- | --------------------------------- |
| id            | 编号                              |
| coupon_id     | 购物券ID                          |
| user_id       | 用户ID                            |
| order_id      | 订单ID                            |
| coupon_status | 购物券状态（1：未使用 2：已使用） |
| get_time      | 获取时间                          |
| using_time    | 使用时间                          |
| used_time     | 支付时间                          |
| expire_time   | 过期时间                          |

### 5.3.18 收藏表（favor_info）

| 字段名      | 字段说明                   |
| ----------- | -------------------------- |
| id          | 编号                       |
| user_id     | 用户id                     |
| sku_id      | skuid                      |
| spu_id      | 商品id                     |
| is_cancel   | 是否已取消 0 正常 1 已取消 |
| create_time | 创建时间                   |
| cancel_time | 修改时间                   |

### 5.3.19 订单明细表（order_detail）

| 字段名                | 字段说明                 |
| --------------------- | ------------------------ |
| id                    | 编号                     |
| order_id              | 订单编号                 |
| sku_id                | sku_id                   |
| sku_name              | sku名称（冗余)           |
| img_url               | 图片名称（冗余)          |
| order_price           | 购买价格(下单时sku价格） |
| sku_num               | 购买个数                 |
| create_time           | 创建时间                 |
| source_type           | 来源类型                 |
| source_id             | 来源编号                 |
| split_total_amount    | 分摊总金额               |
| split_activity_amount | 分摊活动减免金额         |
| split_coupon_amount   | 分摊优惠券减免金额       |

### 5.3.20 订单明细活动关联表（order_detail_activity）

| 字段名           | 字段说明   |
| ---------------- | ---------- |
| id               | 编号       |
| order_id         | 订单id     |
| order_detail_id  | 订单明细id |
| activity_id      | 活动ID     |
| activity_rule_id | 活动规则   |
| sku_id           | skuID      |
| create_time      | 获取时间   |

### 5.3.21 订单明细优惠券关联表（order_detail_coupon）

| 字段名          | 字段说明     |
| --------------- | ------------ |
| id              | 编号         |
| order_id        | 订单id       |
| order_detail_id | 订单明细id   |
| coupon_id       | 购物券ID     |
| coupon_use_id   | 购物券领用id |
| sku_id          | skuID        |
| create_time     | 获取时间     |

### 5.3.22 订单表(order_info）

| 字段名                 | 字段说明                    |
| ---------------------- | --------------------------- |
| id                     | 编号                        |
| consignee              | 收货人                      |
| consignee_tel          | 收件人电话                  |
| total_amount           | 总金额                      |
| order_status           | 订单状态                    |
| user_id                | 用户id                      |
| payment_way            | 付款方式                    |
| delivery_address       | 送货地址                    |
| order_comment          | 订单备注                    |
| out_trade_no           | 订单交易编号（第三方支付用) |
| trade_body             | 订单描述(第三方支付用)      |
| create_time            | 创建时间                    |
| operate_time           | 操作时间                    |
| expire_time            | 失效时间                    |
| process_status         | 进度状态                    |
| tracking_no            | 物流单编号                  |
| parent_order_id        | 父订单编号                  |
| img_url                | 图片路径                    |
| province_id            | 地区                        |
| activity_reduce_amount | 促销金额                    |
| coupon_reduce_amount   | 优惠金额                    |
| original_total_amount  | 原价金额                    |
| feight_fee             | 运费                        |
| feight_fee_reduce      | 运费减免                    |
| refundable_time        | 可退款日期（签收后30天）    |

### 5.3.23 退单表（order_refund_info）

| 字段名             | 字段说明                        |
| ------------------ | ------------------------------- |
| id                 | 编号                            |
| user_id            | 用户id                          |
| order_id           | 订单id                          |
| sku_id             | skuid                           |
| refund_type        | 退款类型                        |
| refund_num         | 退货件数                        |
| refund_amount      | 退款金额                        |
| refund_reason_type | 原因类型                        |
| refund_reason_txt  | 原因内容                        |
| refund_status      | 退款状态（0：待审批 1：已退款） |
| create_time        | 创建时间                        |

### 5.3.24 订单状态流水表（order_status_log）

| 字段名       | 字段说明 |
| ------------ | -------- |
| id           | 编号     |
| order_id     | 订单编号 |
| order_status | 订单状态 |
| operate_time | 操作时间 |

### 5.3.25 支付表（payment_info）

| 字段名           | 字段说明                |
| ---------------- | ----------------------- |
| id               | 编号                    |
| out_trade_no     | 对外业务编号            |
| order_id         | 订单编号                |
| user_id          |                         |
| payment_type     | 支付类型（微信 支付宝） |
| trade_no         | 交易编号                |
| total_amount     | 支付金额                |
| subject          | 交易内容                |
| payment_status   | 支付状态                |
| create_time      | 创建时间                |
| callback_time    | 回调时间                |
| callback_content | 回调信息                |

### 5.3.26 退款表（refund_payment）

| 字段名           | 字段说明                |
| ---------------- | ----------------------- |
| id               | 编号                    |
| out_trade_no     | 对外业务编号            |
| order_id         | 订单编号                |
| sku_id           | 商品sku_id              |
| payment_type     | 支付类型（微信 支付宝） |
| trade_no         | 交易编号                |
| total_amount     | 退款金额                |
| subject          | 交易内容                |
| refund_status    | 退款状态                |
| create_time      | 创建时间                |
| callback_time    | 回调时间                |
| callback_content | 回调信息                |

### 5.3.27 SKU平台属性表（sku_attr_value）

| 字段名     | 字段说明      |
| ---------- | ------------- |
| id         | 编号          |
| attr_id    | 属性id（冗余) |
| value_id   | 属性值id      |
| sku_id     | skuid         |
| attr_name  | 属性名称      |
| value_name | 属性值名称    |

### 5.3.28 SKU信息表（sku_info）

| 字段名          | 字段说明                |
| --------------- | ----------------------- |
| id              | 库存id(itemID)          |
| spu_id          | 商品id                  |
| price           | 价格                    |
| sku_name        | sku名称                 |
| sku_desc        | 商品规格描述            |
| weight          | 重量                    |
| tm_id           | 品牌(冗余)              |
| category3_id    | 三级分类id（冗余)       |
| sku_default_img | 默认显示图片(冗余)      |
| is_sale         | 是否销售（1：是 0：否） |
| create_time     | 创建时间                |

### 5.3.29 SKU销售属性表（sku_sale_attr_value）

| 字段名               | 字段说明       |
| -------------------- | -------------- |
| id                   | id             |
| sku_id               | 库存单元id     |
| spu_id               | spu_id（冗余） |
| sale_attr_value_id   | 销售属性值id   |
| sale_attr_id         | 销售属性id     |
| sale_attr_name       | 销售属性值名称 |
| sale_attr_value_name | 销售属性值名称 |

### 5.3.30 SPU信息表（spu_info）

| 字段名       | 字段说明            |
| ------------ | ------------------- |
| id           | 商品id              |
| spu_name     | 商品名称            |
| description  | 商品描述(后台简述） |
| category3_id | 三级分类id          |
| tm_id        | 品牌id              |

### 5.3.31 SPU销售属性表（spu_sale_attr）

| 字段名            | 字段说明             |
| ----------------- | -------------------- |
| id                | 编号（业务中无关联） |
| spu_id            | 商品id               |
| base_sale_attr_id | 销售属性id           |
| sale_attr_name    | 销售属性名称（冗余） |

### 5.3.32 SPU销售属性值表（spu_sale_attr_value）

| 字段名               | 字段说明             |
| -------------------- | -------------------- |
| id                   | 销售属性值编号       |
| spu_id               | 商品id               |
| base_sale_attr_id    | 销售属性id           |
| sale_attr_value_name | 销售属性值名称       |
| sale_attr_name       | 销售属性名称（冗余） |

### 5.3.33 用户地址表（user_address）

| 字段名       | 字段说明   |
| ------------ | ---------- |
| id           | 编号       |
| user_id      | 用户id     |
| province_id  | 省份id     |
| user_address | 用户地址   |
| consignee    | 收件人     |
| phone_num    | 联系方式   |
| is_default   | 是否是默认 |

### 5.3.34 用户信息表（user_info）

| 字段名       | 字段说明     |
| ------------ | ------------ |
| id           | 编号         |
| login_name   | 用户名称     |
| nick_name    | 用户昵称     |
| passwd       | 用户密码     |
| name         | 用户姓名     |
| phone_num    | 手机号       |
| email        | 邮箱         |
| head_img     | 头像         |
| user_level   | 用户级别     |
| birthday     | 用户生日     |
| gender       | 性别 M男,F女 |
| create_time  | 创建时间     |
| operate_time | 修改时间     |
| status       | 状态         |

# 6.0业务数据采集模块

## 6.1Mysql安装

**本模块内容可参考：本人博客**-[**大数据全套软件配置**](https://blog.csdn.net/m0_58022371/article/details/126440789)

==一站式解决大数据环境搭建问题==

![image-20220919145151982](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919145151982.png)



## 6.2业务数据生成

### 6.2.1 连接MySQL	

![image-20220919145250586](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919145250586.png)

### 6.2.2 建表语句	

1）通过SQLyog创建数据库

![image-20220919145341729](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919145341729.png)

2）设置数据库名称为gmall，编码为utf-8，排序规则为utf8_general_ci

![image-20220919145351609](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919145351609.png)

3）导入数据库结构脚本（gmall.sql）

![image-20220919145401374](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919145401374.png)

### 6.2.3 生成业务数据	

1）在hadoop102的/opt/module/目录下创建db_log文件夹

```
[atguigu@hadoop102 module]$ mkdir db_log/
```

2）把gmall2020-mock-db-2021-01-22.jar和application.properties上传到hadoop102的/opt/module/db_log路径上。

3）根据需求修改application.properties相关配置

```
logging.level.root=info


spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://hadoop102:3306/gmall?characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=000000

logging.pattern.console=%m%n


mybatis-plus.global-config.db-config.field-strategy=not_null


#业务日期
mock.date=2020-06-14
#是否重置  注意：第一次执行必须设置为1，后续不需要重置不用设置为1
mock.clear=1
#是否重置用户 注意：第一次执行必须设置为1，后续不需要重置不用设置为1
mock.clear.user=1

#生成新用户数量
mock.user.count=100
#男性比例
mock.user.male-rate=20
#用户数据变化概率
mock.user.update-rate:20

#收藏取消比例
mock.favor.cancel-rate=10
#收藏数量
mock.favor.count=100

#每个用户添加购物车的概率
mock.cart.user-rate=50
#每次每个用户最多添加多少种商品进购物车
mock.cart.max-sku-count=8 
#每个商品最多买几个
mock.cart.max-sku-num=3 

#购物车来源  用户查询，商品推广，智能推荐, 促销活动
mock.cart.source-type-rate=60:20:10:10

#用户下单比例
mock.order.user-rate=50
#用户从购物中购买商品比例
mock.order.sku-rate=50
#是否参加活动
mock.order.join-activity=1
#是否使用购物券
mock.order.use-coupon=1
#购物券领取人数
mock.coupon.user-count=100

#支付比例
mock.payment.rate=70
#支付方式 支付宝：微信 ：银联
mock.payment.payment-type=30:60:10


#评价比例 好：中：差：自动
mock.comment.appraise-rate=30:10:10:50

#退款原因比例：质量问题 商品描述与实际描述不一致 缺货 号码不合适 拍错 不想买了 其他
mock.refund.reason-rate=30:10:20:5:15:5:5
```

4）并在该目录下执行，如下命令，生成2020-06-14日期数据：

```
[atguigu@hadoop102 db_log]$ java -jar gmall2020-mock-db-2021-01-22.jar
```

5）查看gmall数据库，观察是否有2020-06-14的数据出现

## 6.3Sqoop安装

### 6.3.1 下载并解压	

1）sqoop官网地址：[http://sqoop.apache.org](http://sqoop.apache.org/docs/1.4.7/index.html)

2）下载地址：http://mirrors.hust.edu.cn/apache/sqoop/1.4.6/

3）上传安装包sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz到hadoop102的/opt/software路径中

4）解压sqoop安装包到指定目录，如：

```
[atguigu@hadoop102 software]$ tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
```

5）解压sqoop安装包到指定目录，如：

```
[atguigu@hadoop102 module]$ mv sqoop-1.4.6.bin__hadoop-2.0.4-alpha/ sqoop
```

### 6.3.2 修改配置文件	

1）进入到/opt/module/sqoop/conf目录，重命名配置文件

```
[atguigu@hadoop102 conf]$ mv sqoop-env-template.sh sqoop-env.sh
```

2）修改配置文件

```
[atguigu@hadoop102 conf]$ vim sqoop-env.sh 
```

增加如下内容

```
export HADOOP_COMMON_HOME=/opt/module/hadoop-3.1.3
export HADOOP_MAPRED_HOME=/opt/module/hadoop-3.1.3
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.5.7
export ZOOCFGDIR=/opt/module/zookeeper-3.5.7/conf
```

### 6.3.3 拷贝JDBC驱动	

1）将mysql-connector-java-5.1.48.jar 上传到/opt/software路径

2）进入到/opt/software/路径，拷贝jdbc驱动到sqoop的lib目录下。

```
[atguigu@hadoop102 software]$ cp mysql-connector-java-5.1.48.jar /opt/module/sqoop/lib/
```

### 6.3.4 验证Sqoop	

（1）我们可以通过某一个command来验证sqoop配置是否正确：

```
[atguigu@hadoop102 sqoop]$ bin/sqoop help
```

（2）出现一些Warning警告（警告信息已省略），并伴随着帮助命令的输出：

```
Available commands:
  codegen            Generate code to interact with database records
  create-hive-table     Import a table definition into Hive
  eval               Evaluate a SQL statement and display the results
  export             Export an HDFS directory to a database table
  help               List available commands
  import             Import a table from a database to HDFS
  import-all-tables     Import tables from a database to HDFS
  import-mainframe    Import datasets from a mainframe server to HDFS
  job                Work with saved jobs
  list-databases        List available databases on a server
  list-tables           List available tables in a database
  merge              Merge results of incremental imports
  metastore           Run a standalone Sqoop metastore
  version            Display version information
```



### 6.3.5 测试Sqoop是否能够成功连接数据库	

```
[atguigu@hadoop102 sqoop]$ bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password fgl123
```

出现如下输出：

```
information_schema
metastore
mysql
oozie
performance_schema
```

### 6.3.6 Sqoop基本使用	

将mysql中user_info表数据导入到HDFS的/test路径

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/gmall \
--username root \
--password fgl123 \
--table user_info \
--columns id,login_name \
--where "id>=10 and id<=30" \
--target-dir /test \
--delete-target-dir \
--fields-terminated-by '\t' \
--num-mappers 2 \
--split-by id
```

## 6.4同步策略

**数据同步策略的类型包括：全量同步、增量同步、新增及变化同步、特殊情况**

Ø 全量表：存储完整的数据。

Ø 增量表：存储新增加的数据。

Ø 新增及变化表：存储新增加的数据和变化的数据。

Ø 特殊表：只需要存储一次。

### 6.4.1 全量同步策略	

![image-20220919152640981](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919152640981.png)

### 6.4.2 增量同步策略	

![image-20220919152715505](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919152715505.png)





### 6.4.3 新增及变化策略	

![image-20220919152734323](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919152734323.png)



### 6.4.4 特殊策略	

​		某些特殊的表，可不必遵循上述同步策略。例如某些不会发生变化的表（地区表，省份表，民族表）可以只存一份固定值。

## 6.5业务数据导入HDFS

在生产环境，个别小公司，为了简单处理，所有表全量导入。

中大型公司，由于数据量比较大，还是严格按照同步策略导入数据。

### 6.5.1 分析表同步策略	

![image-20220919152814039](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220919152814039.png)

### 6.5.2 业务数据首日同步脚本	

**1）脚本编写**

**（1）在/home/atguigu/bin目录下创建**

```
[atguigu@hadoop102 bin]$ vim mysql_to_hdfs_init.sh
```

**添加如下内容：**

```
#! /bin/bash

APP=gmall
sqoop=/opt/module/sqoop/bin/sqoop

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password fgl123 \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 where \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/db/$1/$do_date
}

import_order_info(){
  import_data order_info "select
                            id, 
                            total_amount, 
                            order_status, 
                            user_id, 
                            payment_way,
                            delivery_address,
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            expire_time,
                            tracking_no,
                            province_id,
                            activity_reduce_amount,
                            coupon_reduce_amount,                            
                            original_total_amount,
                            feight_fee,
                            feight_fee_reduce      
                        from order_info"
}

import_coupon_use(){
  import_data coupon_use "select
                          id,
                          coupon_id,
                          user_id,
                          order_id,
                          coupon_status,
                          get_time,
                          using_time,
                          used_time,
                          expire_time
                        from coupon_use"
}

import_order_status_log(){
  import_data order_status_log "select
                                  id,
                                  order_id,
                                  order_status,
                                  operate_time
                                from order_status_log"
}

import_user_info(){
  import_data "user_info" "select 
                            id,
                            login_name,
                            nick_name,
                            name,
                            phone_num,
                            email,
                            user_level, 
                            birthday,
                            gender,
                            create_time,
                            operate_time
                          from user_info"
}

import_order_detail(){
  import_data order_detail "select 
                              id,
                              order_id, 
                              sku_id,
                              sku_name,
                              order_price,
                              sku_num, 
                              create_time,
                              source_type,
                              source_id,
                              split_total_amount,
                              split_activity_amount,
                              split_coupon_amount
                            from order_detail"
}

import_payment_info(){
  import_data "payment_info"  "select 
                                id,  
                                out_trade_no, 
                                order_id, 
                                user_id, 
                                payment_type, 
                                trade_no, 
                                total_amount,  
                                subject, 
                                payment_status,
                                create_time,
                                callback_time 
                              from payment_info"
}

import_comment_info(){
  import_data comment_info "select
                              id,
                              user_id,
                              sku_id,
                              spu_id,
                              order_id,
                              appraise,
                              create_time
                            from comment_info"
}

import_order_refund_info(){
  import_data order_refund_info "select
                                id,
                                user_id,
                                order_id,
                                sku_id,
                                refund_type,
                                refund_num,
                                refund_amount,
                                refund_reason_type,
                                refund_status,
                                create_time
                              from order_refund_info"
}

import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          is_sale,
                          create_time
                        from sku_info"
}

import_base_category1(){
  import_data "base_category1" "select 
                                  id,
                                  name 
                                from base_category1"
}

import_base_category2(){
  import_data "base_category2" "select
                                  id,
                                  name,
                                  category1_id 
                                from base_category2"
}

import_base_category3(){
  import_data "base_category3" "select
                                  id,
                                  name,
                                  category2_id
                                from base_category3"
}

import_base_province(){
  import_data base_province "select
                              id,
                              name,
                              region_id,
                              area_code,
                              iso_code,
                              iso_3166_2
                            from base_province"
}

import_base_region(){
  import_data base_region "select
                              id,
                              region_name
                            from base_region"
}

import_base_trademark(){
  import_data base_trademark "select
                                id,
                                tm_name
                              from base_trademark"
}

import_spu_info(){
  import_data spu_info "select
                            id,
                            spu_name,
                            category3_id,
                            tm_id
                          from spu_info"
}

import_favor_info(){
  import_data favor_info "select
                          id,
                          user_id,
                          sku_id,
                          spu_id,
                          is_cancel,
                          create_time,
                          cancel_time
                        from favor_info"
}

import_cart_info(){
  import_data cart_info "select
                        id,
                        user_id,
                        sku_id,
                        cart_price,
                        sku_num,
                        sku_name,
                        create_time,
                        operate_time,
                        is_ordered,
                        order_time,
                        source_type,
                        source_id
                      from cart_info"
}

import_coupon_info(){
  import_data coupon_info "select
                          id,
                          coupon_name,
                          coupon_type,
                          condition_amount,
                          condition_num,
                          activity_id,
                          benefit_amount,
                          benefit_discount,
                          create_time,
                          range_type,
                          limit_num,
                          taken_count,
                          start_time,
                          end_time,
                          operate_time,
                          expire_time
                        from coupon_info"
}

import_activity_info(){
  import_data activity_info "select
                              id,
                              activity_name,
                              activity_type,
                              start_time,
                              end_time,
                              create_time
                            from activity_info"
}

import_activity_rule(){
    import_data activity_rule "select
                                    id,
                                    activity_id,
                                    activity_type,
                                    condition_amount,
                                    condition_num,
                                    benefit_amount,
                                    benefit_discount,
                                    benefit_level
                                from activity_rule"
}

import_base_dic(){
    import_data base_dic "select
                            dic_code,
                            dic_name,
                            parent_code,
                            create_time,
                            operate_time
                          from base_dic"
}


import_order_detail_activity(){
    import_data order_detail_activity "select
                                                                id,
                                                                order_id,
                                                                order_detail_id,
                                                                activity_id,
                                                                activity_rule_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_activity"
}


import_order_detail_coupon(){
    import_data order_detail_coupon "select
                                                                id,
								                                                order_id,
                                                                order_detail_id,
                                                                coupon_id,
                                                                coupon_use_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_coupon"
}


import_refund_payment(){
    import_data refund_payment "select
                                                        id,
                                                        out_trade_no,
                                                        order_id,
                                                        sku_id,
                                                        payment_type,
                                                        trade_no,
                                                        total_amount,
                                                        subject,
                                                        refund_status,
                                                        create_time,
                                                        callback_time
                                                    from refund_payment"                                                    

}

import_sku_attr_value(){
    import_data sku_attr_value "select
                                                    id,
                                                    attr_id,
                                                    value_id,
                                                    sku_id,
                                                    attr_name,
                                                    value_name
                                                from sku_attr_value"
}


import_sku_sale_attr_value(){
    import_data sku_sale_attr_value "select
                                                            id,
                                                            sku_id,
                                                            spu_id,
                                                            sale_attr_value_id,
                                                            sale_attr_id,
                                                            sale_attr_name,
                                                            sale_attr_value_name
                                                        from sku_sale_attr_value"
}

case $1 in
  "order_info")
     import_order_info
;;
  "base_category1")
     import_base_category1
;;
  "base_category2")
     import_base_category2
;;
  "base_category3")
     import_base_category3
;;
  "order_detail")
     import_order_detail
;;
  "sku_info")
     import_sku_info
;;
  "user_info")
     import_user_info
;;
  "payment_info")
     import_payment_info
;;
  "base_province")
     import_base_province
;;
  "base_region")
     import_base_region
;;
  "base_trademark")
     import_base_trademark
;;
  "activity_info")
      import_activity_info
;;
  "cart_info")
      import_cart_info
;;
  "comment_info")
      import_comment_info
;;
  "coupon_info")
      import_coupon_info
;;
  "coupon_use")
      import_coupon_use
;;
  "favor_info")
      import_favor_info
;;
  "order_refund_info")
      import_order_refund_info
;;
  "order_status_log")
      import_order_status_log
;;
  "spu_info")
      import_spu_info
;;
  "activity_rule")
      import_activity_rule
;;
  "base_dic")
      import_base_dic
;;
  "order_detail_activity")
      import_order_detail_activity
;;
  "order_detail_coupon")
      import_order_detail_coupon
;;
  "refund_payment")
      import_refund_payment
;;
  "sku_attr_value")
      import_sku_attr_value
;;
  "sku_sale_attr_value")
      import_sku_sale_attr_value
;;
  "all")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_region
   import_base_province
   import_base_trademark
   import_activity_info
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
   import_order_detail_activity
   import_order_detail_coupon
   import_refund_payment
   import_sku_attr_value
   import_sku_sale_attr_value
;;
esac
```

**说明1：**

​		[ -n 变量值 ] 判断变量的值，是否为空

​		-- 变量的值，非空，返回true

​		-- 变量的值，为空，返回false

**说明2：**

​		查看date命令的使用，[atguigu@hadoop102 ~]$ date --help

**（2）增加脚本执行权限**

```
[atguigu@hadoop102 bin]$ chmod +x mysql_to_hdfs_init.sh
```

**2）脚本使用**

```
[atguigu@hadoop102 bin]$ mysql_to_hdfs_init.sh all 2020-06-14
```

### 6.5.3 业务数据每日同步脚本	

1）脚本编写
（1）在/home/atguigu/bin目录下创建

```
[atguigu@hadoop102 bin]$ vim mysql_to_hdfs.sh
```

添加如下内容：

```
#! /bin/bash

APP=gmall
sqoop=/opt/module/sqoop/bin/sqoop

if [ -n "$2" ] ;then
    do_date=$2
else
    do_date=`date -d '-1 day' +%F`
fi

import_data(){
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password fgl123 \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 and  \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'

hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /origin_data/$APP/db/$1/$do_date
}

import_order_info(){
  import_data order_info "select
                            id, 
                            total_amount, 
                            order_status, 
                            user_id, 
                            payment_way,
                            delivery_address,
                            out_trade_no, 
                            create_time, 
                            operate_time,
                            expire_time,
                            tracking_no,
                            province_id,
                            activity_reduce_amount,
                            coupon_reduce_amount,                            
                            original_total_amount,
                            feight_fee,
                            feight_fee_reduce      
                        from order_info
                        where (date_format(create_time,'%Y-%m-%d')='$do_date' 
                        or date_format(operate_time,'%Y-%m-%d')='$do_date')"
}

import_coupon_use(){
  import_data coupon_use "select
                          id,
                          coupon_id,
                          user_id,
                          order_id,
                          coupon_status,
                          get_time,
                          using_time,
                          used_time,
                          expire_time
                        from coupon_use
                        where (date_format(get_time,'%Y-%m-%d')='$do_date'
                        or date_format(using_time,'%Y-%m-%d')='$do_date'
                        or date_format(used_time,'%Y-%m-%d')='$do_date'
                        or date_format(expire_time,'%Y-%m-%d')='$do_date')"
}

import_order_status_log(){
  import_data order_status_log "select
                                  id,
                                  order_id,
                                  order_status,
                                  operate_time
                                from order_status_log
                                where date_format(operate_time,'%Y-%m-%d')='$do_date'"
}

import_user_info(){
  import_data "user_info" "select 
                            id,
                            login_name,
                            nick_name,
                            name,
                            phone_num,
                            email,
                            user_level, 
                            birthday,
                            gender,
                            create_time,
                            operate_time
                          from user_info 
                          where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                          or DATE_FORMAT(operate_time,'%Y-%m-%d')='$do_date')"
}

import_order_detail(){
  import_data order_detail "select 
                              id,
                              order_id, 
                              sku_id,
                              sku_name,
                              order_price,
                              sku_num, 
                              create_time,
                              source_type,
                              source_id,
                              split_total_amount,
                              split_activity_amount,
                              split_coupon_amount
                            from order_detail 
                            where DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date'"
}

import_payment_info(){
  import_data "payment_info"  "select 
                                id,  
                                out_trade_no, 
                                order_id, 
                                user_id, 
                                payment_type, 
                                trade_no, 
                                total_amount,  
                                subject, 
                                payment_status,
                                create_time,
                                callback_time 
                              from payment_info 
                              where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                              or DATE_FORMAT(callback_time,'%Y-%m-%d')='$do_date')"
}

import_comment_info(){
  import_data comment_info "select
                              id,
                              user_id,
                              sku_id,
                              spu_id,
                              order_id,
                              appraise,
                              create_time
                            from comment_info
                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_order_refund_info(){
  import_data order_refund_info "select
                                id,
                                user_id,
                                order_id,
                                sku_id,
                                refund_type,
                                refund_num,
                                refund_amount,
                                refund_reason_type,
                                refund_status,
                                create_time
                              from order_refund_info
                              where date_format(create_time,'%Y-%m-%d')='$do_date'"
}

import_sku_info(){
  import_data sku_info "select 
                          id,
                          spu_id,
                          price,
                          sku_name,
                          sku_desc,
                          weight,
                          tm_id,
                          category3_id,
                          is_sale,
                          create_time
                        from sku_info where 1=1"
}

import_base_category1(){
  import_data "base_category1" "select 
                                  id,
                                  name 
                                from base_category1 where 1=1"
}

import_base_category2(){
  import_data "base_category2" "select
                                  id,
                                  name,
                                  category1_id 
                                from base_category2 where 1=1"
}

import_base_category3(){
  import_data "base_category3" "select
                                  id,
                                  name,
                                  category2_id
                                from base_category3 where 1=1"
}

import_base_province(){
  import_data base_province "select
                              id,
                              name,
                              region_id,
                              area_code,
                              iso_code,
                              iso_3166_2
                            from base_province
                            where 1=1"
}

import_base_region(){
  import_data base_region "select
                              id,
                              region_name
                            from base_region
                            where 1=1"
}

import_base_trademark(){
  import_data base_trademark "select
                                id,
                                tm_name
                              from base_trademark
                              where 1=1"
}

import_spu_info(){
  import_data spu_info "select
                            id,
                            spu_name,
                            category3_id,
                            tm_id
                          from spu_info
                          where 1=1"
}

import_favor_info(){
  import_data favor_info "select
                          id,
                          user_id,
                          sku_id,
                          spu_id,
                          is_cancel,
                          create_time,
                          cancel_time
                        from favor_info
                        where 1=1"
}

import_cart_info(){
  import_data cart_info "select
                        id,
                        user_id,
                        sku_id,
                        cart_price,
                        sku_num,
                        sku_name,
                        create_time,
                        operate_time,
                        is_ordered,
                        order_time,
                        source_type,
                        source_id
                      from cart_info
                      where 1=1"
}

import_coupon_info(){
  import_data coupon_info "select
                          id,
                          coupon_name,
                          coupon_type,
                          condition_amount,
                          condition_num,
                          activity_id,
                          benefit_amount,
                          benefit_discount,
                          create_time,
                          range_type,
                          limit_num,
                          taken_count,
                          start_time,
                          end_time,
                          operate_time,
                          expire_time
                        from coupon_info
                        where 1=1"
}

import_activity_info(){
  import_data activity_info "select
                              id,
                              activity_name,
                              activity_type,
                              start_time,
                              end_time,
                              create_time
                            from activity_info
                            where 1=1"
}

import_activity_rule(){
    import_data activity_rule "select
                                    id,
                                    activity_id,
                                    activity_type,
                                    condition_amount,
                                    condition_num,
                                    benefit_amount,
                                    benefit_discount,
                                    benefit_level
                                from activity_rule
                                where 1=1"
}

import_base_dic(){
    import_data base_dic "select
                            dic_code,
                            dic_name,
                            parent_code,
                            create_time,
                            operate_time
                          from base_dic
                          where 1=1"
}


import_order_detail_activity(){
    import_data order_detail_activity "select
                                                                id,
                                                                order_id,
                                                                order_detail_id,
                                                                activity_id,
                                                                activity_rule_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_activity
                                                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}


import_order_detail_coupon(){
    import_data order_detail_coupon "select
                                                                id,
								                                                order_id,
                                                                order_detail_id,
                                                                coupon_id,
                                                                coupon_use_id,
                                                                sku_id,
                                                                create_time
                                                            from order_detail_coupon
                                                            where date_format(create_time,'%Y-%m-%d')='$do_date'"
}


import_refund_payment(){
    import_data refund_payment "select
                                                        id,
                                                        out_trade_no,
                                                        order_id,
                                                        sku_id,
                                                        payment_type,
                                                        trade_no,
                                                        total_amount,
                                                        subject,
                                                        refund_status,
                                                        create_time,
                                                        callback_time
                                                    from refund_payment
                                                    where (DATE_FORMAT(create_time,'%Y-%m-%d')='$do_date' 
                                                    or DATE_FORMAT(callback_time,'%Y-%m-%d')='$do_date')"                                                    

}

import_sku_attr_value(){
    import_data sku_attr_value "select
                                                    id,
                                                    attr_id,
                                                    value_id,
                                                    sku_id,
                                                    attr_name,
                                                    value_name
                                                from sku_attr_value
                                                where 1=1"
}


import_sku_sale_attr_value(){
    import_data sku_sale_attr_value "select
                                                            id,
                                                            sku_id,
                                                            spu_id,
                                                            sale_attr_value_id,
                                                            sale_attr_id,
                                                            sale_attr_name,
                                                            sale_attr_value_name
                                                        from sku_sale_attr_value
                                                        where 1=1"
}

case $1 in
  "order_info")
     import_order_info
;;
  "base_category1")
     import_base_category1
;;
  "base_category2")
     import_base_category2
;;
  "base_category3")
     import_base_category3
;;
  "order_detail")
     import_order_detail
;;
  "sku_info")
     import_sku_info
;;
  "user_info")
     import_user_info
;;
  "payment_info")
     import_payment_info
;;
  "base_province")
     import_base_province
;;
  "activity_info")
      import_activity_info
;;
  "cart_info")
      import_cart_info
;;
  "comment_info")
      import_comment_info
;;
  "coupon_info")
      import_coupon_info
;;
  "coupon_use")
      import_coupon_use
;;
  "favor_info")
      import_favor_info
;;
  "order_refund_info")
      import_order_refund_info
;;
  "order_status_log")
      import_order_status_log
;;
  "spu_info")
      import_spu_info
;;
  "activity_rule")
      import_activity_rule
;;
  "base_dic")
      import_base_dic
;;
  "order_detail_activity")
      import_order_detail_activity
;;
  "order_detail_coupon")
      import_order_detail_coupon
;;
  "refund_payment")
      import_refund_payment
;;
  "sku_attr_value")
      import_sku_attr_value
;;
  "sku_sale_attr_value")
      import_sku_sale_attr_value
;;
"all")
   import_base_category1
   import_base_category2
   import_base_category3
   import_order_info
   import_order_detail
   import_sku_info
   import_user_info
   import_payment_info
   import_base_trademark
   import_activity_info
   import_cart_info
   import_comment_info
   import_coupon_use
   import_coupon_info
   import_favor_info
   import_order_refund_info
   import_order_status_log
   import_spu_info
   import_activity_rule
   import_base_dic
   import_order_detail_activity
   import_order_detail_coupon
   import_refund_payment
   import_sku_attr_value
   import_sku_sale_attr_value
;;
esac
```

**（2）增加脚本执行权限**

```
[atguigu@hadoop102 bin]$ chmod +x mysql_to_hdfs.sh
```

**2）脚本使用**

```
[atguigu@hadoop102 bin]$ mysql_to_hdfs.sh all 2020-06-15
```

# 7.0数据环境准备

## 7.1Hive安装部署

### 7.1.1Hive安装部署

（1）把apache-hive-3.1.2-bin.tar.gz上传到Linux的/opt/software目录下

（2）解压apache-hive-3.1.2-bin.tar.gz到/opt/module/目录下面

```
[atguigu@hadoop102 software]$ tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

（3）修改apache-hive-3.1.2-bin.tar.gz的名称为hive

```
[atguigu@hadoop102 software]$ mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
```

（4）修改/etc/profile.d/my_env.sh，添加环境变量

```
[atguigu@hadoop102 software]$ sudo vim /etc/profile.d/my_env.sh
```

添加内容

```
#HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin
```

重启Xshell对话框或者source一下 /etc/profile.d/my_env.sh文件，使环境变量生效

```
[atguigu@hadoop102 software]$ source /etc/profile.d/my_env.sh
```

（5）解决日志Jar包冲突，进入/opt/module/hive/lib目录

```
[atguigu@hadoop102 lib]$ mv log4j-slf4j-impl-2.10.0.jar log4j-slf4j-impl-2.10.0.jar.bak
```



## 7.2Hive元数据配置到Mysql

### 7.2.1拷贝驱动

将MySQL的JDBC驱动拷贝到Hive的lib目录下

```
[atguigu@hadoop102 lib]$ cp /opt/software/mysql-connector-java-5.1.27.jar /opt/module/hive/lib/
```

### 7.2.2配置Metastore到Mysql

（1）在$HIVE_HOME/conf目录下新建hive-site.xml文件

```
[atguigu@hadoop102 conf]$ vim hive-site.xml
```

（2）添加如下内容

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>fgl123</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
</configuration>
```

## 7.3启动Hive

### 7.3.1初始化源数据库

（1）登陆MySQL

```
[atguigu@hadoop102 conf]$ mysql -uroot -p000000
```

（2）新建Hive元数据库

```
mysql> create database metastore;
mysql> quit;
```

（3）初始化Hive元数据库

```
[atguigu@hadoop102 conf]$ schematool -initSchema -dbType mysql -verbose
```

### 7.3.2启动Hive客户端

（1）启动Hive客户端

```
[atguigu@hadoop102 hive]$ bin/hive
```

（2）查看一下数据库

```
hive (default)> show databases;
OK
database_name
default
```

