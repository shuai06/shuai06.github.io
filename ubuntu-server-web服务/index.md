# ubuntu server-Web服务



## <font color=red>WWW介绍</font>
www 万维网

- 静态网站：所有人看到的内容都一样
- 动态应用程序： 
	- 数据库、中间件
	- 每个用户看到的内容不同
	- 根据用户输入返回不同结果

**访问方式**

<img src="http://image.xpshuai.cn/20200424214945523_10883.png" />
FQDN：(Fully Qualified Domain Name)全限定域名：同时带有主机名和域名的名称。（通过符号“.”）

**传输协议**
SSL基本被替代了，TSL使用最多
<img src="http://image.xpshuai.cn/20200424215120585_7753.png" />
put常用于上传文件
options查看都支持哪些方法
header只是头部的信息(请求/响应)

<img src="http://image.xpshuai.cn/20200424235043151_8198.png" />








## <font color=red>HTTP协议</font>
**常见服务端响应状态码**
<img src="http://image.xpshuai.cn/20200425094030209_15175.png" />
更多详细状态码，见地址：

http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html



## <font color=red>Apache2</font>
Apache软件基金会的一个开源Web服务器  
**模块化强大的扩展能力：**
- SSO模块
- 并发限制模块
- 日志监控模块
- WAF模块
- 负载均衡模块
- 音乐/图像处理模块


#### 安装
```bash
sudo apt install apache2

# 默认侦听端口 TCP 80
```

#### 通过众多`directives`进行配置
在Ubuntu中，directives分散于多个配置文件中

<img src="http://image.xpshuai.cn/20200425100139669_32000.png" />

配置文件：`conf-enabled`(启用的)中的文件都是链接到`conf-available`(可用的)下的配置文件（创建了链接的才生效的，所以可以配置一些备用的不设置链接），修改完需要重启服务
模块目录：mods-enabled与mods-available，跟上面两者关系类似
站点：sites-enabled与sites-available
	  不同ip以及不同端口可以映射不同的网站 --> 同一个ip同一个端口来设置多个网站(不同域名都解析到这个ip, 服务端根据域名字段[主机头]来判断返回哪个页面) --> 即为虚拟主机


###### 虚拟主机简单配置
>一般先从available目录开始创建，至于启用与否还得配置enabled
本质上配置指令可以位于任何一个配置文件中

**配置`sites-available`下讲解：**
```bash
# /etc/apache2/sites-available/000-default.conf  # 默认虚拟主机

<VirtualHost *:80>    # 匹配ip，*代表任意ip  , 侦听任意ip的80端口
	ServerName www.example.com			# 设置主机头（虚拟主机的域名）
	ServerAdmin webmaster@localhost		# 报错时显示的管理员邮箱
	DocumentRoot /var/www/html			# 指定Web根目录
	ErrorLog ${APACHE_LOG_DIR}/error.log		# 报错日志
	CustomLog ${APACHE_LOG_DIR}/access.log combined		# 访问日志
	# 侦听端口
</VirtualHost>
```

**实验**
创建`ali`和`baid`两个域名的虚拟主机
1.创建新的虚拟主机（在sites-available）
```bash
# 复制两个配置文件
sudo cp 000-default.conf ali.conf
sudo cp 000-default.conf baid.conf

# 分别进行配置
ServerName www.ali.com
DocumentRoot /var/www/ali
#
ServerName www.baid.com
DocumentRoot /var/www/baid

# 分别编辑页面文件
sudo vim /var/www/ali/index.html
sudo vim /var/www/baid/index.html
```

2.创建链接(在sites-enabled)
```bash
# 可以用ln 创建

# apache提供了更简单的方式
sudo a2ensite ali  
sudo a2ensite baid
# 重新加载
service apache2 reload

## Note：
ServerAlias *.lab.com  # 域名通配符
sudo a2ensite mynewSite	# 启用虚拟主机
sudo systemctl restart apache2.service  # 重启服务
sudo a2dissite mynewSite	# 停用虚拟主机（禁用某个站点）
```

3.这里做实验，域名就简单通过hosts文件做了解析
```bash
192.168.11.130 www.ali.com
192.168.11.130 www.baid.com
```
3.访问



###### 侦听端口
```bash
/etc/apaches/ports.conf
```


###### Apache中默认基本配置
**1.索引文件index：**
DirectoryIndex
```bash
## 可以根据需求,手动修改索引页的名称（一般不建议修改）
# /etc/apache2/mods-available/dir.conf
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm  # 有先后的执行顺序
</IfModule>

# Note：
若没有正确放置index索引文件，会看到文件列表，可能会有安全隐患
```

**2.html格式展示目录列表：Options**
【安全考虑】-->去掉由于没有索引文件会显示文件列表的选项
```bash
## /etc/apache2/apache2.conf   主配置文件
<Directory /var/www/>   # 对文件目录权限的配置
    Options Indexes FollowSymLinks  # Indexes:若没有索引文件,就会显示目录列表;去掉Indexes就不会显示列表了，提示"权限拒绝"
    AllowOverride None
    Require all granted
</Directory>

# 重启服务
```

**3.Permission Denied **
以上1,2两项都不匹配，就会显示"权限拒绝"


**4.ErrorDocument  报错提示**
【安全考虑】--> 默认404或报错页面会泄露信息或扫描器，建议替换为我们自定义的页面-->不告诉用户不成功的具体原因,只告诉他出错了
```bash
# /etc/apache2/conf-available/localized-error-pages.conf  
ErrorDocument 404 /error/404.html       # 定义自己的页面

# 重启服务
```

5.基于目录的Options
```bash
## /etc/apache2/apache2.conf   主配置文件
<Directory /var/www/> 
    Options Indexes 
</Directory>

# 其他配置项(权限要最小化原则，安全起见)
ExecCGI		# 允许CGI脚本执行的目录(/usr/lib/cgi-bin)
Includes	# 允许服务端包含（一个html包含另一个html,模板化）
IncludeNOEXEC	# 允许包含，但进展CGI脚本exec、include

# 重启服务
```

6.环境变量
```bash
# /etc/apache2/envvars	可以修改用户和组和pid等
User /Group 	# 服务器端应答客户端访问的账号(www-data)
PidFile			# /var/run/apache2/apache2.pid

# 最大并发数（默认8192）
APACHE_ULIMIT_MAX_FILES='ulimit -n 65536'
```


#### 模块化
服务器核心只实现了基本功能
```bash
apache2 -l  # 查看默认被编译到apache核心包里面的模块
```
其他功能通过模块实现
ubuntu编译动态加载模块实现运行时模块调用
```bash
apt search libapache2-mod		# 搜索模块
sudo apt install libapache2-mod0auth-mysql	# 安装模块
sudo a2enmod auth_mysql			# 启用模块
sudo a2dismod auth_mysql		# 禁用模块
```

#### 开启HTTPS
`传输过程中`的CIA (机密性、完整性、可用性)，但是不提供`应用层`安全
```bash
# /etc/apache2/sites-available/default-ssl.conf
sudo a2ensite default-ssl  # 开始ssl加密(默认未启动)
sudo a2ebmod ssl	# 默认未启动

_default_:443	# 未指定虚拟主机情况下的443端口访问, 默认是443(除了其他指定了虚拟主机名的，未指定的就访问到这里)
SSLEngine on	# 开启SSl/TSL
SSLCertificateFile  /etc/ssl/certs/ssl-cert-snakeoil.pem  # 证书文件(默认已经自带了一张证书)
SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
SSLCertificateChainFile /etc/apache2/ssl.crt/server-ca.crt  # 证书的加密链文件
SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl      # 证书吊销列表
SSLVerifyClient require   # 要求客户端提供证书(不可抵赖性)

<FilesMatch "\.(cgi|shtml|phtml|php)$">   # 基于文件类型，设置不同环境变量进行加密
      SSLOptions +StdEnvVars
</FilesMatch>
<Directory /usr/lib/cgi-bin>    # 对指定目录
      SSLOptions +StdEnvVars    # 标准环境变量
</Directory>

ssl-unclean-shutdown   # 只是服务器端向客户端断开连接(追求速度)
ssl-accurate-shutdown	# 服务器端与客户端的连接端口了，客户端还有向服务端再确认(追求稳定性)

nokeepalive ssl-unclean-shutdown
downgrade-1.0 force-response-1.0	# 降级响应

# 还可以按自己需要...

## 启用并重启服务
sudo a2ensite default-ssl
service apache2 reload
sudo systemctl restart apache.service

## 可以访问了(ubuntu自己生成的证书浏览器会提示不安全)
```

###### 证书
向证书颁发机构申请的合法证书就可以替换ubuntu自带的证书
```bash
# 使用CA机构签名证书
sudo openssl req -newkey rsa:2048 -nodes -keyout server.key -out server.pem

# 使用自签名证书
sudo openssl req -x509 -days 365 -sha256 -newkey rsa:2048 -nodes -keyout /etc/ssl/private/mysite.key -out /etc/ssl/certs/mysite.pem
```


###### 创建`HTTPS`虚拟主机
和创建一般虚拟主机是类似的

<img src="http://image.xpshuai.cn/20200425131432963_15883.png" />

建议商业证书颁发机构(权威机构颁发，默认受信，增强特性，收费高昂，有效期长)

**那没钱怎么办呢？**
有几家机构已经有免费的了，比如`Let's encrypt`
**Let's encrypt**
- 致力于加速全文实现https的免费证书颁发机构
- 证书有效期90天(可刷新)
- 专门客户端程序 `Certbot`，简化证书步骤
```bash
sudo add-apt-repository ppa:certbot/certbot  # 添加仓库
sudo apt update
sudo apt install cerbot  python-cerbot-apache
```

使用方法：
```bash
# 前提
域名正常解析、设置好虚拟主机

# 申请证书 (--apache意思是给apache用的， -d是域名,还有子域名)
sudo certbot --apache -d example.com -d www.example.com

# 证书位置
/etc/letsencrypt/live

# 查看证书状态(换成自己的域名)
https://www.ssllab.com/ssltest/analyze.html?d=example.com&latest
```



#### Apache支持PHP
1.安装
```bash
sudo apt install php7.2 libapache2-mod-php7.2   # mod-php模块
# 验证 
php -v
```

2.启用模块
```bash
# 直接启用即可
a2enmod php7.2

# Apache对php的支持是模块化的，而Nginx对php支持的配置方式比较原始
```

索引文件
```bash
vim  /etc/apache2/mods-available/dir.conf       # index.php

# 重启
sudo systemctl restart apache2
```

配置文件
```bash
/etc/php/7.2/    # 下面不同的目录，是针对不同应用的，这里apache的在apache子目录下面
/usr/lib/php/7.2/    # 为不同的使用场景的配置文件(开发环境与生产环境，一般情况下会把这里的文件链接到上面的配置中)
```



## <font color=red>Keepalived</font>
高可用的一种解决方案：Keepalived

#### 实现高可用(HA)
- 在服务器组内漂移的VIP（多台服务器实现一个组，但也是一个域名，解析到一个ip）
- Master独占VIP，其他Server检查Master可用性
- Keepalives并非专门针对Apache而设计
- 常用于负载均衡技术环境
- 组内服务器应提供相同内容和配置

组内不同服务器有优先级，多个服务器共用一个漂移的ip地址(VIP)，处于正常状态的服务器叫Maste


#### 安装
```bash
sudo apt-get install keepalived

# 因为无配置，所有默认启动失败
/etc/keepalived/keepalived.conf		# 配置文件（需要创建）
```
配置文件：
```bash
# 可选配置项，但是一般建议配置
global_defs {  
    notification_email {
    myemail@mycompany.com   # 发送邮件给谁
    }
    notification_email_from keepalives@mycompany
    smtp_server 192.168.1.150
    smtp_connect_timeout 30
    router_id mycompany_web_prod
}
# 关键配置
# 1号配置实例
vrrp_instance VI_1 {
    smtp_alter
    interface enp0s3	 # 网卡接口优先级
    virtual_router_id 51 # 定义服务器组
    priority 100		 # 优先级(自己定义，保证顺序即可)
    advert_int 5		 # 周期性通告的间隔时间
    virtual_ipaddress {  
        192.168.1.200	 # VIP（DHCP以外）
    }
}


# 重启服务
```

```bash
systemctl start keepalived 
systemctl status -l keeplived
# 网卡绑定VIP
	 ip a
	 等待30s

# 安装第二台服务器，并进行类似的配置(优先级修改,vip地址填写一样的)

# 模拟切换
systemctl stop keepalived 
systemctl start keepalived 
```

不仅可以配合Apache使用，还可配合其他使用






## <font color=red>Ubuntu 18.04新特性</font>
哈哈哈，前几天20.04都已经正式发布了，我听课进度太慢
>内核是关键

<img src="http://image.xpshuai.cn/20200425142824240_1739.png" />

<img src="http://image.xpshuai.cn/20200425144457182_27054.png" />

###### 网络配置

<img src="http://image.xpshuai.cn/20200425145333762_14082.png" />
<img src="http://image.xpshuai.cn/20200425145724172_15710.png" />
多个地址，要用逗号分隔
重启服务：`sudo netplan apply`

<img src="http://image.xpshuai.cn/20200425145944914_17134.png" />
桥接的物理网卡设置为没有ip

**桌面端：**

<img src="http://image.xpshuai.cn/20200425150429971_9910.png" />


```bash
# ifconfig / ifup / if down  已经默认不安装了
# 对应上面
ip link set eth0 up  / ip link set eth0 down
```

**networkctl命令**
```bash
# 此命令地位更加突出
networkctl status / networkctl status eth0   # 查看不同网卡的状态和详细信息
```


**ip命令**
ip link
```bash
sudo ip link ens33 down  # down掉, 相当于ifup
sudo ip link ens33 up    # 启动网卡, 相当于 ifdown

ip link ls ens33  # 查看网卡的link信息
ip -s -d link ls ens33  # 查看网卡的link信息(更详细)
ip -s -s -d link ls ens33  # 查看网卡的link信息(更更详细)
ip link set dev ens33 mtu 1500  # 修改MTU
ip link set dev ens33  address 00:11:22:33:44:55  # 修改MAC
```

ip addr
```bash
ip addr    # ip a
ip addr show  # 跟上面没什么差别

ip addr add dev ens33 192.168.1.10/24  # 手动给网卡添加ip
ip addr del dev ens33 192.168.1.10/24  # 手动给网卡删除ip
ip addr fulsh dev ens33  # 清除网卡信息
```

ip route
```bash
ip route show  # 查看路由信息   ip route
ip route get 1.1.1.1   # 查看到达指定目的ip，计算要经过的路由
sudo ip route default via 192.168.1.2  # 添加默认路由(网关地址)
sudo ip route dev ens33 2.0.0.0/8 via 192.168.1.1 # 指定去往某个网络，都要经过1.1这个路由，并指定网卡进行转发
```

ip maddress
```bash
# 组播地址
ip maddress # 查看所有网卡都属于哪个组播地址
ip maddress ls ens33  # 查看指定网卡属于哪个组播地址
ip maddress add 33:33:00:00:00:01 dev ens33 # 把哪个网卡加入到某个组播地址
```

ip neighbor
查arp缓存
```bash
# 查看
ip neigh
ip -s -s neighbor show
# 手动绑定ip与mac地址（防止arp欺骗）  lladdr即逻辑链路层地址， perm是永久生效
ip neighbor add dev ens33 192.168.0.131 lladdr 00:11:22:33:44:55 nud perm
```

ip monitor
监视网络里面地址等解析的变化
```bash
ip monitor all
# 有着不同的状态
```



## <font color=red>Nginx</font>
- 开源，轻量，快速，可扩展的Web服务器
- Github, Netflix, Wordpress都基于Nginx
- 但是可配置能力弱于Apache
- 适用于大流量，高并发环境（稳定快速，占用资源少--C10K问题）
- 在Top 1000个网站中使用率最高

**Apache基于线程**
- 基于进程(运行中的文件就叫进程)消耗资源多
- 同进程下的多线程共享内存（一损俱损）

**Nginx使用事件驱动的架构**
- 新客户端请求时不创建新的进程/线程
- 事件处理器处理请求任务
- 更少的处理时间，更少的内存消耗

**Apache与Nginx共用**
- Apache处理动态内容
- Nginx处理静态内容


#### 安装
```bash
sudo apt install nginx

# 配置文件在 /etc/nginx
```


#### 配置文件
```bash
# /etc/nginx
# 主配置文件 nginx.conf

# 虚拟主机配置文件
sites-available
sites-enabled    # 启用的，是个链接

snippets    # 可以复用的片段
```

###### 主配置文件
```bash
# /etc/nginx/nginx.conf
user www-data;        # 用户
worker_processes auto;    # 进程数
include /etc/nginx/modules-enabled/*.conf;    # 引入的模块配置

events {
    worker_connections 768;  # 每个进程最大的并发连接数
    # multi_accept on;
}

http {
    sendfile on;    # 对内核 静态资源的访问速度(都要开着)
    tcp_nopush on;    # 优化了对tcp协议栈性能(优化了数据包的大小,多个小包变成大包再发)
    tcp_nodelay on;   # 与上面的参数相反(从发包的延时进行优化,来一个包就发)
    keepalive_timeout 65;        # 没一个TCP连接最大的保活的时间
    types_hash_max_size 2048;    # MIME类型镜头内容hash表大小

    ##
    # Virtual Host Configs
    ##
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

```

###### 虚拟主机配置文件
ip和端口一样，使用不同域名来指向不同站点
```bash
# /etc/nginx/sites-available/    # 可用站点
# /etc/nginx/sites-enabled/      # 已启用站点

# /etc/nginx/sites-available/default
server{
    listen 80 default_server;    # 侦听的端口
    root /var/www/htmll;    # 根目录
    index index.html index.htm index.nginx-debian.html;    # 索引文件
    server_name www.xpsshuai.cn;    # 主机名(域名)
}

location / {    # 主url
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;    # 资源没有就404
}
```


#### 新建虚拟主机(Server Blocks)实例
>Nginx中叫`服务器块`，但东西跟Apache中虚拟主机是一样的

```bash
sudo mkdir /var/www/qq.com
# 每个单独的Server Block一个配置文件
sudo cp /etc/nginx/sites-available/default  /etc/nginx/sites-available/qq.com
sudo vim /etc/nginx/sites-available/qq.com 
# 内容
server{
    listen 80;   # 因为这里都侦听80端口，所以为了和默认站点区分，要指定不同域名   
    server_name www.qqq.com;   
    root /var/www/qq.com;    
}
# 其他配置项可以取消注释自行配置
location ~ /\.ht {   # 这个建议启用，以..开头,以ht结尾的你我们就拒绝访问-->当然也可以增加更多策略
   deny all;
}


# 创建启用的符号链接(Apache是site2enable，这里没有专门的命令)
sudo ln -s /etc/nginx/sites-available/qq.com  /etc/nginx/sites-enabled/qq.com 
# 重启
sudo systemctl restart nginx

# 访问域名，就会访问到qq.com，因为请求中的host字段是不一样的

# 其他站点同上
```


#### Nginx支持PHP
WebServer本身只是提供资源的，必须配合php等脚本才能进行逻辑的运算

1.安装fastCGI process manager(FPM)  --> 就能配合Nginx配合进行处理(Nginx把接收到的请求转发给php处理)
```bash
sudo apt install php-fpm   # 安装

# 查看位置  ls /var/run/php/
/var/run/php/php7.2-fpm.sock  # 进程名称(记住路径)
# 验证 
php -v
    
```

2.配置Nginx
```bash
# 修改配置  sudo vim /etc/nginx/sites-available/qq.com
location ~ \.php$ {   # 匹配所有访问.php的请求
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;    # 意思是: 把所有访问.php的拓展名的资源，都转发给这里pass的php进程
    }
# 还有，索引页面也要添加 index.php
```

3.配置PHP
```bash
# sudo vim /etc/php/7.2/fpm/php.ini
# 这里只简单讲几个配置
cgi.fix_pathinfo=0       # 必须修改为0, 安全
file_uploads = On         # 如果不需要就关掉，看自己需求
max_file_uploads = 20    # 限制单个文件上传大小
allow_url_fopen = On     # 是否允许文件包含(如果没业务需要，建议关闭)
allow_url_include = Off
memory_limit = 128M      # 指定内存大小(最小128M,看业务需求)
```

4.重新加载
```bash
sudo nginx -t
sudo systemctl reload nginx
# 重启php
sudo systemctl restart php7.2-fpm
```

5.重新访问php页面，查看是否正确解析



#### Nginx支持SSL
**自签名证书** (自己测试或者不需要那么安全的情况下) --> 机密性和完整性可以保证，但是身份认证是不会被公网所信任的
```bash
# x509标准格式的, 私钥是绝对不能泄漏的, 后面是绑定的一个证书
sudo openssl req -x509 -days 365 -sha256 -newkey rsa:2048 -nodes -keyout /etc/ssl/private/qq.key -out /etc/ssl/certs/qq.pem
# 一旦域名绑定了证书，就必须通过这个域名来访问

# 若向CA证书颁发机构申请证书方法 同上Apache
```

服务块配置
```bash
# sudo vim /etc/nginx/sites-available/qq.com
server {
     listen 443 ssl;        # 443端口是 ssl, 通过https访问
     server_name www.qqq.com;
     ssl_certificate /etc/ssl/certs/qq.pem;
     ssl_certificate_key /etc/ssl/private/qq.key;

     root /var/www/qq.com
}

# 重启服务
systemctl restart nginx


# 访问https://xxxxxx    会发现浏览器会提示证书不信任
```

**Let's encrypt**
第三方的https的开源的免费证书颁发机构，向他来申请证书

1.安装
```bash
sudo add-apt-repository ppa:certbot/certbot  # 添加仓库
sudo apt update
sudo apt install cerbot    # 客户端程序(简化了申请流程)
sudo apt install python-cerbot-nginx
```

2.使用方法：
```bash
# 前提
域名正常解析、设置好虚拟主机

# 2.1申请证书 (--nginx意思是给nginx用的， -d是域名,还有子域名) , -m是管理员邮箱
sudo certbot --nginx -d example.com -d www.example.com -m admin@example.com

# 证书存放位置
/etc/letsencrypt/live

# 2.2修改自己配置文件中证书的位置
# sudo vim /etc/nginx/sites-available/qq.com
ssl_certificate


# 2.3 手动证书更新(因为默认证书有效期是90天)
sudo certbot renew --dry-run

# 2.4测试 查看证书状态(输入自己的域名) 可以查看到一些安全隐患等详细信息
https://www.ssllab.com/ssltest/
# ssl的信息扫描(不安全的地方)也可以使用自己本地的工具
```


#### Nginx反向代理
>很多情况下，Nginx不是作为WebServer使用的，最常见的就是作为`反向代理`

首先说一下`代理`:     代理客户端访问外网服务器。代理是重新生成的请求(比如公司内部限制员工上网的场景)
`反向代理`：
    - 代理服务器对外提供服务，接受客户端请求
    - 安全层过滤客户端恶意请求(云)
    - 可以放在防火墙外面，甚至放在云上(云WAF本质上就是反向代理, 请求通过DNS解析到云waf上，进行过滤)

- Apache不适用于高并发，高负载环境
- Nginx没有内建支持动态内容处理能力

**结合Apache/Nginx的优势**
- Nginx接受入站请求和静态内容缓存，动态请求发给Apache
- Apache完成动态内容处理
- Nginx作为反向代理（动态请求转发给Apache）
- Nginx放在WebServer前面，再将请求发给WebServer。然后服务端把数据通过Nginx向用户交付数据
- 不仅减缓了Apache的流量压力，也进行了一定的安全过滤


#### Nginx实现反向代理
>同一台服务器的流量转发

1.本机安装Apache
```bash
注意: 先停掉nginx的服务
systemctl stop nginx
再安装
sudo apt install apache2
```

2.修改Apache侦听端口
```bash
1.
# vim /etc/apache2/ports.conf
#让apache不占用80, 让nginx侦听80端口
 Listen 8080
<IfModule ssl_module>
   Listen 8443
</IfModule>
<IfModule mod_gnutls.c>
    Listen 8443
</IfModule>

2.修改apache虚拟站点默认侦听端口
vim /etc/apache2/sites-available/000-default.conf
也改为8080

3.重启服务
systemctl restart apache2
```

3.Nginx反向代理设置
```bash
# vim /etc/nginx/sites-available/default
#修改nginx配置，侦听80端口，然后动态部分的流量转发给apache,而静态文件还是放Nginx上读取(这里以php为例)
# 动态内容进行转发
location ~ \.php$ {
     include snippets/fastcgi-php.conf;
     # fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;   # 这里就不用nginx解析php了，用apache(配置在上面小结)
     proxy_pass  http://127.0.0.1:8080;  # 凡是php的动态内容都转发给本机的8080(即Apache)
     proxy_set_header X-Real-IP $remote_addr;  # 通常会添加一些代理请求头(便于区分)
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 这个头部虽然不是RFC标准，但是也是记录了经过了哪些代理地址
     proxy_set_header Host $host;
    proxy_set_header X-Forward-Proto $scheme;
    }
# 静态内容留给自己（增加一个location）      
location ~* \.(js|css|jpg|png|svg|html|htm)$ {
   expires 10d;   # 指定过期时间 10天
}
```

4.检查Nginx配置并重启
```bash
sudo nginx -t  # 检查配置语法
sudo systemctl restart nginx  # 重启
```

5.配置Apache支持php解析
```bash
# 1.安装
sudo apt install php7.2 libapache2-mod-php7.2   # mod-php模块
# 验证 
php -v

# 2.启用模块
a2enmod php7.2

# 3.索引文件
vim  /etc/apache2/mods-available/dir.conf       # 增加index.php

# 4.重启apache
sudo systemctl restart apache2
```

6.测试能否正常访问



#### 多台服务器的Nginx反响代理，进行流量转发
实验环境：一台ubuntu安装apache, 两台虚拟机安装Nginx
客户端向第一台Nginx服务器发送请求，转发给第二台Nginx服务器，然后转发给最后一台Apache
链式代理


**1.Nginx机器 N1**
1.安装Nginx
2.配置(这里做实验就简单地用了默认的配置了)
```bash
# /etc/nginx/sites-available/default
location / {   # 所有发到我这里的请求都转发给N2
    proxy_pass http://192.168.0.104;
}
```
3.重启Nginx


**2.Nginx机器 N2**
1.安装Nginx
2.配置
```bash
# /etc/nginx/sites-available/default
location / {   # 所有发到我这里的请求都转发给N2
    proxy_pass http://192.168.0.105;
}
```
3.重启Nginx

**3.Apache机器 A3**
安装apache
然后客户端访问N1, 就会看到A3的页面




#### Nginx负载均衡
> 基于反向代理实现负载均衡

**负载均衡：** 前面放一个Nginx进行反向代理，后面放多个Apache作为WebServer

**基本配置**
```bash
# 定义一个服务器组(要转发到的目标组)
upstream srvs {  # upstream只是一个组的名称
    least_conn;        # 负载均衡方法规则 --最少连接数优先 (详细见下面)
    server 192.168.106:80  weight=1;  # server 是后面的一个WebServer(apache或Nginx)的地址， weight 是权重(越大接收的流量就越多)
    server 192.168.106:80  weight=2;  
}

# 配置
server {
    location / {
        proxy_pass http://srvs;   # 上面定义的组的名称(不需要被DNS解析，仅仅是一个标识)
    }
}

```

**负载均衡方法**
```bash
round-robin       # 论询
least_conn        # 最少连接数优先(已经建立的连接数)
least_time        # 响应速度最快
hash              # 基于请求的Hash值
ip_hash           # 基于ip地址的hash值(适用于需要会话保持的场景)
```



#### 日志
```bash
/var/log/nginx/access.log
/var/log/nginx/error.log
```







## <font color=red>Tomcat</font>
Tomcat是由Apache组织开发的一个Servlet(Server Applet)容器
- 实现对servlet, JSP, JAVA Socket等支持
- 包含http服务器
- 作为中间件使用
- 可与Nginx结合使用(只用Tomcat的话, 抗压能力强)


servlet是优于CGI的方式的

###### 安装 open jdk
一般tomcat和java配合较多
jdk是开发环境，jre是运行环境
```bash
sudo apt install default-jdk  #安装openjdk。 使用默认版本 或者指定openjdk-8-jdk      
```

###### 安装 oracle jdk
安装
```bash
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:webupd8team/java
sudo apt install oracle-java8-installer -y
```

java版本选择
```bash
sudo update-alternatives --config java
sudo update-alternatives --config javac

java --version
```

或者手动下载oracle jdk包



###### 官方库安装tomcat
```bash
# ubuntu中收录的8
sudo apt install tomcat8
sudo apt install tomcat8-docs
sudo apt install tomcat8-admin
sudo apt install tomcat8-examples

# tomcat默认不是主要作为WebServer的，一般是配合Apache和Nginx，所以默认使用8080端口
```


###### 手动安装tomcat
适用于指定具体版本
1.先安装jdk
```bash

sudo apt install default-jdk  #安装openjdk。 使用默认版本 或者指定openjdk-8-jdk   
# 验证
java --version

sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat       # 创建tomcat帐号（home放在opt下面）
```

2.再下载安装包
<img src="http://image.xpshuai.cn/20200428150102394_2075358316.png" />

3.移动到/opt/tomcat目录
```bash
mv apache-tomcat-8.5.31 /opt/tomcat/
```

4.修改该文件递归的属主属组改为新建的`tomcat`用户
```bash
# 建议建立符号链接 
sudo ln -s /opt/tomcat/apache-tomcat-8.5.31 /opt/tomcat/latest  # 把新版本添加链接即可(版本更新之后不需要更改配置文件，配置文件都配置latest,而只需把latest修改指向即可)

sudo chown -R tomcat:/opt/tomcat
sudo chmod +x /opt/tomcat/latest/bin/*.sh
# sudo chmod +x /opt/tomcat/apache-tomcat-8.5.31/bin/*.sh
```


5. 手动创建systemd管理文件(手动安装就没有systemd的后台管理文件)
`/etc/systemd/system/`
`sudo vim tomcat.service`
配置内容如下
<img src="http://image.xpshuai.cn/20200428152651153_2000734872.png" />


6.启动服务
```bash
# 重新reload
sudo systemctl daemon-reload
# 启动tomcat
sudo systemctl start tomcat
sudo systemctl status tomcat
# 设置开机启动启动
sudo systemctl enable tomcat
```

7.Web管理接口配置
```bash
# http://ip:8080/manager/
# tomcat管理页面，默认是只允许本机访问的
## 1.若想开启，就需进行配置（这里指定允许某个ip访问）
sudo vim /opt/tomcat/latest/conf/tomcat-user.xml       # 自动安装tomcat后的路径在/etc
<tomcat-user>
    <role rolename="admin-gui"/>    # 定义两个角色
    <role rolename="manager-gui"/>
    <user username="admin" password="qwe123" roles="admin-gui, manager-gui"/>   # 定义用户
</tomcat-user>

## 2.指定登录主机
# sudo vim /opt/tomcat/latest/webapps/manager/META-INF/context.xml   #  自动安装tomcat后的路径自己找
如下图(懒得弄格式了)

# sudo vim /opt/tomcat/latest/webapps/host-manager/META-INF/context.xml
这是管理主机的配置文件

## 3.重启服务，然后测试
systemctl restart tomcat8
xxx:8080/manager/html        # 管理页面就可以部署war包等操
xxx:8080/host-manager/html
```


<img src="http://image.xpshuai.cn/20200428154603944_415659665.png" />






















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu-server-web%E6%9C%8D%E5%8A%A1/  

