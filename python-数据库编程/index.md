# python-数据库编程




## Python DBA API

###### 包含的内容
<img src="http://image.xpshuai.cn/pydb1.png"></img>

###### 访问数据库流程
<img src="http://image.xpshuai.cn/pydb2.png"></img>





#### 与MySQL

```python
import pymysql

db_config = {
    'host': '127.0.0.1',
    'user': 'hhh',
    'password': '123',
    'db': 'test',
    'charset': 'utf8'
}

# 插入
try:
    conn = pymysql.connect(**db_config)
    cursor = conn.cursor()
	
    # 插入
    id = '2018002'
    name =  'admin'
    age = 18
    # insert into student values(id,name,age)
    sql = 'insert into student(id, name, age) values(%s,%s,%s)'
    cursor.execute(sql, (id, name, age))

    # 删除
    # table = 'students'
    # condittion = 'age > 20'
    # sql = 'delete from {table} where {condittion}'.format(table=table, condittion=condittion)
    
    # 查询
    sql = "SELECT password FROM admin WHERE name='%s'" % (name)
    cursor.execute(sql)
    pws = cursor.fetchall()
   
except:
    print('error')
    conn.rollback()
finally:
    conn.commit() # 数据有变动一定记得提交/双保险
    cursor.close()
    conn.close()
```

###### cursor对象支持的方法

<img src="http://image.xpshuai.cn/pydb3.png"></img>



#### 与MongoDB

```python
from pymongo import *

#建立连接
client = MongoClient('127.0.0.1',27017)
#指定数据库
db = client.test
#指定集合
collection = db.col

# print(type(collection),collection)

# python连接mongodb就搞定
mydict ={
    '_id': 6,
    'name': 'admin',
    'age': 18,
    'addr': 'didu'
}

# collection.insert(mydict)
# print(collection.find())
# for i in collection.find():
#     print(i)

# for i in collection.find({'name':'zhanglinlin'}):
#     print(i)

collection.update({'name':'zhanglinlin'},{'$set':{'age':22}},{'mult':'true'})


# collection.remove()
```



#### 与redis

```python
import redis

# # 连接,给定参数ip/port， redis默认端口6379
r = redis.Redis(host='127.0.0.1', port='6379')
# # print(type(r),r)
#
# # 设置键值对
r.set('name', 'admin')
# # 获取该键的值
str = r.get('name')
print(str.decode('utf-8')) # 解码

## 自动解码 参数：decode_responses=True
r = redis.Redis(host='127.0.0.1',port='6379',decode_responses=True)
r.set('name','哈哈')
str = r.get('name')
print(str)

## StrictRedis
r = redis.StrictRedis(host='127.0.0.1',port='6379')
r.set('name','哈哈')
str = r.get('name')
print(str,str.decode('utf-8'))
## `Redis`和`StrictRedis`区别：Redis兼容旧版本python2

#
k_v = {
    'a1':'a',
    'a2':'b',
    'a3':'c'
}
r = redis.StrictRedis(host='127.0.0.1',port='6379')
# 批量设置值
r.mset(**k_v)
# 批量取值
print(r.mget('a1','a2','a3'))

#
r = redis.StrictRedis(host='127.0.0.1',port='6379')
# 往列表添加值从头部开始
r.lpush('list1','haha')
r.lpush('list1',3,4,5)
# 获取列表值
print(r.lrange('list1',0,-1))

#
r = redis.StrictRedis(host='127.0.0.1',port='6379')
r.sadd('set1','aa')
r.sadd('set2','aa',8,10,'bb')
print(r.smembers('set2'))

```



#### 与memcached

```python
import memcache


# 建立连接
mc = memcache.Client(['127.0.0.1:11211'], debug=True)

# 设置一个
mc.set('name', 'xps',time=60)
# 设置多个
mc.set_multi({"username":"handsome", "gender":"man"}, time=60)

# 获取一个
mc.get('age')
# 获取多个
# 不管是元组还是列表都行，只要可迭代就行
res1 = mc.get_multi(("username", "gender"))
res2 = mc.get_multi(["username", "gender"])

print(res1)
print(res2)


# 删除一个
mc.delete("key")
# 删除多个
mc.delete_multi(["key1", "key2"])

"""
# 0表示永不过期  断电就会完蛋   适合做验证码
# prepend 前插
# append  后插
# telnet ip port 远程连接
"""
```



这里仅仅做了简单的介绍，具体还需要自己练习中学习

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/python-%E6%95%B0%E6%8D%AE%E5%BA%93%E7%BC%96%E7%A8%8B/  

