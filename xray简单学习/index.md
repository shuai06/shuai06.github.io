# xray简单学习


# xray学习

平时使用`webscan`是最多的


结合基础爬虫

```powershell
xray.exe webscan --basic-crawler http://xxxxx

# 有弊端（爬虫是不受我们控制的。如果是代理就行了:我们点哪里就扫哪里）
```



## 基本使用

#### 代理

###### 原理

![](http://image.xpshuai.cn/xray%E8%A2%AB%E5%8A%A8%E4%BB%A3%E7%90%86.png)

###### 设置代理步骤

![](http://image.xpshuai.cn/xray%E8%AE%BE%E7%BD%AE%E8%A2%AB%E5%8A%A8%E4%BB%A3%E7%90%86.png)

其中，证书的生成(只要程序位置不变，证书就不需要重新安装)：`ca.crt`, 然后安装证书

生成报告后(可以边扫边看)，点击对应条目，我们可以看到具体的请求/响应信息等


## 高级用法

#### 1.常用配置

>配置文件：config.yaml

###### 允许扫描的域

- Mitm -> Restriction
  默认会把所有流量扫描，可以指定我们想要扫描的域名的流量
  includes允许扫描的域，`*`表示任意；excludes不允许扫描的域

###### 为代理添加认证

- Mitm ->auth
  如果把xray放到公网上，别人也可以使用，我们就需要设置账号密码

###### 扫描插件配置

- Plugins
  插件可以打开(`enable:true`)和关闭(`false`),也可以自定义字典等  
  或者直接在命令行指定启用哪些插件：`--plugins xss,xxe`

###### 发包速率限制

- http -> max_qps
  限制扫描速度，防止被waf拉黑

###### 扫描代理配置

- http -> proxy
  代理支持以下形式

```
#1.http代理
http:
  proxy: "http://127.0.0.1:8080" # 漏洞扫描时使用的代理

#2.socks代理
http:
  proxy: "socks5://127.0.0.1:1111" # 漏洞扫描时使用的代理
```

**log如果改成`debug`，可以看到具体发包请求详情：**
![](http://image.xpshuai.cn/xray_log.png)

**也可以设置cookie**

大致配置项：
![](http://image.xpshuai.cn/xray_cookie.png)

#### 2.xray与burp联动使用

###### （1）Burpsuite作为xray的上游代理（`抓取xray发包来学习`）

![](http://image.xpshuai.cn/xray_after_burp.png)

**作用：**
burp可以拿到xray的数据包（可以学习xray是如何检测的的）

**设置方法：**

1. xray设置代理: `http://127.0.0.1:8080`
2. burp监听8080端口
3. 启动xray，设置监听端口为`--listen 127.0.0.1:1111`
4. 浏览器配置代理为`1111`



###### （2）xray作为Burpsuite的上游代理（`协助测试`）

![](http://image.xpshuai.cn/xray_before_burp.png)

**作用：**
burp可以选择是否放行，forward之后，可以用xray进行扫描

**设置方法：**

1. xray配置文件不配置代理
2. burp正常设置监听8080端口
3. burp打开`user option -> Upstream Proxy Server -> 添加为:127.0.0.1:1111`
4. 浏览器设置burp代理8080
5. 启动xray，设置监听端口为`--listen 127.0.0.1:1111` 

**其他方式：**
使用插件




## 使用xray编写自定义poc

xray是用go语言开发的  
poc是yaml格式的

###### 原理

![](http://image.xpshuai.cn/xray_poc.png)

每个规则都是http请求，进行改造


###### POC构成

- name
- rules：最关键部分。请求方法,路径,表达式(有唯一的返回值)。支持多条(就会发送几条数据包)
- details：漏洞检测成功之后，输出一些提示信息
  ![](http://image.xpshuai.cn/poc.png)



###### 具体操作

>看官方文档

vscode和jetBrain都行

vscode安装`YAML`查看，配置文件`setting.json`用来校验

按照上面的格式编写  

运行：`xray.exe webscan --plugins phantasm --poc ./poc-yaml-test.yml --listen 127.0.0.1:1111 `

对单个url扫描：
`--url http://www.test.com` 更好用

Note:	phantasm这里是在命令行启用



详细内容查看官方文档


> 调试poc的方法：设置上游代理为burp，在burp查看xray发送的数据包

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/xray%E7%AE%80%E5%8D%95%E5%AD%A6%E4%B9%A0/  

