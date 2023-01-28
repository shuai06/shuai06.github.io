# python-PyQuery



  
**Python爬虫解析库，主流的有**

- PyQuery
- Beautifulsoup
- Scrapy Selectors
- 正则表达式。

PyQuery和scrapy Selectors都是基于lxml模块，而lxml和正则表达式都是C语言写的，只有Beautifulsoup是用纯Python编写的，所以在实测中，`Beautifulsoup` 的解析速度比其他几种`慢了5倍以上`！

`正则表达式`的构造稍微`复杂`一点，一般在结构化的网页中没必要用正则（易出错）;

Scrapy Selectors支持css，xpath以及正则表达式 ;

`PyQuery`只支持css（css语法更精简一些），` 类似于jQuery`

Scrapy Selector中的css语法和PyQuery中的略有不同，本文以PyQuery为例（不用Scrapy框架的话，PyQuery就够用了）



----


  
### 安装

```bash
pip3 install pyquery
```



### 使用

###### 初始化

```python
import requests
from pyquery import PyQuery as pq


url = 'https://www.guokr.com/'
r = requests.get(url)

# 实例化
doc = pq(r.text)

```



###### 获取元素

跟jQuery是一样的，也就不细说了，简单举几个小例子

```python
# 1.获取class是content-title的元素
# class是点.        id是#
print(doc('h2.content-title'))      # 获取果壳网

# 2.获取标签的文本内容
print(doc('h2.content-title').text())

# 3.遍历（先items()）
lis = doc('h2.content-title').items()
for li in lis:
    print(li.text())


# 4. 获取class为content的div标签下面的ul下面的li
# 用空格表示子孙节点
lis = doc('div.content ul li').items()
#lis = doc('div.content li').items()
for i in lis:
    print(i.text())
    
    
# 5. 如果类名不唯一，还可以加上其他的类名一起定位
print(doc('div.cont.a.b.c.d'))
# 标签里的空格表示`并列`，表示这个div标签有cont,a,b,c,d这五个类名，但在css语法里空格表示`嵌套`，所以我们要添加其他类名的时候`不能输入空格`，而是直接用`小数点`来添加其他类名
    
    
    
# 6. 获取属性         attr("属性名")
lis = doc('div.content li').items()
for i in lis:
    print(i.text(),i('a').attr('href'))

    
    
# 7. 其他的一些选择器
lis = doc('div.content ul li')
#父节点,包含父节点的所有子孙节点的内容
#相当于
#print(doc('div.content ul'))
print(lis.parent())
#祖先节点,就相当于所有源代码了
print(lis.parents())
#兄弟节点,即同级节点，不包含自己
print(lis.siblings)

```



#### 其他技巧

1.伪类选择器

```python
#第二个标签
lis = doc('div.content li:nth-child(2)').items()
for i in lis:
    print(i.text(),i('a').attr('href'))
    
# 第一个a标签的语法是 a:first-child,最后一个是a:last-child,其它位置的语法如上图所示，第几个括号里就是几（当然第一个你也可以写成 li:nth-child(1))

```

类似的

```python
#div.content 下面第二个（含）之后的li标签
lis = doc('div.content li:gt(1)').items()
for i in lis:
    print(i.text(),i('a').attr('href'))
    
# gt就是greater than,大于的意思，lt (less than)是小于
```

筛选文本

```python
lis = doc('div.content ul').items()
for i in lis:
    #文本包含问号的li标签
    print(i("li:contains('？')").text())
```









2.修改标签属性

```python
#用remove把特定标签移除，然后再进行遍历
lis = doc('div.content ul').remove('.content-article').items()
for i in lis:
    print(i.text())
    
    
# 还有如修改属性，增加css之类的一些使用率较低的，用到的时候去官方文档查
```



###### 

###### 直接在Chrome里调试



```python

Chrome浏览器自带css的查询方法，按f12或者右键检查，打开Elements面板，按ctrl+f，

这里支持xpath，css语法，以及普通的字符查找

要注意的是右边的数字，显示的是满足条件的标签数量，可以按向下的箭头过一遍，看看是不是自己想要的信息。

```



京东的商品栏目

```python
import requests
from pyquery import PyQuery as pq


url = 'https://www.jd.com/'
r = requests.get(url)

# 实例化
doc = pq(r.text)

flag = 1
# for i in doc("li.cate_menu_item:nth-child(3) a").items():
for i in doc("li.cate_menu_item:gt(1) a").items():
    # print(i.text())
    print(i.attr('href'))
    print(i.text())
    flag = flag + 1
    print("------------")

print(flag)

```











> 参考文章：https://www.jianshu.com/p/7eb136bbe317

> 更多用法：https://pythonhosted.org/pyquery/



















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python-pyquery/  

