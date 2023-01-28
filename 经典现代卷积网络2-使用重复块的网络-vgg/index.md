# 经典现代卷积网络2-使用重复块的网络(VGG)

<script type="text/javascript" src="/js/src/bai.js"></script>

VGG:更大更深、VGG块
![VGG](http://image.xpshuai.cn/20220915190047.png)



**VGG块**的组成规律是：连续重复使用数个相同的
- 填充为1、窗口形状为3×3的卷积层后(3*3卷积比5*5卷积好)
- 接上一个步幅为2、窗口形状为2×2的最大池化层。
卷积层保持输入的高和宽不变，而池化层则对其减半


VGG架构：
![](http://image.xpshuai.cn/20220915190502.png)





```python

import time
import torch
from torch import nn, optim



def vgg_block(num_convs,in_channels,out_channels):
    """定义vgg块

    Args:
        num_convs (int): 需要卷积层数量
        in_channels (int): 输入通道数
        out_channels (int): 输出通道数
    """
    layers = []
    for _ in range(num_convs):
        layers.append(
            nn.Conv2d(in_channels,out_channels,kernel_size=3,padding=1)
        )
        layers.append(nn.ReLU())
        in_channels = out_channels
    layers.append(nn.MaxPool2d(kernel_size=2,stride=2))
    return nn.Sequential(*layers)



# 实现vgg架构,vgg含有5个vgg块
conv_arch = (
    (1,64),
    (1,128),
    (2,256),
    (2,512),
    (2,512),
)



def vgg(conv_arch):
    conv_blks = []
    in_channels = 1
    for (num_convs,out_channels) in conv_arch:
        conv_blks.append(
            vgg_block(num_convs,in_channels,out_channels)
        )
        in_channels = out_channels
    net = nn.Sequential(
        *conv_blks,
        nn.Flatten(),
        nn.Linear(out_channels*7*7,4096),nn.ReLU(),nn.Dropout(0.5),
        nn.Linear(4096,4096),nn.ReLU(),nn.Dropout(0.5),
        nn.Linear(4096,10)
    )
    return net


net = vgg(conv_arch)


# 经典设计模式，每次图片大小缩小一半 通道数增加一杯
X = torch.randn(size=(1, 1, 224, 224))
for blk in net:
    X = blk(X)
    print(blk.__class__.__name__,'output shape:\t',X.shape)



# 这里是作为演示，因为VGG-11比AlexNet计算量更大，因此我们构建一个通道数较少的神经网络
ratio = 4
small_conv_arch = [(pair[0], pair[1] // ratio) for pair in conv_arch]
print(small_conv_arch)
net = vgg(small_conv_arch)



# X = torch.randn(size=(1, 1, 224, 224))
# for blk in net:
#     X = blk(X)
#     print(blk.__class__.__name__,'output shape:\t',X.shape)


lr, num_epochs, batch_size = 0.05, 10, 128
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size, resize=224)
d2l.train_ch6(net, train_iter, test_iter, num_epochs, lr, d2l.try_gpu())
```


总结：
- VGG使用可重复使用的卷积块来构建深度卷积神经网络
- 不同的卷积块个数个超参数可以得到不同复杂度的变种


> 我电脑已经跑不动了，菜的人只能看着理论......









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E7%BB%8F%E5%85%B8%E7%8E%B0%E4%BB%A3%E5%8D%B7%E7%A7%AF%E7%BD%91%E7%BB%9C2-%E4%BD%BF%E7%94%A8%E9%87%8D%E5%A4%8D%E5%9D%97%E7%9A%84%E7%BD%91%E7%BB%9C-vgg/  

