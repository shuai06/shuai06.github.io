# 发现内网存活主机方法-简单整理


  
#### <font color=red>基于UDP发现内网存活主机  </font>
###### UDP显著特性:
1.UDP 缺乏可靠性。UDP 本身不提供确认,超时重传等机制。UDP 数据报可能在网络中被复制,被重新排序,也不保证每个数据报只到达一次。
2.UDP 数据报是有长度的。每个 UDP 数据报都有长度,如果一个数据报正确地到达目的地,那么该数据报的长度将随数据一起传递给接收方。而 TCP 是一个字节流协议,没有任
何(协议上的)记录边界。
3.UDP 是无连接的。UDP 客户和服务器之前不必存在长期的关系。大多数的UDP实现中都选择忽略源站抑制差错,在网络拥塞时,目的端无法接收到大量的UDP数据报
4.UDP 支持多播和广播。

###### 1.nmap扫描
```bash
# -sU
nmap -sU -T5 -sV --max-retries 1 192.168.1.100 -p 500
```

###### 2.msf扫描
```bash
use auxiliary/scanner/discovery/udp_probe

# 或这个模块
use auxiliary/scanner/discovery/udp_sweep
```

###### 3.unicornscan扫描
Linux下推荐
```bash
unicornscan -mU 192.168.1.100
```

###### 4.ScanLine扫描
McAfee出品,win下使用推荐。管理员执行。
项目地址:https://www.mcafee.com/ca/downloads/free-tools/scanline.aspx
网盘地址:http://pan.baidu.com/s/1i4A1wLR 密码:hvyx
自己搜索资源吧

输入`sl`运行ScanLine


###### 5.在线基于Nmap的udp扫描:
[点击链接](https://pentest-tools.com/network-vulnerability-scanning/udp-port-scanner-online-nmap)



  
####  <font color=red> 基于ARP发现内网存活主机</font>
###### 1.nmap扫描
```bash
nmap -sn -PR 192.168.1.1/24
```

###### 2.msf扫描
```bash
use auxiliary/scanner/discovery/arp_sweep
...
```

###### 3.netdiscover
```bash
netdiscover -r 192.168.1.0/24 -i wlan0
```

###### 4.arp-scan(linux)
(推荐)速度与快捷
项目地址:`https://linux.die.net/man/1/arp-scan`
arp-scan没有内置kali,需要下载安装。
```bash
apt-get install arp-scan

arp-scan --interface=wlan0 --localnet
```

###### 5.Powershell
```bash
powershell.exe -exec bypass -Command "Import-Module .\arpscan.ps1;Invoke-ARPScan -CIDR 192.168.1.0/24"
```

###### 6.arp scannet
项目地址: https://sourceforge.net/projects/arpscannet/files/arpscannet/arpscannet%200.4/


###### 7.arp-scan(windows)
(推荐)速度与快捷
项目地址:https://github.com/QbsuranAlang/arp-scan-windows-/tree/master/arp-scan(非官方)
```bash
arp-scan.exe -t 192.168.1.1/24
```

###### 8.arp-ping.exe
```bash
arp-ping.exe 192.168.1.100
```

###### 9.其他
如cain的arp发现,一些开源py,pl脚本等,不一 一介绍。

>下载的工具，后门自查



  
####  <font color=red> 基于netbios发现内网存活主机</font>
IBM公司开发,主要用于数十台计算机的小型局域网。该协议是一种在局域网上的程序可以
使用的应用程序编程接口(API),为程序提供了请求低级服务的同一的命令集,作用是为了给局域网提供网络以及其他特殊功能。
系统可以利用WINS服务、广播及Lmhost文件等多种模式将NetBIOS名-——特指基于
NETBIOS协议获得计算机名称——解析为相应IP地址,实现信息通讯,所以在局域网内部使
用NetBIOS协议可以方便地实现消息通信及资源的共享。

###### 1.nmap扫描
```bash
# nbstat.nse
nmap -sU --script nbstat.nse -p137 192.168.1.0/24 -T4
```

###### 2.msf扫描
```bash
use auxiliary/scanner/netbios/nbname
```

###### 3.nbtscan扫描
项目地址:  http://www.unixwiz.net/tools/nbtscan.html
NBTscan version 1.5.1:  https://github.com/scallywag/nbtscan
```bash
### Windows
nbtscan-1.0.35.exe -m 192.168.1.0/24

nbtscan -n    # 推荐


### Linux （推荐）
tar -zxvf ./nbtscan-source-1.0.35.tgz     # (其他版本自己寻找)
make
nbtscan -r 192.168.1.0/24
nbtscan -v -s: 192.168.1.0/24
```

###### 4.NetBScanner:
项目地址:https://www.nirsoft.net/utils/netbios_scanner.html



  
####  <font color=red> 基于snmp发现内网存活主机</font>
**SNMP协议主要由两大部分构成:** SNMP管理站和SNMP代理。
SNMP管理站是一个中心节点,负责收集维护各个SNMP元素的信息,并对这些信息进行处理,最后反馈给网络管理员;而SNMP代理是运行在各个被管理的网络节点之上,负责统计该节点的各项信息,并且负责与SNMP管理站交互,接收并执行管理站的命令,上传各种本地的网络信息

###### 1.nmap扫描
```bash
nmap -sU --script snmp-brute 192.168.1.0/24 -T4
```

###### 2.msf扫描
```bash
use auxiliary/scanner/snmp/snmp_enum
```

###### 3.SMBScan
项目地址:https://www.mcafee.com/us/downloads/free-tools/snscan.aspx
依然是一块macafee出品的攻击


###### 4.NetCrunch
项目地址:https://www.adremsoft.com/demo/
内网安全审计工具,包含了DNS审计,ping扫描,端口,网络服务等。


###### 5.snmp for pl扫描:
项目地址:https://github.com/dheiland-r7/snmp


###### 其他扫描:
**6.snmpbulkwalk**

**7.snmp-check**

**8.snmptest**


**附录**
```bash
use auxiliary/scanner/snmp/aix_version
use auxiliary/scanner/snmp/snmp_enumuse auxiliary/scanner/snmp/arris_dg950
use auxiliary/scanner/snmp/snmp_enum_hp_laserjet
use auxiliary/scanner/snmp/brocade_enumhash
use auxiliary/scanner/snmp/snmp_enumshares
use auxiliary/scanner/snmp/cambium_snmp_loot
use auxiliary/scanner/snmp/snmp_enumusers
use auxiliary/scanner/snmp/cisco_config_tftp
use auxiliary/scanner/snmp/snmp_login
use auxiliary/scanner/snmp/cisco_upload_file
use auxiliary/scanner/snmp/snmp_set
use auxiliary/scanner/snmp/netopia_enum
use auxiliary/scanner/snmp/ubee_ddw3611
use auxiliary/scanner/snmp/sbg6580_enum
use auxiliary/scanner/snmp/xerox_workcentre_enumusers
```

**其他内网安全审计工具(snmp):**
项目地址: `https://www.solarwinds.com/topics/snmp-scanner`
项目地址: `https://www.netscantools.com/nstpro_snmp.html`


**snmp for pl :**
```bash
wget http://www.cpan.org/modules/by-module/NetAddr/NetAddr-IP-4.078.tar.gz
tar xvzf ./NetAddr-IP-4.078.tar.gz
cd NetAddr-IP-4.078/
perl Makefile.PL
make
make install
./snmpbw.pl

```

  
#### <font color=red>基于ICMP发现内网存活主机  </font>
###### 1.nmap扫描
```bash
nmap ‐sP ‐PI 192.168.1.0/24 ‐T4

nmap ‐sn ‐PE ‐T4 192.168.1.0/24
```

###### 2.CMD下扫描
```cmd
for /L %P in (1,1,254) DO @ping ‐w 1 ‐n 1 192.168.1.%P | findstr "TTL="
```

###### 3.powershell扫描
```powershell
# powershell脚本  ./Invoke‐TSPingSweep.ps1
powershell.exe ‐exec bypass ‐Command "Import‐Module ./Invoke‐TSPingSwe
ep.ps1; Invoke‐TSPingSweep ‐StartAddress 192.168.1.1 ‐EndAddress 192.168.1.2
54 ‐ResolveHost ‐ScanPort ‐Port 445,135"


# 或
tcping.exe ‐n 1 192.168.1.0 80
```


  
####  <font color=red> 基于SMB发现内网存活主机</font>
###### 1.基于msf
```bash
# 模块 scanner/smb/smb_version

```

###### 2.基于cme(参考第九十三课)
```bash
cme smb 192.168.1.0/24
```

###### 3.基于nmap
```bash
nmap ‐sU ‐sS ‐‐script smb‐enum‐shares.nse ‐p 445 192.168.119
```

###### 4.基于CMD
```bash
for /l %a in (1,1,254) do start /min /low telnet 192.168.1.%a 445
```

###### 5.基于powershell
```powershell
## 一句话扫描:
#单IP:
445 | %{ echo ((new‐object Net.Sockets.TcpClient).Connect("192.168.1.119",$_)) "$_ is open"} 2>$null

#多ip:
1..5 | % { $a = $_; 445 | % {echo ((new‐objectNet.Sockets.TcpClient).Connect("192.168.1.$a",$_)) "Port $_ is open"}2>$null}

#多port,多IP:
118..119 | % { $a = $_; write‐host "‐‐‐‐‐‐"; write‐host "192.168.1.$a"; 80,445 | % {echo ((new‐object Net.Sockets.TcpClient).Connect("192.168.1.$a",$_)) "Port $_ is open"} 2>$null}
```


  
####  <font color=red> 基于MSF发现内网存活主机 </font>
###### 一
search搜索
```bash
search scanner type:auxiliary
```

主要介绍`scanner`下的五个模块,辅助发现内网存活主机,分别为:
```bash
auxiliary/scanner/discovery/arp_sweep    # 发现内网存活主机
auxiliary/scanner/discovery/udp_sweep    # 发现内网存活主机
auxiliary/scanner/ftp/ftp_version      # 发现FTP服务
auxiliary/scanner/http/http_version    # 发现HTTP服务
auxiliary/scanner/smb/smb_version      # 发现SMB服务
```


###### 二
主要介绍`scanner`下的五个模块,辅助发现内网存活主机,分别为:
```bash
auxiliary/scanner/ssh/ssh_version        # 发现SSH服务
auxiliary/scanner/telnet/telnet_version    # 发现TELNET服务
auxiliary/scanner/discovery/udp_probe    # 发现内网存活主机
auxiliary/scanner/dns/dns_amp            # 发现内网存活主机
auxiliary/scanner/mysql/mysql_version    # 发现mysql服务
```


###### 三
主要介绍`scanner`下的五个模块,辅助发现内网存活主机,分别为:
```bash
auxiliary/scanner/netbios/nbname    # 发现内网存活主机
auxiliary/scanner/http/title        # 发现内网存活主机
auxiliary/scanner/db2/db2_version    # 发现db2服务
auxiliary/scanner/portscan/ack    # 发现内网存活主机
auxiliary/scanner/portscan/tcp    # 发现内网存活主机
```



###### 四
主要介绍`scanner`下的五个模块,辅助发现内网存活主机,分别为:
```bash
auxiliary/scanner/portscan/syn    # 发现内网存活主机
auxiliary/scanner/portscan/ftpbounce    # 发现内网存活主机
auxiliary/scanner/portscan/xmas        # 发现内网存活主机
auxiliary/scanner/rdp/rdp_scanner        # 发现内网存活主机
auxiliary/scanner/smtp/smtp_version        # 发现内网存活主机
```



###### 五
主要介绍`scanner`下的三个模块,以及db_nmap辅助发现内网存活主机,分别为:
```bash
auxiliary/scanner/pop3/pop3_version        # 发现内网存活主机
auxiliary/scanner/postgres/postgres_version    # 发现内网存活主机
auxiliary/scanner/ftp/anonymous            # 发现内网存活主机

db_nmap            # 发现内网存活主机(MSF内置强大的端口扫描工具Nmap,为了更好的区别,内置命令为:db_nmap, 使用方法与nmap一致)
#如：
db_nmap ‐p 445 ‐T4 ‐sT 192.168.1.115‐120 ‐‐open
hosts        # 查看数据库中已发现的内网存活主机
hosts ‐h     # hosts命令也支持数据库中查询与搜索,方便快速对应目标存活主机
hosts ‐S 192    # 搜索关键字
```

hosts ‐h  查看参数：
```bash
‐a,‐‐add Add the hosts instead of searching
‐d,‐‐delete Delete the hosts instead of searching
‐c <col1,col2> Only show the given columns (see list below)
‐C <col1,col2> Only show the given columns until the next restart (se
e list below)
‐h,‐‐help Show this help information
‐u,‐‐up Only show hosts which are up
‐o <file> Send output to a file in csv format
‐O <column> Order rows by specified column number
‐R,‐‐rhosts Set RHOSTS from the results of the search
‐S,‐‐search Search string to filter by
‐i,‐‐info Change the info of a host
‐n,‐‐name Change the name of a host
‐m,‐‐comment Change the comment of a host
‐t,‐‐tag Add or specify a tag to a range of hosts
```




###### 六
主要介绍`post`下的六个模块,辅助发现内网存活主机,分别为:
```bash
windows/gather/arp_scanner
windows/gather/enum_ad_computers
windows/gather/enum_computers
windows/gather/enum_domain
windows/gather/enum_domains
windows/gather/enum_ad_user_comments
```

1.基于`windows/gather/arp_scanner`发现内网存活主机
```bash
meterpreter > run windows/gather/arp_scanner RHOSTS=192.168.1.110‐120 THREADS=20
```

2.基于`windows/gather/enum_ad_computers`发现域中存活主机
```bash
meterpreter > run windows/gather/enum_ad_computers
```

3.基于`windows/gather/enum_computers`发现域中存活主机
```bash
meterpreter > run windows/gather/enum_computers
```

4.基于`windows/gather/enum_domain`发现域中存活主机
```bash
meterpreter > run windows/gather/enum_domain
```

5.基于`windows/gather/enum_domains` 发现域中存活主机
```bash
meterpreter > run windows/gather/enum_domains
```

6.基于`windows/gather/enum_ad_user_comments`发现域中存活主机
```bash
meterpreter > run windows/gather/enum_ad_user_comments
```


`POST`下相关模块如下：
```bash
linux/gather/enum_network
linux/busybox/enum_hosts
windows/gather/enum_ad_users
windows/gather/enum_domain_tokens
windows/gather/enum_snmp
```

  


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%8F%91%E7%8E%B0%E5%86%85%E7%BD%91%E5%AD%98%E6%B4%BB%E4%B8%BB%E6%9C%BA%E6%96%B9%E6%B3%95-%E7%AE%80%E5%8D%95%E6%95%B4%E7%90%86/  

