# ubuntu server-LAMP




## <font color=red>LAMP环境安装WordPress</font>
#### WEB应用程序基础套件

- 操作系统( Linux )
- WEB服务器( Apache、Nginx )
- 数据库( MySQL、MariaDB、 PostgreSQL)
- 应用开发语言( PHP、Python、 Perl )

#### 大量开源应用基于LAMP开发

- Wordpress
- Wiki
- Owncloud
- Phpmyadmin
- ... ...





#### 安装LAMP(Linux + Apache + MySQL +PHP)

自动安装
```bash
# 一次性安装 tasksel    简单快捷
apt install tasksel

# 运行工具
sudo tasksel
# 选择安装包,q 勾选，确定即可

#测试
echo "<?php phpinfo();?>" > /var/www/html/info.php
```

手动安装
```bash
apt install apache2
apt install mysqk-server
    mysql_secure_installation
apt install php libapache2-mod php-mysql

#测试
echo "<?php phpinfo();?>" > /var/www/html/info.php
```

#### WordPress
- 开源免费的个人博客系统
- 全球1/3的个人博客采用
- 基于LAMP开发
- 开箱即用

**安装**
```bash
wget http://wordpress.org/latest.tar.gz

sudo tar -zxvf latest.tar.gz -C /var/www/html
# 设置权限
sudo chown www-data:www_data -R /var/www/html/myshop
# 还没有数据库
```

**创建数据库**
```sql
-- 进入sql命令行（建议用户名密码和库名不要这么简单易猜）
create database wpdb;
create user 'wpuser'@'localhost' identified by '12345678';
grant all on wpdb.* to 'wpuser'@'localhost' identified by '12345678' with grant option;
flush privileges;
```


**配置文件**
```bash
cd /var/www/html/wordpress/
cp wp-config-sample.php wp-config.php    # 从模板文件复制
vim wp-config.php

# 修改库名，用户，密码
# 他会自动写进去表
```


**虚拟主机**
不建议使用默认站点
```bash
cd /etc/apache2/sites-available/
cp 000-default.conf  myblog.conf
    vim myblog.conf
    ServerName www.myblog.com
    DocumentRoot /var/www/html/wordpress

# 其他安全配置
```
**启用站点**
```bash
a2ensite myblog.conf
a2dissite 000-default.conf    # 禁用默认站点

systemctl reload apache2
```

**Web配置**
访问设置的域名`http://myblog.com`


**改为中文**
从官网下载中文语言包解压到`/wp-content/`目录下，
修改`wp-config.php`
```php
define('WPLANG', 'zh_CN')
```

或者直接下载中文版`https://cn.wordpress.org`






## <font color=red>PHPMyAdmin & 电商网站搭建</font>
#### phpMyAdmin
    基于Web应用的MySQL管理应用, 本身也是一个应用程序


1.修改身份验证方式
```sql
# 进入sql终端
select user,host,plugin from mysql.user;

"auth_socket"  --> "mysql_native_password"
```


2.安装PhpMyAdmin
```bash
apt install phpmyadmin
```

3.修改`/etc/apache2/apache2.conf`主配置文件
```bash
# 包含phpmyadmin的配置文件
Include /etc/phpmyadmin/apache.conf
```

测试 `http://wwwwww/phpmyadmin/`



#### 电商网站搭建
安装`prestashop`开源的电商平台
- 开源免费
- 全球有25万台电商平台使用
- 基于LAMP套件

1.安装依赖包
```bash
sudo apt install php7.2-gd php7.2-xml php7.2-curl php7.2-zip
```

2.下载
```bash
wget https://download.prestashop.com/download/releases/prestashop_1.7.4.1.zip
sudo unzip prestashop_1.7.4.1.zip  -d /var/www/html/myshop
sudo chown www-data:www_data -R /var/www/html/myshop
```

3.创建数据库
```sql
sudo mysql
-- 进入sql命令行（建议用户名密码和库名不要这么简单易猜）
create database myshop;
create user 'myshop'@'localhost' identified by '12345678';
grant all on myshop.* to 'myshop'@'localhost'  identified by '12345678' with grant option;
flush privileges;
```


4.虚拟主机(不建议使用默认站点)
```bash
cd /etc/apache2/sites-available/
cp 000-default.conf  myshop.conf
    vim myshop.conf
    ServerName www.myshop.com
    DocumentRoot /var/www/html/myshop

# 这里只配置了最简单的，其他的配置按照前面的章节自行配置
```
5.启用站点
```bash
sudo a2enmod rewrite   # 启用这个模块
sudo a2ensite myshop.conf
#a2dissite 000-default.conf    # 禁用默认站点

systemctl reload apache2
```

6.访问站点，进行Web配置
表名的前缀不要默认，否则如果有漏洞，容易被猜解
安装完成之后，要删除`install`文件

>这个网站源码是免费的，但是主题是收费的......且支付接口没国内的





## <font color=red>LEMP环境安装NextCloud</font>
LEMP(Linux, Nginx, MySQ, PHP)
Engine-x  -> Nginx

1.安装
```bash
sudo apt install nginx

sudo apt install mariadb-server mariadb-client
   sudo mysql_secure_installation

# 针对不同软件包，安装不同的组件，这里安装下面这些
sudo apt install php7.2  php7.2-fpm  php7.2-mysql  php7.2-mbstring  php7.2-xml  php7.2-gd  php7.2-curl  php7.2-imagick  php7.2-zip  php7.2-bz2  php7.2-intl


# 配置php
sudo vim /etc/php/7.2/fpm/php.ini
cgi.fix_pathinfo=0
date.timezone = Asia/shanghai
```

2.配置Nginx
```bash
# 这里使用默认站点
# sudo vim /etc/nginx/sites-available/default

index index.php
 location ~\.php$ {
     fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
     include snippets/fastcgi-php.conf;
     include fastcgi_params;
 }

# 重启
sudo nginx -t
sudo systemctl restart nginx
```

3.访问, 验证环境是否成功


4.下载NextCloud
```bash
# NextCloud    免费开源的个人云盘
# 官网下载或wget下载
wget https://download.nextcloud.com/server/releases/nextcloud-13.0.4.zip
sudo unzip nextcloud-13.0.4.zip  -d /var/www/html/mycloud

sudo chown www-data:www-data -R /var/www/html/mycloud
```

5.建库
```sql
-- 进入sql命令行
create database mycloud;
create user 'mycloud'@'localhost' identified by '12345678';
grant all privileges on mycloud.* to 'mycloud'@'localhost' identified by '12345678' with grant option;;
flush privileges;
```

6.生成证书(安全考虑)
```bash
# 这里使用自签名证书
sudo openssl req -x509 -days 365 -sha256 -newkey rsa:2048 -nodes -keyout /etc/ssl/private/mysite.key -out /etc/ssl/certs/mysite.pem
```

7.配置文件
```bash
# cd /etc/nginx/site-available/
# sudo vim mycloud.conf


server {
    listen 80;
    server_name www.mycloud.com;
    # 不让访问者通过http访问，强制重定向
    return 301 https://www.mycloud.com$request_uri;
}

server {
    listen 443 ssl http2;
    # 证书位置
    ssl_certificate  /etc/ssl/certs/mysite.pem;
    ssl_certificate_key  /etc/ssl/private/mysite.key; 
    # 头部
    add_header Strict-Teansport-Security "max-age-31536000" always;  # ssl增强版协议的头部
    # 等经验丰富后，自己要添加头部信息你
    add_header X-Conten-Type-Option nosniff;
    add_header X-XSS-Protection "1;mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Doamin-Policies none;

    root /var/www/html/mycloud/;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
    
    location = /.well-known/carddav {
        retun 301 $schema://$host/remote.php/dav;
    }
    
    location = /.well-known/calddav {
        retun 301 $schema://$host/remote.php/dav;
    }
    
    location = /.well-known/acme-challenge {
        allow all;
    }
    
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    gzip off;

    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;
    
    location = / {
        rewrite ^ /ince.php$uri;
    }

    location = /(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    # .......
}

## ......剩下详细的配置
```

8.启用站点配置
```bash
ln -s /etc/nginx/site-available/mycloud.conf  /etc/nginx/site-enabled/mycloud.conf

# reload 并 restart nginx
```

9.Data目录
```bash
# 文件资源默认会在项目根目录下的Data目录
# 建议不要放在默认位置, 这里放在上一级目录
sudo mkdir /var/www/html/cloud_data
sudo chown www-data:www-data -R /var/www/html/cloud_data
```

10.网页设置
```bash
访问 https://xxxxx.com
# 设置用户名/密码，数据库，data目录
# 注意账户安全哦(不要设置admin等)
# 修改语言
```

11.客户端
```bash
sudo add-apt-repository ppa:nextcloud-devs/client
sudo apt install nextcloud-client

# 或官网下载PC和android的客户端
```



## <font color=red>其他Web应用</font>
#### 1.Cockpit
> 基于Web的远程管理和系统监控

- 基于Web的远程管理应用
- 将所有组件打包在一起
- 不以来单独部署的LAMP/LEMP, 一条命令安装部署

安装
```bash
sudo apt install cockpit cockpit-docker
```

访问方式: 
```bash
https://www.com:9090

登录账户同操作系统
可以看到系统的详细信息，管理

可以添加其他安装了Cockpit的服务器在一个机器的Web界面安装(前提是登录窗口时候进行勾选), 多台服务器
```

官网地址: `https://cockpit-projece.org`



#### Netdata
> 基于Web建构的Linux系统实时性能监视工具

- cpu, 内存，硬盘，网络，进程，硬件
- 占用系统资源少(1-3%)
- 实时自动告警

系统出问题之前，往往有异常的征兆

依赖包
```bash
sudo apt install git zlib1g-dev uuid-dev libmnl-dev gcc make autoconf autoconf-archive autogen automake pkg-config curl python python-yaml python-mysqldb python-psycopg2 nodejs lm-sensors netcat
```

安装
```bash
bash <(curl -Ss https://my-netdata.io/kickstart-static64.sh)  # 现在不能访问了
# 或用apt安装
sudo apt install netdata


# 访问
http://xxxxx.com:19999
```


开关
```bash
# 启动：位置根据系统会有不同。建议加上-D参数前台运行，不要后台运行
$ sudo netdata -D
$ 或
$ sudo /usr/sbin/netdata -D
# 或（Mac上）
$ sudo /usr/local/sbin/netdata -D

# 关闭（方法很多种，往往只有一种生效）
$ sudo killall netdata
# 或
$ sudo pkill -9 netdata
# 或
$ sudo service netdata stop
# 或
$ sudo /etc/init.d/netdata stop
# 或
$ sudo systemctl stop netdata
```


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-lamp/  

