# 03-漏洞发现

 
### 基于WEB应用扫描测试

    被动式扫描 x-ray    （长亭科技开发的）
    https://xray.cool/xray/#/tutorial/introduce

Github: https://github.com/chaitin/xray/releases （国外速度快）
网盘: https://yunpan.360.cn/surl_y3Gu6cugi8u （国内速度快）
    **主动式扫描 **：工具：   awvs（可以录制填写账号密码）、 netsparker、 appscan(特别占用电脑资源)
**    注意验证后扫描 **
    无验证码：模拟登陆    比如  awvs（可以录制填写账号密码）
    有验证码：带入cookie   比如  awvs（桌面版：扫描option->cookie。网页版：custom cookie, 把网页数据包赋值过来）
    

    抓app数据包的差异导致问题：awvs(custome  header的添加；旧版：header and cookie)

### 基于操作系统扫描测试

    Nessus 
    Openvas
    Goby


### 基于NMAP使用扫描测试

https://blog.csdn.net/qq_29277155/article/details/50977143


## 其他安全漏洞扫描补充

    未授权访问
    中间件安全
    逻辑越权问题





>更多安全扫描见：https://www.uedbox.com/tools/type/scanner/

>注意：部分漏洞（越权）利用工具无法探针，只能手工

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/03-%E6%BC%8F%E6%B4%9E%E5%8F%91%E7%8E%B0/  

