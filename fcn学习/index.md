# FCN学习

<script type="text/javascript" src="/js/src/bai.js"></script>

## FCN(全卷积神经网络, Fully Convolutional Networks for Semantic Segmentation)
> 首个端对端的针对像素级预测的全卷积网络
论文地址：论文：https://arxiv.org/abs/1411.4038

FCN是图像语义分割的开山之作，这是一篇发表在2015 CVPR上的一篇论文，拿到了当年的best paper honorable mention。

传统的CNN分类网络，只能标识整个图片的类别，不能标识每个像素点的类别



### 语义分割
常见的分类任务：
● 语义分割：对**像素**进行分类。经典网络有FCN
● 实例分割：不同实例也用不同颜色。经典网络有Mask R-CNN
● 全景分割：除了目标分割出来，还要把背景分割出来。经典网络有Panoptic FPN






### FCN核心思想
比较简单，其实现存的指明的分类模型都可以转化为FCN模型，  
将传统的CNN拿过来，然后将最后的全连接和全局池化层去掉，替换为一个转置卷积层(如果前面卷积层让图片缩小了32倍，这里转置卷积层就放大32倍)，从而实现每个像素的预测 （输出空间映射而不是分类的分数）
其中通道数就是类别的个数+1，当前前面可以加一个1x1卷积来降低运算量(降低维度)
![](http://image.xpshuai.cn/20221003111119.png)


全卷积⽹络先使⽤卷积神经⽹络抽取图像特征，然后通过1×1卷积层将通道数变换为类别个数，最后通过转置卷积层将特征图的⾼和宽变换为输⼊图像的尺⼨。



### FCN网络架构
**大致步骤：**
1. 将一个用于分类的卷积网络（比如VGG16, VGG系列对于提取特征十分有效）转换为全卷积的，也就是将全连接层进行卷积化，得到一个热图（heatmap）
2. 通过上采样将热图恢复到输入图片的尺寸得到pixelwise predictions。然后以最大概率为依据逐个像素进行分类。loss就是所有像素上的softmax loss之和。
3. 添加skip connections来将浅层精细的语义信息和深层粗糙的语义信息融合起来。也就是解决上面提到的“位置和语义的tension”

  

    
FCN正向编码网络结构：
![](http://image.xpshuai.cn/20221003111759.png)




FCN反向编码网络结构：
![](http://image.xpshuai.cn/20221003111902.png)



### FCN-32s
![](http://image.xpshuai.cn/20221003112244.png)



### FCN-16s
![](http://image.xpshuai.cn/20221003112257.png)



### FCN-8s
![](http://image.xpshuai.cn/20221003112316.png)



## 代码实现举例

```python
# 输出的类别预测与输⼊图像在像素级别上具有⼀⼀对应关系：通道维的输出即该位置对应像素的类别预测。


%matplotlib inline
import torch
import torchvision
from torch import nn
from torch.nn import functional as F
from d2l import torch as d2l


# 我 们 使 ⽤ 在ImageNet数 据 集 上 预 训 练 的ResNet-18模 型 来 提 取 图 像 特 征， 并 将 该 ⽹ 络 记为pretrained_net。ResNet-18模型的最后⼏层包括全局平均汇聚层和全连接层，然⽽全卷积⽹络中不需要它们。
pretrained_net = torchvision.models.resnet18(pretrained=True)
list(pretrained_net.children())[-3:]


# 接下来，我们创建⼀个全卷积⽹络net。它复制了ResNet-18中⼤部分的预训练层，去除了最后的全局平均汇聚层和最接近输出的全连接层。
net = nn.Sequential(*list(pretrained_net.children())[:-2])




# 接下来，我们使⽤1 × 1卷积层将输出通道数转换为Pascal VOC2012数据集的类数（21类）。最后，我们需要将
# 特征图的⾼度和宽度增加32倍，从⽽将其变回输⼊图像的⾼和宽。

num_classes = 21
net.add_module('final_conv', nn.Conv2d(512, num_classes, kernel_size=1))
net.add_module('transpose_conv', nn.ConvTranspose2d(num_classes, num_classes,
kernel_size=64, padding=16, stride=32))
# 这里的转置卷积：一般卷积下采样多少，我们转置卷积就上采样多少
# 我们可以看到如果步幅为s，填充为s/2（假设s/2是整数）且卷积核的⾼和宽为2s，转置卷积核会将输⼊的⾼和宽分别放⼤s倍


# 初始化转置卷积层，双线性插值的上采样可以通过转置卷积层实现
def bilinear_kernel(in_channels, out_channels, kernel_size):
    factor = (kernel_size + 1) // 2
    if kernel_size % 2 == 1:
        center = factor - 1
    else:
        center = factor - 0.5
    og = (torch.arange(kernel_size).reshape(-1, 1),
        torch.arange(kernel_size).reshape(1, -1))
    filt = (1 - torch.abs(og[0] - center) / factor) *  (1 - torch.abs(og[1] - center) / factor)
    weight = torch.zeros((in_channels, out_channels,
        kernel_size, kernel_size))
    weight[range(in_channels), range(out_channels), :, :] = filt
    return weight



# 读取图像X，将上采样的结果记作Y。为了打印图像，我们需要调整通道维的位置。
img = torchvision.transforms.ToTensor()(d2l.Image.open('../img/catdog.jpg'))
X = img.unsqueeze(0)
Y = conv_trans(X)
out_img = Y[0].permute(1, 2, 0).detach()

d2l.set_figsize()
print('input image shape:', img.permute(1, 2, 0).shape)
d2l.plt.imshow(img.permute(1, 2, 0));
print('output image shape:', out_img.shape)
d2l.plt.imshow(out_img);


# 在全卷积⽹络中，我们⽤双线性插值的上采样初始化转置卷积层。对于1 × 1卷积层，我们使⽤Xavier初始化参数。
# W = bilinear_kernel(num_classes, num_classe, 64)
# net.transpose_conv.weight.data.copy_(W);


# 读取数据集
batch_size, crop_size = 32, (320, 480)
train_iter, test_iter = d2l.load_data_voc(batch_size, crop_size)


# 训练
def loss(inputs, targets):
    return F.cross_entropy(inputs, targets, reduction='none').mean(1).mean(1)
num_epochs, lr, wd, devices = 5, 0.001, 1e-3, d2l.try_all_gpus()
trainer = torch.optim.SGD(net.parameters(), lr=lr, weight_decay=wd)
d2l.train_ch13(net, train_iter, test_iter, loss, trainer, num_epochs, devices)

# 预测
def predict(img):
    X = test_iter.dataset.normalize_image(img).unsqueeze(0)
    pred = net(X.to(devices[0])).argmax(dim=1)
    return pred.reshape(pred.shape[1], pred.shape[2])

def label2image(pred):
    colormap = torch.tensor(d2l.VOC_COLORMAP, device=devices[0])
    X = pred.long()
    return colormap[X, :]



voc_dir = d2l.download_extract('voc2012', 'VOCdevkit/VOC2012')
test_images, test_labels = d2l.read_voc_images(voc_dir, False)
n, imgs = 4, []
for i in range(n):
    crop_rect = (0, 0, 320, 480) X = torchvision.transforms.functional.crop(test_images[i], *crop_rect)
    pred = label2image(predict(X))
    imgs += [X.permute(1,2,0), pred.cpu(),
    torchvision.transforms.functional.crop(
            test_labels[i], *crop_rect).permute(1,2,0)]
d2l.show_images(imgs[::3] + imgs[1::3] + imgs[2::3], 3, n, scale=2);




```



## Note
- **backbone**：骨干网络，意指特征提取网络，一般就是使用vgg, resnett,mobilenet是常用的，可以说目标识别网络softmax之前的部分都以用来作为backbone

- **Bottleneck(瓶颈)**
![](http://image.xpshuai.cn/20221003112508.png)







---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/fcn%E5%AD%A6%E4%B9%A0/  

