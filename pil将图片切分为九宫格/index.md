# PIL将图片切分为九宫格

<script type="text/javascript" src="/js/src/bai.js"></script>
> 看到一个好玩的方式，学习一下

## 思路
读取图片->填充图片为正方形(fill_image函数)->将图片切分为9张(cut_image函数)->保存图片(save_image)->over


## 代码
```python
from PIL import Image
import sys


# 将图片填充为正方形
def fill_image(image):
    width, height = image.size
    # 选取长和宽中较大值作为新图片的宽
    new_image_length = width if width > height else height
    # 生成新图片[白底]
    new_image = Image.new(image.mode, (new_image_length, new_image_length), color='white')
    # 将之前的图粘贴在新图上，居中
    if width > height:  # 原图宽大于高，则填充图片的竖直维度
        # (x,y)二元组表示粘贴上图相对下图的起始位置
        new_image.paste(image, (0, int((new_image_length - height) / 2)))
    else:
        new_image.paste(image, (int((new_image_length - width) / 2), 0))
    return new_image


# 切图
def cut_image(image):
    width, height = image.size
    item_width = int(width / 3)
    box_list = []
    # (left, upper, right, lower)
    for i in range(0, 3):  # 两重循环，生成9张图片基于原图的位置
        for j in range(0, 3):
            # print((i*item_width,j*item_width,(i+1)*item_width,(j+1)*item_width))
            box = (j * item_width, i * item_width, (j + 1) * item_width, (i + 1) * item_width)
            box_list.append(box)
    image_list = [image.crop(box) for box in box_list]
    return image_list

# 保存
def save_images(image_list):
    index = 1
    for image in image_list:
        image.save(str(index) + '.jpg')
        index += 1


if __name__ == '__main__':
    image_path = "img/mcat_jpg.jpg"
    img = Image.open(image_path)
    # image.show()
    image = fill_image(img)
    image_list = cut_image(image)
    # save_images(image_list)
```




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pil%E5%B0%86%E5%9B%BE%E7%89%87%E5%88%87%E5%88%86%E4%B8%BA%E4%B9%9D%E5%AE%AB%E6%A0%BC/  

