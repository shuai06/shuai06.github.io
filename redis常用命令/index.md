# redis常用命令



  
﻿在Ubuntu系统中**默认配置文件地址**:

```
/etc/redis/redis.conf
```
  

  
```
port6379 	# 默认端口    
logfile /var/log/redis.log 	# 日志文件位置    
dbfilename dump.rdb 		# RDB持久化数据文件    
bind 0.0.0.0	# 指定IP进行监听    

```



#### 组成部分：
  
1. 服务端:   Redis-service

2. 客户端:   Redis-cli

```
# 进入数据库
redis-cli
# 退出
exit
```

   
  
## 基本数据类型：
  
> - String:字符串
> - Hash:哈希
> - List:列表
> - Set:集合
> - Sorted set:有序集合



```
# 查看帮助
help @sorted_set
```



### String:字符串

string是redis最基本的类型，一个key对应一个value。

```
# 设置值
SET key value

# 得到值
GET key

# 将key的值设置为新的，并返回旧的值
GETSET key value_new

# 设置多个键值
MSET name xps sex nan

#取出多个键的值
MGET name sex

# 获取key的值的长度
STRLEN name

# 将该值追加到name的值xps的后面
APPEND name ppp

# 设置持续时间
SETEX name 10 xps

# 如果该key有值就不执行
SET name xxx

# 每次执行将value的数字加1
set age 10
INCR age

# 每次执行将value的数字减1
DECR age
```

### 全局Key操作：

```
keys *    # 查看所有的键
DEL  key
EXISTS  key
RENAME old_key new_key
TYPE key

```



### Hash:哈希（字段里面也有值）

Hashes类型看成具有String Key和String Value的map容器。

```
类似于这种结构：
b = {
  one:xps
  two:xp
  three:x
}
```



```
# 设置该key字段的值
HSET key field value
HSET b one xps
HSET b one xps two xp three x

# 获取该key字段的值
HGET key field
HGET b one

# 获取hash表里面所有的值
HVALS key
Hvals b

# 获取hash表里面所有的字段
Hkeys b

# 获取hash表中的字段数量
HLEN b

```

### List类型（例如：微博评论）

List类型是按照插入顺序排序的字符串链表。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表

```
# 向列表头部插入数据（顺序：c,b,a）
LPUSH names a b c

# 插入数据（x,y,z）
RPUSh names2 x y z

# 获取列表里面某个索引对应的值
LINDEX key index
LINDEX names 0

# 弹出并删除最后一个元素
RPOP names

# 弹出并删除最前面一个元素
LPOP names

# 列表的长度
LLEN key

#
LSET key index value

```



### Set类型(微博粉丝，关注人)

Set类型没有排序的字符集合。如果多次添加相同元素，Set中将仅保留该元素的一份拷贝。

```
# 向集合添加成员
SADD setkey a b c d 

# 查看集合成员数量
SCARD key

#  迭代集合中的所有元素
SMEMBERS key
SMEMBERS setkey

#  判断集合是否存在该元素(存在为1，不存在为0)
SMEMBERS key value
SMEMBERS setkey a
SMEMBERS setkey z  

#删除并返回集合中的一个随机元素 
SPOP setkey

```

### Sorted Set类型（排行榜，top,直播）

每一个成员都会有一个分数(score)(**权重**)与之关联。成员是唯一的，但是分数(score)却是可以重复的。

```
# 向有序集合添加成员，并且给他权重
ZADD key score1 member1 [score2 member2]
ZADD akey 1 a 2 b 3 c

# 显示集合成员数
ZCARD zkey

# 计算有序集合指定区的分数的成员数
ZCOUNT zkey 1 3

# 移除有序集合一个或多个成员
ZREM zkey number

# 查找 迭代有序集合所有元素
ZSCAN zkeys 


```



### 









---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/redis%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4/  

