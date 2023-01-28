# 使用PyTorch预测泰坦尼克生还作业

<script type="text/javascript" src="/js/src/bai.js"></script>

> B站河北工业大学老师刘二课后作业


数据地址：https://www.kaggle.com/c/titanic/data?select=test.csv

```python
"""
课后作业：
建立DataLoader:
https://www.kaggle.com/c/titanic/data
使用DataLoader建立分类器

"""
import numpy as np
import torch
from torch.utils.data import Dataset
import pandas as pd
from torch.utils.data import DataLoader


# 数据准备
# 1、定义自己的数据集类
class My_dataset(Dataset):  # 定义自己的数据集类，其继承于Dataset
    def __init__(self, filepath):  # 初始化自己的文件读取路径
        # 从原始数据集中提取5个特征
        features = ["Pclass", "Sex", "SibSp", "Parch", "Fare"]
        # 读取数据
        data = pd.read_csv(filepath)
        self.len = data.shape[0]  # [0]代表行数，[1]代表列数
        self.x_data = torch.from_numpy(np.array(pd.get_dummies(data[features])))  # dummies相当于one-hot编码
        # np.array(data['survived'])是对data['survived']创建一个矩阵
        # torch.from_numpy()是将括号内的矩阵形式转换为张量形式，方便torch处理
        self.y_data = torch.from_numpy(np.array(data['Survived']))

    def __getitem__(self, index):  # 输入index就返回X-data中index对应的值
        return self.x_data[index], self.y_data[index]

    def __len__(self):  # 返回容器内元素个数，为后面的len()函数进行准备
        return self.len


# 2、准备自己的数据，实例化数据
dataset = My_dataset(r"./data/train.csv")
# 3、建立自己的数据集加载器
train_loader = DataLoader(dataset=dataset, batch_size=1, shuffle=True, num_workers=0)


# 定义模型
class Model(torch.nn.Module):  # 设置要从torch神经网络模块中要继承的模型函数
    def __init__(self):
        super(Model, self).__init__()  # 对继承于torch.nn的父模块类进行初始化
        # 这里包括2个线性层，每一个线性层输出都用激活函数激活
        self.linear1 = torch.nn.Linear(6, 3)  # 五个特征转化为了6维，因为get_dummies将性别这一个特征用两个维度来表示，即男性[1,0],女性[0，1]
        self.linear2 = torch.nn.Linear(3, 1)
        self.sigmoid = torch.nn.Sigmoid()  # 激活函数从Sigmoid这一大类激活函数中选取sigmoid这一种激活函数

    # 前向传播
    def forward(self, x):
        x = self.sigmoid(self.linear1(x))
        x = self.sigmoid(self.linear2(x))
        return x

    # 定义的预测函数
    def predict(self, x):  # 该函数用在测试集过程，因此只有前向传播，没有什么
        with torch.no_grad():
            x = self.sigmoid(self.linear1(x))
            x = self.sigmoid(self.linear2(x))
            y = []  # 将测试的结果都汇集到这个列表中
            for i in x:
                if i > 0.5:
                    y.append(1)
                else:
                    y.append(0)
            return y


model = Model()  # 构建实例化类
# 构建损失函数和优化器
criterion = torch.nn.BCELoss(reduction='mean')  # reduction=‘mean’代表返回损失的平均值，‘sum’代表返回损失和
optimizer = torch.optim.SGD(model.parameters(), lr=0.005)  # 优化器选择sgd方法，其中优化对象为创建的实例，优化元素为其权重偏置等网络参数
# 进行训练（前向，反向，迭代更新）
if __name__ == '__main__':
    for epoch in range(100):
        for i, data in enumerate(train_loader, 0):  # 按照索引的方式提取相关数据
            inputs, label = data
            # 转换一次数据类型
            inputs = inputs.float()
            label = label.float()
            y_pred = model(inputs)
            # 将维度降至1维并输出出来
            y_pred = y_pred.squeeze(-1)  # 前向输出结果是[[12],[34],[35],[23],[11]]这种，需要将这个二维矩阵转换成一行[12,34,35,23,11]
            loss = criterion(y_pred, label)  # 将预测的值与标签进行比较，并求解出误差值
            print(epoch, i, loss.item())  # 输出迭代次数，每次迭代的顺序以及损失
            optimizer.zero_grad()  # 之前的梯度进行清零，否则梯度会累加起来
            loss.backward()
            optimizer.step()
test_data = pd.read_csv(r"./data/test.csv")  # 读取测试集文件，并进行测试
features = ["Pclass", "Sex", "SibSp", "Parch", "Fare"]  # 测试集数据的特征应和训练集的保持一致
test = torch.from_numpy(np.array(pd.get_dummies(test_data[features])))  # 将测试集数据进行独热化处理，并输出0,1结果
# 进行预测
y = model.predict(test.float())
outputs = pd.DataFrame({'PassengerId': test_data.PassengerId, 'Survived': y})
print(outputs)





```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E4%BD%BF%E7%94%A8pytorch%E9%A2%84%E6%B5%8B%E6%B3%B0%E5%9D%A6%E5%B0%BC%E5%85%8B%E7%94%9F%E8%BF%98%E4%BD%9C%E4%B8%9A/  

