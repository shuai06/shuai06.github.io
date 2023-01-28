# ubuntu server-入侵检测IDS


## 入侵检测(IDS)

- 安全的关键在于看见
- 入侵检测( Intrusion Detection System )
- 设备或者软件， 用于监视网络和系统的恶意、违规行为
- 基于特征、基于异常(流量异常，行为异常..)、基于信誉(依赖机器学习)
- 基于网络的入侵检测 NIDS
- 基于主机的入侵检测 HIDS (常用的方法：`检查系统文件的完整性`)
- `告警`而非实时阻断(`联动`其他手段实现阻断)
- 监视和警告文件/目录的变更
- IPS(入侵防御系统) / IDP（入侵检测防御系统）



#### Tripwire(绊网)

- 开源免费的安全和数据完整性检测及告警软件(有企业版)
- Tripwire 公司
- 军事和工业领域
- Tripwire
- 基于文件完整性检查的HIDS
- 对全新安装的系统创建基线库
- 基于策略生成基线和检查指定文件目录的指定属性(HASH、 Perm、Owner)



**安装**

```bash
sudo apt-get install tripwire

# 同时会提示你安装Postfix
# 要求你设置密码文（两个密钥对 对文件完整性做签名）
# site key		一组服务器主机 相同的site key，适用于重复使用
# local key		只针对我本机的文件加密的情景

# 配置文件会以key加密方式保存的，如果想修改可以修改/etc/tripwire/twcfg.txt然后执行命令来实现
# 要监视的文件目录的策略文件：/etc/tripwire/twpol.txt
```



文件结构

```bash
ls /etc/tripwire

site.key
xps-local.key
tw.cfg	# 加了密的
twcfg.txt	# 配置文件。
tw.pol
twpol.txt	# 策略文件。rulename 策略名

# 其中变化很大的文件要根据情况来进行监控

# 可以在策略文件，自己添加新的规则
```



**初始化数据库**

```bash
ls /var/ib/tripwire/	# 存放文件完整性的数据库
	
	
sudo tripwire --init	# 要先进行数据库初始化
  	No such file or directory	# 会有这个提示信息，Error就不正常了（有的文件不存在）

vi /etc/tripwire/twpol.txt  	# 要注释所有不存在的文件/目录

sudo twadmin -m P /etc/tripwire/twpol.txt   # 然后基于修改之后的策略文件，重新生成pol文件

sudo tripwire --init	#重新初始化数据库
```



**检查验证**

```bash
sudo tripwire --check	# 检查

# 查看验证结果

```



**添加规则**

```bash
vi /etc/tripwire/twpol.txt 
    # Rules for Project
    {
        rulename = "Project Ruleset",
        severity = $(SIG_ HI)
    }
    {
        /project -> $(SEC CRIT);
    }
```

```bash
sudo twadmin -m P /etc/tripwire/twpol.txt
# 先创建上面策略文件中的文件，否则会报错文件不存在
# 重新初始化数据库
sudo tripwire --init
```



**基于上次最新的检查日志(增量型的方式)，来进行更新**

```bash
##更新数据库(本地系统更新)
ll /var/lib/tripwire/report

# 每次数据库变更都要执行一次
tripwire -mu-a -S -C /etc/tripwire/tw.cfg -r /var/ib/tripwire/report/xps.twr

# 或者使用下面这个方式，一次执行即可
tripwire --update -- accept-all


## 查看报告（r 读取。这样的方法更加详细）
twprint -m r -F /var/lib/tripwire/report/xps-2020801-100037.twr

## 查看数据库(d 数据。内容很多)
twprint -m d -d /path/to/database.twd


```





### 补充命令

**硬件信息**

```bash
# 安装
sudo apt install Ishw

# 查看
sudo lshw 

# 输出结果
sudo lshw -html > hwinfo .html
```



**硬盘信息**

```bash
# 安装
sudo apt install hdparm

hdparm -i /dev/sda
hdparm -I /dex/sda	# 更加详细信息
```









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/ubuntu-server-%E5%85%A5%E4%BE%B5%E6%A3%80%E6%B5%8Bids/  

