# Pytorch学习-nn

<script type="text/javascript" src="/js/src/bai.js"></script>


## nn.Mdule
torch.nn是专门为深度学习设计的模块，它的核心数据结构是`Module`



以实现全连接层(仿射层)为例，它的输出y和输入x满足$y=wx+b$
```python

import torch as t
from torch import nn


# 继承nn.Module
class Linear(nn.Module):
    # 必须重写__init__()和前向传播函数forward
    def __init__(self, in_feature, out_feature):
        super().__init__()   # 记得调用
        # __init__()中可自行定义可学习参数，并封装成nn.Parameter(是一种特殊的tensor，默认徐亚求导)
        # nn.Parameter内的参数是网络中的可学习参数
        self.W = nn.Parameter(t.randn(in_feature, out_feature))
        self.b = nn.Parameter(t.randn(out_feature))
        # 比如一些层都可以定义在这里

    def forward(self, x):
        """
        实现了前向传播,它的输入可以是一个或者多个tensor
        :param x:
        :return:
        """
        x = x.mm(self.W)  # 矩阵乘法，相当于x@(self.W)
        return x + self.b.expand_as(x)

    # 反向传播无需手动编写，nn.Module能利用autograd自动实现反向传播，这笔Function简单许多


layer1 = Linear(4,3)
input = t.randn(2,4)
output = layer1(input)
print(output)

# 可学习参数会通过named_parameters返回一个迭代器
for name, parameter in layer1.named_parameters():  
    print(name, parameter)  # w和b

```


实现多层感知机(MLP)
![](https://image.geoer.cn/20220905172144.png)
```python
"""
多层感知机
"""
import torch as t
from torch import nn



# 继承nn.Module
class Linear(nn.Module):
    # 必须重写__init__()和前向传播函数forward
    def __init__(self, in_feature, out_feature):
        super().__init__()   # 记得调用
        # __init__()中可自行定义可学习参数，并封装成nn.Parameter(是一种特殊的tensor，默认徐亚求导)
        # nn.Parameter内的参数是网络中的可学习参数
        self.W = nn.Parameter(t.randn(in_feature, out_feature))
        self.b = nn.Parameter(t.randn(out_feature))
        # 比如一些层都可以定义在这里

    def forward(self, x):
        """
        实现了前向传播,它的输入可以是一个或者多个tensor
        :param x:
        :return:
        """
        x = x.mm(self.W)  # 矩阵乘法，相当于x@(self.W)
        return x + self.b.expand_as(x)

    # 反向传播无需手动编写，nn.Module能利用autograd自动实现反向传播，这笔Function简单许多

class Perceptron(nn.Module):
    """
    多层感知机MLP
    """
    def __init__(self, in_feature, hidden_feature, out_feature):
        super().__init__()   # 记得调用
        # 此处的Linear是前面自定义的全连接层
        self.layer1 = Linear(in_feature, hidden_feature)
        self.layer2 = Linear(in_feature, out_feature)

    # 在forwar中加上各层之间的处理函数，并定义层与层之间的关系
    def forward(self, x):
        x = self.layer1(x)
        x = t.sigmoid(x)
        x = self.layer2(x)

        return x

perception = Perceptron(3,4,1)
for name, parameter in perception.named_parameters():
    print(name, parameter.size())  # w和b


```

MLP运行结果：
```bash
layer1.W torch.Size([3, 1])
layer1.b torch.Size([1])
layer2.W torch.Size([3, 1])
layer2.b torch.Size([1])
```


nn.Modlue还有很多其他层，可以自己翻看官方文档，有以下几个主注意的点：
1.关注构造参数的作用
![](https://image.geoer.cn/20220905172717.png)

2.关注属性、可学习的网络、和包含的子Module
![](https://image.geoer.cn/20220905172933.png)


3.输入输出的形状
![](https://image.geoer.cn/20220905172843.png)





## 常用的神经网络层
### 图像相关层
#### 1.卷积层
> 卷积层的本质就是卷积层、池化层、激活层及其它层的叠加

https://pytorch.org/docs/stable/nn.html#convolution-layers



以二维卷积类Conv2d为例：
https://pytorch.org/docs/stable/generated/torch.nn.Conv2d.html#torch.nn.Conv2d
![](https://image.geoer.cn/20220905174352.png)

卷积输出结果的形状：
![](https://image.geoer.cn/20220905174456.png)





#### 2.池化层
> 主要用于下采样。
> 增加池化层可以保留主要特征的同时降低参数量，从而在一定程度上防止过拟合。
> 池化层没有学习参数

https://pytorch.org/docs/stable/nn.html#pooling-layers



#### 3.其他层
- Linear：全连接层
- BatchNorm:批标准化层
- Dropout:防止过拟合


### 激活函数
PyTorch实现了常见激活函数，他们可以作为独立的Layer使用


### 构建神经网络
前面的层都是前一层直接作为下一层的输入，这种称为前馈神经网络(FNN).
对于此类网络，重写forward比较麻烦，可以简化使用：
- `Sequential`：特殊的module，可以包含多个子module，在前向传播时候会将输入一层层传递下去
- 和`ModuleList`：特殊的module，可以包含多个子module，可以向list一样视同，但是不能直接将输入传给ModuleList

Sequential的三种写法
```python
import torch as t
from torch import nn


# 第一种
net1 = nn.Sequential()
net1.add_module('conv', nn.Conv2d(3,3,3))
net1.add_module('batchnorm', nn.BatchNorm2d(3))
net1.add_module('active', nn.ReLU())


# 第二种
net2 = nn.Sequential(
    nn.Conv2d(3,3,3),
    nn.BatchNorm2d(3),
    nn.ReLU()
)


# 第三种
from collections import OrderedDict
net3 = nn.Sequential(OrderedDict([
    ('conv',nn.Conv2d(3,3,3)),
    ('batchnorm',nn.BatchNorm2d(3)),
    ('active', nn.ReLU()),

]))



print(net1)
"""
Sequential(
  (conv): Conv2d(3, 3, kernel_size=(3, 3), stride=(1, 1))
  (batchnorm): BatchNorm2d(3, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
  (active): ReLU()
)
"""


# 可以根据名字或者序号取出子module
print(net1[1])
print(net2.conv)


```



为什么多此一举用ModuleList，而不用自带的list呢？
```python

class MyModule(nn.Module):
    def __init__(self):
        super().__init__()
        self.li = [nn.Linear(3,4), nn.Conv2d(3,3,(1,2)), nn.ReLU()]  # 自定义的list不能被主module识别，在反向传播的时候将无法调整子module的参数
        self.module_li = nn.ModuleList([nn.Linear(3,4), nn.Conv2d(3,3,(1,2)), nn.ReLU()])  #能被主module识别

    def forward(self):
        pass

```

除了ModuleList，还有ParameterList
如果在__init__()中需要构造list/tuple/dict等对象，那么一定要思考是够该用ModuleList和ParameterList代替






### RNN
...





## nn.functional
nn.module中大多数的layer在nn.functional中都有一个阈值对应的函数。
**区别：**
- 使用nn.module实现的layer是一个特殊的类，会自动提取可学习参数
- 使用nn.functional实现的layer更像是纯函数

**使用：**
如果模型具有可学习参数，那么最好使用nn.module，否则都行。
对于激活函数层、池化层等都没有可学习参数(Note:也不用放在构造函数__init__中)，但还是建议使用nn.module中






## 优化器
常用的优化方法封装在`torch.optim`中，所有优化方法都继承自`optim.Optimizer`

应学会使用：
- 优化方法的基本使用方法
- 如何对模型的不同部分设置不同的学习率
- 如歌调整学习率


这里以SGD为例：
```python
# 以前面的多层感知机为例
# ...
from torch import optim
# 为一个网络设置学习率
optimizer = optim.SGD(params=perception.parameters(), lr=1)
optimizer.zero_grad()  # 梯度清零，等价于 net.zero_grad()

input = t.rand(32,3)
out = perception(input)
out.backward(out)  # 真正的反向传播过程在下一步执行
optimizer.step()   # 执行优化




# 不同参数设置不同的学习率
bias_params = [param for name, param in perception.named_parameters() if name.endswith('.b')]
weight_params = [param for name, param in perception.named_parameters() if name.endswith('.W')]

optimizer = optim.SGD([
    {'parms':bias_params},
    {'parms':weight_params, lr = 0.0001}
], lr=0.1)



```

调整学习率有两种做法：
- 修改`optimizer.param_group`中对应学习率
- 新建一个优化器

```python
# 新建一个优化器
optimizer = optim.SGD([
    {'parms':bias_params},
    {'parms':weight_params, lr = 0.0001}
], lr=0.1)

# 手动衰减学习率，保存动量
for param_group in optimizer.param_groups:
    param_group['lr'] *= 0.1

```





## PyTorch中保存模型
```python

# 所有module对象都具有state_dict()函数,会返回当前module所有状态数据。保存后下次使用利用load_state_dict加载进来
t.save(perception.state_dict(), "多层感知机.pth")

perception2 = Perceptron(3,4,1)
perception2.load_state_dict(t.load("多层感知机.pth"))

```


## 在GPU上运行module
两步：
1. `model=model.cuda`：将模型的所有参数转存到GPU上
2. `inout.cuda`：将输入数据防止在GPU上


多块GPU上进行并行计算，PyTorch提供了2个函数：
- `nn.parallel.data_parallel()` 直接利用多块GPU并行得到计算结果
- 和`nn.parallel.DataParallel()`返回一个新的module，能够自动在多块GPU上进行并行加速

```python

# 方法1
new_net = nn.parallel.DataParallel(perception, device_ids=[0,1])
out = new_net(input)
# 将一个batch的数据均分成多分，分别送到对应的GPU上进行计算，然后将各块GPU得到的梯度进行累加...

# 方法2
nn.parallel.data_parallel(new_net, input,device_ids=[0,1] )


```






## 使用PyTorch实现线性回归(与前一篇文章作对比)

```python

import torch


# 1.准备数据集
x_data = torch.Tensor([[1.0], [2.0], [3.0]])  # 3行1列的tensor
y_data = torch.Tensor([[2.0], [4.0], [6.0]])


# 2.使用类来设计模型
class LinearModel(torch.nn.Module):  # Module构造出来的对象，会自动构建反向传播过程
    def __init__(self):
        super(LinearModel, self).__init__()
        self.linear = torch.nn.Linear(1, 1)  # torch.nn.Linear构造一个对象，参数是权重和偏差，也是继承子Module的会自动进行反向传播
        # (in_features: size, out_features: size of each out feature, bias=True:要不要偏置量)

    def forward(self, x):  # 实现这个方法
        # 计算 y heat
        y_pred = self.linear(x)  # linear是前面建立的对象，可调用的callable的对象（def __call__(self, *args, **kwargs)）
        return y_pred


model = LinearModel()  # model是callable的  model(x)
# 3.构建损失和优化器
criterion = torch.nn.MSELoss(size_average=False)
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)  # 梯度下降
# 可以尝试不同的优化器

# 4.训练
for epoch in range(100):
    y_pred = model(x_data)  # 1.前向传播，计算y heat
    loss = criterion(y_pred, y_data)  # 2.计算损失
    print(epoch, loss)  # 打印

    optimizer.zero_grad()  # 梯度会自动计算，务必梯度清零！
    loss.backward()    # 3.反向传播
    optimizer.step()   # 4.更新 update

# 打印权重和偏置
print("w=", model.linear.weight.item())  # model下面的linear，下面的weight
print("b=", model.linear.bias.item())

# 测试模型
x_test = torch.Tensor([[4.0]])
y_test = model(x_test)
print("y_pred=", y_test.data)


```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pytorch%E5%AD%A6%E4%B9%A0-nn/  

