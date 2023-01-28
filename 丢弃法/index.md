# 丢弃法(dropout)

<script type="text/javascript" src="/js/src/bai.js"></script>


>除了前一节介绍的权重衰减以外，深度学习模型常常使用丢弃法（dropout）来应对过拟合问题

标准处理流程：
- $ℎ=σ(𝑊1𝑥+𝑏1)$  第一个隐藏层
- $ℎ′=𝑑𝑟𝑜𝑝𝑜𝑢𝑡(ℎ)$  进行随机的删除
- $𝑜=𝑊2ℎ′+𝑏2$  第二层输出
- $𝑦=𝑠𝑜𝑓𝑡𝑚𝑎𝑥(𝑜)$  softmax层输出


- 在**测试模型**时，我们为了拿到更加确定性的结果，一般**不使用丢弃法**。




## 实现
```python
import torch
from torch import nn
from d2l import d2ltorch as d2l


# 定义丢弃函数
# 将以dropout的概率丢弃X中的元素
def dropout_layer(X,dropout):
    assert 0 <= dropout <=1 # 概率肯定是在0-1之间
    if dropout ==1:
        return torch.zeros_like(X)
    # 这种情况下把全部元素都丢弃
    if dropout ==0:
        return X
    # mask矩阵为0-1之间的均匀随机分布，大于dropout为1小于为0
    mask = (torch.randn(X.shape)>dropout).float()
    # 优先使用乘法而不是X[mask]=0，乘法速度远大于选取
    return mask*X/(1.0-dropout)

# 测试一下
# torch.tensor.uniform从均匀分布中抽样数值进行填充  
# torch.zeros(3,3).uniform_(0,1)

X= torch.arange(16, dtype = torch.float32).reshape((2, 8))
# print(X)
# print(dropout_layer(X, 0.))
print(dropout_layer(X, 0.5))
# print(dropout_layer(X, 1.))


# 定义具有两个隐藏层的多层感知机，每个隐藏层包含256个单元
num_inputs, num_outputs, num_hiddens1, num_hiddens2 = 784, 10, 256, 128
dropout1, dropout2 = 0.2, 0.5
class Net(nn.Module): # (object)写法是继承，自己回顾一下老男孩的课程
    def __init__(self,num_inputs,num_outputs,num_hiddens1,num_hiddens2,is_training = True):
        # 使用super方法可以重新调用父类中的函数
        # python3 直接写成 ： super().__init__()
        # python2 必须写成 ：super(本类名,self).__init__()
        super(Net,self).__init__()
        self.num_inputs = num_inputs
        self.training = is_training
        # 隐藏层的设置，都是线性层，注意做好层数的前后连接
        self.lin1 = nn.Linear(num_inputs,num_hiddens1)
        self.lin2 = nn.Linear(num_hiddens1,num_hiddens2)
        self.lin3 = nn.Linear(num_hiddens2,num_outputs)
        self.relu = nn.ReLU()
    
    def forward(self,X):
        H1 = self.relu(self.lin1(X.reshape(-1,self.num_inputs))) # 第一隐藏层
        # 只有在训练模型时才使⽤dropout
        if self.training == True:
            # 在第⼀个全连接层之后添加⼀个dropout层
            H1 = dropout_layer(H1, dropout1) 
        H2 = self.relu(self.lin2(H1)) # 第二隐藏层
        if self.training == True:
            # 在第⼆个全连接层之后添加⼀个dropout层
            H2 = dropout_layer(H2, dropout2)
        out = self.lin3(H2) # 输出层不需要激活函数
        return out
net = Net(num_inputs, num_outputs, num_hiddens1, num_hiddens2)


## 训练和测试模型
num_epochs, lr, batch_size = 5, 100.0, 256
loss = torch.nn.CrossEntropyLoss()
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size)
d2l.train_ch3(net, train_iter, test_iter, loss, num_epochs, batch_size, params, lr)




```


## 简单实现
在PyTorch中，我们只需要在全连接层后添加Dropout层并指定丢弃概率。
在训练模型时，Dropout层将以指定的丢弃概率随机丢弃上一层的输出元素；在测试模型时（即model.eval()后），Dropout层并不发挥作用。
```python
net = nn.Sequential(nn.Flatten(),
    nn.Linear(784, 256),
    nn.ReLU(),
    # 在第⼀个全连接层之后添加⼀个dropout层
    nn.Dropout(dropout1),
    nn.Linear(256, 128),
    nn.ReLU(),
    # 在第⼆个全连接层之后添加⼀个dropout层
    nn.Dropout(dropout2),
    nn.Linear(128, 10))
def init_weights(m):
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)


optimizer = torch.optim.SGD(net.parameters(), lr=0.5)
d2l.train_ch3(net, train_iter, test_iter, loss, num_epochs, batch_size, None, None, optimizer)

```

- 我们可以通过使用丢弃法应对过拟合。
- 丢弃法只在训练模型时使用。

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E4%B8%A2%E5%BC%83%E6%B3%95/  

