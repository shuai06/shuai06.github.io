# upload-labs通关记录




本文主要记录一下`upload-labs`最新21关卡的过程，作为自己的一个笔记。

> 靶场地址：https://github.com/c0ny1/upload-labs/

  

**ps：**

\- 为了快速验证效果，此处上传的webshell代码为`<php? phpinfo(); ?>`

\- 解题思路五花八门，我这里想起了啥就用啥。

\- 因为比较菜，所以做题直接看代码了就(谁料有的题目看了代码都看不懂)，哈哈哈。

<img src="http://image.geoer.cn/20200218195326654_1642698147.png" />

## 相关函数

```
trim()  去除空格


strrchr()查找字符串在另一个字符串中`最后一次`出现的位置，并返回从该位置到字符串结尾的所有字符。

strrchr("被查找的字符串", "出现的字符串")

strstr() - 查找字符串的首次出现

strrpos() - 计算指定字符串在目标字符串中最后一次出现的位置

str_ireplace('要替换的对象','替换为','字符串')


deldot($file_name);//删除文件名末尾的点


::$DATA ---> win上，会把`::$DATA`之后的数据当成文件流处理，不会检测后缀名，而且保持`::$DATA`之前的文件名（目的是不检查后缀名）


pathinfo(path,options)返回一个关联数组包含有 path 的信息。

包括以下的数组元素：

[dirname]

[basename]

[extension]


第二个参数可能的值：

PATHINFO_DIRNAME - 只返回 dirname

PATHINFO_BASENAME - 只返回 basename

PATHINFO_EXTENSION - 只返回 extension

注释：如果不是要求取得所有单元，则 pathinfo() 函数返回字符串。


intval() 函数用于获取变量的整数值。


@uppack




`rename()` 函数 ：重命名文件或目录。如果成功，该函数返回 TRUE。如果失败，则返回 FALSE。rename(oldname,newname,context)


```









##  Pass-01

> js验证

点击上传文件之后，burp没有拦截到数据包，并且验证反应速度快

F12查看代码后确认



######  绕过

**F1: **

禁用js，直接上传php

<img src="http://image.geoer.cn/20200217215459803_1606358356.png" />

成功：

<img src="http://image.geoer.cn/20200217215605878_1982942629.png" />



**F2：**

在本地创建html，action为服务器请求地址，（去掉相应js部分）

**F3:**

先把shell改为允许的类型,再bup中再改回来



效果都是一样的





## Pass-02

> 服务端验证 -- 文件MIME类型



只检测文件自带的类型

<img src="http://image.geoer.cn/20200217215941532_643347454.png" />





##### 绕过：

burp修改数据包中修改MIME类型（Content-Type）为允许的类型

<img src="http://image.geoer.cn/20200217220124560_567357512.png" />







# Pass03-Pass10:      黑名单校验

## Pass-03          

> 服务端验证(黑名单方式) -- 多种格式后缀解析



对上传的文件先过滤，后判断文件后缀，再随机命名

<img src="http://image.geoer.cn/20200217220446232_1542284348.png" />



**比如：**         a.php

先删掉文件名前后的空格

删除末尾的点：  a.php

strrchr之后：      .php

转换为小写：     .php

去除字符串::$DATA

再收尾去空格

最后加上日期重命名





###### 绕过

> 前提:  `httpd-conf`文件中`AddType application/x-httpd-php` 没有被注释，而且有规定类型的文件

对 phtml，php3，php4，php5，pht，htaccess没有进行过滤，那么利用这两种方法进行绕过。



**补充:**

asp:   asa, cer, cdx 等

jsp:   jspx, jspf

aspx:   ashx,asmx,ascx



**准备工作：**

修改httpd-conf配置文件,这里可以把php3等想要的文件后缀加上，再把注释符#去掉

此类的正则表达式，文件名字满足即可在apache当做php解析。比如以下格式的文件：

php3，php4, php5, phtml，phps

修改为： `AddType application/x-httpd-php .php .phtml .php3 .php4 .php5 .phps `

**F1:  修改后缀名**

burp拦截数据包，修改filename为`php3`,上传成功

右键图片查看地址, 从这里可以得知上传的文件被重命名了,  打开图片地址，OK



**F2:  上传图片马**

上传图片马，然后利用BP拦截，将后缀名 `改为phtml,php3,php4,php5,pht`

<img src="http://image.geoer.cn/20200218144512768_1843220897.png" />



  

​    

​    

## Pass-04

> 服务端（php5之类的也进行了过滤）



<img src="http://image.geoer.cn/20200217222608134_260661208.png" />



但是没有过滤`.htaccess`，可以考虑重写文件解析规则绕过



info.php.jpg.



###### 绕过

> 前提: apache搭建，  没有对上传文件重命名



**.htaccess文件**

1.先上传一个`.htaccess`文件，内容如下：

```xml
<FilesMatch "jpg">

SetHandler application/x-httpd-php

</FilesMatch>
```



2.然后再上传一个info.php, burp拦截数据包修改filename为`info.jpg`   / 或者直接上图片马， url访问图片路径，正常解析





## Pass-05

<img src="http://image.geoer.cn/20200217223711503_294591319.png" />

连.htaccess都放黑名单了



**缺陷：**

`strrchr函数`：  搜索参数在字符串中的位置，并返回从该位置到字符串结尾的所有字符

如果有多个，匹配的是`最后一个`（这里并没有进行递归检测，前面的就会漏掉）

如果是`a.php. jpg. `是不能正常解析的，但是`a.php.  .`就行



###### 绕过：

多写一个空格和点：上传文件 `info.php`， burp修改file为`info.php. .`

**比如：**

```
a.php.  .
```

去空：`a.php.  .`

删除末尾点：`a.php.  `        最后一个点被deldot删除

返回最后一个点的内容`. `       第二个strrchr匹配了第一个点和后面的空格

前后去空：后缀为`.`, 不在黑名单范围，成功绕过

最后上传文件名为：`a.php.`, 在windows保存文件的之后的`.`会消失



如下图，win服务器保存的文件：

<img src="http://image.geoer.cn/20200217224221081_348766282.png" />



正常解析：

<img src="http://image.geoer.cn/20200217224244474_173158029.png" />





##  Pass-06

<img src="http://image.geoer.cn/20200218145809732_1096167348.png" />



没有进行后缀名大小写转换





###### 绕过

burp修改filename后缀名为`.phP`，绕过检测，上传OK

正常解析

<img src="http://image.geoer.cn/20200218150044751_461792168.png" />







## Pass-07

<img src="http://image.geoer.cn/20200218150316897_1558000982.png" />



最后没有将后缀首尾去空

可以利用win文件名字保存格式的特性: 比如最后一个为空格`info.php `, 

`.php `不在黑名单`.php`等中，成功绕过，

并且windows保存的时候会把最后的空格去除





###### 绕过

burp修改filename为`info.php `，后面加一个空格，上传





## Pass-08

<img src="http://image.geoer.cn/20200218151055198_961499757.png" />



少了提前删除文件名末尾的点 的功能(deldot)；

`strrchr`过滤匹配的是最后一个点；

没有对上传的文件重命名





###### 绕过:

**F1： 后缀名加点绕过**

burp修改文件名，

```
a.php.
```

`a.php. `可以带空格

上传。



**F2：利用Windows解析漏洞（后缀修改为1.php:1.jpg）**

```
1.php:1.jpg
```

没成功： 上传的是`1.php`，但是文件内容是空的了



  

​    

​    

## Pass-09

> Windows文件流绕过



<img src="http://image.geoer.cn/20200218152924608_961861669.png" />



少了 `$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA`    --> 含义：把后缀中是 ::DATA替换为空  

`str_ireplace('要替换的对象','替换为','字符串')`函数



由上可知，文件`没有对后缀名进行去”::DATA”处理`。

php在window的时候如果文件名+"::DATA”处理。php在window的时候如果文件名+"::DATA”处理。php在window的时候如果文件名+"::DATA"会把::DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::DATA"之前的文件名 他的目的就是不检查后缀名。



> ps:只能是Windows系统，并且只能时php文件





###### 绕过

```
info.php::$DATA
```

1.如图 ，在BP中修改图片马的 jpg 的后缀名，将其改为 php ，并且在后面加上 ::$DATA ;

<img src="http://image.geoer.cn/20200218161807873_1645216989.png" />

2.在网页上检查图片马的执行情况时，将图片马的后缀名中 ::$DATA ，删去，便可浏览。

如图，这个是没有去掉后缀时，会发生报错。（因为我们实际上传的是PHP文件，即upload文件夹中只有php文件，而不存在带有后缀 ::$DATA的文件）

<img src="http://image.geoer.cn/20200218162201913_251841335.png" />





# Pass 10 - 11 （将filename拼接路径会带来极大风险）

## Pass-10

<img src="http://image.geoer.cn/20200218163307924_285954321.png" />





###### 绕过

**构造后缀名点空绕过**

`strrchr`函数 搜索参数在字符串中的位置，并返回从该位置到字符串结尾的所有字符

如果有多个，匹配的是最后一个（前面的就会漏掉

由上可知，该关进行的是现在文件末尾先去点，再去空便没有再做任何处理，因此我们可以利用这个漏洞，构造后缀名为php+点+空+点

burp修改文件名`info.php. .`

<img src="http://image.geoer.cn/20200218163126238_1032488863.png" />







## Pass-11

> 双写绕过



没有strrchr

<img src="http://image.geoer.cn/20200218163843491_1975433720.png" />





str_ireplace函数：

`str_ireplace('要替换的对象','替换为','字符串')`,   他把我们的php后缀替换为空

**函数缺陷：**

只是执行一次/或者没采用递归过滤





###### 绕过

扰乱，删除里面后，外面还能组合成功后缀

burp修改filename=`info.pphphp`

info.pphphp --- > 'info.php'



<img src="http://image.geoer.cn/20200218163953790_921378631.png" />







# Pass 12 - 13 将filename当做变量加入最终路径会带来极大风险

## Pass - 12

> GET请求，URL网页上00截断

> 前提条件：php5.34以下， magic_quotes_gpc为off状态

**魔术引号 magic_quotes_gpc 作用**： 对（反斜杠？，单引号，双引号，none）产生影响, 自动加上/



**白名单绕过**

)<img src="http://image.geoer.cn/20200218165050537_741059186.png" />

save_path可控， `$_GET['save_path']` 通过get获取，可以把get中url参数带上00截断



%00 截断（类似注释符号，后面的不再执行）

> Note： %00 是URL编码后的结果， 选中，解码转换（url decode）为小方块，才是真正的%00, Hex下是00



数据包中 `save_path=../upload/shell.php%00` （注意 url_decode）后面的就没用了，filename="shell.jpg"



由源码可知，图片的save_path通过get传递，然后和后缀ima_path直接拼接而成的，所以我们可以直接使用 %00 截断，将后缀名 .jpg 截断，从而以 php 运行





###### 绕过

URI中00截断



**方法一：拦包00截断**

由于是在get的url中，`%00`就不进行decode了，

<img src="http://image.geoer.cn/20200218170335768_2032329037.png" />



上传成功：

查看图片路径：

<img src="http://image.geoer.cn/20200218170644694_1142415094.png" />

查看服务器保存文件：

<img src="http://image.geoer.cn/20200218170437679_903547409.png" />



访问的时候，去掉后面的jpg，只访问php，OK：

<img src="http://image.geoer.cn/20200218170513646_522012281.png" />







**方法二：前端直接修改为 "php+00截断"格式**

如下可得，直接将参数save_path修改为php+00截断的格式，因此它会将之后上传的所有文件以PHP的格式进行解析。

ps:图片是盗取的

<img src="http://image.geoer.cn/20200218170957259_1424254637.png" />



burp只修改下面的filename为`x.jpg`即可







## Pass - 13

> POST请求，00截断



**白名单绕过**

<img src="http://image.geoer.cn/20200218165118835_726563878.png" />





**F1:**

同12关，只是修改的地方不同

**1.**burp拦截。在post后面加上`info.php%00` (%00是url decode之后的)，再发送即可

<img src="http://image.geoer.cn/20200218172018042_1656668337.png" />



**2.**burp拦截，然后在上面的upload 的后面加上 `123.php+`（这里的123可以为任何名字，他只是一个代号。并且 php后面的 `+ `好也是一个标记，他的二进制代码为 `2b`）

由上可知，将二进制中的 + 改为 00 截断，即将 + 的二进制码 2b 改为 `00` .

<img src="http://image.geoer.cn/20200218171415601_503298826.png" />



同样，访问的时候访问php文件：

<img src="http://image.geoer.cn/20200218171807564_1778234073.png" />







**F2 意外发现：文件覆盖**

<img src="http://image.geoer.cn/20200218172227317_2113414909.png" />

由于名字可以不相同，即我们可以任意定义uploads旁边的 PHP 文件名，如果将文件名改为uploads中本身存在的文件，那么新的文件将会覆盖原来的文件。

<img src="http://image.geoer.cn/20200218172321670_308115506.png" />







## Pass - 20 （也是类似的，便于比较就放这里了）



<img src="http://image.geoer.cn/20200218172852998_939565232.png" />

<img src="http://image.geoer.cn/20200218172525227_655899960.png" />

这里进行过滤的是他的那个文件名，而不是我们上传文件最开始的文件名，所以要在他的文件名上做手脚



`$file_ext = pathinfo($file_name,PATHINFO_EXTENSION)`   返回拓展后缀



这里的文件名`filename`是可控的：  `$_POST['save_name']`



###### 绕过

 我们可以在save_path中加入php+00截断，由于是POST发送方式，因此我们需进行00截断。`111.php%00upload-19.jpg` （注意url decode）

<img src="http://image.geoer.cn/20200218174147577_857047164.png" />



上传OK：

<img src="http://image.geoer.cn/20200218174246606_1092693624.png" />



去掉00后面内容，访问php，正常解析

<img src="http://image.geoer.cn/20200218174323338_682125752.png" />







# Pass14 - 17 （文件包含漏洞）

## Pass - 14

> 获取文件头进行了验证



**图片马制作：**

win下：copy a.jpg /b + b.php /a shell.jpg

Linux下：cat a.jpg b.php > shell.jpg



文件包含漏洞（xx.php?file=uploads/x.jpg） 文件不限格式后缀

**意义**：如果存在包含漏洞，需要上传一张图片(图片马)，就可以getshell



<img src="http://image.geoer.cn/20200218175651617_999905544.png" />



`$strInfo = @unpack("C2chars", $bin);`// C为无符号整数，网上搜到的都是c，为有符号整数，这样会产生负数判断不正常



`$bin = fread($file, 2);` //只读2字节进行验证

表示只是对文件头做了检测

 `$strInfo = @unpack("C2chars", $bin);` 标示前两个字符按照，c格式，数组索引chars1,chars2                @unpack

 `$typeCode = intval($strInfo['chars1'].$strInfo['chars2']);`



**文件头：** 不同后缀类型文件头部代码中有GIF89a,NG,PK,7Z， 部分验证会验证文件头

> 常见文件头：https://blog.csdn.net/xiangshangbashaonian/article/details/80156865



**突破：** 伪造可行的文件头



###### 绕过

上传图片马，上传。打开测试页面，引用刚才的图片马







## Pass - 15    getimagesize

<img src="http://image.geoer.cn/20200218185841047_937742626.png" />



函数拓展：`getimagesize（）`   判断是否为一个完整的图像

索引2`$info[2]` 是图像的类型

<img src="http://image.geoer.cn/20200218185758469_1439877358.png" />



image_type_to_extension 获取的是MIME类型

```ph
$info = getimagesize($filename);

$ext = image_type_to_extension($info[2]);  # 获取后缀jpeg  ,   $info[2] 是文件类型


if(stripos($types,$ext)>=0)  # 是否在白名单中
```







###### 绕过

上传图片马，

打开测试页面，直接包含`include`file='文件.jpg', 就是当成php解析









## Pass - 16   php_exif模块

<img src="http://image.geoer.cn/20200218185919196_54375063.png" />



函数拓展：`exif_imagetype（）`

<img src="http://image.geoer.cn/20200218190317887_1240325255.png" />



php_exif模块来判断文件类型

版本：PHP>=4.3.0,  PHP5, PHP7



###### 绕过    

需要开启`php_exif`模块：

1.打开配置文件，php拓展，`php_exif`打开；

2.配置文件在`php.ini`,分号是注释，可以直接解除注释启用,并将此行移动到extension=php_exif.dll之前，使之首先加载*。

<img src="http://image.geoer.cn/20200218220405641_1769279545.png" />





上传图片马（注意MIME与后缀和文件头要一直，然后打开测试页面，包含gif，OK











## Pass - 17   （图片二次编码）



其实就是检测两次

**本质：** 检测 MIME和后缀是否对应



**原理：** 将一个正常显示的图片，上传到服务器。寻找图片被渲染后与原始图片部分对比仍然相同的数据块部分，将Webshell代码插在该部分，然后上传。

具体实现需要`自己编写Python程序`，人工尝试基本是不可能构造出能绕过渲染函数的图片webshell的。

```
//使用上传的图片生成新的图片

 $im = imagecreatefromjpeg($target_path);
```



###### 绕过  

好了，还是太菜，还是看大佬的解决方案学习学习把吧:[点击这里](https://xz.aliyun.com/t/2657)

**1.GIF**

关于绕过gif的二次渲染,我们只需要找到渲染前后没有变化的位置,然后将php代码写进去,就可以成功上传带有php代码的图片了.



**2.PNG**

png的二次渲染的绕过并不能像gif那样简单.



**3.JPG**





> ps: 这一关是二次渲染的，还是多学习学习吧









# 18 -19 条件竞争 （多线程发包）

## Pass - 18



<img src="http://image.geoer.cn/20200218222155590_1020797820.png" />



`unlink`函数：   数删除文件。若成功，则返回 true，失败则返回 false



**执行步骤：**

1.取出上传文件的后缀名

2.临时文件移动（`move_uploaded_file`）到一个路径下面，

3.如果后缀名在白名单，进行最后移动，保存

4.如果后缀名不在白名单，`unlink`删除临时文件





###### 绕过

> 趁他不注意（多线程），在临时文件里面写一个创建后门文件的功能，这样虽然临时文件不满足被删除，但是自己自动创建的文件已经存在服务器目录了



**F1：**

其中，诱饵上传文件代码

```php
<?php

$myfile = fopen("shell.php", "w") or die("Unable to open file!");

$txt = "<?php  phpinfo();   ?>\n";

fwrite($myfile, $txt);

fclose($myfile);

?>
```

然后，如果已知新建文件的路径，可以直接访问

​            如果不知道，可以抓取数据包查看



**方法：**

1.利用burp的Intruder使用多线程发包(如下图)，

2.然后不断在浏览器访问我们的webshell，会有一瞬间的访问成功。（即当线程足够的时候 ，将可能会跳过某个步骤，而直接访问到我们的 webshell，新文件也会创建成功 )

<img src="http://image.geoer.cn/20200218222719652_1510553817.png" />





**F2:**   Python

```python
\#!/usr/bin/env python3

\# coding:utf-8



import hackhttp

from multiprocessing.dummy import Pool as ThreadPool





def upload(lists):

​    hh = hackhttp.hackhttp()

​    raw = """POST /upload-labs/Pass-17/index.php HTTP/1.1

Host: 127.0.0.1

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:49.0) Gecko/20100101 Firefox/49.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3

Accept-Encoding: gzip, deflate

Referer: http://127.0.0.1/upload-labs/Pass-18/index.php

Cookie: pass=18

Connection: close

Upgrade-Insecure-Requests: 1

Content-Type: multipart/form-data; boundary=---------------------------6696274297634

Content-Length: 341



-----------------------------6696274297634

Content-Disposition: form-data; name="upload_file"; filename="18.php"

Content-Type: application/octet-stream



<?php

$myfile = fopen("shell.php", "w") or die("Unable to open file!");

$txt = "<?php  phpinfo();   ?>\n";

fwrite($myfile, $txt);

fclose($myfile);

?>

-----------------------------6696274297634

Content-Disposition: form-data; name="submit"



上传

-----------------------------6696274297634--

"""

​    code, head, html, redirect, log = hh.http('http://127.0.0.1/upload-labs/Pass-17/index.php', raw=raw)

​    print(str(code) + "\r")


pool = ThreadPool(10)

pool.map(upload, range(10000))

pool.close()

pool.join()
```

只是把burp换成了python，其他操作方法是一样的



###### 修复问题

应该先判断文件类型，再移动









## Pass - 19





```php
if( $this->cls_rename_file == 1 ){  //是否用$im_path成功给$upload_file重命名
```





和第18关有点类似，都对关键进行了重命名，由于我们是利用burp不断的发送相同的包，那么一旦有两个包同时传到这边是，那么系统对一个重命名时，另一个刚好“逃脱”了，那么上传成功。





###### 绕过      

在这关是一个`白名单`，`利用了appche的一个解析漏洞`，它会将 “`.php.7z` 当作` .php`来解析，而刚好 “ .php.7z ”又属于白名单，

所以上传一个 “ .php.7z ” 的文件，然后 再利用条件竞争漏洞进行多线程不断的发送 “ .php.7z”的文件，

由于版本问题，这次的文件没有上传到upload的文件夹下面，而是上传到 WWW 的文件夹下面，但是不影响，并且这次的文件上传后就`不会被删掉`，就会保存在 该文件夹下面，这样的话我们将可以`稳定的访问到`。



可以看到，burp进行多线程发包后，在浏览器一致访问上传文件的地址，一定会有那么一瞬间会访问到（这时，文件里新建webshell的代码执行了，新的后门建立成功了，我们就可以访问后门了）

<img src="http://image.geoer.cn/20200218224226603_822814111.png" />













# 00截断和move_uploaded_file（）函数

## Pass - 20      

`pathinfo(）函数`：以数组的形式返回关于文件路径的信息。

pathinfo(path,options)

`move_uploaded_file(A,B`)函数：此函数将会检查文件A是否是合法的上传文件，如果是，将把文件A移动到B的目录下；否则将会返回false，并且不执行任何操作。



###### 绕过

**方法一：**  %00截断 （前面已经写了）

`111.php%00upload-19.jpg`  其中，%00是url-decode之后的



**方法二：** 

利用`move_uploaded_file（）函数和黑名单结合`存在的漏洞：

move_uploaded_file（A,B）函数：此函数将会检查文件A是否是合法的上传文件，如果是，将把文件A移动到B的目录下；`否则将会返回false，并且不执行任何操作`。



由于这里利用了黑名单，则move_uploaded_file（）函数在判断时只会判断是否为黑名单，如果不是，那么他将会直接将文件保存在指定目录下。所以在这里我们利用 `/.`(斜杠点)和 ` 空格 `（空格）进行绕过。

（1） “/.”(斜杠点)

​    `upload-19.php/.`



（2）“ ”（空格）

​    `upload-19.php `

​    

（3）“ .”（空格）

 `upload-19.php.`

  `upload-19.php. `   点+空格

   等等...



**方法三：** 

Windows冒号截断

```
upload-19.php:.jpg
```

(我做的实验，php上传成功，但是内容清空了)







## Pass - 21     

> 参考: https://blog.csdn.net/kekefen01/article/details/90346413



通过传递数组绕过。属于逻辑漏洞，没有考虑数组第二个元素不存在的情况。



###### 绕过

使用多个数组

```
------WebKitFormBoundaryDTdqZdRoon2GpWOE

Content-Disposition: form-data; name="upload_file"; filename="info.php"

Content-Type: image/jpeg



<?php phpinfo();?>



------WebKitFormBoundaryDTdqZdRoon2GpWOE

Content-Disposition: form-data; name="save_name[0]"



info.php/

------WebKitFormBoundaryDTdqZdRoon2GpWOE

Content-Disposition: form-data; name="save_name[2]"



jpg

------WebKitFormBoundaryDTdqZdRoon2GpWOE

Content-Disposition: form-data; name="submit"



上传

------WebKitFormBoundaryDTdqZdRoon2GpWOE--
```

最终文件名为“info.php/.” 因为move_uploaded_file底层会调用tsrm_realpath函数导致，递归删除文件名最后的/.







> 参考文章：https://blog.csdn.net/weixin_43901038/article/details/94890631

> https://www.cnblogs.com/shellr00t/p/6426945.html

> https://thief.one/2016/09/22/%E4%B8%8A%E4%BC%A0%E6%9C%A8%E9%A9%AC%E5%A7%BF%E5%8A%BF%E6%B1%87%E6%80%BB-%E6%AC%A2%E8%BF%8E%E8%A1%A5%E5%85%85/

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/upload-labs%E9%80%9A%E5%85%B3%E8%AE%B0%E5%BD%95/  

