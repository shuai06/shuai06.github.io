# 经典现代卷积网络4-含并行连结的网络(GoogLeNet)

<script type="text/javascript" src="/js/src/bai.js"></script>
2014年提出
GoogLeNet吸收了NiN中网络串联网络的思想，并在此基础上做了很大改进。在随后的几年里，研究人员对GoogLeNet进行了数次改进，这里将介绍这个模型系列的第一个版本。

![20220915200530](http://image.xpshuai.cn/20220915200530.png)


## Inception块
GoogLeNet中的基础卷积块叫作Inception块
4个路径从不同层面抽取信息，然后再输出通道维合并
![结构](http://image.xpshuai.cn/20220915195600.png)

![](http://image.xpshuai.cn/20220915195702.png)

- 提供了4条路径（四种方法），输入输出等同高宽，然后作为不同的通道叠加
- 每个路径输出的通道数，可以自己来决定，相当于通过通道数来给每个路径分配权重
- 至于为什么给每个通道分配这些数量，玄学吧？反正作者没解释过
- inception块相比3X3和5x5卷积层有更少的参数和计算复杂度


跟单3x3或5x5卷积层相比，Inception块有更少的参数个数和计算复杂度

Inception后续有很多变种
![20220915203121](http://image.xpshuai.cn/20220915203121.png)




Inception块：
```python

import torch
from torch import nn
from torch.nn import functional as F
from d2l import d2ltorch as d2l

class Inception(nn.Module):
    # `c1`--`c4` 是每条路径的输出通道数
    def __init__(self,in_channels,c1,c2,c3,c4,**kwargs):
        super(Inception,self).__init__(**kwargs)
        # 线路1，单1 x 1卷积层
        self.p1_1 = nn.Conv2d(in_channels, c1, kernel_size=1)
        # 线路2，1 x 1卷积层后接3 x 3卷积层
        self.p2_1 = nn.Conv2d(in_channels,c2[0],kernel_size=1)
        self.p2_2 = nn.Conv2d(c2[0],c2[1],kernel_size=3,padding=1)
        # 线路3，1 x 1卷积层后接5 x 5卷积层
        self.p3_1 = nn.Conv2d(in_channels,c3[0],kernel_size=1)
        self.p3_2 = nn.Conv2d(c3[0],c3[1],kernel_size=5,padding=2)
        # 线路4，3 x 3最⼤汇聚层后接1 x 1卷积层
        self.p4_1 = nn.MaxPool2d(kernel_size=3,stride=1,padding=1)
        self.p4_2 = nn.Conv2d(in_channels,c4,kernel_size=1)
    
    def forward(self,x):
        p1_1 = F.relu(self.p1_1(x))
        p2_1 = F.relu(self.p2_1(x))
        p2_2 = F.relu(self.p2_2(p2_1))
        p3_1 = F.relu(self.p3_1(x))
        p3_2 = F.relu(self.p3_2(p3_1))
        p4_1 = F.relu(self.p4_1(x))
        p4_2 = F.relu(self.p4_2(p4_1))
        p1,p2,p3,p4 = p1_1,p2_2,p3_2,p4_2
        # 在通道维度上连结输出
        return torch.cat((p1, p2, p3, p4), dim=1)

```

## 模型


GoogLeNet ⼀共使⽤9个Inception块和全局平均汇聚层的堆叠来⽣成其估计值。
Inception块 之间的最⼤汇聚层可降低维度。
第⼀个模块类似于AlexNet 和LeNet，Inception块的栈从VGG继承，全局平 均汇聚层避免了在最后使⽤全连接层。

GoogLeNet结构
![20220915201257](http://image.xpshuai.cn/20220915201257.png)



**多个inception块叠加（一个stage一般意味着高宽减半操作：**
![20220915200326](http://image.xpshuai.cn/20220915200326.png)
下面是详细讨论每个stage：
![20220915200353](http://image.xpshuai.cn/20220915200353.png)
![20220915200407](http://image.xpshuai.cn/20220915200407.png)
![20220915203000](http://image.xpshuai.cn/20220915203000.png)


```python

# 接下来我们自己动手实现这个网络
# 第⼀个模块使⽤64个通道,7X7卷积层
b1 = nn.Sequential(
    nn.Conv2d(1,64,kernel_size=7,padding=3,stride=2),
    nn.ReLU(),
    nn.MaxPool2d(kernel_size=3,padding=1,stride=2)
)
b2 = nn.Sequential(
    nn.Conv2d(64,64,kernel_size=1),
    nn.ReLU(),
    nn.Conv2d(64,192,kernel_size=3,padding=1),
    nn.MaxPool2d(kernel_size=3,padding=1,stride=2)
)


# stage3开始就要使用inception块了
b3 = nn.Sequential(
    # 输入192 输出 64 + 128 + 32 + 32 = 256
    Inception(in_channels= 192,c1 = 64,c2=(96,128),c3=(16,32),c4=32),
    # 输出为 128 + 192+ 96+ 64 = 480
    Inception(256, 128, (128, 192), (32, 96), 64),
    nn.MaxPool2d(kernel_size=3,padding=1,stride=2)
)

# stage4 stage3能写出来，4就照样做就可以了
b4 = nn.Sequential(
    Inception(480, 192, (96, 208), (16, 48), 64),
    Inception(512, 160, (112, 224), (24, 64), 64),
    Inception(512, 128, (128, 256), (24, 64), 64),
    Inception(512, 112, (144, 288), (32, 64), 64),
    Inception(528, 256, (160, 320), (32, 128), 128),
    nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
)
# stage5是输出层
b5 = nn.Sequential(
    Inception(832, 256, (160, 320), (32, 128), 128),
    Inception(832, 384, (192, 384), (48, 128), 128),
    # 自适应平均池化层, 能够自动选择步幅和填充，比较方便
    nn.AdaptiveAvgPool2d((1,1)),
    nn.Flatten()
)


# 这里b1-b5都是nn.Sequential，但是好像可以直接嵌套进新的nn.Sequential
net = nn.Sequential(b1, b2, b3, b4, b5, nn.Linear(1024, 10))


# 这里输入不是224X224，改为96X96，加快训练速度
X = torch.rand(size=(1, 1, 96, 96))
for layer in net:
    X = layer(X)
    print(layer.__class__.__name__,'output shape:\t', X.shape)
    
    
    
lr, num_epochs, batch_size = 0.1, 10, 128
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size, resize=96)
d2l.train_ch6(net, train_iter, test_iter, num_epochs, lr, d2l.try_gpu())
```







> 可以看出来，我已经在堆文字了.....确实学不下去了，改天再回来继续认真看......


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E7%BB%8F%E5%85%B8%E7%8E%B0%E4%BB%A3%E5%8D%B7%E7%A7%AF%E7%BD%91%E7%BB%9C4-%E5%90%AB%E5%B9%B6%E8%A1%8C%E8%BF%9E%E7%BB%93%E7%9A%84%E7%BD%91%E7%BB%9C-googlenet/  

