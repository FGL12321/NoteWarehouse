[TOC]

# 第三章：python

## 3.1变量与运算符

### 案例一：变量与运算符

**根据以下题目进行练习，将结果写入桌面文件test.py中（自行创建）**

1.定义一个变量x，对其赋值为3，查看变量类型
2.对实数3和5进行相加、相乘。
3.创建一个列表[3,6,9],访问列表第1个元素。
4.对列表[1, 2, 3] 、[4, 5, 6]进行连接。
5.对集合{1, 2, 3}、{3, 2, 1} 进行比较，测试是否相等。

- 通过=可以为变量赋值
- type()函数查看变量类型
- 使用+、-、*、/进行变量间的运算
- 列表元素查看可以通过列表的索引，使用“+”号进行列表连接
- 使用==符号可以判断内容是否相等

```python
#测试1&2
#查看数据类型
x=3
y=5
z=x+y
a1=x*y 
type(a1)   

#测试3
#创建一个列表
#输出列表第一个元素
list=[1,2,3,4,5,6]  
list[0]             

#测试4
#连接两个链表
list1=[1,2,3,4,5,6]
list2=[7,8,9,10]
list1.extend(list2) 
list1+list2

#测试5：测试集合中元素是否相等
set1={1,2,3}
set2={1,3,2}
set1==set2
#true
```

## 3.2选择结构与循环结构实训

### 案例一：使用for循环输出满足条件的内容

编写程序，使用for循环输出50~99之间能被3整除但不能同时被5整除的所有整数。

```python
for i in range(50, 100):
    if i % 3 == 0 and i % 5 != 0:
        print(i)
```

### 案例二：使用while循环

编写程序，使用while循环输出50~99之间能被3整除但不能同时被5整除的所有整数。

```python
i=50
while i<=99:
    if i % 3 == 0 and i % 5 != 0:
        print(i)
    i+=1
```

## 3.3数据结构实战

### 案例一：列表

**任务说明：**

1.使用列表推导式查找列表中最大元素的所有位置

**代码编写：**

```python
#生成20个介于[1, 10]的整数
x = [randint(1, 10) for i in range(20)]

#上面的一行代码相当于
for i in range(20):
    print(randint(1,10))

#最大整数出现的位置
m = max(x)
[index for index, value in enumerate(x) if value == m]


#更好理解的一种形式
x = [randint(1, 10) for i in range(20)]
print(x)
m = max(x)

for index, value in enumerate(x):
    print(enumerate(x))
    if value == m:
        print(index)
```

**任务说明：**在列表推导式中同时遍历多个列表或可迭代对象。

**代码编写：**

```python
#第一种实现方式
[(x, y) for x in [1, 2, 3] for y in [3, 1, 4] if x != y]

#第二种实现方式
result=[]
for x in [1,2,3]:
    for y in[3,1,4]:
        if x!=y:
            result.append((x,y))
result
```

### 案例二：元组

Python 的元组与列表类似，不同之处在于元组的元素不能修改。

元组使用小括号，列表使用方括号。

元组创建很简单，只需要在括号中添加元素，并使用逗号隔开即可。

**任务说明：元组的创建与元素访问**

**代码编写：**

```python
>>>x = (1, 2, 3)       #直接把元组赋值给一个变量
>>>type(x)             #使用type()函数查看变量类型
<class  'tuple'>

>>>x[0]                #元组支持使用下标访问特定位置的元素
1

>>>x[-1]               #最后一个元素，元组也支持双向索引
3

>>>x = (3)             #这和x = 3是一样的
>>>x
3

>>> x = (3,)           #如果元组中只有一个元素，必须在后面多写一个逗号

>>>x
(3,)

>>>x = ()             #空元组
>>> x = tuple()        #空元组
>>> tuple(range(5))    #将其他迭代对象转换为元组
(0, 1, 2, 3, 4)
```

**任务说明：很多内置函数的返回值也是包含了若干元组的可迭代对象，例如enumerate()、zip()等等**

**代码编写：**

```python
>>> list(enumerate(range(5)))
[(0, 0), (1, 1), (2, 2), (3, 3),(4, 4)]

>>> list(zip(range(3), 'abcdefg'))
[(0, 'a'), (1, 'b'), (2, 'c')]


#关于zip的使用说明
my_list = [11,12,13]
my_tuple = (21,22,23)
print(list(zip(my_list,my_tuple)))
```

**任务说明：使用生成器对象__next__()方法或内置函数next()进行遍历**

**代码编写：**

```python
>>> g = ((i+2)**2 for i in range(10))  #创建生成器对象
>>> g
<generator object <genexpr> at 0x0000000003095200>

>>> tuple(g)                           #将生成器对象转换为元组
(4, 9, 16, 25, 36, 49, 64, 81, 100, 121)

>>> list(g)             #生成器对象已遍历结束，没有元素了
[] 

>>> g = ((i+2)**2 for i in range(10))  #重新创建生成器对象
>>> g.__next__()        #使用生成器对象的__next__()方法获取元素
4

>>> g.__next__()        #获取下一个元素
9

>>> next(g)             #使用函数next()获取生成器对象中的元素
16
```

**任务说明：使用for循环直接迭代生成器对象中的元素**

**代码编写：**

```python
g = ((i+2)**2 for i in range(10))
for item in g:               
#使用循环直接遍历生成器对象中的元素
    print(item, end=' ')
    
#4 9 16 25 36 49 64 81 100 121
```

### 案例三：字典

​		字典（dictionary）是包含若干“键:值”元素的无序可变序列，字典中的每个元素**包含用冒号分隔开的“键”和“值”两部分**，表示一种映射或对应关系，也称**关联数组**。定义字典时，每个元素的“键”和“值”之间用冒号分隔，不同元素之间用逗号分隔，所有的元素放在一对大括号“｛｝”中。
​		字典中元素的“键”可以是Python中**任意不可变数据，例如整数、实数、复数、字符串、元组等类型等可哈希数据，但不能使用列表、集合、字典或其他可变类型作为字典的“键”**。
字典中的**“键”不允许重复，“值”是可以重复的。**

**任务说明：定义字典**

**代码编写：**

```python
>>> x = dict() # 空字典
>>> type(x)    # 查看对象类型
>>> <class 'dict'>
>>> x = {}     # 空字典
```

**任务说明：创建字典**

**代码编写：**

```
>>> aDict = {'server':'db.diveintopython3.org', 'database':'mysql'}
>>> aDict
{'server': 'db.diveintopython3.org', 'database': 'mysql'}
```

==字典中内容没有顺序，所以查看字典内容时顺序可能不同==

```python
#使用内置类dict以不同形式创建字典
keys = ['a','b','c','d']
values = [1, 2, 3, 4]
dictionary = dict(zip(keys, values)) # 根据已有数据创建字典
dictionary

d = dict(name='Dong', age=39) # 以关键参数的形式创建字典
d

aDict = dict.fromkeys(['name','age','sex']) # 以给定内容为“键”，创建“值”为空的字典
aDict
```

**任务说明：使用“键”访问字典元素的“值”**

**代码编写：**

```
aDict = {'age':39,'score':[98,97],'name':'Dong','sex':'male'}
aDict['age'] # 指定的“键”存在，返回对应的“值”
39

aDict['address'] # 指定的“键”不存在，抛出异常
KeyError: 'address'
```

### 案例四：集合

​		**集合（set）**属于Python**无序可变序列**，使用一对大括号作为定界符，元素之间使用逗号分隔，同一个集合内的每个元素都是唯一的，**元素之间不允许重复**。
​		集合中只能包含**数字、字符串、元组**等不可变类型（或者说可哈希）的数据，而不能包含列表、字典、集合等可变类型的数据。

**任务说明：创建集合**

**代码编写：**

```python
>>> a = {3,5} # 创建集合对象
>>> a_set = set(range(8,14)) # 把range对象转换为集合
>>> a_set
{8, 9, 10, 11, 12, 13}
>>> b_set = set([0,1,2,3,0,1,2,3,7,8]) # 转换时自动去掉重复元素
>>> b_set
{0, 1, 2, 3, 7, 8}
>>> x = set() # 空集合
```

**任务说明：想集合中添加元素**

**代码编写：**

```python
>>> s = {1,2,3}
>>> s.add(3) # 添加元素，重复元素自动忽略
>>> s
{1, 2, 3}
>>> s.update({3,4}) # 更新当前集合，自动忽略重复的元素
>>> s
{1, 2, 3, 4}
```



## 3.4函数实训

### 案例一：斐波那契数列计算

**任务说明：**

1. 结合生成器函数yield和while循环实现15项斐波那契数的计算。
2. 通过__next__() 方法获取内容，使用print方法将每一项之间通过空格隔开，不换行展示。
3. 将代码实现的过程存储到sequences.py文件中，将代码文件保存在“/home/qingjiao/Desktop”目录中。

学习相关知识点

- yield语句与return语句的作用相似，都是用来从函数中返回值。
- 每次执行到**yield语句并返回一个值之后会暂停或挂起后面代码的执行**。
- 可以通过生成器对象的__next__()方法、内置函数next()、for循环遍历生成器对象元素或其他方式显式“索要”数据时恢复执行。
- print()函数中，使用“end = ' ' ” 可以设置输出的多个内容在同一行展示，而且每个内容通过空格隔开。

**代码编写：**

```python
def Fab(n):
 
    """
    使用yield来返回值；用非递归来实现斐波那契数列，使用yield来返回值，
    """
    a, b, m = 0, 1, 0
    i = 1
    while m < n:
        yield b
        a, b, m = b, a+b, m+1
        i += 1
 
 
t = Fab(15)
num = 1
for i in t:
    print(i,end=" ")
    num += 1
    
#运行结果：1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 


#更简单的方法
a = 0
b = 1
m = 0
while m < 15:
    print(b)
    a, b, m = b, a + b, m + 1
```

## 3.5文件处理I/O操作实训

**任务说明：向文本文件中写入内容，然后再读出。**

```python
s = 'Hello world\n文本文件的读取方法\n文本文件的写入方法\n'
with open('sample.txt', 'w') as fp: #默认使用cp936编码
 fp.write(s)

with open('sample.txt') as fp: #默认使用cp936编码
 print(fp.read())
 
#运行结果
Hello world
文本文件的读取方法
文本文件的写入方法
```

**任务说明：遍历并输出文本文件的所有行内容。**

```python
with open('sample.txt') as fp: #假设文件采用CP936编码
    for line in fp:            #文件对象可以直接迭代
        print(line)
        
#运行结果
Hello world

文本文件的读取方法

文本文件的写入方法
```

**任务说明：假设文件data.txt中有若干整数，每行一个整数，编写程序读取所有整数，将其按降 序排序后再写入文本文件data_asc.txt中。**

```python
s = '23\n24\n50\n243\n546\n234\n'
with open('date.txt','w')as we:
    we.write(s)
data = [int(item) for item in data]      #列表推导式，转换为数字
data.sort(reverse=True)                  #降序排序
data = [str(item)+'\n' for item in data] #将结果转换为字符串
with open('data_desc.txt', 'w') as fp:   #将结果写入文件
    qwer = fp.writelines(date)
```

**任务说明：统计文本文件中最长行的长度和该行的内容。**

```python
with open('sample.txt') as fp:
    result = [0, '']
    for line in fp:
        t = len(line)
        if t > result[0]:
            result = [t, line]
print(result)
```

**任务说明：使用pick模块写入二进制文件。**

```python
import  pickle

i = 13000000
a = 99.056
s = '中国人民 123abc'
lst = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
tu = (-5, 10, 8)
coll = {4, 5, 6}
dic = {'a':'apple', 'b':'banana', 'g':'grape', 'o':'orange'}
data = (i, a, s, lst, tu, coll, dic)
with open('sample_pickle.dat', 'wb') as f:
   try: 
      pickle.dump(len(data), f)         #要序列化的对象个数
      for item in data:
           pickle.dump(item, f)         #序列化数据并写入文件
   except:
       print('写文件异常')
```

**任务说明：使用pickle模块读上例中二进制文件的内容。**

```python
import pickle
with open('sample_pickle.dat', 'rb') as f:
   n = pickle.load(f)                     #读出文件中的数据个数
   for i in range(n):
    x = pickle.load(f)                   #读取并反序列化每个数据
       print(x)
```

**任务说明：使用struct模块写入二进制文件。**

```python
import struct

n = 1300000000
x = 96.45
b = True
s = 'a1@中国'
sn = struct.pack('if?', n, x, b)  #序列化，i表示整数，f表示实数，?表示逻辑值
with open('sample_struct.dat', 'wb') as f:
   f.write(sn)
   f.write(s.encode())
```

**任务说明：使用struct模块读取上例中二进制文件的内容**

```python
import struct

with open('sample_struct.dat', 'rb') as f:
     sn = f.read(9)
     n, x, b1 = struct.unpack('if?', sn)   #使用指定格式反序列化
     print('n=',n, 'x=',x, 'b1=',b1)
     s = f.read(9).decode()
     print('s=', s)
```

**任务说明：使用扩展库openpyxl读写Excel 2007以及更高版本的文件**

```python
import openpyxl
from openpyxl import Workbook

fn = r'test.xlsx'                                #文件名
wb = Workbook()                                  #创建工作簿
ws = wb.create_sheet(title='你好，世界')          #创建工作表
ws['A1'] = '这是第一个单元格'                      #单元格赋值
ws['B1'] = 3.1415926
wb.save(fn)                                      #保存Excel文件
wb = openpyxl.load_workbook(fn)                  #打开已有的Excel文件
ws = wb.worksheets[1]                            #打开指定索引的工作表
print(ws['A1'].value)                            #读取并输出指定单元格的值
ws.append([1,2,3,4,5])                           #添加一行数据
ws.merge_cells('F2:F3')                          #合并单元格
ws['F2'] = "=sum(A2:E2)"                         #写入公式
for r in range(10,15):
   for c in range(3,8):
       ws.cell(row=r, column=c, value=r*c)       #写入单元格数据
wb.save(fn)
```

**任务说明：把记事本文件test.txt转换成Excel 2007+文件。假设test.txt文件中第一行为表头，从第二行开始是实际数据，并且表头和数据行中的不同字段信息都是用逗号分隔。**

```python
def main(txtFileName):
   new_XlsxFileName = txtFileName[:-3] + 'xlsx'
   wb = Workbook()
   ws = wb.worksheets[0]
   with open(txtFileName) as fp:
       for line in fp:
           line = line.strip().split(',')
           ws.append(line)
   wb.save(new_XlsxFileName)
main('test.txt')
```

**任务说明：输出Excel文件中单元格中公式计算结果**

```python
import openpyxl
#打开Excel文件
wb = openpyxl.load_workbook('data.xlsx', data_only=True)

 #获取WorkSheet
ws = wb.worksheets[1]

 #遍历Excel文件所有行，假设下标为3的列中是公式
for row in ws.rows:
   print(row[3].value)
```



## 3.5python概述

### 案例一：Matplotlib图表初体验

```python
import matplotlib.pyplot as plt

x = ["1-3", "4-6", "7-9", "10-12"] # 设置x轴数值
y = [257, 301, 428, 475] # 设置y轴数值
plt.plot(x, y)
plt.grid(True, linestyle="--", alpha=0.5)  # (是否设置网格，设置为虚线，设置网格深浅度)
plt.show()

```

![image-20220909200607621](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220909200607621.png)

## 3.6Numpy数据计算

### 案例一：一维数组的创建、索引及切片

**任务说明：**

（1）NumPy提供的array函数可以创建一维数组或多维数组。使用array函数把如下图示的员工的年龄创建为一维数组

（2）使用数组的索引，分别获取“Mary”和“LiLi”的年龄
（3）对数组进行分割，同时获取“Mary”和“LiLi”的年龄、“LiLi”和“Cendy”的年龄以及“Mary”和“Cendy”的年龄

**代码实现：**

```python
#（1）创建出数组
import numpy as np
score=np.array([["Mary",22,95.5],["LiLi",23,56],["Cendy",22,90]])
score

#（2）对数据进行切片（先行后列）
score[:2,1]

#（3）数组分割
```

### 案例二：Numpy常用的函数

**任务说明：**

**（1）算术函数**
使用NumPy 算术函数add()、subtract()、multiply()、divide()实现如下两个数组之间的加减乘除。

**代码实现：**

```python
import numpy as np
n1 = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
n2 = np.array([1, 2, 3])  # 一维数组

n3=np.add(n1,n2)
n4=np.subtract(n1,n2)
n5=np.divide(n1,n2)
```

**(2)排序函数**

```python
import numpy as np
dt = np.dtype([('name',"S10"),('age',int),('KPI',int)]) 
n1=np.array([('Mary', 22, 95.5), ('LiLi', 23, 56), ('Cendy', 22, 90)],dtype = dt)
n1.sort(axis=0,order="KPI") # 对数组进行排序
n1
```

![image-20220910155536867](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910155536867.png)

## 3.7Pandas数据分析

### 案例一：创建DataFrame)对象

**任务说明：以员工薪资构成为例，其包含基本薪资、绩效薪资和补贴，数据如下：**

![image-20220909202502919](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220909202502919.png)

**代码实现：**

```python
import pandas as pd

data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df
```

![image-20220910101617640](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910101617640.png)

```python
import numpy as np
arr = np.array([4, 7, 1, 6, 1, 8, 4, 9, 1, 6]) 
arr1=np.unique(arr)
arr1

#array([1, 4, 6, 7, 8, 9])
```

### 案例二:   数据抽取、增加、修改及删除

**任务说明：**

![image-20220910095232582](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910095232582.png)

![image-20220910095245980](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910095245980.png)

**代码实现：**

```python
#（1）使用loc属性和iloc属性抽取第一行数据

import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[0]
```

![image-20220910101645453](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910101645453.png)

```python
#（2）增加奖金列方式一：直接赋值法
import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df['奖金']=['122','222','333']
df

```

![image-20220910101551447](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910101551447.png)

```python
#（2）增加奖金列方式二：loc属性法
import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[:,'奖金']=['122','222','333']
df
```

```python
#（2）增加奖金列方式三：指定位置插入
import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.insert(loc=2, column='奖金', value=100)  # 在最后一列后，插入值全为3的c列
df

```

```python
#（3）增加行数据：在指定行，加入指定数据
import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.insert(loc=2, column='奖金', value=100)  # 在最后一列后，插入值全为3的c列
df.loc[3]=['Alice','1000','1000','1000','100']
df
```

![image-20220910102908964](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910102908964.png)

```python
（3）增加多行数据
import pandas as pd
data=[['Mary',"2500","5800","500"],["LiLi","2500","7800","650"],["Gendy","2500","10500","200"]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.insert(loc=2, column='奖金', value=100)      # 在最后一列后，插入值全为3的c列
df.loc[3]=['Alice','1000','1000','1000','100']  # 在指定行添加数据
data1=[['Liming',"2500","5800","500"],["LiLi1","2500","7800","650"],["Gendy1","2500","10500","200"]]
df1=pd.DataFrame(data1,columns=['姓名','基本薪资','绩效津贴','补贴'])
df1

df.append(df1,ignore_index=True)#让索引重写进行排序

```

![image-20220910141136422](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910141136422.png)

### 案例三：数据缺失值处理

**任务说明：**

**1）删除缺失值**
使用 dropna 函数删除含有缺失值的行

**2）填充缺失值**
DataFrame 对象中的 fillna 函数可以实现填充缺失数据

**代码实现：**

```python
#（1）查看缺失值

import pandas as pd
df = pd.read_csv('property-data.csv')
df.info()
```

![image-20220910142740215](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910142740215.png)

```python
#(2)删除含有缺失值的行

import pandas as pd
df = pd.read_csv('property-data.csv')
new_df=df.dropna()
new_df
```

默认情况下，dropna() 方法返回一个新的 DataFrame，不会修改源数据。

![image-20220910143029301](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910143029301.png)

```python
#（3）对空值进行填充替代

import pandas as pd
df = pd.read_csv('property-data.csv')
df.fillna(12345, inplace = True)
print(df.to_string())
```

![image-20220910143143918](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910143143918.png)

### 案例四：数据的计算函数

**任务说明：**

![image-20220910144313821](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910144313821.png)

**代码实现：**

```python
#(1)任务1：使用 DataFrame 对象的 sum 函数实现行/列数据的求和运算，效果如下：
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df['奖金']=['122','222','333']
df.loc[:,'总薪资']=df.sum(axis=1)
df
```

![image-20220910144401951](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910144401951.png)

```python
#（2）任务2：每行的平均数
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df['奖金']=['122','222','333']
df.loc[:,'总薪资']=df.mean(axis=1)
df
```

![image-20220910145403512](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910145403512.png)

```python
#该行最大值
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df['奖金']=['122','222','333']
df.loc[:,'总薪资']=df.max(axis=1)
df

#该行最小值
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df['奖金']=['122','222','333']
df.loc[:,'总薪资']=df.max(axis=0)
df
```

**下面的内容为每一列的求和与计算：**

```python
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[3]=df.iloc[:,1:5].max(axis=0)#求每一列的最大值
df
```

![image-20220910151357567](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910151357567.png)

```python
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[3]=df.iloc[:,1:5].mean(axis=0)#求出每一列的平均值
df
```

![image-20220910151350706](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910151350706.png)

```python
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[3]=df.iloc[:,1:5].min(axis=0)#求出每一列的最小值
df
```

![image-20220910151559518](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910151559518.png)

```python
import pandas as pd
data=[['Mary',2500,5800,500],["LiLi",2500,7800,650],["Gendy",2500,10500,200]]
df=pd.DataFrame(data,columns=['姓名','基本薪资','绩效津贴','补贴'])
df.loc[3]=df.iloc[:,1:5].sum(axis=0) #求出每一列的和
df
```

![image-20220910151829847](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910151829847.png)

## 3.8Matplotlib数据可视化

```python
import matplotlib.pyplot as plt 
import random # 画出温度变化图 



# 0.准备x, y坐标的数据 
x = range(60) 
y_shanghai = [random.uniform(15, 18) for i in x] 

# 创建画布
plt.figure(figsize=(20, 8), dpi=80) 

# 绘制折线图 
plt.plot(x, y_shanghai) 

#添加网格
plt.grid(True, linestyle='--', alpha=1)

# 构造x轴刻度标签 
x_ticks_label = ["11h{}min".format(i) for i in x] 

# 构造y轴刻度 
y_ticks = range(40) 

# 修改x,y轴坐标的刻度显示 
plt.xticks(x[::5], x_ticks_label[::5]) 

plt.yticks(y_ticks[::5])

# 增加北京的温度数据 
y_beijing = [random.uniform(1, 3) for i in x] 
# 绘制折线图 
plt.plot(x, y_shanghai) 
# 使用多次plot可以画多个折线 
plt.plot(x, y_beijing, color='r', linestyle='--')

plt.xlabel("time") 
plt.ylabel("temperature") 
plt.title("temperature variation", fontsize=20)

# 显示图像 
plt.show()
```

![image-20220910163852888](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220910163852888.png)
