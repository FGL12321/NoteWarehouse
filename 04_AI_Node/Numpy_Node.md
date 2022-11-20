[TOC]

# Numpy

# 1.0Numpy优势

![image-20220901150946022](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901150946022.png)

## 1.1 Numpy介绍

​		**Numpy（Numerical Python）是一个开源的Python科学计算库，用于快速处理任意维度的数组。** 

​		**Numpy支持常见的数组和矩阵操作。对于同样的数值计算任务，使用Numpy比直接使用Python要简  洁的多。** 

​		**Numpy使用ndarray对象来处理多维数组，该对象是一个快速而灵活的大数据容器。**

## 1.2 ndarray介绍

NumPy提供了一个**N**维数组类型**ndarray**，它描述了相同类型的“items”的集合。

用ndarray进行存储： 

```
import numpy as np # 创建ndarray 
score = np.array([[80, 89, 86, 67, 79], 
                    [78, 97, 89, 67, 81], 
                    [90, 94, 78, 67, 74],
                    [91, 91, 90, 67, 69],
                    [76, 87, 75, 67, 86],
                    [70, 79, 84, 67, 84],
                    [94, 92, 93, 67, 64], 
                    [86, 85, 83, 67, 80]])
```

![image-20220901152331825](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901152331825.png)

## 1.3 ndarray**与**Python原生list运算效率对比 

```python
import random 
import time import numpy as np 
a = [] 
for i in range(100000000): 
	a.append(random.random())
# 通过%time魔法方法, 查看当前行的代码运行一次所花费的时间
%time sum1=sum(a) 
b=np.array(a) 
%time sum2=np.sum(b)
```

​		从中我们看到ndarray的计算速度要快很多，节约了时间。 

​		机器学习的最大特点就是大量的数据运算，那么如果没有一个快速的解决方案，那可能现在python也在机器学习领域达不到好的效果。 

## 1.4 ndarray的优势

![image-20220901153042061](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901153042061.png)

​		**从图中我们可以看出ndarray在存储数据的时候，数据与数据的地址都是连续的，这样就给使得批量操作数组元素时速度更快**

### 1.4.1  内存块风格

​		这是因为ndarray中的所有元素的类型都是相同的，而Python列表中的元素类型是任意的，所以ndarray在存储元素时内存可以连续，而python 原生list就只能通过寻址方式找到下一个元素，这虽然也导致了在通用性能方面Numpy的ndarray不及Python原生list，但在科学计算中，Numpy 的ndarray就可以省掉很多循环语句，代码使用方面比Python原生list简单的多。

### 1.4.2  ndarray支持并行化运算（向量化运算）

numpy内置了并行运算功能，当系统有多个核心时，做某种计算时，numpy会自动做并行计算

### 1.4.3  效率远高于纯**Python**代码

​		Numpy底层使用C语言编写，内部解除了GIL（全局解释器锁），其对数组的操作速度不受Python解释器的限制，所以，其效率远高于纯Python代码。 

# 2.0 N维数组-ndarray

## 2.1ndarray属性

**数组属性反映了数组本身固有的信息。**

| 属性名字         | 属性解释           |
| ---------------- | ------------------ |
| ndarray.shape    | 数组维度的元组     |
| ndarray.ndim     | 数组维度           |
| ndarray.size     | 数组中的元素数量   |
| ndarray.itemsize | 一个数组元素的长度 |
| ndarray.dtype    | 数组元素的类型     |

## 2.2ndarray形状

**二维数组：**

![image-20220901154143087](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901154143087.png)

**三维数组：**

![](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901154217277.png)





```
a = np.array([[1,2,3],[4,5,6]]) 

b = np.array([1,2,3,4]) 

c = np.array([[[1,2,3],[4,5,6]],
			 [[1,2,3],[4,5,6]]]) 
```

![image-20220901154117432](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901154117432.png)

## 2.3ndarray类型

![image-20220901155108468](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901155108468.png)

```
a = np.array([[1, 2, 3],[4, 5, 6]], dtype=np.float32) 

arr = np.array(['python', 'tensorflow', 'scikit-learn', 'numpy'], dtype = np.string_) 
```

![image-20220901155513174](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901155513174.png)

# 3.0基本操作

## 3.1生成数组的方法

### 3.1.1 生成**0**和**1**的数组 

```python
ones = np.ones([4,8]) #生成默认值为1的二维数组
np.zeros_like(ones)	  #生成默认值为0的二维数组
```

![image-20220901162000207](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901162000207.png)



### 3.1.2 从现有数组生成

#### 3.1.2.1 生成方式

```
a = np.array([[1,2,3],[4,5,6]]) # 从现有的数组当中创建 
a1 = np.array(a) # 相当于索引的形式，并没有真正的创建一个新的 
a2 = np.asarray(a)
```

![image-20220901162324512](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220901162324512.png)

#### 3.1.2.2  关于array和asarray的不同

​		array和asarray都可以将结构数据转化为ndarray，但是主要区别就是当数据源是ndarray时，array仍然会copy出一个副本，占用新的[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)，但asarray不会。

```python

import numpy as np
 
#example 1:
data1=[[1,1,1],[1,1,1],[1,1,1]]
arr2=np.array(data1)
arr3=np.asarray(data1)
print('data1:\n',data1)
print('arr2:\n',arr2)
print('arr3:\n',arr3) 
```

```
#运行结果：说明两者都进行了复制
data1:
[[1, 1, 1], [1, 2, 1], [1, 1, 1]]
arr2:
[[1 1 1]
 [1 1 1]
 [1 1 1]]
arr3:
[[1 1 1]
 [1 1 1]
 [1 1 1]]
```

此时数据进行修改

```python
import numpy as np
 
#example 2:
arr1=np.ones((3,3))
arr2=np.array(arr1)
arr3=np.asarray(arr1)
arr1[1]=2
print('arr1:\n',arr1)
print('arr2:\n',arr2) 
print('arr3:\n',arr3) 
```

```python
#asarray的值也发生了改变
arr1:
[[ 1.  1.  1.]
 [ 2.  2.  2.]
 [ 1.  1.  1.]]
arr2:
[[ 1.  1.  1.]
 [ 1.  1.  1.]
 [ 1.  1.  1.]]
arr3:
[[ 1.  1.  1.]
 [ 2.  2.  2.]
 [ 1.  1.  1.]]
```

### 3.1.3 生成固定范围的数组

#### 3.1.3.1 np.linspace (start, stop, num, endpoint)

- 创建等差数组 — 指定数量 
- 参数:
  - start:序列的起始值 
  - stop:序列的终止值 
  - num:要生成的等间隔样例数量，默认为50 
  - endpoint:序列中是否包含stop值，默认为ture 

```
# 生成等间隔的数组 
np.linspace(0, 100, 11)
```

**返回值：**

```
array([  0.,  10.,  20.,  30.,  40.,  50.,  60.,  70.,  80.,  90., 100.])
```

#### 3.1.3.2 np.arange(start,stop, step, dtype)

- 创建等差数组 — 指定步长 
  - 参数
  - step:步长,默认值为1

```
np.arange(10, 50, 2)
```

![image-20220903124842128](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903124842128.png)

#### 3.1.3.3 np.logspace(start,stop, num)

- 创建等比数列 
  - 参数:
  - num:要生成的等比数列数量，默认为50

```
# 生成10^x 
np.logspace(0, 2, 3)
```

![image-20220903125026467](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903125026467.png)

### 3.1.4生成随机数组

#### 3.1.4.1 使用模块介绍

- np.random模块

#### 3.1.4.2正态分布

**一、基础概念复习：正态分布（理解）** 

**a.什么是正态分布** 

​		正态分布是一种概率分布。正态分布是具有两个参数μ和σ的连续型随机变量的分布，第一参数μ是服从正态分布的随机变量的均值，第二个参 数σ是此随机变量的标准差，所以正态分布记作N(μ，σ )。

**b. 正态分布的应用** 

​		生活、生产与科学实验中很多随机变量的概率分布都可以近似地用正态分布来描述。

**c. 正态分布特点** 

​		μ决定了其位置，其标准差σ决定了分布的幅度。当μ = 0,σ = 1时的正态分布是标准正态分布。 标准差如何来？

![image-20220903125653099](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903125653099.png)

二、正态分布创建方式 

- **np.random.randn(d0, d1, …, dn)** 

功能：从标准正态分布中返回一个或多个样本值 

- **np.random.normal(loc=0.0, scale=1.0, size=None)** 

loc：float 

此概率分布的均值（对应着整个分布的中心centre） 

scale：float 

此概率分布的标准差（对应于分布的宽度，scale越大越矮胖，scale越小，越瘦高） 

size：int or tuple of ints 

输出的shape，默认为None，只输出一个值 



- **np.random.standard_normal(size=None)** 

返回指定形状的标准正态分布的数组。 

**举例1：生成均值为1.75，标准差为1的正态分布数据，100000000个** 

```
x1 = np.random.normal(1.75, 1, 100000000) 
```

**返回结果：** 

```
#生成均匀分布的随机数 x1 = np.random.normal(1.75, 1, 100000000) 
# 画图看分布状况 

# 1）创建画布 
plt.figure(figsize=(20, 10), dpi=100) 

# 2）绘制直方图 
plt.hist(x1, 1000) 

# 3）显示图像 
plt.show()
```

![image-20220903130915945](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903130915945.png)

**举例2：随机生成4支股票1周的交易日涨幅数据 4支股票，一周(5天)的涨跌幅数据，如何获取？ 随机生成涨跌幅在某个正态分布内，比如均值0，方差1 股票涨跌幅数据的创建**

```
# 创建符合正态分布的4只股票5天的涨跌幅数据 
stock_change = np.random.normal(0, 1, (4, 5)) 

stock_change
```

![image-20220903131122886](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903131122886.png)

#### 3.1.4.3 均匀分布

- **np.random.rand(d0, d1, ..., dn)** 
  - 返回[0.0，1.0)内的一组均匀分布的数。 
- **np.random.uniform(low=0.0, high=1.0, size=None)** 
  - 功能：从一个均匀分布[low,high)中随机采样，注意定义域是左闭右开，即包含low，不包含high. 
  - 参数介绍: 
    - low: 采样下界，
    - float类型，默认值为0； 
    - high: 采样上界，float类型，默认值为1； 
    - size: 输出样本数目，为int或元组(tuple)类型，例如，size=(m,n,k), 则输出mnk个样本，缺省时输出1个值。 
  - 返回值：ndarray类型，其形状和参数size中描述一致。
- **np.random.randint(low, high=None, size=None, dtype='l')** 
  - 从一个均匀分布中随机采样，生成一个整数或N维整数数组，
  -  取数范围：若high不为None时，取[low,high)之间随机整数，否则取值[0,low)之间随机整数。

```python
import matplotlib.pyplot as plt 
# 生成均匀分布的随机数 
x2 = np.random.uniform(-1, 1, 10000) 

# 画图看分布状况 
# 1）创建画布 
plt.figure(figsize=(10, 10), dpi=100) 

# 2）绘制直方图 
plt.hist(x=x2, bins=1000) 
# x代表要使用的数据，bins表示要划分区间数 

# 3）显示图像 
plt.show()
```

![image-20220903131559653](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903131559653.png)

## 3.2 数组的索引、切片

**一维、二维、三维的数组如何索引？** 

- 直接进行索引,切片 
- 对象[:, :] -- 先行后列 

**二维数组索引方式：** 

**举例：获取第一个股票的前3个交易日的涨跌幅数据** 

```
# 二维的数组，两个维度 
stock_change[0, 0:3] 
```

**返回结果：** 

```
array([-0.03862668, -1.46128096, -0.75596237]) 
```

**三维数组索引方式：** 

```python
# 三维 

a1 = np.array([ [[1,2,3],[4,5,6]], [[12,3,34],[5,6,7]]]) 

# 返回结果 

array([[[ 1, 2, 3], 
	    [ 4, 5, 6]], 
	   [[12, 3, 34], 
	    [ 5, 6, 7]]]) 

# 索引、切片 >>> a1[0, 0, 1] # 输出: 2
```

# 4.0 ndarray运算

## 4.1 逻辑运算

```python
#生成10名同学，5门功课的数据
score=np.random.randint(40,100,(10,5))
#取出最后4名同学的成绩，用于逻辑判断
test_score=score[6:,0:5]
#逻辑判断,如果成绩大于60就标记为True否则为False 
test_score>60
```

## 4.2 通用判断函数

**np.all()**

```python
#判断前两名同学的成绩[0:2,:]是否全及格

np.all(score[0:2,:]>60)

False
```

**np.any()**

```python
#判断前两名同学的成绩[0:2,:]是否有大于90分的

np.any(score[0:2,:]>80)

True
```

## 4.3 np.where（三元运算符） 

**np.where()**

复合逻辑需要结合np.logical_and和np.logical_or使用

```python
#判断前四名学生,前四门课程中，成绩中大于60且小于90的换为1，否则为0 
np.where(np.logical_and(temp>60,temp<90),1,0)

#判断前四名学生,前四门课程中，成绩中大于90或小于60的换为1，否则为0 
np.where(np.logical_or(temp>90,temp<60),1,0)
```



## 4.4 统计运算

```python
#接下来对于前四名学生,进行一些统计运算

#指定列去统计
temp=score[:4,0:5]

print("前四名学生,各科成绩的最大分：{}".format(np.max(temp,axis=0)))

print("前四名学生,各科成绩的最小分：{}".format(np.min(temp,axis=0)))

print("前四名学生,各科成绩波动情况：{}".format(np.std(temp,axis=0)))

print("前四名学生,各科成绩的平均分：{}".format(np.mean(temp,axis=0)))
```



# 5.0数组间运算

## 5.1数组与数运算

```python
arr = np.array([[1, 2, 3, 2, 1, 4], [5, 6, 1, 2, 3, 1]]) 
arr + 1 
arr / 2 

# 可以对比python列表的运算，看出区别 
a = [1, 2, 3, 4, 5] 
a * 3
```

```python
arr1 = np.array([[1, 2, 3, 2, 1, 4], [5, 6, 1, 2, 3, 1]]) 

arr2 = np.array([[1, 2, 3, 4], [3, 4, 5, 6]])
```

==上面这个能进行运算吗，结果是不行的！==

## 5.2数组与数组运算

```python
arr1 = np.array([[0],[1],[2],[3]]) 
arr1.shape 
# (4, 1) 
arr2 = np.array([1,2,3]) 
arr2.shape 
# (3,) 


arr1+arr2 
# 结果是： 
array([[1, 2, 3], 
	   [2, 3, 4], 
	   [3, 4, 5], 
	   [4, 5, 6]])
```

上述代码中，数组arr1是4行1列，arr2是1行3列。这两个数组要进行相加，按照广播机制会对数组arr1和arr2都进行扩展，使得数组arr1和arr2 都变成4行3列。 



# 6.0矩阵



## 6.1矩阵和向量

![image-20220903135037155](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135037155.png)

## 6.2加法和标量乘法

![image-20220903135048201](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135048201.png)

## 6.3矩阵向量乘法

![image-20220903135131887](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135131887.png)

## 6.4矩阵乘法

![image-20220903135145811](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135145811.png)

## 6.5矩阵乘法的性质

![image-20220903135211645](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135211645.png)

## 6.6逆，转置

![image-20220903135239523](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135239523.png)

## 6.7矩阵运算

- np.matmul 
- np.dot 

![image-20220903135249356](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903135249356.png)

```python
>>> a = np.array([[80, 86], 
				  [82, 80], 
				  [85, 78], 
				  [90, 90], 
				  [86, 82], 
				  [82, 90], 
				  [78, 80], 
				  [92, 94]]) 
				  
b = np.array([[0.7], [0.3]]) 

np.matmul(a, b) 
array([[81.8], [81.4], [82.9], [90. ], [84.8], [84.4], [78.6], [92.6]]) 

np.dot(a,b) 
array([[81.8], [81.4], [82.9], [90. ], [84.8], [84.4], [78.6], [92.6]])
```





