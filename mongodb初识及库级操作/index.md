# MongoDB初试及基本操作



  
#### 分布式：

一个任务分成多个子任务完成  （副手）

#### 集群：

一个任务由多态服务器同时服务



**大数据**

文档结构，海量数据，大容量存储，高校存储大量二进制文件



**场景：**

1.需要不断扩容；

2.新应用，需求会变，数据模型无法确定；

3.高可用（不宕机）

4.需要整合多个外部数据源



|  **SQL术语**  | **MongoDB术语** |        **解释说明**         |
| :---------: | :-----------: | :---------------------: |
|  database   |   database    |           数据库           |
|    table    |  collection   |         数据库表/集合         |
|     row     |   document    |        数据记录行/文档         |
|   column    |     field     |          字段/域           |
|    index    |     index     |           索引            |
| table joins |               |     表连接，MongoDB不支持      |
| primary key |  primary key  | 主键，MongoDB自动将_id字段设置为主键 |

文档：就是一个对象，由键值对构成，是json的一种扩展 Bson形式

字段(域)之间可以是不同类型:

``{‘book’:'xxx', 'hero':'yyy'}`

`{'name':'sjz', 'age':18}`



**下载完成后，将可执行文件添加到PATH路径中**

export PATH=/user/local/mondodb/bin:$PATH

**服务端mongod**

配置文件位置  etc/mongod.conf

默认端口  27017

**查看端口：**

```
ss -tnl
```

**数据库服务启动/停止/重启**

```
sudo service mongodb start/stop/restart
```

**查看状态：**

```
sudo service mongodb status
```

**客户端启动**

```
mongo
```

**连接mongodb:**	

```
# 进入
mongo	
# 退出
exit 或者 ctrl + c 或 quit()
```



## 数据库级操作：

**1.查看所有数据库**

```
show dbs
```

**2.创建数据库**

```
use mydb

# 创建之后，如果不插入数据的话， show dbs是看不到刚建立的数据库的
```

3.显示当前数据库

```
db
```

**4.删除数据库**

```
db.dropDatabase()  # 注意大小写
```







---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/mongodb%E5%88%9D%E8%AF%86%E5%8F%8A%E5%BA%93%E7%BA%A7%E6%93%8D%E4%BD%9C/  

