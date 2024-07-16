# StudentManager代码审计


<!--more-->



## 简介

继续入门代码审计的小项目，今天是StudentManager

项目地址为：https://github.com/Hui4401/StudentManager





**IDEA导入servlet包：**
1.点击Project Structure
2.点击Libraries
3.点击上方的+号，在弹出窗选中java，找到tomact所在的目录，在lib中找到`servlet-api.jar`添加进去即可



**部署中可能遇到404的问题**：https://blog.51cto.com/u_14604401/5692898





另外一个注意点：修改数据库连接的时候注意多个servlet中都要修改....弄了半天没懂当时



![image-20240715155619554](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240715155619554.png) 



## Cookie绕过登录验证



在`index.jsp`处：

遍历所有cookie字段，判断是否cookie中存在name的键

​	如果cookie中存在name键，就把name对应的值作为`findWithId(`)方法的参数调用该方法
​		如果返回的结果是老师就跳转到`one_page_student`

​		如果是学生就进入`student/main.jsp`



跟进`findWithId()`:

![image-20240715160347347](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240715160347347.png)

这里仅仅是把name的值，在数据库中查询，如果查到结果就返回

题外话：这里sql语句直接进行拼接，肯定存在注入





**复现：**

伪造一下存在的教师账户(cookie中添加存在用户111)，不用登录就可以直接跳转到后台

```http
GET /index.jsp HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: Phpstorm-9e28e970=dbc7cd43-3413-4de5-aa40-3254070809b3; JSESSIONID=0F189B92035E90638972A9FB86734AC0; name=111
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
```



绕过认证直接进入后台

![image-20240716094131087](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094131087.png)



















## SQL 注入

### 1

之前前看到`findWithId()`中存在SQL语句拼接，所以存在sql注入可以利用

![image-20240716094326802](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094326802.png)





### 2

`check_login`中将获取到的用户信息传给`checkAccount`进行验证：

![image-20240716094442999](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094442999.png)



可以看到也是拼接了id和password：

![image-20240716094541902](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094541902.png)



存在注入，这里可以万能密码登录了：

![image-20240716094618706](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094618706.png)







### 3



又是一个`findWithid`

![image-20240716095743696](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095743696.png)

![image-20240716095754075](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095754075.png)



```
uid=zzu'/**/and/**/extractvalue(1,concat(0x7e,(select/**/@@version),0x7e))#&name=zzu&sex=%E7%94%B7&email=123%40test.com&password=123456

```







### 4

在`scoreD.java`里面的`findWithId`:

![image-20240716095027193](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095027193.png)

追踪发现在这里进行了调用：

![image-20240716095212707](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095212707.png)





我这里连接数据库有点问题，所以借图：https://xz.aliyun.com/t/12105?![image-20240716095509464](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095509464.png)





其他注入点：

![image-20240716094944959](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716094944959.png)

注意：cookie注入还有一点坑点，空格和,号不能用



![image-20240716095620236](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716095620236.png)

全局搜索关键字就行





## 任意用户注册

![image-20240716100100816](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716100100816.png)





通过session获取验证码的值，如果验证码正确则注册成功：





验证码`randStr`对应的值是在`code.jsp`中设置的，且并没有去设置失效的时间，可以重复利用

![image-20240716100248839](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716100248839.png)



![image-20240716100543435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716100543435.png)

注册一个用户aaa
抓包然后把用户名aaa改成bbb，别的不动

全部注册成功















## 越权修改头像+任意文件上传

![image-20240716100918237](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716100918237.png)



这个上传头像是个上传图片文件的功能，这里的mac也没演示，可以试试能不能截断：

![image-20240716100942575](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716100942575.png)

通过获取的id上传到文件夹id命名的文件



展示头像时候是直接调用id查看这个id名的文件：

![image-20240716101145652](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716101145652.png)





所以上传头像的时候抓包替换id为别人的：

![image-20240716101211205](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716101211205.png)









## 目录穿越

刚才上传头像代码处，尝试把id修改为`../`来实现目录穿越的文件上传

![image-20240716101507017](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716101507017.png)

![image-20240716101742509](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716101742509.png)

可以看到穿越成功：

![image-20240716101759808](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240716101759808.png)



























---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/studentmanager%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/  

