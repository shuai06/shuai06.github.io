# ubuntu server- ubuntu-server-数据库管理服务



#  ubuntu-server-数据库管理服务
>数据就是生产力

数据就是应用的核心
按照结构组织，存储和管理数据的仓库
Oracle, DB2, MySQL, MSSQL, PostgreSQL, Infomix, MongoDB



<img src="http://image.xpshuai.cn/20200430085915705_995731668.png" />


## <font color=red>数据库管理系统(DBMS)</font>
<img src="http://image.xpshuai.cn/20200430090504409_350097281.png" />


## <font color=red>MySQL数据库管理</font>
<img src="http://image.xpshuai.cn/20200430091905869_1159836654.png" />


安装
```bash
sudo apt install mysql-server
```

安全配置
```bash
sudo mysql_secure_installation
# 建议禁止root远程登录
# 这里设置的root的密码不是，后面才设置, 因为初始身份认证方式是auth_socket
```


Root帐号密码登录
```bash
sudo mysql    # 初始身份认证方式是auth_socket，所以暂时不需要输入密码 
SELECT user,host,authentication_string, plugin,host FROM mysql.user;
# 给root设置密码认证方式mysql_native_password
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '12345678'; 
FULSH PRIVILEGES;
# 登录
mysql -u root -p
```


创建数据库个人帐号(建议不要使用root进行运行,而应该使用个人帐号, 且密码不能相同)
```bash
create user 'xps'@'%' identified by '12345678';
# 赋予权限(4级)
# *.* 是哪个库哪个表，这里表示所有库所有表
grant all privileges on *.* to 'xps'@'%' with grant option;
```

修改密码
```bash
set password for 'xps'@'%'=password('12345678');
```

网络访问帐号(可以远程访问)
```bash
# 一般这种情况，都限制要操作的库和操作
# %
use mysql
# 直接grant赋予权限，就可以创建不存在的用户
grant all privileges on *.* to admin@'%' identified by '12345678' with grant option;
# 移除权限
revoke all on db_name from user_name;
# 查看密码策略(一旦允许网络连接，密码级别就会变成中安全级别了)
select @@validate_password_policy;
```

编辑配置文件
```bash
# sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 修改侦听网卡为自己网卡的ip地址
bind-address        = 192.168.0.112

# 重启网卡

# 使用mysql客户端连接， 或Falcon等图形化客户端
mysql -h 19.168.0.112 -u root
```



## <font color=red>SQL语句</font>
#### 增删改查
**最基本**
```sql
-- 当前帐号
select user();

-- 查看已经有的库
select databases;

-- 选取(切换)库
use test;
select database();

-- 显示表(已选择库)
show tables;

-- 简单创建库
create database test;
```

**创建表**
```sql
create table users(
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,  
    name CHAR(25) NOT NULL,
    pass CHAR(25) NOT NULL,
    age INT(9) NOT NULL
 )
```


<img src="http://image.xpshuai.cn/20200430105007407_399865569.png" />

char的性能要优于varchar


**插入记录**
```sql
insert into users values(1,'xps','1234567890',20);
insert into users(id,name,pass,age) values(null,'xps','1234567890',20);   # 指定列
```

**查看表结构**
```sql
desc users;

show create table users;   # Engine=InnoDB   存储引擎

show full columns from users;
```

**查询**
```sql
select * frm users where age <20 and age >16;
select name,pass from users order by age;
select name,pass from users order by age desc;  # 降序

select concat(name,0x7e,pass,0x7e) as user_pass from users;   # concat 结合， 表头变成user_pass
select group_concat(name, pass) from users;

```


**修改记录**
```sql
update users set name='xpsss' where id=1;
```

**删除记录**
```sql
delete from users where id=1;
```

**删除表**
```sql
drop table tests;

drop table if exists tests;
```

**删除库**
```sql
drop database test;
```

**查看数据引擎**
```sql
show engines;
# 修改存储引擎（这里指定表）
alter table student engine=MYISAM;
```

**修改表名**
```sql
alter table student RENAME student2;
```

**修改字段数据类型**
```sql
alter table student MODIFY name varchar(30);
```

**修改字段名**
```sql
alter table student CHANGE name uname varchar(40);
```

**增加字段**
```sql
alter table student ADD techer_name vchar(20) NOT NULL after id;  # 位置放在id字段的后面
```

**删除字段**
```sql
alter table student FROP teacher_name;
```

#### 备份
###### 备份导出
备份库，表
```bash
mysqldump -u root -p tes_db  >db_back.sql
mysqldump -u root -p test_db users >db_back.sql
mysqldump --all-databases -u root -p >all_db_back.sql
```

备份指定数据
```sql
# 先进入sql命令终端，进行导出
select * from users; > q.sql
## 或者
# 1. 先把sql语句保存为文件，
echo 'select * from users;' > q.sql
# 2.然后执行来进行导出
mysql -u root -p test  < q.sql  >out.csv
```

导出CSV
```sql
show variables like '%secure%';  # 先找到备份路径，找到secure_file_priv路径就是默认导出的路径
# 保存， 并执行分隔或换行
select * from users into outfile '/var/lib/mysql-files/aaa.csv' fields terminated by ',' enclosed by lines '"' terminated by '\n';
```

###### 导入
导入
```bash
# 创建库的命令
mysqladmin -u root -p create db2  # 也可以在sql终端  create database xxx;
# 导入
mysql -u root -p db2 < db_bak.sql
```

导入CSV
```bash
LOAD DATA INFILE '/var/lib/mysql-files/aaa.csv' INTO TABLE users
FIELDS TERMINARED BY ',' ENCLOSED BY '"'
LINES TERMINARED BY \n IGNORE 1 ROWS;
```






## <font color=red>MariDB</font>
#### 安装配置
安装
```bash
sudo apt install mariadb-server
```

安全认证
```bash
sudo mysql_secure_installation
```

改为密码认证
```bash
sudo mysql
use mysql;

select user.host,plugin,password from mysql.user;   # 查看
update user set plugin='' where user ='root'   # mysql_native_password或空就行
fulsh privileges;
```

配置文件
```bash
# /etc/mysql/mariadb.conf.d/50-server.cnf
bind-address = 0.0.0.0 # 每一个网卡都会侦听3306
```


#### 主备服务器(数据库复制)
**复制  Replication**
Master -> Slave    读/写
限定所有库，指定库，表 进行复制


**复制原理**
bin-log        二进制日志
所有DB事件先写入bin-log
Slave从Master读取bin-log


**用途**
- 扩展：Slave分担master负载
- 分析：数据分析不影响Master
- 备份：作为一种数据备份手段
- 分布：异地保存数据副本（也可以实现应用访问本地库）


###### 演示
1.安装两台独立的MariaDB服务器
```
复制之前用导入导出数据库的方式保持Master/Slave数据一致
复制前锁定库(禁止写入)        # 生产环境
    flush tables with read lock;
```

2.配置 Master
```bash
# suo vim /etc/mysql/conf.d/mysql.cnf    #增加如下配置
[mysqld]
log-bin    # 开启bin-log功能
binlog-do-db=test
# binlog-ignore-db="mysql"   # 除了忽略的库，其他的全复制
server-id=1
```

3.配置master的mariadb远程访问
`bind-address  0.0.0.0`

4.重启服务
```bash
sudo systemctl restart mariadb
```

5.配置master
创建用户帐号
```bash
#192.168.0.111 是slave的地址， *.*允许复制所有数据库, replicate用户名
grant replicate *.* to 'replicate'@'192.168.0.111' with identified by 'passwd';
```

6.配置Slave
```bash
## 1.
# suo vim /etc/mysql/conf.d/mysql.cnf    #增加如下配置
[mysqld]
server-id=1

## 2.开启远程访问bind-address  0.0.0.0

## 3. 重启数据库服务
```

7.登录sql终端

```bash
# master_user是master服务器上创建的mysql的用户
change master to master_host="192.168.0.112",master_user="replicate",master_password='password';
```

8.打开master锁
`unlock tables;`

9.查看slave状态
```sql
-- slave的sql终端
show slaves status\G
start slave;    # 如果没启动，可以手动启动
```

测试效果
```bash
# 在master执行
cd test;
insert into users values(1,'yyy','pass');
delete from users where id=1;

# 在slave上查询，发现数据已经同步过来了，近乎实时
```

**Mysqltuner -- 配置调优工具包**
安装
```bash
sudo apt install mysql
```
运行
`sudo Mysqltuner`
会生成一堆信息配置调优建议(绿色的表示没问题，其他颜色的可以根据帮助来优化)









## <font color=red>Postgresql</font>
>下一代关系数据库

#### 配置和修改
- `高并发`情况下`性能`优于MySQL
- 侦听端口 TCP `5432`

**安装**
```bash
sudo apt install postgresql
```

**身份认证**
- 默认使用IDENT身份认证方式
- 使用与操作系统同名的数据库帐号(`postgres`相当与MySQL中的root)

**进入**
```bash
sudo -u postgres psql
# 直接登录到库的名称(template1是默认生成的)
sudo -u postgres psql template1
```


**修改密码**
```bash  
# sql终端
# 使用加密的方式
alert user postgres with encrypted password 'your_passwd';
\q    # 退出

# bash终端
# 修改配置文件
vim /etc/postgresql/10/main/pg_hba.conf
local    all    postgres    md5
# 本地登录    所有数据库    身份    密码md5加密


# 重启服务

# 登录  -W输入密码
psql -U postgres -W -d db_name
```

**配置文件**
```bash
/etc/postgresql/<version>/main    # 配置文件目录
/etc/postgresql/10/main/pg_ident.conf    # IDENT身份认证配置
/etc/postgresql/10/main/postgresql.conf    # 网络身份认证配置
    # listen_address='*'        # IPV6 '::'
```

**网络连接**
```bash
# 客户端
sudo apt install postgresql-client

# 设置允许哪些主机连接我们(必须指定掩码)
vim /etc/postgresql/10/main/pg_hba.conf
local    all    postgres    192.168.0.111/32 md5
#重启服务

# 客户端连接
psql -h 192.168.0.112 -U postgres -W -d db_name
```

**创建帐号**
```bash
# postgres权限太大，不建议平时使用

# F1 服务器端终端
sudo -u postgres createuser xps  # gropuser xps

# F2  sql命令行
create user foobar;    # drop user xps
```

**文档手册**
```bash
sudo apt install postgresql-doc-10
/usr/share/postgresq-doc-10/html/index.html

官方文档:
https://www.postgresql.org/docs/current/static/tutorial.html

```

#### 基本指令
创建库
```bash
#
sudo -u postgres createdb mydb;
# 或
create database mydb;
```

删除库
```bash
sudo -u postgres dropedb mydb;
# 或
drop database mydb;
```

创建表
```sql
create database users
(
uname varchar(25);
pwd carchar(25);
date date
);
``

插入数据
​```bash
insert into users values('xps','1234567890','1997-07-07');

# 批量插入数据(数据提前保存)
copy users from '/home/user/users.txt'
```

快捷命令
```bash
\c db_name    # 切换数据库 （use mydb;）
\c - username    # 切换用户
\l            # 列出数据库(show databases;)
\dt             # 列出表(show tables;)
\d    table_name    # 查看表结构
```




## <font color=red>NoSQL概览与MongoDB</font>

<img src="http://image.xpshuai.cn/20200430164041499_1710130793.png" />

<img src="http://image.xpshuai.cn/20200430164659724_1776362163.png" />

<img src="http://image.xpshuai.cn/20200430165730935_61571111.png" />

**常见的NoSQL数据库类型**
1.键值对类型

<img src="http://image.xpshuai.cn/20200430165836806_1146424929.png" />

<img src="http://image.xpshuai.cn/20200430165955029_600511168.png" />

<img src="http://image.xpshuai.cn/20200430170233715_1141118525.png" />

<img src="http://image.xpshuai.cn/20200430170407242_2033108350.png" />

<img src="http://image.xpshuai.cn/20200430170848764_1597209188.png" />

<img src="http://image.xpshuai.cn/20200430171004638_167406085.png" />

`Paper`推荐多看看



#### MongoDB
安装
```bash
sudo apt-get install -y mongodb
```

客户端
```bash
# 本地登录
mongo
# 远程登录
mongo 192.168.0.112:27017/db_name

# 常用管理命令
help
```


进入库
```
use mydb;
# 如果不存入数据，他还是就没了
```


输入数据
```
# 创建文档集
db.createCollection('users')
# 插入数据
db.users.insert({'name':'uuu','uid':123})
```

查询库
```mongodb
show dbs
show collections
```

多赋值
```mongodb
db.users.insert({'name':'uuu','uid':123,gid:[111,222,333]})
```

查询
```mongodb
db.users.find();
db.users.findOne({'uid':1001})
db.users.findOne({'uid':{$gt：1000}})   # 大于
db.users.find({'uid':{$gt：1000}})   # 大于

db.users.find({$or: [{name:'ubuntu'}, {name;'centos'}]})   # or

db.users.findOne({'uid':1001}, {name:0})    # name:0  不显示name字段
```

修改记录
```mongodb
db.users.update({'uid':1001}, {$set:{name:'hhh'}})
```

删除记录
```mongodb
db.users.remove({'uid':1001}, )
```

删除collection
```mongodb
db.users.drop()
```

删库
```mongodb
db.dropDatabase()
```

详细用法参考官方文档......









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/ubuntu-server-%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AE%A1%E7%90%86%E6%9C%8D%E5%8A%A1/  

