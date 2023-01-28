# 常见语义分割评价指标

<script type="text/javascript" src="/js/src/bai.js"></script>


## 常见语义分割评价指标
![20221003114905](http://image.xpshuai.cn/20221003114905.png)



### Pixel Accuracy/global accuary
> 分子是所有预测正确像素总和，分母是所有的像素


#### **例子：**
预测正确是用绿色表示，预测错误的用红色表示。
那对于预测0来说，就是：绿色的0/(红色的0+绿色的0)
![20221003115059](http://image.xpshuai.cn/20221003115059.png)

对预测为1的：正确的为绿色，错误的为红色
![20221003115119](http://image.xpshuai.cn/20221003115119.png)

预测2
![20221003115134](http://image.xpshuai.cn/20221003115134.png)

预测3
![20221003115146](http://image.xpshuai.cn/20221003115146.png)

预测4
![20221003115208](http://image.xpshuai.cn/20221003115208.png)


最后得到的混淆矩阵：
![20221003115251](http://image.xpshuai.cn/20221003115251.png)






#### **代码：**
```python

```



### mean Accuracy
> 每个类别的accuracy计算出来，然后求平均
#### 计算
![20221003115403](http://image.xpshuai.cn/20221003115403.png)
用上图的每个类别计算出来，取平均即可


#### **代码：**
```python

```



### mean IoU  ☆
> 每个类别的IOU，然后求平均
这里的IOU的计算和目标检测中的IOU是类似的(交集比上并集)

#### 计算
分子：每个类别预测正确的像素个数
分母：真实标签像素个数 + 预测为这个类别的像素的个数-分子

![20221003115557](http://image.xpshuai.cn/20221003115557.png)

如上图例子，将这个5个类别的IoU分别求出来，取平均就可


#### **代码：**
```python

```




## 参考
来自B站霹雳吧啦Wz






---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E5%B8%B8%E8%A7%81%E8%AF%AD%E4%B9%89%E5%88%86%E5%89%B2%E8%AF%84%E4%BB%B7%E6%8C%87%E6%A0%87/  

