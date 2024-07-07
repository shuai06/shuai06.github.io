# Inxedu系统代码审计


<!--more-->



## 项目简介



因酷开源网校系统是由北京因酷时代科技有限公司以下简称（因酷教育软件）研发并推出的国内首家Java版开源网校源代码建站系统，并免费提供给非商业用途用户使用，是用户体验最好、运营功能最全、性价比最高的在线教育软件。

该项目是**SSM框架**





**项目部署：**

1.配置数据库(这里我弄的时候存在一个问题：由于Linux系统对大小写敏感，所以如果在类Unix系统上部署环境，代码中数据库中数据表的大小写会影响，建议将代码中数据表的名字都统一修改为小写)

![image-20240706230313281](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706230313281.png)

2.配置tomcat

![image-20240706230130768](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706230130768.png)



前台账户密码：

lmx193@163.com

111111



后台账户密码：

admin

111111



- `web.xml`：程序启动时tomcat会首先加载web.xml中的配置 。通过web.xml完成DispathcheServlet的声明，并将我们的请求转发到springmvc中。我们可以首先查看web.xml中是否配置了全局过滤器。判断是否能够bypass。

- `applicationContext.xml`是spring核心配置文件,这里会加载一些其他的配置文件。

- `sping-mvc.xml`文件中主要的工作是：启动注解、扫描controller包注解；静态资源映射；视图解析（defaultViewResolver）；文件上传（multipartResolver）;返回消息json配置。



## pom.xml

审计一套系统，可以先看看pom.xml中加载了那些组件 ，如果这些组件中本身存在漏洞，就可以直接利用这些漏洞。







## 思路

1. 以文件上传为例，**先黑盒寻找**并测试文件上传**功能点**，**抓包得到对应路由等信息，从而定位到功能实现的具体代码**（代码溯源），正向然后审计其是否存在漏洞。
2. 通过**关键字搜索**关于上传功能的代码，查看是否存在漏洞，如果存在则**反向追踪**代码，找到路由

> 关于 1 day也是类似的思路，通过网上爆出的路由或者payload（或者通过看comment）来在代码中整体搜索定位漏洞点，然后逆向追踪

## 前台文件上传

**这里正向追踪：先黑盒寻找上传文件的接口/功能点 --->burp抓包找到接口地址--->然后去代码中寻找(全局搜索/controller层找)**



登录学员账号，寻找上传点，发现头像上传功能，测试：

![image-20240706232650331](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706232650331.png)



看到请求地址为：

![image-20240706232753495](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706232753495.png)



去代码中搜索`image/jok4`，没搜到：

![image-20240706232831158](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706232831158.png)



没搜到原因有几种：

- 搜索时候注意：比如`image/gok4`这种**两个路径可能是拼接起来的**(比如controller的类有一个路径+方法上也有路径)，所以可以分开去搜`image`，`gok4`

- 这个项目中，这个路径是引入的自己打包的**第三方jar包**中代码中的，IDEA的搜索是无法搜索jar包里面关键字的（实际审计中，可以使用工具进行扫描有无关键字）。实际上这个项目是将处理文件上传的代码封装到了一个 Jar 包然后引用的，**一般情况下如果碰到搜不到的情况，就去 pom.xml 里看看有没有声明依赖的 Jar 包**

> note:找到代码中接口后（如果看不懂是干啥的，可以结合数据包判断是干啥的功能）



我们定位到了代码位置：

![image-20240706233241597](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706233241597.png)



然后继续正向往下分析代码，依次看controller层-->service层-->dao层--->mapper文件



可以定位到`gok4`方法：

```java
该方法的逻辑如下：
1、检查上传文件的大小，如果超过4M，则返回错误信息。   //--->uploadfile参数为前端传递的文件
2、根据fileType参数判断文件后缀(使用getSuffix方法)是否符合要求，如果文件类型不符合或者后缀为jsp，则返回错误信息。 //--->fileType参数为前端传递
3、根据上传文件的后缀和param参数，确定上传文件的保存路径。  //--->param参数为前端传递
4、创建保存文件的目录（如果目录不存在）。
5、将上传文件保存到指定的路径(通过transferTo())。
6、返回上传成功的响应信息，包括文件路径和状态码。
7、如果发生异常，记录错误日志，并返回上传失败的错误信息。
```

![image-20240706233426295](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240706233426295.png)

跟进`getSuffix`中，发现其代码中除了获取文件后缀其他没什么过滤：

```java
    public static String getSuffix(String str) {
        return str.substring(str.lastIndexOf(".") + 1);
    }
}
```



再结合之前抓包的url：`http://127.0.0.1:8080/image/gok4?&param=temp&fileType=jpg,gif,png,jpeg`

将已知的信息串联起来，总结这段代码的判断逻辑为：文件后缀包含 jpg,gif,png,jpeg，且不为 jsp 即进行下一步上传操作。



可以看到param和fileType为代码中我们可控的



我们的思路：

1. 上传一个不是jsp的且可以解析的，比如jspx，即`http://127.0.0.1:8080/image/gok4?&param=temp&fileType=jpg,gif,png,jpeg,jspx`
2. 利用windows的特性，比如`::$DATA`，即`http://127.0.0.1:8080/image/gok4?&param=temp&fileType=jpg,gif,png,jpeg,jsp$$DATA`



我们使用冰蝎自带的jsp测试：

![image-20240707095232111](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707095232111.png)

成功上传：

![image-20240707095551892](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707095551892.png)



## 后台文件上传



逆向追踪：先通过"在文件中查找"来搜索文件上传功能的关键字（搜索时候后缀限制为`.java`），然后反向回溯功能点



文件上传关键字：

```
File
java.io.File
FileUpload
FileUploadBase
FileItemIteratorImpl
FileItemStreamImpl
FileUtils
UploadHandleServlet
FileLoadServlet
FileOutputStream
DiskFileItemFactory
MultipartRequestEntity
MultipartFile
com.oreilly.servlet.MultipartRequest
MultipartHttpServletRequest
org.apache.commons.fileupload
MultipartFile
CommonsMultipartResolver
......
```



我们搜索`FileUpload`，

![image-20240707095837379](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707095837379.png)

位置com/inxedu/os/common/controller/VideoUploadController.java

看起来是一个视频上传的功能：

![image-20240707100211247](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707100211247.png)

可以直接上传jsp



我们回溯功能点在哪里（好了，它在controller里面，不用回溯了）

可知该上传接口为：`/video/uploadvideo`

![image-20240707100345336](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707100345336.png)









构造：

方法1： 找到该接口在系统中对应的位置

方法2：自己构造文件上传的表单(接口改为该接口即可)**(简便)**



构造表单：

```html
<form action="http://192.168.3.214:8080/video/uploadvideo" enctype="multipart/form-data" id="frmUpload" method="post">
<input name="uploadfile" type="file">
<input type ="text" name = "fileType" value="">
<input id="btnUpload" type="submit" value="上传">
</form>
```

![image-20240707100632667](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707100632667.png)

抓包修改

![image-20240707103427050](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707103427050.png)



我这里测试一直失败报服务器错误，可能是环境搭建的问题，先不管了









note：解决IDEA下debug断点进不去：

https://blog.csdn.net/xujie102360/article/details/81476774

https://blog.csdn.net/searlas/article/details/80826777

![image-20240707110830424](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707110830424.png)



## SQL注入

已知系统使用的是mybatis，mybatis最常见的注入就是错误使用`${}`导致，

所以我们直接搜索`${`，看看哪些是通过`${}`取值并且没有过滤的



![image-20240707104233314](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104233314.png)



找到了一处，我们**逆向追踪调用逻辑**，寻找是否含有过滤并且参数我们是否可控。



daoImpl:

![image-20240707104350963](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104350963.png)

dao:

![image-20240707104404853](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104404853.png)



ctrl+单击，来到ServiceImpl:

![image-20240707104447938](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104447938.png)

参数为`ids`，由形参`articleIds`而来

也没有进行过滤



service：![image-20240707104550997](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104550997.png)



往上走去找controller：传递的参数为`artidArr`，也确实没有过滤

![image-20240707104617107](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104617107.png)



`deleteArticle`方法在这里被调用：参数为前端传来的`articelId`，我们可控：

![image-20240707104735651](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104735651.png)





![image-20240707104813816](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707104813816.png)

可知接口为：`/admin/article/delete`



打开系统登录后台。找到文章管理部分，选择删除，抓包

![image-20240707105658610](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707105658610.png)

直接将数据包放到sqlmap中跑：

![image-20240707105645898](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707105645898.png)







可以看到这套系统中还有多处存在SQL注入漏洞：

![image-20240707105811068](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707105811068.png)







## XSS

待完成































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/inxedu%E7%B3%BB%E7%BB%9F%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/  

