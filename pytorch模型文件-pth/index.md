# PyTorch模型文件.pth浅析

<script type="text/javascript" src="/js/src/bai.js"></script>


## 保存模型
在pytorch进行模型保存的时候，一般有两种保存方式:
- 一种是保存整个模型
- 另一种是只保存模型的参数

```python
torch.save(model.state_dict(), "my_model.pth")  # 只保存模型的参数
torch.save(model, "my_model.pth")  # 保存整个模型

```




## 后缀的格式
我们在训练模型的时候保存模型一般是保存`.pth`和`.pkl`，有时候也用`.pt`，有什么区别呢？

其实，他们只是后缀名不同而已,格式没啥区别  
一般惯例是使用`.pth`  
另外，为什么会有 `.pkl`这种后缀名呢？因为Python有一个序列化模块`pickle`，使用它保存模型时，通常会起一个以`.pkl`为后缀名的文件。刚好`torch.save()`也是使用`pickle`来保存模型的。






## 模型文件浅析
查看一下模型文件
```python
module_save = "model_save/module_attunet.pkl"

if os.path.exists(module_save):
    # net.load_state_dict(torch.load(module_save))
    a = torch.load(module_save)
    print(type(a))  # <class 'collections.OrderedDict'>
    print(len(a))   # 240
    for k in a.keys():
        print(k)    # 查看键

```

![](http://image.xpshuai.cn/20221021100200.png)

```python
for k in net.keys():
         print(k, net[k].shape, sep="    ")
```
![](http://image.xpshuai.cn/20221021100618.png)







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pytorch%E6%A8%A1%E5%9E%8B%E6%96%87%E4%BB%B6-pth/  

