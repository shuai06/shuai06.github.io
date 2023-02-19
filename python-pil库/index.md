# Python PIL库

<script type="text/javascript" src="/js/src/bai.js"></script>
> 上次接触PIL是2017年的事情了，由于好几年不用都忘了个一干二净，最近又接触到图像处理方面的工作，重新学一下

## 前言
Pillow 是 Python 中较为基础的图像处理库，主要用于图像的基本处理，比如裁剪图像、调整图像大小和图像颜色处理等。与 Pillow 相比，OpenCV 和 Scikit-image 的功能更为丰富，所以使用起来也更为复杂，主要应用于机器视觉、图像分析等领域，比如众所周知的“人脸识别”应用 。

Pillow 提供了丰富的图像处理功能，可概括为两个方面：
- 图像归档（包括创建缩略图、生成预览图像、图像批量处理等）
- 图像处理（包括调整图像大小、裁剪图像、像素点处理、添加滤镜、图像颜色处理等
）

## 常用操作
### 基本操作
#### 图像打开与读取
在PIL中，任何图像都可以用Image对象表示

```python
"""
方法	描述
Image.open(filename,mode=‘r’)	根据文件名读取图片
Image.new(mode,size color)	根据给定参数创建一个新图像
Image.fromarray(obj, mode=None)	从array数据创建图像
Image.show()	显示图像
Image.save(fp,format=None)	图像保存(可以完成格式转换)


Image的属性：
属性	描述
Image.format	图像格式
Image.filename	图像的文件名或路径
Image.mode	图像色彩模式，L为灰度，RGB为真彩
Image.size	图像的宽和高，返回二元元组
"""

"""

```



**open方法：**
实例：读取图片并显示，并查看对象属性
```python
from PIL import Image

im = Image.open("img/mcat.png")
im.show()
# im.rotate(45).show() # 将图片旋转，并用系统自带的图片工具显示图片
# 查看对象属性
print(im.size)  # 查看图片大小
print(im.readonly)  # 查看是否为只读，1为是，0为否
print(im.format)  # 查看图片的格式
print(im.info)  # 查看图片的相关信息
print(im.mode)  # 查看图片的模式


```
![20220717102300](https://image.geoer.cn/20220717102300.png)


#### 创建图像
**Image.new**
```python
# im = Image.new(mode,size,color)  # 创建图片
# im.show()  # 展示图片
"""
参数说明如下：
mode：图像模式，字符串参数，比如 RGB（真彩图像）、L（灰度图像）、CMYK（色彩图打印模式）等
size：图像大小，元组参数（width, height）代表图像的像素大小
color：图片颜色，默认值为 0 表示黑色，参数值支持（R,G,B）三元组数字格式、颜色的十六进制值以及颜色英文单词
"""

from PIL import Image

# 创建一个灰度图像
newL = Image.new("L", (28, 28), 255)
newL.show()

# 创建一个RGB图像
im1 = Image.new("RGB", (28, 28), (20, 200, 45))
im1.show()

im2 = Image.new("RGBA", (28, 28), (20, 200, 45, 255))
im2.show()
```

**以数组的形式创建图像，PIL.image.fromarray(obj,mode=None)**
```python
"""
PIL.image.fromarray(obj,mode=None)
obj - 图像的数组，类型可以是numpy.array()
mode - 如果不给出，会自动判断

可以将一个数组（具体一点就是像素数组）转换为图像，从图像的本质去处理图像
"""


"""
用fromarray()函数实现图像的灰度化（使用了两种方法）
"""

from PIL import Image
import numpy as np


im = Image.open("img/mcat.png")
# print(im.size)  # (362, 386)
# print(im.getdata())

b = im.resize((362, 386))
bdata_li = list(b.getdata())
# print(bdata_li)
# print(type(bdata_li))  # <class 'list'>

obj1 = []
obj2 = []
for i in range(len(bdata_li)):
    obj1.append([sum(bdata_li[i])/3])  # 灰度化方法1：RGB三个分量的均值
    obj2.append([0.3*bdata_li[i][0]+0.59*bdata_li[i][1]+0.11*bdata_li[i][2]])  # 灰度化方法2：根据亮度与RGB三个分量的对应关系：Y=0.3*R+0.59*G+0.11*B


obj1 = np.array(obj1).reshape((362, 386))
obj2 = np.array(obj2).reshape((362, 386))

array_img1 = Image.fromarray(obj1)
array_img2 = Image.fromarray(obj2)
array_img1.show()
array_img2.show()


```



#### 复制和粘贴图像
```python
from PIL import Image

im = Image.open("img/mcat.png")
print(im.format)

im3 = im.copy() # 复制图像
im3.show()
```

粘贴
```python
# im_copy.paste(image, box=None, mask=None)
# mask：可选参数，为图片添加蒙版效果

from PIL import Image

im = Image.open(r"7.jpg")
# 复制一张图片副本
im_copy = im.copy()
# 对副本进行裁剪
im_crop = im_copy.crop((0, 0, 200, 100))
# 创建一个新的图像作为蒙版，L模式，单颜色值
image_new = Image.new('L', (200, 100), 200)
# 将裁剪后的副本粘贴至副本图像上，并添加蒙版
im_copy.paste(im_crop, (100, 100, 300, 200), mask=image_new)
# 显示粘贴后的图像
im_copy.show()

```



#### 获取图像信息
```python

# -*- coding:utf-8 -*-
from PIL import Image
img1 = Image.open("test.png")
img1.show()

# getbands() - 显示该图像的所有通道，返回一个tuple
bands = img1.getbands()
print bands

# getbbox() - 返回一个像素坐标，4个元素的tuple
bboxs = img1.getbbox()
print bboxs

# getcolors() - 返回像素信息，是一个含有元素的列表[(该种像素的数量，(该种像素)),(...),...]
colors = img1.getcolors()
print colors

# getdata() - 返回图片所有的像素值，要使用list()才能显示出具体数值
#data = list(img1.getdata())
#print data

# getextrema() - 获取图像中每个通道的像素最小和最大值,是一个tuple类型
extremas = img1.getextrema()
print extremas

# getpixel() - 获取该坐标
pixels = img1.getpixel((87,180))
print pixels

# histogram() - 返回图片的像素直方图
print(img1.histogram())
```



### 图像处理

#### 格式转换
##### save 方法
> save 方法用于保存 图像，当不指定文件格式时，它会以默认的图片格式来存储；如果指定图片格式，则会以指定的格式存储图片


```python
"""
im.save(outfile, format, options…)
如果变量format缺省，如果可能的话，则从文件名称的扩展名判断文件的格式。该方法返回为空。关键字options为文件编写器提供一些额外的指令
"""

```
比如：jpg转为png：
```python
from PIL import Image
im = Image.open("3d.jpg")
print(im)
im.save("3d.png")     ## 将"3d.jpg"保存为3d.png"
im = Image.open("3d.png")  ##打开新的png图片
print(im.format, im.size, im.mode)

```

Note: 并非所有的图片格式都可以用 save() 方法转换完成，比如将 PNG 格式的图片保存为 JPG 格式，如果直接使用 save() 方法就会出现错误(引发错误的原因是由于 PNG 和 JPG 图像模式不一致导致的。其中 PNG 是四通道 RGBA 模式，即红色、绿色、蓝色、Alpha 透明色；JPG 是三通道 RGB 模式。因此要想实现图片格式的转换，就要将 PNG 转变为三通道 RGB 模式)





##### convert 方法
> 在做深度学习的时候，我们首先会用到python PIL模块中的convert函数将原始图片（例如png）转化为对应的像素值，再将像素值转化成tensor之后进行模型的训练


Image 类提供的 convert() 方法可以实现图像模式的转换。该函数提供了多个参数，比如 mode、matrix、dither 等，其中最关键的参数是 mode，其余参数无须关心
```python
"""
语法：
im.convert(mode, params)  # mode：指的是要转换成的图像模式， params：其他可选参数
im.save(fp)  # 保存图片

"""
```

图像模式：
![20220717102735](https://image.geoer.cn/20220717102735.png)

**其中最常用的有三种：**
**1.RGB模式：**
RGB色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，每个通道的值在0~255之间。
```python
im2 = im.convert('RGB')
```

png转jpg：
```python

from PIL import Image

im = Image.open("img/mcat.png")
print(im.format)

im3 = im.convert("RGB")
im3.save("img/mcat_jpg.jpg")
im2 = Image.open("img/mcat_jpg.jpg")
im2.show()

```

**2.1模式**
例子：二值化
```python
from PIL import Image

im = Image.open("img/mcat.png")
im.show()

# '1'为二值图像，非黑即白。每个像素用8个bit表示，0表示黑，255表示白。
im2 = im.convert('1')
im2.show()

```
![20220717103442](https://image.geoer.cn/20220717103442.png)



**3.L模式**
灰度图像：
```python
from PIL import Image

im = Image.open("img/mcat.png")
im.show()

# 'L'为灰度图像，每个像素用8个bit表示，0表示黑，255表示白，其他数字表示不同的灰度。
# 转换公式：L = R * 299/1000 + G * 587/1000+ B * 114/1000
im2 = im.convert('L')
im2.show()

```
![20220717103640](https://image.geoer.cn/20220717103640.png)





#### 缩放图片
```python
"""
im_new = im.resize(size, resample=image.BICUBIC, box=None, reducing_gap=None)  # 注意要重新赋值
im_new.show()  # 缩放后的图片


参数：

size：元组参数 (width,height)，图片缩放后的尺寸

resample：可选参数，指图像重采样滤波器，与 thumbnail() 的 resample 参数类似，默认为 Image.BICUBIC

box：对指定图片区域进行缩放，box 的参数值是长度为 4 的像素坐标元组，即 (左,上,右下)。注意，被指定的区域必须在原图的范围内，如果超出范围就会报错。当不传该参数时，默认对整个原图进行缩放
(0, 0, 120, 180)代表的是以原图的左上角为原点，选择宽和高分别是(120,180)的图像区域

reducing_gap：可选参数，浮点参数值，用于优化图片的缩放效果，常用参数值有 3.0 和 5.0

"""


from PIL import Image


im = Image.open("img/mcat.png")
im.show()

b = im.resize((100, 200))
b.show()

```
![20220717111201](https://image.geoer.cn/20220717111201.png)



#### 创建缩略图
缩略图指的是将原图缩小至一个指定大小（size）的图像。通过创建缩略图可以使图像更易于展示和浏览
```python
"""
im.thumbnail(size,resample)  # 直接在原图的基础上修改
im.show()  # 缩放后的图片


参数：
size：元组参数，指的是缩小后的图像大小
resample：可选参数，指图像重采样滤波器，有四种过滤方式，分别是 Image.BICUBIC（双立方插值法）、PIL.Image.NEAREST（最近邻插值法）、PIL.Image.BILINEAR（双线性插值法）、PIL.Image.LANCZOS（下采样过滤插值法），默认为 Image.BICUBIC
"""

from PIL import Image


im = Image.open("img/mcat.jpg")
# im.show()

im.thumbnail((100, 200))
im.show()
im.save(file+".thumbnail","JPEG")
# 缩略图不能直接双击打开，而可以使用PIL.image的open读取，然后使用show()方法进行显示。


```
比较：
- resize()方法可以缩小也可以放大，而thumbnail()方法只能缩小；
- resize()方法不会改变对象的大小，只会返回一个新的Image对象，而thumbnail()方法会直接改变对象的大小，返回值为none；
- resize()方法中的size参数直接规定了修改后的大小，而thumbnail()方法按比例缩小，size参数只规定修改后size的最大值。


#### 图像裁剪
Image.crop(box)	
```python
from PIL import Image, ImageFilter

img = Image.open(r"img/mcat.png")
box = (0, 58, 100, 200)
# 区域由一个4元组定义，表示为坐标是 (left, upper, right, lower)，Python Imaging Library 使用左上角为 (0, 0)的坐标系统
im2 = img.crop(box)

im2.show()


```



#### 图像分离与合并
##### split方法
```python
from PIL import Image


im = Image.open("img/mcat_jpg.jpg")
r, g, b = im.split()  # split 方法使用较简单，分离通道
r.show()
g.show()
b.show()


```
![20220717111724](https://image.geoer.cn/20220717111724.png)



##### merge方法

```python
"""
im_merge = PIL.Image.merge(mode, bands)
im_merge.show()

参数：
mode：指定输出图片的模式
bands：参数类型为元组或者列表序列，其元素值是组成图像的颜色通道，比如 RGB 分别代表三种颜色通道，可以表示为 (r, g, b)
"""

from PIL import Image


im = Image.open("img/mcat.png")
r, g, b, a = im.split()  # split 方法使用较简单，分离通道
r.show()
g.show()
b.show()

# merge合并为rgb的
im2 = Image.merge('RGB', (r, g, b))
im2.show()

```





##### blend方法
Image 类也提供了 blend() 方法来混合 RGBA 模式的图片（PNG 格式）
```python
"""
PIL.Image.blend(image1,image2, alpha)

参数：
image1：图片对象1
image2：图片对象2
alpha：透明度 ，取值范围为 0 到 1，当取值为 0 时，输出图像相当于 image1 的拷贝，而取值为 1 时，则是 image2 的拷贝，只有当取值为 0.5 时，才为两个图像的中合。因此改值的大小决定了两个图像的混合程度

"""


from PIL import Image 
  
# creating a image1 object and convert it to mode 'P' 
im1 = Image.open(r"1.PNG").convert('L') 
  
# creating a image2 object and convert it to mode 'P' 
im2 = Image.open(r"2.PNG").convert('L') 
  
# alpha is 0.0, a copy of the first image is returned 
im3 = Image.blend(im1, im2, 0.0) 
  
# to show specified image  
im3.show()
```


##### alpha_composite(im1,im2)
```python
from PIL import Image
# 图片合成

# PIL的alpha_composite(im1,im2) 图像通道融合
# im2要和im1的size和mode一致，且要求格式为RGBA
im1 = Image.open("test.png")
im2 = Image.open("test2.png")

newim1 = Image.alpha_composite(im1,im2) # 将im2合成到im1中，如果其一是透明的，
# 才能看到结果，不然最终结果只会显示出im2
newim1.show()
#print(im1.mode)

```

##### composite(im1,im2,mask)
```python
mask = Image.open("mask.png")
newim3 = Image.composite(im2,im1,mask)
newim3.show()
```


#### 几何变化
##### transpose
该函数可以实现图像的垂直、水平翻转

语法：
```python
im_out = im.transpose(method)  # 生成新的图像对象
"""
method取值：

Image.FLIP_LEFT_RIGHT：左右水平翻转
Image.FLIP_TOP_BOTTOM：上下垂直翻转
Image.ROTATE_90：图像逆时针旋转 90 度
Image.ROTATE_180：图像旋转 180 度
Image.ROTATE_270：图像旋转 270 度
Image.TRANSPOSE：图像转置
Image.TRANSVERSE：图像横向翻转

"""

```

##### rotate
当我们想把图像旋转任意角度时，可以使用 rotate() 函数
语法：
```python
im_out = im.rotate(angle, resample=PIL.Image.NEAREST, expand=None, center=None, translate=None, fillcolor=None)  # 返回图像对象

"""
参数：
angle：表示任意旋转的角度
resample：重采样滤波器，默认为 PIL.Image.NEAREST 最近邻插值方法
expand：可选参数，表示是否对图像进行扩展，如果参数值为 True 则扩大输出图像，如果为 False 或者省略，则表示按原图像大小输出
center：可选参数，指定旋转中心，参数值是长度为 2 的元组，默认以图像中心进行旋转
translate：参数值为二元组，表示对旋转后的图像进行平移，以左上角为原点；translate的参数值可以为负数
fillcolor：可选参数，填充颜色，图像旋转后，对图像之外的区域进行填充

"""
```


##### transform
该函数能够对图像进行变换操作，通过指定的变换方式，产生一张规定大小的新图像
语法：
```python
im_out = im.transform(size, method, data=None, resample=0)  # 返回图像对象

"""
参数：
size：指定新图片的大小
method：指定图片的变化方式，比如 Image.EXTENT 表示矩形变换
data：该参数用来给变换方式提供所需数据
resample：图像重采样滤波器，默认参数值为 PIL.Image.NEAREST
"""

```






#### 图像滤波
滤波器能够有效抑制噪声的产生，并且不影响被处理图像的形状、大小以及原有的拓扑结构
Pillow 通过`ImageFilter`类达到图像降噪的目的，该类中集成了不同种类的滤波器，通过调用它们从而实现图像的平滑、锐化、边界增强等图像降噪操作。
对图像添加一些滤镜效果

图像降噪滤波器：
![](https://image.geoer.cn/20220717113217.png)

语法：`im_ft = im.filter(filt_mode)  # 返回图像对象，里面传入滤波器`


实例：浮雕滤波
```python
from PIL import Image, ImageFilter

im = Image.open(r"img/mcat_jpg.jpg")
im_ft = im.filter(ImageFilter.EMBOSS)  # 添加浮雕滤波器
im_ft.show()

```
![](https://image.geoer.cn/20220717113338.png)


实例：轮廓滤波
```python
from PIL import Image, ImageFilter

im = Image.open(r"img/mcat_jpg.jpg")
im_ft = im.filter(ImageFilter.EMBOSS)  # 添加浮雕滤波器
im_ft.show()

```
![](https://image.geoer.cn/20220717124056.png)



实例：中值滤波
```python
from PIL import Image, ImageFilter

img = Image.open(r"img/mcat.png")
img.show()
# BLUR - 模糊处理
# CONTOUR - 轮廓处理
# DETAIL - 增强
# EDGE_ENHANCE - 将图像的边缘描绘得更清楚
# EDGE_ENHANCE_NORE - 程度比EDGE_ENHANCE更强
# EMBOSS - 产生浮雕效果
# SMOOTH - 效果与EDGE_ENHANCE相反，将轮廓柔和
# SMOOTH_MORE - 更柔和
# SHARPEN - 效果有点像DETAIL
filter_img = img.filter(ImageFilter.MedianFilter)
filter_img.show()
```
![20220717113622](https://image.geoer.cn/20220717113622.png)


#### 图像增强
增强（或减弱）图像的亮度、对比度、色度和锐度
![](https://image.geoer.cn/20220717114111.png)

1、ImageEnhance.Brightness：调整图像的亮度
```python
from PIL import Image, ImageFilter, ImageEnhance

img = Image.open(r"img/mcat.png")
print(img.size)
# factor 为 0 时生成纯黑图像，为 1 时还是原始图像，该值小于1为亮度减弱，大于1为亮度增强，值越大，图像越亮。
enh_bri = ImageEnhance.Brightness(img)
new_img1 = enh_bri.enhance(factor=0)
new_img2 = enh_bri.enhance(factor=0.5)
new_img3 = enh_bri.enhance(factor=1)
new_img4 = enh_bri.enhance(factor=1.5)

new_img1.show()
new_img2.show()

```

2、ImageEnhance.Color：调整图像的色彩平衡
```python
img = Image.open(img_path)
enh_col = ImageEnhance.Color(img)
new_img = enh_col.enhance(factor=1.5)
# factor 为 0 时生成灰度图像，为 1 时还是原始图像，该值越大，图像颜色越饱和。
```


3、ImageEnhance.Contrast：调整图像的对比度
```python
img = Image.open(img_path)
enh_con = ImageEnhance.Contrast(img)
new_img = enh_con.enhance(factor=1.5)
# factor 为 0 时生成全灰图像，为 1 时还是原始图像，该值越大，图像颜色对比越明显。

```

4、ImageEnhance.Sharpness：调整图像的锐化程度
```python
img = Image.open(img_path)
enh_sha = ImageEnhance.Sharpness(img)
new_img = enh_sha.enhance(factor=1.5)
# factor 为 0 时生成模糊的图像，为 1 时还是原始图像，为 2 时生成完全锐化的图像。

```




#### ImageColor
...



#### ImageFont
...


### Image与Numpy

```python
from PIL import Image
import numpy as np

im = Image.open("1.jpg")
print(np.asarray(im))  # 三维数组
# print(im.shape)  # 输出图片的尺寸
# im_arr = np.array(im) # 将图片转化为numpy矩阵 
na = np.asarray(im)  # 将图片转换为数组
na[0][0][0] = 0  # 修改数组的值
im_new = Image.fromarray(na)  # 将数组转换为图片


```




### 将多个小图拼接为一个大图
```python

import os
from PIL import Image


IMAGES_PATH = r'img/'  # 图片集地址
IMAGES_FORMAT = ['.jpg', '.JPG']  # 图片格式
IMAGE_SIZE = 96  # 每张小图片的大小
IMAGE_SIZE_x = 96
IMAGE_ROW = 2  # 图片间隔，也就是合并成一张图后，一共有几行
IMAGE_COLUMN = 4  # 图片间隔，也就是合并成一张图后，一共有几列
IMAGE_SAVE_PATH = 'final.jpg'  # 图片转换后的地址

# 获取图片集地址下的所有图片名称
image_names = [name for name in os.listdir(IMAGES_PATH) for item in IMAGES_FORMAT if os.path.splitext(name)[1] == item]

# 简单的对于参数的设定和实际图片集的大小进行数量判断
if len(image_names) != IMAGE_ROW * IMAGE_COLUMN:
    raise ValueError("合成图片的参数和要求的数量不能匹配！")

print(image_names)
# 定义图像拼接函数
def image_compose():
    to_image = Image.new('RGB', (IMAGE_COLUMN * IMAGE_SIZE_x, IMAGE_ROW * IMAGE_SIZE))  # 创建一个新图
    # 循环遍历，把每张图片按顺序粘贴到对应位置上
    for y in range(1, IMAGE_ROW + 1):
        for x in range(1, IMAGE_COLUMN + 1):
            from_image = Image.open(IMAGES_PATH + image_names[IMAGE_COLUMN * (y - 1) + x - 1]).resize((IMAGE_SIZE_x, IMAGE_SIZE), Image.ANTIALIAS)
            to_image.paste(from_image, ((x - 1) * IMAGE_SIZE_x, (y - 1) * IMAGE_SIZE))
    return to_image.save(IMAGE_SAVE_PATH)  # 保存新图


image_compose()

```
![](https://image.geoer.cn/20220717120507.png)








---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python-pil%E5%BA%93/  

