# ubuntu server-备份还原


**备份还原**

>备份是免于损失的有效手段
>防火防盗防自己（保持良好睡眠）
>版本管理也视为一种备份(上下文不同)


##  备份还原基础

### 首先考虑问题

**1. what to backup**

```bash
数据、配置、操作系统、运行环境
```

**2. where to back**

```bash
本地、远程、异地三个副本
磁盘、光盘、磁带、一次性写入介质
```

**3. how to restore it**

```bash
备份系统自动恢复
人工手动恢复
```


## 备份方式

### <font color=red>Shell脚本手动备份</font>

>tar打包压缩减小备份文件大小

写个简单的shell脚本

```bash
## 如脚本内容,对指定目录打包
#!/bin/bash
#备份源
backup_files="/opt/google  /etc/Documents"
#备份目标
dest="/backup"
#创建归档文件(主机名-时间)
day=$(date +%Y-%m-%d-%A-%T)
hostname=$(hostname -s)
archive_file="$hostnane-$day.tgz"
#显示备份状态
echo "Backing up $backup_files to $dest/$archive_file"
date
#执行备份.
tar czPf $dest/$archive_file $backup_files

```

然后手动运行shell脚本(或做一个定时任务crontab)

```bash
chmod u+x backup.sh
./backup.sh    
```

问题：在业务繁忙时候，影响服务器性能和网络带宽，影响正常业务运行（所有基于业务空闲时候来决定备份时间）


验证备份

```bash
tar -tzvf /backup/backup.tgz
```

还原/恢复

```bash
# 解压到指定目录
tar -zxvf /backup/backup.tgz -C /tmp

# 解压到原目录(会覆盖已有文件)
cd /
sudo tar -zxvf /backup/host-Monday.tgz 
```



### <font color=red>Shell脚本自动备份</font>

>cron计划任务

配置文件：`/etc/crontab`
配置

```bash
sudo crontab -e    # 这里是root权限运行（不同用户有自己的计划任务）
30 23 0 * * 1 /bin/bash /home/xps/backup.sh    # 每周一的23:30执行

# 分(m,0-59)  时(h,0-23)  日(dom,1-31) 月(mon,1-12) 周(dow,0-7) 命令(command)

# ctrl + o 保存， ctrl + x 退出

crontab -l    # 查看当前用户已有计划任务列表
```

### 全量滚动备份计划

例如：每天做一个日备份，到一周就合并为一个周备份，到一个月就合并为一个月备份

```bash
两个月为周期滚动备份
每月1日执行月备份
每周六执行周备份
每天进行日备份
6个日备份 + 4个周备份 + 2个月备份
```

脚本

```bash
#备份源
backup_files=" /home /var/spool/mail /etc /root /boot /opt"
#备份目标
dest="/backup"
#归档文件名变量(星期几 / 主机名).
day=$(date +%A)
hos tnane=$(hostname -s)
#当前是每月中第几个星期
day_nun=S(date +%-d)
if (( $day .nun <= 7 )); then
week_file= "$hostname -week1.tgz"
ellf (( $day_num > 7 & $day_nun <= 14 )); then 
week_file="$hostname -week2.tgz "
elif (( $day_nun >14 && $day_nun <= 21 )); then
week_file=" $hostname -week3.tgz"
elif ( $day_num >21 && $day_num < 32 )); then
week_file=" hostname -week4. tgz"
fi
#单数月 / 双数月
nonth_num=$(date +%m)
month=$(expr $month_num % 2)    # 如果余数
if [ $month -eq θ ]; then        # 如果余数等于0
nonth_file="$hostname - nonth2.tgz"
else
month_file="$hostname -nonth1.tgz"
fi
#创建归档文件(每月1日-->月备份、 每周六-->周备份、每天-->日备份)
if [ $day_num == 1 ]; then
archive_file=$month_file
elif [ $day != "Saturday" ]; then
archive_file=" $hostname -$day.tgz"
else
archive_file=$week_file
fi


#显示备份开始信息
echo "Backing up Sbackup_ ftles to $dest/$archive_file"
date
echo
#使用tar命令备份
tar CzPf $dest/$archive_file $backup_files
#显示备份结束信息
echo
echo "Backup fintshed "
date
#结束后查看备份文件大小
ls -lh $dest/

```

也可以加入到计划任务里面执行


缺陷：

- 每个备份都是完整备份
- 备份时间长,磁盘空间占用量大



### 增量差异备份

#### <font color=red>Rsync</font>

- Linux 下众多备份工具，只rsync是必须掌握的一个
- 在两个目录或两个主机之间同步文件（ 从源`单向`同步到目标，但不双向同步）
- 支持增量备份，节省时间带宽源
- 参数众多
- 学习时建议使用例子文件（使用不当可能破坏文件）
- 还原时：先把最前面的完整备份还原，然后依次还原后面的增量备份
- 


**文件服务器**

```bash
# 创建演示文件
mkdir sampledir
touch sampledir/file{1..10}
```

**客户端**同步(`下拉`)

```bash
# 安装
apt install rsync
# 同步服务端的文件
rsync -azP -e ssh xps@1.1.1.1:~/sampledir  backup/
# 也可以基于密钥、密码身份认证
# sampledir 后的/表示拷贝其中所有内容,没有/表示拷贝该目录

#服务端新增、修改文件后再次运行此命令,只有变更的部分同步
#服务器断删除文件后再次运行此命令,客户端不同步删除动作
```


同步本地文件夹到服务端

```bash
# 源    目标
rsync /var/log/mysql  ~/mysql_log_backup
```

计划任务

```bash
# 每天执行计划任务执行rsync
crontab -e
@daily rsync -azPe ssh xps@1.1.1.1:~/sampledir /backup

```

向上同步(`推`)

```bash
rsync -azP -e ssh backup/ xps@1.1.1.1:~/sampledir
-a =-rlptgoD    # 归档，保留修改时间戳、符号链接、权限等元数据
-r        # 递归拷贝目录内容(不保留权限)
-l        # 拷贝符号链接
-p        # 保留权限
-g        # 保留组所有者
-o        # 保留用户所有者
-D        # 保留设备文件
-v        # 显示详细信息
-z        # 压缩
-P        # 生成进度文件，实现进程报告和断点续传
-e        # 指定远程shell类型，默认rsync明文传输数据
--exclude pattern    # 指定排除的文件名
--exclude-from filename.txt    # 排除文件名的特征文件(把几个文件名写到一个文件里面)
--include pattern    # 指定包含的文件名
--include-from filename    

# 排除文件
rsync -azP --exclude 'dir*' source/ destination/
rsync -azP --exclude-from 'exclude-list.txt' source/ destination/

# SSH非标准端口
rsync -azP -e "ssh -P port_number" source destination

```



**rsync默认不删除目标文件**

```bash
--delete	#镜像源目的内容，源删文件目的也删
--backup	#删除/覆盖前备份源文件（改名防覆盖)
--backup-dir	#指定备份路径
--remove-source-files	#成功传输后删除来源文件
--dry-run	#模拟执行命令，并输出结果(并不真正修改文件)

rsync --remove-source-files -azPv srcl dest/
rsync --dry-run --remove-source-files -azPv src/ dest/

```





**增量备份：**

```shell
# touch backup.sh
# chmod +x backup.sh
# mv backup.sh usr/local/bin
# vi backup.sh

#/bin/bash
CURDATE=$(date +%Y-%m-%d)
rsync -avb --delete --backup-dir=/backup/$CURDATE /src /dest

```



#### 其他工具

**双向同步工具**

```bash
sudo apt install unison
unison ~lsampledir ssh:l/1.1.1.1//homelyuanfh/documents
```



**实时同步工具**

```bash
sudo apt install lsyncd
#lsyncd实时监视本地文目录变化
#默认使用Rsync实现文件同步SCP / SSHFS
#参考以前章节内容
```



**SCP/SSHFS**







#### <font color=red>Bacula</font>

##### 简介

Bacula 是一款开源的跨平台网络备份工具，它提供了基于企业级的客户端/服务器的备份恢复解决方案。通过它，系统管理人员可以对数据进行备份、恢复，以及完整性验证等操作。同时，它还提供了许多高级存储管理功能，使系统管理人员能够很容易发现并恢复丢失的或已经损坏的文件。正因如此，Bacula 也被誉为**最好的开源企业级备份工具**。




- **基于网络**的开源备份、还原、验证软件
- 对标商业备份软件的功能
- 客户端支持Windows(很久不更新了，主在其他系统)、Linux、Unix、MAC有多个开源工具组合而成
- 软件架构
- **Director** :主进程，控制所有备份、恢复、验证、归档操作
- **Console**:管理员于Director通信的操作借口，分为三种类型
    - 基于文本的命令行版本(完全的功能)
    - 基于Gnome GTK+的GUI版本(大部分功能)
    - wxWidgets GUI版本(大部分功能)
- **File Daemon** : Bacula客户端程序，**安装在需要备份的计算机上**
    - 负责在Director请求时提供文件属性和数据
    - 恢复阶段负责恢复文件属性和数据，以后台进程的形式运行
- **Storage Daemon** :读写备份存储介质，执行数据存储和恢复
- **Catalog** :为所有备份文件维护索引和卷数据库
    - 快速定位和恢复归档文件
    - 支持 MySQL、PostgreSQL、SQLite
    - 摘要信息(备份任务、客户端、被备份的文件以及他们存放在哪个卷上)
    - 不存放被备份的文件本身
    - 是bacula区别于简单备份工具的关键
- Monitor :允许监视Dir、FD和SD(仅GTK+ GUI可用)
- C/S、模块化结构，可单机或分布式部署
  最新版支持一次性写入介质和Amazon s3
- 适合大型环境





![架构](http://image.xpshuai.cn/becula_arch.png)



##### 安装配置



###### 1.安装数据库

```bash
sudo apt install bacula-director-pgsql
# 各种根据自己情况选项
# 数据库安装在远程还是本地，创建数据库账号的密码或直接回车他自己会随机生成

sudo su postgres
psql
\l	# 可以查看库
\c bacula	# 连接上这个库
\d
```



###### 2.安装 Director

```bash
# 和上面的数据库是可以不安装在一台机器上的

sudo apt install bacula-director

#-随机生成数据库密码
#-依赖Postfix(后期配置)
#-侦听TCP9101
```



**2.1修改配置文件**

```bash
#1.主配置文件
# vim /etc/bacula/bacula-dir.conf

# 如果要所有网卡都侦听9101，则直接注释当前DirAddress即可
# 否则有多高个网卡要侦听不同的端口，还需要在Director下增DirAddresses,并注释之前的DIRport
Director {
	#......
	DirAddresses ={
	  ip = { addr = 127.0.0.1;port = 9101;}
	  ip = { addr = 192.168.1.1; port = 9101;}	
	}
}

#2.重启服务
sudo systemctrl restart bacula-director

```



###### 3.安装SD(storage)

```bash
sudo apt install bacula-sd  
# 默认监听tcp 9013端口
```



同样也需要修改配置文件

```bash
#1.SD配置文件
# vim /etc/bacula/bacula-sd.conf
#SDAddress和SDPort注释掉，并和上面一样增加
Strage {
	#......
	SDAddress ={
	  ip = { addr = 127.0.0.1;port = 9101;}
	  ip = { addr = 192.168.1.1; port = 9101;}	
	}
}
#2.重启服务
sudo systemctrl restart bacula-sd

```



###### 4.安装FD(客户端)

```bash
sudo apt install bacula-fd

# 
```



配置

```bash
# /etc/bacula/bacula-fd.conf

# 1.FD配置
FileDaemon {
	#FDAddress = 127.0.0.1
	# 这个机器所有ip都侦听9102端口
	}
	
#仅客户端安装

#2.重启服务
sudo systemctrl restart bacula-fd
#默认监听TCP 9102

```





**在网络中其他计算机上安装FD Windows客户端**

- https://sourceforge.net/projects/bacula/files/





```bash
#配置FD

## 客户端配置
#sudo vi /etc/bacula/bacula-fd.conf

Director {
	Name = Server-dir # 指定服务器的名字Name
	Password= "FmvH"	# 
	#director:
}

## 服务器端配置
# sudo vi /etc/bacula/bacula-dir.conf
Client {
	Name = Client-fd  # 指定客户器的名字Name
	Address = 192.168.1.2
	Password ="FmvH"	# 
}

```



###### 5.安装Console

```bash
sudo apt install bacula-console

sudo bconson	#进入控制台
show clients	#配置文件添加成功
status			#验证连接是否成功
```



##### 使用

一些名词：

![1](http://image.xpshuai.cn/becula_n1.png)

![2](http://image.xpshuai.cn/becula_n2.png)





```bash
level = Full	# 全量备份

File = xxxx	# 要备份的目录

Schedule  {}	# 指定备份周期

```





执行磁盘备份(文件)



添加Lable/指定备份客户端

```bash
# sudo vi /etc/bacula/bacula-dir.conf

Pool {
	Name = File
	Label Format = "Vol-S{Year}_${Month}_${Day}" #备份文件名称格式，便于识别和管理
}

Job {
	Name = "BackupClient1"
	JobDefs = "DefaultJob"
	Level = Full
	Client = client-fd
	}
```



指定备份位置

```bash
# sudo vi/etc/bacula/bacula-sd.conf

#
Device {
	Name = FileChgr1-Dev1
	Archive Device = /backups    #需要可写权限
}

Device {
	Name = FileChgr1-Dev2
	Archive Device = /backups2    #需要可写权限
}
```



运行备份

```bash
bconsole - run - BackupClient1
```

查看备份任务状态

```bash
bconsole - status - director — list jobs
message	#查看成功/报错详细信息
```

配置FileSet

```bash
FileSet {
File = /usr/sbin
}
```



报错等待连接存储

```bash
sudo vi letc/bacula/bacula-dir.conf

Autochanger {
Name = File1
#Do not use "localhost"" here
Address = 192.168.1.1  #此处不要使用localhost
}

```



Bacula 通知fd向sd存储数据，若sd侦听于localhost，网络不通









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%A4%87%E4%BB%BD%E8%BF%98%E5%8E%9F/  

