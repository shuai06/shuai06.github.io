# 文件上传漏洞


## 简介

上传漏洞顾名思义，就是攻击者上传了一个可执行文件如木马，病毒，恶意脚本，WebShell等到服务器执行，并最终获得网站控制权限的高危漏洞。

危害：能够直接获取到Web权限，后续操作还不好说么





## 上传技巧&绕过

### upload-lab

结合upload-labs关卡的文章

我前面也写了个通关的文章；

另一篇wp：https://github.com/LandGrey/upload-labs-writeup



![upload-labs](http://image.xpshuai.cn/img/image-20220118153454307.png)





### 编辑器类上传漏洞

**百度Ueditor**

```
controller.ashx?action=catchimage
```



**fckeditor**
查看版本

```
/fckeditor/editor/dialog/fck_about.html

/FCKeditor/_whatsnew.html
```

![image-20220118154051352](http://image.xpshuai.cn/img/image-20220118154051352.png)



**Kindeditor**
上传页面

```
kindeditorlasp/upload_json.asp
kindeditorlasp.net/upload_json.ashx
kindeditorljsp/upload_json.jsp
kindeditor/php/upload_json.php
```



![image-20220118154120671](http://image.xpshuai.cn/img/image-20220118154120671.png)



**ewebeditor**

... ...





### 中间件解析漏洞

**IIS6.0** -> windows server03
文件名：logo.jpg ===>有漏洞： logo.asp;.jpg
文件夹：image/logo.jpg =>有漏洞： image.asp/qq.jpg
几乎100%存在漏洞（官方未有补丁）



**IIS7.X** -> win7、win server2008
www.a.com/logo.png
www.a.com/logo.png/.php
将前面的logo.php进行php格式解析
验证： 访问网站上一个图片，后面加上.php/ , 如果出现乱码，则存在



**apache 低版本**
**逐级解析**
www.a.com/index.php
www.a.com/index.php.xxx.qqq

1. 向上(前面)解析，如果不识别就向上，直到识别为止(未知拓展名解析漏洞)
2. AddHandler导致的的解解析析漏洞
   如果运维人员给.php后缀增加了处理器： AddHandler application/x-httpd-php .php`那么，在有多个后缀的情况下，只要一个文件名中含有.php后缀，即被识别成PHP文件，没必要是最后一个后缀。 利用这个特性，将会造成一个可以绕过上传白名单的解析漏洞。 
3. Apache HTTPD 换行解析漏洞(CVE-2017-1571)
   2.4.0~2.4.29版本
   此漏洞形成的根本原因，在于正则表达式中不仅匹配字符串结尾位置，也可以匹配\n 或 \r
   在解析PHP时，1.php\x0A将被按照PHP后缀进行解析，导致绕过一些服务器的安全策略。
   <FilesMatch .php$>
   SetHandler application/x-httpd-php

**Nginx 低版本**
和IIS7.X 版本一样

- 配置文件错误导致的解析漏洞
  对于任意文件名，在后面添加/xxx.php（xxx为任意字符）后,即可将文件作为php解析。
  例：info.jpg后面加上/xxx.php，会将info.jpg 以php解析。
  **ps: ** 该漏洞是Nginx配置所导致，与Nginx版本无关





## Bypass

### 命名下的变异

1.可解析的脚本后缀，也就是该语言有多个可解析的后缀

![脚本后缀](http://image.xpshuai.cn/img/image-20220118154504496.png)

2.大小写混合，如果系统过滤不严，可能大小写可以绕过

3.中间件安全，部分中间件存在文件解析漏洞

![文件解析漏洞](http://image.xpshuai.cn/img/image-20220118154543977.png)

服务器特性:
1.会将Request中的不能编码部分的%去掉
2.Request中如果有unicode部分会将其进行解码

```bash
# IIS
lIS6.0两个解析缺陷:目录名包含.asp、.asa、.cer的话，则该目录下的所有文件都将按照asp解析，例如:

/abc.asp/1.jpg会当做/abc.asp进行解析。
/abc.php;1.jpg会当做/abc.php进行解析。
IIS/7.5（两次%00截断： xxx.asp%00.asp%00.jpg + 二次上传 ？）

# Apache1.x 2.X解析漏洞
Apache在以上版本中，解析文件名的方式是从后向前识别扩展名，直到遇见Anache可识别的扩展名为止。

# Nginx
以下Nginx容器的版本下，上传一个在waf白名单之内扩展名的文件shell.jiog，然后以shell.jpg.php进行请求

Nginx 0.5.*
Nginx 0.6.*
Nginx 0.7<= 0.7.65
Nginx 0.8<= 0.8.37
以下Nginx容器的版本下，上传一个在waf白名单之内扩展名的文件shell.jpg，然后以
·shell.jpg%20.php·进行请求。

# Nginx 0.8.41 - 1.5.6:
# PHP CGI解析漏洞
IS7.0/7.5和Nginx<0.8.3 以上的容器版本中默认php配置文件cgi.fix_pathinfo=1时，上传一个存在于白名单的扩展名文件shell.jpg，在请求时以shell.jpg/shell.php请求，会将shell.jpg以php来解析。
```



4.系统特性

![系统特性](http://image.xpshuai.cn/img/image-20220118154748278.png)

```bash
test.asp.
test.asp (空格)
test.php:1.jpg
test.php::$DATA
test.php_

```

![系统特性](http://image.xpshuai.cn/img/image-20220118154818983.png)

5.语言漏洞，流行的三种脚本语言基本都存在00截断漏洞。

JSP文件名绕过:

![image-20220118154934765](http://image.xpshuai.cn/img/image-20220118154934765.png)



6.双后缀，与系统和中间件无关，偶尔存在于代码逻辑之中

```bash
可解析的后缀+大小写混合
可解析的后缀+大小写混合+中间件漏洞
.htaccess + 大小写混合
可解析的后缀+大小写混合+系统特性
可解析的后缀+大小写混合+语言漏洞
可解析的后缀+大小写混合+双后缀

```



> 上面的可以绕过脚本本身的限制的，但是不能过waf



### 文件名绕过

1.文件名加回车
2.shell.php(%30-%99).jpg绕过
3.如果有改名功能，可先上传正常文件，再改名.
4.%00
5.00(hex)
6.长文件名(windows 258byte / linux 4096bydte)，可使用非字母数字，比如中文等最大程度的拉长。
7.重命名





### 协议解析不一致,绕过waf(注入跨站也可尝试)

1.垃圾数据

![垃圾数据](http://image.xpshuai.cn/img/image-20220118155153315.png)



### 未解析所有文件

multipart协议中，一个POST请求可以同时上传多个文件。**许多WAF只检查第一个上传文件，没有检查上传的所有文件**，**而实际后端容器会解析所有上传的文件名**，攻击者只需把paylaod放在后面的文件PART，即可绕过。



### 参数下的变异

- Content-Disposition 一般可以进行任意修改甚至删除
- Content-Type 视情况而定, 需要考虑网站上传验证是否进行处理
- filename 可以修改

一些payload:

```bash
# 文件名覆盖
filename="1.txt";filename="php.php"

# 遗漏文件名（当WAF遇到name="myfile";;时，认为没有解析到filename。而后端容器继续解析到的文件名是t3.jsp，导致WAF被绕过）
name="myfile";;filename="php.php"


filename="x.php%00.jpg"
filename=php.php;        #去掉双引号和添加一个分号，匹配的是`php.php;`和分号
filename="x.php
filename="'x.php           #后面的单引号没闭合
filename="'x.php"
filename= ;filename="php.php"        #多写几个变量，buwng不往后匹配第一个留空，没有值就

# 文件名回车
filename="x.
p
h
p
"                    #用多个换行(%0a 的url decode)来避开


```

不规则Content-Disposition文件名覆盖:

![](http://image.xpshuai.cn/img/image-20220118155331607.png)



多个content-Disnosition文件名覆盖(Win2k8 +IIS7.0+ PHP):

![](http://image.xpshuai.cn/img/image-20220118155402339.png)



更换filename位置(iis 6):

![](http://image.xpshuai.cn/img/image-20220118155419185.png)



删除content-type字段



删除content-disposition空格:

![](http://image.xpshuai.cn/img/image-20220118155441338.png)





修改 Content-Disposition字段值的大小写:

![](http://image.xpshuai.cn/img/image-20220118155500736.png)



boundarv空格(Win2k3 +lIS6.0+ ASP):

![](http://image.xpshuai.cn/img/image-20220118155519326.png)



boundary边界不一致(Win2k3 + IS6.0+ ASP):



php+apache畸形的boundary:
php在解析multipart data的时候有自己的特性，对于boundary的识别，只取了逗号前面的内容，例如我们设置的 boundarv为--aaaa,123456 , php解析的时候只识别了--aaaa ,后面的内容均没有识别。然而其他的如WAF在做解析的时候，有可能获取的是整个字符串，此时可能就会出现BYPASS

![](http://image.xpshuai.cn/img/image-20220118155600100.png)



`===`绕过：

![](http://image.xpshuai.cn/img/image-20220118155804211.png)



去除"""绕过：

![](http://image.xpshuai.cn/img/image-20220118155811407.png)



少“绕过：

![](http://image.xpshuai.cn/img/image-20220118155825148.png)





文件名.php+回车：这样引号就在另一行。同时上传内容的一句话前面加个中文字符







Content-Disposition:

![](http://image.xpshuai.cn/img/image-20220118155621710.png)





Content-type:

![](http://image.xpshuai.cn/img/image-20220118155651220.png)



### Fuzz结合 Bypass





## Java代码审计

### 文件上传方式

Java中文件上传方式很多，常用的有三种：

#### 1.通过文件流的方式上传





#### 2.通过ServletFileUpload方式上传





#### 3.通过MultipartFile方式上传







### 关键字

```bash
org.apache.commons.fileupload
java.io.File
MultipartFile
RequestMethod
MultipartHttpServletRequest
CommonsMutipartresover

File
lastIndexOf
indexOf
FileUpload
getRealPath
getServletPath
getPathInfo
getContentType
equalsIgnoreCase
FiltUtils
MultipartRequestEntity
UploadHandleServlet
FileLoadServlet
FileOutpuStream
getInputStream
DiskFileIntemFactory
...
```













## 漏洞防御

- 文件后缀名截取校验时候，忽略大小写
- 严格检测文件类型，推荐白名单
- Java小于jdk7u40时可能存在截断漏洞，注意版本影响
- 限制文件上传的大小和频率
- 对上传的文件重命名、自定义后缀等



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%E6%BC%8F%E6%B4%9E/  

