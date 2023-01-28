# h5py库初学

<script type="text/javascript" src="/js/src/bai.js"></script>
> 简单入门用法，记录一下

## 简介
h5py文件是存放两类对象的容器，数据集(dataset)和组(group)：
- dataset类似数组类的数据集合，和numpy的数组差不多
- group是像文件夹一样的容器，它好比python中的字典，有键(key)和值(value)。group中可以存放dataset或者其他的group。”键”就是组成员的名称，”值”就是组成员对象本身(组或者数据集)。



## 创建h5py文件
```python
import h5py

f=h5py.File("myh5py.hdf5","w") # w 写，r读

```


## 创建dataset数据集
创建了一个只有20个整数的数据集，但是没有赋值所以为0
```python
import h5py


f=h5py.File("myh5py.hdf5", "w")  # w 写，r读
# deset1是数据集的name，（20,）代表数据集的shape，i代表的是数据集的元素类型
d1=f.create_dataset("dset1", (20,), 'i')
for key in f.keys():
    print(key)
    print(f[key].name)
    print(f[key].shape)
    print(f[key][:])

```
![结果](http://image.xpshuai.cn/20220716224441.png)




**赋值：**

```python
# 赋值
d1[...]=np.arange(20)

# 或者我们可以直接按照下面的方式创建数据集并赋值
f["dset2"]=np.arange(15)

```
![赋值](http://image.xpshuai.cn/20220716224746.png)


如果我们有现成的numpy数组，那么可以在创建数据集的时候就赋值，这个时候就不必指定数据的类型和形状了，只需要把数组名传给参数data:`d1=f.create_dataset("dset1",data=a)`


## 创建group
```python
import h5py
import numpy as np


f=h5py.File("myh5py.hdf5","w")

# 创建一个名字为group1的组
g1=f.create_group("group1")

# 在b这个组里面分别创建name为dset1,dset2的数据集并赋值。
g1["dset1"]=np.arange(10)
g1["dset2"]=np.arange(12).reshape((3,4))

for key in g1.keys():
    print(g1[key].name)
    print(g1[key][:])


```
![](http://image.xpshuai.cn/20220716225208.png)





**查看结构：**
```python

import h5py
import numpy as np


f = h5py.File("myh5py.hdf5","w")

# 创建名字为group1,group2的组
g1 = f.create_group("group1")
g2 = f.create_group("group2")
# 创建数据集dst
dst = f.create_dataset('dst', data=np.random.normal(2, 1, 10))

# 在group1组里面创建一个组car1和一个数据集dset1。
c1 = g1.create_group("car1")
d1 = g1.create_dataset("dset1" , data=np.arange(10))


# 根目录下的组和数据集
print(".............")
for key in f.keys():
    print(f[key].name)

# group11这个组下面的组和数据集
print(".............")
for key in g1.keys():
    print(g1[key].name)

print("===")
print(c1.keys())

```

![](http://image.xpshuai.cn/20220716230909.png)



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/h5py%E5%BA%93%E5%88%9D%E5%AD%A6/  

