# Anaconda中jupyter notebook更改解释器

<script type="text/javascript" src="/js/src/bai.js"></script>

1.激活指定虚拟环境
```python
conda activate dlenv
```

2.安装ipykerbel
```python
conda install ipykernel
```

3.向ipykernel注入指定虚拟环境
```python
# python -m ipykernel install --user --name "要注入的虚拟环境"--display-name "显示名称"
python -m ipykernel install --user --name dlenv --display-name "dlenv"

```

4.打开jupyter notebook，即可选择刚加入的虚拟环境了
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220903104654.png)






---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/anaconda%E4%B8%ADjupyter-notebook%E6%9B%B4%E6%94%B9%E8%A7%A3%E9%87%8A%E5%99%A8/  

