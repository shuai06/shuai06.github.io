# ResNet

<script type="text/javascript" src="/js/src/bai.js"></script>

## ResNet


ResNet沿用了VGG全3×3卷积层的设计。残差块里首先有2个有相同输出通道数的3×3卷积层。每个卷积层后接一个批量归一化层和ReLU激活函数。然后我们将输入跳过这两个卷积运算后直接加在最后的ReLU激活函数前。这样的设计要求两个卷积层的输出与输入形状一样，从而可以相加。如果想改变通道数，就需要引入一个额外的1×1卷积层来将输入变换成需要的形状后再做相加运算。


## 残差块

![20220916154717](https://image.geoer.cn/20220916154717.png)


**论文中的图：**
![20220920193516](https://image.geoer.cn/20220920193516.png)
1.恒等映射:
上图中有一个曲线，称为shortcut connection(快捷连接），就是恒等映射。既不增加额外参数，也不增加计算复杂度。而恒等映射表示：输入=输出；
2.图中F（x）的作用：
图中F（x）执行卷积操作，目的是提取图片中的更多特征，或者是其它层没有学习的特征；

**理解：**
>残差网络提出的目的就是为了减少梯度消失，更多的提取特征。
假设我们输出目标是H(x),通过上图操作，实际输出是F（x）+x，因此F（x）=H(x)-x就是我们的目标残差。换个角度思考，假设网络模型优化的很好，通过卷积和池化已经提取不了特征，那么网络中F（x）=0，则残差块输入等于输出，这时候就变成恒等映射。很多利用残差块叠加深层神经网络，即使层数很多，但是运行时间没有很大提升，这是因为前面的残差块已经提出足够的特征，后面的残差块没有特征提取变成恒等映射，故网络模型运行时间没有显著提升。
通过优化使H（x）-x=F（x）无限逼近于0，当F（x）已经成为0时，后续操作无须进行，跳过执行，降低复杂度（恒等映射）。



```python
## 残差块的实现

import time
import torch
from torch import nn, optim
import torch.nn.functional as F
from d2l import d2ltorch as d2l


device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# 第一步：定义残差块
class Residual(nn.Module): 
    def __init__(self, in_channels, out_channels, use_1x1conv=False, stride=1):
        super(Residual, self).__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, padding=1, stride=stride)
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3, padding=1)
        if use_1x1conv:
            self.conv3 = nn.Conv2d(in_channels, out_channels, kernel_size=1, stride=stride)
        else:
            self.conv3 = None
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.relu = nn.ReLU(inplace=True)

    def forward(self, X):
        Y = F.relu(self.bn1(self.conv1(X)))
        Y = self.bn2(self.conv2(Y))
        if self.conv3:
            X = self.conv3(X)
        return F.relu(Y + X)



```



## 残差网络

ResNet的前两层跟之前介绍的GoogLeNet中的一样：
在输出通道数为64、步幅为2的7×7卷积层后接步幅为2的3×3的最大池化层。
不同之处在于ResNet每个卷积层后增加的批量归一化层。

GoogLeNet在后面接了4个由Inception块组成的模块。ResNet则使用4个由残差块组成的模块，每个模块使用若干个同样输出通道数的残差块。
第一个模块的通道数同输入通道数一致。由于之前已经使用了步幅为2的最大池化层，所以无须减小高和宽。
之后的每个模块在第一个残差块里将上一个模块的通道数翻倍，并将高和宽减半。

ResNet架构：
![20220916160728](https://image.geoer.cn/20220916160728.png)

> 一般就用ResNet-34比较多(34个卷积层)或者50，很少用特别大的



![20220920191741](https://image.geoer.cn/20220920191741.png)


```python
# 最前面跟GoogleNet的b1是一样的
net = nn.Sequential(
        nn.Conv2d(1, 64, kernel_size=7, stride=2, padding=3),
        nn.BatchNorm2d(64), 
        nn.ReLU(),
        nn.MaxPool2d(kernel_size=3, stride=2, padding=1))



def resnet_block(in_channels, out_channels, num_residuals, first_block=False):
    if first_block:  # 第一个要特殊处理
        assert in_channels == out_channels # 第一个模块的通道数同输入通道数一致
    blk = []
    for i in range(num_residuals):
        # 需要注意first_block已经做了3*3的最大池化，所以没必要做变换.第二层，第三层，第四层跳跃连接时，维度不同，需要先经过1*1卷积变换再相加。
        if i == 0 and not first_block:  
            # 每一个block有2个esidual，每一个Residual有2个卷积层
            blk.append(Residual(in_channels, out_channels, use_1x1conv=True, stride=2))  # 减半
        else:
            blk.append(Residual(out_channels, out_channels))
    return nn.Sequential(*blk)


# 为ResNet加入所有残差块
net.add_module("resnet_block1", resnet_block(64, 64, 3, first_block=True))  # 第一个高宽不变
net.add_module("resnet_block2", resnet_block(64, 128, 4)) # 下面三个：重复两个block，通道数加倍，高宽减半
net.add_module("resnet_block3", resnet_block(128, 256, 6)) # 自己可以设置每个里面有多少个block，这里设置的是2
net.add_module("resnet_block4", resnet_block(256, 512, 3))


# 最后，与GoogLeNet一样，加入全局平均池化层后接上全连接层输出。
net.add_module("global_avg_pool", nn.AdaptiveAvgPool2d((1,1))) # GlobalAvgPool2d的输出: (Batch, 512, 1, 1)
net.add_module("fc", nn.Sequential(nn.Flatten(), nn.Linear(512, 10)))   # 展开，全连接层


# 这里每个模块里有4个卷积层（不计算1×11×1卷积层），
# 加上最开始的卷积层和最后的全连接层，共计18层。
# 这个模型通常也被称为ResNet-18
# 通过配置不同的通道数和模块里的残差块数可以得到不同的ResNet模型，例如更深的含152层的ResNet-152




# 来观察一下输入形状在ResNet不同模块之间的变化。
X = torch.rand((1, 1, 224, 224))
for name, layer in net.named_children():
    X = layer(X)
    print(name, ' output shape:\t', X.shape)


"""
0  output shape:	 torch.Size([1, 64, 112, 112])
1  output shape:	 torch.Size([1, 64, 112, 112])
2  output shape:	 torch.Size([1, 64, 112, 112])
3  output shape:	 torch.Size([1, 64, 56, 56])
resnet_block1  output shape:	 torch.Size([1, 64, 56, 56])
resnet_block2  output shape:	 torch.Size([1, 128, 28, 28])
resnet_block3  output shape:	 torch.Size([1, 256, 14, 14])
resnet_block4  output shape:	 torch.Size([1, 512, 7, 7])
global_avg_pool  output shape:	 torch.Size([1, 512, 1, 1])
fc  output shape:	 torch.Size([1, 10])
"""

```


训练：
```python

lr, num_epochs, batch_size = 0.001, 5, 256

# 调用沐神的第三放包里的训练和加载数据的方法
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size, resize=96)
optimizer = torch.optim.Adam(net.parameters(), lr=lr)
d2l.train_ch5(net, train_iter, test_iter, batch_size, optimizer, device, num_epochs)

```




## 使用nn.module实现
> 不知道对不对
```python
import time
import torch
from torch import nn, optim
import torch.nn.functional as F



 # 残差块
class ResidualBlock(nn.Module):
    def __init__(self,inchanel,outchanel,stride=1,shortcut=None):
        super(ResidualBlock,self).__init__()
        self.left=nn.Sequential(
            nn.Conv2d(inchanel,outchanel,3,stride,1,bias=False),
            nn.BatchNorm2d(outchanel),
            nn.ReLU(inplace=True),
            nn.Conv2d(outchanel,outchanel,3,1,1,bias=False),
            nn.BatchNorm2d(outchanel)
        )
        self.right=shortcut
    def forward(self, x):
        out=self.left(x)
        residual = x if self.right is None else self.right(x)
        out += residual



class ResNet34(nn.Module):
    """
    实现主module:ResNet34
    ResNet34包含多个layer，每个layer又包含多个residual block
    用子model实现residual block，用__make_layer__函数实现layer
    """
    def __init__(self,num_classes=10):
        """
        构建ResNet34网络的各层结构
        :param num_classes:
        """
        super(ResNet34,self).__init__()
        self.pre=nn.Sequential(
            nn.Conv2d(1, 64, kernel_size=7, stride=2, padding=3),
            nn.BatchNorm2d(64), 
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
            )
        self.layer1 = self.__make_layer__(64,128,3,stride=1, is_short=False)
        self.layer2 = self.__make_layer__(128,256,4,stride=2)
        self.layer3 = self.__make_layer__(256,512,6,stride=2)
        self.layer4 = self.__make_layer__(512,512,3,stride=2)
        self.fc = nn.Linear(512,num_classes)
 
    def __make_layer__(self,inchannel,outchannel,block_num,stride=1, is_short=True):
        """
        构建layer，包含多个residual block
        """
        if is_short:
            shortcut = nn.Sequential(
                nn.Conv2d(inchannel,outchannel,1,stride,bias=False),
                nn.BatchNorm2d(outchannel)
            )
        else:
            shortcut = None
            
        layers = []
        layers.append(ResidualBlock(inchannel,outchannel,stride,shortcut))
        for i in range(1,block_num):
            layers.append(ResidualBlock(inchannel,outchannel))
        return  nn.Sequential(*layers)
 
    def forward(self, x):
        x = self.pre(x)
 
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
 
        x = F.avg_pool2d(x,7)
        x = x.view(x.size(0),-1)
        return self.fc(x)


model = ResNet34()
print(model)
```












---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/resnet/  

