# ubuntu server-Zabbix




>Zabbix -- 最流行的开源的企业级监控系统之一

- 基本可对全平台，全系统做监控：网络设备、操作系统、应用程序
- Unix、L inux、Windows、MacOS
- 监控方式：Agent(硬件设备,被监控机器上安装，自己的作为Zabbix服务器) / Agenless

#### 系统环境

- 修改：IP(静态)、Hostname、 DNS、NTP
```bash
# 修改主机名
vim /etc/cloud/cloud.cfg
    preserve_hostname；true

vim /etc/hostname
    zab.lab.com    # 作为zabbix服务器

# 域名解析
10.1.8.132 zab.lab.com

# 重启zabbix服务器

```

#### 安装&配置
###### 1.安装配置Apache

```bash
# 管理是Web界面，在Zabbix服务器机器上
apt install apache2

vim /etc/apache2/conf-enabled/security.conf
    ServerTokens Prod    # 显示最少的主机头信息

vim /etc/apache2/apache2.conf
    ServerName zab.lab.com    # 主机名

vim /etc/apache2/sites-enabled/000-default.conf
    ServerAdmin webmaster@lab.com

# 重启服务
systemctl restart apache2
```

###### 2.安装php

```bash
apt install php php-cgi libapache2-mod-php php-common php-pear php-mbstring
# 启动模块
a2enconf php7.2-cgi
systemctl reload apache2

vi /etc/php/7.2/apache2/php.ini
    date.timezone = "Asia/Shanghai"

# 测试页面   
vi /ar/www/htm/index.php
    <html> <body> <?php print "PHP Test"; ?></body></html>

```

###### 3.安装MySQL

```bash
apt install mariadb-server
mysql_secure_installation    # 安全初始化

mysql 
    CREATE DATABASE zabbix_db CHARACTER SET utf8 collate utf8 bin;
    grant all privileges on zabbix_db.* to zabbix@'localhost' identified by 'pass123';
    grant all privileges on zabbix_db.* to zabbix@'%' identified by 'pass123';
    flush privileges;

```

###### 4.安装Zabbix

```bash
# 下载适合ubuntu发行版本(这里是bionic)的二进制包（根据自己的下载）
wget https://repo.zabbix.com/zabbix/4.0/ubuntu/ pool/main/z/zabbix-release/zabbix-release_ 4.0-3+bionic_ all.deb

apt install ./zabbix-release_4.0-3+bionic_all.deb && apt update    # dpkg -i xx 也可；增加了一条zabbix官方源的文件

sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-agent    # 这里zabbix-agent安装是因为把zabbix-server自己也纳入监视范围

zcat /usr/share/doc/ zabbix- server-mysql/create.sql.gz | mysql -u zabbix -p zabbix_db    # zcat 可以查看压缩文件里面的内容；初始化表结构
# 修改zabbix服务端配置文件中数据库连接密码
vi /etc/zabbix/zabbix_server.conf
    DBName=zabbix_db
    DBUser=zabbix
    DBPassword=pass123
# 重启服务
systemctl restart zabbix-server
```

###### 5.配置Zabbix

```bash
# 配置agent（这里是服务端监视自己的agent）
vi /etc/zabbix/zabbix_agentd.conf
    Hostname= zab.lab.com    # 让他找到我们的zabbix server的地址

# 重启
systemctl restart zabbix-agent
systemctl enable zabbix-server    # 开机自启

vi /etc/ apache2/conf-enabled/zabbix.conf
    Allow from 10.0.0.10/24    # 设置谁能访问我zabbix的管理页面

# 重启apache
systemctl restart apache2

```

###### 6.访问Web&初始化

```bash
# htp://zab.lab.com/zabbix/
# 设置数据库连接参数
DB Password
Hostname

# 登陆: Admin / zabbix
# 修改密码
# 添加监视的主机（agent模式下，目标端口要开）

```
至此，zabbix服务端已经配置完毕



###### 7.在被监控服务器上安装配置agent

```bash
## 安装
wget hts://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release.4.0-3+bionic.all.deb
sudo apt install ./zabbix-release_4.0-3+bionic.all.deb
sudo apt update
sudo apt install zabbix-agent    # 安装zabbix-agent


# 修改agent服务器的主机名 和 DNS域名解析

# PSK / CA加密服务器与客户端之间的通信

## 配置
vi /etc/zabbix/zabbix_agentd.conf
    Server=zab.lab.com    # 指向zabbix-server
    ServerActive=zab.lab.com
    Hostname= zab.lab.com
    TLSConnect=psk    # 这里设置psk加密的参数
    TLSAccept=psk
    TLSPSKIdentity=PSK 001    # ID必须唯一
    TLSPSKFile=/etc/zabbix/zabbix_agentd.psk
    
# 随机生成文件内容 psk的值
openssl rand -hex 32 | sudo tee /etc/zabbix/zabbix_agentd.psk

sudo systemctl start zabbix-agent
sudo systemctl enable zabbix-agent

```

###### 8.服务端添加agent

```bash
# Web界面
Configuration -> Hosts -> Create host -> ... -> Template(指定监视目标的什么服务) -> Security(都选择PSK, agent的id, psk的值)

```























---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu-server-zabbix/  

