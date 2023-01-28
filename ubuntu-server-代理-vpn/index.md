# ubuntu server-代理&VPN






>你看见的不是你真的看见的

- 时间、带宽、管控都影响网络使用体验
- 作为改善技术，VPN/代理和突破限制改善体验
- 区别：代理主要提高访问`速度`；VPN更注重`安全`性
- 在某些应用场景中，作用非常相似
- 作为中间层，双向模拟（翻墙）

【访问者】 --> 【中间层】 --> 【目标】


## <font>代理(Proxy)</font>
#### 网络代理服务
- 客户端访问请求发给Proxy，而非直连目标
- 代理转发请求给目标，并将结果返回给客户

#### 正向代理
提高速度、内容控制、安全
（代替正常用户来访问资源）

#### 反向代理
速度、安全、分发
（代替一个代理服务器接收资源，转发给后面的代理服务器，然后把后面代理服务器资源分发给用户。比如Nginx反向代理）
安全性：反向代理服务器上可以进行安全性过滤，比如waf等


#### Squid
>全面的一个代理软件
###### 用squid实现正向代理
1.安装squid
```bash
sudo apt install squid
```

2.配置
```bash
vim /etc/squid/squid.conf    # 主配置文件

# acl：访问控制列表/资源列表 
acl localnet src 192.168.0.0/16    # 先配置内部网段，指定源地址，只给这个网段做代理

acl menhu111 dstdomain .sina.com .sina.com.cn .163.com    # 目标域名，多个以空格分隔。小心页面元素CSS、图片(有些资源不属于那个域的下面)
acl xiaban111 time MTWHFAS 18:00-23:59    # 定义时间，下班时间，MTWHFAS（周1-7），不限制访问。是服务器本地时间
# Mon, Tues, Wednes, tHurs, Fri, sAtur, Sun, D: 工作日
acl noexes url_regex -i \.exe$    # URL以exe结尾（-i大小写不敏感）
acl httpsurls url_regex -i ^https  # 以 ^URL起始
acl noporn url_regex -i sex    # 
acl zipfiles url_regex -i \.zip$


# http访问规则顺序匹配（按前后顺序执行）
# 原则：【禁止规则】一定要放在【允许规则】前面
http_access deny !Safe_ports    # 不是Safe_ports的端口，都禁止
http_access deny noexes noporn   # 原则：【禁止规则】一定要放在【允许规则】前面
http_access deny CONNECT !SSL_ports  # 如果不是SSL_ports的端口，如果发起Connect请求的话，也是禁止的
http_access allow localhost    # 允许所有人请求我代理服务器的本地
http_access allow localnet menhu111 xiaban111    # 允许本地地址访问后面指定的目标域名
http_access allow localhost manager    # 缓存到代理服务器本地
http_access deny manager
http_access deny all    # 除了上面明确允许的，其他的一概禁止(这条规则要放到最后面)
# http_access allow all
```
3.修改完之后，重新加载配置文件
```bash
sudo kill -SIGHUP `cat /var/run/squid.pid`

```


默认侦听3128代理端口，在配置文件可以修改
```bash
http_port 3128
# 默认侦听所有网卡的所有3128端口,建议不要对公网开放代理
# 指定侦听网卡
http_port 192.168.0.1:3128

```

访问日志
```bash
access_log daemon:/var/log/squid/access.log squid
```

缓存
```bash
cache_dir ufs /var/spool/squid3 100 16 256    # 缓存路径，多大，周期
refresh_pattern -i \.(gif|png|jpg|jpeg|ico)$ 3600 90% 86400    # 缓存图片
```




## <font>VPN</font>
#### 原理
###### 企业内部系统不应对外开放
- 出差在外的企业员工需要访问公司内部系统
- 分公司、合作商需要安全的访问指定企业内部系统
- 专线网络费用高昂且不灵活
- 远程访问安全性欠缺

###### 虚拟专用网(VPN -- Virtual Private Network)
- 基于并不安全的网络线路，通过加密技术构建安全的信息通道（tunnel）
- VPN用户获得如同在内网一样的访问使用体验


###### 与Proxy相比
- VPN更加安全、成本高、部署难度大
- VPN一旦部署完成后简单使用
- Proxy属于一种中间层，VPN更接近路由（对应用透明，无需设置代理）
- VPN更完善的认证、加密、授权控制

###### VPN类型
- PPTP
- L2TP（用的少）
- IPSec（保密性要求很高，过于复杂。常用于网络边界之间）
- SSL（非常适合终端用户）


OPenVPN
- 全特性的开源SSL VPN
- 跨平台 Windows、Linux、macox、ios、android



#### OpenVPN
- 通过非受信网络连接上网时保证安全
- 出差员工访问企业内部系统
- 伪装为国外地址，规避地理限制和审核（翻墙）
- 需要（独立的）证书颁发机构CA，完成PKI架构


###### 安装&配置
>一台VPN服务器，一台CA服务器（每个服务器只做一个用途）

题外话：ubuntu server 1804修改主机名之前，先把`/etc/cloud/cloud.conf`配置文件中修改为的`preserve_hostname`修改为true，然后再修改hostname和hosts


1.安装OpenVPN（在VPN Server）
```bash
sudo apt install openvpn
```

2.安装EasyRSA(在VPN Server+ 在CA Server)
```bash
# ubuntu软件源也有，但是不是最新的，这里去github下载最新的
# ca server 和vpn server都需要下载EasyRSA(一个作为客户端一个作为服务端)
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.6/EasyRSA-unix-v3.0.6.tgz
tar -xvf EasyRSA-unix-v3.0.6.tgz
```

3.配置安装（在CA服务器上）
```bash
cd ~/EasyRSA-v3.0.6/
cp vars.example vars
#vim vars
    set_var EASYRSA_REQ_COUNTRY  "CN"
    set_var EASYRSA_REQ_PROVICE  "BJ"
    set_var EASYRSA_REQ_CITY     "BJ"
    set_var EASYRSA_REQ_ORG       LAB""
    set_var EASYRSA_REQ_EMAIL     "admin@lab.com"
    set_var EASYRSA_REQ_OU        "IT"

# 初始化PKI架构
./easyrsa init-pki
# 会创建一个pki的目录，以后生成的证书会在这个目录下，证书的私钥会在这个目录下的privte下，证书申请文件会在这个目录下的reqs下
```

4.创建证书颁发机构，生成CA公私钥（在CA Server）
```bash
./easyrsa build-ca nopass    #证书私钥文件的访问密码，但是这里不设置
# 下面设置证书的名字，默认不改
# ca.crt    公钥证书，用于验证CA前面的其他证书（C/S都需要）
# ca.key    CA的私钥，用于前面所有C/S证书（应私密保存）
# 证书服务器在不使用时，建议关机
```

5.生成VPN服务器私钥和证书申请文件（在VPN Server）
```bash
cd EasyRSA-v3.0.6/
./easyrsa init-pki    # 只是初始化PKI架构
./easyrsa gen-req vpnserver nopass    # 生成证书申请文件，起个名字，设置密码
# pki/reqs/vpnserver.req
# pki/private/vpnsenver.key    # 私钥文件要好好保存
sudo cp pki/private/vpnserver.key  /etc/openvpn    # 拷贝私钥文件

# 将证书请求文件传输至CA签发
scp pki/reqs/vpnserver.req  xps@CA_IP:/tmp
```

6.导入并签发VPN服务器证书（在CA Server）
```bash
./easyrsa import-req /tmp/vpnserver.req vpnserver    # 导入，加一个名字
./easyrsa sign-req vpnserver vpnserver    # 签发， pki/issued/vpnserver.crt
# 证书类型  server / client

## 将证书回传至vpnserver
scp pki/issued/vpnserver.crt xps@vpnserver_IP:/tmp/    # vpnserver证书
scp pki/ca.crt xps@vpnserver_IP:/tmp/    # CA公钥证书（自建的证书，要想被信任，必须获得ca服务器的公钥）

```

7.拷贝证书文件（在vpnserver）
```bash
sudo cp /tmp/{vpnserver.crt, sa.crt}  /etc/openvpn

# 生成Diffie-Hellman密钥（密钥交换）
./easyrsa gen-dh    # 生成HMAC签名（TLS完整性校验）   dh.pem 
openvpn --genkey  --secret ta.key    # 生成安全密钥
sudo cp ta.key /etc/openvpn/
sudo cp pki/dh.pem  /etc/openvpn/

# 服务端证书设置全部完成

```

8.客户端证书（在VPN Server）
```bash
# 客户端也要证书来证明自己的身份
## 在 VPN服务器上生成客户端证书
## 使用脚本批量自动生成客户端配置文件
## 生成客户端证书密钥 / 请求文件

mkdir -p ~/client-configs/keys    # 证书文件目录
chmod -R 700 ~/client-configd     # 配置文件目录
cd ~/EasyRSA-v3.0.6/
./easyrsa gen-req client_1 nopass    # client_1
cp pki/private/client_1.key  ~/client-configs/keys/
scp pki/reqs/client_1.req xps@CA_IP:/tmp/    # 证书的请求文件

```

9.导入并签发证书（在CA Server）
```bash
./easyrsa import-req /tmp/client_1.key client_1   # 导入
./easyrsa sign-req client client_1              # 签发（证书类型，证书column name）    
scp pki/issued/client_1.crt xps@vpnserver_IP:/tmp/
```

10.复制证书文件（在VPN Server）
```bash
cp /tmp/client_1.crt  ~/client-configs/keys/
cp ~/EasyRSA-v3.0.6/ta.key  ~/client-configs/keys/
sudo cp /etc/openvpn/ca.crt ~/client-configs/keys/
```

**客户端证书文件准备完毕，下面步骤都是在VPN服务器上配置**

11.vpn服务器配置文件
```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz  /etc/openvpn/
sudo gzip -d /etc/openvpn/server.conf.gz
sudo vim /etc/openvpn/server.conf    # 配置文件，根据自己的情况来修改
    tls-auth ta.key 0
    key-direction 0
    cipher AES-256-CBC
    auth SHA256
    dh dh.pem
    user nobody
    group nogroup
    ; 下面，强制客户端【所有】流量路由到tun0（我的vpn隧道）
    push "redirect-gateway def1 bypass-dhcp"    
    push "dhcp-option DNS 202.106.8.20"
    push "dhcp-option DNS 8.8.8.8"    ; DNS服务器可设多个
    ;端口和协议
    port 443
    proto tcp    ；如果用TCP的1194，就无需这条
    explicit-exit-notify 0
    ;指定服务器证书文件
    cert vpnserver.crt
    key vpnserver.key
    
```

12.启动服务器路由功能
```bash
sudo vim /etc/sysctl.conf
    net.opv4.ip_forward = 1
sudo sysctl -p
```

13.确认默认路由网卡名称
```bash
ip route |grep default
    default via 192.168.8.2  dev enp0s3 proto static
```

14.修改防火墙规则实现NAT
```bash
# 连上我的vpn之后，都把我的地址转换为路由绑定网卡的内网的地址，隐藏原本的地址
sudo vim /etc/ufw/before.rules    # 优先级高于普通UFW规则
    # START OPENVPN RULES
    *nat
    :POSTROUTING ACCEPT [0:0]
    -A POSTROUTING -s 10.8.0.0/8 -o enp0s3 -j MASQUERADE    # -s 源地址
    COMMIT
    # END OPENVPN RULES
```

15.开启防火墙  
```bash
sudo vim /etc/degault/ufw    # 防火墙默认未开启转发包，这里开启转发包
    DEFAULT_FORWARD_POLICY="ACCEPT"    

# 启动防火墙及规则
sudo ufw allow 1194/udp   # vpn的端口允许
sudo ufw allow OpenSSH    # 否则ssh连接不上了
sudo ufw disable
sudo ufw enable

# 启动OpenVPN服务
sudo systemctl start openvpn@server    # server.conf 的名称一致
sudo systemctl status openvpn@server
sudo systemctl enable openvpn@server
ip addr show tun0    # 发现多了一个网卡tun0
```


###### 客户端配置
1.配置文件模板（用户只需傻瓜式操作即可，所有证书提前申请好，其他的用脚本方式执行）
```bash
mkdir -p ~/client-configs/files    # 配置文件存放的目录
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf  ~/client-configs/base.conf

# Linux客户端有这个文件：  /etc/openvpn/update-resolv-conf 

vim ~/client-configs/base.conf
    # script-security 2        # 下面三行，仅仅针对Linux客户端(才取消注释)
    # up /etc/openvpn/update-resolv-conf
    # down /etc/openvpn/update-resolv-conf
    remote vpnserver_IP 1194
    proto udp
    user nobody   
    group nogroup     # Centos是nobody
    # ca ca.crt
    # cert client.key
    # key client.key
    # tls-auth ta.key 1
    cipher AES-265-CBC
    auth SHA256
    key-direction 1

```

2.客户端配置脚本
```bash
vim ~/client-configs/make_config.sh    # 脚本参数为客户端标识名
    #!bin/bash
    KEY_DIR=~/client-configs/keys
    OUTPUT_DIR=~/client-configs/files
    BASE_CONFIG=~/client-configs/base.conf
    cat ${BASE_CONFIG} \
    <(echo-e '<ca>')\
    ${KEY_ DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>')\
    S{KEY_ DIR}/${1}.crt \        # ${1} 是脚本运行时的参数
    <(echo -e '</cert>\n<key>')\
    S{KEY_ DIR/${1}.key\
    <(echo e '</key>\n<tls-auth>')I
    ${KEY_DIR}/ta.key\
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_ DIR}/${1}.ovpn


chmod 700 ~/client-configs/make_config.sh
cd ~/client-configs
# 为每个用户申请证书
sudo ./make_config.sh client_1
    # 生成client_1.ovpn

# 分别为每个用户端生成相应证书，然后运行脚本生成配置文件
# 将.ovpn配置文件拷贝到客户单

# 如果是Linux客户端，要把配置文件client.ovpn中下面三行取消注释
    script-security 2       
    up /etc/openvpn/update-resolv-conf
    down /etc/openvpn/update-resolv-conf
    
# 如果是Centos系统，配置文件client.ovpn要修改
group nobody
```


3.客户端安装
```bash
## Linux客户端安装
sudo apt install openvpn

## Linux客户端
vim client.ovpn
    script-security 2      
    up /etc/openvpn/update-resolv-conf
    down /etc/openvpn/update-resolv-conf

```

4.建立VPN连接
```bash
## 命令行方式
sudo openvpn --config client_1.ovpn > /dev/null 2>&1 &

## ubuntu图形化方式
# 先安装openvpn的gnome小插件
sudo apt install metwork-manager-openvpn
sudo apt install metwork-manager-openvpn-gnome
sudo systemctl restart network-manager
设置，网络，VPN，添加，从文件导入ovpn文件


## 其他VPN客户端
Cisco Concentrator    # network-manager-vpnc
Cisco OpenConnect     # network-manager-openconnect
PPTP(Microsoft VPN)   # network-manager-pptp
stringSwan(for some IPsec VPNs)    # network-manager-strongswan

```

5.其他平台客户端
```bash
## Windows客户端
https://openvpn.net/index.php/open-source/downloads.html
# C:\Program Files\OpenVPN\config\client2.ovpn
# 要以管理员身份运行

##IOS 
App Store搜索OpenVPN Connect

## Android
OpenVPN Connect

## MacOS
https://tunnelblick.net/downloads. html
安装结束自动提示导入客户端配置文件

```

###### 撤销客户端证书
1.CA Server
```bash
cd EasyRSA-3.0.4/
./easyrsa revoke client2
./easyrsa gen-crl    # 生成证书吊销列表CRL(crl.pem)
scp pki/crl.pem xps@vpnserver_IP:/tmp
```

2.VPN Server
```bash
# 在vpn server调用
sudo cp /tmp/crl.pem /etc/openvpn
sudo vi /etc/openvpn/server.conf
    crl-verify crl.pem    # 最后一行添加吊销指令
sudo systemctl restart openvpn@server

```










---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E4%BB%A3%E7%90%86-vpn/  

