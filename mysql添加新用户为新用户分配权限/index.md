# mysql创建新用户并分配权限


  

1、使用root用户登录mysql

2、添加具有本地(localhost/127.0.0.1)访问权限的用户

```
create user 'newuser'@'localhost' identified by 'password';
```

3、创建具有远程访问权限的用户

```
create user 'newuser'@'%' identified by 'password';
```

创建之后记得执行下面指令更新权限：
```
flush privileges;
```


3、为新用户分配本地权限，可以指定数据库dbname和表名，可以用*替指所有。

```
grant all privileges on `dbname`.* to 'newuser'@'localhost' identified by 'password';
```


4、为新用户分配远程权限，可以指定数据库dbname和表名，可以用*替指所有。
```
grant all privileges on `dbname`.* to 'newuser'@'%' identified by 'password';
```

分配所有权限
```
> grant all privileges on *.* to  'test'@'%'  identified by '123456'
```

分配好之后之后记得执行下面指令更新权限：
```
flush privileges;
```

5、如果还有问题，可以使用root账号登陆上去查询一下，再看看有没问题。
```
use mysql
```

6、查看user表
```
select Host, User, Password from user;
```




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/mysql%E6%B7%BB%E5%8A%A0%E6%96%B0%E7%94%A8%E6%88%B7%E4%B8%BA%E6%96%B0%E7%94%A8%E6%88%B7%E5%88%86%E9%85%8D%E6%9D%83%E9%99%90/  

