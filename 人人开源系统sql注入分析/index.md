# 人人开源系统 Mybatis Plus SQL注入分析


<!--more-->









开源代码地址：https://gitee.com/renrenio/renren-security



已知接口信息为`/sys/log/error/page `，按照关键字全局搜索，定位到代码位置：`renren-admin/src/main/java/io/renren/modules/log/controller/SysLogErrorController.java`

![image-20240716104348126](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716104348126.png)





![image-20240716104822121](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716104822121.png)

![image-20240716105359334](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716105359334.png)

注入点参数为`ORDER_FIELD`，跟进service层`sysLogErrorService`



![image-20240716105009789](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716105009789.png)



调用了`selectPage`方法进行查询

继续跟进`getPage`方法



![image-20240716105244074](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716105244074.png)

看到从前端获取两个参数`Constant.PAGE`和`Constant.LIMIT`并且二者不能为空

然后进行排序：把`orderField`字段传给`OrderItem.asc`，然后再将结果添加到page对象的order属性中去



继续跟进`OrderItem.asc`: 是一些Mybatis-plus中的初始化的操作

![image-20240716105729710](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716105729710.png)



继续跟进`page.addOrder`:

![image-20240716105959219](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716105959219.png)

```java
    /**
     * 排序字段信息
     */
    @Setter
    protected List<OrderItem> orders = new ArrayList<>();
```



service层的getPage方法跟踪结束，主要作用是：获取前端排序字段的值，加入到page类的order属性中去。

引用一下刚学到的师傅的话：

> 这里 order 属性是个关键，该项目使用了 MyBatis Plus，MyBatis Plus 会根据实体类中的 orders 字段 （如果存在的话）或者 orderBy 方法来自动生成 ORDER BY 子句。这是 MyBatis Plus 提供的一种方便的排序机制。 在 MyBatis Plus 中，orders 字段通常是一个 List 类型的字段，用于描述排序的条件。 OrderItem 包含了要排序的列名、排序方式等信息。





回到`SysLogErrorServiceImpl`，继续跟踪到Dao层：`baseDao.selectPage`直接将page对象传递给了`selecttList`方法，且`selectList`方法中没有对page对象进行过滤：

![image-20240716110404805](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716110404805.png)



> 引用：
>
> 该方法为采用了 Mybatis 的代理开放方式的实现，通过直接创建接口来实现增删改查，这里方法就直接进行了数据库查询，在 page 类中存在排序问题（orderField 参数），导致 sql 语句中使用**动态排序**，存在 ORDER BY 语句对 orderField 参数进行${}拼接，从而产生了 sql 注入漏洞

![image-20240716110642842](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716110642842.png)



验证：



![image-20240716110828060](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716110828060.png)









参考文章：https://tttang.com/archive/1726/#toc_order-by



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E4%BA%BA%E4%BA%BA%E5%BC%80%E6%BA%90%E7%B3%BB%E7%BB%9Fsql%E6%B3%A8%E5%85%A5%E5%88%86%E6%9E%90/  

