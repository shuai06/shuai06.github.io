# ubuntu server-远程管理




  

# 第七章-远程管理

>看我遥控你

## <font color=red>远程访问方式</font>
**为什么要远程管理？**
服务器大多部署与专门的机房：
- 电磁、噪声、氧气、温度湿度等都不适宜人类长期居住
- 避免闲杂人等结束业务服务器

**远程管理**
- 不同的操作系统都支持远程管理技术
- 命令行远程管理工具
- 图形化远程管理工具


#### Telnet
>账号证明就是你

古老的命令行远程管理工具（保底的，建议能不用就不用）

- 不安全（明文）
- 应尽量避免适用（适用于不支持ssh的环境）
- 客户端程序包含在所有系统的默认安装中
- 服务端口默认TCP 23

**客户端**
```bash
#可以连接任何域名和端口
telnet 1.1.1.1 80

#和80端口建立socket连接后，可以发送http数据
```

**服务器端程序安装**
```bash
sudo apt install telnetd

vim /etc/issus.net # 避免泄露版本信息,
#打开文件，修改tenlet的banner信息（可以修改为一些告知的警告信息:"本公司的服务器,出问题承担法律责任"哈哈）

# 查看服务状态
systemctl status inetd.service
```

#### SSH
>账号证明就是你

**服务器安装**
```bash
# ubuntu中ssh-server和ssh-client都默认安装了
sudo apt install openssh-server  # openssh是个工具套件(包含了sftp,scp等)

# sftp(替代了sftp)  <--->  ftp(明文传输,不太安全)
# scp(替代了rcp)   <--->  rcp(远程拷贝文件,明文传输,不太安全)


#查看服务状态
systemctl status sshd.service


# 服务端口默认`22`  

# 配置文件(ubuntu上来就可以直接使用不用配置)
vim /etc/ssh/sshd_config
#修改配置在这个文件

```
SSH1、SSH2两个版本(版本2更安全)


**配置文件**
`vim /etc/ssh/sshd_config`
常用配置项：
```bash
Banner /etc/ssh_banner.sh   # 登录前的banner,指定文件
Port  2222		# 修改默认端口(修改后，下次连接会改变端口)
PubkeyAuthentication yes	# 开启后使用公钥登录/关闭(no)
ListenAddress 12.34.56.78  # 指定哪块网卡才能侦听22(被ssh连接)
PermitRootLogin	yes		# 是否允许root远程登录(不过还是建议使用普通账号),建议设置为no是禁止所有方式登录   #prohibit-password是禁用账号密码方式
Protocol 2  # 默认是2.0的协议，可以加一下这条配置，指定只接受2.0版本的连接
Allowsers user1 user2		# DenyUsers user3    允许/禁止ssh连接的用户
AllowGroup sshusers			# 允许/禁止ssh连接的用户`组`
PasswordAuthentication no  # 禁止使用密码登录(单一密码认证的安全性很低)

# 更多配置查文档
```

**一些零散问题  --- SSH工具**
1.scp
```bash
#默认连接22端口，如果修改了就需要指定端口号
scp a.txt 192.168.1.10:  # 拷贝一个文件(默认本地账号与原创系统账号同名，默认目标是主目录)   注意冒号
scp a.txt user1@192.168.1.10:/home/b.txt   # 指定用户名、目标路径

scp -rv dirs/ user1@192.168.1.10:   # 拷贝一个目录， -r递归,-v详细信息

scp user1@192.168.1.10:/tmp/b.txt . # 拷贝目标服务器的文件到我的机器当前目录上(下载)
```

2.SFTP（ssh和ftp的组合）
```bash
# 登录
sftp xps@192.168.1.10

# 下载文件
get a.txt

# 上传
put

# 一些命令：help, ls, cd, get, wget, put, mput
# 退出 quit   exit

```

**ssh公钥登录**
非对称算法（公钥算法）
- 公钥
- 私钥
公钥加密，私钥解密；私钥加密，公钥解密

生成秘钥对
```bash
# 1.在我的机器(客户端)产生一对， 选择rsa算法，秘钥长度选择4096
ssh-keygen -t rsa -b 4096

# 修改...
# 命名秘钥对
ssh-keygen -t rsa -b 4096 -f id_mail
# 修改私钥密码文(这个密码要设置更加安全一点) -p， -f指定修改哪个key的
ssh-keygen -p -f ~/.ssh/id_rsa    # 有多个的话，-f指定


# 2.设置公私钥保存的位置，和打开的密码
# 存放在下面两个文件
~/.ssh/id_rsa  # 私钥
~/.ssh/id_rsa.pub  # 公钥
#ps：know_hosts是连接过的主机

# 3.把公钥文件-->传送到目标服务器上(服务器上会变成.ssh/authorized_key)
ssh-copy-id  xps@192.168.0.10  #如果有多个，-i选择公钥



# 4. 建议设本地私钥文件权限
chmod 400 .ssh/id_rsa
# 5.服务端公钥权限也要修改
.ssh/authorized_key

# 6.现在可放心登录了(现在输入的密码是对私钥加密的密码)
ssh xps@192.168.0.10

# Note：记得配置文件修改为禁止使用ssh密码登录,使用公钥登录：
PasswordAuthentication no
PubkeyAuthentication yes
# Note:私钥千万不能泄露哦

```


## <font color=red>SSH隧道</font>
通过ssh隧道，封装一些其他的服务数据，保证安全性

**登录**
```bash
ssh xps@192.168.0.10

# 登录之后，不进入shell界面（只是获得会话后运行一个命令，但效果是一样的） -- 命令服务端去ping另外一台机器
# -i指定私钥，-p指定端口
ssh -i ~/.ssh/id_main ssh xps@192.168.0.10 -p 2222 ping 192.168.0.1

# (cstream -t限制传输速度通信带宽)拷贝文件夹到目标机器（本地打包, 在服务端解包），pv显示带宽占用情况 
# --- 企业用的网络带宽(质量好)比家用的成本很贵
tar -cj dir/ | pv |cstream -t 100k | ssh xps@192.168.0.10 'tar -xj'
```


**ssh隧道 -- 实现远程映射**
```bash
###### 端口映射（两个机器之间，添加一个加密的管道 来连接）
# 映射远程端口23到本机2001端口
#（-f端口转发,-N不占用命令行窗口, -L侦听本机的端口，localhost理解为隧道，23为目标机器端口，最后是目标机器ip）
ssh -fN -L2001:localhost:23 xps@192.168.0.10 
# 现在，telnet本机的2001端口，就相当于通过机密的隧道来映射到目标的23端口
# 其他类型的服务也可以封装...

### 拓展：远程映射
# 隧道的起点：我的机器，隧道的终点：'另外机器'。然后通过'另外机器',访问目标机器(sina)
# 本机的2002端口，通过ssh隧道，映射到'另外机器'(世界上随便一个机器)的ip的80端口，
# 用途：内网特殊的服务器运行访问外网(进行映射)，就能突破对内容不能上网的限制
# 限制：一对一，针对的外网的某一台（提高：一个端口访问外网所有资源，自己找资料）
ssh -fN -L2002:123.206.96.26:80 xps@192.168.0.10  
# 再telnet 127.0.0.1:2002,就是达到了访问123.206.96.26的80端口的目的(最后的ip只是建立隧道,类似中间的一个)

```



<img src="https://image.geoer.cn/20200330152732562_32379.png" />

<img src="https://image.geoer.cn/20200330154229749_5054.png" />

**ssh隧道 -- 实现文件系统挂载**
比如，一开机，就自动把远程机器的目录/分区挂载到我本机上，跟访问本地目录一样的效果

```bash
## ssh隧道实现文件系统的挂载
# 第一个ip的要挂载的远程机器，后面的本机目录
sshfs xps@192.168.0.10:home/test  /mnt/myfiles	# 远程挂载远程机器目录到本地
# 再查看，会发现本地目录多了远程服务器目录的文件（如果网络带宽不够，性能和体验就比较差）
# 在客户端/服务端对文件进行操作，都是可以的
umount /mnt/myfiles		# 卸载1
fusermount -u /mnt/myfiles    # 卸载2

#配置文件使用持久化挂载	
vim /etc/fstab
user1@192.168.0.10:/path  /mntpotin   fuse,sshfs,	rw,noauto,user,_netdev0
# 目标机器的路径，本机的挂载点
```


**SSH客户端**
若要管理的服务器的数目众多，对每台服务器做一个方便记忆的`标签`，简化登录

在本机(客户端)创建一个客户端配置文件：`~/.ssh/config`  
内容
```bash
host server1  # 别名随意
	Hostname 192.168.1.10  # 对应主机ip，端口，用户名
	Port 22
	User xps
	
host db2
	Hostname 192.168.1.10
	Port 22
	User xps
	
host mail1
	Hostname 192.168.1.10
	Port 22
	User xps
```

以后登录的话，直接：
```bash
# 直接登录别名即可(不仅方便，还隐藏了ip，更加安全)
ssh db1
```




## <font color=red>SSH安全</font>
**SSH防爆破**
密码爆破防护

方法：监视：次数太多就禁止该ip访问

借助`fail2ban`来实现:
fail2ban原理：周期性的检查登录日志，对异常的ip进行屏蔽(调用防火墙实现)
```bash
# 安装
apt install fail2ban

# 默认主配置文件，软件更新时被覆盖(`所以一般不会使用它`)   jail --> 监狱
/etc/fail2ban/jail.conf
# 本地配置文件，软件更新不被覆盖，优先级更高（`一般常修改这个文件`） 复制一份：cp jail.conf jail.local
/etc/fail2ban/jail.local
#配置项(一般只需配置几项)
#1.全局的：
ignoreip=127.0.0.1/8 19.168.0.10  # 忽略并不进行监视的地址（受信的ip地址段）
bantime=-1   # 禁用的时间(单位是s)， -1是永远屏蔽
findtime=1   # 查询间隔时间
maxretry=5   # 连着失败输入多少次密码，我就屏蔽你
#2.Actions: 有异常做什么动作(比如报警邮件，默认是防火墙屏蔽)
#3.Jails设置：针对不同类型的服务(并不仅限于ssh服务)，配置不同的（这里的优先级比全局配置项高）
	[sshd]
	ecabled=true # 启用对ssh的jails
	filter=sshd  # 查找的关键词
	logpath = xxxxxx.log
	# 以ssh服务为例(见下图)

# 重启服务生效 
sudo systemctl restart fail2ban.service


# 查看身份认证日志文件
tail -f /var/log/auth.log

# 查看jail列表(都有哪些jail，jail的情况)
sudo fail2ban -client status
sudo fail2ban -client status sshd # 查看ssh的jail的详细信息

# 查看防火墙规则
sudo iptables -S
sudo iptables -L -n

# 手动解除被禁IP
sudo iptables -D f2b-sshd-s 192.168.1.111 -j REJECT   # 通过删除防火墙某一个规则 -D,  REJECT或DROP看自己的情况
sudo fail2ban-client set sshd unbanip 192.168.1.8

# 手动解除禁用之后，但是如果重启服务，会再自动查看日志：还没有超过被禁期限的ip会再次被禁

```
<img src="https://image.geoer.cn/20200330164545399_6809.png" />





## <font color=red>VNC服务</font>
>这里只介绍vnc，还有其他更优秀的工具，但是vnc的适用范围最广

图形化界面（类似于windows下的远程桌面）
- 使用图形化界面及工具
- 是Linux系统最通用的远程图形管理工具（通用适用于windows）

安装图形环境
```bash
#这里安装简化版的xfce4桌面环境，这里选择vnc的一个软件包:tightvncserver. 也可以自己选择别的版本的
sudo apt install gnome-core xfce4 xfce4-goodies tightvncserver

# 一个vnc可以启动多个侦听端口(默认从5901开始)，从不同端口连接，打开不同会话(每个用户看到的桌面是不一样的)
# 若只侦听一个端口，通过一个端口连接，多个用户看到的都是一个桌面

# 建议：多用户，在每一个不同权限的用户下(su进去，运行启动实例)，这样不同用户连接就是不同的桌面了
```

运行配置
```bash
vncserver  # 启动vnc服务（接下来会让你输入连接密码和只读密码）
~/.vnc/xstartup   # 会生成运行文件
vncserver -kill:1 # 删掉自动生成的实例（因为不一定最适合我们的桌面环境。冒号后面是1是5901,如果是2就是5902）
# 备份一下文件
mv xstartup xstartup.bak
# 创建新运行文件， 手动配置适合我们xfce4的vnc配置
vim xstartup
	#!/bin/bash
	xrdb $HOME/.Xresources
	startxfce4 &

# 修改权限
chmod +x ~/.vnc/xstartup
# 重新运行
vncserver   # tcp的5901端口

# 就可以在客户端用客户端软件连接了(比如：Remote Desktop Viewer) 输入ip:端口

# Note: 服务器上，能少开端口就少开端口，能少开服务就少开服务
```

多用户
```bash
# 每个用户不用会话
# 在每一个不同权限的用户下(su进去，运行启动实例)，这样不同用户连接就是不同的桌面了
su newuser1
vncserver
~/.vnc/xstartup  
vncserver -kill:2
vim xstartup
	#!/bin/bash
	xrdb $HOME/.Xresources
	startxfce4 &
	
chmod +x ~/.vnc/xstartup
vncserver :2   # tcp的5902端口
```

>ps: vnc的加密强度不是很高，所以：在不需要vnc的时候，建议杀掉vnc进程(`vncserver -kill:2`)

因为 vnc的加密强度没有ssh高，所以，尝试：先建立ssh隧道，再在隧道的基础上建立图形化的连接



## <font color=red>PUPPET</font>
也是一个远程管理工具，可以不登录到被管理服务器上  
利用puppet的服务端(我的机器)来管理puppet的客户端(安装在远程服务器上)，进行批量管理...  

全生命周期的远程管理方案  
- 软件安装部署(可以写一个脚本，然后批量下发给安装有puppet客户端的服务器上)
- 权限设置
- 账号管理
- 支持云平台
- 支持容器(Docker)
- 适合大量服务器管理


**准备**
```bash
# 这里以两台机器做实验
# 为了区分，改一下两台计算机名
hostnamectrl set-hostname puppet
hostnamectrl set-hostname client

# 修改hosts文件，修改解析域名(这个随便起) -- 两台机器都要修改
vim /etv/hosts
	192.168.1.10	puppet.lab.com		puppet
	192.168.1.11	client.lab.com		client
	# 其他行都删掉就行
	# 可以在dns服务器上修改，这里通过hosts修改域名解析只是权宜之计(下一章学完DNS之后就在dns服务器修改域名的解析)
```


**安装**
```bash
# 安装puppet服务端程序
sudo apt install puppetmaster 

# 安装puppet客户端程序
sudo apt install puppet
```

**配置**
服务端配置-1
```bash
cd /etc/puppet
sudo mkdir -p modules/apache2/manifests  # manifests 目录下面就是资源文件
touch init.pp
# sudo vim /etc/puppet/modules/apache2/manifests/init.pp

# 比如要给下面的机器安装apache, 
# 一组资源就是一个类
class apache2{
	package{'apache2':   # 软件包
		ensure=>installeds,
	}
	service{'apache2':   # apache的服务是否启动
		ensure=>ture,
		ensure=>ture,
		require=>Package['apache2'],  # 包的名称
	}
}
```

服务端配置-2
```bash
# 上面的配置文件定义了资源，那么这里确定要给哪些服务器安装哪个资源
# 选择哪个资源
cd manifests
touch site.pp

# sudo vim /etc/puppet/manifests/site.pp
# 解析这个节点(通过域名解析到对应ip,只要那些计算机在这个域名即可)
node 'client.lab.com' {
		include apache2   # 选择下发哪些资源。这里以推送apache2资源为例
}

# 重启服务
systemctl restart puppetmaster.service
```


客户端配置
```bash
# puppet的客户端和服务端是通过证书来通信等操作的

# sudo vim /etc/default/puppet          默认没有，自己新建
	START=yes

# 重启服务（会自动向puppet服务端发起请求申请证书）
systemctl restart puppet.service
```

客户端证书前面请求测试
```bash
# 在查看客户端的指纹信息
sudo puppet agent --figerprint
# 服务端下面下发证书后，然后在客户端测试一下，获取证书
sudo puppet agent --test
```

服务端请求查看签名
```bash
# 在服务端查看哪些客户端跟我发起证书请求了
sudo puppet cert list
# 在服务端使用自己的私钥给对应的机器进行签名(sign)
sudo puppet cert sign client.lab.com

# 客户端查看证书下发进度： sudo puppet agent --test
```

完成上面的步骤(资源的下发配置，证书的请求与签名下发，重启服务)之后，在客户端查看`apache2`的服务已经安装好并启用了

日志状态信息查看
```bash
# 查看状态
systemctl status puppet.service

/var/log/syslog

```



上面关于puppet只是简单的一小部分，后续高级用法，请自行探索













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E8%BF%9C%E7%A8%8B%E7%AE%A1%E7%90%86/  

