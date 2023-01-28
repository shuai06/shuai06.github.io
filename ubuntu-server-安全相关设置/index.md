# ubuntu server-安全相关设置



# 142-安全设置

Linux默认安装已经很安全，当很多的了解安全仍非常必要
打造一个相当安全的系统（增加入侵者的成本，安全也是要考虑成本的）

大部分的安全事件来自于内部员工和已经离职的员工

<img src="http://image.xpshuai.cn/20200519144331629_1148039304.png" />


#### 防护思路
- 风险评估：攻击面
- 制定策略：最小化、明确需求
- 安全管理：安全是管理
- 应急预案：突发事件
- 威胁情报：了解安全行业

#### 常见错误理解
- 安全是安全从业者的事情
- 为了安装简单，装所有的包，开所有的服务(版本号、banner信息)
- 有了边界防火墙则不需要主机防火墙
- 随意开放各种访问方式(后门)
- 公司内部网络是安全的


## <font color=red>帐号管理</font>
**用户管理是系统安全最重要的方面**
- 密码策略
- 权限控制

**Ubuntu默认禁用root帐号**
使用不匹配任何加密值的密码，无法直接登陆(sudo)
设置密码之后即可启用root登录
安装过程创建的用户帐号默认属于sudo组
```bash
sudo passwd  # 设置root密码(还是不建议使用root登录系统)
sudo passwd -l root  # 禁用root帐号密码(用完之后解锁: -u) sudo passwd -u root 
```

**增加管理员帐号**
```bash
sudo usermod -aG sudo xps    # 加入sudo组
sudo viudo    # 本质是编辑 /etc/sudoers
sudo su
# 尽量减少管理员帐号数量
# 离职手续办完之前，要把他的帐号禁用然后逐渐删除
```

**多用户环境下，用户主目录默认权限是全局可读**
```bash
ls -ld /home/username
# 手动修改权限(不建议-R，会有一些使用的问题)
sudo chmod 0750 /home/username
# 预先修改配置（新添加的帐号权限都确定了）
sudo vi /etc/adduser.conf
    DIR_MODE=0750
# 再添加用户，主目录权限就是我们规定的了
sudo adduser username
```


**密码复杂度**
```bash
# 使用sudo设置密码时 不受密码规则限制
# 编辑配置文件
sudo vi /etc/pam.d/common-password
    password    [success=1 default=ignore]  pam_unix.so obscure sha512 minlen=8 lcredit=1 ucredit=1 dcredit=2 ocredit=1

# minlen 最小长度
# lcredit 最少包含的小写字母个数
# ucredit 最少包含的大写字母个数
# dcredit 最少包含的数字个数
# ocredit 最少包含的其他字符个数
```

**密码过期**
```bash
sudo chage -l username    # 查看用户帐号
# 设置密码过期时间
sudo chage -E 01/31/2015 -m 5 -M 90 -I 30 -W 14 username
# -E  密码过期时间
# -m / -M     密码使用期限
# -I    密码过期后多少天仍可以改密码，过期锁定帐号
# -W    过期前多少天警告
```

**帐号SSH登录**
应用和服务使用其他身份认证机制，需要单独配置帐号安全
锁定帐号无法是无法组织公钥认证的SSH远程登录用户
```bash
# 删除 .ssh目录
~/.ssh/authorized_keys
```

禁用帐号不会强制断开`已经建立`的SSH连接
```bash
who |grep username    # 查询用户 pt/num 终端
sudo pkill -f pts/num    # 强制踢掉
```

指定可SSH远程登录的用户
```bash
sudo vi /etc/ssh/sshd_config    # 需重启服务生效
    AllowGroups sshlogin    # 设置允许的组

# 创建ssh登录组，创建属于sshlogin组的用户
sudo addgroup sshlogin && sudo adduser username sshlogin
```


**控制台安全**
```bash
# 一般，启用机器后，到了登录界面，不用登录即可进行远程连接，其他服务也尅启动了
# 避免误触，建议禁用ctrl+Alt+Delete 快捷键重启
sudo systemctl mask ctrl-alt-del.target
sudo systemctl daemon-reload
```



## <font color=red>系统信息</font>
**当前登录帐号**
```bash
# 查看登录会话
w

# TTY1 是本地登录，pts/0 是远程终端
# 字段： 用户-终端-来源-登录时间-发呆时间(空闲)---当前正在运行的程序
xps@xps:~$ w
 08:51:23 up  1:15,  3 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
xps      tty1     -                08:49    1:47   0.09s  0.04s -bash
xps      pts/0    192.168.0.115    07:37    1.00s  0.23s  0.01s w
xps      pts/1    192.168.0.115    08:49    3.00s  0.07s  0.03s vim 1.txt
```

**历史登录信息**
```bash
last -d    # 从下往上看

#xps      pts/0        bogon            Tue May 19 07:37   still logged in
#reboot   system boot  4.15.0-99-generi Tue May 19 07:36   still running

# 系统启动之后，就会显示reboot的一条记录，显示内核信息
```

**lsof命令**
Linux中一切皆文件，运行中的文件成为进程
目录中被打开的文件
```bash
sudo lsof +D /home/xps

lsof |grep /var/log/   #查看/var/log/下的文件被哪些进程打开
```

用户打开的所有文件
```bash
sudo lsof -u xps
sudo lsof -u ^xps    # 除了xps账户之外

lsof -u 0  	#查看uid为0的用户打开的文件（uid为0代表是root）
```

结束指定用户的所有进程
```bash
sudo kill -p `lsof -t -u xps`    # -t提取pid值
```

列出所有的网络连接
```bash
sudo lsof -i    # 类似 ss 命令

lsof -i:22	#查看22端口都有哪些进程在访问
lsof -p 1168	#查看1168进程号所打开的文件
lsof -c sshd #查看sshd服务打开的文件
```

其他
```bash
## 利用lsof恢复已删除文件(日志被删除后可以这样)
lsof |grep /var/log/messages #以meaages日志为例,查看状态，记住编号id
cd /proc/xx_刚才的id/fd
ls -l
```

**suid /sgid的安全隐患**
```bash
# suid 在在user的x位置多了个S。在用户执行该程序时,用户的权限度是该程序文件属主的权限。
# sgid与suid类似，只是执行程序时获得的是文件属组的权限。

# 如果suid的属主为root，那任何人都可以root权限执行命令，危害太大

# 找启用了suid的文件
sudo find / -type f -perm -u=s -ls        # 如果有不应该有suid文件，需要注意
# 找启用了guid的文件
sudo find / -type f -perm -g=s -ls    
# 如果员工离职，那员工的账户没被清理，被新人所沿用，就可以读取之前的文件-->因为系统靠uid识别
# 检查无属主和无属组的，根据需要保存和清理(如：离职员工帐号被删除，的文件)
find -xdev \(-nouser -o nogroup \) -print    


# 题外话：
当t出现在其他组的x权限位置时，表示其他组具有SBIT的权限。
SBIT（Sticky Bit）目前只针对目录有效，对于目录的作用是：当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除。 
最具有代表的就是/tmp目录，任何人都可以在/tmp内增加、修改文件（因为权限全是rwx），但仅有该文件/目录建立者与 root能够删除自己的目录或文件。

```




## <font color=red>文件加密</font>
**网络本身不加密**
- 通过不受信网络连接上网
- 加密信息传输通道是首选方案(VPN)
- 传输单个文件可使用文件级加密

**很多应用也不加密**
传输机密信息使用SSH、SCP

**GnuPG**
```bash
# 基于文件的加密。免费开源
# Linux默认安装
# 斯诺登推荐


### 使用

# 1.生成密钥对(公钥，私钥)，发送方和接收方都要生成
gpg --gen-key
#输入name,email,密码文(保护密钥)
#会自动进行系统信息搜集生成随机数据(越随机越安全，如果半天没响应，执行下面的命令)
#生成随机操作 熵(促使)
sudo find / -type f|xargs grep somerandomstring> /dev/null

# 2.公钥交换:接收者先把公钥发给发送者(这里使用原始的方式)
#2.1 接收者导出公钥文件
gpg --export user2_name > user2_name.pub
#2.2 公钥文件拷贝给发送者
scp  mygpg.pub xps1@192.168.0.11:~/

# 3.发送者执行
#3.1查看当前都有哪些密钥
gpg --list-keys
#3.2发送者导入接收者的公钥
gpg --import user2_name.pub
#查看当前都有哪些密钥
gpg --list-keys

# 4.发送者开始加密文件（-recipient 是接收者的名字）
gpg --encrypt --recipient zhangsan secert.txt    # secert.txt是要加密的文件

#5.加密文件发送给接收者
#接收者解密(使用私钥)
gpg --output result.txt --decrypt secret.txt.gpg
#输入私钥的加密密码串


## 其他命令
#查看当前都有哪些密钥
gpg --list-keys
#删除某人的公钥
gpg --delete-keys zhangsan
#删除自己的私钥
gpg --delete-secret-keys xps

## 通过密钥服务器交换密钥(指定密钥服务器， 后面16进制0x---是id值的后8位)
gpg --keyserver certserver.pgp.com --send-key 0xAABBCCDD    # 我发送
gpg --keyserver certserver.pgp.com --recv-key 0xAABBCCDD    # 对方获取
#但是这种方式在国内受限
```

GnuPG大致工作流程：

<img src="http://image.xpshuai.cn/20200519173953416_754675779.png" />





## <font color=red>防火墙</font>
>访问控制设备

**Linux内核包含`Netfilter`子系统**
- 用于操作经过服务器的流量
- 实现包管理
- 用户空间管理工具`iptables`(功能强大，但是命令比较复杂)


**Ubuntu默认的防火墙管理工具`UFW`**
- 操作简单，默认禁用
- 功能不全面
- 主要用作`主机防火墙`(本身就是目标，对自己加防护)


#### UFW
>适合做主机防火墙，如果做网络防火墙不是很好

###### 基本使用
启用
```bash
sudo ufw enable    # 注意：会有提示:"可能会导致现有的连接断开,比如22"-->有默认策略
sudo ufw disable
```

查看状态
```bash
# 规则的顺序很重要(如果中间匹配了，就不往下匹配规则了)

sudo ufw status
sudo ufw status verbose    # 详细展示: Logging: on (low)日志详细级别  Default: allow (incoming), allow (outgoing), deny (routed) -->入站的默认允许，出战的默认禁止，若作为网络防火墙(路由)默认是禁用的
sudo ufw status numbered    # 列出序号
```

默认规则
```bash
sudo ufw default allow|deny|reject DIRECTION(incoming/outigoing/routed)

## 两个拒绝的区别
# deny 无视，也没有响应，不告诉发起者(推荐这个，更安全，防扫描)
# reject 会给发起者一个响应，告诉他
```

**添加策略**
添加规则
```bash
sudo ufw allow 22    # 不指定协议类型时，ctp和udp都开
sudo ufw allow 22/tcp   # 最小化原则，只开tcp的22
sudo ufw deny 22

# ipv4的默认在上面, ipv6的默认在下面
```

设置方向(in ,out)
```bash
sudo ufw allow in http    # 默认设置入站规则
sudo ufw reject out smtp
```

规则描述
```bash
# 规则太多了记不清
sudo ufw reject telnet comment 'telnet is unencrypted'
# 以后不用的规则就可以方面清理删除（安全管理意识很重要 --> 详细文档很重要，统一管理平台:工单）
```

插入规则
```bash
# 指定新规则的位置是2，后面的向后顺延
sudo ufw insert 2 allow 80
```


删除规则
```bash
# 按照id序号删除
sudo ufw delete 3
# 按照 添加时候的规则删除
sudo ufw delete allow http
```

定向规则
```bash
# 5个：源ip，目的ip，源端口(很少做限制)，目的端口，通信协议

# 在2的位置插入，允许 proto协议为tcp，从1.1.1.10来的到1.1.1.9的22端口
sudo ufw insert 2 allow proto tcp from 1.1.1.10 to any port 22 
sudo ufw insert 2 allow proto tcp from 1.1.1.10 to 1.1.1.9 port 22 

sudo ufw insert 2 allow proto tcp from 192.168.0.0/24 to any port 22 # 使用于只允许公司内部访问
sudo ufw insert 2 allow proto tcp from 192.168.0.0/24 to 192.168.0.102 port 22 

sudo ufw deny proto tcp from 2001:db8::/32 to any port 25   # IPv6
    # /etc/default/ufw   启用/禁用IPv6规则(IPV6=YES)
# 其他协议：ah, esp, gre, ipv6, igmp
```

端口名
```bash
# 前面 http 等是与端口对应的，不是随便起的，配置文件如下
/etc/services
# 可以自定义
```

规则
```bash
sudo ufw deny in on eth0 to 224.0.0. proto igmp    # 指定网卡（拒绝eth0入的方向使用igmp协议访问组播地址）
sudo ufw allow in on eth0 to 224.0.0. proto gre

sudo ufw allow proto tcp from any to any port 80,443,8080:8090 comment 'web'  # 一段端口范围->用冒号

sudo ufw allow in on eth0 to any port 80 proto tcp
```


**开启路由模式**
```bash
# 启用路由功能
sudo vi /etc/ufw/sysctl.conf
    netipv4/ip_forward=1
    net/ipv6/conf/default/forwarding=1
    net/ipv6/conf/all/forwarding=1

# 
sudo sysctl -p
ufw disable
ufw enable
```

路由模式规则
```bash
# 启用路由功能(转发数据包)，先添加规则再开启路由也可
# 7个：源网卡接口，目的网卡接口，源ip，目的ip，源端口(很少做限制)，目的端口，通信协议
sudo ufw route allow in on eth1 out on eth2  # eth1网卡in的流量， eth2出的流量
sudo ufw route allow in on eth1 out on eth2 to 192.167.11 port 80 proto tcp
```


限制连接频率
```bash
# 尽量规避暴力破解
ufw limit ssh/tcp    # 30s超过6次就deny
```

###### 安全相关
UFW作为网络防火墙(先开启路由转发)

**IP遮盖**
```bash
# sudo vi /etc/default/ufw
    DEFAULT_FORWARD_POLICY="ACCEPT"

# sudo vi /etc/ufw/sysctl.conf
    net/ipv4/ip_forward=1    # 启用路由功能

# sudo vi /etc/ufw/before.rules
    *nat            # 开启nat地址转换
    ：POSTROUTING ACCEPT [0:0]    # 接收路由转发
    -A POSTROUTING -s 10.8.0.0/8 -o enp0s3 -j MASQUERADE    # 添加规则：制定来源内网网段，指定另一个有公网ip的的网卡然后进行地址转换并发出去
    COMMIT    # 
```

完整语法规则
```bash
sudo ufw route allow/reject/deny/limit insert/delete/pretend in on eth0 out on eth1 from 1.1.1.1 oport 1:65535 to 2.2.2.2 port 22 proto tcp/udp/igmp/esp/ah
# 如果不指定规则id位置，就默认在后面依次；pretend是依次从前面添加
```

小技巧 -- 应用程序
```bash
# 为非标准端口的应用定义名称
# 为每个开放端口的服务程序分别定义程序配置文件  /etc/ufw/applications.d/

sudo ufw allow from 192.168.0.0/24 to any app CUPS
sudo ufw app into CUPS
    [<name>]
    title=<title>
    description=<description>
    portd=<ports>
    ports=12/udp|32|56,78:90/tcp    # 竖线隔开

# 应用防火墙规则
sudo ufw app update <name>

# 适用情景：
非标准业务的服务端口，一个应用同时绑定多个端口，业务描述也可以添加进去，修改也方便
```


###### 日志功能
>UFW默认不记录日志，因为占用存储空间

日志的价值在于：识别攻击、规则排错、诊断问题

```bash
# 全局开启日志，以及级别 / 关闭日志
sudo ufw logging on|off|LEVEL(LOW/medium/high/full)

# 存储位置：/var/log/ufw.log

# 单独为每条规则添加日志
sudo ufw allow log 22/tcp
```


###### 恢复默认规则
```bash
# 系统全新安装的状态(实在搞不清才使用，会提前备份->/etc/ufw/日期命名的文件)
sudo ufw reset
```


#### Shorewall
适合任何网络的高级防火墙
有图形管理界面

因为iptables语法有点麻烦，不想使用




## <font color=red> AppArmor</font>
#### 简介
**AppArmor是一个Linux安全模块**
- 使不安全和不受信进程在受限的约束下运行
- 保证进程访问共享文件、行使特权、与其他进程通信
- 针对某一个进程层面
- 界面美观，操作较简单


**全局强制访问控制机制(MAC)**
- 基于LSM的额外安全功能
- 系统调用前，内核查询AppArmor策略，确定程序是否有权执行操作
- 不同与SELinux(过于庞大, 针对操作系统层面)，与用户无关，可用于限制超级用户权限（所有用户继承）
- 将进程限制在有限的系统资源之上
- 预设的控制机制，一定程度可预防未知安全漏洞（0day）


**Ubuntu默认的安全系统**
默认已安装并加载


**控制策略**
- 基于安装路径识别程序并控制
- 策略保存于相应配置文件(profile)中
- 集合高级静态分析和基于文件`/etc/apparmor.d`
- 部分安装包自带配置文件


#### 使用
**安装额外配置文件**
```bash
sudo apt install apparmor-profiles apparmor-profiles-extra
# apparmor-profiles        # 由上游社区管理维护的配置文件
# apparmor-profiles-extra    # 由ubuntu、debian开发的配置文件
```
安装用户端管理工具
```bash
sudo apt install apparmor-utils
```

查看状态
```bash
sudo aa-status
```

控制模式
```bash
# enforcing      强制策略实施并记录被禁用请求
# complaining    只记录被禁请求，但不强制实施禁止   -->策略调试时候用

## 切换模式
aa-enforce    # 切换为强制模式
aa-complain   # 切换为包容模式
aa-disable    # 禁用指定pofile(不对这个程序做限制，不loaded)
aa-audit      # 审计模式，= enforce + 记录成功访问的请求(并不是很常见)
```

当前已经加载的配置文件
```bash
/sys/kernel/security/apparmor/profiles
```

配置文件命令
- 与可执行程序结对路径相同，` / ` -> ` . `
- 软连接将最终解析到目标程序


默认配置文件
安装包携带并有效（bind9, mysql ...）
安装包携带但为空（mariadb）
安装包无配置文件，额外包故障（samba）


###### samba为例
1.安装samba服务
```bash
# 
sudo apt install samba samba-common apparmor-profiles
sudo aa-status
#/etc/apparmor.d/usr.sbin.smbd     # nmbd服务也可以
```
2.启用强制策略
```bash
sudo aa-enforce /usr/sbin/smbd    # 对smbd进行策略限制
sudo aa-status
```
3.重启服务
```bash
sudo systemctl restart smbd    # 会发现报错无法启动服务(因为配置文件有问题)
tail /vat/log/syslog    # 查看系统日志，发现 apparmor="DENIED"， 建议：从第一个DENIED开始往下看,一次修改一条
```
4.修正配置文件
```bash
sudo vi /etc/apparmor.d/usr.sbin.smbd
    ## 这里根据日志DENIED的内容来写（每一行都有逗号）
    /var/run/smbd rw,    # path路径项权限 根据自己的写...
    capability net_admin,    # capability进程权限

# 重新加载配置文件
sudo apparmor_parser -r usr.sbin.smbd
# 重新加载所有配置文件
sudo systemctl reload apparmor.service
# 然后重启服务，查看能不能起来，如果不能起来->继续查看日志和修改配置的步骤
sudo systemctl restart smbd 
sudo systemctl status smbd 
。。。。。。
```


###### 从头开始创建新的配置文件
不建议对所有程序都做限制，主要面向`对外/接收提供网络服务相关`的程序(apache, samba, 浏览器等等容易遭受攻击的)

```bash
 # 查看进了行网络请求，但是没有apparmor配置文件对其进行保护的程序
sudo aa-unconfined   
sudo aa-unconfined --paranoid    # 所有关联网络端口的进程

# 这里拿mtr作为例子
aa-genprof mtr    # 指定对什么程序进行配置
mtr www.baidu.com    # 运行示例程序
# 继续按照提示选择

# 
sudo apparmor_parse -r usr.bin.mtr
sudo aa-logpro    # 检查日志更新配置文件
```






## <font color=red>防病毒</font>
<img src="http://image.xpshuai.cn/20200520113246789_2113511743.png" />


**恶意软件**
病毒、木马、蠕虫、远控、僵尸程序、勒索、挖矿、漏洞利用

通常防病毒和Linux不会出现在同一句话里，BUT！
- 专门针对Liunx发行版的恶意软件
- 没有100%有效的单个杀毒软件（病毒库互补）

**ClamAV**: 开源免费全平台

#### ClamAV
1.安装
```bash
sudo apt install clamav clamav-daemon   # 如果只安装clamav，就不做自动监控；clamav-daemon 自动查杀
sudo apt install install libclamunrar6    # 对压缩包也查杀病毒

# 有桌面环境的话，可以安装客户端图形管理工具
```

2.更新病毒库
```bash
sudo systemctl stop clamav-freshclam
# 
sudo freshclam    # 会报错(因为他已经自动更新了)，可以先停止掉(上面)，再手动。也可以等待他更新完
less /var/log/clamav/freshclam.log  # 可以查看日志记录

```

代理
```bash
sudo vi /etc/clamav/freshclam.conf
    HTTPProxyServer server_address
    HTTPPrpxyPort port_number

```


图形化前端工具
```bash
clamtk
```

3.手动扫描
```bash
# 扫描会占用系统资源，所以选择业务不是繁忙的时间来查杀
# 手动扫描的当前用户能扫描的位置
clamscan /home/xps -i -r    # -r递归， -i显示

# --max-filesize 限制大小(超过大小就不会扫它)， --max-scansize最大扫多少(比如一个文件我只扫多少)， --exclude-dir排除目录
sudo clamscan --max-filesize=3999M --max-scansize=3999M --exclude-dir=/sys/* -i -r /home
# 默认不具有杀毒能力，只是检测    

# --remove    查到感染的文件，删除
# --bell      查到感染的文件，嘟嘟嘟提醒
# move=/tmp/VIRUS    查到感染的文件，移动到某个位置
# copy=/tmp/VIRUS

```

测试
```bash
wget http://www.eicar.org/download/eicar.com    # 测试病毒文件,含毒，但是没破坏作用

clamscan -i -r ./
```


4.扫描调度(定时任务) -- 防止误杀(管理员要提前做好管理备份或者实时监视)
```bash
at 13:00 tomorrow
    clamscan -i -r /home/xps
    freshclam
    <EOF>
    <CTRL-D>

# 或者其他调度命令 crontab
```


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%AE%89%E5%85%A8%E7%9B%B8%E5%85%B3%E8%AE%BE%E7%BD%AE/  

