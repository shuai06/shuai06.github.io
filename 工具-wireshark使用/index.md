# 工具 - Wireshark使用




<!-- more -->

# Wireshark

## 简介

- 抓包嗅探协议分析
- 安全专家必备技能
- 抓包引擎(wireshark本身是分析数据包, 抓包靠下面两个引擎)
    - Linux上： Libpcap
    - Windows上： Winpcap
- 解码能力（衡量抓包软件好坏的重要标准）


## 基本使用方法
###### 启动

###### 捕获->选项-> ... ... 
**选择抓包网卡**
<img src="http://image.xpshuai.cn/20200314214656188_15241.png" />

**混杂模式**
勾选上之后，如果数据包不是发送给我指定这个网卡的，也会被抓取

**抓包筛选器**
指定抓什么包，不抓什么包

###### 实时抓包
就是主界面

###### 保存和分析捕获文件
先停止，再保存(可以选择格式, 尽量使用`.pcap`格式)，压缩等, 下次可以导入已经抓取的数据包文件

###### 首选项
编辑->首选项   -> 可以自己根据需要修改和定制界面布局
<img src="http://image.xpshuai.cn/20200314215811593_19130.png" />
<img src="http://image.xpshuai.cn/20200314215743991_3733.png" />


#### 筛选器
- 过滤干扰的数据包
- 抓包筛选器(捕获, 选项里面那个)
- 显示筛选器(主界面那个, 更多使用)    -- 常用语法

可以单独选择，也可以叠加(and, or)上面的条件进行筛选：
<img src="http://image.xpshuai.cn/20200314220435486_29854.png" />
<img src="http://image.xpshuai.cn/20200314220819061_8554.png" />



## 常见协议数据包

数据包的分层结构显示

###### <font color=red>ARP</font>
<img src="http://image.xpshuai.cn/20200315135242728_2582.png" />
<img src="http://image.xpshuai.cn/20200315135851907_10980.png" />

###### <font color=red> IP</font>

**Protocol字段：** 表示上层是： udp--> 17， tcp--> 6, icmp -->2, igmp -->2
四层有好多中协议对应不同的数字
Header checksum字段：校验和, 每两位进行异或
<img src="http://image.xpshuai.cn/20200315141145151_10461.png" />
tcp和udp一般不会有`option`字段（视情况而定）


###### <font color=red> TCP</font>
<img src="http://image.xpshuai.cn/20200315141900321_8252.png" />
<img src="http://image.xpshuai.cn/20200315142047940_28399.png" />

**附：**
三次握手
<img src="http://image.xpshuai.cn/20200315142429852_2643.pn" />

四次挥手
<img src="http://image.xpshuai.cn/20200315142522647_28638.png" />

###### <font color=red> UDP </font>
udp和tcp的类似，自己实践能看懂
udp开销更小
<img src="http://image.xpshuai.cn/20200315142800334_28041.png" />


###### <font color=red> DNS </font>
应用层协议，基于udp
<img src="http://image.xpshuai.cn/20200315143306840_10579.png" />


###### <font color=red> HTTP </font>
<img src="http://image.xpshuai.cn/20200315143952274_22660.png" />
<img src="http://image.xpshuai.cn/20200315144406111_24695.png" />


###### <font color=red> ftp </font>
这里没ftp服务器就先不放图了
ftp也是一个应用层协议

>wishark默认是用端口区分协议(假设我们手动把http不开在80端口，wireshark分不清非标准端口的, 需要我们手动修改.如下图.)

<img src="http://image.xpshuai.cn/20200315144657089_4217.png" />




#### 数据流
比如访问一个网页, 往往产生很多数据包
wireshark提供了<code>数据流</code>

- http
- smtp
- pop3
- ssl 等

**操作(以http为例)：**
<img src="http://image.xpshuai.cn/20200315145059801_20372.png" />
按照上图操作完后，会来到数据流的界面，如下：
<img src="http://image.xpshuai.cn/20200315145205546_28877.png" />



## 信息统计

**1.查看导入数据包文件摘要信息**
导入数据包文件，打开“统计”->“文件属性”：可以看到一些摘要性的信息

**2.节点数**
打开“统计”->“endpoint”： 可以以ip地址区分/udp有多少/二层的包头/其他标准区分 节点的个数等
Tx表示发送, Rx表示接收的数据包

**3.协议分布**
打开“统计”->“协议分级”：可以看到每个协议里面什么协议的占用百分比(DNS流量占用一般都很小,否则很有可能出现异常)

**4.包大小分布**
打开“统计”->“分组长度”：可以看到大包多还是小包多

**5.会话连接**
打开“统计”->“conversations”：可以看到哪两台机器之间数据包会话传送最多啥的

**6.解码方式**
对于非默认端口的数据包，wiresharl只会按端口区分的
右键, 手动decode为指定的协议， 这样wireshark就会按正确的协议来解析

>需要熟悉不同协议数据包的内容, 防止手动解析错误

**7.专家系统**
“分析” -> "专家系统":  可能自动给我们一些提示

>思路：如果很多个数据包，先从统计信息入手，分析出重点，找出方向。然后筛选出来，逐步分析包的内容




## 实践
- 抓包对比nc、ncta加密与不加密的流量
- 企业抓包部署方案  (Wiershark也有不足，大型网络还是需要商业化的软件)
    - Sniffer
    - Cace / riverbed （大部分还是基于wireshark进行了二次开发，改名了应该, 如果需要去网上找吧）
    - Cascad pilot
















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E5%B7%A5%E5%85%B7-wireshark%E4%BD%BF%E7%94%A8/  

