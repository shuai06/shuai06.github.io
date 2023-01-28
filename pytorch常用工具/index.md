# Pytorch常用工具

<script type="text/javascript" src="/js/src/bai.js"></script>



## 数据处理

### Dataset
> 主要负责数据的抽象
> DataSet是抽象类，无法实例化
数据集被抽象为Dataset类,实现自定义的数据集需要集成Dataset，并实现以下两个魔法方法：
- `__getitem__()`: 返回一条数据或一个样本。`obj[index]`等价于`obj.__getitem__(index)`
- `__len__()`:返回样本是数量。`len(obj)`等价于`obj.__len__()`
  

**下面是前面刘二老师讲的糖尿病例子中的数据集部分来说明dataset的用法：**
```python
import torch
import numpy as np

# DataSet是抽象类，无法实例化
from torch.utils.data import Dataset
# DataLoader可实例化
from torch.utils.data import DataLoader


class DiabetesDataset(Dataset):
    def __init__(self, filepath):
        xy = np.loadtxt(filepath, delimiter=',', dtype=np.float32)
        # 获得数据集长度
        self.len = xy.shape[0]
        self.x_data = torch.from_numpy(xy[:, :-1])
        self.y_data = torch.from_numpy(xy[:, [-1]])

    # 获得索引方法
    # 每次返回一个样本
    def __getitem__(self, index):
        return self.x_data[index], self.y_data[index]

    # 获得数据集长度
    def __len__(self):
        return self.len


dataset = DiabetesDataset('diabetes.csv')
# num_workers表示多线程的读取
train_loader = DataLoader(dataset=dataset,batch_size=32,shuffle=True,num_workers=2)


```


**另一个Kaggle上的"Dogs vs Cats"的例子：**
```python
  
  
```
但是这里返回的数据不适合实际使用，存在问题：
- 返回样本的形状不统一(每张图片的大小不一样)，这对于按batch训练的神经网络来说很不友好
- 返回的样本的数值较大，没有进行归一化

针对上述问题，pytorch提供了`torchvision`工具包，
**torchvision**是一个视觉工具包，其中transform模块提供了一系列数据增强的操作：
https://pytorch.org/vision/stable/transforms.html#transforms
将一系列操作使用`Compose`进行连接，如下：
```python

import torch as t
from torch import nn
from torch.utils.data import Dataset, DataLoader
from PIL import Image
import numpy as np
import os
from torchvision import transforms as T

# 通过Compose将一系列操作连接起来
transforms = T.Compose([
    T.Resize(224),  # 缩放图像，保持长宽比不变，最短边为224像素
    T.CenterCrop(224),  # 从图像中间切除224像素X224像素的图层
    T.ToTensor(),  # 图像转换成tensor，归一化至[0,1]
    # 标准化至[-1,1],规定均值和标准差
    T.Normalize(mean=[.5, .5, .5], std=[.5, .5, .5])
])

# Dataset是抽象类，不能被实例化，所以也只能用来抽象数据集，所以读取数据操作交给DataLoader

class DogCatDataset(Dataset):
    def __init__(self, root):
        # 所有图像的绝对路径，这里不实际加载影像，只是指定路径，在__getitem__时才会真正读取图像
        imgs = os.listdir(root)
        self.imgs = [os.path.join(root,img) for img in imgs]
        # 使用
        self.transforms = transforms

    def __getitem__(self, index):
        img_path = self.imgs[index]
        # dog：1，cat：0
        label = 1 if 'dog' in img_path.split("/"[-1]) else 0
        data = Image.open(img_path)
        if self.transforms:
            data = self.transforms(data)
        return data, label

    def __len__(self):
        return len(self.imgs)


# test
dataset = DogCatDataset("./data/dogcat/traning/")
img, label = dataset[0]  # 相当于调用__getitem__
for img, label in dataset:
    print(img.size(), img.float().mean(), label)

```

除了 上述变换，transforms还可以通过lanbda封装成自定义的转换策略。

同样，transforms分为:
- torchversion.transforms
- 和torchversion.transforms.functional : 提供了更灵活的操作，可以对多个对象以相同的参数进行操作，在使用时需要自己指定所有参数，具体方法翻看官方文档

torchvision中除了上述的transforms，还定义了常见的数据集，不过多介绍。
这里只介绍一个常用的Dataset：`ImageFolder`，它的实现和上面十分类似，它假设所有的图像按文件件保存，每个文件件下都是同一类别的图像...可以直接使用`.class_idx`按照类别打标签




### DataLoader
考虑到实际中一次处理的是一个batch的数据，同时还有对一批数据打乱顺序和并行加速等，所以提供了DataLoader

https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader


```python
from torch.utils.data import DataLoader


train_loader = DataLoader(dataset=dataset,batch_size=32,shuffle=True,num_workers=2)

dataiter = iter(train_loader)
imgs, labels = next(dataiter)
imgs.size()

# dataloader是一个可迭代对象，可以像迭代器一样使用
for batch_datas, batch_labels in train_loader:
    print(batch_datas,batch_labels)
```

处理数据中，如果遇到某样本无法读取，比如某张图像损坏，此时在`__getitem__`方法会抛出异常，此时要将出错样本剔除/返回None，然后在DataLoader中实现自定义的`collate_fn`将空对象过滤掉


DataLoader默认使用单进程进行加载数据，可以指定进程数。

**建议：**
- 高负载的操做放在`__getitem__`中。多进行加载数据时，程序会并行调用`__getitem__`实现加速
- DataSet中尽量包含只读对象，避免修改任何可变对象。在使用多进程加载数据时候，每个进程都会复制DataSet对象并执行`_worker_loop`函数。如果某一个进程修改了部分数据，那么在另一个进程复制的对象中，这部分数据并不会被修改。


PyTorch还提供了`Sampler`模块，用来对数据进行采样。当shuffle参数为True时，会调用RandomSampler采样器打乱数据。默认的采样器是`SequentialSampler`按顺序采样。有个很有用的采样器：`WeightedRandomSampler`会根据每个样本权重选择数据


多进程中如果主程序终止其他进程一直占用GPU和内存，如何杀：
```bash
ps x | grep <py文件名> | awk '{print &1}' | xargs kill -9
```



## 预训练模型
torchvision还提供了深度学习中各种经典的网络结构及预训练模型，在`torchvision.models`中，包括：
- 经典的分类模型：VGG, ResNet, DenseNet, MobileNet...
- 语义分割模型：FCN, DeepLabV3...
- 目标检测模型：Faster RCNN....
- 实例分割模型：Mask RCNN...

可以在其模型基础上根据需求对网络结构进行修改



## 可视化工具
在训练神经网络时候，通常希望更直观了解训练情况，如损失函数的曲线，输入的图像，输出的图像等信息，以帮助优化提供方向和依据。

### 1.TensorBoard
TensorBoard最初是作为TF的可视化工具流行起来的，PyTorch1.1.0之后内置了TensorBoard相关接口，手动安装TensorBoard后便可调用相关结构进行数据可视化

TensorBoard安装&启动：
```bash
pip install tensorboard

# 启动
tensorboard --logdir=[path即log'文件保存路径]
```


**常见操作：**
记录标量  tb.add_histogram("conv1.bias", model.conv1.bias, epoch)
显示图像  tb.add_image("images", grid)
显示直方图 tb.add_histogram()
显示网络结构  tb.add_graph(model, images)
可视化embedding   tb.add_embedding()
...
```python
import torch
import torch.nn as nn
import torch.optim as opt
torch.set_printoptions(linewidth=120)
import torch.nn.functional as F
import torchvision
import torchvision.transforms as transforms
from torch.utils.tensorboard import SummaryWriter


# 定义一个函数统计最后分类正确率
def get_num_correct(preds, labels):
    return preds.argmax(dim=1).eq(labels).sum().item()


class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(in_channels=1, out_channels=6, kernel_size=5)
        self.conv2 = nn.Conv2d(in_channels=6, out_channels=12, kernel_size=5)

        self.fc1 = nn.Linear(in_features=12*4*4, out_features=120)
        self.fc2 = nn.Linear(in_features=120, out_features=60)
        self.out = nn.Linear(in_features=60, out_features=10)

    def forward(self, x):
        x = F.relu(self.conv1(x))
        x = F.max_pool2d(x, kernel_size = 2, stride = 2)
        x = F.relu(self.conv2(x))
        x = F.max_pool2d(x, kernel_size = 2, stride = 2)
        x = torch.flatten(x,start_dim = 1)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.out(x)

        return x

# 导入数据并创建训练加载器
train_set = torchvision.datasets.FashionMNIST(root="./data",
        train = True,
         download=True,
        transform=transforms.ToTensor())
train_loader = torch.utils.data.DataLoader(train_set,batch_size = 100, shuffle = True)


# 在tensorboard中展示图像和图
tb = SummaryWriter()
model = CNN()
images, labels = next(iter(train_loader))
grid = torchvision.utils.make_grid(images)
tb.add_image("images", grid)  # 显示图像
tb.add_graph(model, images)  # 显示网络结构
tb.close()
# 这段代码执行后，打开浏览器本地的6006端口


# 记录标量/可视化训练循环中各参数变化情况
device = ("cuda" if torch.cuda.is_available() else cpu)
model = CNN().to(device)
train_loader = torch.utils.data.DataLoader(train_set,batch_size = 100, shuffle = True)
optimizer = opt.Adam(model.parameters(), lr= 0.01)
criterion = torch.nn.CrossEntropyLoss()

tb = SummaryWriter()

for epoch in range(10):

    total_loss = 0
    total_correct = 0

    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)
        preds = model(images)

        loss = criterion(preds, labels)
        total_loss+= loss.item()
        total_correct+= get_num_correct(preds, labels)

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    tb.add_scalar("Loss", total_loss, epoch)
    tb.add_scalar("Correct", total_correct, epoch)
    tb.add_scalar("Accuracy", total_correct/ len(train_set), epoch)

    tb.add_histogram("conv1.bias", model.conv1.bias, epoch)
    tb.add_histogram("conv1.weight", model.conv1.weight, epoch)
    tb.add_histogram("conv2.bias", model.conv2.bias, epoch)
    tb.add_histogram("conv2.weight", model.conv2.weight, epoch)

    print("epoch:", epoch, "total_correct:", total_correct, "loss:",total_loss)

tb.close()


```




### 2.Visdom
Meta专门为PyTorch开发的一个可视化工具，开源于2017年3月, 轻量级，可以生人大多数科学运算可视化任务。
支持多种数据可视化：数值、图像、文本、视频...通同时支持PyTorch、Torch和Numpy

**Visdom两个重要概念：**
- `env`:环境。不同环境的可视化结果相互隔离，互不影响。如果不指定env则默认使用main
- `pane`:窗格。可以对它进行拖动、缩放、保关闭等。一个程序可以使用同一个env中的不同pane，每个pane都可以可视化或记录不同的信息


安装:
```bash
 pip install visdom
```

启动服务：
```bash
python -m visdom.server

# 或直接挂后台
nohup python -m visdom.server &

# visdom服务是一个Web Server服务，默认绑定8097端口，cs之间使用tornado进行非阻塞交互
visdom成功启动后，会返回一个网址（如下图）。根据显示的网址然后在浏览器里输入：http://localhost:8097 进行登录。


```
![20220906111022](http://image.xpshuai.cn/20220906111022.png)



注意：
- 需要手动指定保存env，可以在web界面点击save按钮 or 在程序中调用save方法。否则重启后env信息会消失
- cs之间使用tornado，可视化操作不会阻塞当前程序，网络异常也不会导致程序退出


**常用操作：**

Visdom可视化神经网络的训练过程大致分为3步：
1. 实例化一个窗口
2. 初始化窗口的信息
3. 更新监听的信息


```python
import torch as t
import visdom
 
# 新建一个连接客户端
# 指定env = u'test1'，默认端口为8097，host是‘localhost'
vis = visdom.Visdom(env=u'test1',use_incoming_socket=False)
 
x = t.arange(1, 30, 0.01)
y = t.sin(x)
vis.line(X=x, Y=y, win='sinx', opts={'title': 'y=sin(x)'})
```


Visdom支持Pytorch的Tensor和Numpy的ndarray，但是不支持Python的int、float等，因此每次传入数据都需要转换成ndarry或者Tensor数据类型
有两个使用较多的参数：
- `win`:指定pane的名字，如果不指定会自动分配一个新的pane，如果两次操作指定相同的pane，那么新的操作会覆盖当前pane的内容。建议每次操作都重新指定pene
- `ops`:可视化配置，接收一个字典，主要用于设置pean的显示格式

训练过程中，为了避免覆盖之前的pane的内容，需要指定参数` update='append`,处理使用update参数，还可以使用`vis.updateTrace`方法更新图

vis作为一个客户端对象，可以使用常见的画图函数，包括：
- line：类似Matlab中的plot操作，用于记录某些标量的变化，如损失、准确率等
- image：可视化图片，可以是输入的图片，也可以是GAN生成的图片，还可以是卷积核的信息
- text：用于记录日志等文字信息，支持html格式
- histgram：可视化分布，主要是查看数据、参数的分布
- scatter：绘制散点图
- bar：绘制柱状图



监听单一数据：示例：监听train_loss的变化
```python
# 首先导入库
from visdom import Visdom
import numpy as np
import time


# 1.实例化一个窗口
wind = Visdom(env=u"test1", use_incoming_socket=False)
# 2.初始化窗口信息
wind.line([0.],  # Y的第一个点的坐标
          [0.],  # X的第一个点的坐标
          win='train_loss',  # 窗口的名称
          opts=dict(title='train_loss')  # 图像的标例
          )
# 3.更新数据
for step in range(10):
    # 随机获取loss,这里只是模拟实现
    loss = np.random.randn() * 0.5 + 2
    wind.line([loss], [step], win='train_loss', update='append')  # wind.line()    update='append'
    time.sleep(0.5)

```
![20220906111156](http://image.xpshuai.cn/20220906111156.png)


监听多条数据：示例：监听train_loss和acc
```python

from visdom import Visdom
import numpy as np
import time
 
# 实例化窗口
wind = Visdom()
# 初始化窗口参数
wind.line([[0., 0.]], [0.], win='train', opts=dict(title='loss&acc', legend=['loss', 'acc']))
# 更新窗口数据
for step in range(10):
    loss = 0.2 * np.random.randn() + 1
    acc = 0.1 * np.random.randn() + 0.5
    wind.line([[loss, acc]], [step], win='train', update='append')
    time.sleep(0.5)
```





通过update参数（append/new）控制数据更新：
```python
import visdom
import torch as t

vis = visdom.Visdom(env="image test")

# append 追加数据
for ii in range(0, 10):
    # y = x
    x = t.Tensor([ii])
    y = x
    vis.line(X=x, Y=y, win='polynomial', update='append' if ii > 0 else None)

# updateTrace 新增一条线
x = t.arange(0, 9, 0.1)
y = (x ** 2) / 9
vis.line(X=x, Y=y, win='polynomial', name='this is a new Trace', update='new')
```


**Visdom可视化图像：**
- `image`接收一个二维(HxW)或三维(3xHxW)的向量
- `images`接收一个四维向量(NxCxHxW),类似torchvision中的make_grid功能，将多张image拼接在一起

```python
from visdom import Visdom
import cv2
import numpy as np
import torch
import os
# 新建一个连接客户端
# 指定env = u'test1'，默认端口为8097，host是‘localhost'
vis = Visdom(env=u'test1',use_incoming_socket=False)
 
# 读入图像
path1 = os.getcwd()
 
print("Current Path: {0}".format(path1))
image = cv2.imread('gudongche-01.jpg')
# # openCV按照BGR读取，而visdom 默认按照RGB显示,因此要进行通道转换
img = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
# visdom类似于pytorch中的卷积模型，接收的数据都要求通道数在前
img = np.transpose(img, (2, 0, 1))
# 将numpy类型转换为torch类型
img = torch.from_numpy(img)
# 可视化图像
vis.image(img, win='test')

# 
# viz.image(
#     img,
#     opts={'title': 'Random!', 'caption': 'Click me!'},

```



在一个image窗口中不断更新显示图像:
```python
import time
import cv2
import visdom
import numpy as np
 
viz = visdom.Visdom(env="image test")
 
 
img = cv2.imread("gudongche-01.jpg")
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
img = np.transpose(img, (2, 0, 1))
# img = img.astype(np.float32) / 255
print(img.shape, img.dtype)
# image demo
image = viz.image(np.random.rand(3, 256, 256), opts={'title': 'image1', 'caption': 'How random.'})
for i in range(100):
    viz.image(np.random.randn(3, 256, 256), win=image)
    time.sleep(0.5)

```

多个图像：
```python
import time
import cv2
import visdom
import numpy as np
 
viz = visdom.Visdom(env="image test")
 
 
img = cv2.imread("gudongche-01.jpg")
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
img = np.transpose(img, (2, 0, 1))
# img = img.astype(np.float32) / 255
print(img.shape, img.dtype)
# image demo
images = viz.images(
    np.random.randn(20, 3, 64, 64),
    opts=dict(title='Random images', caption='How random.', nrow=5)
)
for i in range(100):
    # 可视化20张随机的彩色图片，每一行5张
    viz.images(np.random.randn(20, 3, 64, 64), win=images, nrow=5)
    time.sleep(0.5)
```

**文本可视化：**
`vis.text`
支持HTML标签




## 使用GPU加速：CUDA
PyTorch中`Tensor`和`nn.Module`(包含常用的layer、损失函数、容器Sequential等)数据结构分为`CPU`和`GPU`两个版本，这两个数据结构都带有一个`.cuda()`方法(转换成对应的GPU对象)
Note:
- `tensor.cuda`会返回一个新对象，新对象的数据转移到GPU上，之前的Tensor还在原理的CPU上
- `module.cuda`会返回自己，将所有数据迁移到GPU上。其本质也是利用了Tensor


除了`.cuda`方法，还有`.to(device)`方法

使用建议：
- 较小的计算直接在CPU计算，无须放到GPU
- CPU与GPU之间传递数据比较耗时，尽量避免

```python
t1 = t.Tensor(3,4)
# 返回一个新的tensor，保存在第一块GPU上，原来的Tensor没有改变
t1.cuda(0)  # 如果不写0，默认是第一块
t1.is_cuda  # False


n2 = nn.Linear(3,4)
n2.cuda(device=1) 
n2.weight.is_cuda  # True

# 使用to()转移
t2 = t.Tensor(3,4)
t2.to('cuda:0')
t2.is_cuda  


```


Note：大部分损失函数属于nn.Module，用户经常忘记使用.cuda方法，大部分时候不会报错，是因为损失函数没有可学习参数，但为了保险起见，尽量加上`criterion.cuda`
```python
...
criterion.cuda()
loss = criterion(input, target)
...
```

除了`.cuda`，还可以通过`torch.cuda.device`指定默认使用哪块GPU，或者使用`torch.set_default_tensor_type('torch.cuda.FloatTensor')指定默认的Tensor类型为GPU上的...`执行默认GPU，不需要手动调用`.cuda`


如果有多块GPU，手工`.cuda(1)`切换比较繁琐，有两种替代方法：
- 先调用`torch.cuda.set_device(1)`指定使用第二块GPU，后续的`.cuda`都无须更改，切换CPU只需修改这一行代码
- 设置环境变量：`CUDA_VISIBLE_DEVICES`


指定Tensor加载的设备
```python

# t.device('cpu')
# t.device('cpu',0)

# t.device('cuda:0')
# t.device('cuda',0)


### 推荐写法
device = t.device("cuda" if t.cuda.is_available() else "cpu")

# 在确定了设备之后，可以利用to方法将数据与模型加载到指定设备上
x = t.empty((2,3)).to(device)

```


使用`torch.Tensor.new_*`或`torch.*_like`可以创建与该Tensor具有相同类型、相同设备的Tensor

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/pytorch%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7/  

