# 远程连接mysql





  
# 远程连接`MySQL`

想要远程连接`MySQL` 那么必须是普通用户才行 不能是 `root` 用户 所以我们要先创建一个普通用户
1. 创建普通用户
    ```
        CREATE USER 'username'@'%'  IDENTIFIED BY 'password';
    ```
2. 给普通用户赋权
    ```
        GRANT ALL ON *.* TO 'username'@'%'
    ```
3. 刷新系统权限相关表  
    ```
        FLUSH PRIVILEGES
    ```
以上三步执行的 创建用户 --> 给创建的用户赋权 --> 刷新 

这样的话  可能还是不能完成`MySQL`的一个远程连接  这时候需要改变`MySQL`的一个默认配置 这配置文件位于`/etc/mysql/mysql.conf.d/`下的`mysqld.cnf` 里面有找到`bind-address` 把值`127.0.0.1`改为`0.0.0.0`然后接下来就是远程连接了  改完要记得重启`MySQL`服务

接下来就是远程连接`MySQL`首先要保证自己电脑装了`MySQL`才能远程连接 
```
    mysql -u username -p password -h ip地址 -P 端口号（一般默认是3306） 
```
如果想要远程连接 首先要保证 本地安装了`MySQL` 才能远程连接


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5mysql/  

