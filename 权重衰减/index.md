# 权重衰减

<script type="text/javascript" src="/js/src/bai.js"></script>

> 权重衰减就是我们常说的L2正则化


正则化通过为模型损失函数添加惩罚项使学出的模型参数值较小。
权重衰减通过惩罚绝对值较大的模型参数为需要学习的模型增加了限制，这可能对过拟合有效。实际场景中，我们有时也在惩罚项中添加偏差元素的平方和。

![20220913103344](http://image.xpshuai.cn/20220913103344.png)
其中超参数λ>0。
当权重参数均为0时，惩罚项最小。
当λ较大时，惩罚项在损失函数中的比重较大，这通常会使学到的权重参数的元素较接近0。
当λ设为0时，惩罚项完全不起作用

![20220913103519](http://image.xpshuai.cn/20220913103519.png)

# 从0实现
> 我们通过在目标函数后添加L2范数惩罚项来实现权重衰减。

```python
%matplotlib inline
import torch
from torch import nn
from d2l import torch as d2l


n_train, n_test, num_inputs, batch_size = 20, 100, 200, 5
true_w, true_b = torch.ones((num_inputs, 1)) * 0.01, 0.05
# synthetic_data合成数据
train_data = d2l.synthetic_data(true_w, true_b, n_train)
train_iter = d2l.load_array(train_data, batch_size)

test_data = d2l.synthetic_data(true_w, true_b, n_test)
test_iter = d2l.load_array(test_data, batch_size, is_train=False)

# 初始化模型参数
def init_params():
    w = torch.normal(0,1,size=(num_inputs,1),requires_grad=True)
    b = torch.zeros(1,requires_grad=True)
    return [w,b]

# 定义L2惩罚函数
def l2_penalty(w):
    return torch.sum(w.pow(2))/2
    # return torch.sum(w**2)/2


def train(lambd):
    w, b = init_params()
    # 生成一个线性回归函数，损失函数采用平方损失
    net, loss = lambda X: d2l.linreg(X, w, b), d2l.squared_loss
    num_epochs, lr = 100, 0.003
    # 此部分还是绘画模块
    animator = d2l.Animator(xlabel='epochs',
                            ylabel='loss',
                            yscale='log',
                            figsize=(6,3),
                            xlim=[5, num_epochs],
                            legend=['train', 'test'])
    # 依旧是迭代循环
    for epoch in range(num_epochs):
        for X, y in train_iter:
            with torch.enable_grad():
                # 增加了L2范数惩罚项，⼴播机制使l2_penalty(w)成为⼀个⻓度为`batch_size`的向量。
                l = loss(net(X), y) + lambd * l2_penalty(w)
                l.sum().backward()
                d2l.sgd([w, b], lr, batch_size)
        if (epoch + 1) % 5 == 0:
            animator.add(epoch + 1, (d2l.evaluate_loss(net, train_iter, loss),
                                     d2l.evaluate_loss(net, test_iter, loss)))
    # norm是求2范数，item是求数值
    print('w的L2范数是：', torch.norm(w).item())
  

train(0)
# 可以看出有用训练数据较少，出现了严重的过拟合，test函数的loss基本没有下降


train(3)
# 引入L2范数后，test的loss出现了同步下降,有效避免了过拟合
```

## 简洁实现
```python
# 简洁实现
def train_concise(wd):
    net = nn.Sequential(nn.Linear(num_inputs, 1))
    for param in net.parameters():
        param.data.normal_()
    loss = nn.MSELoss()
    num_epochs, lr = 100, 0.003
    # 偏置参数没有衰减。
    # 'weight_decay'就是我们需要的L2范数超参数
    trainer = torch.optim.SGD([{"params":net[0].weight,'weight_decay': wd},{"params":net[0].bias}], lr=lr)
    animator = d2l.Animator(xlabel='epochs', ylabel='loss', yscale='log',xlim=[5, num_epochs], legend=['train', 'test'])
    for epoch in range(num_epochs):
        for X, y in train_iter:
            with torch.enable_grad():
                trainer.zero_grad()
                l = loss(net(X), y)
            l.backward()
            trainer.step()
        if (epoch + 1) % 5 == 0:
            animator.add(epoch + 1, (d2l.evaluate_loss(net, train_iter, loss),d2l.evaluate_loss(net, test_iter, loss)))
    print('w的L2范数：', net[0].weight.norm().item())

train_concise(0)

train_concise(3)
```


- 正则化通过为模型损失函数添加惩罚项使学出的模型参数值较小，是应对过拟合的常用手段。
- 权重衰减等价于L2L 
- 范数正则化，通常会使学到的权重参数的元素较接近0。
- 权重衰减可以通过优化器中的weight_decay超参数来指定。
- 可以定义多个优化器实例对不同的模型参数使用不同的迭代方法。



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E6%9D%83%E9%87%8D%E8%A1%B0%E5%87%8F/  

