# URL跳转漏洞






## 漏洞简介

**URL跳转：**目前很多Web应用因为业务需要，需要与内部的其他服务或者第三方的服务进行交互，这样就需要重定向的功能，由当前网页跳转到第三方网页。

比如Java中可通过Header的重定向功能实现url的跳转

```java
response.sendRedirect(request.getParameter("url"));
```

使用的时候就是：

```
http://www.test.com?url=http://www.xxx.com
```



**URL跳转漏洞：**也叫URL重定向漏洞。由于服务端未对传入的跳转url变量进行检查和控制，可导致恶意用户构造一个恶意地址，诱导用户跳转到恶意网站。因为是从用户可信站点跳转出去的，用户会比较信任该站点，所以其常用于用于钓鱼攻击，通过跳转到恶意网站欺骗用户输入用户名和密码来盗取用户信息，或欺骗用户进行金钱交易；还可以造成xss漏洞。

![一次URL跳转攻击](http://image.xpshuai.cn/img/image-20220117201314903.png)



例子：

```php
<?php
$url=$_GET['url'];
header("Location: $url");

?>
```

恶意用户可以提交:`http://www.aaa.com/login.php?...://www.bbb.com(钓鱼网站)`

来生成自己的恶意链接，安全意识较低的用户很可能会以为该链接展现的内容是www.aaa.com从而可能产生欺诈行为

![跟上链接，这里以百度为例](http://image.xpshuai.cn/img/image-20220117204812603.png)

![成功跳转](http://image.xpshuai.cn/img/image-20220117204908240.png)



## 产生位置

1. 用户**登录**(最常见)、统一身份认证处，认证完后会跳转(在登陆的时候建议多观察url参数)
2. 用户分享、收藏内容过后，会跳转 
3. 跨站点认证、授权后，会跳转 
4.  站内点击其它网址链接时，会跳转

5. 在一些用户交互页面也会出现跳转，如请填写对客服评价，评价成功跳转主页，填写问卷，等等业务，注意观察url
6. 业务完成后跳转这可以归结为一类跳转，比如修改密码，修改完成后跳转登陆页面，绑定银行卡，绑定成功后返回银行卡充值等页面，或者说给定一个链接办理VIP，但是你需要认证身份才能访问这个业务，这个时候通常会给定一个链接，认证之后跳转到刚刚要办理VIP的页面。





## 实现方式

```
1.META标签内跳转
2.javascript跳转
3.header跳转
```





## 漏洞检测

### 检测方法

修改参数中合法的URL为非法URL，然后查看是否能正常跳转或者响应是否包含了任意的构造URL。



### 常见产生漏洞的参数名

在黑盒测试时，需多留意关注常见的可能产生漏洞的参数名，可以用来fuzz：

```bash
# 注意URL中是否含有以下参数值
callback
redirect
redirect_to
redirect_url
redirectUrl
toUrl
url
return
ReturnUrl
fromUrl
goto
to
jump
jump_to
target
redUrl
link
linkto
domain
oauth_callback
...

# 并注意观察后跟的URL地址的具体格式，再构造相应的payload尝试跳转
```





## URL跳转Bypass



1.最常用的：@绕过

```
url=http://www.aaaa.com@www.xxx.com(要跳转的页面)他有的可能验证只要存在aaaa.com就允许访问，做个@解析，实际上我们是跳转到xxx.com的
```



2.绕过一些匹配特定字符

2.1  ?问号绕过

```bash
url=http://www.aaaa.com?www.xxx.com
```

2.2 #绕过

```
url=http://www.aaaa.com#www.xxx.com
```



3.斜杠`/`绕过

```bash
url=http://www.aaaa.com/www.xxx.com
```

反斜线`\`绕过

```bash
url=http://www.aaaa.com\www.xxx.com
```



4.白名单匹配绕过

```bash
#比如匹配规则是必须跳转，aaa.com 域名下，?#等都不行的时候，aaa.com，可以尝试百度inurl:aaa.com的域名，比如xxxaaa.com，这样同样可以绕过。
```



5.xip.io绕过

```bash
# 在绕过ssrf限制中使用过

# 当你访问aaa.com这个域名时，其实这个链接已经被解析到后面这个ip地址上了，那么实际访问的就是后面这个IP地址
url=http://www.aaa.com.220.181.57.217.xip.io
```



6.白名单网站可信

```
如果url跳转点信任百度url，google url或者其他，则可以多次跳转达到自己的恶意界面。
```



7.协议绕过

```bash
#http与https协议转换尝试，或者省略
redict=//www.aaa.com@www.baidu.com

redict=////www.aaa.com@www.baidu.com	# //多斜线
```





8.xss跳转

```html
# 就是XSS造成的跳转，在有些情况下XSS只能造成跳转的危害。 

<meta  content="1;url=http://www.baidu.com" http-equiv="refresh">
```







## 代码审计

> 在白盒审计中，我们则会重点关注可以进行URL跳转的相关方法。



### Spring MVC中使用重定向常用的方法

1.使用ModelAndView方式

2.通过返回String方式

3.使用sendRedirect方式

4.通过设置Header来进行跳转





### 常用关键字

根据以上常用方法，可以提取如下关键字：

```bash
redirect
sendRedirect
ModelAndView
Location
addAttribute
getHost
setHeader
forward
...
```

其他关键字补充：

相关的参数名：

```bash
# 注意URL中是否含有以下参数值
callback
redirect
redirect_to
redirect_url
redirectUrl
toUrl
url
return
ReturnUrl
fromUrl
goto
to
jump
jump_to
target
redUrl
link
links
linkto
domain
oauth_callback


```



在审计中，我们就可以先搜索url跳转中常见的关键字，定位到可能存在问题的代码区域，并查看前后逻辑及参数是否可控...











## 漏洞修复

- 最有效的是，严格控制要跳转的域名。若已知需要跳转URL，则可以直接在代码中写成固定值
- 也可以使其只根据路径跳转，而不是根据其URL参数跳转
- 若url事先无法确定，只能通过前端参数传入，则必须在跳转的时候对url进行按规则校验：即控制url是否是你们公司授权的白名单或者是符合你们公司规则的url
- XSS漏洞的注意事项 ：跳转url检测中也加入了CRLF头部注入漏洞的检测逻辑, 具体就是在请求参数中加入了`%0d%0a`这种测试代码，需要对这些参数进行删除处理(事实上：在判断到一个参数中包含` %00 -> %1f `的控制字符时都是不合法的，需对其进行删除)。
- 设置二次提醒，提醒用户





## 参考

https://www.cnblogs.com/linuxsec/articles/10926152.html


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/url%E8%B7%B3%E8%BD%AC%E6%BC%8F%E6%B4%9E/  

