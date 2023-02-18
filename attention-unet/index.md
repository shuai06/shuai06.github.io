# Attention UNet

<script type="text/javascript" src="/js/src/bai.js"></script>

## 论文简介
论文地址：https://arxiv.org/abs/1804.03999

出自于MIDL2018(深度学习医学影像会议), 论文中提出了一种应用于医学影像的基于`attention gate`的模型。
Attention UNet使用了标准的 UNet的网络架构，并在UNet的基础上中引入注意力机制，在对编码器每个分辨率上的特征与解码器中对应特征进行拼接之前，使用了一个注意力模块，重新调整了编码器的输出特征。该模块生成一个门控信号，用来控制不同空间位置处特征的重要性。
attention模块用在了skip connection上，原始U-Net只是单纯的把同层的下采样层的特征直接concate到上采样层中，改进后的使用attention模块对下采样层同层和上采样层上一层的特征图进行处理后再和上采样后的特征图进行concate



## 网络架构
下图即其网络架构，其中红色的部分就是`注意力block`
![](http://image.geoer.cn/20221010103858.png)

与标准UNet整体结构相似，不同的是在红框内增加了attention gate。
在decoder时候，从encoder提取的部分进行了attention gate再进行decoder。
在对 encoder 每个分辨率上的特征与 decoder 中对应特征进行拼接之前，使用了一个AGs，重新调整了encoder的输出特征。该模块生成一个门控信号，用来控制不同空间位置处特征的重要性

有两个输入：
- `x`:来自浅层网络部分(跳跃连接的输入)
- `g`:来自深层网络部分(前一个block的输入)


通过跳跃连接合并多个比例提取的特征图，以合并组粒度和细粒度的密集预测。粗粒度的特征图会捕获上下文信息，并突出显示前景对象的类别和位置；AG抑制了无关背景区域中的热证响应，而无需在网络之间裁剪ROI。


**下面看一下attention gated:**
![](http://image.geoer.cn/20221010104402.png)
对应前面整体网络结构中的：
![](http://image.geoer.cn/20221010104427.png)


**attention gate的过程：**
1. x和g都被送入到1x1卷积中，将它们变为相同数量的通道数，而不改变大小
2. 在上采样操作后(有相同的大小)，他们被累加并通过ReLU
3. 通过另一个1x1的卷积和一个sigmoid，得到一个0到1的重要性分数$\alpha$，分配给特征图的每个部分
4. 然后用这个注意力图乘以skip输入，产生这个注意力块的最终输出

其中，Attention coefficients(注意力系数)取值范围为0~1, 
Attention coefficients倾向于在目标器官区域取得大的值，在背景区域取得较小的值，有助于提高图像分割的精度。


## pytorch实现
> 代码还是有一点问题，先存这里，有问题回来修改


model.py
```python
from torch import nn
from torch.nn import functional as F
import torch
from torchvision import models
import torchvision

class conv_block(nn.Module):  # 形状没有发生变化
    def __init__(self,ch_in,ch_out):
        super(conv_block,self).__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(ch_in, ch_out, kernel_size=3,stride=1,padding=1,bias=True),
            nn.BatchNorm2d(ch_out),
            nn.ReLU(inplace=True),
            nn.Conv2d(ch_out, ch_out, kernel_size=3,stride=1,padding=1,bias=True),
            nn.BatchNorm2d(ch_out),
            nn.ReLU(inplace=True)
        )
    def forward(self,x):
        x = self.conv(x)
        return x

class up_conv(nn.Module):   # 上采样：扩大两倍
    def __init__(self,ch_in,ch_out):
        super(up_conv,self).__init__()
        self.up = nn.Sequential(
            nn.Upsample(scale_factor=2),
            nn.Conv2d(ch_in,ch_out,kernel_size=3,stride=1,padding=1,bias=True),
		    nn.BatchNorm2d(ch_out),
			nn.ReLU(inplace=True)
        )

    def forward(self,x):
        x = self.up(x)
        return x

class Attention_block(nn.Module):
    # self.Att5 = Attention_block(F_g=512, F_l=512, F_int=256)
    def __init__(self, F_g, F_l, F_int):
        super(Attention_block, self).__init__()
        self.W_g = nn.Sequential(
            nn.Conv2d(F_g, F_int, kernel_size=1, stride=1, padding=0, bias=True),
            nn.BatchNorm2d(F_int)
        )

        self.W_x = nn.Sequential(
            nn.Conv2d(F_l, F_int, kernel_size=1, stride=1, padding=0, bias=True),
            nn.BatchNorm2d(F_int)
        )

        self.psi = nn.Sequential(
            nn.Conv2d(F_int, 1, kernel_size=1, stride=1, padding=0, bias=True),
            nn.BatchNorm2d(1),
            nn.Sigmoid()
        )

        self.relu = nn.ReLU(inplace=True)

    def forward(self, g, x):
        # 下采样的gating signal 卷积
        g1 = self.W_g(g)
        # 上采样的 l 卷积
        x1 = self.W_x(x)
        # concat + relu
        psi = self.relu(g1 + x1)
        # channel 减为1，并Sigmoid,得到权重矩阵
        psi = self.psi(psi)
        # 返回加权的 x
        return x * psi

class AttentionUnet(nn.Module):
    def __init__(self, img_ch=3, output_ch=1):
        super(AttentionUnet, self).__init__()

        self.Maxpool = nn.MaxPool2d(kernel_size=2, stride=2)

        self.Conv1 = conv_block(ch_in=img_ch, ch_out=64)
        self.Conv2 = conv_block(ch_in=64, ch_out=128)
        self.Conv3 = conv_block(ch_in=128, ch_out=256)
        self.Conv4 = conv_block(ch_in=256, ch_out=512)
        self.Conv5 = conv_block(ch_in=512, ch_out=1024)

        self.Up5 = up_conv(ch_in=1024, ch_out=512)
        self.Att5 = Attention_block(F_g=512, F_l=512, F_int=256)
        self.Up_conv5 = conv_block(ch_in=1024, ch_out=512)

        self.Up4 = up_conv(ch_in=512, ch_out=256)
        self.Att4 = Attention_block(F_g=256, F_l=256, F_int=128)
        self.Up_conv4 = conv_block(ch_in=512, ch_out=256)

        self.Up3 = up_conv(ch_in=256, ch_out=128)
        self.Att3 = Attention_block(F_g=128, F_l=128, F_int=64)
        self.Up_conv3 = conv_block(ch_in=256, ch_out=128)

        self.Up2 = up_conv(ch_in=128, ch_out=64)
        self.Att2 = Attention_block(F_g=64, F_l=64, F_int=32)
        self.Up_conv2 = conv_block(ch_in=128, ch_out=64)

        self.Conv_1x1 = nn.Conv2d(64, output_ch, kernel_size=1, stride=1, padding=0)
        self.sigmoid = nn.Sigmoid()
    def forward(self, x):
        # encoding path
        x1 = self.Conv1(x)   # [64,512,512]。[3,224,224]-->[64,224,224]    只是通道数改变(self.Conv1改变)，形状不变

        x2 = self.Maxpool(x1) # [64,256,256]。[64,224,224]-->[64,112,112]   通道数不变，形状缩小一倍
        x2 = self.Conv2(x2)  # [128,256,256]。  [64,112,112]-->[128,112,112]

        x3 = self.Maxpool(x2) # [128,128,128]。[128,112,112]-->[128,56,56]
        x3 = self.Conv3(x3)  # [256,128,128]。[128,56,56]-->[256,56,56]

        x4 = self.Maxpool(x3) # [256,64,64]。[256,56,56]-->[256,28,28]
        x4 = self.Conv4(x4)  # [512,64,64]。[256,28,28]-->[512,28,28]

        x5 = self.Maxpool(x4) # [512,32,32]。[512,28,28]-->[512,14,14]
        x5 = self.Conv5(x5)  # [1024,32,32]。[512,14,14]-->[1024,14,14]

        # decoding + concat path
        d5 = self.Up5(x5)    # [512,64,64]。[1024,14,14]-->[512,28,28] 形状扩大2，且通道数增加
        x4 = self.Att5(g=d5, x=x4)  # d5是[512,64,64],x4是[512,64,64]。我这里d5是[512,28,28],x4是[512,28,28]--->注意力机制之后输出x4也为[512,28,28]
        d5 = torch.cat((x4, d5), dim=1)  # [1024,64,64]。x4为[512,28,28],d5为[512,28,28]--->拼接之后为：[1024,28,28]
        d5 = self.Up_conv5(d5)  # [512,64,64]。通道变为，形状不变[512,28,28]

        d4 = self.Up4(d5)    # [256,128,128]。形状扩大2，且通道数变小：[256,56,56]
        x3 = self.Att4(g=d4, x=x3)  # [256,128,128],[256,128,128]。[256,56,56],[256,56,56]
        d4 = torch.cat((x3, d4), dim=1)  # [512,128,128]。拼接完用倒数变量：[512,56,56]
        d4 = self.Up_conv4(d4)  # [256,128,128]。[512,56,56]-->[256,56,56]

        d3 = self.Up3(d4)    # [128,256,256]。[256,56,56]-->[128,112,112]
        x2 = self.Att3(g=d3, x=x2)  # [128,256,256],[128,256,256]。[128,112,112],[128,112,112]
        d3 = torch.cat((x2, d3), dim=1)  # [256,256,256]。[128,112,112]-->[256,112,112]
        d3 = self.Up_conv3(d3)  # [128,256,256]。[128,112,112]

        d2 = self.Up2(d3)    # [64,512,512]。通道数减少，形状变大2倍：[64,224,224]
        x1 = self.Att2(g=d2, x=x1)  # [64,512,512],[64,512,512]。[64,224,224],[64,224,224]
        d2 = torch.cat((x1, d2), dim=1)  # [128,512,512]。[128,224,224]
        d2 = self.Up_conv2(d2)  # [64,512,512]。[64,224,224]

        d1 = self.Conv_1x1(d2)  # [2,512,512]。输出通道变为2:[2,224,224]
        d1 = self.sigmoid(d1)   # [2,512,512]。[2,224,224]

        return d1
```

data_set.py
```python
# 导入库
import os 
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F
from torch import optim
from torch.utils.data import Dataset, DataLoader, random_split
from tqdm import tqdm
import warnings
warnings.filterwarnings("ignore")
import os.path as osp
import matplotlib.pyplot as plt
from PIL import Image
import numpy as np
from torchvision import transforms



transform = transforms.Compose([transforms.ToTensor(),
                                            ])
# 自定义数据集CamVidDataset
class RSDataset(torch.utils.data.Dataset):
    """
    Args:
        images_dir (str): path to images folder
        masks_dir (str): path to segmentation masks folder
        class_values (list): values of classes to extract from segmentation mask
        augmentation (albumentations.Compose): data transfromation pipeline 
            (e.g. flip, scale, etc.)
        preprocessing (albumentations.Compose): data preprocessing 
            (e.g. noralization, shape manipulation, etc.)
    """
    
    def __init__(self, images_dir, masks_dir):
        self.trans = transforms.Compose([
            transforms.Resize([224,224]),
            transforms.ToTensor()
        ])
        
        # self.resize = transforms.Resize([100,100])
        self.ids = os.listdir(images_dir)
        self.images_fps = [os.path.join(images_dir, image_id) for image_id in self.ids]
        self.masks_fps = [os.path.join(masks_dir, image_id) for image_id in self.ids]
 
    
    def __getitem__(self, i):
        image = Image.open(self.images_fps[i])
        mask = Image.open(self.masks_fps[i]).convert("RGB")
        image = transform(image)
        mask = transform(mask)
        mask = mask[0:2, :, :]  # 通道数变为2
        return image, mask
        
    def __len__(self):
        return len(self.ids)


```


train.py
```python
from audioop import cross
import torch.nn as nn
import d2l
from model import AttentionUnet
from torch import optim
import torch
import numpy as np
from d2l import torch as d2l
from tqdm import tqdm
import pandas as pd
from data import RSDataset
from torch.utils.data import Dataset, DataLoader, random_split
import os
import matplotlib.pyplot as plt
from torchvision.utils import save_image
import visdom




def draw_loss(n_epochs, losses):
    epochs_range = range(n_epochs)
    plt.plot(epochs_range, losses, '-g', label='train loss')
    plt.title('loss')
    plt.legend()
    plt.savefig('figure/loss_figure.png')
    
    
    

def train_net(model,
              n_epochs,
              batch_size,
              # weight,
              train_loader,
              optimizer,
              checkpoint_dir='weights',
              lr=1e-4):

    # Model on cuda
    if torch.cuda.is_available():
        model = model.cuda()

    print('''
    Starting training:
        Epochs: {}
        Batch size: {}
        Learning rate: {}
        Training size: {}
    '''.format(n_epochs, batch_size, lr, train_dataset.__len__()))

    # optimizer = torch.optim.Adam(model.parameters(), lr=lr, weight_decay=0.0005)
    # criterion = nn.NLLLoss(weight=weight)
    
    # criterion = lossf
    module_save = "model_save/module_attunet.pkl"
    
    # if os.path.exists(module_save):
    #     net.load_state_dict(torch.load(module))
    #     print('module is loaded !')
    
    epoch_losses = []  # 暂存
    #损失函数选用多分类交叉熵损失函数
    # criterion = nn.CrossEntropyLoss()
    criterion = nn.BCELoss()
    for epoch in range(n_epochs):
        for i, (image, label) in enumerate(train_loader):
            image, label = image.cuda(), label.cuda()
            # print("image.shape")
            # print(image.shape)
            image_heat = model(image)
            # print("image_heat.shape")
            print(image_heat.shape)
            # print("label.shape")
            print(label.shape)
            print(type(label))
            # print(torch.squeeze(label).shape)
            #loss = criterion(image_heat, torch.squeeze(label))
            loss = criterion(image_heat, label)
            
            # compute gradient and do SGD step
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            
            # img = torch.stack([image[0], image_heat[0], label[0]], dim=0)  # 将以上三张图进行拼接
            # print(img.shape)
            save_image(image_heat[0], f'out_tmp_img/{i}.png')
        
            # if i % 10 == 0:
            epoch_losses.append(loss.item())
            print(f'epoch: 第{str(epoch+1)}/共{str(n_epochs)}  batchs: {str(i)}  loss: {loss.item()}')
            torch.save(model.state_dict(), module_save)
        scheduler.step()
        avg_loss = np.mean(epoch_losses)
        print(f"第{epoch+1}轮平均loss--->{avg_loss}")
        # wind.line(avg_loss, epoch,win="train_loss", update='append')

    # # save loss figure
    # # draw_loss(n_epochs, losses,)

    # # save loss data
    # with open('loss/loss.txt', 'w') as loss_file:
    #     loss_file.write('train loss:\n')
    #     for i, loss in enumerate(losses):
    #         output = '{' + str(i) + '}: {' + str(loss) + '}\n'
    #         loss_file.write(output)
    #     loss_file.write('-' * 50)
    #     loss_file.write('\n')
            


if __name__ == '__main__':

    model = AttentionUnet(output_ch=2).cuda()   # 类别+1(背景)?????
    #model.load_state_dict(torch.load(r"checkpoints/Unet_100.pth"),strict=False)
     
    # 设置数据集路径
    # DATA_DIR = r'/home/xps/code/RemoteAICode/pytorch_segement/AttentionUnetTest/dataset/'  # 根据自己的路径来设置
    DATA_DIR = r'/root/code/AttentionUnetTest/dataset/'
    img_dir = os.path.join(DATA_DIR, 'JPEGImages')
    msk_dir = os.path.join(DATA_DIR, 'SegmentationClass')
        
    train_dataset = RSDataset(img_dir,  msk_dir)
    train_loader = DataLoader(train_dataset, batch_size=1, shuffle=True,drop_last=True)
    
    
    #选用adam优化器来训练 
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-4, weight_decay=0.0005)
    scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=1, gamma=0.1, last_epoch=-1)


    #训练20轮
    epochs_num = 50    
    
    # torch.cuda.set_device(1)
    train_net(model=model,
              n_epochs=epochs_num,
              train_loader=train_loader,
              optimizer=optimizer,
              batch_size=1)
              # weight=torch.FloatTensor([0.2, 15, 15]).cuda())
              
    torch.cuda.empty_cache()
       
              
 
    
```






## 参考
https://blog.csdn.net/weixin_41693877/article/details/108395270
https://blog.csdn.net/qq_31622015/article/details/90701328#commentBox
https://blog.csdn.net/qq_43426908/article/details/123755654?


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/attention-unet/  

