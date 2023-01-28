# mysql表结构复制



  
﻿**复制结构：仿照student表结构复制一张new_student表,新表是空的：**

```
create table new_student like student;
```
**复制数据：但是没主键，结构不重要：**

```
create table new_student2 as
(    
	select * from student
);
```
**复制表结构与数据，一模一样：**
```
insert into new_student3 select * from student;
```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/mysql-%E8%A1%A8%E7%BB%93%E6%9E%84%E7%9A%84%E5%A4%8D%E5%88%B6/  

