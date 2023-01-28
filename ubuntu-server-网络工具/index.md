# ubuntu server-网络工具




## 网络带宽

- 开测试防火墙性能NetIO
- 光纤专线的带宽是否缩水
- 实际承受的流量
- iperf3
- 模拟出来的流量并非真实带宽



## 测试网络带宽

#### <font color=red>iperf3</font>

- 支持TCP/SCTP/UDP协议

- 跨平台

- 多连接(模拟多用户)

- 支持IPv6

  

**安装**

```bash
# 服务端和客户端都是这一个软件包
apt install iperf3

# 默认侦听端口：5201
```



**参数**

```bash
# iperf3

  -p, --port      #         server port to listen on/connect to
  -f, --format    #[kmgKMG]  format to report: Kbits, Mbits, KBytes, MBytes
  -i, --interval  #         seconds between periodic bandwidth reports
  -F, --file name           # xmit/recv the specified file， 指定文件打到对方
  -A, --affinity n/n,m      set CPU affinity
  -B, --bind      <host>    bind to a specific interface
  -V, --verbose             more detailed output
  -J, --json                output in JSON format
  --logfile f               send output to a log file
  -d, --debug               emit debugging output
  -v, --version             show version information and quit
  -h, --help                show this message and quit
Server specific:
  -s, --server              run in server mode
  -D, --daemon              run the server as a daemon	# 后台运行
  -I, --pidfile file        write PID file
  -1, --one-off             handle one client connection then exit # 只接收第一个客户端的测试
Client specific:
  -c, --client    <host>    run in client mode, connecting to <host>
  -u, --udp                 #use UDP rather than TCP
  -b, --bandwidth #[KMG][/#] target bandwidth in bits/sec (0 for unlimited)  # 限制流量带宽
                            (default 1 Mbit/sec for UDP, unlimited for TCP)
                            (optional slash and packet count for burst mode)
  -t, --time      #         time in seconds to transmit for (default 10 secs) # 测试时长(默认10s)
  -n, --bytes     #[KMG]    number of bytes to transmit (instead of -t)
  -k, --blockcount #[KMG]   number of blocks (packets) to transmit (instead of -t or -n)
  -l, --len       #[KMG]    length of buffer to read or write
                            (default 128 KB for TCP, 8 KB for UDP)
  --cport         <port>    bind to a specific client port (TCP and UDP, default: ephemeral port)
  -P, --parallel  #         number of parallel client streams to run  # 并发数量
  -R, --reverse             run in reverse mode (server sends, client receives)
  -w, --window    #[KMG]    set window size / socket buffer size
  -C, --congestion <algo>   set TCP congestion control algorithm (Linux and FreeBSD only)
  -M, --set-mss   #         set TCP/SCTP maximum segment size (MTU - 40 bytes)
  -N, --no-delay            set TCP/SCTP no delay, disabling Nagle's Algorithm
  -4, --version4            only use IPv4
  -6, --version6            only use IPv6
  -S, --tos N               set the IP 'type of service'
  -L, --flowlabel N         set the IPv6 flow label (only supported on Linux)
  -Z, --zerocopy            use a 'zero copy' method of sending data
  -O, --omit N              omit the first n seconds
  -T, --title str           prefix every output line with this string
  --get-server-output       get results from server
  --udp-counters-64bit      use 64-bit counters in UDP test packets
  --no-fq-socket-pacing     disable fair-queuing based socket pacing

```



**使用**

```bash
# 作为服务端
iperf3 -s


# 作为客户端，连接服务端
iperf3 -c 192.168.0.111
```





## 查看当前带宽占用

#### 1.<font color=red>nload</font>

须在问题发生当下执行命令


```bash
# 安装
apt install nload

# 使用
sudo nload
nload -uk -U m enp0s3	# 查看指定网卡流量（如果不指定会自动查找网卡）
nload -m		# 同时查看多网卡流量
```



#### 2.<font color=red>iftop </font>

优点：会显示流量通信之间的主机双方

```bash
# 安装
apt install iftop

# 运行
sudo iftop
sudo iftop -n -N		# 不解析域名、端口（以ip地址形式显示通信双方）

# 其他参数
-p	# 混杂模式（凡是流经你网卡的流量都会被探测）
-i eth0		 # 指定网卡
```



#### 3.<font color=red>iptraf </font>


```bash
# 安装两个软件包
sudo apt install iptraf iptraf-ng

sudo iptraf-ng	# 字符菜单方式展示

# 选择，高亮显示，详细信息，配置...
```





#### 4. <font color=red>nethogs</font>

可显示每个进程所使用的带宽

```bash
# 产生大流量不一定是病毒

# 安装
sudo apt install nethogs

```



####  <font color=red>其他工具</font>

​	.... ...





























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E7%BD%91%E7%BB%9C%E5%B7%A5%E5%85%B7/  

