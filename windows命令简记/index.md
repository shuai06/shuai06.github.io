# Windows命令简记





  
#### 1、windows系统基础命令
  
```powershell
ipconfig /all 获取ip地址     linux：ifconfig -a
Route print   路由信息
Arp -a        arp缓存
Netsh firewall show config  查看防火墙规则
Netsh firewall show state   
Netstat -an   获取端口信息
Whoami        当前用户权限
Hostname      主机名称
Set           环境变量
Query user    查看远程终端在线用户
Systeminfo    获取操作系统版本、类型、位数等相关信息、安装；
-b：      显示包含于常见每个链接或监听端口的可执行组件；
-o：      显示与每个连接相关的所属进程ID；
-v：      与b一起使用时将显示包含于为所有可执行组件创建连接或者监听端口的组件；
Netstat -anb  进程号、端口开放情况、开放端口程序、监听端口组件
Netsata -ano  tcp/udp协议信息、端口、进程号
Netstat -anvb 进程号、端口所用协议、调用的可执行组件、第三方进程的系统路径等
Net start 获取服务信息利用第三方漏洞提权、关闭杀毒软件、防火墙、以及关闭某些防护进程
Net stop servicesname  停止服务命令
Net start servicename  开启服务命令
Tasklist /svc          获取运行的进程名称、服务、PID 
Driverquery            查看已安装驱动程序列表
Net start              查看已经启动的windows服务
Msinfo32               获取更加详细的信息
Taskkill               是windows自带的终止进程程序
TASKKILL [/S system [/U username [/P [password]]]]{ [/FI filter] [/PID processid | /IM imagename] } [/T] [/F]
#例如：taskkill /pid 452 /f        taskkill /im 360tray /f

#用户管理命令：
Net user hack 123 /add    添加hack用户 密码为123
Net localgroup adminnistrators hack /add   添加hack为管理员权限
Net localgroup adminnistrators    查看当前系统管理员
Net localgroup "remote desktop users" hack /add   加入远程桌面用户组
Net user hack     查看指定用户的信息
Net user guest /active:yes   激活guest用户
Net user guest 123
```

#### 2、windows系统开启远程桌面（mstsc/rdesktop）

```powershell
#XP/Win2k3/Win7/Win2k8/Win8.1/Win10/2012/2016（0：ON、1：OFF）：
REG ADD HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server /v fDenyTSConnections /t REG_DWORD /d 0 /f
                                                              

#Win2k3/Win7/Win2k8/Win8.1/Win10/2012/2016：（0：ON、1：OFF）：
wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1

#Winserver2008/2012/Win2k3/win7
wmic  /namespace:\\root\cimv2\terminalservices path win32_terminalservicesetting where (__CLASS !="") call setallowtsconnections 1
                            

#Winserver2008/2012/
wmic /namespace:\\root\cimv2\terminalservices path win32_tsgeneralsetting where (TerminalName ='RDP-Tcp') call setuserauthenticationrequired 1

#XP/Win2k3/Win7/Win2k8/Win8.1/Win10/2012/2016（Metasploit）：
meterpreter > run getgui -e
```

#### 3、查看远程端口

###### 通过注册表查看：

```
REG query HKLM\SYSTEM\CurrentControlSet\Control\Terminal" "Server\WinStations\RDP-Tcp /v PortNumber 查询rdp端口号
set /a port = query hex value 
```

###### 通过命令查看

```powershell
Tasklist /svc |find "TermService"
Netstat -ano |find "1980"
```



> 本文转载

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/windows%E5%91%BD%E4%BB%A4%E7%AE%80%E8%AE%B0/  

