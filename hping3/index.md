# hping3


> hping3主要是测试防火墙的拦截规则，对网络设备进行测试

#### 主要功能

- 防火墙测试（参考：http://0daysecurity.com/articles/hping3_examples.html）
- 端口扫描
- Idle扫描
- 拒绝服务攻击
- 文件传输
- 木马功能
- ......



#### **主要参数**

```bash
  -h  --help      显示帮助
  -v  --version   显示版本
  -c  --count     发送数据包的数目
  -i  --interval  发送数据包间隔的时间 (uX即X微秒, 例如： -i u1000)
      --fast      等同 -i u10000 (每秒10个包)
      --faster    等同 -i u1000 (每秒100个包)
      --flood      尽最快发送数据包，不显示回复。
  -n  --numeric   数字化输出，象征性输出主机地址。
  -q  --quiet     安静模式
  -I  --interface 网卡接口 (默认路由接口)
  -V  --verbose   详细模式
  -D  --debug     调试信息
  -z  --bind      绑定ctrl+z到ttl(默认为目的端口)
  -Z  --unbind    取消绑定ctrl+z键
      --beep      对于接收到的每个匹配数据包蜂鸣声提示

模式选择
  default mode     TCP   // 默认模式是 TCP
  -0  --rawip      RAWIP模式，原始IP模式。在此模式下HPING会发送带数据的IP头。即裸IP方式。使用RAWSOCKET方式。
  -1  --icmp       ICMP模式，此模式下HPING会发送IGMP应答报，你可以用--ICMPTYPE --ICMPCODE选项发送其他类型/模式的ICMP报文。
  -2  --udp        UDP 模式，缺省下，HPING会发送UDP报文到主机的0端口，你可以用--baseport --destport --keep选项指定其模式。
  -8  --scan       SCAN mode. //扫描模式 指定扫描对应的端口。
                   Example: hping --scan 1-30,70-90 -S www.target.host    // 扫描
  -9  --listen     listen mode  // 监听模式
  
IP 模式
  -a  --spoof      spoof source address  //源地址欺骗。伪造IP攻击，防火墙就不会记录你的真实IP了，当然回应的包你也接收不到了。
  --rand-dest      random destionation address mode. see the man. // 随机目的地址模式。详细使用 man 命令
  --rand-source    random source address mode. see the man.       // 随机源地址模式。详细使用 man 命令
  -t  --ttl        ttl (默认 64)  //修改 ttl 值
  -N  --id         id (默认 随机)  // hping 中的 ID 值，缺省为随机值
  -W  --winid      使用win* id字节顺序  //使用winid模式，针对不同的操作系统。UNIX ,WINDIWS的id回应不同的，这选项可以让你的ID回应和WINDOWS一样。
  -r  --rel        相对id字段(估计主机流量)  //更改ID的，可以让ID曾递减输出，详见HPING-HOWTO。
  -f  --frag       拆分数据包更多的frag.  (may pass weak acl)   //分段，可以测试对方或者交换机碎片处理能力，缺省16字节。
  -x  --morefrag   设置更多的分段标志    // 大量碎片，泪滴攻击。
  -y  --dontfrag   设置不分段标志    // 发送不可恢复的IP碎片，这可以让你了解更多的MTU PATH DISCOVERY。
  -g  --fragoff    set the fragment offset    // 设置断偏移。
  -m  --mtu        设置虚拟最大传输单元, implies --frag if packet size > mtu    // 设置虚拟MTU值，当大于mtu的时候分段。
  -o  --tos        type of service (default 0x00), try --tos help          // tos字段，缺省0x00，尽力而为？
  -G  --rroute     includes RECORD_ROUTE option and display the route buffer    // 记录IP路由，并显示路由缓冲。
  --lsrr           松散源路由并记录路由        // 松散源路由
  --ssrr           严格源路由并记录路由      // 严格源路由
  -H  --ipproto    设置IP协议字段，仅在RAW IP模式下使用   //在RAW IP模式里选择IP协议。设置ip协议域，仅在RAW ip模式使用。

ICMP 模式
  -C  --icmptype   icmp类型(默认echo请求)    // ICMP类型，缺省回显请求。
  -K  --icmpcode   icmp代号(默认0)     // ICMP代码。
      --force-icmp 发送所有icmp类型(默认仅发送支持的类型)    // 强制ICMP类型。
      --icmp-gw    设置ICMP重定向网关地址(默认0.0.0.0)    // ICMP重定向
      --icmp-ts    等同 --icmp --icmptype 13 (ICMP 时间戳)            // icmp时间戳
      --icmp-addr  等同 --icmp --icmptype 17 (ICMP 地址子网掩码)  // icmp子网地址
      --icmp-help  显示其他icmp选项帮助      // ICMP帮助

UDP/TCP 模式
  -s  --baseport   base source port             (default random)              // 缺省随机源端口
  -p  --destport   [+][+]<port> destination port(default 0) ctrl+z inc/dec    // 缺省随机源端口
  -k  --keep       keep still source port      // 保持源端口
  -w  --win        winsize (default 64)        // win的滑动窗口。windows发送字节(默认64)
  -O  --tcpoff     set fake tcp data offset     (instead of tcphdrlen / 4)    // 设置伪造tcp数据偏移量(取代tcp地址长度除4)
  -Q  --seqnum     shows only tcp sequence number        // 仅显示tcp序列号
  -b  --badcksum   (尝试)发送具有错误IP校验和数据包。许多系统将修复发送数据包的IP校验和。所以你会得到错误UDP/TCP校验和。
  -M  --setseq     设置TCP序列号 
  -L  --setack     设置TCP的ack   ------------------------------------- (不是 TCP 的 ACK 标志位)
  -F  --fin        set FIN flag
  -S  --syn        set SYN flag
  -R  --rst        set RST flag
  -P  --push       set PUSH flag
  -A  --ack        set ACK flag   ------------------------------------- （设置 TCP 的 ACK 标志 位）
  -U  --urg        set URG flag      // 一大堆IP抱头的设置。
  -X  --xmas       set X unused flag (0x40)
  -Y  --ymas       set Y unused flag (0x80)
  --tcpexitcode    使用last tcp-> th_flags作为退出码
  --tcp-mss        启用具有给定值的TCP MSS选项
  --tcp-timestamp  启用TCP时间戳选项来猜测HZ/uptime

Common //通用设置
  -d  --data       data size    (default is 0)    // 发送数据包大小，缺省是0。
  -E  --file       文件数据
  -e  --sign       添加“签名”
  -j  --dump       转储为十六进制数据包
  -J  --print      转储为可打印字符
  -B  --safe       启用“安全”协议
  -u  --end        告诉你什么时候--file达到EOF并防止倒回
  -T  --traceroute traceroute模式(等同使用 --bind 且--ttl 1)
  --tr-stop        在traceroute模式下收到第一个不是ICMP时退出
  --tr-keep-ttl    保持源TTL固定，仅用于监视一跳
  --tr-no-rtt       不要在跟踪路由模式下计算/显示RTT信息 ARS包描述（新增功能，不稳定）
ARS packet description (new, unstable)
  --apd-send       发送APD描述数据包(参见docs / APD.txt)
```



**常用模式**

```bash
-0 --rawip 	# IP原始报文
-1 --icmp	# ICMP模式
-2 --udp 	# UDP模式
-8 --scan	# 扫描模式
-9 --listen	# 监听模式

```



#### 常用方式

```bash
## SYN方式扫描主机端口
hping --scan 1-30,70-90 -S www.target.host
hping --scan 445,135 -S 192.168.0.110
#目标主机回复的 'S..A'，代表SYN/ACK


## 测试防火墙对ICMP包的反应、是否支持traceroute、是否开放某个端口、对防火墙进行拒绝服务攻击(DoS attack)。
#例如，以LandAttack方 式测试目标防火墙(Land Attack是将发送源地址设置为与目标地址相同，诱使目标机与自己不停地建立连接)。
# -a 伪造源地址
hping3 -S -a 114.114.114.114 -p 53 114.114.114.114 -c


## DRDDOS(UDP反射DDoS攻击)  -c发包的数量
hping3 --udp -a 114.114.114.114 -p 53 114.114.114.114 -c 5
```



使用hping3进行flood DoS攻击：

```bash
hping3 -c 5000 -d 150 -S -w 64 -p 80 --flood <10.0.40.246>
-c：发送数据包的个数
-d：每个数据包的大小
-S：发送SYN数据包
-w：TCP window大小
-p：目标端口，你可以指定任意端口
--flood：尽可能快的发送数据包
--rand-source :使用随机的Source IP Addresses.

```























---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/hping3/  

