# Ueditor的SSRF漏洞分析


<!--more-->



## 前言

ueditor编辑器的v1.4.3版本（jsp)存在SSRF漏洞



源码地址：https://github.com/fex-team/ueditor





下面是刚看博客跟着大师傅学的经验：https://fuping.site/2018/05/25/UEditor-SSRF-In-JSP/

**在比如看到在版本`1.4.3.1`修复了SSRF漏洞**:

![image-20240717084814520](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717084814520.png)

那么就是`v1.4.3`存在这个SSRF漏洞，我们独立做漏洞分析的时候可以对比版本1.4.3与1.4.3.1有什么不同，从而找到存在问题的地方

去github查看某个版本的时候，可以通过如下**看tag**的方式：https://github.com/fex-team/ueditor/tree/v1.4.3.1/jsp



![image-20240717084534548](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717084534548.png)



查看版本1.4.3.1下的jsp代码：

![image-20240717085113458](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717085113458.png)

可以发现在**该版本有一次commit**，commitId 为`a1820147cfc3fbe2960a7d99f8dfbe338c02f0b6`。根据内容字面意思应该是增加了修复SSRF的代码。

然后就**对比一下两个版本**中jsp部分的代码有何不同



**github中对比两个版本代码不同的方法：**

- 方法1：https://docs.github.com/zh/pull-requests/committing-changes-to-your-project/viewing-and-comparing-commits/comparing-commits



比较两次commit：比如我们比较这两次提交，下面url中前面是用户名和仓库名，后面`..`两边分别是两次的commit id

https://github.com/fex-team/ueditor/compare/0853f9fe820a236dc5ab74f1ead2430043b00fb5..a1820147cfc3fbe2960a7d99f8dfbe338c02f0b6

我们这里只比较jsp中的变换，可以看到如下，`jsp/src/com/baidu/ueditor/hunter/ImageHunter.java`中的`validHost`方法进行了一些改动：

![image-20240717091329493](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717091329493.png)





比较变化：

https://github.com/fex-team/ueditor/compare/master...octocat:master

![image-20240717091047010](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717091047010.png)

......其他更方便的方式自己探索



- 方法2：拉取下来之后：https://blog.csdn.net/qq_16517483/article/details/125566057









## 漏洞分析

前面知道了`jsp/src/com/baidu/ueditor/hunter/ImageHunter.java`的`validHost`方法就行了修改：

![image-20240717091329493](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717091329493.png)



可以看到`v1.4.3.1`新增了主机名是否为本地内部地址的判断：

```java
	private boolean validHost ( String hostname ) {
		try {
			InetAddress ip = InetAddress.getByName(hostname);   //获取ip地址根据主机名

			if (ip.isSiteLocalAddress()) {		//判断是否为内部内网地址(a, b, c 三类)
				return false;
			}
		} catch (UnknownHostException e) {
			return false;
		}

		return !filters.contains( hostname );

	}
```



而`v1.4.3`中仅仅做了是否不为过滤的ip地址的限制









下载`v1.4.3`的jsp的源码：

![image-20240717092115203](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717092115203.png)





ctrl+单击发现是在`com/baidu/ueditor/hunter/ImageHunter.java`中的`captureRemoteData`方法中调用了`validHost`方法：

![image-20240717092546657](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717092546657.png)

该方法首先会调用`validHost`对传入的url进行验证，如果不合法就抛出异常；如果合法就继续往下走，通过`validContentState`验证返回状态码是否满足；后续进行一系列文件大小和文件类型等其他条件判断，都符合后才进行后续图片报错，否则抛出异常。



继续追踪，在`capture`方法中调用了`captureRemoteData`。

![image-20240717093054526](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717093054526.png)



`capture`又在`com/baidu/ueditor/ActionEnter.java中的invoke`方法中被调用

![image-20240717093653842](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717093653842.png)

![image-20240717093802373](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717093802373.png)

即当传递的`action`参数对应为`catchimage`时，才可能触发SSRF漏洞。





**漏洞验证：**

搭建环境配置







入口在`jsp/controller.jsp`

![image-20240717152436221](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717152436221.png)



![image-20240717163006413](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20240717163006413.png)

构造漏洞路径：

`http://localhost:8088/jsp/controller.jsp?action=catchimage&source[]=http://192.168.135.133:8080/test.jpg`

可根据页面返回的结果不同判断该地址端口是否开放：

1. 如果抓取不存在的图片地址时，页面返回{"state": "SUCCESS", list: [{"state":"\u8fdc\u7a0b\u8fde\u63a5\u51fa\u9519"} ]}，即state为“远程连接出错”。
2. 如果成功抓取到图片，页面返回{"state": "SUCCESS", list: [{"state": "SUCCESS","size":"5103","source":"http://192.168.135.133:8080/tomcat.png","title":"1527173588127099881.png","url":"/ueditor/jsp/upload/image/20180524/1527173588127099881.png"} ]}，即state为“SUCCESS”。
3. 如果主机无法访问，页面返回{"state":"SUCCESS", list: [{"state": "\u6293\u53d6\u8fdc\u7a0b\u56fe\u7247\u5931\u8d25"}]}，即state为“抓取远程图片失败”。

由于除了在config.js中的catcherLocalDomain配置了过滤的地址外，没有针对内部地址进行过滤，所以可以根据抓取远程图片返回结果的不同，来进行内网的探测。



具体复现过程可参考：https://fuping.site/2018/05/25/UEditor-SSRF-In-JSP/



















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/ueditor%E7%9A%84ssrf%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/  

