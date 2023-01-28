# Python解析IP地址CIDR的形式




### 方法1：使用IPy库

```python

from IPy import IP
ip = IP('127.0.0.0/30')
for x in ip:
	print(x)     

```



### 方法二：使用netaddr库

- CIDR也能直接转成IP地址段

```python
from netaddr import *
ip = IPNetwork('192.0.2.16/29')
ip_list = list(ip)
print(ip_list)

[IPAddress('192.0.2.16'), IPAddress('192.0.2.17'), IPAddress('192.0.2.18'), IPAddress('192.0.2.19'), IPAddress('192.0.2.20'), IPAddress('192.0.2.21'), IPAddress('192.0.2.22'), IPAddress('192.0.2.23')]
```





- IP段208.130.29.30-35转换成CIDR格式

```python

from netaddr import *
startip = '208.130.29.30'
endip = '208.130.29.35'
cidrs = netaddr.iprange_to_cidrs(startip, endip)
for k, v in enumerate(cidrs):
	iplist = v
 	print(iplist)


```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/python%E8%A7%A3%E6%9E%90ip%E5%9C%B0%E5%9D%80cidr%E7%9A%84%E5%BD%A2%E5%BC%8F/  

