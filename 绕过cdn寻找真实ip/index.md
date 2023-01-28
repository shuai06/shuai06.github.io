# 绕过CDN寻找真实IP



在没有CDN的情况下，直接`ping`或`nslookup`或`dig`即可得到真实IP地址，但是有的站点使用了CDN。
两种简单检测有无CDN的方法：
1. 多地ping
2. nslookup 检测（如果下方域名解析出现了多个ip的话，基本就确定这个网站使用了CDN服务）



#### 一、DNS历史解析记录
相关查询网站：
```bash
# iphistory：
https://viewdns.info/iphistory/
# DNS查询：
https://dnsdb.io/zh-cn/
# 微步在线：
https://x.threatbook.cn/
# 域名查询：
https://site.ip138.com/
# DNS历史查询：
https://securitytrails.com/
# Netcraft：
https://sitereport.netcraft.com/?url=github.com

# SecurityTrail
```



#### 二、查找子域名
重要的站点会做CDN，而一些子域名站点并没有加入CDN，而且跟主站在同一个C段内，这时候，就可以通过查找子域名来查找网站的真实IP。
**常用的子域名查找方法和工具：**
1、搜索引擎查询：如Google、baidu、Bing等传统搜索引擎，`site:baidu.com inurl:baidu.com`，搜target.com|公司名字。

2、一些在线查询工具，如：
```bash
http://tool.chinaz.com/subdomain/
http://i.links.cn/subdomain/    
http://subdomain.chaxun.la/
http://searchdns.netcraft.com/
https://www.virustotal.com/
https://dnsdb.io/zh-cn/
https://x.threatbook.cn/
```

3、 子域名爆破工具
```bash
Layer子域名挖掘机
wydomain：https://github.com/ring04h/wydomain    
subDomainsBrute:https://github.com/lijiejie/
Sublist3r:https://github.com/aboul3la/Sublist3r
其他工具
```



#### 三、网站邮件头信息
邮箱注册，邮箱找回密码、RSS邮件订阅等功能场景，通过网站给自己发送邮件，从而让目标主动暴露他们的真实的IP，`查看邮件头信息`，获取到网站的真实IP。


#### 四、网络空间安全引擎搜索
关键字或网站域名，就可以找出被收录的IP，很多时候获取到的就是网站的真实IP
```bash
# 钟馗之眼：
https://www.zoomeye.org
# Shodan：
https://www.shodan.io
# Fofa：
https://fofa.so
# 其他
```


#### 五、利用SSL证书寻找真实IP
证书颁发机构(CA)必须将他们发布的每个SSL/TLS证书发布到公共日志中，SSL/TLS证书通常包含域名、子域名和电子邮件地址。
**SSL证书搜索引擎：**
```bash
# Censys 证书搜索
https://censys.io/ipv4?q=github.com
# 其他
```



#### 六、国外主机解析域名
大部分 CDN 厂商因为各种原因只做了国内的线路，而针对国外的线路可能几乎没有

1.国外多地ping工具：
```bash
https://asm.ca.com/zh_cn/ping.php
http://host-tracker.com/
http://www.webpagetest.org/
https://dnscheck.pingdom.com/
```

2.自己挂代理



#### 七、扫描全网
通过Zmap、masscan等工具对整个互联网发起扫描，针对扫描结果进行关键字查找，获取网站真实IP

1、ZMap号称是最快的互联网扫描工具，能够在45分钟扫遍全网。
```
https://github.com/zmap/zmap
```
2、Masscan号称是最快的互联网端口扫描器，最快可以在六分钟内扫遍互联网。
```
https://github.com/robertdavidgraham/masscan
```

扫描全网开放特定端口的IP，然后获取他们的特定页面的HTM源代码，用这些源代码和目标网站的特定页面的HTM源代码做对比，如果匹配上来了，就很可能是目标网站的真实P，工具匹配会匹配出来很多，最后还是要人工筛选。



#### 八、配置不当导致绕过
在配置CDN的时候，需要指定域名、端口等信息，有时候小小的配置细节就容易导致CDN防护被绕过。

案例1：为了方便用户访问，我们常常将www.test.com 和 test.com 解析到同一个站点，而CDN只配置了www.test.com，通过访问test.com，就可以绕过 CDN 了。

案例2：站点同时支持http和https访问，CDN只配置 https协议，那么这时访问http就可以轻易绕过。


#### 九、遗留文件
遗留文件（比如google镜像，百度镜像）
比如遗留文件中: phpinfo的`server_ddr`字段， `http_x_forwarded_for`  



#### 十、其他工具
```bash
# fuckcdn
https://github.com/Tai7sy/fuckcdn

# w8fuckdn
https://github.com/boy-hack/w8fuckcdn
```




#### 十一、以量打量
以量打量（需要肉鸡，cdn的流量用完就显示真实ip）

>ps：这个方法，额，也算是个方法吧......



#### 十二、其他方法
**1.网站漏洞**
比如有代码执行漏洞、SSRF、存储型的XSS都可以`让服务器主动访问我们预设的web服务器`，那么就能在日志里面看见目标网站服务器的真实IP

**2.配合本地hosts文件**
用超级ping 扫子域名 再继续、 及get-site-ip获取 (修改host的映射，如能正常访问，这ip就是正确的，进一步使用代理来找出真是ip)


#### 十三、一键干死CDN
>包含了夸张因素，针对大公司的效果比较好，体量小的网站，基本搜不到

1.先查看网站logo的hash值
这里python2来计算：
```python
import mmh3   # mmh3安装需要先安装vc++4.0
import requests
response = requests.get('https://www.xiaodi8.com/img/favicon.ico')
favicon = response.content.encode('base64')
hash = mmh3.hash(favicon)
print 'http.favicon.hash:'+str(hash)

# 可以ico文件，也可以是其他特征明显的图片等
```

2.然后shodan搜索：
```bash
# 对特征明显的图片等的hash进行搜索
http.favicon.hash:-1507567567
```




参考文章
```bash
https://mp.weixin.qq.com/s/JkqLu0SBcOGIq7JdLzj8Uw   # ByPass, 我在此基础上补充了自己之前的内容
https://www.cnblogs.com/zuoxiaolongzzz/p/12496467.html  # 通过shodan搜索相同favicon.ico的网站
https://www.cnblogs.com/miaodaren/p/9177379.html        # Shodan的http.favicon.hash语法详解与使用技巧
```




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E7%BB%95%E8%BF%87cdn%E5%AF%BB%E6%89%BE%E7%9C%9F%E5%AE%9Eip/  

