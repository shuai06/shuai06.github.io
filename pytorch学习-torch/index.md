# Pytorch学习-Torch与autograd

<script type="text/javascript" src="/js/src/bai.js"></script>

# Tensor
## Tensor基础
Tensor，又名张量。
可以简单认为Tensor是一个支持高效科学计算的数组。它可以是一个数(标量)、一维数组(向量)、二维数组(矩阵、黑白图像等)、或者更高维的数组(高阶数据、视频等)。它与Numpy的ndarray类似，但Tensor支持GPU加速。





## 基本操作
与Numpy的接口设计比较类似。
- 比如`torch.sum(a,b)`和`s.sum(b)`是等价的
- 从修改存储角度，分为两类操作：
- 不会修改自身存储的数据的操作，如`a.add(b)`，加法结果会返回一个新的tensor
- 会修改自身存储的数据的操作，如`a.ddd_(b)`，有个下划线，加法的结果会被存储在a中返回



### 1.创建Tensor
创建方法很多，常用的如下表：
| 函数                              | 功能                      |
| --------------------------------- | ------------------------- |
| Tensor(*sizes)                    | 基础构造函数              |
| tensor(data,)                     | 类似np.array的构造函数    |
| ones(*sizes)                      | 全1Tensor                 |
| zeros(*sizes)                     | 全0Tensor                 |
| eye(*sizes)                       | 对角线为1，其他为0        |
| arange(s,e,step)                  | 从s到e，步长为step        |
| linspace(s,e,steps)               | 从s到e，均匀切分成steps份 |
| rand/randn(*sizes)                | 均匀/标准分布             |
| normal(mean,std)/uniform(from,to) | 正态分布/均匀分布         |
| randperm(m)                       | 随机排列                  |

上表中创建数据类型的同时可以指定`数据类型dtype`和`存储设备device`

创建未初始化的 5,3 维度的Tensor
```python
# 创建未初始化的 5,3 维度的Tensor
x = t.empty(5, 3)
print(x)
# 输出
tensor([[1.5695e-43, 1.5554e-43, 1.5975e-43],
        [1.3593e-43, 1.5975e-43, 1.4714e-43],
        [1.5134e-43, 1.6956e-43, 4.4842e-44],
        [1.6395e-43, 1.5414e-43, 1.3593e-43],
        [1.6535e-43, 1.3593e-43, 1.4714e-43]])
 

```

创建一个5x3的随机初始化的Tensor:

```python
x = torch.rand(5, 3)
print(x)

```

创建一个5x3的long型全0的Tensor:
```python
x = torch.zeros(5, 3, dtype=torch.long)
print(x)

```

直接根据数据创建:
```python
x = torch.tensor([[5.5, 3], [1,2]])
print(x)

#输出
tensor([[5.5000, 3.0000],
        [1.0000, 2.0000]])
```



输入数据是一个Tensor:
```python
x = torch.Tensor(torch.rand(2,6))
x
```


指定形状：
```python
x = torch.Tensor(3,7)
x
```

通过现有的Tensor来创建:
此方法会默认重用输入Tensor的一些属性，例如数据类型，除非自定义数据类型
```python

x = x.new_ones(5, 3, dtype=torch.float64)  # 返回的tensor默认具有相同的torch.dtype和torch.device
print(x)

x = torch.randn_like(x, dtype=torch.float) # 指定新的数据类型
print(x) 

```


torch.ones()相关
```python
torch.ones(2,3) # 创建形状2,3的全1的tensor

torch.zeros(2,3) # 创建形状2,3的全0的tensor

torch.ones_like(input_t) # 创建一个跟输入tensor形状一样的全1的tensor

torch.eye(2,3, dtype=t.int) # 创建一个对角线值为1，其余值为0的tensor

```




torch.Tensor()与torch.tensor()区别：
- torch.Tensor()是类，默认是torch.FloatTensor()
- torch.tensor()是函数，data支持list/tuple/array/scalar等类型。直接从data进行数据复制，并根据源数据的类型生成对应类型的Tensor。且其接口与numoy更像，所以更推荐这种创建方法
```python
# torch.Tensor()能直接创建空的张量
torch.Tensor()

# torch.tensor()不能创建空的，必须传入一个数据
torch.tensor() # 报错：TypeError: tensor() missing 1 required positional arguments: "data"
torch.tensor(())  # 等效与创建空的

```


创建一个起始值为1，终止值为19，步长为1的tensor:
```python
torch.arange(1, 19, 3)
```

创建一个区间内3等份是tensor:
```python
torch.linspace(1,10,3)
```

创建一个形状的tensor，取值是标准正态分布中抽取的随机值:
```python
torch.randn(2,3)
```

长度为5，随机排列的:
```python
torch.randperm(5)
```


创建一个大小为(2,3)，值全1的tensor，保留原始的数据类型和设备:
```python
a = tensor.tensor((), dtype=tensor.int32)
a.new_ones((2,3))

```



通过shape或者size()来获取Tensor的形状:
```python

print(x.size()) # 注意：返回的torch.Size其实就是一个tuple, 支持所有tuple的操作。

print(x.shape)

```


统计元素总数:
```python
a.numel()

a.nelement()

```

### 2.Tensor的类型
**设备类型：**
- cuda
- cpu
`t.dtype`


数**据类型：**
每个数据类型都有CPU和GPU版本
`t.device`




**不同数据类型Tensor之间相互转换的方法：**
- 如：`tensor.type(new_type)`,快捷方法`tensor.float()`,`tensor.half()`等
- 设备类型转换：`t.cuda()`,`t.cpu()`,或`t.to(divice)`
- `t.*_like(t1)`生成和t1相同属性的新tensor,`t.new_*(new_shape)`生成和相同属性但是相撞不同的新tensor,`t1.as_type(t2)`修改tensor的类型


```python
# 以下代码只有在PyTorch GPU版本上才会执行
if torch.cuda.is_available():
    device = torch.device("cuda")          # GPU
    y = torch.ones_like(x, device=device)  # 直接创建一个在GPU上的Tensor
    x = x.to(device)                       # 等价于 .to("cuda")
    z = x + y
    print(z)
    print(z.to("cpu", torch.double))       # to()还可以同时更改数据类型

```



### 3.索引
> Note:索引出来的结果与原数据共享内存，也即修改一个，另一个会跟着修改。
比如：
```python
y = x[0, :]
y += 1
print(y)
print(x[0, :]) # 源tensor也被改了

# 输出
tensor([1.4963, 1.1113, 1.8689, 1.4671, 1.9555, 1.4945])
tensor([1.4963, 1.1113, 1.8689, 1.4671, 1.9555, 1.4945])
```

常见索引：
```python
"""
对于x
tensor([[1.4963, 1.1113, 1.8689, 1.4671, 1.9555, 1.4945],
        [0.7284, 0.2248, 0.1715, 0.6737, 0.0110, 0.6075]])
"""

x[0] # 第一行
x[:, 1] # 第二列
x[1, -2:]  # 第二行最后两个元素

```


```python
# 返回布尔值
print(x>1)
tensor([[ True,  True,  True,  True,  True,  True],
        [False, False, False, False, False, False]])


print((x>1).int())

```



返回满足条件的结果：
```python
print(x[x>1]) # 返回的结果不共享内存
print(x.masked_select(x>1))

# where保留原始索引，不满足条件位置置0
print(torch.where(x>1), x, torch.zeros_like(x))
```






除了常用索引来选择数据，还有一些高级的选择函数：
| 级的选择函数:                   |                                                       |
| ------------------------------- | ----------------------------------------------------- |
| 函数                            | 功能                                                  |
| index_select(input, dim, index) | 在指定维度dim上选取，比如选取某些行、某些列           |
| masked_select(input, mask)      | 例子如上，a[a>0]，使用ByteTensor进行选取，选取结果不共享内存              |
| nonzero(input)                  | 非0元素的下标                                         |
| gather(input, dim, index)       | 根据index，在dim维度上选取数据，输出的size与index一样 |
|                                 |                                   

gather:
```python
a = torch.arange(0,16).view(4,4)

# 选择对角线上元素
index = t.tensor([0,1,2,3])
a.gather(0, index)

# 选择反对角线上元素
index = t.tensor([3,2,1,0]).t()
a.gather(1, index)

```

gather的逆操作：scatter_。gather将数据从input中按index取出；scatter_将按照index数据写入
> scatter_是inplace操作，会直接对当前数据进行修改



item():将一个标量Tensor转换成一个Python number：
```python
# 只针对包含一个元素的tensor有效
x = torch.randn(1)
print(x)
print(x.item())

```









### 4.拼接
- cat(tensor,dim):将多个tensor在指定维度上进行拼接
- stack(tensor,dim):将多个tensor沿一个**新的维度**进行拼接
```python
a = torch.arange(0,16).view(2,8)

# cat执行在dim=0上拼接
torch.cat((a,a), 0)
#结果
tensor([[ 0,  1,  2,  3,  4,  5,  6,  7],
        [ 8,  9, 10, 11, 12, 13, 14, 15],
        [ 0,  1,  2,  3,  4,  5,  6,  7],
        [ 8,  9, 10, 11, 12, 13, 14, 15]])

# cat执行在dim=1上拼接
torch.cat((a,a), 0)
#结果
tensor([[ 0,  1,  2,  3,  4,  5,  6,  7,  0,  1,  2,  3,  4,  5,  6,  7],
        [ 8,  9, 10, 11, 12, 13, 14, 15,  8,  9, 10, 11, 12, 13, 14, 15]])



# stack在dim=0上拼接
b = torch.stack((a,a), 0)
#结果
tensor([[[ 0,  1,  2,  3,  4,  5,  6,  7],
         [ 8,  9, 10, 11, 12, 13, 14, 15]],

        [[ 0,  1,  2,  3,  4,  5,  6,  7],
         [ 8,  9, 10, 11, 12, 13, 14, 15]]])
#形状
print(b.shape)
torch.Size([2, 2, 8])


```

高级索引
>不与原tensor共享内存

```python
a = torch.arange(0,16).view(2,2,4)

a[[1,0], [1,1], [2,0]] # a[1,1,2] 和a[0,1,0]
# tensor([14,  4])

```






### 5.逐元素&归并操作
常用逐元素操作：
![20220905104714](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220905104714.png)

举例：
```python
clamp(torch.clamp(a,min=2,max=4)) # 上下截断

```

常用归并操作：
![20220905105000](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220905105000.png)
大多数执行归并函数都有一个维度参数dim，指定在哪个维度上进行
关于dim的解释有点乱，下面是经验总结(并非所有函数都符号如下变化)：
假设输入形状(m,n,k):
- 如果指定dim=0，那么输出形状为(1,n,k)或(n,k)
- 如果指定dim=1，那么输出形状为(m,1,k)或(m,k)
- 如果指定dim=2，那么输出形状为(m,n,1)或(m,n)

维度中是否有1，取决于参数keepdim是否为True(保留维度)


```python
(b.sum(dim=0, keepdim=False)).shape #torch.Size([3])
(b.sum(dim=0, keepdim=True)).shape # torch.Size([1, 3])



#
b.sum(dim=0) # tensor([2., 2., 2.])

b.sum(dim=1) # tensor([3., 3.])

```




### 6.比较
![20220905105929](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220905105929.png)

以max为例，有三种情况：
- max(input)
- max(input, dim)  指定维度返回最大值
- max(tensor1, tensor2) 返回两个tensor上对应位置较大的元素


比较两个整形tensor可以用`==`比较，对于有精度限制的浮点数需要使用`allclose`比较




### 7.算术操作

- 加法: `add()`
- 减法: `sub`
- 乘法: `multiply`
- 除法: `div`
- 倒数: `reciprocal`




同一个操作有很多种形式，以加法为例子：
```python
## 形式1
y = torch.rand(5, 3)
print(x + y)


## 形式2
print(torch.add(x, y))

# 还可以指定输出
result = torch.empty(5, 3)
torch.add(x, y, out=result)
print(result)


# 形式3：inplace
# adds x to y
y.add_(x)
print(y)


```
> 注：PyTorch操作inplace版本都有后缀_, 例如x.copy_(y), x.t_()




## 改变形状
### 查看Tensor的维度
- tensor.shape
- tensor.size()  与上面等价
- tensor.dim()  查看维度，等价于 len(tensor.shape),对应Numpy的array.ndim
- tensor.numel()  查看元素数量，等价于tensor.shape[0]*tensor.shape[1]...或者np.prod(tensor.shape)，对应Numpy中的array.size





### **改变维度：**
#### reshape()
> 会自动先把内存中不连续的Tensor变成连续的，然后进行形状变化等价于tensor.contiguous().view(new_shape)


#### view()
> 仅能处理空间中连续的Tensor，经过view之后的Tensor仍共享存储区
> 如果原Tensor在空间上不连续，会报错


用`view()`来改变Tensor的形状：
```python
y = x.view(15)
z = x.view(-1, 5)  # -1所指的维度可以根据其他维度的值推出来
print(x.size(), y.size(), z.size())

# 输出：torch.Size([5, 3]) torch.Size([15]) torch.Size([3, 5])

```
> 注意view()返回的新Tensor与源Tensor虽然可能有不同的size，但是是共享data的，也即更改其中的一个，另外一个也会跟着改变。(顾名思义，view仅仅是改变了对这个张量的观察角度，内部数据并未改变)


reshape()此函数并不能保证返回的是其拷贝。推荐先用clone创造一个副本然后再使用view
```python
x_cp = x.clone().view(15)
x -= 1
print(x)
print(x_cp)
# 使用clone还有一个好处是会被记录在计算图中，即梯度回传到副本时也会传到源Tensor。

```

**常用快捷变形方法：**
- tensor.view(dim1, -1, dim2)  指定其中一个维度为-1，pytorchhi自动计算对应形状
- tensor.view_as(other) 把形状变得跟另一个一样，等价于tensor.view(other.shape)
- tensor.squeeze()  将tensor中尺寸为1的维度减掉。例如：形状(1,3,1,4)会变为(3,4)
- tensor.flatten(start_dim=0, end_dim=-1) 将tensor形状中的某些连续的维度合并为一个维度。例如，(2,3,4,5)会变为(2,12,5)
- tensor[None]和tensor.unsqueeze(dim):为tensor新建一个维度，该维度尺寸为1



### Tensor的转置
- transpose: 只能用于两个维度的转置
- permute：可以对任意高纬度矩阵进行转置
- tensor.t()
- tensor.T

转置会使得tensor在空间上不连续，此时最好通过`tensor.contiguous()`将其变成连续的

```python
# C * H * W
img1 = torch.randn(3,128,256)

img2 = img1.permute(1,2,0) # 代表 H,W,C的索引来进行转置

```


### 9.其他操作
常用的线性代数基本操作
![20220905110224](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220905110224.png)



此外，在`torch.distributions`中还提供了可自定义参数的概率分布函数和采样函数。






## Tensor与Numpy相互转换
> 我们很容易用`numpy()`和`from_numpy()`将Tensor和NumPy中的数组相互转换。但是需要注意的一点是： 这两个函数所产生的的Tensor和NumPy中的数组共享相同的内存（所以他们之间的转换很快），改变其中一个时另一个也会改变！！
> torch.tensor()会进行数据拷贝，但会消耗更多时间和空间


Tensor转Numpy：numpy
```python
a = torch.ones(5)
b = a.numpy()
print(a, b)

a += 1
print(a, b)
b += 1
print(a, b)

```



Numpy转Tensor：from_numpy
```python
import numpy as np
a = np.ones(5, dtype=np.float32)
b = torch.from_numpy(a) # dtype为float32，所以共享内存(修改b的值a也会变)
print(a, b)

a += 1
print(a, b)
b += 1
print(a, b)


```
注意1: 使用`torch.Tensor()`创建的张量默认是float32，如果numpy的数据类型与默认的不一致，那么数据仅仅会被复制，不会共享内存

注意2: 无论输入什么类型，`torch.tensor()`都只进行数据复制，不会共享内存，要注意
```python
a_tensor = torch.tensor(a)
a_tensor[0,1] = 111
a_tensor   # a和s_tendor不共享内存
```

PyTorch还提供了`torch.utils.dlpack`模块，仍是共享内存的

所有在CPU上的Tensor（除了CharTensor）都支持与NumPy数组相互转换。




## 命名张量
允许用户将显式名称与Tensor的维度关联起来，便于其他操作，推荐使用维度代名词进行维度操作。

```python
import warning
warning.filter("ignore")

imgs = torch.randn(1,2,2,3, names=('N', 'C', 'H', 'W'))
imgs.names

# 输出 ('N', 'C', 'H', 'W')

# 旋转
imgs_rotate = imgs.transpose(2,3)
imgs_rotate.names

# 修改部分维度的名称 rename(H='Height')

# 对未命名的张量命名，不需要的用None表示 refine_names('N','W',None,'C')

# 通过维度的名词进行维度变换
align_to()
```

命名张量可以提高安全性，如果两个张量维度相同但是Tensor的维度名称没有对齐，那么也无法进行计算
```python
imgs = torch.randn(1,2,2,3, names=('N', 'C', 'H', 'W'))
imgs2 = torch.randn(1,2,2,3, names=('C', 'N', 'W', 'H'))
# img2 + imgs2 会报错

```


## Tensor的基本结构
分为：
- 信息区(Tensor): 头信息区主要保存size、Stride、数据类型等
- 存储区(Storeage):真正的数据被保存册亨连续数组

绝大多数操作不是修改Tensor的存储区，而是修改Tensor的头信息(更节省内存，提升了速度)。此外，有些操作会导致Tensor不联系，这需要调用`tensor.contiguous()`变成连续的数据，该方法会复制数据到新内存，不再与原来的数据共享存储区。


## 使用Torch从零实现线性回归

```python
"""
不使用深度学习框架的技术，只使用Torch从零实现线性回归
"""
import torch
import torch as t
import matplotlib.pyplot as plt


device = t.device('cpu')  # 如果使用GPU就修改

# 设置随机种子，保证在不同机器上运行时输出一致
t.manual_seed(2022)


def get_fake_dta(batch_size=8):
    '''
    产生随机数据：y=2x+3,加上一些噪声
    :param batch_size:
    :return: x, y
    '''
    # 均值为batch_size，方差为1
    x = t.rand(batch_size, 1, device=device) * 5
    y = x*2+3 + t.rand(batch_size, 1, device=device)
    return x, y


# 随机初始化参数
w = t.rand(1, 1).to(device)
b = t.rand(1, 1).to(device)
lr = 0.02  # 学习率

for ii in range(500):
    x, y = get_fake_dta(batch_size=4)

    # 前向传播
    y_pred = x.mm(w) + b.expand_as(y)  # expand_as用到了广播机制
    loss = 0.5 * (y_pred -y) ** 2  # 均方误差
    loss = loss.mean()

    # 反向传播： 手动计算梯度
    dloss = 1
    dy_pred = dloss * (y_pred-y)  #

    dw = x.t().mm(dy_pred)  # 梯度
    db = dy_pred.sum()

    # 更新参数
    w.sub_(lr * dw)
    b.sub_(lr * db)

    if ii % 50 == 0:
        # 画图
        x = t.arange(0,6).float().view(-1,1)
        y = x.mm(w)+b.expand_as(x)
        plt.plot(x.numpy(), y.numpy())

        x2, y2 = get_fake_dta(batch_size=32)
        plt.scatter(x2.numpy(), y2.numpy())

        plt.xlim(0, 5)
        plt.ylim(0, 13)
        plt.show()
        plt.pause(0.5)

print(f"w:{w.item():.3f}, b:{b.item():.3f}")

```
可以看到还是稍微复杂的...



# autograd

autograd自动求导，能够根据输入和前向传播过程自动构建计算图，执行反向传播。
只需要对Tensor增加`requires_grad=True`属性

几个简单例子：

```python
a = torch.randn(3,4, requires_grad=True)
a.requires_grad # True


b = torch.randn(3,4, requires_grad=True)
c = (a + b).sum() # c的requires_grad也被设置成了True
c.backward()
c  # tensor(4.4589, grad_fn=<SumBackward0>)
a.grad  # 梯度

c.grad_fn # 查看这个Tensor的反向传播函数：<SumBackward0 at 0x142081550>  这里是sum的，所以是这个函数名

c.grad_fn.next_functions  # 保存了frad_fn的输入(tuple)


## 叶子节点(is_leaf=True)
#requires_grad为False的时候就是叶子Tensor
#requires_grad为True，且是由用户创建的时候也是叶子Tensor，她的梯度信息会被保留下来
a.is_leaf  # True

```

autograd沿着计算图从根几点溯源，利用链式法则计算所有叶子节点的梯度。每个前向传播操作的函数都有阈值对应的反向传播函数来计算输入Tensor的梯度，这些函数的名字通常以Backward结尾

反向传播过程中，非叶子节点的梯度不会被保存，如果想查看这些变量的梯度，有两种办法：
- 使用`autograd.grad`函数
- 使用`hook`方法（推荐使用）




其他：
- 由用户创建的节点叫做叶子节点，叶子节点的grad_fn为None。对于在叶子节点中需要求导的tensor，因为其梯度是累加的，所以剧透AccumulateGrad标识
- Tensor默认不需要求导，如果一个节点的requires_grad被设置为True，那么所有依赖他的节点也是True
- 多次反向传播过程中，梯度是不断累加的。多次反向传播，需要指定retain_graph=True来保存中间缓存
- 反向传播中，非叶子节点的梯度不会被保存，可以使用autograd.grad或hook来获取
- Tensor的grad与data形状一致，应该避免直接修改tensor.data
  




## 使用autograd从零实现线性回归

```python
"""
加上autograd从零实现线性回归
"""
import torch
import torch as t
import matplotlib.pyplot as plt
import numpy as np

device = t.device('cpu')  # 如果使用GPU就修改

# 设置随机种子，保证在不同机器上运行时输出一致
t.manual_seed(2022)


def get_fake_dta(batch_size=8):
    '''
    产生随机数据：y=2x+3,加上一些噪声
    :param batch_size:
    :return: x, y
    '''
    # 均值为batch_size，方差为1
    x = t.rand(batch_size, 1, device=device) * 5
    y = x*2+3 + t.rand(batch_size, 1, device=device)
    return x, y


# 随机初始化参数
w = t.rand(1, 1, requires_grad=True)
b = t.rand(1, 1, requires_grad=True)
lr = 0.03  # 学习率
losses = np.zeros(500)

for ii in range(500):
    x, y = get_fake_dta(batch_size=4)

    # 前向传播:计算loss
    y_pred = x.mm(w) + b.expand_as(y)  # expand_as用到了广播机制
    loss = 0.5 * (y_pred -y) ** 2  # 均方误差
    loss = loss.sum()
    losses[ii] = loss.item()

    # 反向传播: 自动计算梯度
    loss.backward()

    # 更新参数
    w.sub_(lr * w.grad.data)
    b.sub_(lr * b.grad.data)

    # 梯度清零
    w.grad.data.zero_()
    b.grad.data.zero_()

    if ii % 50 == 0:
        # 画图
        x = t.arange(0,6).float().view(-1,1)
        y = x.mm(w)+b.expand_as(x)
        plt.plot(x.numpy(), y.numpy())

        x2, y2 = get_fake_dta(batch_size=32)
        plt.scatter(x2.numpy(), y2.numpy())

        plt.xlim(0, 5)
        plt.ylim(0, 13)
        plt.show()
        plt.pause(0.5)

print(f"w:{w.item():.3f}, b:{b.item():.3f}")


```

稍微简单一些了......





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pytorch%E5%AD%A6%E4%B9%A0-torch/  

