# 实现Softmax回归

<script type="text/javascript" src="/js/src/bai.js"></script>


## 从零实现softmax回归
softmax的公式:
$$
softmax(X)_{ij} = \frac{e^{X_{ij}}}{\sum_k{e^{X_{ij}}}}
$$


   
```python
import torch
from IPython import display
from d2l import torch as d2l

# 这里对图像进行展平，将它们视为长度为784的向量，因为数据有10个类别，所以输出维度为10
batch_size = 256
# 每次选取256张图片
train_iter, test_iter = d2l.load_data_fashion_mnist(batch_size)

num_inputs = 784
num_outputs = 10
W = torch.normal(mean=0,
                 std=0.01,
                 size=(num_inputs, num_outputs),
                 requires_grad=True)
b = torch.zeros(num_outputs, requires_grad=True)


def softmax(x):
    x_exp = torch.exp(x)
    partition = x_exp.sum(dim=1, keepdim=True)
    # 这里直接返回的就是张量
    return x_exp / partition  #这里应用了广播机制


# 实现softmax回归模型
def net(x, w=W, b=b):
    return softmax(torch.matmul(x.reshape(-1, w.shape[0]), w) + b)

y = torch.tensor([0, 2, 2])
y_hat = torch.tensor([[0.1, 0.3, 0.6], [0.3, 0.2, 0.5], [0.1, 0.2, 0.6]])
# 这种选取方式相当于选取了[0,0],[1,2],[2,2]三个点
y_hat[[0, 1, 2], y]

# 定义交叉熵函数
def cross_entropy(y_hat, y):
    return -torch.log(y_hat[range(len(y_hat)), y])

cross_entropy(y_hat, y)


# 定义预测类别与真实类别之间的误差函数


def accuracy(y_hat, y):
    """计算预测正确的数量"""
    if len(y_hat.shape) > 1 and y_hat.shape[1] > 1:  # y_hat维数大于1且y_hat的列数大于1
        # 将每个行中元素最大值的下标提取出来,作为y_hat的值(预测的类别)
        y_hat = y_hat.argmax(dim=1)
    # cmp用于判断每一行判断的分类与y实际值的比较
    cmp = y_hat.type(y.dtype) == y
    # 为True转换为1,False
    return float(cmp.type(y.dtype).sum())


accuracy(y_hat, y) / len(y)



# 该类的主要用处在于储存正确预测的数量和预测的总数量,包含在创建的两个变量中
class Accumulator:
    """在n个变量上面累加"""
    def __init__(self, n):
        self.data = [0.0] * n

    # 这个函数用于储存正确预测的数量和预测的总数量
    def add(self, *args):
        self.data = [a + float(b) for a, b in zip(self.data, args)]

    def reset(self):
        self.data = [0.0] * len(self.data)

    def __getitem__(self, idx):
        return self.data[idx]



# 接下来定义一个评估任意net模型的准确率函数
def evaluate_accuracy(net, data_iter):
    """计算在指定数据集上的模型精确度"""
    # isinstance() 用于判断目标函数是否属于目标类型,返回为bool类型
    if isinstance(net, torch.nn.Module):
        net.eval()  # 将模型设置为评估模式，评估模式下不会进行梯度计算
    metric = Accumulator(2)  # 正确预测数,预测总数
    # numel,返回tensor里面的元素个数
    for X, y in data_iter:
        metric.add(accuracy(net(X), y), y.numel())
    return metric[0] / metric[1]


def train_epoch_ch3(net, train_iter, loss, updater):
    if isinstance(net, torch.nn.Module):
        net.train()
    metric = Accumulator(3)
    for X, y in train_iter:
        # 对X进行softmax转换
        y_hat = net(X)
        l = loss(y_hat, y)
        if isinstance(updater, torch.optim.Optimizer):
            # 梯度清除
            updater.zero_grad()
            # 计算梯度
            l.backward()
            # 参数更新
            updater.step()
            # 类中放入 损失函数\准确率\样本数
            metric.add(float(l) * len(y), accuracy(y_hat, y), y.size().numel())
        else:  # 这里如果是自己创建的计算方式,那么就按如下方式进行计算
            # 计算梯度
            l.sum().backward()
            # 参数更新
            updater(X.shape[0])
            metric.add(float(l.sum()), accuracy(y_hat, y), y.numel())
    # 这里返回的是损失率和预测准确率
    return metric[0] / metric[2], metric[1] / metric[2]



class Animator: #@save
    """在动画中绘制数据。"""
    def __init__(self, xlabel=None, ylabel=None, legend=None, xlim=None,
        ylim=None, xscale='linear', yscale='linear',
        fmts=('-', 'm--', 'g-.', 'r:'), nrows=1, ncols=1,
        figsize=(6.5, 4.5)):
        # 增量地绘制多条线
        if legend is None:
            legend = []
        d2l.use_svg_display()
        self.fig, self.axes = d2l.plt.subplots(nrows, ncols, figsize=figsize)
        if nrows * ncols == 1:
            self.axes = [self.axes, ]
        # 使⽤lambda函数捕获参数
        self.config_axes = lambda: d2l.set_axes(
        self.axes[0], xlabel, ylabel, xlim, ylim, xscale, yscale, legend)
        self.X, self.Y, self.fmts = None, None, fmts

    def add(self, x, y):
        # 向图表中添加多个数据点
        if not hasattr(y, "__len__"):
            y = [y]
        n = len(y)
        if not hasattr(x, "__len__"):
            x = [x] * n
        if not self.X:
            self.X = [[] for _ in range(n)]
        if not self.Y:
            self.Y = [[] for _ in range(n)]
        for i, (a, b) in enumerate(zip(x, y)):
            if a is not None and b is not None:
                self.X[i].append(a)
                self.Y[i].append(b)
                self.axes[0].cla()
        for x, y, fmt in zip(self.X, self.Y, self.fmts):
            self.axes[0].plot(x, y, fmt)
            self.config_axes()
            display.display(self.fig)
            display.clear_output(wait=True)



def train_ch3(net,train_iter,test_iter,loss,num_epochs,updater):
    """完整的迭代周期
        num_epochs 迭代次数
    """    
    animator = Animator(xlabel='epoch', xlim=[1, num_epochs], ylim=[0.3, 0.9],
    legend=['train loss', 'train acc', 'test acc'])
    for epoch in range(num_epochs):
        train_metrics = train_epoch_ch3(net, train_iter, loss, updater)
        test_acc = evaluate_accuracy(net, test_iter)
        animator.add(epoch + 1, train_metrics + (test_acc,))
    train_loss, train_acc = train_metrics
    assert train_loss < 0.5, train_loss
    assert train_acc <= 1 and train_acc > 0.7, train_acc
    assert test_acc <= 1 and test_acc > 0.7, test_acc


lr = 0.1
# d2l.sgd 小批量梯度下降法更新参数
def updater(batch_size):
    return d2l.sgd([W, b], lr, batch_size)
num_epochs = 10
train_ch3(net, train_iter, test_iter, cross_entropy, num_epochs, updater)


def predict_ch3(net, test_iter, n=6): #@save
    """预测标签（定义⻅第3章）。"""
    for X, y in test_iter:
        break
    trues = d2l.get_fashion_mnist_labels(y)
    preds = d2l.get_fashion_mnist_labels(net(X).argmax(axis=1))
    titles = [true +'\n' + pred for true, pred in zip(trues, preds)]
    d2l.show_images(
        X[0:n].reshape((n, 28, 28)), 1, n, titles=titles[0:n])
predict_ch3(net, test_iter)


```

## 使用PyTorch实现softmax回归

```python
"""
使用PyTorch实现sofamax回归
"""

import torch
from torch import nn
from d2l import d2ltorch as d2l
batch_size = 256
train_iter,test_iter = d2l.load_data_fashion_mnist(batch_size)


# PyTorch不会隐式地调整输⼊的形状。因此，
# 我们在线性层前定义了展平层（flatten），来调整⽹络输⼊的形状
net = nn.Sequential(nn.Flatten(), nn.Linear(784, 10))
def init_weights(m):
    """初始化权重"""
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)


# 交叉熵损失
loss = nn.CrossEntropyLoss()

# 随机梯度下降
trainer = torch.optim.SGD(net.parameters(), lr=0.1)

num_epochs = 10
from visdom import Visdom
import time

wind = Visdom(env=u"testV", use_incoming_socket=False)
# 2.初始化窗口信息
wind.line([0.],  # Y的第一个点的坐标
          [0.],  # X的第一个点的坐标
          win='train_loss',  # 窗口的名称
          opts=dict(title='train_loss')  # 图像的标例
          )



def train_ch_test(net,train_iter,test_iter,loss,num_epochs,updater):
    """完整的迭代周期
        num_epochs 迭代次数
    """
    for epoch in range(num_epochs):
        # line
        train_metrics = d2l.train_epoch_ch3(net, train_iter, loss, updater)
        test_acc = d2l.evaluate_accuracy(net, test_iter)
        train_loss, train_acc = train_metrics
        print(train_metrics)
        # 使用visdom画出训练损失
        wind.line([train_loss], [epoch+1], win='train_loss', update='append')
        time.sleep(0.5)
#     animator.add(epoch + 1, train_metrics + (test_acc,))
    train_loss, train_acc = train_metrics
    assert train_loss < 0.5, train_loss
    assert train_acc <= 1 and train_acc > 0.7, train_acc
    assert test_acc <= 1 and test_acc > 0.7, test_acc


train_ch_test(net, train_iter, test_iter, loss, num_epochs, trainer)
```



## 简洁实现
```python
import torch
from torch import nn
from d2l import torch as d2l
batch_size = 256
train_iter,test_iter = d2l.load_data_fashion_mnist(batch_size)

# PyTorch不会隐式地调整输⼊的形状。因此，
# 我们在线性层前定义了展平层（flatten），来调整⽹络输⼊的形状
net = nn.Sequential(nn.Flatten(), nn.Linear(784, 10))
def init_weights(m):
    """初始化权重"""
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)

loss = nn.CrossEntropyLoss()
trainer = torch.optim.SGD(net.parameters(), lr=0.1)
num_epochs = 10
d2l.train_ch3(net, train_iter, test_iter, loss, num_epochs, trainer)
```

也可以:http://www.xpshuai.cn/Pytorch%E5%AD%A6%E4%B9%A0-nn.html


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E5%AE%9E%E7%8E%B0softmax%E5%9B%9E%E5%BD%92/  

