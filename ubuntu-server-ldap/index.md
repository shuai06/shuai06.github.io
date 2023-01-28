# ubuntu server-LDAP



# LDAP
>看书前先看目录

#### 理解

不同于文件系统的目录(D/d) -> 首字母大写是目录服务，首字母小写的文件系统目录

目录服务是企业网络的核心基础结构
```bash
- 如此重要的服务并不被大多数熟悉
- 很少独立使用，与其他服务结合使用
- 微软的企业战略（埋坑: 一旦底层用了微软的AD,你以后的一些肯定也要用我的集成的产品）
- 类似: 每一本书都有自己的目录
```

数据集中存储的应用需求
```
- 集中存储、集中管理、集中定位
- 身份认证、公钥分发、邮件路由、地址查询、应用配置、单点登录(SSO -> 只登录一次就不用重复登录了)
- 面向大量频繁的数据查询操作，少量写操作（基本不变的数据）
- 权限限制授权用户查询
```


我是否应该使用目录服务(根据自己公司情况)
```
- 用户认证
- 机器认证
- 用户和组
- 电话地址薄
- 组织呈现
- 资产跟踪
- 集中应用配置
- PEX / 网络设备配置
```



#### DAP

**软件架构**
- 不同与SQL的后端层级化数据库（`主要面向查询操作`）
- 与DNS的分布式结构更接近，命名上通常与DBS命名保持一致(在目录服务内所有存储对象都有唯一的路径名称)
- 数据库基于键值对的方式存储数据
- 标准统一的服务前端应用查询访问接口(基于OSI协议栈,  X.500)
- 统一的客户端查询组件和协议(C/S)
- 支持通信会话加密
- Directory information tree (DIT)


**与自建数据库开发相比**
- 专门针对查询服务的数据库结构类型
- 统一标准的查询结构以及通信协议（DAP ->目录访问协议）




#### LDAP
> Lightweight Directoy Access Protocol    轻量目录访问服务


- 命名上通常与DBS命名保持一致(在目录服务内所有存储对象都有唯一的路径名称)
- 数据库基于键值对的方式存储数据
- X.500标准的部分特性集, 各厂商不同
- 基于TCP/IP协议通信(TCP `389`)
- 可构建面向互联网和本地网络的目录服务

最常见的LDAP服务
```bash
# 微软的活动目录 AD
企业网络资源全部加入域，统一存储、统一呈现、统一管理、安全边界

# Novell eDirectory
```



**命名空间**
```bash
# 全局路径： Distinguished Name (DN)
# 相对路径： Relative Distinguished
# 目录容器： Directory Container (DC)   每一层
# 组织单位:  Organisation Unit (OU)     通常按照部门划分
# 数据项： entries (object, class)
        一组预先定义的属性集合
        uid, 姓，名，帐号，邮箱，电话, 照片...

# 基础架构：Schema
        objectClass 定义所有对象类型及其属性
        Schame 可按应用需要扩展(美国对象在ASN.1结构中都有自己的位置 -- SNMP)

# 通用名： common name (CN)
```

**传统命名空间**
- 无固定的命名规则
- 按照地理空间命名

<img src="http://image.xpshuai.cn/20200507135251359_1631864771.png" />

<img src="http://image.xpshuai.cn/20200507135329746_833117774.png" />

```bash
# 写法
uid=babs, ou=People, dc=example, dc=com
```

**LDAP协议**
LDAP最初作为TCP/IP客户udan与X.500目录服务网关之间发通信协议
LDAPv3:
- 强身份认证和数据加密
- 基于证书的SSL加密
- 使用Unicode编码实现国际化
- Schema可扩展


#### 安装并配置LDAP
>拿出一台服务器作为专门的LDAP的服务器，以后的身份认证都使用集中的LDAP的(不局限与操作系统，`Web应用也可以`，只要程序支持ldap就可以使用这个做身份认证)

先修改主机名：
```bash
sudo sed -i 's/preserve_hostname:false /preserve_hostname:true /g' /etc/cloud/cloud.cfg

echo "192.168.0.104 ldap.lab.com" | sudo tee -a /etc/hosts  # ip地址与名称的解析关系
# 修改主机名
sudo hostnamectl set-hostname ldap.lab.com

reboot
```

安装 OpenLDAP
```bash
# Ubuntu Linux中默认的LDAP实现
sudo apt install slapd ldap-utils
# slapd 是服务端和后台进程(即LDAP server)
# ldap-utils  （ldap工具）
    ldapadd       # 添加数据
    ldapdelete    # 删除数据
    ldapmodify    # 修改
    ldapsearch    # 查询（最常使用）
    ldappasswd    # 修改密码
    ... ...
```

修改配置文件
```bash
  
# sudo vim /etc/ldap/ldap.conf
BASE    dc=lab,dc=com    # 目前和dns的域名保持一直即可
URI     ldap://ldap.lab.com:389    # 可以通过这个地址对..进行查询
```

初始化配置
```bash
# 一次就够了!!!
sudo dpkg-reconfigure slapd

# 还可以设置一次密码
# 数据库建议选择MDB
```

验证
```bash
# 多种验证方式
sudo slapcat

sudo tree /etc/ldap/slapd.d/   # 有内容就可

# 默认管理员帐号 admin

ldapsearch -x -LLL -b dc=lab,dc=com 
ldapsearch -x -LLL -b dc=lab,dc=com dn  # 后面的内容是筛选
# -x    简单身份验证
# -LLL  以标准LDIF文件格式显示结果
# -b    基地址
```


**向目录中添加记录**
保存信息
```bash
# 保存信息到文件
# vim add.ldif

# ou: 物理的层级结构划分
dn: ou=People,dc=lab,dc=com
objectClass: organizationalUnit
ou: People
# 物理的
dn: ou=Group,dc=lab,dc=com  # ou的名称叫Group, 一个对象只能在一个ou中, ou是物理上的
objectClass: organizationalUnit
ou: Group

# 逻辑的
# 组帐号s
dn: cn=delevopment,ou=Group,dc=lab,dc=com   # 创建一个组，cn别名
objectClass: organizationalUnit
cn: delevopment
gidNumber: 3000
# 用户帐号
dn: uid=zhangsan,ou=People,dc=lab,dc=com   # 创建一个组，cn别名
objectClass: inetOrgPerson    # 继承某些对象类型
objectClass: posixAccount
objectClass: shadowAccount
uid: zhangsan
sn: zhang
givenName: san
cn: san zhang     # 别名
displayName: San Zhang
uidNumber: 2000
gidNumber: 3000    # 让用户加入某个组
userPassword: 123465
loginShell: /bin/bash
homeDirectory: /home/zhangsan/
```

导入：
```bash
ldapadd -x -D cn=admin,dc=lab,dc=com -W -f add.ldif
# -x    简单身份验证
# -D    指定管理员帐号(完整的别名： cn=admin,dc=lab,dc=com)   
# -W    提示输入密码
# -f    指定ldif文件
```

验证
```bash
ldapsearch -x -LLL -b dc=lab,dc=com 'uid=zhangsan' cn sn gidNumber uidNumber givenName

# 查看属性类型和值 参考网址
https://ldapwiki.com/wiki/ObkectClass#Types
```



**修改数据**
跟上面是类似的

<img src="http://image.xpshuai.cn/20200507151131491_1817511878.png" />





#### Web管理
手动写配置太麻烦，图形化的简化了操作
Web应用程序(有很多种)： `ldap-account-manager` (lam)

```bash
sudo apt install apache2
sudo apt install php php-cgi libapache2-mod-php php-common php-pear php-mbstring

sudo a2enconf php7.2-cgi    # 激活这个模块
sudo systemctl reload apache2
sudo systemctl restart apache2

sudo apt install ldap-account-manager


# ldap-account-manager使用的数据库使用的是目录服务的数据库，但是他的数据也没有写到数据库中，而是存在了配置文件中
# 访问即可
# http:/192.168.0.104/lam

# 点击右上角 LAM configuration 配置  edit server... , 进行简单配置
# 其他配置根据自己需求来改

```


<img src="http://image.xpshuai.cn/20200507155013573_1158190765.png" />

<img src="http://image.xpshuai.cn/20200507153818370_1691714384.png" />

<img src="http://image.xpshuai.cn/20200507153853899_1124405028.png" />



**设置安全访问**
```bash
sudo vim /etc/apache2/conf-enabled/ldap-account-manager.conf
    #Require all granted    注释掉所有ip可访问
    Require ip 127.0.0.1 192.168.0.0/24

# 重启服务
```


**其他ldap基于web的管理工具**
```bash
# phpLDAPAdmin
sudo apt install phpldapadmin

# 创建组帐号 
# 创建帐号
# ...
```



#### 客户端

###### Windows客户端
win自带的`GINA`提供Windows系统登录界面， 支持本机、域账户登陆

`pGina`开源身份认证提供者
    - 集成并替换window系统默认的GINA
    - 通过插件支持大量身份验证数据存储(LDAP, MySQL, RADIUS)
    - `http://pgina.org` 下载安装稳定版

进行一些配置



###### Linux客户端
域名解析
```bash
echo "192.168.0.104 ldap.lab.com" |sudo trr -a /etc/hostss
```

安装客户端
```bash
sudo apt install libnss-ldap libpam-ldap ldap-utils nscd
# 指定一些参数
ldap://ldap.lab.com
dc=lab,dc=com
cn=admin,dc=lab,dc=com
```

修改配置文件
```bash
# sudo vim /etc/nsswitch.conf
passswd: compat systemd ldap
group: compat systemd    ldap
shadow: compat ldap


# sudo vim /etc/pam.d/common-password
    删除 use_authtok 这一小块

# sudo vim /etc/pam.d/common-session
    session optional  pam_mkhomedir.so skel=/etc/skel umask=077   # 允许创建用户主目录
 
sudo systemctl restart nscd.service

# 验证
getent passwd zhangsan 

reboot

# 登录的时候，点击 not list 才能进入刚才的帐号
```





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-ldap/  

