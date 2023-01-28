# ubuntu server-文件服务




# 文件服务

>我的就是你的 -  你的还是你的

网络诞生最初的动因是去中心化和资源共享  
文件是初期最主要的资源共享形式（把文件存放在集中的位置供大家访问）  

去中心化是另外一个话题

- 早期的军事需求
- 现在的信任需求

## <font color=red>FTP服务</font>

最古老的就是FTP  

- 产生于安全出现之前
- 明文传输一切
- 目前主要用于公开提供的文件下载服务(目前还没消亡)
- 使用简单、兼容性好
- 还有SFTP、FTPS等

###### FTP服务端软件

- Server U
- vsftpd
- Proftpd

###### 为了安全考虑

- 独立部署于企业防火墙之外
- 只读挂载存储或采用只读存储设备(CD / DVD)



## <font color=red>vsftp安装和配置和管理</font>

vsftpd  --  灵活、安全的FTP服务端软件

#### 安装

```bash
sudo apt install vsftpd

# 安装过程中生成ftp账号（anonymous） 主目录在：/srv/ftp
# 默认是21端口进行会话
```

FTP账号

```bash
# 安装完之后会自动创建一个ftp账号（这个账号是不能使用shell和登录机器 ）
/etc/passwd    # bin/false   表示不能使用shell
/etc/shadow    # *	表示不能使用终端
/srv/ftp		   # ftp用户主目录
```

#### 配置

两种配置模式

```bash
anonymous		# 匿名模式（适用于公开共享文件） -- vsftpd安装好之后是默认禁用匿名登陆的
Standard		# 认证模式
```

常见FTP客户端

```bash
# 用哪个账号登录，看到的文件就是哪个用户的主目录

# 1.使用命令行连接ftp服务器
ftp 192.168.1.130

# 2.浏览器
ftp://192.168.111.130

# 3.ftp客户端访问程序（FileZilla）

# 4.文件同步客户端软件
```

##### 1.匿名登录

配置文件位置

```bash
/etc/vsftpd.conf    # 主配置文件
```

**开启匿名模式**

```bash
sudo vim /etc/vsftpd.conf 
# 修改内容为
anonymous_enable=YES	# 启用匿名账号登录
local_enable=NO			# 禁用认证用户登录

# 重启vsftpd服务，检查状态

然后使用账号anonymous登录(命令行需要,其他方式不用)，密码不用输即可()
看到的目录是`/srv/ftp/`
```

**开启匿名账号上传**
默认：匿名账号是不允许匿名上传的

```bash
sudo vim /etc/vsftpd.conf 
# 修改内容
write_enable=YES	# 全局设置（以下配置生效的依赖）
anon_upload_enable=YES	# 允许匿名上传文件
anon_mkdir_write_enable=YES	# 允许匿名创建目录

# 重启服务，检查状态
sudo systemctl restart vsftpd.service

但是还是不能上传的（因为vsftpd强制禁止在根目录下匿名上传）
Note： ftp的主目录是不能进行上传文件的，因为这个目录对同组用户没有写的权限(`ls -ld`查看当前目录的权限)，就算你chmod修改了权限也是不行的
【解决办法】:在根目录新建文件然后上传
sudo mkdir /srv/ftp/upload  # 创建上传目录
sudo chown ftp:ftp /srv/ftp/upload # 配置新建文件系统属主数组

# 继续新增配置  /etc/vsftpd.conf 
anon_umask=022  # 上传文件权限掩码(让上传的文件配置不同的权限 -- 看自己需要来配置)
anon_other_wrote_enable=YES	# 允许删除和重命名(否则删除不了也无法重命名)
```

**修改FTP主目录**

```bash
sudo mkdir /srv/files/ftp
sudo chown -d /srv/files/ftp ftp
```


**文件传输日志**
默认是开启的，可以查看恶意文件上传记录

```bash
# sudo vim /etc/vsftpd.conf 

xferlog_enable=YES	# 默认开启
xferlog_file=/var/log/vsftpd.log  # 默认日志存放位置(注释不注释他都默认往里面写，可修改)

# 如果修改，重启服务
```

**其他配置**

```bash
idle_session_timeout=600	# 会话超时时长
no_anon_password=YES		# 命令行登录禁用密码提示
hide_ids=YES		# 显示属主/数组名称，而不显示uid和gid(uid会比较敏感)

# 重启服务
```

**端口**

- 会话指令 -> TCP  21端口       
- 数据传输模式
- 主动模式（受客户端防火墙影响）-- 客户端随机产生一个端口，告诉服务器  ->服务端只使用 20端口		
- 被动模式 （兼容性好，建议使用）         -- 服务端告诉客户端使用哪个端口传输

```bash
# 用什么模式，客户端说了算
# 都是客户端随机产生一个端口，告诉服务端

# 被动模式连接
ftp -p 192.168.0.130
```

<img src="http://image.xpshuai.cn/20200413153011895_32240.png" />


限定`被动模式`数据通信端口

```bash
# sudo vim /etc/vsftpd.conf   新增配置
# 开放一个端口的范围(尽量少开) 
pasv_min_port=40001
pasv_max_port=40101
```

然后在服务器端边界防火墙上开放上述100个端口


##### 2.身份认证登录

**身份认证登录TFP**

- 安装之后的默认配置
- 不能上传
- 可回溯到根目录（`存在安全隐患!!!` --> 使用chroot来解决）

**chroot**
chroot 将`所有`登录用户锁定在自己的主目录里（防止目录回溯）

```bash
# 配置文件 sudo vim /etc/vsftpd.conf
# 全局的配置参数(对所有用户)
chroot_local_user=YES   # 但是chroot的根目录不能是可写的

# 重启服务

# 默认主目录可写， 但是chroot的根目录不能是可写的，vsftp的禁止用户登录，好猫短   ————> 怎么办呢？看下面
```

**（接上文）两种解决办法:**
1.为FTP登录指定新的主目录（建议使用）

```bash
sudo mkdir ~/ftp && chmod -w ftp    # 新建主目录，并删除写入权限
# sudo vim /etc/vsftpd.conf
local_root=/home/xps/ftp

# 客户端只能看到这个目录里面，跳不出去
```

2.允许根目录写入（不建议，存在安全风险）

```bash
# 配置文件 sudo vim /etc/vsftpd.conf
write_enable=YES
local_root=/home/xps
allow_writeable_chroot=YES	
```


**chroot 将`指定`登录用户锁定在自己的主目录里（防止目录回溯）**

```bash
# 配置文件 sudo vim /etc/vsftpd.conf
# chroot_local_user=YES   # 注释掉前面配置的全局的配置项

# 启用用户列表功能
chroot_list_enable=YES  # 如果chroot_local_user=YES也开启了，就起了相反的作用(除了指定列表里面的用户之外, 都锁定在主目录里)
# 指定用户列表文件
chroot_list_file=/etc/vsftpd.chroot_list
# 编辑用户账号列表
vim /etc/vsftpd/chroot_list
# 添加用户
xps
ftp

# 重启服务
```

禁止指定FTP登录账号

```bash
/etc/ftpusers  # 默认禁止FTP登录账号(将用户加入进去)
```

**上传文件权限掩码**

```bash
# 跟匿名用户的配置是类似的
local_umask=022
```

**文件同步客户端**
自动把我电脑的新的文件，上传到FTP服务器(类似于备份) -- 有一些第三方软件(ubuntu自带的同步软件`backup`等)


## <font color=red>FTP安全</font>

###### 1.`FTPS`: FTP over Secure Socket Layer(SSL)

>账号无需shell登录权限  

基于证书的加密

开启FTPS：

```bash
# 安全性--指的是在数据传输过程中不会被别人嗅探
ssl_enable=YES		# 启动FTPS
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem   # 公钥
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key  # 私钥

# 重启服务
# 然后再连接的时候就是加密的了(浏览器需要ftps://ip)

# 怎么加密的呢?
# 简单来说：安装FTP时候默认会生成一对公钥和私钥
# 不建议使用安装FTP时候默认生成的证书，建议手动生成(每个应用一个证书, 每个证书有效期一两年)
```

**手动生成证书 --> 替换默认证书(安全考虑)：**
1.生成秘钥文件

```bash
# 使用ssl的一个开源组件openssl
openssl genrsa -des3 -out ftp.key 4096  # 1024的已经不安全了
# 然后设置密码保护
```

2.生成不加密的秘钥文件

```bash
# 先还原为明文的密钥对
openssl rsa -in ftp.key -out ftp.key.insecure  # 用完就删掉这个insecure文件
# 改名(这是加密的)
mv ftp.key  server.key.secure 
# 改名(这是明文的)
mv server.key.insecure  ftp.key
```

3.生成证书请求文件(使用上面的明文秘钥)   -- 实验使用的自签名证书

```bash
openssl req -new -key ftp.key -out ftp.csr
# 然后输入一些证书的信息
```

这里使用自签名证书(生产环境建议由证书颁发机构前面生成证书)  
3.生成自签名证书

```bash
openssl x509 -req -days 365 -in ftp.csr -signkey ftp.key -out ftp.pem   # -days 365有效期
```

4.部署证书

```bash
# ftp.key放到一个更安全的目录去
sudo cp ftp.key /etc/ssl/private/  # 此目录root才能看
sudo cp ftp.pem /etc/ssl/certs/
```


5.修改`/etc/vsftpd.cong`中的key和公钥为刚才手动生成的证书， 然后重启服务


上面的操作也可以合并为一条命令

```bash
openssl x509 -req -node -days 365 -newkey rsa:4096 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/private/vsftpd.pem

```


###### 2.`SFTP`: 基于SSH加密隧道传输文件

>账号需要shell登录权限（可设置禁止权限）

远程登录那一章已经学了




## <font color=red>NFS服务</font>

> 适用于类UNIX系统

- NFS -- Network File System
- 最早由SUN公司开发
- 类unix平台最主要的文件共享方法
- 基于RPC(Remote Procedure Call)协议
- NFS是一个RPC Server （v3 、v4）
- 协议本身不加密，可结合SSH，Kerberos实现加密
- 隐藏式身份验证（难点）
- 权限配置是关键
- 服务端口: TCP `2049`


**安装**

```bash
sudo apt-get install nfs-kernel-server  # 服务端
sudo apt-get install nfs-common	# 客户端

# 安装完，服务已经都启动了
ps -ef |grep rpc
ps -ef |grep nfs
```

**进程**

```bash
rpcbind		# RPC服务进程
nfsd		# NFS主进程(身份识别)
mountd		# 根据 /etc/exports 配置验证权限
lockd		# 锁定（需C/S同时启用）
statd		# 一致性（需C/S同时启用）
```

**身份识别**

- 客户端提供 UID / GID (与用户名、组名无关)
- 服务器按客户端UID /GID 赋权
- 若服务器无客户端UID / GID账号，则将客户端映射为匿名账号
- nobody / nogroup (uid: 65534)
- 客户端使用root账号（UID 0），默认映射为匿名账号
- 可修改配置文件映射 UID 0 为服务端 ROOT （存在安全隐患）

>如果客户端uid和服务端uid相同，则客户端会获得服务器端账号的权限；若不同，则客户端获取服务端的nobody权限(除非客户端是root账号,且服务端启用了客户端的root映射为服务端的root)

隐式的身份认证

**权限体系**

- 1.共享权限
- 2.文件系统权限


**配置**

```bash
sudo vim /etc/exports	# 主配置文件
# 内容：共享目录,谁可以访问我, 权限，(sync是数据先写到内存在硬盘可靠,async是写到内存然后写的硬盘速度快)，no_subtree_check是不进行子目录的检查(建议使用)
/exports/public		192.168.1.0/24(rw,sync,no_subtree_check)  # 允许访问的范围可以使用通配符： *.lab.com
		no_root_squash		# 禁用root默认映射为nobody(加在上面括号中的配置中)
# 
sudo mkdir -p /exports/public		# 创建共享目录
sudo chown nobody:nogroup /exports/public	# 设置权限
sudo systemctl restart nfs-kernel-server	# 重启服务

sudo exportfs	# 查看共享目录
cat /var/lib/nfs/etab	# 共享目录
cat /var/lib/nfs/xtab	# 客户端信息

```


**共享权限**

```bash
ro/rw		# 只读/只写
sync / async	# 同步写入硬盘 / 暂存于内存中
all_squash	# 所有用户全部映射为nobody
anonuid / anongid	# 支付那个匿名ID（默认65534）
secure / insecure	# 使用1024以下/以上端口
hide / no_hide	# 共享 / 不共享NFS子目录
```

**客户端挂载**

```bash
# 和本地挂载设备一样(挂载到本地，但是访问是通过网络流量的)
sudo mount 192.168.0.30:/export/public	my_nfs/

# 写入文件的话
1.临时获取root权限，会默认转为nobody
sudo xxxxx 
2.
chmod修改其他用户的权限后，直接写入即可(客户端的uid与服务端的uid相同的话)
```

**启动挂载**
风险：nfs服务器如果出现问题，本地机器启动就会特别慢

```bash
# sudo vim /etc/fstab
192.168.1.30:/home	/my_nfs/	nfs no_auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0  # 建议 no_auto 不自动挂载
```

**其他命令**

```bash
rpcinfo -p 192.168.1.30  # 查询RPC服务注册状态(客户端与服务端都可以执行)
tail /var/log/kern.log	 # 服务器日志
showmount -e localhost	 # 显示共享目录
df -h					 # 客户端查看挂载
```

**企业环境有统一域名和身份验证时**

```bash
# 也要配置这个
sudo vim /etc/idpamd.conf
Domain = lab.com
```

**Windows客户端**
企业版才可以添加nfs的支持




## <font color=red>SAMBA服务</font>

**SMB / CIFS 协议**

- 全称Server message block / Common internet file system
- 最早由IBM研发，后由微软采用并不断完善
- Windows文件和打印共享（之前使用的SMB协议, 后来改进为cifs协议）
- 端口：TCP 139(局域网), 445() / UDP 137(用来名称解析),  138
- Samba 是开源世界逆向了SMB协议后打造的兼容微软SMB的文件共享服务
- Samba还有其他功能、用途和使用场景

**功能不仅限于此，但最常用的用途，还是做文件共享服务：**

- NFS只适用于类Unix系统环境
- Samba适用与Windows、Linux混合环境的文件共享需要


**Samba实现了CIFS服务的四个基本功能**
1.文件和打印共享
2.认证和授权
3.名称解析
4.服务宣告(browsing)

**传输协议：**

```bash
NetBIOS		# 局域网
NetBIOS over TCP/IP		# 跨网段(把NetBIOS数据包封装之后跨网络传输)
```

**后台进程**

```bash
Smbd	# 文件共享主进程 TCP: 139/445
Nmbd	# WINS通信、名称解析 UDP: 137/138
Winbindd	# 同步系统账号(把Linux系统的账户拷到samba的用户数据库中进行处理)
其他10多个进程
```


#### 安装

```bash
sudo apt install samba libpam-winbind
```

#### 配置

配置文件：

```bash
/etc/samba/dhcp.conf	# 指定WINS服务器（跨网段时候用）
/etc/samba/smb.conf		# 主配置文件
```

###### 一. 配置验证用户名的Samba服务

1.配置文件/etc/samba/smb.conf	

```bash
workgroup = WORKGROUP  # [global]  工作组：net config worktation查看工作组

[Private]	# 起个名(共享文件夹的名称)
comment = prvivate share	# 描述
path = /srv/private/			# 共享路径
browseable = no				# 是否允许访问者可显示它这个共享文件夹(完整账号)
guest ok = no				# 禁用来宾账号
writable=yes				# 可读写(read only=yes)
create mask = 0700		 	# 新建文件权限
vaild users=@samba			# 可访问共享的用户组(稍后会创建)

#
```

2.创建用户/组/共享文件夹

```bash
sudo adduser user1
sudo groupadd smaba  # 上面配置的组的名称
sudo gpasswd -a user1 samba
sudo smbpasswd -a user1	# 设置用户SMB密码(不是操作系统的密码)
sudo mkdir /srv/private/  # 创建共享目录
sudo setfacl -R -m "g:samba:rwx" /srv/private/	# 设置acl访问控制列表，-m修改 (ll之后看到后面有个+号，表示有更多权限) 
getfacl /srv/private/  # 查看权限
testparm		# 检测
sudo systemctl restart smbd.service nmbd.service
```

**客户端访问**
win上可以访问Linux上的Samba文件夹, Linux上也可以访问Linux上的Samba文件夹

Linux

```bash
# 图形化
点击Connect to Server，输入地址：
smb://192.168.0.130

#  命令行
smbclient -L //192.168.0.130   # 查看目标都有哪些共享信息
smbclient -L //192.168.0.130 -U user1	
mount -t cifs -o username=user1  //192.168.0.130/private/public	/mnt  # 挂载
```

Windows

```bash
# win+R
\\192.168.0.130

# 命令行工具来访问
net user # 查看会话
net user \\host\private/user1 passwd
net user \\host\private /delete  # 删除已经建立的会话
net user g:  \\host\private 		# 映射为本地盘符

net config workstation 
```

共享名称以`$`结尾的，在win上会认为隐藏共享(Linux上还是会显示)

<img src="http://image.xpshuai.cn/20200414155510404_5305.png" />

###### 二. 配置公开访问的Samba服务(开放共享文件件)

将不提供登录账号的用户映射为guest，无需输入密码  

1.配置文件

```bash
#配置文件/etc/samba/smb.conf	
[Global]	# 起个名(共享文件夹的名称)
workgroup = WORKGROUP 
security = user # share已废止、Domain、ADS、Server
map to guest = bad user # 映射为guest
guest ok = yes		# 允许来宾账号
```

2.开放共享文件夹

```bash
[Public]
comment = public share
path = /srv/public/
browseable = yes	# 允许看到
writable = yes
guest ok = yes

```

3.创建共享目录

```bash
mkdir -p  /srv/public/

# 设置访问列表（设置user,nobody代表guest）
sudo setfacl -R -m "u:nobody:rwx" /srv/public/
getfacl /srv/public/  # 查看权限

testparm		# 检测
# 重启服务
sudo systemctl restart smbd.service nmbd.service
```


**客户端访问**
同上



**服务器信息**

```bash
smbd --version  # 查看版本

sudo smbstatus	# 查看状态和连接状况

```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu-server-%E6%96%87%E4%BB%B6%E6%9C%8D%E5%8A%A1/  

