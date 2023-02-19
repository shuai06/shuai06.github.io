# Cobalt Strike DNS Beacon



## 部署域名解析
首先，用一台公网的Linux系统的VPS作为 C&C 服务器 (注意：VPS 的 53 端口一定要开放)，并准备好一个可以配置的域名 (这里我们假设是 hack.com)。然后，去配置域名的记录。  
首先创建记录 A，将自己的域名`www.hack.com`解析到 VPS 服务器地址。  
然后，创建 NS 记录，将`test1.hack.com`指向`www.hack.com` 

第一条 A 类解析是在告诉域名系统，`www.hack.com`的 IP 地址是`xx.xx.xx.xx`
第二条 NS 解析是在告诉域名系统，想要知道`test.hack.com`的 IP 地址，就去问`www.hack.com` 

![](https://image.geoer.cn/20220621150543.png)
为什么要设置 NS 类型的记录呢？因为 NS 类型的记录不是用于设置某个域名的 DNS 服务器的，而是用于设置某个子域名的 DNS 服务器的。


**验证域名解析设置是否成功？**
1.验证A类解析
在随便一台电脑上 ping 域名 www.hack.com ，若能 ping 通，且显示的 IP 地址是我们配置的 VPS 的地址，说明第一条 A 类解析设置成功并已生效。
![](https://image.geoer.cn/20220621150932.png)



2.验证NS解析
先查看VPS的53端口是否打开：
![](https://image.geoer.cn/20220621152040.png)

2.1然后在我们的 VPS 上执行以下命令监听 UDP53 端口:`tcpdump -n -i eth0 udp dst port 53`

2.2在任意一台机器上执行`nslookup test1.hack.com`命令，如果在我们的 VPS 监听的端口有查询信息，说明第二条记录设置成功

![](https://image.geoer.cn/20220621151411.png)



## CS开启监听DNS Beacon
![](https://image.geoer.cn/20220621151802.png)



## 生成DNS木马


## 上线
只要木马在目标主机执行成功，我们的 CobaltStrike 就能接收到反弹的 shell。但是默认情况下，主机信息是黑色的。
我们需要执行以下命令，让目标主机信息显示出来
![](https://image.geoer.cn/20220621152248.png)

```bash
checkin
mode dns-txt
```
![](https://image.geoer.cn/20220621152311.png)






---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/cobalt-strike-dns-beacon/  

