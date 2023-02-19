# IDEA控制台乱码(Tomcat日志乱码)解决办法




1.打开 `File->Setting->Editor->File Encodings`，将`Global Encoding`、`Project Encoding`、`Default encodeing for properties files`这三项都设置成UTF-8，然后点击OK。如下图：

![image-20220112153430975](https://image.geoer.cn/img/image-20220112153430975.png)





2.打开 `Help->Edit Custom VM Options`

加上下面一行：

```
-Dfile.encoding=UTF-8
```

![image-20220112153623173](https://image.geoer.cn/img/image-20220112153623173.png)



3.打开 `Edit Configrations`

将`vm option`为` -Dfile.encoding=utf-8`，点击OK

![image-20220112153728537](https://image.geoer.cn/img/image-20220112153728537.png)



4.重启IDEA


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/idea%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%B9%B1%E7%A0%81-tomcat%E6%97%A5%E5%BF%97%E4%B9%B1%E7%A0%81-%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95/  

