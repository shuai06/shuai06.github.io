# ubuntu server-DNS服务




# DNS服务

>您是谁家老小



## <font color=red>域名解析原理</font>

DNS：将容易基于的主机名映射到IP地址
**计算机命名规范：**

- Netbiso名称（微软）
- Hostname
- DNS分布式名称系统(但是不同的dns服务器是有层级关系的，呈树状结构)

**DNS服务结构：**

- 域名          比如：sina.com.cn
- FQDN(fully qualified domain names )     比如:www.sina.com.cn  即主机名(www)+域名(sina.com.cn)

**DNS记录类型**

| 记录类型 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| NS       | 域名服务器记录(找a.com与里面的域名对应的ip的话，就要去问域名服务器,问他www的对应的主机的ip地址是多少)，一个域可以有多个域名服务器(`dig a.com ns`) |
| A        | 主机记录(域名解析到ip地址)                                   |
| CNAME    | 别名记录(给域名指向另一个A记录的域名,可以叠加)               |
| MX       | 邮件交换记录                                                 |
| PTR      | 指针记录（反向查询）                                         |
| SOA      | 起始授权记录(soa记录对应的域名服务器，才是合法的)            |



<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200405094807682_18444.png" />

**DNS命名空间**

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200405100714099_17972.png" />`www.ubuntu.com.`严格来说是有点的，但是为了方便，最后面不写点也可以

**DNS查询：** 逐级委派
由机构`LCANN`负责管理, 负责根域`.`



## <font color=red>DNS查询寻结构</font>

**三种DNS服务器：**

1. Master(DNS域里面中，master服务器可以修改DNS记录，在master上做得修改也会自动同步到其他slave等dns服务器) 一般只设置一台
2. Slave（数据文件来自master的同步）  -- 前两者两种服务器都会保存dns记录类型
3. Cache	（不保存任何记录，一般都是运营商提供的，跟下面的dns查询有关系，意义：本个区域内有缓存了减少dns会话的流量，提高了效率，减轻了带宽占用，超出生命周期之后才会再次查询）
   1. TTL
      4.Forward（转发类型的，比如会去转发给本地运营商的dns服务器(比如家用无线路由器会企业的域控)，而不是自己去进行递归查询啥的）
<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200405101258270_2676.png" />

同时是三种角色


##### 安装DNS服务

使用最广的DNS服务的软件包：`BIND（Berkley Internet Naming Daemon）`
在ubuntu上：

```bash
# 安装bind(即软件包) 9是版本，dnsutils是用来检查的软件包
sudo apt install bind9 dnsutils

# DNS服务区只保存和解析本域各种域名记录
# DNS服务都包含13个根域名服务器地址   
cat /etc/bind/db.root    # -> ipv4和ipv6地址(AAAA记录)
	# 事实分布于世界的数百台根域服务器，不只是13个（国内是有根域服务器的）
```

**DNS服务器的DNS服务器配置**
自己做迭代
指定递归域名服务器（自建尽量禁用其他地址来连接我们递归查询功能, 除非查询的是域内的域名）

```bash
# 见 DNS主备部署
```

**DNS默认服务端口**
TCP 53  / UDP 53

> 这里是以`Bind`软件包为例

**还有其他软件包：**

1. Djbdns
   - 衍生版本：Dbndns, ndjbsns
2. dhamasq
   - DNS + DHCP打包的轻量解决方案 --> 经常用于`无线渗透，搭建无线AP`
     3.PowerDNS
   - 模块化开源DNS服务器软件





## <font color=red>DNS主备部署</font>

对于dns服务器本身的dns地址可以执行运营商等的地址

### <font color=green>正向区域</font>

#### 配置`Master` DNS服务器

###### 执行区域文件（正向区域 --> 域名解析为ip）

>企业内部自用， 一般一个域中会有两个DNS服务器

1.

```bash
# sudo vim /etc/bind/named.conf.local

# zone定义一个区域， type类型，指定存具体记录的文件
zone "lab.com"{
	type master;
	file "/etc/bind/db.lab.com"
}
```

2.编辑区域文件

```bash
# 拷贝一个现有的区域文件模板，进行修改(也可手动创建)
cp db.local db.lab.com

sudo vim db.lab.com
# 文件内容
; BIND data file for lab.com loopback interface
;
$TTL	604800    # 允许缓存DNS服务器的缓存时长，单位s， 1D ->一天,12h -> 12小时
@	IN	SOA	soa.lab.com. root.localhost. (  # @表示本域的域名，   IN ->互联网记录类型， SOA起始授权记录; root.localhost.管理员邮箱地址(点就是@ ->root.lab.com.), 括号多行内容
			      2		; Serial    # 版本号，如果slave的与master服务器字段值相同则不同步数据，不同则同步数据
			 604800		; Refresh	# slave向master的更新周期
			  86400		; Retry     # 重试周期
			2419200		; Expire	# 最大重试时间
			 604800 )	; Negative Cache TTL
;
@	IN	NS	ns1		# NS记录，dns域的域名服务器的名称
@	IN	A	10.1.8.128	# 给哪个域名创建A记录，访问本机的域名解析到哪里
@	IN	AAAA	::1     # ipv6使用的
ns1	IN	A	10.1.8.10   # 给域的ns记录域名解析到哪个ip
soa	IN	A	10.1.8.10
www	IN	CNAME web       # 把www.lab.com解析为web.com.lab
web	IN	CNAME w3		# 有一个CNAME记录
w3	IN	A	  10.1.8.128 # 最终解析为A记录的ip
@	IN	MX 10 mx1     # 优先级，数字越小优先级越高
@	IN	MX 10 mx2    
mx1	IN	A	10.1.8.1   # 再把mx的域名解析到ip
mx2	IN	A	10.1.8.2
ftp IN	A 	10.1.8.128
ftp IN	A 	10.1.8.129  # 同一个a记录命名指向不同的ip，不区分优先级，而是轮询

```

3.重启服务

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

4.服务器配置检查

```bash
sudo named-checkconf   # 检查语法错误

sudo named-checkzone lab.com /etc/bind/db.lab.com  # 正向区域检查

sudo named-checkzone lab.com /etc/bind/db.lab.com  # 反向区域检查
```

5.客户端解析验证

```bash
# dig在win上没有
ping lab.com

dig lab.com ns @10.1.8.10  # 解析ns记录，@dns服务器

dig www.com @10.1.8.10

dig com a @10.1.8.100   # 不指定的话，默认就是a记录

dig lab.com mx @10.1.8.100
```

客户端解析验证

```bash
# nslookup在linux和win都有

nslookup
# 设置参数 
server 10.1.8.100    # dns服务器
set q=ns   # 查询记录类型
lab.com	# 查询目标

## Windows本机hi进行dns本地缓存， 而Linux本地是不做缓存的
ipconfig /displaydns   # 显示本地缓存的dns记录
ipconfig /flushdns     # 清空本地缓存的dns记录
```


#### 配置`Cache` DNS服务器

主配置文件`/etc/bind/named.conf`

```bash
# 只是一个入口文件
include "/etc/bind/named.conf.options";  #  规范: 【分号结尾】
include "/etc/bind/named.conf.local";    #
include "/etc/bind/named.conf.default-zones";
```

###### 全局转发DNS服务器

对于不属于我们域的解析转发给其他dns服务器  （我们内部的nds服务器不知道怎么解析之后要去找谁）
配置转发器的文件: ` vim /etc/bind/named.conf.options`
1.全局转发：

```bash
acl "local"{   # options 块之前
	10.1.8.0/24；	# 本地网段(为公司内部的地址提供服务)
}； 
options {
	recursion yes;   # recursion即递归查询(公网上的dns服务器一般是禁止递归查询的)
	allow-recursion { local; };  # 允许对我发起递归查询的地址范围
	listen-on { 10.0.2.53; };  # 指定侦听的地址
	forwarders {
		9.9.9.9;  # 全局性的转发器，转发给谁
		8.8.8.8;
	};
};
```

`sudo systemctl restart bind9.service`

2.局部转发

```bash
zone "sina.com.cn" {   # 创建一个域名
	type forward;
	forwarders {
		202.99.96.68;  #局部转发，针对某个域名转发给某一个ip
		202.106.0.20;
	};
};
```

`sudo systemctl restart bind9.service`

解析过之后，就有了缓存，在一定时间内就不会转发再进行请求了  





### <font color=green>反向区域</font>

#### 配置`Master DNS`服务器

###### （i反向区域: ip解析为域名 --> 反垃圾邮件）

反垃圾邮件：攻击者伪造域内邮件发送恶意文件；检查外部的邮件是否是从真正的邮件服务器发送过来的
1.

```bash
# sudo vim /etc/bind/named.conf.local

# zone定义一个区域， type类型，指定存具体记录的文件
zone "8.1.10.in-addr-arpa"{  # 反着写,这里一个网段的地址10.1.8就写为8.1.10
	type master;
	file "/etc/bind/db.10.1.8";
}
```

2.

```bash
# 拷贝一个模板文件
sudo cp /etc/bind/db.127 /etc/bind/db.10.1.8

# 重启服务
sudo systemctl restart bind9
sudo systemctl status bind9
```

3.

```bash
# 编辑反向区域配置文件
vim /etc/bind/db.10.1.8
# 文件内容
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@   IN  SOA ns.lab.com. root.lab.com. (
                  1     ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
              604800 )   ; Negative Cache TTL
;
@   IN  NS  ns.lab.com.   # 本地的ns记录
10   IN  PTR ns.lab.com.   # PTR 反向指针记录
145	 IN	PTR		mx1.lab.com	 #145即10.1.8.145
2	 IN	PTR		mx2.lab.com	
2	 IN	PTR		w3.lab.com	# 一个ip也可以解析为多个域名

```

4.检测一下反向解析是否正常

```bash
dig -x 10.1.8.2@10.1.8.2
```

#### 配置`Slave` DNS服务器

- 为例实现冗余容错(防止某台dns服务器宕机)，通常会为每个与安装多个Slave DNS服务器  
- 修改记录只能在Master DNS服务器(`一个域内只允许有一个`)上操作，通过`版本号`(Serial)通知Slave服务器同步  (所以。作为master的管理员，配置更新之后需要手动修改版本号的值)
- 安全考虑
- 服务器全局`禁止区域传输`（同步本域所有DNS记录）
- 只允许指定IP，指定区域的Slave服务器进行区域传输
- 区域数据同步使用TCP 53端口

1.

```bash
# dig进行区域传输获取信息  axfr(黑客最喜欢拿的)
dig @10.1.8.10 lab.com axfr

## 禁止区域传输：
# 全局配置文件   
# sudo vim named.conf.options
# 在option中添加：
allow-transfer { none; };

# 重启

# 禁用之后，不影响dig来查询某记录，只是拒绝一次性把记录拿走
```

2.继续修改master服务器配置

```bash
# sudo vim /etc/bind/named.conf.local
# 在正向/反向zone字段里面都要添加
allow-transfer { 10.0.2.54; };  # 允许10.0.2.54 -> 就是slave服务器
```

3.继续修改master服务器配置

```bash
# vim /etc/bind/db.lab.com
# 增加一条NS记录和A记录	
@	IN	NS	ns2
ns2	IN	A	10.1.8.20


# 编辑反向的域名解析记录
# vim db.10.1.8
20	IN	PTR	ns2.lab.com.


# 重启服务
```

4.安装slave dns服务器并配置

```bash
sudo vim /etc/bind/named.conf.options
# 内容
acl "local"{   # 
	10.1.8.0/24；	
}； 
options {
	recursion yes;   
	allow-recursion { local; };  
	listen-on { 10.0.8.20; };  # 侦听自己的
	allow-transfer { none; };
	
	forwarders {
		9.9.9.9;  
		8.8.8.8;
	};
	............
};

# 重启服务
```

5.slave dns服务器上创建区域

```bash
# sudo vim /etc/bind/named.conf.local

# 正向zone区域（和master上的对应好）
zone "lab.com"{
	type slave;
	file "db.lab.com";  # 不能使用绝对路径(区域文件不用手动创建)
	masters { 10.1.8.10; };
}
# 反向区域
zone "8.1.10.in-addr.arpa"{
	type slave;
	file "db.lab.com";  # 不能使用绝对路径
	masters { 10.1.8.10; };
}


# 重启服务


#区域文件db.lab.com不用手动创建
# 会自动从master上同步下来
# 位置在 /var/cache/bind/   而且文件是加了密的

# 查看slave dns服务器的情况
grep bind /var/log/syslog
```

**记录更新通知**
Slave服务器到达更新周期
客户端数据加密保存（不能直接修改）
Master服务器通知版本号

master服务器的dns配置更新之后，一定要手动修改版本号，然后重启

在区域配置文件配置（master服务器的dns配置更新之后会通知slave服务器）：

``` bash
# sudo vim /etc/bind/named.conf.local
# 正反向zone都要配置
also-notify { 10.1.8.20; };
```

























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-dns%E6%9C%8D%E5%8A%A1/  

