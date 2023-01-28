# iptables与firewall



  
# iptables与firewall防火墙


  
- **配置网卡文件**  
1. 修改配置文件  
```
vim /etc/sysconfig/network-scripts/ifcfg-eno16777736

#文件内容如下
HWADDR="00:0C:29:C2:F1:76"
TYPE="Ethernet"     #类型
BOOTPROTO="dhcp"
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736"
UUID="af3aaad5-b615-493c-872e-76503b9adad5"  #唯一标识
ONBOOT="yes"   #开机自启
```

 2. 基于图形界面配置网卡  **nmtuio** 
 ```
 nmtuio 

systemctl restart network  
 ```


3.  nm-connection-editor工具  
```
nm-connection-editor
```

4. 图形界面点击右上角



- **配置防火墙**  
规则： 自上而下做匹配
几种状态
```
accept  允许流量通过
reject    明确拒绝
drop        对方不知道你是否在线(显示超时)


```

1. **iptables**  
```
比较老,逐渐被取代
基于命令行
难度最高

iptables -L    #显示所有的策略
iptables -F       #清空所有策略
iptables -P INPUT DROP     #默认禁止所有流量策略,不能reject

-I  #插入到最前面
-A   #插入到最后面
-p  # 允许的协议
--dport     #本机端口
-s    #来源
-j    #本身为意义，后面跟动作


iptables -I  INPUT -p icmp -j ACCEPT  #只允许外部到内部的ping

iptables -I INPUT -s 192.168.10.0/24 -p tcp  --dport 22 #允许来自某个网段的访问，端口为22

-D #删除某一条
iptables -D INPUT 1  #删除第一条

iptables -I INPUT -p tcp --port 22 -j REJECT  #禁止外部所有22端口的流量

iptables -I INPUT -p tcp --port 22:200 -j REJECT  #禁止外部所有20到200端口的流量


service iptables save

```

**firewalld 防火墙(分为下面两个)** 
```
zone  区域
sbit    安全位
public    默认策略,当前使用

```

2. **firewall-cmd 命令行**
```
Runtime  当前生效，重启后失效
Premanent 当前暂时不生效，但是永久生效
--premanent       # 建议添加这个

firewall-cmd -reload     #重启

firewall-cmd --get-default-zone    #查看当前区域

firewall-cmd --set-default-zone=drop    #切换默认区域到丢包

firewall-cmd --permanent --zone=drop --change-interface=en0167777  #重启之后，把这个网卡区域换为drop
firewall-cmd --permanent get-zone-of-interface=en0167777  #重启之后，查看

紧急模式
firewall-cmd --panic-on    #切断所有网络连接，包括ping的数据包
firewall-cmd --panic-off    #关闭紧急模式

查询服务
firewall-cmd --zone=public --query-service=http    #查看是否允许http
firewall-cmd --zone=public --query-service=ssh

添加允许服务
firewall-cmd --zone=public --add-servcice=http    #允许从外访问http服务

firewall-cmd --zone=public --add-servcice=https    #重启才可以 firewall-cmd --reload
firewall-cmd --zone=public --add-servcice=https

禁止服务
firewall-cmd --zone=public --remove-servcice=https

firewall-cmd --permanent --zone=public --add-servcice=https
firewall-cmd --permanent --zone=public --query-servcice=https    


添加端口号
firewall-cmd --zone=public --add-port=8080/tcp    #把这个端口添加到ctp协议组，允许
firewall-cmd --zone=public --add-port=80-90/tcp    #端口端


端口转发（隐藏原始端口）
firewall-cmd --permanent --zone=public --add-forward-port=888:porto:tcp:toport=22:toaddr=192.168.10.10    #转发888到22
firewall-cmd --reload


富规则（复规则）
精准匹配
firewall-cmd --permanent --zone=public --add-rich-rule="rule family"="ipv4" source address="192.168.10.0/24" service name="ssh" reject"

firewall-cmd reload

```


3. **firewall-config  图形化**
```

firewall-config


```


4. **TCP Wrappers**

```
（优先级：先匹配允许，再匹配拒绝）
/etc/hosts.allow    #白名单
                                允许服务名称、IP地址
/etc/hosts.deny    #黑名单
                                拒绝服务名称、IP地址

白名单(编辑后就生效)
sshd:192.168.10.0/24    #允许这个网段的主机访问


黑名单(编辑后就生效)
sshd:192.168.10.*        # 所有主机，通配符


```



1.2.3.    数据链路层（协议，端口号，来访IP地址）
4.          应用层（服务名称）

四个工具，只要有一个把他禁止，就禁止了！

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/iptables%E4%B8%8Efirewall%E9%98%B2%E7%81%AB%E5%A2%99/  

