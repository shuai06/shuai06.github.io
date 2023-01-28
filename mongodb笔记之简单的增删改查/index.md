# MongoDB简单的curd


  
## ﻿集合创建：

```
db.createCollection("test_col,",{capped:true, size:10})

# "test_col"  表名字(集合名字)
# capped， 默认false，不设置上限，true设置上限

```

## 查看当前数据库的集合：
  
```
show collections
```

## 删除集合：
  
```
db.集合名称.drop()
```



### 查询：
  
```
# 查找集合中所有的数据
db.collection_name.find() 	# 查询文档

# pretty() 方法以格式化的方式来显示所有文档 美观
db.collection_name.find().pretty()

# 指定_id查找
db.collection_name.find({_id:1}).pretty()
```

**_id：**

> 如果插入数据不给定id，他会自动创建，可以通过id查找文档

### 插入：

```
# 向集合插入文档
db.collection_name.insert(document)

例子：db.col_test.insert({name:'xx', gender:'nan'})
（在集合不创建的时候也可以，集合会自动被创建）

```

### 更新：

```
db.collection_name.update({}) 	# 更新文档

db.collection_name.update({'count':88},{$set:{'count':89}})
# count 由88变成89，只会作用于第一条数据

例子：db.集合名称.update({name:'xx'}, {$set:{'name':'xps'}},  {multi:true})
将name为xx的改为yy， multi多行，默认false，只作用于第一个，为true时修改多条

# 更新多行，这个3.2的版本才支持
db.col_name.updateMany()
```

### 删除：

```
db.collection_name.remove({}) 	# 删除集合所有文档 全部删除

db.集合名称.remove({gender:'nan', {justone:true}})  # 依据条件删除一条
justone默认false，删除多条

#删除多条3.2版本才有
db.col_name.deleteMany()

# 删除集合
db.col_name.drop()
```

### **保存** （如果集合不存在，则执行添加操作）

```
db.集合名称.save(document)
```



****

### 数据类型

object ID  文档ID    （不会重复，12字节的16进制数）

String 字符串

Boolean  存储一个布尔值

Integer 整数

Double 浮点值

Arrays 数组或列表

Object 用于嵌入式的文档，即一个值为一个文档

Null 存储NUll值

Times tamp 时间戳

Data 当前日期活时间的UNIX时间格式

object ID  ：

（不会重复，12字节的16进制数，前4当前时间，...)


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/mongodb%E7%AC%94%E8%AE%B0%E4%B9%8B%E7%AE%80%E5%8D%95%E7%9A%84%E5%A2%9E%E5%88%A0%E6%94%B9%E6%9F%A5/  

