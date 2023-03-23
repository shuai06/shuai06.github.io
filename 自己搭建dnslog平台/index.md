# 自己搭建dnslog平台


<!--more-->


昨天挖SRC的时候使用公开的dnslog平台，被坑了一把，对方可能检测到了流量的报警，迅速把站关了.....损失一个RCE呜呜  
所以自己搭建一个dnslog平台看看



## 准备工作
- 一台公网VPS  
- 一个域名  
我这里都是用的阿里云的

  
使用的平台：https://github.com/lanyi1998/DNSlog-GO
  



## 配置
1.到阿里云云解析DNS中，进行相应的配置：
  
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20230323151723.png)

然后，在云服务器ECS安全组规则里开放UDP的53端口




2.下载并解压：https://github.com/lanyi1998/DNSlog-GO



3.配置文件：
```bash
 vim config.yaml
```
![image.png](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//image.png.png)


4.（根据情况选择）
我这里阿里云机器遇到53端口占用的问题，解决办法：




5.启动
```bash
./main
```


6.访问dnslog平台即可使用

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//20230323152807.png)

但是我这里有个小疑问，本地解析出来的127.0.0.1  
可能是没设置dns host???


  
其他参考文章：
https://www.f12bug.com/archives/dnslog%E5%B9%B3%E5%8F%B0%E6%90%AD%E5%BB%BA
https://mp.weixin.qq.com/s/m_UXJa0imfOi721bkBpwFg

  








---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E8%87%AA%E5%B7%B1%E6%90%AD%E5%BB%BAdnslog%E5%B9%B3%E5%8F%B0/  

