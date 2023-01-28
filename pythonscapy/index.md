# Python之Scapy




## 关于

Scapy是一个Python程序，使用户能够发送，嗅探和剖析并伪造网络数据包。此功能允许构建可以探测，扫描或攻击网络的工具。

换句话说，Scapy是一个功能强大的交互式数据包操作程序。它能够伪造或解码大量协议的数据包，通过线路发送，捕获它们，匹配请求和回复等等。Scapy可以轻松处理大多数经典任务，如扫描，跟踪路由，探测，单元测试，攻击或网络发现。它可以取代hping，arpspoof，arp-sk，arping，p0f甚至是Nmap，tcpdump和tshark的某些部分。Scapy主要做两件事：发送数据包和接收答案。

 在python中可以通过scapy这个库轻松实现构造数据包、发送数据包、分析数据包，为网络编程之利器！

项目地址：https://github.com/secdev/scapy



## 安装

Scapy 有两种使用方式：

1. 直接在shell中使用
```bash
# 先下载项目到本地
git clone git@github.com:secdev/scapy.git
# 运行
./run_scapy
# 交互式使用
```



2. 作为Python的第三方库使用
```python
# 安装
pip3 install scapy
# 导入
from scapy.all import *
```

   



## 使用

###### 构造数据包

```python
from scapy.all import *


# 2.构造数据包     IP()创建一个默认数据包
#  ls(IP()) 可以查看IP数据包可以有哪些参数。
# 其他数据包同理： TCP(),

pkt = IP(dst="114.114.114.114")

pkt.show()  # 查看数据包信息

pkt.summary()   # 方法查看概要信息

hexdump(pkt)    #  查看数据包的字节信息


# 使用 '/' 操作符来给数据包加上一层
# 例如构造一个TCP数据包，在IP层指明数据包的目的地址。在TCP层可以设定数据包的目的端口等等。UDP数据包同理。
tcpkt = IP(dst="114.114.114.114")/TCP()
tcpkt.show()

# 数据包的目标端口可以用范围来表示，发送的时候就会发送dport 不同的多个数据包
tcpkt = IP(dst="114.114.114.114")/TCP(dport=(22,33))
#  如果设置了多个参数为范围的，最后发送的数据包就是笛卡尔积。
tcpkt = IP(dst="114.114.114.114")/TCP(dport=(22,33), sport=(4567, 4568))
for tcp in tcpkt:
    print(tcp.dport, tcp.sport)


```



###### 发送数据包 (可能需要管理员权限)

```python
# 3.发送数据包
# 发送数据包可能需要管理员权限，使用sudo python3 进入python即可。
## scapy发送数据包有常用的如下几种方法：
"""
send(pkt)  发送三层数据包，但不会受到返回的结果(发完就拉倒)。

sr(pkt)  发送三层数据包，返回两个结果，分别是接收到响应的数据包和未收到响应的数据包。

sr1(pkt)  发送三层数据包，仅仅返回接收到响应的数据包。

sendp(pkt)  发送二层数据包。

srp(pkt)  发送二层数据包，并等待响应。

srp1(pkt)  发送第二层数据包，并返回响应的数据包

"""
# 例如：
ans, uans = sr(pkt)
print(ans)
print(uans)
```



###### 简单应用

```bash

# 4.应用
# 1）可以构造数据包来实现一个简单的 SYN端口扫描 ，flags="S" 表示发送SYN数据包
port_scan = IP(dst="192.168.0.121")/TCP(dport=[22,80,135,443,3306,3389], flags="S")
ans, uans = sr(port_scan)
ans.summary()
'''
端口返回的flag位为 SA，表示这些端口是开放的。
而 RA 表示reset ack， 说明这些端口是关闭的。
【见下面图片】
'''


#  2）实现一个基于TCP的traceroute
ans, uans = sr(IP(dst="114.114.114.114", ttl=(1,20), id=RandShort())/TCP(flags="0x2"))
for snd, rcv in ans:
    print(snd.ttl, rcv.src, isinstance((rcv.payload, TCP))



#  3) 模拟TCP的三次握手
source = IP(dst="192.168.0.121")/TCP(dport=22)
rsp = sr1(source)
source2 = IP(dst="192.168.0.121")/TCP(dport=22, flags="A", seq=rsp.ack, ack=rsp.seq+1)
rsp2 = sr1(source2)
print(rsp2.show())

```

<img src="http://image.xpshuai.cn/scapy_port_scan.png"></img>



###### 其他

```python

# 5.其他
# scapy 还可以用来读取网络流量包或监听网卡流量。
"""
1. 使用函数 rdpcap("/abc/def/xxxx.pcap")  可以读取包的内容，
2. 再使用  haslayer(TCP)  或  haslayer(ICMP)  等等来判断数据包的类型。
3. 使用  sniff(iface="who1",count=100,filter="tcp xxxx")  可以监听网卡流量，iface声明监听的网卡，filter是过滤条件，count是符合过滤条件的数据包的个数，达到指定的数据包个数后会停止监听，不设count则没有限制，按ctrl-c 结束监听。
4. sniff也支持无线网卡的监听模式。

"""
```



###### 抓包、分析包

```python
"""
prn指向一个回调函数，意为将收到的包丢给prn指向的函数处理（注意：回调的意义！每收到一个包就丢到回调函数里执行一下，执行完了才再跑回来继续抓包）

filter为包过滤规则（语法参照tcpdump过滤规则）

store为是否要存储抓到的包（注意，如果没有存储则不会将抓到的包赋值给a，因为没有存下就没有东西可以赋，此参数默认开启）

timeout为抓包时长，比如抓30秒就结束（注意：如果没有指定抓包时长则会一直抓下去，程序会一直卡在这里）

iface为指定抓包的网卡
"""

a = sniff(prn=abc, filter='tcp port 80 and ip 192.168.1.1', store=1, timeout=30, iface='eth0')   

# 此函数可以将抓到的包存到本地（注意：将包写入本地不能使用open（'packet.cap', 'r'）,因为open函数只能写入字符串）。
wrpcap('packet.cap', a)    
  
# 此函数可以将本地存储的数据包读取出来
bbb = rdpcap('/root/Desktop/ftp.cap') 
# 读取出来的对象是由N个数据包组成的可迭代对象，每次迭代一个包
for i in bbb:
    try: 
        # 输出数据包的应用层负载
        print(i.getlayer('Raw').fields['load'].decode().strip())   
    except :
        continue
```



先简单了解一下，剩下的后面慢慢学



> 更多用法见官方文档：https://scapy.readthedocs.io/en/latest/

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/pythonscapy/  

