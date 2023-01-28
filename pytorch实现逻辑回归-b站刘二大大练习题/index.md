# PyTorch实现逻辑回归(B站刘二大大练习题)

<script type="text/javascript" src="/js/src/bai.js"></script>


```python
"""
分类问题：
比如手写数字识别,输出的是属于哪个数字的概率

import torchvision   这个数据集中提供示例数据集，download设置True会自动下载
# MNIST
train_set = torchvision.datasets.MNIST(root='../dataset/mnist', train=True, download=True)
test_set = torchvision.datasets.MNIST(root='../dataset/mnist', train=False, download=True)
# CIFAR 数据集torchvision.datasets.CIFAR10


# 比如二分类，使用sigmoid或其它函数
之前的仿射模型：y heat = x * w + b
逻辑回归模型：y heat = sigmoid(x*w + b)

损失函数也改变了：BCE损失(二分类的交叉熵)
loss = -(y*logy heat + (1-y)*log(1 - y heat))



那么Mini-Batch损失函数：
对它做均值

使用pytorch实现逻辑回归
"""


import torch
import torch.nn.functional as F


# 1.准备数据集
x_data = torch.Tensor([[1.0], [2.0], [3.0]])  # 3行1列的tensor
y_data = torch.Tensor([[0], [0], [1]])


# 2.使用类来设计模型
class LogisticRegressionModel(torch.nn.Module):  # Module构造出来的对象，会自动构建反向传播过程
    def __init__(self):
        super(LogisticRegressionModel, self).__init__()
        self.linear = torch.nn.Linear(1, 1)  # torch.nn.Linear构造一个对象，参数是权重和偏差，也是继承子Module的会自动进行反向传播
        # sigmoid中没有参数

    def forward(self, x):
        y_pred = F.sigmoid(self.linear(x))
        return y_pred


model = LogisticRegressionModel()  # model是callable的  model(x)
# 3.构建损失和优化器
criterion = torch.nn.BCELoss(size_average=False)
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)  # 梯度下降 model.parameters()自动完成参数的初始化操作
# optimizer = torch.optim.Adam(model.parameters(), lr=0.01)  # 梯度下降

# 4.训练
for epoch in range(1000):
    y_pred = model(x_data)  # 1.前向传播，计算y heat
    loss = criterion(y_pred, y_data)  # 2.计算损失
    print(epoch, loss)  # 打印

    optimizer.zero_grad()  # 梯度会自动计算，务必梯度清零！
    loss.backward()    # 3.反向传播
    optimizer.step()   # 4.更新 update

# 打印权重和偏置
print("w=", model.linear.weight.item())  # model下面的linear，下面的weight
print("b=", model.linear.bias.item())

# 测试模型
x_test = torch.Tensor([[4.0]])
y_test = model(x_test)
print("y_pred=", y_test.data)


```

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pytorch%E5%AE%9E%E7%8E%B0%E9%80%BB%E8%BE%91%E5%9B%9E%E5%BD%92-b%E7%AB%99%E5%88%98%E4%BA%8C%E5%A4%A7%E5%A4%A7%E7%BB%83%E4%B9%A0%E9%A2%98/  

