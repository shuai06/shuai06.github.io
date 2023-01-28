# 经典现代卷积网络3-网络中的网络(NiN)

<script type="text/javascript" src="/js/src/bai.js"></script>

前几节介绍的LeNet、AlexNet和VGG在设计上的共同之处是：先以由卷积层构成的模块充分抽取空间特征，再以由全连接层构成的模块来输出分类结果。
NiN提出了另外一个思路，即串联多个由卷积层和“全连接”层(NiN使用1×11×1卷积层来替代全连接层，从而使空间信息能够自然传递到后面的层中去)构成的小网络来构建一个深层网络


NiN是在AlexNet问世不久后提出的

![](http://image.xpshuai.cn/20220915192405.png)


**NiN块**是NiN中的基础块。它的架构：
- 无全连接层
- 交替使用**一个卷积层**加**两个充当全连接层的1×1卷积层**串联而成。
其中第一个卷积层的超参数可以自行设置，而第二和第三个卷积层的超参数一般是固定的。
- 除使用NiN块以外，NiN还有一个设计与AlexNet显著不同：NiN去掉了AlexNet最后的3个全连接层(全连接层耗时且容易过拟合)，取而代之地，NiN使用了输出通道数等于标签类别数的NiN块，然后使用**全局平均池化层**对每个通道中所有元素求平均并直接用于分类

![](http://image.xpshuai.cn/20220915193724.png)


https://zhuanlan.zhihu.com/p/47391705


```python

import torch
from torch import nn
from d2l import torch as d2l



def nin_block(in_channels, out_channels, kernel_size, strides, padding):
	"""
	NiN块
	"""
    return nn.Sequential(
	    nn.Conv2d(in_channels, out_channels, kernel_size, strides, padding),
	    nn.ReLU(),
	    # 这里等于是用1*1卷积代替全连接层，见附录
	    nn.Conv2d(out_channels, out_channels, kernel_size=1), nn.ReLU(),
	    nn.Conv2d(out_channels, out_channels, kernel_size=1), nn.ReLU())


# alexnet同时期算法
net = nn.Sequential(
    nin_block(1, 96, kernel_size=11, strides=4, padding=0),
    nn.MaxPool2d(3, stride=2),
    nin_block(96, 256, kernel_size=5, strides=1, padding=2),
    nn.MaxPool2d(3, stride=2),
    nin_block(256, 384, kernel_size=3, strides=1, padding=1),
    nn.MaxPool2d(3, stride=2),
    nn.Dropout(0.5),
    # 标签类别数是10
    nin_block(384, 10, kernel_size=3, strides=1, padding=1),
    nn.AdaptiveAvgPool2d((1, 1)),
    # 将四维的输出转成⼆维的输出，其形状为(批量⼤⼩, 10)
    nn.Flatten())

	# softmax写在train函数里面了，交叉熵
    

X = torch.randn(size=(1, 1, 224, 224))
for blk in net:
    X = blk(X)
    print(blk.__class__.__name__,'output shape:\t',X.shape)



lr, num_epochs, batch_size = 0.1, 10, 128
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size, resize=224)
d2l.train_ch6(net, train_iter, test_iter, num_epochs, lr, d2l.try_gpu())
```


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E7%BB%8F%E5%85%B8%E7%8E%B0%E4%BB%A3%E5%8D%B7%E7%A7%AF%E7%BD%91%E7%BB%9C3-%E7%BD%91%E7%BB%9C%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9C/  

