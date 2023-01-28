# SSRF 302跳转 Bypass


> 今天面试问到了302SSRF，之前没了解过，发现CTFHub技能树里面有，就来记录一下学习

## 简介

**302用来做临时跳转**

302 表示临时性重定向，访问一个url时，被重定向到另一个url上。

302常用于页面跳转，比如未登陆的用户访问用户中心，则重定向到登录页面，访问404页面会重新定向到首页。

**301适合永久重定向**
301比较常用的场景是使用域名跳转。

比如，我们访问 http://www.baidu.com 会跳转到 https://www.baidu.com，发送请求之后，就会返回301状态码，然后返回一个location，提示新的地址，浏览器就会拿着这个新的地址去访问。

**注意：** 301请求是可以缓存的， 即通过看status code，可以发现后面写着from cache。
　或者你把你的网页的名称从php修改为了html，这个过程中，也会发生永久重定向。



**301重定向和302重定向的区别：**
302重定向只是暂时的重定向，搜索引擎会抓取新的内容而保留旧的地址，因为服务器返回302，所以，搜索搜索引擎认为新的网址是暂时的。

而301重定向是永久的重定向，搜索引擎在抓取新的内容的同时也将旧的网址替换为了重定向之后的网址。




## 题目

> 题目：SSRF中有个很重要的一点是请求可能会跟随302跳转，尝试利用这个来绕过对IP的检测访问到位于127.0.0.1的flag.php吧



打开题目，发现地址为：`http://challenge-d3b433f06a62d54f.sandbox.ctfhub.com:10800/?url=`

发现后面有过get的url参数，尝试访问：`127.0.0.1/flag.php`，提示“被拦截，不允许内部ip访问”

![被拦截](http://image.xpshuai.cn/img/image-20220112162357324.png)

那我使用file协议获取flag其源码：`?url=file:///var/www/html/flag.php`

![flag源码](http://image.xpshuai.cn/img/image-20220112164113528.png)

可见通过`REMOTE_ADDR`请求头限制本地IP请求，但是源码中并没有之前的“hacker! Ban Intranet IP”，所以查看`index.php`页面的源码：`?url=file:///var/www/html/index.php`

![index.php源码](http://image.xpshuai.cn/img/image-20220112164359513.png)



可以看到存在黑名单，限制了`127`、`172`、`10`、`192`网段，这里可以通过`localhost`的方式绕过: `?url=localhost/flag.php`
也可以得到flag：

![localhost获取flag](http://image.xpshuai.cn/img/image-20220112164508498.png)





但是题目说了使用302，这里还是要学习一下这个知识点的。

使用短网址，将`http://127.0.0.1/flag.php`转为短网址：



利用生成的短地址构造Payload：`?url=surl-2.cn/0nPI`，发送请求，利用302跳转，得到flag：

![image-20220112163235049](http://image.xpshuai.cn/img/image-20220112163235049.png)



![image-20220112165633914](http://image.xpshuai.cn/img/image-20220112165633914.png)











## 302跳转

> 可绕过ssrf一些利用限制



使用dict协议查看端口开放情况，当端口开放时会返回 Bad request 3 当端口未开放时：利用gopher协议反弹shell 或者限制了只能使用HTTP,HTTPS，设置跳转重定向为True（默认不跳转）**即利用302跳转**







短网址绕过（DNS重定向302） http://t.cn/RwbLKDx

xip.io来绕过(DNS重定向302)：http://xxx.192.168.0.1.xip.io/ == 192.168.0.1 (xxx 任意） http://xip.io  指向任意ip的域名：xip.io(37signals开发实现的定制DNS服务)









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ssrf-302%E8%B7%B3%E8%BD%AC-bypass/  

