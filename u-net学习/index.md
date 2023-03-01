# U-Net学习

<script type="text/javascript" src="/js/src/bai.js"></script>

> 文献地址：https://arxiv.org/pdf/1505.04597v1.pdf

## 简介&网络结构
UNet是医学分割领域经典论文，其结构与字母`U`类似而得名，是一个Encoder-Decoder结构，可以看到下图，左侧是Encoder部分(下采样，特征提取,缩小图片尺寸)，右侧是Decoder部分(上采样,扩大图片尺寸)。
![unet网络](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220921095308.png)
说明几点先：
1. 下采样过程中，特征图缩小的尺寸是上一层的一半；而在上采样过程中特征图变为上一层的一倍。通道数量变化相反
2. 下采样的时候卷积层特征尺寸变化小，原论文使用max pooling进行尺度缩小；上采样也一样，使用upsampling+conv进行尺度增大。
3. 然后，结构图中的灰色箭头(copy and crop)目的是将浅层特征与深层**特征融合**(其实就是使用cat进行`拼接`)，这样可以既保留浅层特征图中较高精度的特征信息，也可以利用深层特征图中抽象的语义信息。（左边的特征层(浅层特征)进行中心裁剪拼接到右边(深层特征)）


但是现在主流的实现方式并不是按照原论文实现的，而**主流的实现方式**是：
- 在3x3的卷积层加上一个`padding`(也就是每次经过这个3x3的卷积层的时候不会去改变特征层的高和宽)
- 在卷积和ReLU之间加`BatchNormalization`
这么做的**好处**：
假设是到上采样的时候，加上padding之后的形状不变，和左边是一样的，就不需要进行中心裁剪(仔细观察网络图可以发现)，直接进行拼接：
![20220921140459](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220921140459.png)


**另一个注意点：**
如果按照原论文给的网络，假设给出下图左边的区域，得到的只有右边的区域。原作者对于边缘位置损失像素的区域进行了镜像：
![20220921100740](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220921100740.png)

> 但是如果按照上面那种大小不变的情况，就不会出现边缘缺失数据的情况，当然也不会使用下面的overlap了
**为了得到好的效果(推荐做法)：**
对于大的高分辨率的影像，一般不整个放进去，而是分割开放进去，每个区域之间有一个`overlap`重叠区域，来得到较好效果：
![20220921100852](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220921100852.png)


**再一个注意点：**
语义分割，一般我们使用c作为GT，也就是前景色和背景色


细胞细胞之间不容易分割，所以细胞之间的背景区域分给更大的权重(右边图d有个热力图，红色代表权重越大)，损失的权重也更大一些；其他大面积的背景区域分配较小的权重
![20220921101920](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220921101920.png)


## 实现细节
```python
import torch as t
from torch import nn

## 下采样的基本结构
# 参考：https://github.com/JavisPeng/u_net_liver/blob/a1b9553d8ba8c6e5a3d4c5fabd387e130e60a072/dataset.py#L16
# 常用的两个卷积操作简单封装
class DoubleConv(nn.Module):
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.conv_down = nn.Sequential(
            # 主流实现方式
            # - 在3x3的卷积层加上一个padding(也就是每次经过这个3x3的卷积层的时候不会去改变特征层的高和宽)
            # - 在卷积核ReLU之间加BatchNormalization
            nn.Conv2d(in_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(in_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )
        
    def forward(self, input):
        return self.conv_down(input)


# U-Net网络
class Unet(nn.Module):
    def __init__(self, in_ch, out_ch):
        super().__init__()
        # 下采样
        self.conv1 = DoubleConv(in_ch, 64)
        self.pool1 = nn.MaxPool2d(2)
        self.conv2 = DoubleConv(64, 128)
        self.pool2 = nn.MaxPool2d(2)
        self.conv3 = DoubleConv(128, 256)
        self.pool3 = nn.MaxPool2d(2)
        self.conv4 = DoubleConv(256, 512)
        self.pool4 = nn.MaxPool2d(2)
        self.conv5 = DoubleConv(512, 1024)
        # 上采样
        # 转置卷积(有参数可以训练)可以实现上采样，也可以使用Upsample上采样(通过插值完成，没有训练参数，速度更快)(保证k=stride,stride即上采样倍数)
        self.up6 = nn.ConvTranspose2d(1024, 512, 2, stride=2)
        self.conv6 = DoubleConv(1024, 512)
        self.up7 = nn.ConvTranspose2d(512, 256, 2, stride=2)
        self.conv7 = DoubleConv(512, 256)
        self.up8 = nn.ConvTranspose2d(256, 128, 2, stride=2)
        self.conv8 = DoubleConv(256, 128)
        self.up9 = nn.ConvTranspose2d(128, 64, 2, stride=2)
        self.conv9 = DoubleConv(128, 64)
        self.conv10 = nn.Conv2d(64, out_ch, 1)

    def forward(self, x):
        c1 = self.conv1(x)
        p1 = self.pool1(c1)
        c2 = self.conv2(p1)
        p2 = self.pool2(c2)
        c3 = self.conv3(p2)
        p3 = self.pool3(c3)
        c4 = self.conv4(p3)
        p4 = self.pool4(c4)
        c5 = self.conv5(p4)
        up_6 = self.up6(c5)
        merge6 = t.cat([up_6, c4], dim=1)  # cat 拼接
        c6 = self.conv6(merge6) # 对拼接之后的继续向上进行卷积
        up_7 = self.up7(c6)
        merge7 = t.cat([up_7, c3], dim=1)
        c7 = self.conv7(merge7)
        up_8 = self.up8(c7)
        merge8 = t.cat([up_8, c2], dim=1)
        c8 = self.conv8(merge8)
        up_9 = self.up9(c8)
        merge9 = t.cat([up_9, c1], dim=1)
        c9 = self.conv9(merge9)
        c10 = self.conv10(c9)
        out = nn.Sigmoid()(c10)
        return out

```

看一下：
```python
unet = Unet(1,10)
print(unet)
```

















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/u-net%E5%AD%A6%E4%B9%A0/  

