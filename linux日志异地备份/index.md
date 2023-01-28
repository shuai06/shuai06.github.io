# Linux日志异地备份

  
## 日志的异地备份

> 日志服务器的建立尤其重要

这里使用Centos的登录日志做实验



`tail -f /var/log/secure` 实时查看日志变化

`echo " " > var/log/secure` 最简单的清理日志



#### 常见日志文件
  
```bash
/var/log/messages        大多数系统日志信息记录在此处
/var/log/secure            安全和身份认证相关的信息和错误的日志文件
/var/log/mallog            与邮件服务器相关的日志文件
/var/log/cron            与定时任务相关的日志文件
/var/log/boot.log        与系统启动有关的日志文件
```





#### 发送方：

###### 考虑问题

1.我发送什么信息表到日志服务器

2.考虑发送到ip哪个端口

3.使用什么协议



###### 配置

配置文件

```bash
vim /etc/rsyslog.conf

# 这里配置登录日志， 指定日志类别和等级，协议和端口
### begin forwarding rule ###
#*.* @@remote-host:514      @@代表使用TCP

authpriv.* @@192.168.0.111:514

```

```bash
### Note：
# 1.暂时关闭防火墙 
setup  # 关闭防火墙
iptables -nL # 查看一下

# 2.关闭SELinux
setenforce 0
getenforce

# 重启rsyslog 服务
```





#### 接收方：

###### 考虑问题

1.收谁的日志

2.收完存哪

3.用什么协议



###### 配置

配置文件

```bash
vim /etc/rsyslog.conf


# 指定协议和端口
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514

# 收谁的，存哪
# 按照ip地址来收
:fromhost-ip, isequal, "192.168.0.100" /var/log/client_log/192.168.0.100.log
```

```bash
# 重启rsyslog 服务

netstat -antlp |grep 514  # 查看端口是否开始监听了
```



完成

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/linux%E6%97%A5%E5%BF%97%E5%BC%82%E5%9C%B0%E5%A4%87%E4%BB%BD/  

