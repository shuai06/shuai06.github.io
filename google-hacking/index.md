# Google Hacking


## 基本

通配符

```bash
通配符				语义				说明					示例
+				包含关键词		+前面必须要有一个空格			 admin +login
-				排除关键词		-前面必须要有一个空格			 mysql -csdn
~				同义词			 ~前面必须要有一个空格		  mysql -csdn
*				模糊查询		*代替任意字符					mysql**
""				强调										 "mysql"

```





# 高级操作符

## 概述

**将学习如下的高级操作符：**

- intitle, allintitle
- related
- inurl, allinurl
- phonebook
- filetype
- rphonebook
- allintext
- bphonebook
- site
- author
- link
- group
- inanchor
- msgid
- daterange
- insubject
- cache
- stocks
- info
- define



## 操作符语法

基本语法：`operator`:`search_term`

#### 原则

- 在操作符、冒号、搜索关键字之间是没有空格的；
- 布尔操作符和特殊字符(例如`OR`和`+`)仍可用于高级操作符查询，但是不能把它们放在冒号之前而把冒号和操作符分开；
- `ALL`操作符(以单词ALL开头的操作符)非常古怪。一般情况下，一个查询中只能使用一次ALL操作符，而且不能和其他操作符混用；
- 操作符搜索的搜索关键字部分的语法和前一章中讲到的关键字语法是一样的。 例如，你可以使用一个单词或者用引号引起来的词组作为关键字。如果用词组作为关键字的话，须保证在操作符、冒号和词组的第一个引号之间没有任何空格字符。



#### 实例

- `intitle:Google`这个查询将返回标题包含单词Google的页面。
- `intitle:"index of"`这个查询将返回标题包含词组index of的页面。回忆前一章中学到的知识，我们可以知道这个查询也可以写成
- `intitle:index.of`,因为句号可以匹配任意字符。利用这种技术，我们可以避免键入词组中的空格以及两端的引号。”
- `intitle:"index of" private` 这 个查询将返回标题包含词组index of,且网页的任何地方(URL、标题、文本等)包含单词private的页面。要注意的是，intitle 只对词组index of起作用，而不会影响单词private,这是因为引号外的第一个空格位于词组index of之后。Google把这个空格解释为高级操作符搜索关键字的结尾，然后接着处理查询中剩下的部分。
- `intitle:"index of" "bakcup files"`这 个查询返回标题包含词组index of,且网页的任何地方(URL、标题、文本等)包含词组backup files的 页面。同样要注意到intitle仅对词组index of起作用。





#### 正文

###### Intitle 与 Allintitle	：在页面标题中搜索

ps：只有在单词`intitle`后面的单词或词组才被认为是查询关键字，而`Allintitle`没有这一规则

**例如：**

`intitle:"index of" "backup files"`

`Allintitle:"index of" "backup files"`	每个链接的文档标题中都找到了两个词，更精确

>慎重使用allintitle操作符。在和其他高级操作符一起使用时，它就显得非常笨拙，而且会
>打乱整个查询，使得无法得到结果。也许有些极端，但是即便在一次查询中使用一-串intitle也
>好过使用拙劣的allintitle.





###### intext  与 Allintext	：在网页内容里查找字符串

由于这个操作符是以单词all开头的，所以在它之后的每个搜索关键字都将作为查询语句的一部分。

> 基于这个原因，alintext操作符不能和其他的高级符混合使用。



###### Inurl 与 Allinurl		：在URL中查找文本



###### Site 		：把搜索精确到特定的站点

注意：Google是`从左到右`读取服务器名的





###### Filetype		：搜索指定类型的文件

`filetype:pdf`

**常见的文件拓展名：**

```bash
pdf
ps
wk1, wk2, wk3, wk5, wki, wks, wku
Lwp
Mw
Xls
Ppt
Doc
wks, wps, wdb
Wri
Rtf
Swf
ans, txt
html
htm
php
asp
cfm
aspx
jsp
CGI
DO
PL
XML
DOC
PHTML
PHP3
TXT
EXE
XLS
PPT

```



> 提示：ext操作符可以用来替代filetype. filetye:xls查 询与ext:xIs等价。





###### Link		：搜索与当前网页存在链接的网页

`link:www.baidu.com`





###### Inanchor		：在链接文本中查找文本

可以和link操作符相比较使用，因为都能用来搜索链接。但是inachor操 作符搜索的是一个链接的文本表示，而不是实际的URL 。

> inanchor 操作符搜索的是锚点，或者说是链接上显示的文本





###### Cache		：显示网页的缓存版本



###### Numrange		：搜索数字

numrang操作符需要`两个参数`，一个是最小数，一个是最大数，以破折号分隔。

例如，查找12345：`numrange:12344-12346`或简写`12344..12346`





###### Daterange	：查找在某个特定日期范围内发布的网页

这个操作符的参数通常也必须表示为`一个范围`，即用破折号分开的两个日期。如果你只想查找某个特定日期索引的网页，必须提供同样的日期两次，并以破折号分开。





###### Info		：显示Google的摘要信息

该操作符的参数必须是: `一个有效的URL或者网站名`。





###### Related	：显示相关站点



###### Author 	：搜索Groups中新闻组帖子的作者



###### Group	  ：搜索Group标题

可以用来在Google Groups帖子的标题中搜索含有关键字的帖子。

group操作符并不能很好地和其他操作符混合使用。



###### Insubject	：搜索Google Group主题行

insubject操作符实际上和intitle搜索是一样的，它们返回同样的结果。



###### as_Msgid		：通过消息ID来查找Group帖子





###### Stocks		：搜索股票信息

这个操作符的参数必须是一个有效的股票简称

> stocks操作符不能和其他的操作符 及搜索关键字同时使用。





###### Define	：显示某个术语的定义



> define操作符不能和其他操作符或搜索关键字混合使用。



###### Phonebook		：搜索电话列表

phonebook操作符可以用来搜索商业和住宅电话列表。

例如：`phonebook: john ny`是johb在



###### view		：以什么形式展示

在Web查询的结尾放上`view:map`或者y`view:timeline`，以便在地图视图或者一个酷酷的时间轴视图中查看结果。



###### 其他操作符

。。。。。。



#### 操作符的混合使用

<img src="https://image.geoer.cn/googlehack.png"> </img>











# 基础

#### 使用缓存进行匿名浏览

可以实现`匿名`浏览，如果配合代理服务器效果更好：通过语法查找`inurl:"nph-prxy.cgi "start browsing"`等

选项`cached text only`其实是添加了换一个`&strip=1`参数，强制只显示缓存文本，禁止任何外部引用



>Google的缓存版本页面会对搜索关键字进行高亮显示。但是你可能还不知道可以使用Google的
>高亮工具在缓存网页中高亮显示某些并不包含在原始搜索中的关键字。





#### 目录列表

查找目录列表：`intitle:"index.of" "parent directory"` 其中`.`是通配符

查找特定目录：`intitle:index.of.admin`或`intitle:index.of inurl:admin`

查找特定的文件：`filetype:log inurl:ws_ftp.log`，使用filetype和inurl来查找

服务器版本：`intitle:index.of server at`

服务器版本：`intitle:index.of Apache/1.3.24 Server at` 等（其他服务器...）

搜索目标是否有列目录

```bash
site:"xpshuai.cn" intext:index of/ | ../ | Parent Directory
```



#### 遍历

目录遍历：`intitle:index.of inurl:"/admin/"`和`intitle:index.of inurl:"/var/www/"`等等

递增置换（“取一个数并用下一个更大的或者更小的数来替代这个数”的形象的说法）

拓展遍历（用于查找文件名相同，但是拓展名不同的文件如备份文件-->一旦你找到了HTM文件，就可以应用置换技术来查找具有相同的文件名，但扩展名不同的文件）









# 文档加工与数据库挖掘

#### 配置文件

`inurl:conf OR inurl:config OR inurl:cfg`

**Google黑客在搜索配置文件时常考虑的几点：**
●使用可用的配置文件中特有的单词或词组来创建一个强大的基本搜索。
●过滤掉sample. example、 test. howto和tutorial单 词以剔除明显的示例文件。升5 6 sm
●用-cvs过滤掉CVS存储器，它们通常存放了默认的配置文件。
●如果是搜索UNIX程序的配置文件，那么就要过滤掉manpage或者Manual（`-manpage`）
●搜索示例配置文件中最常改变的域，并且对该域执行缩简搜索，以剔除可能的“垃圾”或者样例文件。

**常见的配置文件搜索示例：**

<img src="https://image.geoer.cn/google_conf1.jpg"></img>

<img src="https://image.geoer.cn/google_conf2.jpg"></img>





#### 日志文件

例如：`filetype:log username putty`



#### Office文档

**搜索可能敏感的Office文档的查询样例：**

<img src="https://image.geoer.cn/google_office.jpg"></img>





#### 数据库挖掘

###### 登录入口

不管登录入口的安全强弱程度如何，它的存在都暴露了目标可能使用的软件和硬件的类型。简单来说，登录入口非常适合于顺藤摸瓜。



###### 帮助文件



###### 错误消息

<img src="https://image.geoer.cn/google_error1.jpg"> </img>

<img src="https://image.geoer.cn/google_error1.jpg"> </img>

<img src="https://image.geoer.cn/google_error3.jpg"> </img>

<img src="https://image.geoer.cn/google_error4.jpg"> </img>





###### 数据库转储

<img src="https://image.geoer.cn/google_zuanchu.jpg"></img>



###### 实际的数据库文件

如：`filetype"mdb inurl:user.mdb`





# 其他信息搜索

#### 确定E-mail地址

1.查找邮件服务器：`host -t mx qq.com` 、 Windows上：`nslookup -Cqtype=mx qq.com`

2.挑选出一个服务器，并且远程登录到端口25：`telnet mx1.qq.com 25 `

3.伪装简单邮件传送协议(SMTP)

4.正面测试

5.反面测试

6.完成，可以确定某个邮箱存在



#### 确定电话号码

通过Google的数字范围(numrange)操作符在一个特定的范围内查找包含的电话号码

先指定起始数字，接着键入`..`，再键入终止数字。



#### 确定人





#### 特殊的操作符

`filetype:ppt site:www.****.gov`







# 搜索exp与查找目标

#### 搜索公开的exp站点

如：`filetype:c exploit`

然后分离url：`grep cached exp |awk-F "-" '{print $1}'| sort -u`





#### 通过常见代码字符串搜索exp



#### 利用Google代码搜索查找代码



#### 搜索恶意软件和可执行文件



#### 利用源代码搜索目标





# 简单有效的安全性搜索



```bash


error | warning
login | logon
username |userid |employee.ID | "your username is"
password | "your password is" site:edu.cn

# -ext 必须和site一起用
site:edu.cn	-ext:html	# 排除html的文件类型

# 临时文件、备份目录
inurl:tmp | inurl:temp | inurl:backup | inurl:bak
```







# 跟踪搜索Web服务器、登录入口和网络硬件

#### 定位并剖析Web服务器

1.目录列表

2.Web服务器软件的错误消息

3.默认文档

4.示例程序







# 用户名、口令和其他秘密信息

搜索用户名

搜索口令

信用卡账号

社保号码

个人财务数据









































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/google-hacking/  

