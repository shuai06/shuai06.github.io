# python-Beautifulsoup



  
Beautifulsoup也是用来解析网页数据的
  
#### BeautifulSoup`对象四种类型`:
  
- tag
- NavigableString
- BeautifulSoap
- Comment


  
## 初始化

###### 安装

```shell
pip3 install beautifulsoup4

```



###### 导入

```python
from bs4 import BeautifulSoup
```





## 使用

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.title)
```









#### 查找

###### 基本

```python
soup = BeautifulSoup(open("index.html"))  #文件句柄
soup = BeautifulSoup("<html>data</html>")  #字符串

```



###### find 和 find_all

搜索当前 tag 的所有 tag 子节点，并判断是否符合过滤器的条件

**语法：**

```python
find(name=None, attrs={}, recursive=True, text=None, **kwargs) 

find_all(name=None, attrs={}, recursive=True, text=None, limit=None, **kwargs)

```



通过 attrs 参数传递： 

```python
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>') 

print(data_soup.find_all(attrs={"data-foo": "value"}))
```



按 class_ 查找:

```bash
css_soup = BeautifulSoup('<p class="body bold strikeout"></p>') 

print(css_soup.find_all("p", class_="strikeout")) 
print(css_soup.find_all("p", class_="body"))
```



其他搜索方法：

```python
find_parents()　　　　　 # 返回所有祖先节点

find_parent()　　　　　　# 返回直接父节点

find_next_siblings()　　 # 返回后面所有的兄弟节点

find_next_sibling()　　 # 返回后面的第一个兄弟节点

find_previous_siblings() # 返回前面所有的兄弟节点

find_previous_sibling() #　返回前面第一个兄弟节点

find_all_next()　　　　 # 返回节点后所有符合条件的节点

find_next()　　　　　　# 返回节点后第一个符合条件的节点

find_all_previous()　　 # 返回节点前所有符合条件的节点

find_previous()　　　　 # 返回节点前所有符合条件的节点
```



 

从文档中找到所有a标签的链接:

```python
for link in soup.find_all('a'):
    print(link.get('href'))
```



从文档中获取所有文字内容:

```python
print(soup.get_text())

```



#### 对象

###### Tag

`Tag`对象与XML或HTML原生文档中的tag相同

tag中最重要的属性: `name`和`attributes`

- name

  ```python
  tag.name
  
  # 复赋值
  tag.name = "xxx"
  ```

- attributes

  一个tag可能有很多个属性，一个属性可能有很多个值。

  ```python
  # 获取tag的指定属性的属性值
  tag['class']
  
  # 获取tag的全部属性及属性值:
  tag.attrs
  
  # tag属性及属性值添加
  tag['class'] = 'row'
  
  # 属性值的删除
  del tag['class']
  
  # 查看属性值
  print(tag.get('class'))
  
  
  ```

  多值属性tag：   返回类型一般是list,但当某个属性在任何版本的HTML定义中都没有被定义为多值属性时，会将这个属性作为字符串返回

  ```python
  css_soup = BeautifulSoup('<p class="body row"></p>')
  css_soup.p['class']
  # ["body", "row"]
  
  css_soup = BeautifulSoup('<p class="body"></p>')
  css_soup.p['class']
  # ["body"]
  
  id_soup = BeautifulSoup('<p id="my id"></p>')
  id_soup.p['id']
  # 'my id'
  
  
  ```

  > 如果转换的文档是XML格式,那么tag中不包含多值属性

  获取tag某个子节点:   `soup.tag名`， `soup.a`

  获取tag全部子节点:  `.contents` 和 `.children`

  获取tag全部子节点及子孙节点：``.descendants``

  ```python
  for child in head_tag.descendants:
      print(child)
  
  ```

  节点内的字符串:     

  ```python
  # .string
  head_tag.string
  
  # .strings     如果tag中包含多个字符串`.strings` 来循环获取
  head_tag.strings
  for string in soup.strings:
      print(repr(string))
      
  # .stripped_strings  去除空行空白
  for string in soup.stripped_strings:
      print(repr(string))
      
  ```

  父亲节点：`.parent`

  所有父辈节点：`.parents`

  兄弟节点: `.prettify()`, 其中`next_sibling`和`previous_sibling`是前后兄弟

  **回退和前进：**  通过 `.next_elements` 和 `.previous_elements` 的迭代器就可以向前或向后访问文档的解析内容,就好像文档正在被解析一样

  



###### find_all

```python
# 过滤str
soup.find_all('b')


# 过滤正则
for tag in soup.find_all(re.compile("^b")):
   print(tag.name)

# 过滤多个
soup.find_all(["a", "b"])

# True:可以匹配任何值,下面代码查找到所有的`一级`tag
for tag in soup.find_all(True):
   print(tag.name)

# limit 显示返回数量
soup.find_all("a", limit=2)

# 只搜索tag的直接子节点,
soup.html.find_all("title", recursive=False)
```

```python
def test(tag):  #包含 class 属性却不包含 id 属性
   return tag.has_attr('class') and not tag.has_attr('id')

# 调用
soup.find_all(test)
```



```python
#下面两行代码是等价的
soup.find_all('div', limit=1)

soup.find('div')

```

> **区别**:find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.find_all() 方法没有找到目标是返回空列表, find() 方法找不到目标时,返回 None







###### NavigableString

可遍历字符串

通过 `unicode()` 方法可以直接将 `NavigableString` 对象转换成Unicode字符串:

```python
unicode_string = unicode(tag.string)
```





###### BeautifulSoup

表示的是一个文档的全部内容,可以把它当作 `Tag` 对象，是一个特殊的 Tag，我们可以分别获取它的类型，名称，以及属性。



###### Comment

```python
html_doc='<a href="http://z.com" class="row" id="box"><!-- hhh --></a>'

soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.a.string)   # hhh
print(type(soup.a.string))  #  <class 'bs4.element.Comment'>

"""
a 标签里的内容实际上是注释，但是如果我们利用 .string 来输出它的内容，我们发现它已经把注释符号去掉了，所以这可能会给我们带来不必要的麻烦。
"""
# 在使用前最好做一下判断
if type(soup.a.string)==bs4.element.Comment:
    print soup.a.string
```





 























---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python-beautifulsoup/  

