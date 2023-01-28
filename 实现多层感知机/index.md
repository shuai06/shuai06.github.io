# 实现多层感知机

<script type="text/javascript" src="/js/src/bai.js"></script>


多层感知机在输出层与输入层之间加入了一个或多个全连接隐藏层，并通过激活函数对隐藏层输出进行变换。

## 从零实现

```python
import torch
from torch import nn
from d2l import d2ltorch as d2l
batch_size = 256
train_iter,test_iter = d2l.load_data_fashion_mnist(batch_size=batch_size)


# num_inputs 输入层，图片为28*28，固定大小
# num_outputs 输出层，输出为类别数量 10类
# 输入输出层大小为固定
num_inputs, num_outputs, num_hiddens = 784, 10, 256
num_hiddens2 = 128
# 先生成第一层，即w1*x+b1
# W1层，生成一个随机层
# 试着把W1层改为全为0或者全为1会如何
W1 = nn.Parameter(torch.rand(num_inputs, num_hiddens, requires_grad=True) * 0.01)
# b1为偏差，初始为0
b1 = nn.Parameter(torch.zeros(num_hiddens, requires_grad=True))
# 输出层第二层
W2 = nn.Parameter(torch.randn(num_hiddens, num_hiddens2, requires_grad=True) * 0.01)
b2 = nn.Parameter(torch.zeros(num_hiddens2, requires_grad=True))

# 自己添加第三层
W3 = nn.Parameter(torch.randn(num_hiddens2, num_outputs, requires_grad=True) * 0.01)
b3 = nn.Parameter(torch.zeros(num_outputs, requires_grad=True))
params = [W1, b1, W2, b2, W3, b3]

# 实现ReLU函数
def relu(x):
    a = torch.zeros_like(x)
    return torch.max(x,a)

def net(X):
    X = X.reshape((-1, num_inputs)) # flattern操作
    H = relu(X@W1 + b1) # 这⾥“@”代表矩阵乘法 等效于torch.matmul()
    S = relu(H@W2 + b2)
    return (S@W3+b3)

loss = nn.CrossEntropyLoss()

num_epochs, lr = 10, 0.1
updater = torch.optim.SGD(params, lr=lr)
d2l.train_ch3(net, train_iter, test_iter, loss, num_epochs, updater)

d2l.predict_ch3(net, test_iter)
```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E5%AE%9E%E7%8E%B0%E5%A4%9A%E5%B1%82%E6%84%9F%E7%9F%A5%E6%9C%BA/  

