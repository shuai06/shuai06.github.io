# torch.argmax 小记

<script type="text/javascript" src="/js/src/bai.js"></script>


最近碰到`torch.argmax`的场景，就在这里简单记录一下。这里分二维和三维及以上记录



## 二维
```python
>>> a = torch.rand(15).reshape(3,5)
>>> a
tensor([[0.7237, 0.6488, 0.2557, 0.0333, 0.4103],
        [0.8674, 0.7288, 0.3758, 0.6329, 0.9911],
        [0.4652, 0.1548, 0.8584, 0.4093, 0.2682]])
>>> a.argmax(dim=0)
tensor([1, 1, 2, 1, 1])
>>> a.argmax(dim=1)
tensor([0, 4, 2])
```
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221021102505.png)



## 三维及以上
```python
>>> a = torch.rand(30).reshape(2,3,5)
>>> a
tensor([[[0.6043, 0.8942, 0.6633, 0.7719, 0.3094],
         [0.3755, 0.5932, 0.0996, 0.8829, 0.4801],
         [0.1362, 0.4843, 0.2369, 0.3898, 0.5511]],

        [[0.2321, 0.4191, 0.6576, 0.1157, 0.8961],
         [0.6723, 0.8386, 0.2332, 0.3209, 0.8477],
         [0.9402, 0.4330, 0.4449, 0.3894, 0.8684]]])
>>> a.argmax(dim=0)
tensor([[0, 0, 0, 0, 1],
        [1, 1, 1, 0, 1],
        [1, 0, 1, 0, 1]])
>>> a.argmax(dim=1)
tensor([[0, 0, 0, 1, 2],
        [2, 1, 0, 2, 0]])
>>> a.argmax(dim=2)
tensor([[1, 3, 4],
        [4, 4, 0]])
```
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221021102920.png)


四维同理











---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/torch-argmax-%E5%B0%8F%E8%AE%B0/  

