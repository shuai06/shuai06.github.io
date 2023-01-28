# 使用IDEA对Jar包进行动态调试


> 初学JAVA啥也不会，从头开始学



1.新建IDEA项目

2.右键新建lib文件夹，并把jar包放进来(会自动反编译)

3.点击右上角"Add Configurations"

![img](https://cdn.nlark.com/yuque/0/2022/png/1224444/1641108800249-038a2893-0677-4f97-a278-ef62d77d931f.png)

4.新建Remote，应用

![img](https://cdn.nlark.com/yuque/0/2022/png/1224444/1641108931220-5c61ef8e-e254-4ab6-9078-da2104e8f807.png)

5.来到原来jar包，别在项目里面反编译那个jar包目录下

命令行下执行如下命令

```bash
# 我这里以Behinder.jar为例
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 Behinder.jar
```

6.然后单击IDEA的`debug`按钮

运行程序/如果是weblogic之类的就浏览器访问地址等方式

就可以看到程序在断点处暂停，然后就可以进行逐步调试了

![img](https://cdn.nlark.com/yuque/0/2022/png/1224444/1641109139689-95cc6dd7-7e36-4cdf-ab90-f6ea502b090b.png)


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/%E4%BD%BF%E7%94%A8idea%E5%AF%B9jar%E5%8C%85%E8%BF%9B%E8%A1%8C%E5%8A%A8%E6%80%81%E8%B0%83%E8%AF%95/  

