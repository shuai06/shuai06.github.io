# ubuntu server-账号管理



​    


## <font color=red>用户账号管理基础</font>
**用户账号管理**
- 为新员工创建账号
- 修改账号密码策略
- 禁用休假员工账号
- 删除离职员工账号(一般先禁用, 如果要删除, 注意提前把重要文件备份)
- 设置账号访问权限(只赋予其工作必需的权限)

**每个人保管自己的密码**
- 不要将密码泄露给任何人
- 保证密码复杂度
- 不要将密码写在纸条贴在限制器上

#### 关于Linux账号管理基础知识
**Root账号**
- 能做任何事情（甚至删除操作系统自身）
- 大部分Linux发行版安装时要求设置root账号密码
- 而Ubuntu默认禁用Root账号
	- 可`sudo`或`临时切换为root`账号
	- 可手动启用root账号
- 平时以普通员工账号管理计算机
- 在Ubuntu中执行`sudo rm -rf /`后，有提示

**安装过程中创建管理员个人账号：** 作为日常管理使用


**创建账号**
```bash
# 本机用户账号存储文件: `/etc/passwd`  
# 冒号分隔：第二列是占位符(x来代替密码，此文件不存储密码,在shadow中)，第三四列（uid,组id）root的id是0，第五列是用户描述，第六列是主目录,最后一列是终端类型
# 逗号分隔的部分：

# 创建用户（useradd） 建议添加-m参数
useradd user01 -m   # 一般是这样使用多
useradd user01 -m -d /home/user01   # -d自定义主目录位置，父目录必须是存在

# 如果只是sudo useradd user01    #是没有用户家目录的，且该用户的终端为sh
# 新添加的账号的uid是递增的（系统会自动创建一个和uid相同的组id组账号）

# 参数：
-d		# 账号主目录（复制于/etc/skel）
-m		# 同时创建主目录（默认首次登录时创建主目录）   用户主目录内容拷贝自`/etc/skel`
-c		# 全名（描述,即/etc/passwd中第五列那个）
-e		# 账号过期（YYYY-MM-DD） -- 适合临时工作的人
-N		# 不创建同名组账号(gid会默认设置为100)   useradd user01 -m -N
-g		# 指定主组（必须已经存在）		useradd user01 -m -g sudo
-G		# 额外组



# Note:
账号名最长32个字符（不要以数字或其他特殊字符开头）


# 给用户设置密码
sudo passwd user02
```



## <font color=red>账号数据库</font>
**1./etc/passwd**
格式：`用户名:密码占位符:UID:GID:全名(描述),房间号,公司电话,家庭电话,others:主目录:shell`


**2./etc/shadow**
存放密码等其他相关信息, 默认root才能查看与读写
格式：`用户名:密码:上次修改时间:密码最小使用期限:密码最长使用期:'密码'过期前几天提醒:密码过期几天后账号会被锁定:'账号'过期日(距离1970-01-01的天数):保留`

- 密码位置为`!`、`*`的账号不能直接登陆系统(其他账号登录可切换为其账号) , `!`表示密码被锁定
- 上次修改时间：自从1970-01-01起的`天数`，如果为`0`表示用户下次登陆需要修改密码，`空`表示关闭密码过期功能(不会被锁定)
- 密码最长使用期限<密码最小使用期 时候，用户无法更改密码(一般会设置密码有效期为90天) 
- 密码过期几天后账号会被锁定： 
	- 账号过期用户不能登录
	- 密码过期看是否锁定账号


>账户修改：建议多登陆几个会话备用，防止手滑出现问题导致登不上去


**设置账号密码**
```bash
passwd user01

# 参数：
-l		# 锁定账户密码   sudo passwd -l user01
-u		# 解锁账号   sudo passwd -u user01
-d		# 删除密码（账号无密码可登录）
-n/-x   # 密码最小 / 最大 使用期限
-w		# 密码过期前几天发警告
-i		# 密码过期后锁账号
-e		# 密码立刻过期（下次登录必须修改密码）
-s		# 查看账号的密码状态(L 锁定, P有密码,NP没密码 )  passwd -aS

# 查看账号密码状态（同上）
passwd -a -S
```

**添加账号**
adduser   参数少，更简洁  
```bash
adduser user02
# 基于useradd的perl脚本
# 并非所有Linux发行版中都包含
# 向导方式运行（不需记忆命令参数）
# 位置： /usr/sbin/adduser


# 另一个批量添加账号   newusers
sudo newusers users.txt  # 文件内容按passwd格式填写:   user05:pass15:::hahaha:/home/user15:/bin/bash
```


**删除账号**
userdel或deluser
```bash

sudo userdel user02 # 不会删除主目录
userdel -r user02   # 删除账号同时删除主目录（一般先禁用几个月等到不需要一下文件时候再删除但是要提前备份）
userdel -f user02   # 强制删除文件

rm -rf /home/user02   # 删除用户主目录
```

**切换账号**
```bash
# 切换到root账号
su			# 需要输入root密码（默认root没密码,先给root设密码[sudo passwd root]再操作）
sudo su		# 输入当前账号密码(是输入的当前账号的密码)
sudo -i		# 同上（root前面的命令提示符是井号）  exit退出到自己的回话
sudo -s		# 同上

# 切换到其他账号
su user01	# 切换到其他账号（输入的user01的密码）
sudo su user01	#切换到其他账号（输入的是当前用户的密码）
```



## <font color=red>组账号管理</font>
>分组归类统一管理

- 将用户账号分组管理和指派权限  
- 用户创建同时生成同名的组账号  
- 每个文件有`唯一`的所属账号和所属组（属主、数组）
	- 每个用户可以同属属于多个组，但`只有一个主组`  

查看当前用户所属的组： `groups` 

cat /etc/groups
格式： `组名:密码(通常不使用):GID:逗号分隔的'成员'用户账号`


**组管理**
```bash
sudo groupadd g_name		# 添加组
sudo groupdel g_name		# 删除组

gpasswd -a u_name g_name	# 将用户加入'额外组'
gpasswd -d u_name g_name    # 将用户从组中删除

usermod -aG g_name user	# 将用户加入'额外组'  a是添加,G是组
	# -a    附件（否则替换）
usermod -g g_name user  # 修改用户的'主组'    -g    把user01的主组改为IT: sudo usermod -g IT user01

usermod -d /home/user2 user2 -m  # 将某用户主目录内容移动到当前目录
usermod -l user2 user1		# 将用户user1修改为user2(只是名称换了，其他的权限和id都不变)

# 等等其他参数......

```


## <font color=red>密码策略</font>
使用足够复杂的密码  
Pluggable Authentication Module(PAM)  
```bash
# 安装pam
apt install libpam-cracklib

# vim /etc/pam.d/common-password    增加配置
password      required    pam_pwhistory.so.remember=9 use_authok
# 最近几次的密码不能相同
```

###### 权限
每个文件和文件夹拥有唯一的属主和属组（`ls -l`）

**权限类型：**
- r, w, x			读，写，执行		4, 2,	1 （这个数字是二进制转换来的，假设数组的权限为r--,对应二进制为100,转为十进制为4，其他类似）
- u，g，o		属主，属组，其他

```
# 除了第一位，后面每三位一组（所有者u，所有组g，其他用户o）
-rwxrwxrwx

# 权限		文件					   文件夹
# R		读取文件内容，拷贝		 列出目录内容
# W		更改文件内容				 创建、删除文件及文件夹
# X		作为应用程序执行文件		 cd进入；读取文件属性和权限
```

**权限修改命令：**
chmod
```bash
## 只有其属主和root可以修改权限

## 对文件
# + - 方式
chmod u+x a.txt
chmod u-r a.txt
chmod g-x a.txt
chmod o+r a.txt

# 数字方式 
chmod 777 b.txt
chmod 764 b.txt


## 对目录(一般都给x权限，不然cd不进去)
chmod u+x code/
chmod u-wx code/  
# 对目录：r只能看到里面的文件，要写看到属性，得加x权限
# 父级目录的权限/子文件的权限

# 递归修改权限
chmod 770 -R code/


## 权限与搜索
find . -perm 770   # 搜索权限为770的文件
#  -type f   只搜文件；    -type d  只搜目录
find /path/dir/ -type f perm 664 -user xps -group xps  # 属于xps组，属于xps用户，权限为664的文件
find /path/dir/ -type f perm 755 -user xps -exec ls -l {}\;   # 搜索完之后执行命令
find /path/dir/ -type f perm 755 -user xps -exec chmod 770 {}\;   # 搜索完之后再进行权限修改

# 如果用户删了，并且该用户有文件没删除干净，下面找到没有所属用户的文件或文件夹
find /home -nouser -exec rm -rf {}\;    # 查找无有效属主对象，然后删除
```



## <font color=red>属主与属组</font>
**修改属主和属组**
```bash
# chown需要sudo
sudo chown user01 a.txt   # 修改a.txt的所有者为user01
sudo chown -R user01 /code   # 递归修改目录的所有者

# 修改所有组为user01
sudo chgrp user01 a.txt
# 所有者和所有组都修改
chown user01:user01 a.txt
```

**特殊权限**
1. Setuid(4)
主要针对可执行程序  
一旦设置了suid之后：任何用户执行程序时都具备使用属主的权限
设置：`chmod u+s a.sh`或`chmod 4777 /bin/cat` (前面三个二进制, 二进制为100,即4；第二种数值方式对链接不生效)
比如使用`passwd`修改自己密码的时候修改的是shadow文件(他的属主是root)，它就是设置了`s`权限
<img src="http://image.xpshuai.cn/20200318203240319_8090.png" />
>如果不太懂，尽量不要随便设置suid，否则后患无穷

2. Setgid(2)
主要针对可执行程序和目录
对于程序一旦设置了sgid：任何用户执行程序时都具备使用属组的权限
应用于目录时：可实现`共享文件访问效果`(新建文件的数组集成目录属组):  一个用户再在里面新建文件之后，一个组里的用户就都可以共享
设置：`chmod g+s code/`或`sudo chmod 2775 /bin/cat`
设置完之后，数组位置权限就多了s权限
<img src="http://image.xpshuai.cn/20200318203113628_4438.png" />

3. stick bit(1)
`针对目录`的受限删除位（root、属主可以删）
典型例子：`/tmp`目录
设置： `chmod o+t code/`
加了stick位(其他用户的位置)，自己创建的文件只有自己能删除，别人无法删除
<img src="http://image.xpshuai.cn/20200318202916501_30856.png" />



## <font color=red>权限管理</font>
###### 掩码

决定文件和目录的`默认权限`  
	- 文件默认权限：`666`-掩码（如果掩码为002, 6-2=4，权限就为664）
	- 文件夹默认权限：`777`-掩码（如果掩码为002, 7-2=5，权限就为775）

命令：`umask`查看当前掩码
修改掩码命令：`umask 007`（临时性的）
**长久修改：**
配置文件`vi /etc/login.defs`
```
# 配置其中umask
UMASK 	022   # 属主的掩码0，数组的掩码2，其他用户的掩码2
USERGROUPS_ENAB	yes  # 改为yes，属主的掩码来覆盖数组的掩码
# 要想使得真正的权限与umask设置的一致：
F1.可以把上面的改为no
F2.设置用户账号名称和组账号名称不一样，让用户id和组id也不一样（推荐这种方式修改）
sudo usermod -g users user01 # 把用户的主组改为users

```

umask
	002			用户名与组名相同、UID与GID相同

umask
	027     		通常设置027就行





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ubuntu-server-%E8%B4%A6%E5%8F%B7%E7%AE%A1%E7%90%86/  

