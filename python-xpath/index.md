# python-XPath



  
###### XPath

XPath，全称 XML Path Language，即 XML 路径语言，它是一门在 XML 文档中查找信息的语言。最初是用来搜寻 XML 文档的，但同样适用于 HTML 文档的搜索。所以在做爬虫时完全可以使用 XPath 做相应的信息抽取。
  



  
### python库lxml的安装

```bash
pip3 install lxml
```


  
### XPath常用规则

| 表达式            | 描述                                       |
| ----------------- | ------------------------------------------ |
| nodename          | 选取此节点的所有子节点                     |
| /                 | 从当前节点选取直接子节点                   |
| //                | 从当前节点选取子孙节点                     |
| .                 | 选取当前节点                               |
| ..                | 选取当前节点的父节点                       |
| @                 | 选取属性                                   |
| *                 | 通配符，选择所有元素节点与元素名           |
| @*                | 选取所有属性                               |
| [@attrib]         | 选取具有给定属性的所有元素                 |
| [@attrib='value'] | 选取给定属性具有给定值的所有元素           |
| [tag]             | 选取所有具有指定元素的直接子节点           |
| [tag='text']      | 选取所有具有指定元素并且文本内容是text节点 |



### XPath中的运算符

| 运算符 | 描述             | 实例              | 返回值                                         |
| ------ | ---------------- | ----------------- | ---------------------------------------------- |
| or     | 或               | age=19 or age=20  | 如果age等于19或者等于20则返回true反正返回false |
| and    | 与               | age>19 and age<21 | 如果age等于20则返回true，否则返回false         |
| mod    | 取余             | 5 mod 2           | 1                                              |
| \|     | 取两个节点的集合 | //book \| //cd    | 返回所有拥有book和cd元素的节点集合             |
| +      | 加               | 6+4               | 10                                             |
| -      | 减               | 6-4               | 2                                              |
| *      | 乘               | 6*4               | 24                                             |
| div    | 除法             | 8 div 4           | 2                                              |
| =      | 等于             | age=19            | true                                           |
| !=     | 不等于           | age!=19           | true                                           |
| <      | 小于             | age<19            | true                                           |
| <=     | 小于或等于       | age<=19           | true                                           |
| >      | 大于             | age>19            | true                                           |
| >=     | 大于或等于       | age>=19           | true                                           |









### 解析方式

```python
from lxml import etree

text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0"><a href="link5.html">a属性</a>
     </ul>
 </div>
'''

# 1.读取文本解析节点
html=etree.HTML(text) #初始化生成一个XPath解析对象
result=etree.tostring(html,encoding='utf-8')   #解析对象输出代码


# 2.读取HTML文件进行解析
html=etree.parse('test.html',etree.HTMLParser()) #指定解析器HTMLParser会根据文件修复HTML文件中缺失的如声明信息
result=etree.tostring(html)   #解析成字节
#result=etree.tostringlist(html) #解析成列表




```



#### 获取元素节点

```python
# 1.获取所有节点             //
from lxml import etree

html=etree.parse('test.html',etree.HTMLParser())
result=html.xpath('//*')  #//代表获取子孙节点，*代表获取所有

# 如要获取li节点，可以使用//后面加上节点名称，然后调用xpath()方法
html.xpath('//li')   #获取所有子孙节点的li节点



# 2.获取子节点            /
result=html.xpath('//li/a')  #通过追加/a选择所有li节点的所有直接a节点，因为//li用于选中所有li节点，/a用于选中li节点的`所有直接子节点a`



# 3.获取父节点       ..           parent::
# 使用`..`来实现也可以使用`parent::`来获取父节点
html=etree.HTML(text, etree.HTMLParser())
result=html.xpath('//a[@href="link2.html"]/../@class')
result1=html.xpath('//a[@href="link2.html"]/parent::*/@class'

                   
                   
# 4.根据属性匹配节点             //div[@class='box']
html=etree.HTML(text, etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]')
print(result)                


                   
# 5.获取节点中的文本       text()
html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]/a/text()') #获取a节点下的内容
result1=html.xpath('//li[@class="item-1"]//text()') #获取li下所有子孙节点的内容           
                   
                   
                   
# 6. 获取属性               @href, @class,  @src
result=html.xpath('//li/a/@href')  #获取a的href属性
result=html.xpath('//li//@href')   #获取所有li子孙节点的href属性
                   
                   
# 7. 属性多值匹配节点(某个属性的值有多个时，一个节点有多个属性值)     	contains()
text1='''
<div>
    <ul>
         <li class="aaa item-0"><a href="link1.html">第一个</a></li>
         <li class="bbb item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result1=html.xpath('//li[contains(@class,"aaa")]/a/text()')              
                   
                   

# 8.多属性匹配（根据多个属性确定一个节点， 多个节点有相同的属性值）      and
text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="fore"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[@class="aaa" and @name="fore"]/a/text()')
result1=html.xpath('//li[contains(@class,"aaa") and @name="fore"]/a/text()')
 
                   

                   
# 9. 按序选择
# 可能同时匹配多个节点，但我们只想要其中的某个节点，如第二个节点或者最后一个节点，这时可以利用中括号引入索引的方法获取特定次序的节点：   [1], [last()],    position()
result=html.xpath('//li[contains(@class,"aaa")]/a/text()') #获取所有li节点下a节点的内容
result1=html.xpath('//li[1][contains(@class,"aaa")]/a/text()') #获取第一个
result2=html.xpath('//li[last()][contains(@class,"aaa")]/a/text()') #获取最后一个
result3=html.xpath('//li[position()>2 and position()<4][contains(@class,"aaa")]/a/text()') #获取第三个
result4=html.xpath('//li[last()-2][contains(@class,"aaa")]/a/text()') #获取倒数第三个
# XPath的函数： http://www.w3school.com.cn/xpath/xpath_functions.asp
        
                   
                   
# 10. 节点轴选择
# 包括获取子元素、兄弟元素、父元素、祖先元素等        
result=html.xpath('//li[1]/ancestor::*')  #获取所有祖先节点
result1=html.xpath('//li[1]/ancestor::div')  #获取div祖先节点
result2=html.xpath('//li[1]/attribute::*')  #获取所有属性值
result3=html.xpath('//li[1]/child::*')  #获取所有直接子节点
result4=html.xpath('//li[1]/descendant::a')  #获取所有子孙节点的a节点
result5=html.xpath('//li[1]/following::*')  #获取当前子节之后的所有节点
result6=html.xpath('//li[1]/following-sibling::*')  #获取当前节点的所有同级节点            
# 更多轴的用法可参考：http://www.w3school.com.cn/xpath/xpath_axes.asp             
                   
```



















> 本文参考：https://www.cnblogs.com/zhangxinqi/p/9210211.html        博主写的很详细，也很美观

> XPath的更多用法参考：http://www.w3school.com.cn/xpath/index.asp

> python lxml库的更多用法参考：http://lxml.de/















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python-xpath/  

