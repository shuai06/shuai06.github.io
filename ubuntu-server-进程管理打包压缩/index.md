# ubuntu server-进程管理&打包压缩




  
#### 进程管理  
######  ps  

```bash
信息：
PID： 进程id
TTY: 运行进程的终端设备
STAT： 进程状态(Sleep, Running)
TIME： 该进程占用的cpu时间
COMMAND: 命令名称

参数：
-x： 当前用户启动的所有进程
-ax： 所有用户启动的进程
-u： 进程详细信息
-aux:
-w： 显示进程文件完整路径
-auxw:
ps u PID    ($$: 当前shell的进程id)

ps -L pid号   # 查看当前进程下面的线程

ps fajx   # 查看进程树

pstree   # 更直观的进程树


# 结束进程
kill pid   # 默认Term
kill -STOP pid   # 暂停进程  （杀病毒杀不掉的时候可以先stop）
kill -CONT pid   # 回复已暂停的进程
kill -KILL pid   # kill -9 pid
kill -l  # 所有kill命令的信号 

```







#### 归档打包和压缩

###### gzip  (GUN Zip)
```bash

gzip file1

gunzip file1.gz

# Gzip不支持多文件/目录的归档打包
# 会干掉原始文件

```


###### tar打包归档 (Archive)


```bash
tar cvf file2.tar file1 file2 file3   # 并不进行压缩
gzip file.tar                         # 可以打包之后再压缩

gunzip   file.tar.gz   # 先解压
tar xcvf file2.tar # 解包



#  tar 结合gz
tar zcvf file2.tar.gz file1 file2 file3   # 参数z 打包同事亚索

tar -ztvf file.tar.gz    # -t 显示文件内容但是不解压


tar jcvf c.tar.bz2 a b # bz2

```



###### bzip2
速度可能慢一点，但是对文本文件压缩更小
```bash

bzip2 f   # 压缩

nunzip f.bz2

# tar 的 -j也可以
tar jcvf c.tar.bz2 a b 

```

#### sudoer
`sudo`用户组   `cat /etc/group`中sudo组，才能加sudo运行命令
- 处于安全的考虑 `sudo`  
- visudo
- 文件`/etc/sudoers`

```shell

组 所有主机=() 系统的哪些命令  
%sudo All=(ALL:ALL) ALL


# 指定配置
user_alias ADMINS = user1, user2  # 指定一些用户的别名
ADMINS ALL = (ALL)NOPASSWD:ALL    # 该别名的权限

root ALL=(ALL) ALL


```







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86%E6%89%93%E5%8C%85%E5%8E%8B%E7%BC%A9/  

