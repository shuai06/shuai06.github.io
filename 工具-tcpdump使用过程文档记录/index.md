# 工具 - tcpdump使用&过程文档记录






# <font color=red>工具-tcpdump使用</font>

- No_GUI的抓包分析工具

- Linux、Unix系统默认安装


###### 抓包
默认只抓68个字节
```bash
#参数:  -i网卡    -s 抓包大小   -w保存到文件
tcpdump -i ens33 -s 0 -w file.pcap

#抓取指定端口的流量
tcpdump -i ens33 port 22
```

###### 读取抓包文件
```bash
#查看保存到文件中的数据包
tcpdump -r file.pcap
#参数： -A 以ascii码形式显示内容
tcpdump -A -r file.pcap
#参数： -X 以16进制形式显示内容
tcpdump -X -r file.pcap
```

###### 显示筛选器
```bash
#利用OS自带的管道
#参数： -n 以ip地址显示，不显示域名；   显示第三列(ip+端口)并去重
tcpdump -n -r file.pcap | awk '{print $3}' | sort -u

#tcpdump自身的显示筛选： 只显示源地址为这个的
tcpdump -n -src host 192.168.0.1 -r file.pcap
#只显示目的地址为这个的
tcpdump -n -dst host 192.168.0.130 -r file.pcap
#按端口号筛选
tcpdump -n port 53 -r file.pcap
#显示udp的53端口
tcpdump -n udp port 53 -r file.pcap
#显示tcp的
tcpdump -n tcp port 80 -r file.pcap
#显示icmp的
tcpdump -n icmp -r file.pcap
#显示ip的
tcpdump -n ip -r file.pcap
#-X以16进制显示
tcpdump -nX port 80 -r file.pcap

```

###### 高级筛选
TCP的包头：
<img src="http://image.xpshuai.cn/20200315180543581_5798.png" />
可以看到上图中Flag中的标志位  
tcpdump可以对如此细致的东西进行筛选  

```bash
#只显示第13号字节中值转换为10进制为24(push+ack)的数据包
tcpdump -A -n 'tcp[13]24' -r file.cap

```



>PS:  nc、Wireshark、tcpdump必须非常熟练地掌握



# <font color=red>过程文档记录工具 </font>
介绍几个kali中自带的

###### 1.Dradis
基于web的工具
- 短期临时小团队资源共享
- 各种插件导入文件（兼容很多种扫描器日志的导入）

**使用：**
webapp  
初始化后，输入密码即可登录
导入导出(支持不同文件格式)扫描器日志
（具体自己操作一下就知道了）


###### 2.Keepnote
层级化记录信息：
不同项目可以使用不同文件夹，子文件
可以导出

###### 3.Truecrypt
一款`加密工具`
2014年停止更新（官方原因是安全性不够，但实际使用却依然较安全）

**选择加密区域：**
1.创建加密文件（create volume）
2.创建卷(全盘加密)  

**选择加密方式**
1.标准加密卷：
2.隐藏加密卷： 若被人强迫输入加密密码，可以输入一个密码打开后什么都没有(outer volume), 而重要信息存在一块隐藏空间(hidden volume)里面。hidden volume大小不能超过outer volume。【首先指定外部卷(迷惑别人的)的大小(尽量大一些,包括内部卷的大小)和密码, 内部机密卷存在于外部卷的空间中(设置它的大小和密码)】


选择加密算法（也可以多种加密算法叠加用）  
设置密码  
指定卷格式  
生成随机key(用鼠标再屏幕上乱滑,划得月乱，key越复杂)


**使用：**
选择挂载的文件，就像挂载一块硬盘分区一样
使用完成后：`dismount`或`dismount all`就行

对于隐藏加密卷: 输入外部卷的密码就是外部卷的内容(存储大小也隐藏了另一部分)，输入内部卷的密码打开的就是内部卷的内容









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E5%B7%A5%E5%85%B7-tcpdump%E4%BD%BF%E7%94%A8%E8%BF%87%E7%A8%8B%E6%96%87%E6%A1%A3%E8%AE%B0%E5%BD%95/  

