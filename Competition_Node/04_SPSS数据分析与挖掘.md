[TOC]









# 05_SPSS数据分析与挖掘

**spss下载**

[SPSS Modeler - 中国 | IBM](https://www.ibm.com/cn-zh/products/spss-modeler)

## 1.1数据读取

### 案例一：文本文件的读取

在下方选择"源"，将"变量文件"托出来。

![image-20220907102631666](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907102631666.png)

将"变量文件"右键编辑：完之后点击应用-确定。

![image-20220907102903665](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907102903665.png)

在输出选项中选择"表格"，在txt文件上选择链接

![image-20220907103029028](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907103029028.png)

![image-20220907103057165](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907103057165.png)

在表格上点击"运行"

![image-20220907103159523](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907103159523.png)

![image-20220907103224444](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907103224444.png)

### 案例二：Excel文件的读取

在"源"中将excel拖出来

![image-20220907103557994](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907103557994.png)

右键excel编辑

![image-20220907104334537](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907104334537.png)

在输出里将"表格"拖出来。右键excel点击"连接"。右键点击" 表格"运行。

![image-20220907104628272](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907104628272.png)

### 案例三：SPSS文件的读取

![image-20220907104724191](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907104724191.png)

![image-20220907104930382](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907104930382.png)

### 案例四：数据库文件的读取

打开控制面板-windows工具-ODBC数据源

![image-20220907105601408](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907105601408.png)

打开之后点击"添加"

![image-20220907105713918](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907105713918.png)

![image-20220907105829816](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907105829816.png)

![image-20220907110101262](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907110101262.png)

![image-20220907110747506](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907110747506.png)

选择mdb文件之后点击确定。

之后回到"源"中将数据库文件拖出来，

![image-20220907110959456](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907110959456.png)

右键编辑

![image-20220907111258186](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907111258186.png)

别忘了选择**表名称**，最后点击**应用-确定**

## 1.2数据清洗

### 案例一： 缺失值分析处理

#### 1.2.1.1 缺失值定义

让**模拟数据.txt**连接一个输出"数据审核"

![image-20220907131025716](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907131025716.png)

运行数据审核。

<img src="https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907131226576.png" alt="image-20220907131226576" style="zoom:50%;" />

观察"有效"这一列，前20行应该有20条有效数据，小于20行应该就是有缺失值。

右键点击模拟数据.txt，

![image-20220907131743559](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907131743559.png)

找到有缺失值的那一列，点击缺失值，选择指定

![image-20220907131939616](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907131939616.png)

勾选**定义空白**

![image-20220907132223655](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907132223655.png)

#### 1.2.1.2 缺失值处理

**运行数据审核**，点击质量，将有缺失值的那一列"缺失插补"

![image-20220907132725615](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907132725615.png)

再点击上方的"生成"，点击"缺失值过滤节点"

![image-20220907132842987](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907132842987.png)

![image-20220907133031179](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907133031179.png)

配置完之后就会生成一个'过滤节点'，将这个过滤节点连接到过滤节点中

![image-20220907133213799](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907133213799.png)

这样操作的话是直接删除该列，如果不想删除有缺失值的列可以运行数据审核，将后面的方法固定里面的改为平均值。

![image-20220907134137312](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907134137312.png)

然后点击**生成-缺失值超节点**

![image-20220907134242675](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907134242675.png)

![image-20220907134321651](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907134321651.png)

之后就会生成一个**缺失值插补**的节点

将这个节点与模拟数据.txt进行连接。

![image-20220907134536305](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907134536305.png)

### 案例二：异常值分析忻处理

首先通过数据审核确定异常值。

![image-20220907174613118](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174613118.png)

异常值确定--散点图

![image-20220907174751324](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174751324.png)

离群值和极值的处理。

![image-20220907174824195](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174824195.png)

![image-20220907174916458](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174916458.png)

![image-20220907174938721](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174938721.png)

![image-20220907174952886](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907174952886.png)

### 案例三： 重复值分析处理                                                                                                                                                                                                                                                                                                 

![image-20220907135542711](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907135542711.png)

![image-20220907135749398](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907135749398.png)



连接一下，编辑**"区分"**

![image-20220907140025575](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907140025575.png)

![image-20220907140205223](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907140205223.png)

![image-20220907140247650](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907140247650.png)

用个输出的表格输出

![image-20220907140839463](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220907140839463.png)



## 1.3数据基本分析

### 案例一：数据质量分析

![image-20220908084847704](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908084847704.png)

![image-20220908085014327](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908085014327.png)

![image-20220908085103068](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908085103068.png)

![image-20220908085226300](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908085226300.png)

![image-20220908085245955](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908085245955.png)

![image-20220908085518176](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908085518176.png)

![image-20220908090117126](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090117126.png)

![image-20220908090207284](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090207284.png)

![image-20220908090259844](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090259844.png)

![image-20220908090506473](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090506473.png)

![image-20220908090530506](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090530506.png)

### 案例二：描述性统计分忻

![image-20220908090912685](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908090912685.png)

![image-20220908091221612](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091221612.png)

![image-20220908091244374](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091244374.png)

![image-20220908091331509](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091331509.png)

点击生成

![image-20220908091651350](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091651350.png)

### 案例三：探索性分析

![image-20220908091934800](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091934800.png)

![image-20220908091919154](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908091919154.png)

![image-20220908092029791](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092029791.png)

### 案例四：二分类变量相关性分析

![image-20220908100008414](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908100008414.png)

![image-20220908092131289](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092131289.png)

![image-20220908092159187](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092159187.png)

![image-20220908092216264](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092216264.png)

![image-20220908092302949](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092302949.png)

![image-20220908092334612](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092334612.png)

### 案例五：变量的重要性分析

![image-20220908092354723](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092354723.png)

![image-20220908095926970](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908095926970.png)

![image-20220908092406758](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092406758.png)

![image-20220908092438833](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092438833.png)

![image-20220908092458909](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092458909.png)

![image-20220908092511511](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220908092511511.png)

## 1.4常用分析方法

### 案例一：二项逻辑回归基本操作

![image-20220910154349192](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910154349192.png)

![image-20220910154434949](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910154434949.png)

![image-20220910154547339](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910154547339.png)

![image-20220910154610545](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910154610545.png)

### 案例二：多项逻辑回归基本操作

![image-20220910155230228](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155230228.png)

![image-20220910155408984](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155408984.png)

### 案例三：关联分析基本操作

![image-20220910155459985](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155459985.png)

![image-20220910155737268](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155737268.png)

![image-20220910155804625](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155804625.png)

![image-20220910155825004](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155825004.png)

![image-20220910155858699](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155858699.png)

![image-20220910155913500](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910155913500.png)

### 案例四：时间序列分析基本操作

![image-20220910160057579](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160057579.png)

![image-20220910160220793](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160220793.png)

![image-20220910160248981](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160248981.png)

![image-20220910160307208](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160307208.png)

![image-20220910160326250](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160326250.png)

![image-20220910160343621](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160343621.png)

![image-20220910160406536](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160406536.png)

![image-20220910160424457](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160424457.png)

![image-20220910160443872](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160443872.png)

![image-20220910160459933](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160459933.png)

![image-20220910160527957](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160527957.png)

![image-20220910160553785](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160553785.png)

![image-20220910160611693](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160611693.png)

![image-20220910160642823](https://note-01-1313694416.cos.ap-nanjing.myqcloud.com/image-20220910160642823.png)

**120题+50题**