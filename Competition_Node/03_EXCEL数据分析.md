# EXCEL

## 1基本操作

### 案例一：分列日期

**需求**：使用Excel数据分列将日期中的星期一至星期日单独拆分出来，放在另一列中。

例如：

![image-20220907102012825](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907102012825-16625172390011.png)

**步骤**：

①选中原始数据：![image-20220907102952710](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907102952710.png)

②点击数据，然后点击分列：![image-20220907103056092](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907103056092.png)

③点击固定宽度：![image-20220907103245058](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907103245058.png)

④在刻度线处点击即可分割

![image-20220907103447129](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907103447129.png)

### 案例二：分列身份证号

①选中身份证号数据：![image-20220907103626314](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907103626314.png)

②③步骤同上；

④在年月日份前加刻度线![image-20220907103912090](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907103912090.png)

**输入函数下拉即可：**

![image-20220913193434865](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220913193434865.png)

### 案例三：筛选原始数据的商品列的日用品

①右键商品类型选择筛选：![image-20220907104850357](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907104850357.png)

②点击小箭头选择筛选条件：![image-20220907105001008](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907105001008-16625191044484.png)

## 2Excel函数

### 案例一：掌握常用统计函数（COUNT，COUNTIF,COUNIFS）

**1.COUNT使用:**

①在数据下方输入=count

②选中单元格，回车即可显示单元格数量

![image-20220907110929426](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907110929426.png)
注意：count只能对数字计数；若对文本计数需要使用counta，用法和count相同

**2.COUNTIF（range，criteria）**

统计指定范围内**满足某个条件的**单元格个数

**需求：**求不同商品类型下有多少种产品？

商品类型原始图：![image-20220907200330885](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907200330885.png)

注：商品名称和商品类型不能重复；

例如：![image-20220907200518457](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907200518457.png)

如果出现上述重复现象，首先要对其进行去重操作

①获取数据：![image-20220907200019767](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907200019767.png)

②点击上方数据，删除重复项

![image-20220907200146536](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907200146536.png)

③在商品类型后面输入countif(),然后选中商品原始数据中商品类型的单元格

选中后点击商品类型：零食，回车生成产品个数。

![image-20220907201000209](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907201000209.png)

④当鼠标在产品个数右下角变成十字符号时双击，下面数据自动生成

![image-20220907201338763](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907201338763.png)

![image-20220907201354580](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907201354580.png)

**3.countifs函数使用：**

**需求：**求不同商品类型下单价50以上和50以下的商品分别有多少种：

原始数据：![image-20220907202153635](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907202153635.png)

①在商品类型后输入=countifs()

![image-20220907202710227](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907202710227.png)

②选中所有商品类型

![image-20220907203108837](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907203108837.png)

③依次选中零食和单价

![image-20220907204312372](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220907204312372.png)

④在各个条件后输入"<"&任意一个单元格

![image-20220908082332449](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908082332449.png)

在P1单元格中输入50，回车生成小于50的数据

⑤当鼠标在产单元格右下角变成十字符号时双击，下面数据自动生成

![image-20220908082901040](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908082901040.png)

⑥将下方大于50的<改为＞号，完成。

### 案例二：掌握常用求和函数(SUM,SUMIF,SUMIFS)

**1，SUM函数**：

在数据下方输入=sum 选中上方数据，点击回车生成总和

![image-20220908083129930](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908083129930.png)

2，SUMIF（条件求和）函数：

**需求：**求不同商品类型下有多少种产品：

①sumif本身不需要辅助列，但求个数的时候要插入一个辅助列

![image-20220908090154781](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908090154781.png)

②在零食后面输入=sumif()，依次选中全部商品类型，零食和辅助列。

![image-20220908090345489](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908090345489.png)

**3，sumifs函数：**

①输入=sumifs()，依次选中辅助列，商品类型，零食，单价

![image-20220908091400389](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908091400389.png)

②输入"<"&，然后点击50的单元格，回车生成数据

![image-20220908091650331](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908091650331.png)





### 案例三：掌握常用逻辑运算函数(IF)

格式：=iF(测试条件，真值，[假值])

例：在单元格中输入=if(点击单价，输入>50,1,0)

![image-20220909093551942](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909093551942.png)

如果大于50则输出1，小于50则输出0

### 案例四：掌握日期函数(YEAR，MONTH,DAY)

在单元格中输入=year(),选中日期单元格即可出现年份，

![image-20220909095333144](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909095333144.png)

输入=month()即可出现月份![image-20220909095508323](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909095508323.png)

输入=day()即可出现具体天数![image-20220909095607973](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909095607973.png)



### 案例五：掌握高效的匹配查找函数(VLOOKUP,MATCH,INDEX)

**1.VLOOKUP函数：**

**需求：**用户住址一栏为空，对数据进行补全；

①输入=vlookup()，点击购买用户id的单元格

![image-20220908104748166](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908104748166.png)

②打开用户原始表，选中A到D列数据输入逗号，选中用户地址，在输入逗号，输入0，选中精确匹配，回车，数据生成

![](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908104828148.png)

2，match函数

需求，求电动牙刷在第几行第几列？

①求第几行：

输入函数=MATCH(查找值，查找区域，匹配类型)

![image-20220909100909911](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909100909911.png)



②求第几列

![image-20220909101053280](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909101053280.png)

3INDEX函数

与match函数相反，得知在几行几列，求它的商品名称

①选中筛选区间，然后输入四行二列，即4，2

![image-20220909101311219](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909101311219.png)

②生成数据：

![image-20220909101409260](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909101409260.png)

## 3Excel报表实战

### 案例一：使用函数实现报表开发

**1，快速生成日期：**

①点击单元格，然后+1即可

![image-20220908105427632](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908105427632.png)



②向下双击，快速生成下方日期

![image-20220908105756874](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908105756874.png)

**2，快速生成星期**

①在日期单元格中输入=，然后选中前方日期单元格

![image-20220908111146282](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908111146282.png)

②点击回车后默认生成数字，点击常规

![image-20220908111316031](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908111316031.png)

③选中其他数据格式，选择日期，星期

![image-20220908111425302](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908111425302.png)

**3，物品销售量**

①**使用countif函数**，选中商品原始数据的全部日期，然后选中日期7/12，中间用，隔开  =COUNTIF(完整原始数据!A:A,A24)

![image-20220908112647807](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908112647807.png)

②单元格右下角双击快速生成下方数据

**4，商品进价：**

①**使用sumifs函数**:在商品进价处输入sumifs()函数；

在商品原始数据中选中商品进价，在函数公式中输入一个逗号，再选中全部日期![image-20220908133346092](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908133346092.png)



②输入逗号后，再选中筛选条件2021/7/12

![image-20220908133623314](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908133623314.png)

5,商品售价；

在商品进价求出来的基础上直接向右拖拽

![image-20220908135803306](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908135803306.png)

7，利润

在单元格中输入=  然后点击商品售价的单元格，再点击商品进价的单元格。

![image-20220908140234055](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220908140234055.png)

### 案例二：创建数据透视表

①点击插入，点击数据透视表

![image-20220909102521332](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909102521332.png)

选中列![image-20220909102648924](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909102648924.png)

②CTRL+shift＋↓选中所有行

③在右方自由拖动完成数据透视表

![image-20220909103140058](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909103140058.png)

## 4Excel数据透视及绘图

### 案例一：添加计算字段

![image-20220909103552825](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909103552825.png)

添加计算字段

![image-20220909103817010](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909103817010.png)

### 案例二：切片筛选器数据透视表

①首先点击数据透视表中的一个数据

![image-20220909104213867](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909104213867.png)

②点击想要插入的切片器，以商品类型为例![image-20220909104327805](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909104327805.png)

③点击不同的商品类型，数据透视表会有相应的变化

![image-20220909104415612](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909104415612.png)

④右键选择大小和属性，可以对切片器进行属性设置

![image-20220909104610550](https://fadimages-1313787074.cos.ap-nanjing.myqcloud.com/image-20220909104610550.png)



### 案例三：创建图表

