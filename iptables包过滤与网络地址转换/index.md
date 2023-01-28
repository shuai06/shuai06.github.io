# iptables包过滤与网络地址转换




## iptables

#### 功能

4个表`raw`和`mangle`，`nat`,`filter`（后两个表用的多）

```bash
iptables -t filter -nvL	# -t显示哪个表
```

三个链(Chain)：`INPUT`,`FORAWRD`(转发规则链),`OUTPUT`



#### 特点

- 比较老,逐渐被取代
- 基于命令行
- 难度最高

#### 用法

```bash
iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型]

# 注意事项：
不指定表名时，默认指filter表
不指定链名时，默认只表内是所有链
除非设置链的默认策略。否则必须指向匹配条件
选项、链名、控制类型使用大写字母，其余均为小写

```

几种控制类型

```bash
ACCEPT  # 允许流量通过
REJECT    # 明确拒绝(比如ping时候的"主机不可达")
DROP        # 对方不知道你是否在线(显示超时)
LOG		# 记录日志信息
```



规则优先顺序：`自上而下`



#### 包过滤

用法

```bash
iptables -L    # 显示所有的策略
iptables -F       # "清空所有"策略
iptables -F FORWARD	# 清空FORWARD表所有规则
iptables -P INPUT DROP     # 默认禁止所有流量策略,不能reject

## 参数
-I   # 插入到最前面(用的较多)
-A   # 插入到最后面(用的较少)
-D	 # 删除
-p   # 允许的协议
--dport     # 本机端口（属于隐含匹配，要配合通用匹配使用）
-s    # 来源地址
-d	  # 目的地址
-i	  # 入站网卡
-o	  # 出站网卡
-j    # 本身为意义，后面跟动作

-L	# 列出所有规则条目
-n:	# 以数字形式显示地址、端口等信息
-v:	# 以更详细的方式显示规则信息
--lines-numbers: # 查看规则时，显示规则的序号
-mac-source MAC地址	# MAC地址匹配（属于显式匹配）


iptables -I  INPUT -p icmp -j ACCEPT  #只允许外部到内部的ping

iptables -I INPUT -s 192.168.10.0/24 -d 0.0.0.0 -p tcp  --dport 22 #允许来自某个网段的访问，端口为22

-D #删除某一条
iptables -D INPUT 1  #删除第一条(后面跟行号)

iptables -I INPUT -p tcp --port 22 -j REJECT  #禁止外部所有22端口的流量

iptables -I INPUT -p tcp --port 22:200 -j REJECT  #禁止外部所有20到200端口的流量

# 保存规则（永久生效）
service iptables save

# 导出当前规则状态
iptables-save > /usr/share/1.txt
# 从文件导入规则
iptables-restore < /usr/share/1.txt

```

 



#### 网络地址转换(NAT)

> NAT表



```bash
# 查看NAT表规则 
iptables -t nat -nvL

路由后规则（POSTROUTING）

## 设置规则
# -s 是给内网的谁做转换, --to-source是转为网关的地址
iptables -t nat -A POSTROUTING -p tcp -o eth1 -s 192.168.1.0/24 -j SNAT --to-source 12.34.56.78 

# 如果网关地址是动态的，就这样
iptables -t nat -A POSTROUTING -p tcp -o eth1 -s 192.168.1.0/24 -j SNAT --to-source -j MASQUERADE

## 配置FORWARD链的转发限制


```



#### 目标地址转换(DNAT)

> 情景：外网机器访问内网机器

使用链：`路由前规则（PREROUTING）`



```bash
# 查看NAT表规则 
iptables -t nat -nvL

# 设置规则，访问内网主机的8080端口（网关的80映射到内网主机的8080）
iptables -t nat PREROUTING -i eth1 -d 12.34.56.78 -p tcp --dport 80 -J DNAT --to-destination 192.168.1.1:8080
```









## firewalld 防火墙

```bash
zone  #区域
sbit    #安全位
public    #默认策略,当前使用
```



```bash
Runtime  # 当前生效，重启后失效
Premanent # 当前暂时不生效，但是永久生效
--premanent       # 建议添加这个

firewall-cmd -reload     #重启

firewall-cmd --get-default-zone    #查看当前区域

firewall-cmd --set-default-zone=drop    #切换默认区域到丢包

firewall-cmd --permanent --zone=drop --change-interface=en0167777  #重启之后，把这个网卡区域换为drop
firewall-cmd --permanent get-zone-of-interface=en0167777  #重启之后，查看

##紧急模式
firewall-cmd --panic-on    #切断所有网络连接，包括ping的数据包
firewall-cmd --panic-off    #关闭紧急模式

##查询服务
firewall-cmd --zone=public --query-service=http    #查看是否允许http
firewall-cmd --zone=public --query-service=ssh

##添加允许服务
firewall-cmd --zone=public --add-servcice=http    #允许从外访问http服务

firewall-cmd --zone=public --add-servcice=https    #重启才可以 firewall-cmd --reload
firewall-cmd --zone=public --add-servcice=https

##禁止服务
firewall-cmd --zone=public --remove-servcice=https

firewall-cmd --permanent --zone=public --add-servcice=https
firewall-cmd --permanent --zone=public --query-servcice=https    


##添加端口号
firewall-cmd --zone=public --add-port=8080/tcp    #把这个端口添加到tcp协议组，允许
firewall-cmd --zone=public --add-port=80-90/tcp    #端口端


##端口转发（隐藏原始端口）
firewall-cmd --permanent --zone=public --add-forward-port=888:porto:tcp:toport=22:toaddr=192.168.10.10    #转发888到22
firewall-cmd --reload


##复规则（复规则）
##精准匹配
firewall-cmd --permanent --zone=public --add-rich-rule="rule family"="ipv4" source address="192.168.10.0/24" service name="ssh" reject"

firewall-cmd reload

```













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/iptables%E5%8C%85%E8%BF%87%E6%BB%A4%E4%B8%8E%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2/  

