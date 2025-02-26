# Labelme不同物体不同颜色标注和导出




使用labelme进行语义分割数据集进行多个类别物体标注时候经常需要指定导出颜色，在这里备忘一下

ps:我这里用的mac系统



1.去这个目录下修改文件`/Users/xps/miniconda3/envs/labelme_ai/lib/python3.9/site-packages/imgviz/label.py`

![image-20241112094754415](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20241112094754415.png)





2. 找到`json_to_dataset.py` 文件

   先查找labelme安装路径，`json_to_dataset.py`文件通常在这个目录下

```
pip show labelme

```

![image-20241112095333400](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20241112095333400.png)



然后我找到了`/Users/xps/miniconda3/envs/labelme_ai/lib/python3.9/site-packages/labelme/cli/json_to_dataset.py`

修改这个文件

![image-20241112095525583](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20241112095525583.png)





然后导出的时候就正常了







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/labelme%E4%B8%8D%E5%90%8C%E7%89%A9%E4%BD%93%E4%B8%8D%E5%90%8C%E9%A2%9C%E8%89%B2%E6%A0%87%E6%B3%A8%E5%92%8C%E5%AF%BC%E5%87%BA/  

