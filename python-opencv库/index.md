# Python OpenCV库

<script type="text/javascript" src="/js/src/bai.js"></script>

> 图像处理，怎么能不学OpenCV呢



## 前言
OpenCV-Python使用Numpy，这是一个高度优化的数据库操作库，具有MATLAB风格的语法。所有OpenCV数组结构都转换为Numpy数组。这也使得与使用Numpy的其他库（如SciPy和Matplotlib）集成更容易。
OpenCV依赖一些库，比如Numpy，自行安装。
![20220717132210](https://image.geoer.cn/20220717132210.png)


OpenCV应用领域：
1、计算机视觉领域方向
2、人机互动
3、物体识别
4、图像分割
5、人脸识别
6、动作识别
7、运动跟踪
8、机器人
9、运动分析
10、机器视觉
11、结构分析
12、汽车安全驾驶



## 安装：
```bash
# pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python
conda install -c conda-forge opencv
```

关于M1下安装请看：https://blog.csdn.net/jjw_zyfx/article/details/119851647


## 读取操作
```python
import cv2


# 打开 & 显示
im = cv2.imread("./mcat_jpg.jpg")  # 读取的图像直接表示ndarray类型的三维矩阵，彩色图片的维度是(h,w,c)
cv2.imshow("image", im)  # 显示图片
cv2.waitKey(5)  # 需配合cv2.waitKey()才能显示


# 保存图片：cv2.imwrite(filename,image)
cv2.imwrite("mcat_cv.jpg", im)


# 用cv2打开的彩色三通道的通道顺序是BGR，而不是RGB。
# 可以使用split()和merge()方法实现通道转换
im3 = cv2.imread("./mcat_jpg.jpg")
b,g,r=cv2.split(im3)
im4=cv2.merge((r,g,b))
cv2.imshow('ImWindow',im4)
cv2.waitKey()



```

![20220717132906](https://image.geoer.cn/20220717132906.png)

**PIL库和opencv库在读取图片上的差异：**
- opencv：图片的通道顺序为BGR，显示的尺寸为（高/行数，宽/列数，通道数）
- PIL：通道顺序为RGB，显示的尺寸为（宽，高）
![](https://image.geoer.cn/20220717132025.png)


## 获取并修改图像中的像素点
```python
import numpy as np
import cv2 as cv

img = cv.imread('1.jpg')
# 获取某个像素点的值
px = img[100,100]
# 仅获取蓝色通道的强度值
blue = img[100,100,0]
# 修改某个位置的像素值
img[100,100] = [255,255,255]

```

## 获取图像属性
![20220717140247](https://image.geoer.cn/20220717140247.png)


## 图像通道的拆分与合并
```python
# 通道拆分
b,g,r = cv.split(img)
# 通道合并
img = cv.merge((b,g,r))

```



## 基本变换
### 图像缩放：resize()
```python
![20220717135713](https://image.geoer.cn/20220717135713.png)
"""
参数：
src: 输入图像对象
dsize：输出矩阵/图像的大小，为0时计算方式如下：dsize = Size(round(fxsrc.cols),round(fysrc.rows))
fx: 水平轴的缩放因子，为0时计算方式： (double)dsize.width/src.cols
fy: 垂直轴的缩放因子，为0时计算方式： (double)dsize.heigh/src.rows
interpolation：插值算法
 cv2.INTER_NEAREST : 最近邻插值法
 cv2.INTER_LINEAR 默认值，双线性插值法
 cv2.INTER_AREA 基于局部像素的重采样（resampling using pixel area relation）。对于图像抽取（image decimation）来说，这可能是一个更好的方法。但如果是放大图像时，它和最近邻法的效果类似。
 cv2.INTER_CUBIC 基于4x4像素邻域的3次插值法
  cv2.INTER_LANCZOS4 基于8x8像素邻域的Lanczos插值
  cv2.INTER_AREA 适合于图像缩小， cv2.INTER_CUBIC (slow) & cv2.INTER_LINEAR 适合于图像放大


"""


import cv2

img = cv2.imread("./mcat_jpg.jpg")  # 读取的图像直接表示ndarray类型的三维矩阵，彩色图片的维度是(h,w,c)

width = 350
height = 450
# width = int(img.shape[1] * scale_percent / 100) # 保留宽高比
# height = int(img.shape[0] * scale_percent / 100)
dim = (width, height)

# resize image
resized = cv2.resize(img, dim, interpolation=cv2.INTER_AREA)

print('Resized Dimensions : ', resized.shape)

cv2.imshow("Resized image", resized)
cv2.waitKey(0)
cv2.destroyAllWindows()

```






### 颜色空间的转换：cv2.cvtColor()
```python
"""
参数： 
img: 图像对象
code：
cv2.COLOR_RGB2GRAY: RGB转换到灰度模式
cv2.COLOR_RGB2HSV： RGB转换到HSV模式（hue,saturation,Value）
cv2.COLOR_BGR2RGB：RGB通道转换
"""
import cv2
from cv2 import COLOR_RGB2GRAY

im = cv2.imread("./mcat_jpg.jpg")  
gray=cv2.cvtColor(im,COLOR_RGB2GRAY) # RGB转换到灰度模式
cv2.imshow('gray',gray)
cv2.waitKey()

```
![](https://image.geoer.cn/20220717133634.png)



### 图像阈值化：cv2.threshold()
图像阈值分割是一种广泛应用的分割技术，利用图像中要提取的目标区域与其背景在灰度特性上的差异，把图像看作具有不同灰度级的两类区域(目标区域和背景区域)的组合，选取一个比较合理的阈值，以确定图像中每个像素点应该属于目标区域还是背景区域，从而产生相应的二值图像。
```python

"""
cv2.threshold (源图片, 阈值, 填充色, 阈值类型)


有四个参数，第一个原图像，第二个进行分类的阈值，第三个是高于（低于）阈值时赋予的新值，第四个是一个方法选择参数，常用的有：
cv2.THRESH_BINARY（黑白二值）
cv2.THRESH_BINARY_INV（黑白二值反转）
cv2.THRESH_TRUNC （得到的图像为多像素值）
cv2.THRESH_TOZERO
cv2.THRESH_TOZERO_INV
返回两个值
ret:阈值
img：阈值化处理后的图像
"""
import cv2
from cv2 import COLOR_RGB2GRAY, THRESH_BINARY
import matplotlib.path as plt

img = cv2.imread("./mcat_jpg.jpg")  # 读取的图像直接表示ndarray类型的三维矩阵，彩色图片的维度是(h,w,c)
# 一般处理灰度图，所以先转为灰度图
img=cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)

# 不同类型阈值处理
thresh, img1 = cv2.threshold(img, 1, 255, cv2.THRESH_BINARY)
# thresh, img2 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY_INV)
# thresh, img3 = cv2.threshold(img, 127, 255, cv2.THRESH_TRUNC)
# thresh, img4 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO)
# thresh, img5 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO_INV)
#
cv2.imshow('imag1',img1)
cv2.waitKey()

```
![20220717134331](https://image.geoer.cn/20220717134331.png)

阈值类型：
![20220717133853](https://image.geoer.cn/20220717133853.png)


### 旋转：cv2.getRotationMatrix2D()
```python
"""
参数：
center：旋转的中心点坐标
angle：旋转角度，单位为度数，正数表示逆时针旋转
scale：同方向的放大倍数

"""
import cv2

img = cv2.imread("./mcat_jpg.jpg")  # 读取的图像直接表示ndarray类型的三维矩阵，彩色图片的维度是(h,w,c)

rows,cols = img.shape[:2]
# 先获得仿射旋转的矩阵 getRotationMatrix2D
mat_rotation = cv2.getRotationMatrix2D((cols/2, rows/2), 45, 1) # #第一个参数旋转中心，第二个参数旋转角度，第三个参数：缩放比例
# 获得仿射旋转后的图像 warpAffine
res = cv2.warpAffine(img, mat_rotation, (rows,cols))


cv2.imshow('imag1',res)
cv2.waitKey()


```



![20220717134755](https://image.geoer.cn/20220717134755.png)

### 图像的矩阵变换：transpose()
旋转
```python
import cv2

img = cv2.imread("./mcat_jpg.jpg")  # 读取的图像直接表示ndarray类型的三维矩阵，彩色图片的维度是(h,w,c)

image = cv2.transpose(img)

cv2.imshow('imag1',image)
cv2.waitKey()

```
![20220717135018](https://image.geoer.cn/20220717135018.png)

另一种旋转方法：`cv2.flip`：
对图像矩阵进行翻转处理，参数可以设置为1，0，-1，分别对应着水平翻转、垂直翻转、水平垂直翻转。

```python
image1 = cv2.flip(image, -1) #原图顺时针旋转180度
image22 = cv2.flip(image, 0)#等于原图顺时针旋转270度
image33 = cv2.flip(image, -1)


```



OpenCV读入图片的矩阵格式是：（h, w, c）。 深度学习中，因为要对不同通道应用卷积，所以会采取另一种方式（c, h, w）
```python
print(im.shape) # (326,220,3)
im=im.tanspose(2,0,1)  
print(im.shape)  # (3,326,222)

```


在深度学习搭建CNN时，往往要做相应的图像数据处理，比如图像要扩展维度，比如扩展成（batch_size,channels,height,width）
```python
im=np.expand_dims(im,axis=0)
print(im.shape) #(1,3,326,220)

```



## 绘制几何图形
```python
# 绘制直线
cv.line(img,start,end,color,thickness)

# 绘制圆形
cv.circle(img,centerpoint, r, color, thickness)

# 绘制矩形
cv.rectangle(img,leftupper,rightdown,color,thickness)

# 向图像中添加文字
cv.putText(img,text,station, font, fontsize,color,thickness,cv.LINE_AA)

```


## 图像的加法
cv.add()函数把两幅图像相加，或者可以简单地通过numpy操作添加两个图像，如res = img1 + img2。
两个图像应该具有相同的大小和类型，或者第二个图像可以是标量值。

```python
# 注意：OpenCV加法和Numpy加法之间存在差异。OpenCV的加法是饱和操作，而Numpy添加是模运算。
# 这种差别在你对两幅图像进行加法时会更加明显。OpenCV 的结果会更好一点。所以我们尽量使用 OpenCV 中的函数。


>>> x = np.uint8([250])
>>> y = np.uint8([10])
>>> print( cv.add(x,y) ) # 250+10 = 260 => 255
[[255]]
>>> print( x+y )          # 250+10 = 260 % 256 = 4
[4]

```


## 图像加法：
```python
import numpy as np
import cv2 as cv
import matplotlib.pyplot as plt

# 1 读取图像
img1 = cv.imread("view.jpg")
img2 = cv.imread("rain.jpg")

# 2 加法操作
img3 = cv.add(img1,img2) # cv中的加法
img4 = img1+img2 # 直接相加

# 3 图像显示
fig,axes=plt.subplots(nrows=1,ncols=2,figsize=(10,8),dpi=100)
axes[0].imshow(img3[:,:,::-1])
axes[0].set_title("cv中的加法")
axes[1].imshow(img4[:,:,::-1])
axes[1].set_title("直接相加")
plt.show()

```

图像的混合
```python
# 这其实也是加法，但是不同的是两幅图像的权重不同，这就会给人一种混合或者透明的感觉。图像混合的计算公式如下：
# g(x) = (1−α)f0(x) + αf1(x)
# 现在我们把两幅图混合在一起。第一幅图的权重是0.7，第二幅图的权重是0.3。函数cv2.addWeighted()可以按下面的公式对图片进行混合操作。


import numpy as np
import cv2 as cv
import matplotlib.pyplot as plt

# 1 读取图像
img1 = cv.imread("view.jpg")
img2 = cv.imread("rain.jpg")

# 2 图像混合
img3 = cv.addWeighted(img1,0.7,img2,0.3,0)

# 3 图像显示
plt.figure(figsize=(8,8))
plt.imshow(img3[:,:,::-1])
plt.show()

```


## 图像处理

### 图像滤波
```python
import cv2
import numpy as np
from copy import copy

img = cv2.imread(r"C:\Users\pc\Desktop\test14-1.bmp")
new_img = copy(img)

#（20,20）表示左上角开始的坐标，0.5表示字母的大小，(0, 0, 255)表示颜色，1表示粗细
cv2.putText(img, 'Original image', (20, 20), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 255), 1)
cv2.imshow('img', img)

# 均值滤波
# 参数：new_img原图像， (5x5)内核大小，分别表示像素宽度，像素高度
img_mean = cv2.blur(new_img, (5, 5))
cv2.putText(img_mean, 'Mean filtering', (20, 20), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 255), 1)
cv2.imshow('img_mean', img_mean)

# 中值滤波
# ksize：参数表示滤波窗口尺寸，必须是奇数并且大于 1。比如这里是 5，
# 中值滤波器就会使用 5×5 的范围来计算，即对像素的中心值及其 5×5 
# 邻域组成了一个数值集，对其进行处理计算，当前像素被其中值替换掉；
img_median = cv2.medianBlur(new_img, 5)
cv2.putText(img_median, 'Median filtering', (20, 20), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 255), 1)
cv2.imshow('img_median', img_median)

# 高斯滤波
# 0表示高斯核函数在X方向的的标准偏差。
img_Gaussian = cv2.GaussianBlur(new_img, (5, 5), 0)
new_img_Gaussian = copy(img_Gaussian)
cv2.putText(img_Gaussian, 'Gaussian filtering', (20, 20), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 255), 1)
cv2.imshow('img_Gaussian', img_Gaussian)

#高斯边缘检测
edged = cv2.Laplacian(new_img_Gaussian, cv2.CV_16S, ksize=5)
result = cv2.convertScaleAbs(edged)
cv2.putText(result, 'Gaussian edge detection', (20, 20), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 255), 1)
cv2.imshow("edged", result)

cv2.waitKey()
# 关闭窗口并取消分配任何相关的内存使用
cv2.destroyAllWindows()

```





## 视频功能


视频中最常用的就是从视频设备采集图片或者视频，或者读取视频文件并从中采样。所以比较重要的也是两个模块:
- 一个是`VideoCapture`，用于获取相机设备并捕获图像和视频，或是从文件中捕获。
- 还有一个`VideoWriter`，用于生成视频。
来看例子理解这两个功能的用法，首先是一个制作延时摄影视频的小例子：
```python
import cv2
import time

interval = 60           # 捕获图像的间隔，单位：秒
num_frames = 500        # 捕获图像的总帧数
out_fps = 24            # 输出文件的帧率

# VideoCapture(0)表示打开默认的相机
cap = cv2.VideoCapture(0)

# 获取捕获的分辨率
size =(int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)),
       int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)))
       
# 设置要保存视频的编码，分辨率和帧率
video = cv2.VideoWriter(
    "time_lapse.avi", 
    cv2.VideoWriter_fourcc('M','P','4','2'), 
    out_fps, 
    size
)

# 对于一些低画质的摄像头，前面的帧可能不稳定，略过
for i in range(42):
    cap.read()

# 开始捕获，通过read()函数获取捕获的帧
try:
    for i in range(num_frames):
        _, frame = cap.read()
        video.write(frame)

        # 如果希望把每一帧也存成文件，比如制作GIF，则取消下面的注释
        # filename = '{:0>6d}.png'.format(i)
        # cv2.imwrite(filename, frame)

        print('Frame {} is captured.'.format(i))
        time.sleep(interval)
except KeyboardInterrupt:
    # 提前停止捕获
    print('Stopped! {}/{} frames captured!'.format(i, num_frames))

# 释放资源并写入视频文件
video.release()
cap.release()
```


人脸检测：
```python
import cv2


if __name__ == '__main__':

    # 打开摄像头
    cap = cv2.VideoCapture(0)
    # 加载人脸检测器
    faca_detector = cv2.CascadeClassifier('./haarcascade_frontaclcatface.xml')

    # 读取每一帧图像
    while True:
        flag, frame = cap.read()  # flag是否读取了图片
        if not flag:
            break

        # 将图像转化为灰度图像
        gray = cv2.cvtColor(frame, code = cv2.COLOR_BGR2GRAY)
        # 对每一帧灰度图像进行人脸检测
        faces = faca_detector.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=10)
        # 对每一个检测到的人脸区域绘制检测方框
        for x,y,w,h in faces:
            cv2.rectangle(frame,
                          pt1 = (x,y),
                          pt2 = (x+w,y+h),
                          color=[0,0,255],
                          thickness=2)

        # 显示检测到的结果
        cv2.imshow('face', frame)
        # 设置显示时长
        key = cv2.waitKey(1000//24)  # 注意要用整除//，因为毫秒为整数
        # 按q键退出
        if key == ord('q'):
            break

    # 销毁内存
    cv2.destroyAllWindows()
    cap.release()


```

....功能太多了...以后慢慢发现吧......



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/python-opencv%E5%BA%93/  

