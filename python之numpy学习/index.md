# Python之Numpy学习




## 介绍

NumPy是一个功能强大的Python库，主要用于对多维数组执行计算。

NumPy这个词来源于两个单词：Numerical和Python。

NumPy提供了大量的库函数和操作，可以帮助程序员轻松地进行数值计算。在数据分析和机器学习领域被广泛使用



**特点：**

1. numpy内置了并行运算功能，当系统有多个核心时，做某种计算时，numpy会自动做并行计算。
2. Numpy底层使用C语言编写，内部解除了GIL（全局解释器锁），其对数组的操作速度不受Python解释器的限制，效率远高于纯Python代码。
3. 有一个强大的N维数组对象Array（一种类似于列表的东西，它是一系列 "同类型数据" 的集合）。
4. 实用的线性代数、傅里叶变换和随机数生成函数。

总之，是一个非常**高效**的用于处理数值型运算的包。






## Numpy数组对象

Numpy中的多维数组称为`ndarray`，这是Numpy中最常见的数组对象。

ndarray对象通常包含两个部分：

- ndarray数据本身
- 描述数据的元数据



**Numpy数组的优势:**

- Numpy数组通常是由相同种类的元素组成的，即数组中的数据项的类型一致。这样有一个好处，由于知道数组元素的类型相同，所以能快速确定存储数据所需空间的大小。
- Numpy数组能够运用向量化运算来处理整个数组，速度较快；而Python的列表则通常需要借助循环语句遍历列表，运行效率相对来说要差。
- Numpy使用了优化过的C API，运算速度较快
  

**Numpy数组和Python列表性能对比：**

比如我们想要对一个Numpy数组和Python列表中的每个素进行求平方。代码如下：

```python
import numpy as np
import time


t1 = time.time()
a = []
for x in range(100000):
    a.append(x**2)
t2 = time.time()
t = t2 - t1
print(t)  # 0.0342259407043457


t3 = time.time()
b = np.arange(100000)**2
t4 = time.time()
print(t4-t3)  # 结果为：0.0002129077911376953  可见速度快了很多

```

看完上面**numpy处理速度如此之快**的例子，是不是更有动力学下去了呢！





## 创建ndarray数组

导入numpy库，在导入numpy库时通常使用“np”作为简写，这也是Numpy官方倡导的写法

```python
import numpy as np
```



常用的创建数组的方式：

### 方式1：基于list或tuple

```python
import numpy as np


# 1.一维数组
# 基于list
arr1 = np.array([1, 2, 3, 4])
# arr1 = np.array([1, 2, 3, 4], dtype=int) 创建数组的时候可以指定数组元素的数据类型
print(arr1)

# 基于tuple
arr2 = np.array((1, 2, 3, 4))
print(arr2)


# 2.二维数组
arr3 = np.array([[1, 2, 3, 4], [5,6,7,8]]) # 每个小列表外面还有一个大的[]符号
print(arr3)

```



### 方式2： 基于np.arrange

```python
# 一维数组
arr1 = np.arange(5)
print(arr1) # [0 1 2 3 4]

# 二维数组
arr2 = np.array([np.arange(3), np.arange(3)])
print(arr2)

```



### 方式3：**基于arange以及reshape创建多维数组**

```python

# 创建三维数组
arr3 = np.arange(24).reshape(2, 3, 4) # arange的长度与ndarray的维度的乘积要相等，即 24 = 2X3X4
print(arr3)

# 创建二维数组
arr2 = np.arange(30).reshape(5, 6)
print(arr2)

```



### 方法4：**empty方法创建数组**

该方式可以创建一个空数组，`dtype`可以指定随机数的类型，否则随机采用一种类型生成随机数。

```python
import numpy as np

dt = np.numpy([2, 2], dtype=int)

```





### 方法5：**使用zeros/ones创建数组**

调用`zeros/ones`方法会创建一个全为0/1值的数组，通常在数组元素位置，大小一致的情况下来生成临时数组。0/1充当占位符。

```python
import numpy as np


dt = np.zeros([3, 5], dtype=int)
print('数组：', dt)
print('数据类型：', dt.dtype)
dt = np.ones([5, 3], dtype=float)
print('数组：', dt)
print('数据类型：', dt.dtype)

"""
数组： [[0 0 0 0 0]
 [0 0 0 0 0]
 [0 0 0 0 0]]
数据类型： int64
数组： [[1. 1. 1.]
 [1. 1. 1.]
 [1. 1. 1.]
 [1. 1. 1.]
 [1. 1. 1.]]
数据类型： float64
"""
```



### 方法6：**numpy.random创建数组**

**1.使用`numpy.random.rand`创建数组**

很多情况下手动创建的数组往往不能满足业务需求，因此需要创建随机数组。

用于生成[0.0, 1.0)之间的随机浮点数， 当没有参数时，返回一个随机浮点数，当有一个参数时，返回该参数长度大小的一维随机浮点数数组，参数建议是整数型，因为未来版本的numpy可能不支持非整形参数。

```python
import numpy as np


dt = np.random.rand(10)
print('数组：', dt)
print('数据类型：', dt.dtype)

```



**2.使用`numpy.random.randn`创建数组**

它能产生符合正态分布的随机数

```python
import numpy as np


dt = np.random.randn(3, 5)
print('数组：', dt)
print('数据类型：', dt.dtype)


"""
数组： [[ 0.67710547  1.97369481 -0.35002974 -1.0614128  -0.19732801]
 [-1.13230353 -0.53488291 -1.17557386 -0.67431941 -0.78063236]
 [-0.97807192  0.48440274  1.00098854 -1.9964855   0.89089835]]
数据类型： float64
"""
```



**3.使用`numpy.random.randint`创建数组**

```python
# 在10和30之间(左闭右开)产生随机数，并从中取5个数值来构建数组。
import numpy as np


dt = np.random.randint(10, 30, 5)
print('数组：', dt)
print('数据类型：', dt.dtype)

# np.random.random_integers(5) 是闭区间

```







### 方法7：**使用linspace创建数组**

`linspace`是基于一个范围来构造数组，参数`num`是开始值和结束值之间需要创建多少个数值。
`retstep`会改变计算的输出，返回一个元组，而元组的两个元素分别是需要生成的数组和数组的步差值。

```python
import numpy as np

dt = np.linspace(20, 30, num=5)
print('数组：', dt)
print('数据类型：', dt.dtype)
dt = np.linspace(20, 30, num=5, endpoint=False)
print('数组：', dt)
print('数据类型：', dt.dtype)
dt = np.linspace(20, 30, num=5, retstep=True)
print('元组：', dt)

```





### 方法8：logspace创建数组

和linspace函数类似，不过创建的是等比数列数组





### 方法9：**fromfunction创建数组**

`fromfunction`方法可以通过一个函数规则来创建数组。该方法中`shape`参数制定了创建数组的规则，`shape=(4,5)`，最终创建的结果就是4行5列的二维数组。

```python
import numpy as np


dt = np.fromfunction(lambda i, j:i + j, (4, 5), dtype=int)
print('数组：', dt)
print('数据类型：', dt.dtype)

```









## Numpy的数值类型

### Numpy数值类型如下：

![http://image.xpshuai.cn/numpy%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png](http://image.xpshuai.cn/numpy%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)

每一种数据类型都有相应的数据转换函数，如下：

```python
print(np.int8(12.332))
print(np.float64(12))

"""
12
12.0
"""
```



### 数组的类型转换

数组转换成list，使用tolist():

```python
a.tolist()

```

转换成指定类型，astype()函数:

```python
a.astype(float)
```



① 创建numpy数组的时候可以通过属性`dtype`显式指定数据的类型。

② 在不指定数据类型的情况下，numpy会自动推断出适合的数据类型。

③ 如果需要更改一个已经存在的数组的数据类型，可以通过`astype`方法进行修改从而得到一个新数组。

```python
a2 = np.array([1,2,3,4])  # 自动推断出合适的数据类型，里面无浮点数，变为int32
print(a2.dtype)
a3 = a2.astype(float)     # astype得到的是一个新数组，原数组没有改变。
print(a2.dtype)
print(a2)
print(a3.dtype)
print(a3)

"""
运行结果：
int32
int32
[1 2 3 4]
float64
[1. 2. 3. 4.]
"""
```



复数不能转换成为整数类型或者浮点数。





## Numpy数组的属性

**查看数组的属性:**

```python
import numpy as np


a = np.array([[1,2,3],[4,5,6]])
print(a)
print(a.ndim)    # ndarry维度   ndarry维度跟最外层括号[]有关
print(a.shape)   # ndarry形状
print(a.size)    # ndarry元素个数
print(a.dtype)   # ndarry元素类型

```

- **dtype属性**，ndarray数组的数据类型，数据类型的种类。

```python
print(np.arange(4, dtype=float))
print(np.arange(4, dtype='D')) # # 'D'表示复数类型
print(np.array([1.22,3.45,6.779], dtype='int8'))

```

- **ndim属性**，数组维度的数量

```python
a = np.array([[1,2,3], [7,8,9]])
print(a.ndim) # 2
```



- **shape属性**，数组对象的尺度，对于矩阵，即n行m列,shape是一个元组（tuple）

```python
a = np.array([[1,2,3],[4,5,6]])
print(a.shape)
```

- **size属性**用来保存元素的数量，相当于shape中nXm的值

```python
print(a.size)
```



- **itemsize**属性返回数组中各个元素所占用的字节数大小。

```python
a.itemsize
```

- **nbytes属性**，如果想知道整个数组所需的字节数量，可以使用nbytes属性。其值等于数组的size属性值乘以itemsize属性值。

```python
print(a.nbytes)
print(a.size*a.itemsize)
```

- **T属性**，数组转置

```python
b = np.arange(24).reshape(4,6)

print(b.T)
```

- **复数的实部和虚部属性，real和imag属性**

```python
d = np.array([1.2+2j, 2+3j])

# real属性返回数组的实部
d.real

# imag属性返回数组的虚部
d.imag
```

- **flat属性**，返回一个numpy.flatiter对象，即可迭代的对象。

```python 
for item in arr.flat:
    print(item)

print(arr.flat[0]) # 索引
arr.flat[1] = 555  # 也可以进行赋值
```



## ndarray数组的切片和索引

**一维数组的切片与索引**：和python的list类似

```python
arr[1:4]

arr[:6:2]
```





**二维数组的切片与索引**：和python的list类似

```python
arr = np.arrange(12).reshape(3,4)

a[0:3, 0:2] # 两个坐标轴上分别切片
```



```python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])
print(a[...,1])  # 取第二列
print(a[1,...])  # 取第二行
print(a[2])      # 取第三行

# ...表示不进行任何操作，和冒号:一样的意思

```



```python
x = np.array([[1,2],[3,4],[5,6]])
print(x)
                               # 数组，数组 → 取行和列中对应元素，维度为1维
y = x[[0,1,2],[0,1,0]]         # [0,1,2] 取的每行的所有，即第0行、第1行、第2行 此方法相对切片可以得到任意位置
print(y)

```

```python
x = np.array([[1,2],[3,4],[5,6]])
print(x)
                               # 切片，数组 → 取行对应元素，维度不变
y = x[1:3,[0,1]]               # 取第0列、第1列元素
z = x[1:3,[0,1,0]]             # 取第0列、第1列、第0列元素
print(y)
print(z)

```

```python
x = np.array([[1,2],[3,4],[5,6]])
y = x[[True,True,False]]       # 取第0行、第1行，第3行不取
print(y)
```

```python
x = np.arange(32).reshape((8,4))
print(x)
x[[-4,-2,-1,-7]]                # 取-4列、-2列、-1列、-7列的数组
```











## 处理数组形状

### 形状转换

1.**reshape()和resize()**

直接使用`reshape`函数创建一个改变尺寸的新数组，原数组的shape保持不变，但是新数组和原数组共享一个内存空间，也就是修改任何一个数组中的值都会对另外一个产生影响，另外要求新数组的元素个数和原数组一致。

```python
b.reshape(4,3)

```



函数`resize()`的作用跟reshape（）类似，但是会改变所作用的数组，相当于有inplace=True的效果

```python
b.resize(4,3)

```





**2.ravel()和flatten():**将多维数组转换成一维数组，如下：

```python
b.ravel()

b.flatten()

"""
两者的区别在于返回拷贝（copy）还是返回视图（view），flatten()返回一份拷贝，需要分配新的内存空间，对拷贝所做的修改不会影响原始矩阵，而ravel()返回的是视图（view），会影响原始矩阵。
"""
```



**3.用tuple指定数组的形状**

```python
b.shape=(2,6)
```



**4.转置**

前面描述了数组转置的属性（T），也可以通过transpose()函数来实现。

转置时重塑的一种特殊形式，它返回的是源数据的视图(不会进行任何赋值操作)。

```python
import numpy as np
arr = np.arange(15).reshape((3,5))
print(arr)

# 方法一：
print(arr.T)               # 用数组的T方法进行转置
print(arr.T)             #  默认将第一个维度和第二个维度进行转换
print(arr.T.shape)       #  第一个维度到最后一个维度，第二个维度到倒数第二个维

# 方法二：
print(np.transpose(arr))   # 用transpose方法一进行转置

# 方法三：
print(arr.transpose(1,0))  # 用transpose方法二进行转置

print(arr)                 # 源数据没有变化

```



```python
import numpy as np
arr = np.arange(120).reshape((2,3,4,5))
arr.transpose(0,3,2,1).shape        # 原本是arr.transpose(0,1,2,3) 因此指定第二个轴和第四个轴进行交换


##########################
import numpy as np
arr = np.arange(120).reshape((2,3,4,5))
arr.swapaxes(1,3).shape             # 跟 arr.transpose(0,3,2,1).shape 等价，swapaxes方法直接填写第一轴和第三轴即可

```

```python
# 元素0可表示为arr[0][0][0]，元素6可表示为 arr[0][1][2]  # 去掉一层括号，看它在哪个位置

arr = np.arange(24).reshape((2,3,4))
print(arr)
print(arr.transpose((1,0,2)))   # 表示将轴1和0位置互换，轴2不变，即代表将轴0和1对换，轴2不变，亦即将arr[x][y][z]中x和y位置互换，整个数组将变换
print(arr.swapaxes(1,2))        # 表示将轴1和轴2位置互换，轴0不变


```

```python
# 矩阵线性代数相乘
arr = np.arange(6).reshape((2,3))
np.dot(arr.T,arr)
```









### 堆叠数组

```python
c = a*2 # 每个元素都*2
```



- 水平叠加 hstack()

  ```python
  np.hstack((b,c))
  
  
  
  # column_stack()函数以列方式对数组进行叠加，功能类似hstack（）
  np.column_stack((b,c))
  ```







- 垂直叠加 vstack()

  ```python
  np.vstack((b,c))
  
  # row_stack()函数以行方式对数组进行叠加，功能类似vstack（）
  np.row_stack((b,c))
  ```



- concatenate()方法，通过设置axis的值来设置叠加方向
- axis=1时，沿水平方向叠加
- axis=0时，沿垂直方向叠加

```python
np.concatenate((b,c),axis=1)

np.concatenate((b,c),axis=0)
```

对数组的轴为0或1的方向经常会混淆，参考如下图：

![](http://image.xpshuai.cn/numpy%E6%95%B0%E7%BB%84%E6%96%B9%E5%90%91.png)





- 深度叠加

```python
arr_dstack = np.dstack((b,c))
print(arr_dstack.shape)
arr_dstack

```





### 数组拆分

跟数组的叠加类似，数组的拆分可以分为横向拆分、纵向拆分以及深度拆分。

涉及的函数为：

- hsplit()  沿横向轴拆分(axis=1)
- vsplit()  沿axis=0方向
- dsplit() 
- split()







## numpy常用统计函数

**注意函数在使用时需要指定axis轴的方向**，若不指定，默认统计整个数组

```bash
np.sum()，返回求和
np.mean()，返回均值
np.max()，返回最大值
np.min()，返回最小值
np.ptp()，数组沿指定轴返回最大值减去最小值，即（max-min）
np.std()，返回标准偏差（standard deviation）
np.var()，返回方差（variance）
np.cumsum()，返回累加值
np.cumprod()，返回累乘积值
```

举例：

```python
import numpy as np

x = np.arange(30).reshape(3, 10)
print(x)
print(np.max(x))  # 返回整个数组的最大值
print(np.max(x, axis=1))  # 沿axis=1轴方向统计 最大值 输出：[ 9 19 29]

```







## 数组的广播

> 当数组跟一个标量进行数学运算时，标量需要根据数组的形状进行扩展，然后执行运算。这个扩展的过程称为**广播（broadcasting）**



```python
import numpy as np


arr = np.arange(6).reshape((2,3))
print(arr)
print(arr + 2)
```



① 广播是numpy对不同形状的数组进行数值计算的方式。

② 如果两个数组a和b形状相同，即满足 a.shape == b.shape,那么 a*b的结果就是 a与b数组对应位相乘。这要求维数相同，且各维度的长度相同。

③ 广播就是 a.shape != b.shape，但是数组a和b能进行运算，数组a通过广播与数组b兼容。

④ 数组a和b能进行运算有两点情况：

> \1. 要么两个数组的维度相同。
> \2. 要么先比较最右边维度，看是否有一个为1，为1再看左边两个数组的维度是否相同或者为1。

⑤ 多个维度，只看最短的维度是否触发广播机制











## 参考

https://blog.csdn.net/perfectzxiny/article/details/115055241?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.1&utm_relevant_index=3

https://www.zhihu.com/question/433331468/answer/1611695850





















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python%E4%B9%8Bnumpy%E5%AD%A6%E4%B9%A0/  

