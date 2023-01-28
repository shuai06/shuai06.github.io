# CNN基础作业(刘二大大课)

<script type="text/javascript" src="/js/src/bai.js"></script>




作业如下：
![](http://image.xpshuai.cn/20220818143323.png)


网络结构：
![](http://image.xpshuai.cn/20220818152354.png)

额，我这里好像值用了两个全连接层......


```python
import torch
import torch.nn.functional as F
from torchvision import transforms
from torchvision import datasets
from torch.utils.data import DataLoader
import torch.optim as optim


# 准备数据集
batch_size = 64
# transform：把图像转化成图像张量
transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))])

train_dataset = datasets.MNIST(root='./data/mnist/', train=True, download=True, transform=transform)
train_loader = DataLoader(train_dataset, shuffle=True, batch_size=batch_size)

test_dataset = datasets.MNIST(root='./data/mnist/', train=False, download=True, transform = transform)
test_loader = DataLoader(test_dataset, shuffle=False, batch_size=batch_size)


# 设计网络模型
class Net(torch.nn.Module):
    def __int__(self):
        super(Net, self).__int__()
        self.conv1 = torch.nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = torch.nn.Conv2d(10, 20, kernel_size=5)  # 每个卷积核的channel和输入数据的channel一致
        self.conv3 = torch.nn.Conv2d(20, 30, kernel_size=2)
        self.pooling1 = torch.nn.MaxPool2d(2)
        self.pooling2 = torch.nn.MaxPool2d(3)
        self.fc1 = torch.nn.Linear(30, 20)   # 全连接层
        self.fc2 = torch.nn.Linear(20, 10)   #

    def forward(self, x):
        batch_size = x.size(0)
        x = self.pooling1(F.relu(self.conv1(x)))
        x = self.pooling1(F.relu(self.conv2(x)))
        x = self.pooling2(F.relu(self.conv3(x)))
        x = x.view(batch_size, -1)  # 自动算并填充
        x = self.fc1(x)
        x = self.fc2(x)

        return x


model = Net()
# 迁移到GPU上计算（cuda: 0 是对应GPU的编号）
# device = torch.device("cuda: 0" if torch.cuda.is_available() else "cpu")
# device.to(device)

# 损失和优化器
# 因为网络模型已经有点大了，所以梯度下降里面要用更好的优化算法，比如用带冲量的（momentum），来优化训练过程
criterion = torch.nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.5)


# 一轮训练
def train(epoch):
    running_loss = 0.0
    for batch_idx, data in enumerate(train_loader, 0):
        inputs, target = data
        # inputs, target = inputs.to(device), target.to(device)  # 迁移到GPU，注意要和模型放到同一块GPU
        optimizer.zero_grad()  # 优化器 先清零

        # forward + backword + update
        outputs = model(inputs)
        loss = criterion(outputs, target)
        loss.backward()
        optimizer.step()

        running_loss += loss.item()
        if batch_idx % 300 == 299:
            print("[%d, %5d] loss:%.3f" % (epoch+1, batch_idx+1, running_loss/2000))

def test():
    correct = 0
    total = 0
    # 避免计算梯度
    with torch.no_grad():  # 测试不用算梯度
        for data in test_loader:
            images, labels = data
            # iamges, labels = images.to(device), labels.to(device) # 利用GPU
            outputs = model(images)
            # 返回值一个是每一行的最大值，另一个是最大值的下标（每一个样本就是一行，每一行有10个量）（行是第0个维度，列是第1个维度）
            _, predicted = torch.max(outputs.data, dim=1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()  # 推测出来的分类与label是否相等，真就是1，假就是0，求完和之后把标量拿出来
        print('Accuracy on test set: %d %%' % (100 * correct / total))


# 训练
if __name__ == '__main__':
    for epoch in range(10):
        train(epoch)
        test()

```






---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/cnn%E5%9F%BA%E7%A1%80%E4%BD%9C%E4%B8%9A-%E5%88%98%E4%BA%8C%E5%A4%A7%E5%A4%A7%E8%AF%BE/  

