# sklearn包简介

<script type="text/javascript" src="/js/src/bai.js"></script>
> 只对该包做一个简单的介绍和使用方法的查找，具体可以查看官方文档，使用的时候查找即可



## 简介
scikit-learn  
sklearn是一个Python第三方提供的非常强力的机器学习库，它包含了从数据预处理到训练模型的各个方面。在实战使用scikit-learn中可以极大的节省我们编写代码的时间以及减少我们的代码量，使我们有更多的精力去分析数据分布，调整模型和修改超参。不过sklearn主要应用是传统的机器学习，并且其在数据量较小的情况下非常适用。



## 概括
sklearn拥有可以用于监督和无监督学习的方法，一般来说监督学习使用的更多。sklearn中的大部分函数可以归为估计器(Estimator)和 转化器(Transformer)两类。

**估计器(Estimator)其实就是模型**，它用于对数据的预测或回归。基本上估计器都会有以下几个方法：
- `fit(x,y)`:传入数据以及标签即可训练模型，训练的时间和参数设置，数据集大小以及数据本身的特点有关
- `score(x,y)`:用于对模型的正确率进行评分(范围0-1)。但由于对在不同的问题下，评判模型优劣的的标准不限于简单的正确率，可能还包括召回率或者是查准率等其他的指标，特别是对于类别失衡的样本，准确率并不能很好的评估模型的优劣，因此在对模型进行评估时，不要轻易的被score的得分蒙蔽。
- `predict(x)`:用于对数据的预测，它接受输入，并输出预测标签，输出的格式为numpy数组。我们通常使用这个方法返回测试的结果，再将这个结果用于评估模型。

**转化器(Transformer)用于对数据的处理**，例如标准化、降维以及特征选择等等。同与估计器的使用方法类似:
- fit(x,y) :该方法接受输入和标签，计算出数据变换的方式。
- transform(x) :根据已经计算出的变换方式，返回对输入数据x变换后的结果（不改变x）
- fit_transform(x,y) :该方法在计算出数据变换方式之后对输入x就地转换。
以上仅仅是简单的概括sklearn的函数的一些特点。

sklearn绝大部分的函数的基本用法大概如此。但是不同的估计器会有自己不同的属性，例如随机森林会有Feature_importance来对衡量特征的重要性，而逻辑回归有coef_存放回归系数intercept_则存放截距等等。并且对于机器学习来说模型的好坏不仅取决于你选择的是哪种模型，很大程度上与你超参的设置有关。因此使用sklearn的时候一定要去看看官方文档，以便对超参进行调整。


## 官方文档
地址：https://scikit-learn.org/stable/

**结构：**
- tutorials：是一个官方教程，可以理解快速上手教程，但是看完感觉并没有很快。
- user guide(用户指南）：这里对每一个算法有详细的介绍
- API：这里是库调用的方法
- FAQ：常见问题


**总结:** 一般的做法是API里面找到你要调用的方法，然后可以查看方法参数的情况和使用情况。也可以在指南里面找到具体的解释。


## 库描述
### 结构
可以看到库的算法主要有四类：分类，回归，聚类，降维。其中：

- 常用的回归：线性、决策树、SVM、KNN ；集成回归：随机森林、Adaboost、GradientBoosting、Bagging、ExtraTrees
- 常用的分类：线性、决策树、SVM、KNN，朴素贝叶斯；集成分类：随机森林、Adaboost、GradientBoosting、Bagging、ExtraTrees
- 常用聚类：k均值（K-means）、层次聚类（Hierarchical clustering）、DBSCAN
- 常用降维：LinearDiscriminantAnalysis、PCA


### 常用模块
![](http://image.xpshuai.cn/20220717084654.png)


#### 样本数据集
sklearn为初学者提供了一些经典数据集，通过这些数据集可快速搭建机器学习任务、对比模型性能。数据集主要围绕分类和回归两类经典任务，对于不同需求，常用数据集简介如下：
![](http://image.xpshuai.cn/20220717084802.png)


```python

from sklearn import datasets
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

#使用以后的数据集进行线性回归（这里是波士顿房价数据）
loaded_data=datasets.load_boston()
data_X=loaded_data.data
data_y=loaded_data.target

model=LinearRegression()
model.fit(data_X,data_y)

print(model.predict(data_X[:4,:]))
print(data_y[:4])

#使用生成线性回归的数据集，最后的数据集结果用散点图表示
X,y=datasets.make_regression(n_samples=100,n_features=1,n_targets=1,noise=10)   #n_samples表示样本数目，n_features特征的数目  n_tragets  noise噪音
plt.scatter(X,y)
plt.show()
```


还可以自己加载自己的数据



#### 数据预处理
数据预处理包括：降维、数据归一化、特征提取和特征转换（one-hot）等，这在sklearn里面有很多方法，具体查看api。

![](http://image.xpshuai.cn/20220717084855.png)

- MinMaxScaler：归一化去量纲处理，适用于数据有明显的上下限，不会存在严重的异常值，例如考试得分0-100之间的数据可首选归一化处理
- StandardScaler：标准化去量纲处理，适用于可能存在极大或极小的异常值，此时用MinMaxScaler时，可能因单个异常点而将其他数值变换的过于集中，而用标准正态分布去量纲则可有效避免这一问题
- Binarizer：二值化处理，适用于将连续变量离散化
- OneHotEncoder：独热编码，一种经典的编码方式，适用于离散标签间不存在明确的大小相对关系时。例如对于民族特征进行编码时，若将其编码为0-55的数值，则对于以距离作为度量的模型则意味着民族之间存在"大小"和"远近"关系，而用独热编码则将每个民族转换为一个由1个"1"和55个"0"组成的向量。弊端就是当分类标签过多时，容易带来维度灾难，而特征又过于稀疏
- Ordinary：数值编码，适用于某些标签编码为数值后不影响模型理解和训练时。例如，当民族为待分类标签时，则可将其简单编码为0-55之间的数字



这里用归一化（preprocessing.scale() ）例子：
```python
from sklearn import preprocessing #进行标准化数据时，需要引入个包
import numpy as np
from sklearn.cross_validation import train_test_split
from sklearn.datasets.samples_generator import  make_classification
from sklearn.svm import SVC
import matplotlib.pyplot as plt


X,y=make_classification(n_samples=300,n_features=2,n_redundant=0,n_informative=2,random_state=22,n_clusters_per_class=1,scale=100)

#X=preprocessing.minmax_scale(X,feature_range=(-1,1))
X=preprocessing.scale(X)   #0.966666666667 没有 0.477777777778
X_train,X_test,y_train,y_test=train_test_split(X,y,test_size=0.3)
clf=SVC()
clf.fit(X_train,y_train)
print(clf.score(X_test,y_test))


plt.scatter(X[:,0],X[:,1],c=y)
plt.show()

a=np.array([[10,2.7,3.6],
            [-100,5,-2],
            [120,20,40]],dtype=np.float64)   #每一列代表一个属性
print(a)#标准化之前a　　　　　
print(preprocessing.scale(a))#标准化之后的a　

```

#### 特征选择
![](http://image.xpshuai.cn/20220717085004.png)



#### 模型选择
模型选择是机器学习中的重要环节，涉及到的操作包括数据集切分、参数调整和验证等。对应常用函数包括：

- train_test_split：常用操作之一，切分数据集和测试集，可设置切分比例

- cross_val_score：交叉验证，默认K=5折，相当于把数据集平均切分为5份，并逐一选择其中一份作为测试集、其余作为训练集进行训练及评分，最后返回K个评分

- GridSearchCV：调参常用方法，通过字典类型设置一组候选参数，并制定度量标准，最后返回评分最高的参数

![](http://image.xpshuai.cn/20220717085041.png)



```python
from sklearn import datasets
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

#使用以后的数据集进行线性回归
loaded_data=datasets.load_boston()
data_X=loaded_data.data
data_y=loaded_data.target

model=LinearRegression()
model.fit(data_X,data_y)

print(model.predict(data_X[:4,:]))
print(data_y[:4])

#参数
print(model.coef_)      #如果y=0.1x+0.3   则此行输出的结果为0.1
print(model.intercept_)             #此行输出的结果为0.3
print(model.get_params())       #模型定义时定义的参数，如果没有定义则返回默认值
print(model.score(data_X,data_y))   #给训练模型打分，注意用在LinearR中使用R^2 conefficient of determination打分
```


#### 度量参数
![](http://image.xpshuai.cn/20220717085139.png)


#### 降维
降维也属于无监督学习的一种，当特征维度过多时可通过矩阵的QR分解实现在尽可能保留原有信息的情况下降低维度，一般用于图像数据预处理，且降维后的特征与原特征没有直接联系，使得模型训练不再具有可解释性。


#### 聚类
聚类是一种典型的无监督学习任务，但也是实际应用中较为常见的需求。在不提供样本真实标签的情况下，基于某些特征对样本进行物以类聚。根据聚类的原理，主要包括三种：

- 基于距离聚类，典型的就是K均值聚类，通过不断迭代和重新寻找最小距离，对所有样本划分为K个簇，有一款小游戏《拥挤城市》应该就是基于K均值聚类实现
- 基于密度聚类，与距离聚类不同，基于密度聚类的思想是源于通过距离判断样本是否连通（需指定连通距离的阈值），从而完成样本划分。由于划分结果仅取决于连通距离的阈值，所以不可指定聚类的簇数。典型算法模型是DBSCAN
- 基于层次聚类，具体又可细分为自顶向下和自底向上，以自底向上层次聚类为例：首先将所有样本划分为一类，此时聚类簇数K=样本个数N，遍历寻找K个簇间最相近的两个簇并完成合并，此时还有K-1个簇，如此循环直至划分为指定的聚类簇数。当然，这里评价最相近的两个簇的标准又可细分为最小距离、最大距离和平均距离。



#### 基本学习模型
![](http://image.xpshuai.cn/20220717085258.png)



#### 集成学习模型
![](http://image.xpshuai.cn/20220717085400.png)
当基本学习模型性能难以满足需求时，集成学习便应运而生。集成学习，顾名思义，就是将多个基学习器的结果集成起来汇聚出最终结果。而根据汇聚的过程，集成学习主要包括3种流派：

- bagging，即bootstrap aggregating，通过自助取样（有放回取样）实现并行训练多个差异化的基学习器，虽然每个学习器效果可能并不突出，但通过最后投票得到的最终结果性能却会稳步提升。当基学习器采取决策树时，bagging思想的集成学习模型就是随机森林。另外，与bagging对应的另一种方式是无放回取样，相应的方法叫pasting，不过应用较少
- boosting，即提升法。与bagging模型并行独立训练多个基学习器不同，boosting的思想是基于前面训练结果逐渐训练更好的模型，属于串行的模式。根据实现细节不同，又具体分为两种boosting模型，分别是Adaboost和GBDT，二者的核心思想差异在于前者的提升聚焦于之前分错的样本、而后者的提升聚焦于之前漏学的残差。另外一个大热的XGBoost是对GBDT的一个改进，实质思想是一致的。
- stacking，即堆栈法，基本流程与bagging类似而又不同：stacking也是并行独立训练多个基学习器，而后又将这些训练的结果作为特征进行再次学习。有些类似于深度学习中的多层神经网络。




## 参考
https://zhuanlan.zhihu.com/p/33420189
https://blog.csdn.net/u014248127/article/details/78885180
https://blog.csdn.net/weixin_41395763/article/details/122949178





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/sklearn%E5%8C%85%E5%AD%A6%E4%B9%A0/  

