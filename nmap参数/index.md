# nmap参数简介



  
### 新学一个抓包工具 ->   tcpump

```
tcpdmp -i eth0 -nn host 192.168.1.56 port 22 

# 用nc连
nc 192.168.1.56 -t 22

# 点.  就是ack的意思

```




## 主机发现
  
```
-iL  # 指定文件
-iR 100 -p22 # 随机选择目标100个 的22端口 
nmap 192.168.0.0/24 --exclude  192.168.0.1-100  # 不扫描后面的部分

-sL  # 列出ip（相当于子网掩码的计算）
-sN  # 只ping
-Pn   #不做ping扫描，例如针对防火墙等安全产品， 防止：如果没有回包就不进行了    
-PS/PA/PU/PY  # 用syn，ack，udp， sctp 发现
-PE/PP/PM  #  用echo， 时间戳，子网掩码 （一般没啥结果）
-PO   # ip协议的ping
-n/ -R    # 不做dns解析  / 做反向的解析
--dns-servers   # 调用指定的dns服务器
--system-dns   # 操作系统默认的dns，一般不用
--traceroute    # 过程中进行raceroute
```

## 端口发现

```
-sS/sT/sA/sW/sM   # 默认的  syn扫描/tcp扫描,建立完整连接(最准确)/发送ack / 窗口扫描 / ack+fn
-sU   # udp扫描（准确性不怎么高）
-sN/sF/sX    # tcp的flag全为空 /只有fin/ fin,psh,urg
--scanflag   # 自定义flag   （自己得懂原理）
-sI    # 僵尸扫描
-sY/sZ   # sctp用  
-sO   # ip扫描
-b   # ip中继
-
-p   # 指定端口(tcp与udp都扫),  U: 53  -T:21   (分别指定tcp与udp的端口)   默认1000个
--exclude-ports     # 指定端口中某一段
-F   # 快速扫描(不会扫描100个全部端口)
-r   # 连续扫描(顺序)
--top-ports   # 扫排名前10个
--port-radio   # 扫描更常见的端口(用处不大)
```

## 服务扫描

```
-sV       # 扫描这个端口跑的什么服务
--version-intensity   # 0-9的扫描程度，9最详细
--version-light     # 上面的等于2时
--version-all       # 上面的等于9时
--version-trace     # 将扫描过程进行跟踪
```


## 脚本扫描  （脚本目录:/usr/share/nmap/scripts/）

```
-sC      # 默认
--script=...   # 指定脚本
--script-args=    # 带参数
--script-args-file=    
--script-trace    # 也可以trace
--script-updatedb    # 更新脚本  nmap --script-updatedb
--script-help    # 查看脚本的帮助
```


## 操作系统类型

```
-O   # 操作系统类型
--osscan-limit   # 限制检测的系统类型
--osscan-guess
```


## 时间和性能

```
-T   # 指定间隔时间
-min-hostgroup/ -max-hostgroup   # 最少一次扫描的主机数量 组
--min-parallelism/ --max-parallelism  # 最少一次扫描的主机数量
--min-rtt-timeout/ --max-rtt-timeout/ --initial-rtt-timeout   # 最小最大rtt

--max-retries    #
--host-timeout  # 超时时间
--scan-delay/ --max-scan-delay 10s   # 延时时间(防止被发现)
--min-rate    # 发包最小速率
--max-rate
```


## 防火墙 和躲避

```
-f  # 最大传输mtu值    1500啥的
-D 192.168.0.2  192.168.0.3  192.168.0.4  # 伪造原地址(迷惑)
-S  # 伪造源地址
-e  #  指定网卡
-g  #  指定使用的源端口
--proxies <url1, [url2]>  # 置地广代理服务器替我们发包 
--data  # 数据字段指定内容(必须是16进制)
--data-string
--data-length
--ip-options
--ttl
--spoof-mac   # 欺骗mac地址
--badsum   # 发错的差错校验值
```


## 输出格式


```
-oN / -oX / -oS /-oG      # 正常格式/xml格式/../../

```

## 杂项

```
-6  # ipv6的
-A  #  几个参数的组合（涵盖：-v, -O, --script, --trace）  # trace路由追踪
```






























---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/nmap%E5%8F%82%E6%95%B0/  

