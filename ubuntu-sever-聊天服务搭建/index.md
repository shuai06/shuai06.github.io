# ubuntu sever-聊天服务




## <font color=red>聊天服务与IRC服务</font>
>沟通是最重要的问题

- 发明网络的目的的沟通(信息交换)
- 高效的沟通是生产力
- 为什么不用QQ、微信、Telegram？
    - 企业信息流经他人服务器（泄密）
    - 内部信息内部传递效率更高
    - 更专注于公司内部工作内容

#### IRC
Internet Relay Chat（因特网中继聊天）
是一种通过网络的群体即时聊天方式（也可以用于个人间聊天）
公开的协议（TCP和SSL协议）
IRC服务器可以连接其他的IRC服务器形成一个IRC网络
大多数的IRC服务器不需要客户注册登录
目前已经很少见（黑客和老牌的技术群体的挚爱）
部分流服务商依赖IRC协议进行信息传递

###### IRC服务器的搭建
`Ircd-Hybrid`为例
- 安装配置简单
- 系统资源占用低

1.安装
```bash
# 安装服务端
sudo apt install ircd-hybrid
```

2.管理员口令(operator)
```bash
# 设置管理员的密码（加密的方式保存密文）
mkpaswd pass123    # 生成密文，一会在后面用

```

3.配置
```bash
# sudo vim /etc/ircd-hybrid/ircd.conf

serverinfo {
    name = "irc.lab.com"
    description = "LAB Campany IRC Server"
    network_name = "This is my Beijing Office IRC Network"
    # 其他配置项...
}
operator {    # 管理员的片段（默认是注释的）
    name = "admin"
    user = "*@*";    # 限制可以连接的用户和ip地址，这里是所有用户所有主机
    password = "xxxxxx" # 上面mkpaswd pass123 生成的密文
}
# 后面根据需要配置
```

4.Banner
```bash
sudo vim /etc/ircd-hybrid/ircd.motd    # 客户端登录信息的信息
```

5.重启服务
```bash
sudo systemctl restart ircd-hybrid.service
```

6.客户端
```bash
- mIRC      # Win下的一款商业软件
- irssi    # Linux下常用

# 安装
sudo apt install irssi
sudo apt install ident2    # 服务端才能访问该客户端的计算机名等信息

# 客户端直接连接服务端就行
irssi -c 192.168.175.130

# 服务端口，默认 666x
```


7.客户端基本命令
```bash
/help    # 查看所有命令
/help  channel
/opera admin pass123
/channel list    # 查看频道
/channel add -auto #Football   # 添加频道
/join #Football    # 加入指定的聊天室

# 把自己提升为管理员身份
/oper admin pass123

```



## <font color=red>MatterMost服务搭建</font>

- irssi这种命令行的方式并不是很友好，MatterMost使用十分友好
- 团队群聊SaaS平台
- Go语言开发
- 独立与操作系统的二进制部署方案(它已经给打包好了)
- 基于MySQL、Postgresql数据库
- Slack(收费)的替代选择方案(兼容Slack，二者可进行通信)
- 全平台客户端支持（Web）
- 免费版、收费版
- 适用于企业内部团队文件共享和信息交换


服务端：https://github.com/mattermost/mattermost-server
客户端程序：https://github.com/mattermost/desktop
Web界面的：https://github.com/mattermost/mattermost-webapp


#### 安装部署
1.安装数据库
```bash
# 这里以mysql为例
sudo apt install mysql-server
sudo mysql_secure_installation
```

2.建库
```sql
create user 'muser'@'%' identified by '12345678';

create database mattermost;

grant all privileges on mattermost.* to'muser'@'%';
-- GRANT ALTER,CREATE,DELETE,DROP,INDEX,INSERT,SELECT,UPDATE ON mattermost.* to'muser'@'%';

flush privileges;
```

3.下载mattermost
```bash
wget https://releases.mattermost.com/5.24.2/mattermost-5.24.2-linux-amd64.tar.gz
tar -zxvf mattermost*.gz
sudo mv mattermost /opt    # 个人安装的软件包，一般会放到opt下面
sudo mkdir /opt/mattermost/data
cd /opt
```

4.权限设置
```bash
sudo useradd -rU mattermost
sudo chown -R mattermost:mattermost /opt/mattermost
sudo chmod -R g+w /opt/mattermost
```

5.配置文件
```bash
# sudo vim /opt/mattermost/config/config.json
# 这里配置数据库项， mattermost库的名字
"DriverName": "mysql"
"DataSource": "muser:passw123@tcp(localhost:3360)/mattermost?charset=utf8mb4,utf8&readTimeout=30s&writeTimeout=30s"
```

6.测试数据库连接
```bash
sudo -u mattermost /opt/mattermost/bin/mattermost
# 重点关注端口是否正常开启，一般默认为8065端口
```

7.Systemd 服务单元文件
```bash
# mattermost本身是不带service的，所以我们可以手动创建

sudo touch /lib/system/mattermost.service
# 添加内容...
```


<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20200704214555991_6879.png" />

8.中文字体

```bash
# sudo vim /opt/mattermost/config/config.json
"DefaultServerLocale": "zh-CN"
"DefaultClientLocale": "zh-CN"
"AvailableLocale": "zh-CN"
```

9.启动服务

```bash
sudo systemctl daemon-reload
# 修改完再启动服务
sudo systemctl start mattermost.service
sudo systemctl enable mattermost.service
# 然后查看服务
sudo systemctl status mattermost.service
```

10.Web配置
```bash
# 域名根据自己的来，或ip也行
https://chat.lab.com:8065
# 首个账号被赋予system_admin角色（管理员）

建议：先建立一个团队(相当于一个聊天室)
然后可以根据提示下载客户端

```

11.系统控制台配置
```bash
常规->服务端口(先指定页面上的命令，再在web界面修改)，证书，地址，data存储目录(本地存储/云存储),电子邮件通知
sudo setcap_net_bind_service=+ep./bin/mattermost    # 修改服务端口
```

12.使用
```bash
# 别人进来的方式：
1.邀请链接
2.单独添加成员


# 客户端：
- Web版
- 其他操作系统客户端

# 支持其他插件，可以与第三方软件集成使用
```

文件的传送，共享文件，置顶文件





## <font color=red>Openfire服务搭建</font>

开源免费的IM即时通信服务器
Real-time collaboration(RTC)
由java开发，需要java运行环境
使用拓展通讯和表示协议（XMPP）Extensible Messaging and Presence Protocol
号称单台服务器可支持上万并发用户
兼容所有支持XMPP协议的客户端(spark),不建议web的方式(基于flash)
支持插件开发
支持mysql、postgrersql、内建数据库
支持LDAP、TLS、群集部署


#### 搭建服务端
1.java环境
```bash
sudo apt-get install openjdk-8-jdk

java -version
```

2.数据库
```bash
# 这里以mysql为例
sudo apt install mysql-server
sudo mysql_secure_installation
```

3.建库
```bash
create database openfire;

grant all privileges on openfire.* to'fireuser'@'%' identified by 'pass123';

flush privileges;
```

4.下载，安装
```bash
# http://www.igniterealtime.org/downloads/index.jsp#openfire

sudo dpkg -i openfire.deb
```

5.导入数据库表
```bash
sudo mysql;
show databases;

use openfire;
source /usr/share/openfire/resources/databases/openfire_mysql.sql

show tables;
```

6.Web界面配置
```bash
## http://openfire.lab.com:9090
#进行常用配置

#数据库URL路径： 地址:端口:库名
jdbc:mysql://127.0.0.1/openfire?useUnicode=true&chasacterEncoding=UTF-8&characterSetResults=UTF-8&rewriteBatchedStatements=true

# 通过Web进入管理界面
```

7.创建用户账号


#### 客户端
可以使用`spark`
http://www.igniterealtime.org/downloads/index.jsp#openfire
客户端登录忽略证书爆粗平
spark使用5222端口连接服务端




## Rochet Chat服务搭建
与Mattermost类似
团队群聊SaaS平台
Slack的替代选择方案
全平台客户端支持(WEB)
适用于企业内部文件共享和信息交换
独立于操作系统的`Snap`部署方式


#### 安装
安装异常简单
###### F1.自动安装
```bash
# 推荐
sudo snap install rocketchat-server
```


###### F2.手动安装
1.安装数据库
```bash
sudo apt install mongodb
```
2.安装依赖
```bash
sudo apt install node.js build-essential npm
```
3.指定Node.js版本
```bash
sudo npm install -g n
sudo n 8.9.3    # 这里的版本自己查看一下目前支持的版本号
```

4.下载服务端源码，并安装
```bash
# 下载地址：
https://releases.rocket.chat/latest/download

tar -zxcf xxx.tgz
cd bundle/programs/server
npm install
cd ../

export ROOT_URL=http://1.1.1.1：3000/
export MONGO_URL=mongodb://localhost:27017/rocketchat
export PORT=3000
node main.js    # 启动服务端程序

```

5.访问Web页面
```bash
# 向导页面
创建账号等配置（第一个账号是管理员）
```

6.用户访问3000的地址，可以注册账号，加入聊天等


7.配置数据库集群（自己查，生产环境建议配置）
    生产环境数据库复制集群

Note：数据库建议使用单独的服务器



## <font color=red>系统中文显示</font>

> 此为补充知识点

```bash
# 先安装中文语言包
sudo apt install language-pack-zh-hans
locale -a

# 配置中文语言环境变量
vim /etc/environment
    LANG="zh_CN.UTF-8"
    LANGUAGE="zh_CN:zh:en_US:en"

# 配置中文，英文还是会继续使用的
```









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-sever-%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E6%90%AD%E5%BB%BA/  

