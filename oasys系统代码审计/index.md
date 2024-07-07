# Oasys系统代码审计


<!--more-->





## 简介

 oasys是一个OA办公自动化系统，基于springboot框架开发，数据库使用了Mybatis，使用Maven进行管理的项目



源码下载：https://github.com/misstt123/oasys



因为该项目是基于SpringBoot系统所开发，所以只需要导入数据库即可



按照文档说明修改application.properties配置文件内容:

![image-20240707114239893](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707114239893.png)





Linux下图形验证码不显示问题：https://blog.csdn.net/weixin_52785994/article/details/127362166（tomcat的）

而springboot下如下修改：

对于`application.properties`：

```
server.tomcat.additional-tld-skip-patterns=Djava.awt.headless=true

```

对于`application.yml`：

```
server:
  tomcat:
    additional-tld-skip-patterns: Djava.awt.headless=true

```

![image-20240707143403196](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707143403196.png)

![image-20240707143742552](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707143742552.png)



## SQL注入

由于使用了Mybatis，所以使用`${`搜索看是否有使用拼接并且没有进行过滤



这里发现多个，先从第一个开始看：看看过程中参数是否可控并且有无过滤

![image-20240707143910015](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707143910015.png)



进于allDirector（按住ctrl加鼠标左键），继续跟进看谁调用了allDirector，发现是AddressMapper接口调用的allDirector方法。

![image-20240707145539015](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707145539015.png)

继续向上寻找（ctrl+鼠标左键`allDirector`方法名）：

![image-20240707145625984](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707145625984.png)



发现是在AddController.java文件中，参数值alph就是pinyin，发现pinyin参数可控并且没有防止SQL注入的防御措施，所以此处存在SQL注入：

![image-20240707145802696](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707145802696.png)



接口地址为：`/outaddresspaging`

验证一下：有变化

![image-20240707145936392](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707145936392.png)

![image-20240707145947527](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707145947527.png)









数据包直接放到sqlmap中去跑一下：

```bash
 python sqlmap.py -r post.txt --batch --level 2
```



![image-20240707150428446](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707150428446.png)







其他几个参数也是同样存在注入点的：

![image-20240707150626924](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707150626924.png)



## Mybatis注入的补充知识点

在Mybatis中，

- 占位符`#{}`：对传入的参数进行预编译转义处理。类似 JDBC 中的`PreparedStatement`。
- 拼接符`${}`：对传入的参数不做处理，直接拼接，进而会造成SQL注入漏洞。



但是存在三种不能使用`#{}`情况：

### **1.order by注入**

与JDBC预编译中order by注入一样，在`order by`语句后面**需要是字段名或者字段位置**。因此**不能使用Mybatis中预编译的方式。**

所以，如果此时开发人员在使用order by语句的时候往往会存在注入，如下：

![image-20240707144345767](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707144345767.png)



**正确的使用order by的方式如下：**

```java
// 当插入的数据用户可控时，应使用白名单处理
String orderBy = "{user input}";
String orderByField;
switch (orderBy) {
    case "name":
        orderByField = "name";break;
    case "age":
        orderByField = "age"; break;
    default:
        orderByField = "id";
}
```





### 2.In注入

in 在查询某个范围数据是会用到多个参数(比如`select * from xxx where x in(a,b,c,);`)，**在 Mybtis中如果直接使用占位符`#{}`进行查询会将这些参数看做一个整体，查询会报错。**

因此很多开发人员可能会使用拼接符`${}`对参数进行查询(**因为偷懒/不知道如何正确使用**)，从而造成了SQL注入漏洞，如下：

![image-20240707144542729](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707144542729.png)



在Mybatis中，**正确的in查询的方式**如下：使用foreach配合占位符`#{}`实现IN查询

```xml
<select id="select" parameterType="java.util.List" resultMap="BaseResultMap">
    SELECT *
    FROM user
    WHERE name IN
    <foreach collection="names" item="name" open="(" close=")" separator=",">
      #{name}
    </foreach>
</select>
```







### 3.like注入



使用like语句进行查询时如果使用占位符`#{}`查询时程序**会报错**，如下：`select * from users where username like '%#{username}%'`

此时，经验不足的开发人员可能会使用拼接符`${}`对参数进行查询，从而造成了SQL注入漏洞，如下：`select * from users where username like '%${username}%'`

![image-20240707144914793](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707144914793.png)



**正确的like查询的使用方式**，如下。

```sql
SELECT * FROM users WHERE name like CONCAT("%", #{name}, "%")
```







## 任意文件上传

可以先从前端系统中找上传点

也可以先从代码中搜索关键字（这里先搜索的）



搜索了`fileupload`:(这里是恰巧有这个接口)

![image-20240707151114154](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707151114154.png)



这个接口中直接对前端获取的参数进行获取后没有过滤：

![image-20240707151341295](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707151341295.png)



burp抓包测试：

![image-20240707151435026](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707151435026.png)

![image-20240707151631938](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707151631938.png)

但是是看不到上传路径的，也不知道有无解析

同时也不能进行目录穿越

**无果**



## 任意文件读取

这个直接参考的https://xz.aliyun.com/t/11993

直接复现一下吧



两个漏洞点一样：

src/main/java/cn/gson/oasys/controller/user/UserpanelController.java

src/main/java/cn/gson/oasys/controller/process/ProcedureController.java





代码大概意思是：首先通过getRequestURI方法获取当前访问的相对路径，然后如果路径中存在`/show`，则将该路径中的`/show`替换为空。接下来与rootpath拼接然后通过File打开文件后返回前端。

```java
		/**
		 * 图片预览
		 * @param response
		 * @param fileid
		 */
		@RequestMapping("show/**")   //表示匹配以 /show 为前缀的所有请求，其中 /** 表示任意路径
		public void image(Model model, HttpServletResponse response, @SessionAttribute("userId") Long userId, HttpServletRequest request)
				throws IOException {

			String startpath = new String(URLDecoder.decode(request.getRequestURI(), "utf-8"));  //获取URI路径,并解码
			
			String path = startpath.replace("/show", "");  // 将/show替换为空，那么：..show/..show/..show/..show../666.txt   就会替换为../../../../666.txt
			
			File f = new File(rootpath, path);
			System.out.println(f.getAbsolutePath());
			ServletOutputStream sos = response.getOutputStream();   //获取响应对象的输出流，用于向浏览器发送图片数据
			FileInputStream input = new FileInputStream(f.getPath());  //创建了一个文件输入流，用于读取图片文件的内容
			byte[] data = new byte[(int) f.length()];
			IOUtils.readFully(input, data);   // 读取文件，内容读取到字节数组中
			// 将文件流输出到浏览器
			IOUtils.write(data, sos);
			input.close();
			sos.close();
		}
```

所以，我们只需构造 /show../ 即可替换成 ../ ，最终实现任意文件读取漏洞

```
http://192.168.3.214/show//show..//show..//show..//show..//show..//show..//show..//show..//show..//show../tmp/flag.txt
```



![image-20240707153838881](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707153838881.png)





## XSS

漏洞位于部门管理->添加部门处：

![image-20240707152154680](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707152154680.png)

![image-20240707152206624](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707152206624.png)



可以看到没有过滤直接写入了数据库：

![image-20240707152335576](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707152335576.png)



查看部门列表取出数据也没有进行过滤:

![image-20240707152504331](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707152504331.png)



其他XSS很多







## CSRF





## 越权



































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/oasys%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/  

