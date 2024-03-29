# 深度学习中常见名词


<!--more-->

> 学习了这么久了还是一个小白，容易弄混一些名词
  

**基于深度学习的现在目标检测算法中有三个组件：Backbone、Neck和Head:**   
### 1.backbone
翻译为**主干网络**的意思，既然说是主干网络，就代表其是网络的一部分，那么是哪部分呢？**主要指用于特征提取的，已在大型数据集(例如ImageNet|COCO等)上完成预训练，拥有预训练参数的卷积神经网络**，例如：ResNet-50、Darknet53等。  
这个主干网络大多时候指的是**提取特征**的网络，其作用就是提取图片中的信息，供后面的网络使用。这些网络经常使用的是resnet VGG等，而不是我们自己设计的网络，因为这些网络已经证明了在分类等问题上的特征提取能力是很强的。在用这些网络作为backbone的时候，都是直接加载官方已经训练好的模型参数，后面接着我们自己的网络。让网络的这两个部分同时进行训练，因为加载的backbone模型已经具有提取特征的能力了，**在我们的训练过程中，会对他进行微调，使得其更适合于我们自己的任务。**
  
### 2.head
head是获取网络输出内容的网络，利用之前提取的特征，head利用这些特征，做出预测。主要用于预测目标的种类和位置(bounding boxes)
  
### 3.neck
是放在backbone和head之间的，是为了更好的利用backbone提取的特征，会添加一些用于收集不同阶段中特征图的网络层
  
简而言之，基于深度学习的目标检测模型的结构是这样的：`输入->主干->脖子->头->输出`。主干网络提取特征，脖子提取一些更复杂的特征，然后头部计算预测输出
  

### 4.bottleneck
瓶颈的意思，通常指的是网络输入的数据维度和输出的维度不同，输出的维度比输入的小了许多，就像脖子一样，变细了。
经常设置的参数 bottle_num=256，指的是网络输出的数据的维度是256 ，可是输入进来的可能是1024维度的。
  

### 5.Embedding
深度学习方法都是利用使用线性和非线性转换对复杂的数据进行自动特征抽取，并将特征表示为“向量”（vector），这一过程一般也称为“嵌入”（embedding）
  
### 6.xx任务
用于预训练的任务被称为前置/代理任务(pretext task)，用于微调的任务被称为下游任务(downstream task)

  
### 7.Warm up
Warm up指的是用一个小的学习率先训练几个epoch，这是因为网络的参数是随机初始化的，一开始就采用较大的学习率容易数值不稳定。
  

### 8.end to end  
在论文中经常能遇到end to end这样的描述，那么到底什么是端到端呢？其实就是给了一个输入，我们就给出一个输出，不管其中的过程多么复杂，但只要给了一个输入，会对应一个输出。  
比如分类问题，你输入了一张图片，网络有特征提取，全连接分类，概率计算什么的，但是跳出算法问题，单从结果来看，就是给了一张输入，输出了一个预测结果。  
End-To-End的方案，即输入一张图，输出最终想要的结果，算法细节和学习过程全部丢给了神经网络。
    

持续更新中......



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E4%B8%AD%E5%B8%B8%E8%A7%81%E5%90%8D%E8%AF%8D/  

