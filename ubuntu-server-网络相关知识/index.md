# ubuntu server-网络相关知识




  

## 网络原理

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200214100233476_240785919.png" />

`ping 127.0.0.1` 如果能通，说明本机的TCP/IP协议已经配置成功啦
`arping `

  

## 网卡配置
###### 网卡接口  
-enp2s0 (最近使用)、eth0   （以前这么使用）

```bash
#wl 开头的是无线网卡

# ifconfig 命令
# 查看所有网卡
ifconfig -a
    字段意义：MTU,发包，收包，`collisions`发生多少次冲突

# ip 命令
 ip link     # 查看所有网卡和信息
 ip address

# lshw   查看系统硬件命令
# 只查看网卡的命令
sudo lshw -class network

```

###### 管理网卡
```bash
# 查看网卡参数
sud ethtool enp0s3
  
# -s   配置 ： speed 速度 半双工 
sud ethtool -s duplex half | full speed 1000

```

###### 网络基本设置  
```bash
## 临时设置IP地址
sudo ifconfig eth0 10.0.0.00 netmask 255.255.255.0
# 24位掩码
sudo ifconfig eth0 10.0.0.100/24
# 设置网关 gw
sudo route add default gw 10.0.0.1 eth0
# 网段路由 add-net   （静态路由：去哪个网段，应该怎么转发）
sudo route add-net 0.0.0.0 netmask 0.0.0.0 gw 1.1.1.1
# 主机路由 add-host    
sudo route add-host 2.2.2.2 gw 1.1.1.1
  


# 清除网卡配置
ip addr fulsh eth0

# 向网络中要一个IP
sudo dhcpclient

# 禁用网卡
sudo ifconfig eth0 down
# 启用网卡
sudo ifconfig eth0 up
# 重启网络服务
sudo systemctrl restart networking.service

```



###### 网卡配置文件
`/etc/network/interfaces`

```bash
文件字段：
iface 网卡name inet 网络配置（loopback/dhcp/static）

    sudo eth0
    iface eth0 inet static|DHCP

    # pre-up 系统启动中(网卡运行前)  设置为1000M，全双工
    pre-up/sbin/ethtool -s eth0 speed 1000 duplex full

```


###### 动态获取IP地址
```bash
# sudo vim /etc/network/interfaces
auto eth0
iface inet dhcp

```


###### 静态IP地址
一般服务器是静态的IP
```bash
# sudo vim /etc/network/interfaces
auto eth0
iface inet static
    address 192.168.123.9
    netmask 255.255.255.0        # 掩码
    gateway 192.168.123.1        # 网关
    dns-nameservers 192.168.123.1 202.99.96.68   # 一般dns地址写俩
    # 上面是基本配置, 下面是额外其他配置项
    broadcast 192.168.123.255   # 网段的广播地址
    # dns-search  abc.com  # 如果属于某个dns域,就默认去找这个
    up route add -net 172.16.0.0/24  gw 192.168.23.1 eth0   # 开机之后,添加一个网段(要指定具体网关)
    mtu 1460   # 指定MTU 字节
    # hwaddress 00:11:22:33:44:55   # 指定硬件mac地址
    
# 刷新,重启网络, 配置文件的网络生效(实在不行就重启计算机)
sudo ip address fulsh eth0 
sudo systemctrl restart networking.service

```

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20200214114820323_1273216811.png"></img>
指定路由
清除信息
ip addr flush ens32


> Note： VmWare对网络的支持要好于VirtualBox(比如有些网络的配置可能有限制)



###### Ubuntu 1804配置静态ip

ubuntu18.04 server，启用了新的网络工具`netplan`，对于命令行配置网络参数跟之前的版本有比较大的差别，现在介绍如下：

1. 其网络配置文件是放在`/etc/netplan/50-cloud-init.yaml`, 缺省是用dhcp方式，如果要配置静态地址，则需要修改此文件的想关内容，见如下的例子：

```bash
network:
	version: 2
    ethernets:
        ens33:
            addresses: [192.168.1.20/24]	# 可以是这种数组形式[1.1.1.1, 192.168.1.20/24]也可以是下面那样
            dhcp4: false
            gateway4: 192.168.1.1
            nameservers:
                addresses: [192.168.1.1]
                optional: true
            
```

或者下面这个格式(可能更清晰，有`ipv6`)

```bash
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
# 前几项必填，其他根据需要填写
network:
    version: 2
    ethernets:
        eth0:
            addresses:
                - 公共ipv4地址/20掩码
                - 私有ipv4地址/16
                - 公共ipv6地址/64
            dhcp4: false
            gateway4: "公共ipv4网关"
            gateway6: 公共ipv6网关
            match:
                macaddress: MAC地址(不必须)
            nameservers:	# DNS
                addresses:
                    - "67.207.67.2"
                    - "67.207.67.3"
            set-name: eth0
```

  

2.使其生效的方法：

```bash
sudo netplan apply
```

  

如果配置有问题会报错，如果没问题，则会新的配置会立即生效。

> 注意：此方法只是针对ubuntu18.04 Server版，对于18.04 desktop它缺省是使用NetworkManger来进行管理，可使用图形界面进行配置，其网络配置文件是保存在：/etc/NetworkManager/system-connections目录下的，跟Server版区别还是比较大的。

netplan 工具还有其它比较丰富的功能，详细可参见其的说明文档。



  

## DNS配置
###### DNS配置文件  
`/etc/resolve.conf`      

```bash
/etc/resolve.conf    # DNS设置（软连接）
格式:
    nameserver 114.114.114.114
    nameserver 127.0.0.53
    options edns0
    search www.tendawifi.com

```


###### 主机名解析
1.配置文件 `/etc/hosts`    优先级别`高`于resolve.conf里面的dns服务器的信息  

```bash
# ip 和名称做解析
127.0.0.1 localhost
192.168.0.111 www.qq.com
# 现在这里找映射，有的话就不去resolve里面找了

```

2.配置文件 `/etc/nsswitch.conf`        名称解析顺序配置文件  

```bash
# 按照优先级先后排序来解析  （前面的不满足，才调用后面的）
- files        /etc/hosts
- Resolve        全称systemd-resolved.service(缓存、localhost、本机名)   （默认安装）
- [NOTFOUND=return]        结果即权威  （正则表达式）  （当找不给我到的之后，返回）
- dns                    dns服务器（resolve文件配置的那些）
- mdns4_minimal            MulticastDNS(多播dns服务)     现在很多dns厂家用的就是多播dns服务

# 
hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname

```

> 关于解析优先级：有的攻击者会修改你hosts文件映射到自己的恶意网址，此时如果把dns的顺序放到hosts前面，他就利用不成了

###### 查看路由  
```bash
# 路由信息
route -n

netstat -nr

```

  

## 网桥配置
>把服务器当交换机用实在是大材小用
把多个以太网段以上层透明的方式连接在一起

**应用场景： ** 
1.可做防火墙  
2.可以做虚拟机的服务器（搭建云平台，进行虚拟化的时候）
3.桥接有线网与无线网（比如我的笔记本：连接网线访问互联网，另外让别人通过我的电脑无线上网，相当于一个AP）
4.链路荣誉容错（需启用STP）
5.通过网桥管理工具实现 `bridge-utils`  

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20200218132001375_1018333933.png"></img>



###### 在ubunut搭建网桥  
在虚拟机添加2块网卡(一块NAT，另一块随便)  

**安装网桥管理包： ** 
`sudo apt install bridge-utils ` 

**临时配置网桥： ** 
命令： `btctl ` 

```bash
sudo btctl addr br0     # 添加一个网桥的网卡接口
sudo btctl addif br0 eth0 eth1  # 往网桥接口添加2块网卡
# sudo ifconfig eth0 down   # 把网卡down掉
sudo ifconfig eth0 0.0.0.0 up    # 2者的ip变没了
sudo ifconfig eth1 0.0.0.0 up
sudo ifconfig br0 1.1.1.1/24 up   # 手动配置网桥网卡的ip并up
sudo dhclient br0    # 或者 dhcp自动获取网桥网卡的ip地址
sudo route add default gw 1.1.1.10    # 添加一个网关，就可以访问外网了
```

**持久配置网桥：**
修改配置文件： `/etc/network/interfaces`  

```bash
auto eth0
iface eth0 inet manual

auto eth1
iface eth1 inet manual

auto br0
iface br0 inet dhcp   # dncp  ， 也可以static，下面手动配置address，netmask和gateway
bridge_ports eth0 eth1  # 加入网卡
bridge_stop off   #关闭生成树 ,也可以 on开启成树，参与生成树计算

```
修改保存，然后 重启网络服务/重启服务器  

**查看网桥运行状态 ** 
`sudo brctl show`
**查看mac情况**  
`sudo brctl showmacs br0 `
**查看生成树情况**  
`sudo brctl showstp br0 `

  

## 网卡绑定   (经常使用)  
> 还记得小时候这段一把筷子的故事吗

实现多块网卡的绑定，实现高可用、负载均衡、更大发挥能力

###### 一些常用称呼：
Bonding == Port Trunking == Link aggregation(链路聚合)  == Team

###### 目的：  
将多个物理网卡组合为一个逻辑网卡
- 高可用、负载平衡、高吞吐量


#### 配置：    

在虚拟机做实验：先给虚拟机添加至少两块网卡    

1.第一步  

```bash
sudo echo bonding >> /etc/modules        # 添加内核支持
sudo modprobe bonding       # 手动加载内核(临时)， modeprobe加载bonding
ifconfig -a    # 可以看到自动生成了一个bonding0的网卡(但是没有up)
sudo systemctl stop networking     
# 永久配置，文件
# sudo vim /etc/network/interfaces
bounding    # 添加一个模块
# 重启服务
sudo systemctl restart networking     
  
```
2.配置网卡配置文件    

```bash
# Mode 1 配置
# sudo vim /etc/network/interfaces
auto eth0 
iface eth0 inet manual
bond-master  bond0    # 绑定到哪个bound上
bond-primary  eth0    # 默认做的主的网卡

auto eth01
iface eth01 inet manual
bond-master  bond0      # 备的网卡，协商要绑定那个


auto bond0
iface bond0 inet dhcp   # 让ip地址dhcp； 或者static 手动制定
bond-mode   1    # 绑定模式   active-backup
bond-miimon 100    # 故障检测间隔： 100ms
bond-slaves none         # 指定作为的的成员的网卡, 这里写none是因为：加入在自己物理网卡片段里面写了
  
--
# 重启服务
sudo systemctl restart networking  
  

```

PS: 绑定模式  
bound-mode  

```
1     一个主，一个备(主的故障后才工作)  也叫： active-backup
```


#### 6种模式  
###### Mode 0  ： round-robin(轮询)  
- 网络流量（数据包） 顺序平均分配给Bond中所有物理网卡（可以同时使用三个网卡发送和接收数据）
- 高可用（容错，一个坏了，另一个网卡接管）、负载均衡（多个网卡，单独是，就是拥有了多G的带宽）

  

###### Mode 1 : ative-backup  
- Bond中只有一个网卡Active， 其他网卡全部Stanby（效率并不是很高）  
- 对外只有一个网卡的MAC地址可见  
- 高可用  
实现不了负载均衡，有多个接口也没用  


###### Mode 2:  balance-XOR
- 根据源目的MAC/IP/Port进行计算，确定从哪个网卡发出（性能优于Mode0）
- 高可用，负载均衡

（XOR --- > 异或）  
只要不出现故障，不改变mac地址，第一次使用的第一个物理网卡进行的通信，以后所有的通信都是继续使用第一个物理网卡
流量不是平均的  


###### Mode 3   broadcast  
- 发包广播给Bond中所有网卡，提供最短的故障恢复时间，应用连接不中断  
- 高可用  


###### Mode 4   802.3ad（Dynamic link aggregation）
- 链路聚合LACP组内的网卡使用相同速率、双工设置    
- 要求：计算机安装ethtool；交换机支持IEEE802.3ad标准，并进行额外配置
- 高可用、负载均衡  


###### Mode 5    balance-tlb （Adaptive transmit load balancing）  
- 隧道绑定不需要上连交换机额外配置，根据网卡负载出站负载均衡
- 高可用、负载均衡  

  


###### Mode 6:   balance-alb (Adaptive load balance)  
- Mode 5 + balance-rlb (入站流量负载均衡)  
    - Bond驱动拦截本机的ARP响应包，使用不同网卡硬件MAC替换源MAC
    - 不同的对端使用不同的服务器MAC地址，实现入站负载均衡
 - 不需要上连交换机额外配置  

  

###### 查看bond端口和状态  
```
cat /proc/net/bonding/bond0
```

###### Model 4 配置  
```bash
# sudo vim /etc/network/interfaces
auto eth0     # 其他物理网卡配置项同
iface eth0 inet manual
bond-master  boud0    # 绑定到哪个bound上

auto bon
d0
iface bond0 inet dhcp   # 让ip地址dhcp； 或者static 手动制定
bond-mode 4    # 绑定模式   active-backup
bond-miimon 100    # 故障检测间隔： 100ms
bond-lacp-rate 1         # 每1s发送LACPDU（默认为0，即30s）
bond-slaves eth0 eth1         # 指定作为的的成员的网卡


```



  

## DHCP服务  
通常给客户端自动分配IP地址  

###### Dynamic Host Configuration Protocol
- 透明的配置网络参数  
- IP/掩码、网关、DNS、域名、时间服务器、打印服务器(常用的参数是前三个)
- 通过地址租约循环使用IP地址（有租约期，循环利用）
- 基于UDP， 标准使用 端口： 67(服务端)，68(客户端)

ps: 一般企业，dhcp都在核心交换机上，只有当企业网络非常大复杂时候，才会单独使用一个服务器来提供dhcp服务  

**请求和分配步骤： ** 
1.客户端发生第一个广播(二层，因为此时还没有ip)，数据包：`dhcp discover`  
2.dhcp服务器回复数据包`dhcp offer ` 
3.客户端就再发一个广播包`dhcp request ` 
4.服务器端发送数据包`ack`， 包含ip等参数，标记这个ip为已分配  


###### 安装  
1.在vmware上，打开虚拟网络设置，以vmnet8为例，关闭vmware自带的dhcp服务（如果同时有两台dhcp服务:使用响应快的服务提供的ip），将这台虚拟机使用nat，  
让其他虚拟机都使用这台虚拟机的dhcp服务，把这个虚拟机修改为静态ip, 修改配置文件`/etc/network/inertface`到vmnet8的网段，`sudo ip addr fulsh eth0`清除原来配置，重启网络服务  

2.安装dhcp服务的软件包  

```bash
# 这里用的isc的  
sudo apt install isc-dhcp-server  
```

3.配置dhcp服务  

>Note：能不安装的服务和软件就不要安装，反非必须，就不启用（缩小攻击面）  


```bash
##1.先修改配置文件1  
#sudo vim /etc/default/isc-dhcp-server   # 指定启动DHCP服务的网卡
    INTERFACE="eth0"   # 要启动shcp的网卡name(一般是针对内网的那一块网卡)，可以多个网卡（空格隔开）
    
##2.再修改主配置文件（指定地址池和选项） 
#sudo vim /etc/dhcp/dhcpd.conf       
#option只对dhcp的作用域起效；其他的很对整个服务器起效
#一致的配置写在文件起始位置，其他的个性的单独设置在各个网段的位置
default-lease-time 600;  #租约期限， 单位：s
max-lease-time 7200;  #最大租约期限(到达时间的一半，就刷新使用期；如果到达最大时间，就重新通过dhcp获取)
authoritative;   #启用之后，证明这个是权威的授权dhcp服务器（一般情况下，网络中如果出现其他的dhcp就不会影响了）
#配置地址池(可以手动写，也可以在原注释的基础上配置) 网段
subnet 10.1.8.0 netmask 255.255.255.0 {
    range 10.1.8.100 10.1.8.200;   #可分配的地址段
    option routers 10.1.8.1;  #网关地址
    option domain-name-servers 192.168.1.1,192.168.1.2;  #dns服务器地址，多个用逗号隔开; 一般配到这里就够了
    option ntp-server 1.1.1.1;     #时间服务器
    option domain-name "local.lan";   #统一的域名
}
  
##3.重启服务
  

#客户端一些操作
ipconfig /all
ipconfig /release   # 释放这个ip
ipconfig /renew    #重新获取ip
  
```



3.特殊情况(指定某个ip只分配给某台计算机，IP保留)    

>通过MAC地址来识别  
```bash
#sudo vim /etc/dhcp/dhcpd.conf       
  
host yourname {
    hardware ethernet 00:11:22:33:44:55;
    fixed-address 10.1.8.8;   #尽量用上面地址池以外的地址
}  
#保存，重启
  
```

###### 日常的维护  
日志与状态查询  
```bash
#服务器地址租约结果（可查看分发详细信息）
cat /var/lib/dhcp/dhcpd.leases
#系统日志文件(客户端直接关机不会记录，release的时候会记录)
tail -f /var/log/syslog
#查看服务运行状态
systemctl status isc-dhcp-server.service
#客户端来查看获得地址历史(客户端是Linux时)
less /var/lib/dhcp/dhcpd.leases

```



​    




## NTP服务 （网络时间协议）
>时间都去哪了

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20200308150835487_32007.png"></img>

**时间表准： ** 

- GMT：格林威治标准时间
- UTC：世界协调时间
- CST：China Standard Time UT+8:00

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200308153026228_153.png"></img>

#### NTP协议  
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200308154004613_29225.png"></img>

###### NTP客户端  
- 客户端程序从时间服务器同步时间
- 系统启动时自动同步时间
- 网口激活时自动同步运行
- 手动同步时间  

```bash
# 新版系统，查看客户端时间  
timedatectl  
  
# RTC时间：硬件时间  
```
新版系统使用`timesyncd`客户端同步时间  
向这个域名请求：ntp.ubuntu.com  

```bash

timedatectrl list-timezones   #列出所有时区  
timedatectrl set-timezone    #设置时区
timedatectrl set-time "2019-10-01 18:18:18" #设置系统时间  

timedatectl set-ntp true  #开始网络同步
systemctl status systemd-timesyncd.service  # 查看时间服务状态
  
# 硬件时间(RTC)与UTC时间同步
sudo hwclock -w   # 将系统时间写入到RTC时间
sudo hwlock -s     # 将硬件
hwclock --set --date='2019-10-01 18:18:18'
  
# 配置文件
/etc/systemd/timesyncd.conf
```


###### 一个过时的东西(ntpdate)  
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200308170606501_6440.png"></img>
新版本系统使用`timesyncd`替换`ntpd`的客户端功能
一旦安装了ntpdate / ntp,   timedatectrl将被禁用


###### ntpd -- 客户端+服务器  
>把机器设置成时间服务器  
```bash

apt install ntp  # 安装ntp服务
systemctl start ntp  # 启动服务(123端口就被打开了)
systemctl status ntp  # 查询服务状态
systemctl restart ntp  # 重启服务

##配置文件（我也要跟上一级的时间服务器同步）
vim /etc/ntp.conf
#如果不使用官方的时间服务器，可以自己指定
server 1.1.1.1  
#首选网络时间服务器不可用的时候，使用本机始终作为备用时间源
fudge 127.127.1.1 startum 10  # 层级设置低点( startum 10, 这里设置为10层)
```

配置文件中，主要的时间服务器如下  
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200308171832987_9157.png"></img>

  

###### 命令: ntpq -- ntp服务端的查询命令  
```bash
ntpq -p     
#命令执行结果详解如下图：  
```
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200308172254102_12604.png"></img>
每行前面第一个字符含义：

```
空  表示无效主机  
x   已不再使用
- 已不再使用
#   状态良好但为使用
+   良好且优先使用
*   首选主同步主机    （一般是startum层级最高的）
```


###### 其他命令  
```bash
#查看日期
date  
#设置
date --set 1998-11-11  #手动设置日期（要通过 sudo timedatectl set-ntp 0 来关掉ntp）
date --set 21:21:21    #设置时间

cat /etc/timezone      #查看时区
  
```





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86/  

