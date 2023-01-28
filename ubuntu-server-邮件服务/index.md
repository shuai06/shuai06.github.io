# ubuntu sever-邮件服务




>直到今天，Emai仍然是互联网上最重要的信息传递方式之一


## <font color=red> 电子邮件基础</font>

快速、便捷、通用、异步通信方式
1965年最早由MIT发明并开源（最早用于不同主机用户间通信）
1971年确定邮件地址以`@`符号连接（在不同主机之间传递信息）
MIME类型扩展邮件从纯文本到传递文件（文档、图片、媒体）

邮件可被伪造，邮件炸弹，垃圾邮件

**传统物理邮件：**
写信->到本地邮局->目的邮局->收件人

**电子邮件**
Client1 -> Server1 -> Server2 -> Client2 （邮件地址通过DNS解析完成邮件路由）
无收件人返回报错（不通知），连不上目标服务器周期重发（直至失败）


#### SMTP（Simple Mail Transfer Protocol）
- MTA(Mail transfer agents) 之间通信协议（SMTPD）
    - 由很多小程序实现MTA所有功能
- Sendmai、Postfix、Exim、Qmail
- 客户端发起邮件 MUA -> MTA  也是用SMTP协议


#### MTA(Mail Transfer Agent)
常见的MTA软件：
###### 1. Sendmial
>一个MTA软件包

- 处理着绝大多数互联网电子邮件
- 开源免费版 / 商业版（GUI页面）
- 配置过于`复杂`（默认配置已经可以良好工作）
- 功能全面但性能不占优势


###### 2. Postfix
- 来自于IBM的开源MTA（`Ubuntu默认首选MTA`）
- 速度优于Sendmail，配置管理更简单（配置文件、命令）
- 可以平滑替换Sendmail（模块化）


###### 3. Qmail
- Postfix的可替代选择（模块化的MTA，用户、配置相分离）
- 设计初衷是实现取代Sendmail的更安全快速的MTA
- 从Sendmail迁移到Qmail过程复杂
- 众多插件，包含WebMail、POP服务


###### 4. Exim
- 比Sendmail、Postfix更安全快速，但配置方式不同
- 与Qmail都使用maildir格式邮箱存储


#### 邮箱存储
###### MBOX
- 传统的mbox格式邮箱存储将`所有邮件存于一个文件中`（索引定位具体邮件）
- 文件和索引损坏可能造成邮件丢失，`故障恢复困难`
- 不适于存储于NFS挂载目录中


###### MAILDIR
- Maildir格式包含三个子目录`/cur`,`/new`,`/tmp`， 每个邮件分隔独立保存
- 可并行访问，性能更高（mbox不支持）
- 可用NFS作为邮箱存储


###### MTA可直接投递和读取文件
    更多时候邮件投递由MDA完成


#### MDA(Mail Delivery Agent, 邮件投递代理)
MDA比MTA`额外实现`自动过滤、回复、规则转存、杀毒等
但邮件的`收`还是靠MTA


常见的MDA软件：
###### 1. Mailx
- MDA + MUA（`Ubuntu的默认MDA`）
- 来自MTA的邮件交给Mailx投递到用户邮箱（`/var/mail/`）
- 主目录下：`.forward`文件实现自动转发（如：公司邮箱的邮件，通过.forward进行转发到私人邮箱）


###### 2. main.local
- 常用于BSD的MDA程序
- 功能使用与Mailx类似



###### 3. procmail
- 最流行的MDA之一
- Recipes允许用户自定义邮件处理脚本（`.procmailrc`）



#### MUA(Mail User Agent)
- 读取和展示邮箱中的邮件（不金慈宁宫邮件传递）
- 在哪读邮件/邮件显示格式（MIME）


常见类型的MUA：
###### 1.Mailx
MUA + MDA
服务器本地直接读取邮箱mail


###### 2.Dovecot
>远程客户端访问邮箱

- POP3(Post Office Protocol version 3)
     适合单一客户端，单项通信协议(MUA <- MDA)
- IMAP4(Internet Message Access Protocol 4)
     适合多邮件客户端，双项同步协议( MUA <--> MDA)


>最基本的：MTA+MDA+MUA

典型命令：
![](http://image.xpshuai.cn/ubuntu_mail_cmd.png)


###### 3. 其他邮箱访问方式
    Web Mail
    WAPI（ 微软专有邮箱协议）


###### 4. Fetchmail
    自己本地邮箱服务器 <- 运行商邮件服务器(POP3 / IMAP)
    适用于每月稳定互联网连接的小公司



## <font color=red>电子邮件架构</font>
#### 邮件结构
- 信封：用于邮件路由，对用户透明不可见
- 邮件头：邮件路由传输过程的记录日志（如果遇到不专业的小黑客，可以通过邮件头来溯源）
- 邮件体：邮件要传递的具体内容

#### SPAM
- 垃圾邮件，不请自来的邮件（Junk）
- 处理浪费时间、网络带宽、存储空间

#### 病毒、木马、蠕虫、钓鱼邮件
- 包含恶意附件、链接、伪装、诈骗内容
- 存在安全风险，谨慎打开陌生邮件


#### <font color=red>邮件规范</font>
- `收件人`为事件直接相关人（其他相关人作为`抄送人`）
- 次要相关人按职位顺序  CC(`抄送人`，收件人能看见我发给抄送人，`按职位顺序排列`)、BCC（`密送人`，收件人是不知道我发给密送人了）
- 主题简介清晰，高度概括（`切忌把正文具体内容写到主题`）
- 保证表述清楚的前提下邮件内容尽可能短（`避免语法错误`），不要使用模棱两可/时髦语言
- 适当使用表情符
- 回复、转发完整邮件流（`禁止单独新写邮件作为回复`）
- 压缩附件
- 附个人签名（公司的信息，职位，姓名 -> 或弄成二维码->不太建议，建议直接用`文本形式`）

>沟通是所有企业面对的最大问题

 

## <font color=red>Postfix简单部署</font>
MTA(能发出邮件最关键的组件)

1.先安装DNS服务器
```bash
sudo apt install bind9
```
2.配置
```bash
# sudo vim /etc/bind/named.conf.options
    # 自己解析不了的域名，转发给互联网上其他的
    forwarders {
        8.8.8.8;
    }
  

# sudo vim /etc/bind/named.conf.local
    # 自己的域(可以是任何域名，虽然被别人占用的。可以用来发送恶意文件)
    zone "qq.com"{
        type master;
        file "/etc/bind/db.qq.com";    # 区域文件
    }
  

# cp /etc/bind/db.local.local
# sudo vim /etc/bind/db.qq.com
$TTL    604800
@       IN      SOA     ns.qq.com. root.qq.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns        # 创建NS记录
@       IN      MX  5   mail      # 创建MX记录，优先级(值越大，优先级越低)
ns      IN      A       192.168.0.19    # NS的记录
mail    IN      A       192.168.0.19    # MX的记录
pop     IN      A       192.168.0.19
imap    IN      A       192.168.0.19
smtp    IN      A       192.168.0.19
@       IN      A       192.168.0.19    # @代表本域域名自己

```

3.重启bind服务
```bash
sudo systemctl restart bind9
```

4.网络配置（配置静态ip）
```bash
# sudo vim /etc/netplan/50-cloud-init.yaml
    dhcp4:no
    addresses:[192.168.1.2/24]
    gateway4:192.168.1.1
    nameservers:
            search:[qq.com]
            addresses:[192.168.1.2]    # 指向邮箱服务器的ip


sudo netplan apply
sudo networkctrl status
```

5.修改主机名
```bash
#sudo vim /etc/hostname
    mail.qq.com

# 直接修改hostname后，重启后会变回去，所以修改下面的配置文件

#sudo vim /etc/cloud/cloud.cfg
    preserve_hostname:true

```

6.安装Postfix
```bash
sudo apt install postfix

向导窗口，选择：
Internet Site    # 一般选这个就好
Internet Site  smarthost    # 智能主机
操作系统的主机名    # mail.qq.com

grep mail /etc/group    # 会自动生成一个用户组mail
grep postfix /etc/group    # 会自动生成一个用户组postfix
# 但是postfix默认使用操作系统的用户，要加入本地用户到相应组(只有该用户组的成员才可发邮件)
sudo usermod -aG mail xps    
# sudo useradd -m -G mail xxx2
# sudo passwd xxx2
# mail           # mail命令默认不安装，先安装
sudo apt install mailutils    # 安装
mail    # 再使用

mail xps@qq.com    # 主要收件人
cc:  xps2@163.com    # 抄送人
Subject: test2    # 主题
xxxxxx正文

# ctrl + D 发送邮件

su xxx2        # 切换到该用户查看该用户收到的邮件
mail           # 可以查看该用户收到的邮件

单机邮件一般存在/var/mail/用户名

# 一般大的邮箱服务商，如QQ，163等，会反查发件人邮件的地址是否是合法的
```

>目前，只能发邮件，不能收邮件



## <font color=red>企业级Postfix邮件系统部署</font>
#### 组件
- Postfix: MTA
- Dovecot: POP /IMAP服务
- SASL: 身份认证
- Postfixadmin: 管理邮件、虚拟域、别名等的WEB应用
- LEMP：Postfixadmin的web环境依赖


#### Postfix的两种域
- Local Domain：为本系统账号投递邮件(`/etc/passwd`)
- Virtual Domain：为非本系统账号投递邮件


###### 1.安装LEMP
```bash
sudo apt install nginx
sudo apt install mysql-server

sudo mysql_secure_installation
sudo mysql
ALTER USER 'root'@'localhost' INDETIFIED WITH mysql_native_password BY '1234';    # 修改登录方式
FLUSH PROVILEGES;
# mysql -u root -p1234

sudo apt install php-fpm php-mysql
sudo vim /etc/php/7.2/fpm/php.ini
    cgi.fix_pathinfo=0
    date.timezone = Asia/Shanghai


## 验证能否正常使用
sudo vim /etc/nginx/sites-avaliable/default
    index index.php
    server_name mail.qq.com;
    location ~\.php${
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }
    location ~/.ht{
        deny all;
    }

sudo vim /var/www/html/info.php
    <?php phpinfo(); ?>


systemctl restart nginx.service
systemctl restart php7.2-fmp.service
```

###### 2.安装Postfixadmin
推荐手动下载源码，然后解压
```bash
# 安装依赖包
sudo apt install php-imap php-mbstring php7.2-imap php7.2-mbstring
# 下载源码
sudo wget -P /opt https://github.com/postfixadmin/postfixadmin/archive/postfixadmin-3.2.4.tar.gz
cd /opt && tar -xvf postfixadmin-3.2.tar.gz
mv postfixadmin-postfixadmin3.2/ postfixadmin
# /opt/postfixadmin/public/setup.php    # 做web界面配置的文件
# 软链接目录
ln -s /opt/postfixadmin/public/ /var/www/html/pfa

```

###### 3.配置数据库
```bash
# mysql -u root -p
# 创建数据库和名字
CREATE DATABASE postfix_db;
CREATE USER 'post_user'@'localhost' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON postfix_db.* TO 'post_user'@'localhost'
FLUSH PRIVILEGES;
```

Postfixadmin 数据库连接文件
```bash
cd /opt/postfixadmin/  && cp config.inc.php config.local.php
sudo vim config.local.php
    $CONF['configured'] = true;
    $CONF['default_language'] = 'cn';
    # 数据库连接信息
    $CONF['database_type'] = 'mysqli';
    $CONF['database_host'] = 'localhost';
    $CONF['database_password'] = '1234';
    $CONF['database_name'] = 'postfix_db'

```

###### 4.创建临时目录
```bash
mkdir /opt/postfixadmin/templates_c && chmod 755 -R /opt/postfixadmin/templates_c
chown -R www-data:www-data /opt/postfixadmin/templates_c

```


###### 5.WEB安装阶段
```bash
# 访问前面设置的url和文件
http://mail.qq.com/setup.php

设置 setup password
生成 setup password 的hash值，复制下来，添加到服务端中配置项
# vim config.local.php
$['setup_password'] = 'xxxxxxx hash xxxxxxxx'

然后创建管理员账号与密码
登录管理员账号

```

**创建虚拟域、邮箱账号**
- 域名清单 -- 新建域
- 虚拟用户清单 -- 新建邮箱（别名）

<img src="http://image.xpshuai.cn/ubuntu_mail_virtual.png"></img>

<img src="http://image.xpshuai.cn/ubuntu_mail_virtual2.png"></img>

<img src="http://image.xpshuai.cn/ubuntu_mail_virtual3.png"></img>


###### 6.集成Postfix
>以下命令以root权限运行
```bash
apt install Postfix Postfix-mysql sasl2-bin    # sasl来实现身份认证
```

配置sasl自动启动
```bash
# vim /etc/default/saslauthd
    START=yes

systemctl restart saslauthd.service
```

邮箱目录、访问账号、组
```bash
# 添加组账号vmail
groupadd -g 5000 vmail && mkdir -p /var/mail/vmail
# 创建账号vmail, 属于组vmail
useradd -u 5000 vmail -s /usr/sbin/nologin -d /var/mail/vmail
chown -R vmail:vmail /var/mail/vmail
```

###### 7.数据库配置(3个)
数据库配置文件1
```bash
mkdir -p /etc/postfix/sql

# 去数据库中找虚拟域的域名
vim /etc/postfix/sql/mysql_virtual_domains_maps.cf
    user = post_user
    password = 1234
    hosts = 127.0.0.1
    dbname = postfix_db
    query = SELECT domain FROM domain WHERE domain='%s' AND active='1'


# 想要生效，配置加入 /etc/postfix/main.cf
postconf -e virtual_mailbox_domains=mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf    # 使用上面配置的配置文件,来查询虚拟域
postmap -q qq.com mysql:/etc/postfix/sql/mysql_virtul_domains_maps.cf    # 验证
```


数据库配置文件2
```bash
# 查邮箱
vim /etc/postfix/sql/mysql_virtual_mailbox_maps.cf
    user = post_user
    password = 1234
    hosts = 127.0.0.1
    dbname = postfix_db
    query = SELECT maildir FROM mailbox WHERE username='%s' AND active='1'


# 想要生效，配置加入 /etc/postfix/main.cf
postconf -e virtual_mailbox_domains=mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf    # 使用上面配置的配置文件
postmap -q tom@qq.com mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf    # 验证
```



数据库配置文件3
```bash
# 查别名
vim /etc/postfix/sql/mysql_virtual_alias_maps.cf
    user = post_user
    password = 1234
    hosts = 127.0.0.1
    dbname = postfix_db
    query = SELECT goto FROM alias WHERE address='%s' AND active='1'


# 想要生效，配置加入 /etc/postfix/main.cf
postconf -e mysql_virtual_alias_maps=mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf    # 使用上面配置的配置文件
postmap -q abuse@qq.com mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf    # 验证
```


权限，属主
```bash
chgrp postfix /etc/postfix/sql/mysql_*
```

>至此，postfixadmin与postfix MTA安装完成

###### 8.启用SASL 强制 Postfix发邮件使用Dovecot进行`身份验证`
```bash
# vim /etc/postfix/main.cf

smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $myhostname
smtpd_tls_security_level = may
smtpd_tls_auth_only = no
smtpd_recipient_restrictions = permit_mynetworks permit_sasl_authenticated reject_unauth_destination
```

###### 9.启动安全SMTP端口
```bash
# vim /etc/postfix/master.cf
submission    inet    n     -    y    -    -    smtpd
-o syslog_name=postfix/submission
-o smtpd_tls_security_level=encrypt
-o smtpd_sasl_aut_enable=yes
-o smtpd_client_restrictions=permit_sasl_authenticated,reject
-o milter_macro_daemon_name=ORIGINATING

smtps    inet    n    -    y    -    -    smtpd
-o syslog_name=postfix/smtps
-o smtpd_tls_wrappermode=yes
-o smtpd_sasl_aut_enable=yes
-o smtpd_client_restrictions=permit_sasl_authenticated,reject
-o milter_macro_daemon_name=ORIGINATING

```

###### 10.验证配置
```bash
# 验证配置
postcnf -n

# 重启服务
sytemctl restart postfix

# 证书可自行替换
```


###### 11.安装/配置Dovecot
```bash
# 安装Dovecot
apt install dovecot-imapd dovecot-mysql dovecot-managesieved dovecot-pop3d

# Sieve
负责创建并将邮件自动放入指定用户文件夹
```

配置Dovecot（多配置文件）
```bash
cd /etc/dovecot/conf.d/ && vim 10-auth.conf    # 系统账号/SQL账号
    auth_mechanisms = plain login
    #!include auth-system.conf.ext
    !include auth-sql.conf.ext    # 使用这个配置文件来访问数据库

```

配置SQL账号
```bash
# vim /etc/dovecot/conf.d/auth-sql.conf.ext
passdb {
    driver = sql
    args = /etc/dovecot/dovecot-sql.conf.ext    # 连接数据库文件
}

userdb {    # 用户数据库
    driver = static
    args = uid=vmail gid=vmail home=/var/mail/vmail/%d/%n
}

```

连接数据库配置文件
```bash
# vim /etc/dovecot/conf.d/dovecot-sql.conf.ext
driver = mysql
connect = host=127.0.0.1 dbname=postfix_db password=1324
password_query = SELECT usernme,domain,password FROM mailbox WHERE username='%u',default_pass_schema=MD5-CRYPT

```

指定邮箱存储位置
```bash
# vim /etc/dovecot/conf.d/10-mail.conf
mail_location = maildir:/var/mail/vmail/%d/%n/Maildir
mail_privileged_group = mail

```

配置服务连接
```bash
# vim /etc/dovecot/conf.d/10-master.conf
service auth {
    unix_listener auth-userdb {
        mode = 0600    # 权限掩码
        user = vmail
        group = vmail
    }
    unix_listener /var/spool/postfix/privilege/auth {
        mode = 0660
        user = postfix
        group = postfix
    }
    user = dovecot    # 修改为我们指定的dovecot永固
}

```

配置sieve
```bash
# vim /etc/dovecot/conf.d/15-lda.conf
protocol lda {
    mail_plugins = mail_plugin sieve    # sieve组件，会帮我们自动创建文件夹
}

chgrp vmail /etc/dovecot/dovecot.conf    # vmail执行Dovecot

```


重启服务
```bash
systemctl restart dovecot
```

查看日志
```bash
tail -n 20 -f /var/log/mail.log
# 可以看到前面的服务启动正常
```


###### 12.集成Postfix与Dovecot
```bash
# vim /etc/postfix/master.cf
dovecot    unix    -    n    n    -    -    pipe
flags = DRhu user=vmail:vmail argv=/usr/lib/dovecot/deliver -f ${sender} -d xxxxxx



# vim /etc/postfix.main.cf
virtual_transport = dovecot
dovecot_destination_recipient_limit=1


## 重启服务
systemctl restart dovecot

tail -n 20 -f /var/log/mail.log
# 可以看到前面的服务启动正常
```

###### 13.测试
发送邮件
```bash
使用postfixadmin 的web界面来发送邮件

# 或者 使用mail发送邮件
apt install mailutils
echo "hello xpshuai" | mail "test mail" tom@qq.com

```

查看结果
```bash
tail -n 20 -f /var/log/mail.log
ls -l /var/mail/vmail    # 每个域的邮件存储的目录
```

增加其他虚拟域、邮箱(`jerry@taobao.com`)
```bash
echo "hello jerry, it's tom" |mail -s "Subject"-a From"tom\<tom@qq.com>\jerry@taobao.com
```


###### 配置邮件客户端
Outlook,Thunderbird, ... ...
添加新邮箱账号（名字，地址，密码）
配置Imap、pop3、smtp等的地址，SSL加密




## <font color=red>Web MAIL安装</font>
各大邮件服务商全部提供Web Mail
基于浏览器的统一使用体验（无序配置客户端）

`Roundcube` -- 一款非常流行且稳定的Web Mail应用

###### 1.安装依赖包（可选）
```bash
apt install php7.2-intl php-imagick php7.2-ldap
```

###### 2.下载Roundcube
```bash
cd /var/www/html && wget https://github.com/roundcube/roundcubemail/releases/download/1.4.7/roundcubemail-1.4.7-complete.tar.gz
tar -xvf roundcubemail-1.4.7-complete.tar.gz
mv roundcubemail-1.4.7-complete webmail && cd webmail

```

###### 3.创建数据库
```bash
CREATE DATABASE roundcubedb;
CREATE DATABASE routecubemail;
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON roundcubedb.* 'roundcube'@'loocalhost';
GRANT ALL PRIVILEGES ON routecubemail.* 'roundcube'@'loocalhost';
FLUSH PRIVILEGES;
```

导入数据
```bash
# 当前目录下的SQL文件夹下的sql文件（针对不同类型的数据库进行导入）
mysql -u roundcube -p roundcubedb < SQL/mysql.initial.sql

```

###### 4.配置站点文件
```bash
# 这里使用默认的配置文件
vim /etc/nginx/sites-avaialable/default

location /webmail {
    root /var/www/html;s
    index index.php;
    location ~ ^/webmaik/(.+\.php)$ {
        root /var/www/html;
        try files $uri=404;
        fastcgi_index index.php;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root $fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
    }
    location ~ ^/webmail/(README|INSTALL|LICENSE|CHANGELOG|UPGRADING)$ {
        deny all;
    }
    location ~ ^/webmail/(bin|SQL|config|temp|logs)/ {
        deny all;
    }
}

```

检查配置
```bash
nginx -t
```

重启服务
```bash
systemctl restart nginx
```

###### 5.权限
```bash
chown -R www-data:www-data /var/www/html/webmail
chmod 755 /var/www/html/webmail/temp  /var/www/html/webmail/logs

```

###### 6.Web安装阶段
```bash
# 访问路径
http://mail.qq.com/webmail/installer

## 检查环境
# 一些组件可能没安装，可以安装
apt install php-dompdf
apt install php7.2-xml

## 创建配置文件
...
# IP检查
# 设置数据库密码
# managesieve
# ...具体的自己看配置

## 初始化数据库

```

删除installer目录

登录Web Mail
```bash
## 访问web界面
http://mail.qq.com/webmail

```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1/  

