[TOC]



# Matplotlib_Node

# 1.Matplotlib初识

![image-20220903141445945](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903141445945.png)

**是专门用于开发2D图表(包括3D图表)** 

**以渐进、交互式方式实现数据可视化** 



可视化是在整个数据挖掘的关键辅助工具，可以清晰的理解数据，从而调整我们的分析方法。 

能将数据进行可视化,更直观的呈现 

使数据更加客观、更具说服力 

例如下面两个图为数字展示和图形展示：

# 2.基础绘图功能 **—** 以折线图为例

```python
import matplotlib.pyplot as plt 
import random # 画出温度变化图 

# 0.准备x, y坐标的数据 
x = range(60) 
y_shanghai = [random.uniform(15, 18) for i in x] 

# 1.创建画布 
plt.figure(figsize=(20, 8), dpi=80) 

# 2.绘制折线图 
plt.plot(x, y_shanghai) 

# 3.显示图像 
plt.show()
```

![image-20220903143006415](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903143006415.png)

## 2.1添加自定义信息

### 2.1.1自定义刻度

plt.xticks(x, kwargs) 

x:要显示的刻度值 

plt.yticks(y, kwargs) 

y:要显示的刻度值

 增加以下两行代码

```python
import matplotlib.pyplot as plt 
import random # 画出温度变化图 


# 0.准备x, y坐标的数据 
x = range(60) 
y_shanghai = [random.uniform(15, 18) for i in x] 


# 1.创建画布
plt.figure(figsize=(20, 8), dpi=80) 

# 2.绘制折线图 
plt.plot(x, y_shanghai) 

# 3.构造x轴刻度标签 
x_ticks_label = ["11点{}分".format(i) for i in x] 
# 构造y轴刻度 
y_ticks = range(40) 
# 4.修改x,y轴坐标的刻度显示 
plt.xticks(x[::5], x_ticks_label[::5]) 
plt.yticks(y_ticks[::5])

# 5.显示图像 
plt.show()
```

![image-20220903144901846](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903144901846.png)

### 2.1.2添加网格

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
x_ticks_label = ["11点{}分".format(i) for i in x] 

# 构造y轴刻度 
y_ticks = range(40) 

# 修改x,y轴坐标的刻度显示 
plt.xticks(x[::5], x_ticks_label[::5]) 

plt.yticks(y_ticks[::5])

# 显示图像 
plt.show()
```

![image-20220903145911794](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903145911794.png)

### 2.1.3 添加标题和列说明

```
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
x_ticks_label = ["11点{}分".format(i) for i in x] 

# 构造y轴刻度 
y_ticks = range(40) 

# 修改x,y轴坐标的刻度显示 
plt.xticks(x[::5], x_ticks_label[::5]) 

plt.yticks(y_ticks[::5])

plt.xlabel("时间") 
plt.ylabel("温度") 
plt.title("中午11点0分到12点之间的温度变化图示", fontsize=20)

# 显示图像 
plt.show()
```

![image-20220903150029189](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903150029189.png)

### 2.1.4图像保存

```
# 2.4 图像保存 
plt.savefig("./test.png") 
```

## 2.2一个图像添加多个图像

### 2.2.1多次plot

**需求：再添加一个城市的温度变化** 

收集到北京当天温度变化情况，温度在1度到3度。怎么去添加另一个在同一坐标系当中的不同图形，其实很简单只需要再次plot即可，但是需 

要区分线条，如下显示 ：

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
x_ticks_label = ["11点{}分".format(i) for i in x] 

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

plt.xlabel("时间") 
plt.ylabel("温度") 
plt.title("中午11点0分到12点之间的温度变化图示", fontsize=20)

# 显示图像 
plt.show()
```

![image-20220903150808248](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903150808248.png)

## 2.2.2绘制子图

```
import matplotlib.pyplot as plt
import numpy as np

#plot 1:
xpoints = np.array([0, 6])
ypoints = np.array([0, 100])

plt.subplot(1, 2, 1)
plt.plot(xpoints,ypoints)
plt.title("plot 1")

#plot 2:
x = np.array([1, 2, 3, 4])
y = np.array([1, 4, 9, 16])

plt.subplot(1, 2, 2)
plt.plot(x,y)
plt.title("plot 2")

plt.suptitle("RUNOOB subplot Test")
plt.show()
```

![image-20220903152407807](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903152407807.png)



# 3.其他图形绘制

## 3.1散点图绘制

```
import matplotlib.pyplot as plt
import numpy as np

x = np.array([1, 2, 3, 4, 5, 6, 7, 8])
y = np.array([1, 4, 9, 16, 7, 11, 23, 18])

plt.scatter(x, y)
plt.show()
```

![image-20220903152536753](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903152536753.png)

## 3.2柱状图绘制

```
import matplotlib.pyplot as plt
import numpy as np

x = np.array(["Runoob-1", "Runoob-2", "Runoob-3", "C-RUNOOB"])
y = np.array([12, 22, 6, 18])

plt.bar(x, y, width = 0.1)
plt.show()
```

![image-20220903152617807](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903152617807.png)

## 3.3饼图绘制

```
import matplotlib.pyplot as plt
import numpy as np

y = np.array([35, 25, 25, 15])

plt.pie(y,
        labels=['A','B','C','D'], # 设置饼图标签
        colors=["#d5695d", "#5d8ca8", "#65a479", "#a564c9"], # 设置饼图颜色
        explode=(0, 0.2, 0, 0), # 第二部分突出显示，值越大，距离中心越远
        autopct='%.2f%%', # 格式化输出百分比
       )
plt.title("RUNOOB Pie Test")
plt.show()
```

![image-20220903152740122](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903152740122.png)

## 3.4Matplotlib 3D绘图

**3D线段**

```
#导入三维工具包

from mpl_toolkits import mplot3d    

import numpy as np    

import matplotlib.pyplot as plt  

fig = plt.figure(dpi=500)   #像素500 

#创建3d绘图区域    

ax = plt.axes(projection='3d')    

ax.set_title('3D线段')

x=np.arange(0,4)       

y=np.arange(0,4)    

z=np.arange(0,4)    

ax.plot3D(x,y,z,'red') #调用 ax.plot3D创建三维线图   

plt.show()
```

![image-20220903153036938](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903153036938.png)

plt.figure()作用

figure(num=None, figsize=None, dpi=None, facecolor=None, edgecolor=None, frameon=True)

num:图像编号或名称，数字为编号 ，字符串为名称

figsize:指定figure的宽和高，单位为英寸；

dpi参数指定绘图对象的分辨率，即每英寸多少个像素，缺省值为80 

facecolor:背景颜色

edgecolor:边框颜色

frameon:是否显示边框

关于surface()介绍





| 参数       | 描述                           |
| ---------- | ------------------------------ |
| X,Y,Z      | 2D数组形式的数据值             |
| rstride    | 数组行距（步长大小）           |
| cstride    | 数组列距（步长大小）           |
| color      | 曲面块颜色                     |
| cmap       | 曲面块颜色映射                 |
| facecolors | 单独曲面块表面颜色             |
| norm       | 将值映射为颜色的 Nonnalize实例 |
| vmin       | 映射的最小值                   |
| vmax       | 映射的最大值                   |





**3D散点图**

```python
from mpl_toolkits import mplot3d    

import numpy as np    

import matplotlib.pyplot as plt    

fig = plt.figure(dpi=150)    #像素150

#创建绘图区域    

ax = plt.axes(projection='3d')    

#构建xyz    

z = np.linspace(0, 1, 100)    #0-1等间距产生100个数

x = z * np.sin(20 * z)    

y = z * np.cos(20 * z)     

ax.scatter3D(x, y, z, c='b', marker='o')  #c代表散点颜色 marker代表散点形状  

ax.set_title('3D散点图')    

plt.show()
```

![image-20220903153140698](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903153140698.png)

**3D图**

```
from mpl_toolkits import mplot3d 

import numpy as np 

import matplotlib.pyplot as plt 

fig = plt.figure() 

#创建绘图区域 

ax = plt.axes(projection='3d') 

x=np.arange(-4,4,0.25) 

y=np.arange(-4,4,0.25) 

x,y=np.meshgrid(x,y) #生成坐标矩阵

r=np.sqrt(x**2+y**2) #平方根

z=np.cos(r) 

ax.set_zlim(-2, 2) #z轴的范围

ax.contour(x, y, z, zdir = 'z',offset=-2) #表示投影到z = -2上

ax.plot_surface(x,y,z,cmap='rainbow')# cmap设置颜色映射 

show()
```

![image-20220903153326375](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220903153326375.png)