# 黑暗引擎



 
#### 一. shodan

###### 1.简单介绍

Shodan，是一个暗黑系的谷歌，作为一个针对网络设备的搜索引擎，它可以在极短的时间内在全球设备中搜索到你想找的设备信息。对于渗透工作者来说，就是一个辅助我们寻找靶机的好助手。

###### 2.内置语法 

```bash
# Shodan的参数有很多，这里只介绍简单的几种
hostname："主机或域名"
如 hostname:"google''

port："端口或服务"
如 port:"21"

ip : "ip地址"
如 ip : "168.205.71.64"

net："IP地址或子网"

如 net:"210.45.240.0/24"

vuln :指定漏洞的cve

如 vuln:CVE-2015-8869

但是这个命令最好搭配起来使用，如 country:CN vuln:CVE-2014-0160

os :"操作系统"

​ 如 os:"centOS"

isp："ISP供应商"

如 isp:"China Telecom"

product："操作系统/软件/平台"

如 product:"Apache httpd"

version："软件版本"

如 version:"3.1.6"

geo："经纬度"

如 geo:"39.8779,116.4550"

country`："国家"

如 country:"China"
country:"UN"

city："城市"
如 city:"Hefei"

org："组织或公司"
如 org:"google"

before/after："日/月/年"
如 before:"25/09/2017"
after:"25/09/2017"

asn : "自治系统号码"
如 asn:"AS2233"
```

###### 3.脚本使用  （建议使用本地脚本，也可自主开发）

	项目地址：https://github.com/achillean/shodan-python

先clone, python setup.py install, shodan init 秘钥_value,然后可以在命令行用了就
`shodan search product:"apache"`  只不过显示方式不友好


拓展：自主开发

	官方文档：https://shodan.readthedocs.io/en/latest/index.html

```python
from shodan import Shodan

api = Shodan("cUNKhp6cIaN7J4Y9hXnBtbtT8bd9SKCf")

# Lookup an IP
ipinfo = api.host('8.8.8.8')
print(ipinfo)

# Search for websites that have been "hacked"
for banner in api.search_cursor('ch1_http相关.title:"hacked by"'):
    print(banner)

# Get the total number of industrial control systems services on the Internet
ics_services = api.count('tag:ics')
print('Industrial Control Systems: {}'.format(ics_services['total']))
```



###### 案例测试：

1.phpstudy采集
头部信息 `Server: Apache/2.4.23 (Win32) OpenSSL/1.0.2j mod_fcgid/2.3.9`

2.子域名信息采集
`hostname:"xps.cn"`

3.特定漏洞靶机采集
`vuln:CVE-2020-1983` 普通用户没权限用





#### 二. zoomeye

>对国内更贴近一些

###### 内置语法

	ZoomEye搜索技巧
	指定搜索的组件以及版本
	app：组件名称
	ver：组件版本
	例如：搜索 apache组件    版本2.4
	app:apache ver:2.4
	
	指定搜索的端口
	port:端口号
	例如：搜索开放了SSH端口的主机
	port:22
	一些服务器可能监听了非标准的端口。
	要按照更精确的协议进行检索，可以使用service进行过滤。
	
	指定搜索的操作系统
	OS:操作系统名称
	例如：搜索Linux操作系统
	OS：Linux
	
	指定搜索的服务
	service：服务名称
	例如，搜素SSH服务
	Service：SSH
	
	指定搜索的地理位置范
	country：国家
	city：城市名
	例如：
	country:China
	city：Beijing
	
	搜索指定的CIDR网段
	CIDR:网段区域
	例如：
	CIDR：192.168.158.12/24
	
	搜索指定的网站域名
	Site:网站域名
	例如：
	site:www.baidu.com
	
	搜索指定的主机名
	Hostname:主机名
	例如：
	hostname:zwl.cuit.edu.cn
	
	搜索指定的设备名
	Device：设备名
	例如：
	device:router
	
	搜索具有特定首页关键词的主机
	Keyword：关键词
	例如：
	keyword:technology


###### 脚本开发(多语言开发)

	官方手册:https://www.zoomeye.org/doc

1.登录授权
2.请求搜索
3.回显数据




#### 补充：谷歌黑客高级玩法

	https://www.uedbox.com/shdb/
	https://www.uedbox.com/post/54776/





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E9%BB%91%E6%9A%97%E5%BC%95%E6%93%8E/  

