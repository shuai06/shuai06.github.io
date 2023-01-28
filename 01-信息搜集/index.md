# 01-信息搜集





<!-- more -->

>一些web工具： https://www.uedbox.com/tools/type/webapp


# 1.收集域名信息

#### Whois查询

**在线工具：**

- 爱站网 
- 站长之家
- VirusTotal

#### 备案信息查询

- ICP备案查询网
- 天眼查

# 2.收集敏感信息

#### Google hacking

**常用语法：**
site:
inurl:
intext:
filetype:
intitle:
link:
info:
cache:      缓存

比如：site:edu.cn intext:后台管理  

还可以用它来搜集 后台，数据库文件，SQL注入，配置信息，源代码泄露，未授权访问，robots.txt等敏感信息  

百度，必应，Yandex, 雅虎，Shodan等类似  




# 3.收集子域名信息

#### 子域名检测工具

**工具：**

- Layer子域名挖掘机     (推荐)
- K8
- wydomain
- Sublist3r                    (推荐， 能列出多种资源)
- dnsmaper
- subDomainsBrute      (推荐，python开发的， 可以使用小字典递归地发现三四五级域名)
- Maltego CE 
- ......



#### 搜索引擎枚举

比如：
搜索百度下面的子域名  
site:baidu.com  


#### 第三方聚合应用枚举

很多第三方服务汇聚了大量DNS数据集， 可以通过他们检索某个给定域名的子域名  

**工具：**

- DNSdumpster 网站  
- 在线DNS侦查和搜索的工具  
  来挖掘制定域潜藏的大量子域  


#### 证书透明度(CT)公开日志枚举

是证书授权机构(CA)的一个项目， CA会将每个SSL/TLS证书发布到公共日志中。  
一个SSL/TLS证书通常包含域名、子域名、邮件地址  ... ... 

**方法：**
最简单的就是： 使用搜索引擎搜索一些公开的CT日志  
**推荐：**

- crt.sh:      https://crt.sh 网站
- https://censys.io  网站  

**其他工具：**
一些在线网站 

- 子域名爆破网站：   https://phpinfo.me/domain
- IP反查绑定域名网站：  https://dns.zizhan.com




# 4. 收集常用端口信息

> 方便对症下药  

**工具：**

- Nmap
- 无状态扫描工具 Masscan
- ZMap
- 御剑高速TCP端口扫描工具

https://www.cnblogs.com/botoo/p/10475402.html

#### 常用端口和攻击方向汇总：

###### 文件共享服务端口

| 端口号       | 端口说明                 | 攻击方向                             |
| ------------ | ------------------------ | ------------------------------------ |
| 20/21/**69** | Ftp/**Tftp**文件传输协议 | 允许匿名的上传、下载、爆破和嗅探操作 |
| 2049         | Nfs服务                  | 配置不当                             |
| 139          | Samba                    | 爆破、未授权访问、远程代码执行       |
| 389          | Ldap目录访问权限         | 注入、允许匿名访问、弱口令           |

注：FTP协议的 21端口 --> 传输控制信息， 20端口 ---> 传输数据

###### 远程连接服务端口

| 端口 | 端口说明        | 攻击方向                                             |
| ---- | --------------- | ---------------------------------------------------- |
| 22   | SSH远程连接     | 爆破、SSH隧道及内网代理转发、文件传输                |
| 23   | Telnet远程连接  | 爆破、嗅探、弱口令                                   |
| 3389 | Rdp远程桌面连接 | Shift后门（需Windows Server 2003 及以下系统 ）、爆破 |
| 5900 | VNC             | 弱口令爆破                                           |
| 5632 | PyAnywhere      | 抓密码、代码执行                                     |



###### Web应用服务端口

| 端口号      | 端口说明                  | 攻击方向                          |
| ----------- | ------------------------- | --------------------------------- |
| 80/443/8080 | 常见的web服务端口         | Web攻击、爆破、对于服务器版本漏洞 |
| 7001/ 7002  | WebLogic控制台            | Java反序列化、弱口令              |
| 8080/8089   | JBoss/Resin/Jetty/Jenkins | 反序列化、控制台弱口令            |
| 9090        | WebSphere控制台           | Java反序列化、弱口令              |
| 4848        | GlassFish 控制台          | 弱口令                            |
| 1352        | Lotus domino邮件服务      | 弱口令、信息泄露、爆破            |
| 10000       | Webmin-Web控制面板        | 弱口令                            |



###### 数据库服务端口

| 端口号      | 端口说明          | 攻击方向                     |
| ----------- | ----------------- | ---------------------------- |
| 3306        | Mysql             | 注入、提权、爆破             |
| 1433        | MSSQL数据库       | 注入、提权、SA弱口令、爆破   |
| 1521        | Oracle数据库      | TNS爆破、注入、反弹Shell     |
| 5432        | PostgreSQL数据库  | 爆破、注入、弱口令           |
| 28017/27018 | MongoDB           | 爆破、未授权访问             |
| 6379        | Redis数据库       | 可尝试未授权访问、弱口令爆破 |
| 5000        | SysBase/DB2数据库 | 爆破、注入                   |



###### 邮件服务端口

| 端口 | 端口说明     | 攻击方向   |
| ---- | ------------ | ---------- |
| 25   | SMTP邮件服务 | 邮件伪造   |
| 110  | POP3协议     | 爆破、嗅探 |
| 142  | IMAP协议     | 爆破       |



###### 网络常见协议端口

| 端口  | 端口说明    | 攻击方向                              |
| ----- | ----------- | ------------------------------------- |
| 53    | DNS域名系统 | 允许区域传送、DNS劫持、缓存投毒、欺骗 |
| 67/68 | DHCP服务    | 劫持、欺骗                            |
| 161   | SNMP协议    | 爆破、搜集目标内网信息                |



###### 特殊服务端口

| 端口        | 端口说明                | 攻击方向            |
| ----------- | ----------------------- | ------------------- |
| 2181        | Zookeeper服务           | 未授权访问          |
| 8069        | Zabbix服务              | 远程执行、SQL注入   |
| 9200        | Elasticsearch服务       | 远程执行            |
| 11211       | Memcache服务            | 未授权访问          |
| 512/513/514 | Linux Rexec服务         | 爆破、Rlogin登录    |
| 873         | Rsync服务               | 匿名访问、文件上传  |
| 3690        | Svn服务                 | Svn泄露、未授权访问 |
| 50000       | SAP Mannagement Console | 远程执行1           |





# 5.指纹识别

应用程序在html, css, js 等文件中多多少少会包含一些特征码（比如: WordPress在robots.txt中会包含`wp-admin`, 首页index.php中会包含`generator=wordpress 3.xx`）,帮助快速识别CMS  

**常见的CMS(整站系统)：**

- Dedecms(织梦)
- Discuz
- PHPWEB
- PHPCMS
- ECShop
- Dvbbs
- SiteWeaver
- ASPCMS
- 帝国
- Z-Blog
- WordPress


**工具：**

- 御剑Web指纹识别
- What Web
- WEBrOBO
- 椰树
- 轻量WEB指纹识别
- ...

**在线网站工具：**

- BugScaner：      https://whatweb.bugscaner.com/look/
- 云悉指纹：         https://www.yunsee.cn/finnger.html
- What Web:        https://whatweb.net/

**推荐项目：**
https://github.com/Lucifer1993/cmsprint         国内的
https://github.com/anouarbensaad/vulnx       国外的
https://github.com/Lucifer1993/AngelSword      国内的  （停止维护）


**phpStudy搭建的网站，数据包都有以下特点**
`Server: Apache/2.4.23 (Win32) OpenSSL/1.0.23 mod_fcgid/2.3.9`



# 6.查找真实IP

如果目标服务器只有一个域名 ，获取真实ip就非常重要  
如果没有CDN，可以通过 `www.ip138.com`等工具获取目标的ip

CDN（内容分发网络， 通俗讲就是高速缓存服务器，把内容缓存到了距离我们最近的节点服务器上面...）

#### 判断有无CDN

采用`超级ping`第三方平台判断：唯一IP无cdn，反之有cdn
或者采用在线网站17CE (https://www17ce.com)进行全国多地区的ping服务器操作  

**工具：**

- 超级ping 
- 站长工具等


#### 绕过CDN寻找真实IP

###### （1）内部邮箱源

一般邮件系统都在内部，没有经过CDN解析  
通过网站`注册用户`或者`RSS`订阅功能，查看邮件头中的邮件服务器域名IP，ping这个邮件服务器的域名，就可以获取真实ip  
注意：必须是目标自己的邮件服务器，第三方公共邮件服务器不行  


###### （2）扫描网站测试文件

如：`phpinfo`, `test`等一文件，从而找到真实ip


###### （3）分站域名

很多主站访问量大，挂了CDN，但是分站有可能没有CDN  
可以通过ping二级域名获取分站ip， 很可能出现分站和主站部署同一个ip，但是在同一个C段下面的情况，从而判断目标的真实ip段  


###### （4）国外访问

较小或者业务面向国内的网站的CDN往往只对国内的用户访问加速，而国外就不一定了  
方法：

- 可以通过挂国外的代理
- 或者   使用国外的在线代理网站` App Synthetic Monitor  (https://asm.ca.com/en/ping.php)`访问
  可能会得到真实ip  


###### （5）查询域名解析记录

也许目标很久之前没有使用CDN  
通过网站`NETCRAFT （https://www.netcraft.com/）`来观察域名的ip历史记录，也可以大致分析出真实ip段   


###### （6）目标有自己的App

可以尝试用`Fiddller`或`Burp Suite` 抓取App的请求，从而找到目标的真实ip  


###### （7）绕过CloudFlare CDN查找真实IP

很多网站都使用 CloudFlare 提供的cdn服务  
可通过在线网站`FlareWatch (http://www.crimeflare.us/cfs.html#box)` 对CloudFlare客户网站进行真实ip查询  



###### (9) 其他方法

```
其他方法：遗留文件（比如google镜像），扫全网(比如)，黑暗引擎，dns历史记录（第三方平台，刚好开通cdn之前的ip地址），以量打量（需要肉鸡，cdn的流量用完就显示真实ip）
https://get-site-ip.com/
遗留文件中: phpinfo的server_ddr字段， http_x_forwarded_for
【文章：】
https://www.lstazl.com/cdn检测与绕过/
https://www.fujieace.com/penetration-test/cdn-find-ip.html
```

###### （8）验证获取的IP 是否真实？

如果是Web： 直接输入ip访问，看看相应页面是否和访问域名返回的一样  
目标段比较大的情况下：借助`Masscan`的工具批扫描对应IP段中所有开了80,443,8080端口的ip，然后逐个ip尝试，观察响应结果是否是目标站点  




# 7.收集敏感目录 文件 

从中可以发现隐藏的后台管理页面，文件上传页面，甚至网站备份文件和源码  

**工具：**

- DirBuster                      java编写的，图形化界面  
- 御剑后台扫描珍藏版
- wwwscan
- Spinder.py (轻量级快速单目录后台扫描)
- Sensitivefilescan (轻量级快速单文件目录后台扫描 )
- Weakfilescan (轻量级快速单文件目录后台扫描 )
- awvs 爬行  
- webPathBrute
- robots.txt

**很多在线工具：**

- WebScan   (http://www.webscan.cc/)


# 8.社会工程学

邮件  
社工库  
......


# 9. 其他 web信息搜集

#### 网站源码脚本类型

php, java, python, go, node.js, ......

**脚本类型：** 

1. url直接显示: `xxx.php` 可以直接判断
   2.如果是伪静态： 
   多构造请求数据包，f12，network中的数据包中，可以看到请求的URL(...xxx.jsp)或者接口... 
   比如：`Request URL: http://www.cngis.cn/news/detail.aspx?classid=216454257090494464&id=140` 这里是aspx的

>伪静态怎么注入没理解？？？
> **伪静态** 不是真正意义上的静态格式文件 : 展示的格式是...xxx.html  
> **测试方法：** f12打开network，查看请求数据包（）
>           多提交地址访问抓包分析



#### 网站对应数据库

常用的组合匹配：  如php程序 判定mysql aspx 判定mssql
常见组合： asp+access php+mysql aspx+mssql jsp+mssql或oracle等

端口扫描判定：内网服务器方法失效
access无端口开发， mysql:3306， mssql:1433， oracle:1521，MongoDB：27017,Postgresql:5432, redis:6379等

参考文章：https://www.cnblogs.com/botoo/p/10475402.html


#### 网站搭建平台

查看元素或审查元素抓包获取
比如Server:      Nginx, iis等中间件信息

#### 服务器操作系统

大小写判断 windows大小写不敏感 linux相反


#### 有无WAF防护

**工具：**
可以借用 wafw000f 脚本判断waf类别。  https://github.com/EnableSecurity/wafw00f              wafw00f http://www.cngis.cn

应用的防火墙、硬件的防火墙

SQLMAP的几个WAF绕过方法：   https://9bie.org/index.php/archives/465/


#### 有无其他应用

APP或手机站点 第三方接口（微信公众号等）


#### IP网段网络信息

** IP : **    IP对应网站 IP对应目录扫描   （设置的时候，目录设置为www.xxx,com。 192.168.1.112访问的是站点，但是www.xxx.com访问的是站点下面的一个目录。一个ip对应多个域名多个站点，单一个域名只有一个ip）
**网段：**   C段信息扫描 网段 192.168.0.1 -192.168.0.255     在线C段查询（拿到同网段后，再进行另一个主机，相当于内网渗透了）
**工具:**

- DotNetScan
- iisput增强版


#### robots协议

直接访问地址/robots.txt获取相关目录地址信息




## shodan



## zoomeye

### fofa

>利用搜索。或者脚本开发进行批量/调用

---

# 伪静态

如果看到一个以.html或者.htm结尾的网页，此时可以通过呢在在地址输入框中输入：
javascript:alert(document.lastModified)，来得到网页最后的修改时间，如果得到的时间和现在时间一致，此页面就是伪静态，反之是真静态；因为动态页面的最后修改时间总是当前时间，而静态页面的最后修改时间则是它生成的时间。

https://blog.csdn.net/Fly_hps/article/details/79487137

https://blog.csdn.net/zezai3010/article/details/78797944

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/01-%E4%BF%A1%E6%81%AF%E6%90%9C%E9%9B%86/  

