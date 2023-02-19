# 使用PyTorch预测糖尿病(B站刘二大大练习题)

<script type="text/javascript" src="/js/src/bai.js"></script>


数据集：
![](https://image.geoer.cn/20220814205751.png)



```python
"""
解决糖尿病预测的问题

"""
import torch
import numpy as np


# 1.读取数据
data = np.loadtxt("./data/diabetes.csv.gz", delimiter=",", dtype=np.float32)
x_data = torch.from_numpy(data[:-1, :-1])  # x不取最后一列
y_data = torch.from_numpy(data[:-1, [-1]])  # y取所有行和最后一列， [-1]代表是矩阵


# 2.设计模型
class Model(torch.nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.linear1 = torch.nn.Linear(8, 6)  # 不断降维
        self.linear2 = torch.nn.Linear(6, 4)
        self.linear3 = torch.nn.Linear(4, 1)
        self.sigmoid = torch.nn.Sigmoid()
        self.active = torch.nn.ReLU()

    def forward(self, x):
        x = self.active(self.linear1(x))  # o1
        x = self.active(self.linear2(x))  # o2
        x = self.sigmoid(self.linear3(x))  # y heat
        return x


model = Model()

# 3.构造损失和优化器
criterion = torch.nn.BCELoss(size_average=True)
optimizer = torch.optim.SGD(model.parameters(), lr=0.1)


# 4.训练
for epoch in range(1000):
    # 这里forward 并非mini-batch的设计，只是mini-batch的风格
    y_pred = model(x_data)
    loss = criterion(y_pred, y_data)
    print(epoch, loss.item())

    # backward
    optimizer.zero_grad()
    loss.backward()

    # update
    optimizer.step()

# 测试集
test_data = torch.from_numpy(data[[-1], :-1])
pred_test = torch.from_numpy(data[[-1], [-1]])

print("test_pred = ", model(test_data).item())
print("infact_pred = ", pred_test.item())



```

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E4%BD%BF%E7%94%A8pytorch%E9%A2%84%E6%B5%8B%E7%B3%96%E5%B0%BF%E7%97%85-b%E7%AB%99%E5%88%98%E4%BA%8C%E5%A4%A7%E5%A4%A7%E7%BB%83%E4%B9%A0%E9%A2%98/  

