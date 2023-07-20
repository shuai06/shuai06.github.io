# 基于SpringBoot和Vue3的前后端分离博客


<!--more-->



跟着B站上一位UP主过了一遍这个简单的前后端分离的博客项目，其中还有很多细节没太懂，但是记录一下算是学习过的痕迹~

视频地址：https://www.bilibili.com/video/BV1PQ4y1P7hZ









# 后端



## 一、新建项目，整合MP

### 0.新建spring-boot项目

暂时添加下面四个依赖：

![image-20230718123737906](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com//image-20230718123737906.png)



### 1.安装mysql（docker）

https://juejin.cn/post/7043826861272465445

```bash
docker pull mysql/mysql-server

docker run -p 3306:3306 --name mysql --privileged=true -e MYSQL_ROOT_PASSWORD=123456 -e character-set-server=utf8 -e collation-server=utf8_general_ci -d mysql/mysql-server

# 进入容器
docker exec -it 7b7997b002a8 /bin/sh
# 登录
mysql -u root -p

# 创建数据库和表：执行第二步，然后再执行下面的代码

# 查看权限
select host, user, authentication_string, plugin from mysql.user; 
# 设置root权限和访问
update mysql.user set host='%' where user ='root';
grant all privileges on *.* to 'root'@'%' with grant option;
flush privileges;

SHOW GRANTS FOR 'root'@'%';
```

IDEA测试连接



### 2.创建数据库和表

```
create database vueblog;

use vueblog;

DROP TABLE IF EXISTS `m_blog`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `m_blog` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL,
  `title` varchar(255) NOT NULL,
  `description` varchar(255) NOT NULL,
  `content` longtext,
  `created` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP,
  `status` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */; 


DROP TABLE IF EXISTS `m_user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `m_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL,
  `avatar` varchar(255) DEFAULT NULL,
  `email` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL,
  `status` int(5) NOT NULL,
  `created` datetime DEFAULT NULL,
  `last_login` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `UK_USERNAME` (`username`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */; 


INSERT INTO `vueblog`.`m_user` (`id`, `username`, `avatar`, `email`, `password`, `status`, `created`, `last_login`) VALUES ('1', 'markerhub', 'https://image-1300566513.cos.ap-guangzhou.myqcloud.com/upload/images/5a9f48118166308daba8b6da7e466aab.jpg', NULL, '96e79218965eb72c92a549dd5a330112', '0', '2020-04-20 10:44:01', NULL);


```







### 3.导入mybatis plus相关jar包

Pom.xml

```xml
<!-- 模板引擎-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>LATEST</version>
</dependency>
<!--mybatis-plus代码生成器(版本和mp要一致，不然会报错)-->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-generator</artifactId>
  <version>3.5.3.1</version>
</dependency>

```

引入新依赖之后需要**刷新** maven 依赖，

![image-20230718123904053](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718123904053.png)



### 4.修改application配置文件

注意分清文件格式，这里用的是`application.yaml`

```yaml
# DataSource Config
spring:
    datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/vueblog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai
        username: root
        password: 123456

mybatis-plus:
    mapper-locations: classpath*:/mapper/**Mapper.xml

server:
    port: 8081
```





### 5.**开启mapper接口扫描，添加分页插件**

配置分页MybatisPlusConfig

>  mybatis-plus3.5分页插件使用(PaginationInterceptor):https://blog.csdn.net/moshowgame/article/details/123058673

![image-20230718110036073](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718110036073.png)

新版的写法:

```java
package com.example.vueblog1.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import java.util.Collections;

@Configuration
@EnableTransactionManagement
@MapperScan("com.markerhub.mapper")
public class MybatisPlusConfig {
    @Bean
    public  PaginationInnerInterceptor paginationInnerInterceptor() {
        PaginationInnerInterceptor paginationInterceptor = new PaginationInnerInterceptor();
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        paginationInterceptor.setMaxLimit(-1L);
        paginationInterceptor.setDbType(DbType.MYSQL);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setOptimizeJoin(true);
        return paginationInterceptor;
    }

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.setInterceptors(Collections.singletonList(paginationInnerInterceptor()));
        return mybatisPlusInterceptor;
    }

}

```



旧版的写法：

```java
package com.example.vueblog.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
@MapperScan("com.example.vueblog.mapper")
public class MybatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        return paginationInterceptor;
    }
}


```











### 6.生成代码

> 官方给我们提供了一个代码生成器，然后我写上自己的参数之后，就可以直接根据数据库表信息生成entity、service、serviceImpl、mapper等接口和实现类。

创建CodeGenerator(在com.vueblog包下面)

![image-20230718124107613](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718124107613.png)

需要修改文件里面的`mysql连接的配置信息`和保存的`包配置`

```java
package com.example.vueblog;


import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }


    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        // gc.setOutputDir("D:\\test");
        gc.setAuthor("anonymous");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);

        // 数据源配置 数据库名 账号密码
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/vueblog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(null);
        pc.setParent("com.example.vueblog");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！
                return projectPath + "/src/main/resources/mapper/"
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });

        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix("m_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}

```

运行这个java文件，生成文件的效果如下：

![image-20230718112717050](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718112717050.png)



>  如果报错，可以考虑一下是不是pom中自动生成的包版本太高，可以考虑降级，生成完毕之后再升级回去即可





### 7.测试效果

![image-20230718112858961](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718112858961.png)



运行项目：

![image-20230720185919376](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720185919376.png)





## 二、统一封装返回结果

这里我们用到了一个`Result`的类，这个用于我们的异步统一返回的结果封装，方便前后端对接。一般来说，结果里面有几个要素必要的

- 是否成功，可用code表示（如200表示成功，400表示异常）
- 结果消息
- 结果数据



### **Result类**

路径com.vueblog.common.lang;

新建类：

![image-20230718124737680](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718124737680.png)



```java
package com.example.vueblog.common.lang;
import lombok.Data;
import java.io.Serializable;

@Data
public class Result implements Serializable {
    private int code; //200是正常 400表示异常
    private String msg;
    private Object data;//返回数据

    //定义静态的方法，可以直接调用
    //成功：这个是成功只返回数据的情况
    // 这个也可以用多个构造函数来实现，这个思想用的较多
    public static Result succ( Object data){

        return succ(200,"操作成功",data);
    }
    //成功
    public static Result succ(int code,String msg,Object data){
        Result r = new Result();
        r.setCode(code);
        r.setMsg(msg);
        r.setData(data);
        return r;
    }

    //失败
    public static Result fail(String msg){
        return fail(400,msg,null);
    }
    //失败
    public static Result fail(String msg,Object data){
        return fail(400,msg,data);
    }
    //失败
    public static Result fail(int code,String msg,Object data){
        Result r = new Result();
        r.setCode(code);
        r.setMsg(msg);
        r.setData(data);
        return r;
    }




}

```





### **在UserController 中引用测试**



```
package com.example.vueblog.controller;


import com.example.vueblog.common.lang.Result;
import com.example.vueblog.entity.User;
import com.example.vueblog.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author anonymous
 * @since 2023-07-18
 */
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/index")
    public  Object index(){
        User user = userService.getById(1);
        return Result.succ(200,"操作成功", user);
//        return userService.getById(1);
    }


}

```



![image-20230720185959805](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720185959805.png)





## 三、整合Shiro+jwt，实现会话共享

### Shiro整合jwt逻辑

![image-20230718134929202](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718134929202.png)



用户登录使用jwt生成token，当登录的用户去访问数据时用shiro验证用户是否具有权限。

没有登录的话没有jwt，访问数据时也是shiro判断。



考虑到后面可能需要做集群、负载均衡等，所以就需要会话共享，而shiro的缓存和会话信息，我们一般考虑使用redis来存储这些数据，所以，我们不仅仅需要整合shiro，同时也需要整合redis。在开源的项目中，我们找到了一个starter可以快速整合shiro-redis，配置简单，这里也推荐大家使用。
而因为我们需要做的是前后端分离项目的骨架，所以一般我们会采用token或者jwt作为跨域身份验证解决方案。所以整合shiro的过程中，我们需要引入jwt的身份验证过程。





### 整合

**redis安装：**

我也是通过docker安装

```bash
docker pull redis

docker images

# 启动
docker run -d --name redis -p 6379:6379 redis:latest redis-server --appendonly yes --requirepass "自己设置的密码"
# 说明：redis-server --appendonly yes：在容器执行redis-server启动命令，并打开redis持久化配置


docker ps


# 进入容器
docker exec -ti redis redis-cli

```







我们使用一个shiro-redis-spring-boot-starter的jar包，具体教程可以看官方文档：https://github.com/alexxiyang/shiro-redis/blob/master/docs/README.md#spring-boot-starter

**第一步：导入shiro-redis的starter包：还有jwt的工具包，以及为了简化开发，引入hutool工具包。**

pom.xml中导入:

```xml
<!-- shiro-redis -->
<dependency>
  <groupId>org.crazycake</groupId>
  <artifactId>shiro-redis-spring-boot-starter</artifactId>
  <version>3.3.1</version>
</dependency> 
<!-- hutool工具类 -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.16</version>
</dependency>

<!-- jwt 生成工具 校验工具-->
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.9.1</version>
</dependency>



```

配置文件添加：

![image-20230718143408434](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718143408434.png)



**第二步：自定义ShiroConfig**

![image-20230718151558617](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718151558617.png)

在`com.example.vueblog.config`下新建java类：

![image-20230718144952202](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718144952202.png)

按照官方文档把这些Bean复制下来：

![image-20230718141509836](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718141509836.png)







```java
package com.example.vueblog.config;

import com.example.vueblog.shiro.AccountRealm;
import com.example.vueblog.shiro.JwtFilter;
import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
import org.apache.shiro.mgt.DefaultSubjectDAO;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.mgt.SessionManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.spring.web.config.DefaultShiroFilterChainDefinition;
import org.apache.shiro.spring.web.config.ShiroFilterChainDefinition;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
import org.crazycake.shiro.RedisCacheManager;
import org.crazycake.shiro.RedisSessionDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import javax.servlet.Filter;


/**
 * shiro启用注解拦截控制器
 */
@Configuration
public class ShiroConfig {
    @Autowired
    private JwtFilter jwtFilter;

    @Bean
    public SessionManager sessionManager(RedisSessionDAO redisSessionDAO) {
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        sessionManager.setSessionDAO(redisSessionDAO);
        return sessionManager;
    }

    @Bean
    public DefaultWebSecurityManager securityManager(AccountRealm accountRealm,
                                                     SessionManager sessionManager,
                                                     RedisCacheManager redisCacheManager) {
//        AccountRealm是我们自己定义的
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(accountRealm);
        securityManager.setSessionManager(sessionManager);
        securityManager.setCacheManager(redisCacheManager);

        /*
         * 关闭shiro自带的session，详情见文档
         */
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);
        return securityManager;
    }

    //shiro过滤器链的定义：哪些链接需要经过哪些过滤器
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        Map<String, String> filterMap = new LinkedHashMap<>();

        filterMap.put("/**", "jwt"); // 主要通过注解方式校验权限。所有链接都要经过jwt来校验
        chainDefinition.addPathDefinitions(filterMap);
        return chainDefinition;
    }

    //这些方法用的时候不用自己写，复制根据自己的项目改改就行
    @Bean("shiroFilterFactoryBean")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager,
                                                         ShiroFilterChainDefinition shiroFilterChainDefinition) {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);

        //然后使用jwt的filter，所有的链接都会经过我们定义的filter
        Map<String, Filter> filters = new HashMap<>();
        filters.put("jwt", jwtFilter);  //这里的jwt路径跟上一个方法是对应的
        shiroFilter.setFilters(filters);

        Map<String, String> filterMap = shiroFilterChainDefinition.getFilterChainMap();

        shiroFilter.setFilterChainDefinitionMap(filterMap);

        return shiroFilter;
    }



}




```



**note：**SpringBoot3.0变更

- Spring Boot 3.0 使用 Java 17作为最低版本，如果版本低于17，那么首先要升级你的JDK

- 另外一个很重要的变化就是本次升级之后，最低只支持 Jakarta EE 9，使用 Servlet5.0 和 JPA3.0 规范，不过最新版本RC2已经升级到了 JakartaEE 10，默认使用 Servlet6.0 和 JPA3.1 规范。
  有些同学可能连 Jakarta 是什么都不知道，这个英文单词是印尼首都雅加达的意思，其实就是我们知道的 JavaEE 改名之后就叫 JakartaEE，比如我们之前的 javax.servlet 包现在就叫 jakarta.servlet 

```java
import javax.servlet.http.HttpServletRequest;
// 改为
import jakarta.servlet.http.HttpServletRequest;
```



但是继承` extends AuthenticatingFilter`需要重写方法的参数是`javax.servlet.http.HttpServletRequest`。。。很尴尬

**解决方法：**https://stackoverflow.com/questions/75838823/apache-shiro-encountering-java-lang-noclassdeffounderror-javax-servlet-filter-w

**解决方法（有效）：**换成SpringBoot2

https://blog.csdn.net/javamaxiaojing/article/details/122052030

![image-20230718223901030](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718223901030.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.7</version>
<!--        <version>3.1.1</version>-->
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>vueblog1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>vueblog</name>
    <description>vueblog</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

<!--        模板引擎-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>LATEST</version>
        </dependency>
        <!--mybatis-plus代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
<!--            <version>3.5.3.1</version>-->
            <version>3.2.0</version>
        </dependency>

        <!-- shiro-redis -->
        <dependency>
            <groupId>org.crazycake</groupId>
            <artifactId>shiro-redis-spring-boot-starter</artifactId>
<!--            <version>3.3.1</version>-->
            <version>3.2.1</version>
<!--            <classifier>jakarta</classifier>-->
        </dependency>
        <!-- hutool工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
<!--            <version>5.8.16</version>-->
            <version>5.3.3</version>
        </dependency>

        <!-- jwt 生成工具 校验工具-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>






    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```





**第四步：创建AccountRealm**

![image-20230718151645659](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718151645659.png)

放到`com.example.vueblog.shiro`路径下

![image-20230718145935719](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718145935719.png)

```java
package com.example.vueblog.shiro;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.stereotype.Component;

@Component
public class AccountRealm extends AuthorizingRealm {
    //继承AuthorizingRealm，重写方法
    //为了让realm支持jwt的凭证校验token
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //会到这里处理，强转成JwtToken
        JwtToken jwtToken = (JwtToken) token;
        return null;
    }
}

```







**第五步：创建JwtFilter**

![image-20230718140934456](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718140934456.png)





```java
package com.example.vueblog.shiro;

import cn.hutool.http.server.HttpServerRequest;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.example.vueblog.common.lang.Result;
import com.example.vueblog.util.JwtUtils;
import io.jsonwebtoken.Claims;


import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.ExpiredCredentialsException;
import org.apache.shiro.web.filter.authc.AuthenticatingFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;


@Component
public class JwtFilter extends AuthenticatingFilter {
    //继承shiro内置的过滤器AuthenticatingFilter。重写两个方法
    @Autowired
    JwtUtils jwtUtils;
    //验证token
    @Override
    protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        //从header中获取jwt信息，封装成成自定义的token
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        //获取头部token
        String jwt = request.getHeader("Authorization");
        if(StringUtils.isEmpty(jwt)){
            return null;
        }
        return new JwtToken(jwt);
        //创建token后，会交给shiro进行登录subject.log(),会委托到我们的Realm处理
    }

    // createToken之后会来到onAccessDenied拦截
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest)servletRequest;
        //获取头部token
        String jwt = request.getHeader("Authorization");
        if(StringUtils.isEmpty(jwt)){
            return true;
        }else{
            // 先校验jwt
            Claims claims = jwtUtils.getClaimByToken(jwt);
            //校验是否为空和时间是否过期
            if(claims == null || jwtUtils.isTokenExpired(claims.getExpiration())){
                throw new ExpiredCredentialsException("token已失效,请重新登录");
            }
            //执行登录处理
            return executeLogin(servletRequest,servletResponse);
        }
    }
    //捕捉错误重写方法返回Result
    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request, ServletResponse response) {
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        Throwable throwable = e.getCause() == null ? e : e.getCause();

        Result result = Result.fail(throwable.getMessage());
        //返回json
        String json = JSONUtil.toJsonStr(result);
        try {
            //打印json
            httpServletResponse.getWriter().print(json);
        }catch (IOException ioException){

        }

        return false;
    }

}


```







**第五步：创建JwtToken**

所以我们需要自定义token，需要继承shiro的AuthenticationToken

```java
package com.vueblog.shiro;

import org.apache.shiro.authc.AuthenticationToken;

public class JwtToken implements AuthenticationToken {
    private String token;

    public JwtToken(String jwt){
        this.token = jwt;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}


```











**第六步：创建JwtUtils**

JwtUtils是个生成和校验jwt的工具类，其中有些jwt相关的密钥信息是从项目配置文件中配置的

需要配置application.yaml：

![image-20230718142244848](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718142244848.png)

```yaml
shiro-redis:
  enabled: true
  redis-manager:
    host: 127.0.0.1:6379
    
markerhub:
  jwt:
    # 加密秘钥
    secret: f4e2e52034348f86b67cde581c0f9eb5
    # token有效时长，7天，单位秒
    expire: 604800
    header: token
```



下面的prefix跟上面的配置文件一致：

```java
package com.vueblog.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * jwt工具类
 */
@Slf4j
@Data
@Component
@ConfigurationProperties(prefix = "markerhub.jwt")
public class JwtUtils {

    private String secret;
    private long expire;
    private String header;

    /**
     * 生成jwt token
     */
    public String generateToken(long userId) {
        Date nowDate = new Date();
        //过期时间
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);

        return Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setSubject(userId+"")
                .setIssuedAt(nowDate)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }
		
  	//进行校验
    public Claims getClaimByToken(String token) {
        try {
            return Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            log.debug("validate is token error ", e);
            return null;
        }
    }

    /**
     * token是否过期
     * @return  true：过期
     */
    public boolean isTokenExpired(Date expiration) {
        return expiration.before(new Date());
    }
}

```





**第七步：创建spring-devtools.properties**

resources/WETA-INF目录下创建`spring-devtools.properties`内容为：

```
restart.include.shiro-redis=/shiro-[\\w-\\.]+jar
```





## 四、shiro逻辑开发

> **小提示:登录调用AccountRealm类下面的doGetAuthenticationInfo**



**创建类AccountProfile 用于传递用户可以公开的数据**

![image-20230718233431598](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718233431598.png)

```java
package com.example.vueblog.shiro;

import lombok.Data;

import java.io.Serializable;

@Data
public class AccountProfile implements Serializable {
    private Long id;

    private String username;

    private String avatar;

    private String email;

}


```









**完善AccountRealm**

![image-20230718233713220](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230718233713220.png)

```java
package com.example.vueblog.shiro;

import cn.hutool.core.bean.BeanUtil;
import com.example.vueblog.entity.User;
import com.example.vueblog.service.UserService;
import com.example.vueblog.util.JwtUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class AccountRealm extends AuthorizingRealm {
    @Autowired
    JwtUtils jwtUtils;

    @Autowired
    UserService userService;

    //继承AuthorizingRealm，重写方法
    //为了让realm支持jwt的凭证校验token
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //会到这里处理，强转成JwtToken
        JwtToken jwtToken = (JwtToken) token;

        String userId = jwtUtils.getClaimByToken((String) jwtToken.getCredentials()).getSubject();
        User user = userService.getById(Long.valueOf(userId));
        if(user == null){throw new UnknownAccountException("账户不存在");}
        if(user.getStatus() == -1){
            throw new LockedAccountException("账户被锁定");
        }

        AccountProfile profile = new AccountProfile();
        BeanUtil.copyProperties(user, profile);  //传递数据

        return new SimpleAuthenticationInfo(profile, jwtToken.getCredentials(), getName());
    }
}

```





## 五、异常处理

![image-20230719110455325](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719110455325.png)



**创建GlobalExceptionHandler 类**

![image-20230719111131409](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719111131409.png)

```java
package com.example.vueblog.exception;

import com.example.vueblog.common.lang.Result;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.ShiroException;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 日志输出
 * 全局捕获异常
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ResponseStatus(HttpStatus.UNAUTHORIZED)    //因为前后端分离 返回一个状态 一般是401 没有权限
    @ExceptionHandler(value = ShiroException.class) //捕获运行时异常ShiroException是大部分异常的父类
    public Result hander(ShiroException e){
        log.error("运行时异常-----{}",e);
        return Result.fail(401,e.getMessage(), null);
    }
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = RuntimeException.class)
    public Result hander(RuntimeException e){
        log.error("运行时异常-----{}",e);
        return Result.fail(e.getMessage());
    }




}

```



**然而我们运行测试发现并没有拦截,因为我们没有进行登录拦截**

![image-20230720190041973](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190041973.png)





**@RequiresAuthentication//登录拦截注解**

![image-20230719111806870](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719111806870.png)





运行效果：

![image-20230720190109332](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190109332.png)





## 六、实体校验

当我们表单数据提交的时候，前端的校验我们可以使用一些类似于jQuery Validate等js插件实现，而后端我们可以使用Hibernate validatior来做校验。
我们使用springboot框架作为基础，那么就已经自动集成了Hibernate validatior。(校验登录非空等等)

**0.首先添加依赖**

在pom.xml中

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>

```





**1.首先在实体的属性上添加对应的校验规则**

**User实体类中**

```java
package com.example.vueblog.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import java.time.LocalDateTime;
import java.io.Serializable;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

/**
 * <p>
 * 
 * </p>
 *
 * @author anonymous
 * @since 2023-07-18
 */
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("m_user")
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    @NotBlank(message = "昵称不能为空")
    private String username;

    private String avatar;

    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;

    private String password;

    private Integer status;

    private LocalDateTime created;

    private LocalDateTime lastLogin;


}

```

![image-20230719150307623](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719150307623.png)



**2.在userController类中写一个方法测试**

```java
package com.example.vueblog.controller;


import com.example.vueblog.common.lang.Result;
import com.example.vueblog.entity.User;
import com.example.vueblog.service.UserService;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author anonymous
 * @since 2023-07-18
 */
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @RequiresAuthentication
    @GetMapping("/index")
    public  Object index(){
        User user = userService.getById(1);
        return Result.succ(200,"操作成功", user);
//        return userService.getById(1);
    }

    /**
     *  @RequestBody主要用来接受前端传递给后端的json字符串中数据（请求体数据）。
     *  前端使用POST提交（因为GET无请求体）
     *  @RequestBody和@RequestParam()可以同时使用，但是@RequestBody最多有一个而@RequestParam可以有多个
     *  @Validated注解用于检查user中填写的规则，如果不满足则抛出异常。可以在GlobalExceptionHandler中捕获此异常 进行自定义返回信息
     * @param user
     * @return
     */
    @PostMapping("/save")
    public  Result save(@Validated @RequestBody User user){

        return Result.succ(user);
    }


}

```



![image-20230719151010139](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719151010139.png)







**3.定义捕获异常返回处理**

在捕获异常 GlobalExceptionHandler类中修改如下：

```java
    /**
     * 实体校验异常
     * MethodArgumentNotValidException捕获实体校验异常
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST) //因为前后端分离 返回一个状态
    @ExceptionHandler(value =  MethodArgumentNotValidException.class)//捕获运行时异常
    public Result handler(MethodArgumentNotValidException e){
        log.error("实体捕获异常  ：-----------------{}",e);
        BindingResult bindingException = e.getBindingResult();
        //多个异常顺序抛出异常,让返回的异常信息简短
        ObjectError objectError = bindingException.getAllErrors().stream().findFirst().get();
        return Result.fail(objectError.getDefaultMessage());
    }


```



![image-20230719151616610](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719151616610.png)



**测试**

![image-20230720190146868](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190146868.png)









## 七、解决跨域问题

因为是前后端分离，所以跨域问题是避免不了的，我们直接在后台进行全局跨域处理:



**1.路径`com.example.vuehub.config.CorsConfig`**



```java
package com.example.vueblog.config;


import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 解决跨域问题
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
//                .allowedOrigins("*") SpringBoot升级2.4.0之后，跨域配置中的.allowedOrigins不再可用,将配置中的.allowedOrigins替换成.allowedOriginPatterns即可。
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}

```

![image-20230719152317557](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719152317557.png)



>  **注意：此配置是配置到confroller的，但是在confroller之前是经过jwtFilter的，所以在进行访问之前也配置一下Filter的跨域问题**

**2.jwtFilter进行跨域处理配置:**

 ```java
    /**
     * 对跨域提供支持
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = WebUtils.toHttp(request);
        HttpServletResponse httpServletResponse = WebUtils.toHttp(response);
        httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
        httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,PUT,DELETE");
        httpServletResponse.setHeader("Access-Control-Allow-Headers", httpServletRequest.getHeader("Access-Control-Request-Headers"));
        // 跨域时会首先发送一个OPTIONS请求,这里我们给OPTIONS请求直接返回正常状态
        if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
            httpServletResponse.setStatus(org.springframework.http.HttpStatus.OK.value());
            return false;
        }
        return super.preHandle(request, response);
    }

 ```

![image-20230719152612941](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719152612941.png)



由于我们的接口比较简单，所以就不继承swagger2了

**----基本框架已经搭建完成-----**







## 八、登录接口开发



- **创建LoginDto**

>  因为dto是要提供给service层需要的数据，是从vo转换来的。

`com.example.vueblog.common.dto`

```java
package com.example.vueblog.common.lang.dto;

import lombok.Data;

import javax.validation.constraints.NotBlank;
import java.io.Serializable;

@Data
public class LoginDto implements Serializable {

    @NotBlank(message = "用户名不能为空")
    private String username;
    @NotBlank(message = "密码不能为空")
    private String password;
}


```

![image-20230719155558499](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719155558499.png)





- **在GlobalExceptionHandler类中增加断言异常**

com/example/vueblog/exception/GlobalExceptionHandler.java

```java
    /**
     * 断言异常
     * @param e
     * @return
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = IllegalArgumentException.class)
    public Result handler(IllegalArgumentException e){
        log.error("Assert异常:------------------>{}",e);
        return Result.fail(e.getMessage());
    }



```





- **创建AccountController类**

登录和退出逻辑

com/example/vueblog/controller/AccountController.java

```java
package com.example.vueblog.controller;

import cn.hutool.core.lang.Assert;
import cn.hutool.core.map.MapUtil;
import cn.hutool.crypto.SecureUtil;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.example.vueblog.common.lang.Result;
import com.example.vueblog.common.lang.dto.LoginDto;
import com.example.vueblog.entity.User;
import com.example.vueblog.service.UserService;
import com.example.vueblog.util.JwtUtils;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;

@RestController
public class AccountController {
    @Autowired
    UserService userService;

    @Autowired
    JwtUtils jwtUtils;

    public Result login(@Validated @RequestBody LoginDto loginDto, HttpServletResponse response){
        User user = userService.getOne(new QueryWrapper<User>().eq("userrname",loginDto.getUsername()));
        Assert.notNull(user,"用户不存在");
        //判断账号密码是否错误 因为是md5加密所以这里md5判断
        if(!user.getPassword().equals(SecureUtil.md5(loginDto.getPassword()))){
            //密码不同则抛出异常
            return Result.fail("密码不正确");
        }
        String jwt = jwtUtils.generateToken(user.getId());
        //将token 放在我们的header里面
        response.setHeader("Authorization",jwt);
        response.setHeader("Access-control-Expose-Headers","Authorization");

        return Result.succ(MapUtil.builder()
                .put("id", user.getId())
                .put("username",user.getUsername())
                .put("avatar",user.getAvatar())
                .put("email",user.getEmail()).map()
        );

    }

    //需要认证权限才能退出登录
    @RequiresAuthentication
    @RequestMapping("/logout")
    public Result logout() {
        //退出登录
        SecurityUtils.getSubject().logout();
        return null;
    }

    

}

```



**问题，可能会遇到这个报错：`Caused by: java.lang.ClassNotFoundException: javax.xml.bind.DatatypeConverter `**

其主要原因，主要是 因为 Spring Boot 版本过高了，需要我们给它指定「jaxb」的版本即可。

在pom.xml中添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api -->
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.1</version>
</dependency>

```







**测试效果：**



![image-20230720190217446](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190217446.png)



![image-20230720190243307](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190243307.png)





这里根本没去调用shiro的登录方法吧，写的jwtFilter也根本不会去走...........



此处登陆时发现，jdk版本大于8时会报 java.lang.NoClassDefFoundError: javax/xml/bind/DatatypeConverter



## 九、博客接口开发

列表、详情、编辑功能

没有做删除功能，自己做



Blog的实体类字段也加上验证：

```java
package com.example.vueblog.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import java.time.LocalDateTime;
import java.io.Serializable;
import lombok.Data;
import lombok.EqualsAndHashCode;

import javax.validation.constraints.NotBlank;

/**
 * <p>
 * 
 * </p>
 *
 * @author anonymous
 * @since 2023-07-18
 */
@Data
@EqualsAndHashCode(callSuper = false)
@TableName("m_blog")
public class Blog implements Serializable {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    private Long userId;
    
    @NotBlank(message = "标题不能为空")
    private String title;
    
    @NotBlank(message = "摘要不能为空")
    private String description;
    
    @NotBlank(message = "内容不能为空")
    private String content;

    private LocalDateTime created;

    private Integer status;


}

```





**创建工具类ShiroUtil**

com/example/vueblog/util/ShiroUtil.java

```java
package com.example.vueblog.util;

import com.example.vueblog.shiro.AccountProfile;
import org.apache.shiro.SecurityUtils;

public class ShiroUtil {
    public static AccountProfile getProfile(){

        return (AccountProfile) SecurityUtils.getSubject().getPrincipal();
    }
}

```





**完善BlogController类**

```java
package com.example.vueblog.controller;


import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.lang.Assert;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.vueblog.common.lang.Result;
import com.example.vueblog.entity.Blog;
import com.example.vueblog.service.BlogService;
import com.example.vueblog.util.ShiroUtil;
import org.apache.shiro.authz.annotation.RequiresAuthentication;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;

import org.springframework.beans.factory.annotation.Autowired;

import java.time.LocalDateTime;


/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author anonymous
 * @since 2023-07-18
 */
@RestController
@RequestMapping("/blog")
public class BlogController {
    @Autowired
    BlogService blogService;

    //木有值默认为1
    @GetMapping("/blogs")
    public Result list(@RequestParam(defaultValue = "1") Integer currentPage){
        Page page = new Page(currentPage, 5);
        IPage pageData = blogService.page(page, new QueryWrapper<Blog>().orderByDesc("created"));
        return Result.succ(pageData);
    }

    //查看文章详情
    //@PathVariable动态路由
    @GetMapping("/blog/{id}")
    public Result detail(@PathVariable Long id){
        Blog blog = blogService.getById(id);        //判断是否为空 为空则断言异常
        Assert.notNull(blog,"该博客已被删除");
        return Result.succ(blog);

    }

    //@Validated校验
    //@RequestBody
    //添加删除  木有id则添加 有id则编辑
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blog/edit")
    public Result edit(@Validated @RequestBody Blog blog){
        //一个空对象用于赋值
        Blog temp = null;
        //如果有id则是编辑
        if(blog.getId() != null){
            temp = blogService.getById(blog.getId());//将数据库的内容传递给temp
            //只能编辑自己的文章
            Assert.isTrue(temp.getUserId().longValue()== ShiroUtil.getProfile().getId().longValue(),"没有编辑权限");

        }else{
            temp = new Blog();
            temp.setUserId(ShiroUtil.getProfile().getId());
            temp.setCreated(LocalDateTime.now());
            temp.setStatus(0);
        }
        //将blog的值赋给temp 忽略 id userid created status 引用于hutool
        BeanUtil.copyProperties(blog,temp,"id","userId","created","status");
        blogService.saveOrUpdate(temp);

        return Result.succ(null);
    }

    //删除文章
    //@PathVariable动态路由
    @RequiresAuthentication  //需要认证之后才能操作
    @PostMapping("/blogdel/{id}")
    public Result edit(@PathVariable Long id){
        //判断是否是自己的文章(文章id属于自己)
        Assert.isTrue(blogService.getById(id).getUserId().longValue() == ShiroUtil.getProfile().getId().longValue(),"没有删除权限");

        boolean b = blogService.removeById(id);
        //判断是否为空 为空则断言异常
        if(b==true){
            return Result.succ("文章删除成功");
        }else{
            return Result.fail("文章删除失败");
        }
    }




    }

```







- **查询测试:**

![image-20230720190304997](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190304997.png)





![image-20230720190327330](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190327330.png)



- **新增编辑测试:**

先登录，获取登录的token再将其添加到请求头，去测试后面的授权功能

![image-20230719173237757](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719173237757.png)



![image-20230719173244908](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719173244908.png)





新增：

```
{
	"title":"标题测试",
	"description":"描述测试哦奥德萨",
	"content":"内容测试阿斯顿撒哦i等哈送到哈桑"
}

```

![image-20230720190400649](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190400649.png)

编辑：

```
{
	"id":21,
	"title":"标题测试修改",
	"description":"描述测试哦奥德萨",
	"content":"内容测试阿斯顿撒哦i等哈送到哈桑"
}

```







- **文章删除**

![image-20230720190425046](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720190425046.png)













# 前端

**接下来，我们来完成vueblog前端的部分功能。可能会使用的到技术如下：
vue
element-ui
axios
mavon-editor
markdown-it
github-markdown-css**







## 十、环境准备









1.首先我们上node.js官网(https://nodejs.org/zh-cn/)，下载最新的长期版本，直接运行安装完成之后，我们就已经具备了node和npm的环境啦。



2.安装完成之后检查下版本信息：

```
node -v
npm -v
```



3.安装vue环境

```bash
# 可以配置淘宝npm源
npm get registry #查看原本镜像
npm config set registry http://registry.npm.taobao.org/ #修改成淘宝镜像


# 脚手架vue-cli
# 官方文档：https://cli.vuejs.org/#getting-started
sudo npm install -g @vue/cli


```







## 十一、新建项目

**新建项目：**

```bash
# 切换到创建项目的目录下(默认会创建一个hello示例项目)
vue create vueblog-vue
# 然后会让你选择vue2还是vue3
# 会自动生成一个demo项目目录

# 切换到demo目录下
cd vueblog-vue
# 运行demo
npm run serve

```

用vscode打开项目即可



**Note:vue create 创建项目报错：command failed: npm install --loglevel error**

![image-20230719193659165](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719193659165.png)

解决办法：

清空npm的缓存

```bash
sudo npm cache clean --force
```

或者在文件目录C:\Users下查找 **.npmrc**文件，并删除





**项目架构：**

```
├── README.md            项目介绍
├── index.html           入口页面
├── build              构建脚本目录
│  ├── build-server.js         运行本地构建服务器，可以访问构建后的页面
│  ├── build.js            生产环境构建脚本
│  ├── dev-client.js          开发服务器热重载脚本，主要用来实现开发阶段的页面自动刷新
│  ├── dev-server.js          运行本地开发服务器
│  ├── utils.js            构建相关工具方法
│  ├── webpack.base.conf.js      wabpack基础配置
│  ├── webpack.dev.conf.js       wabpack开发环境配置
│  └── webpack.prod.conf.js      wabpack生产环境配置
├── config             项目配置
│  ├── dev.env.js           开发环境变量
│  ├── index.js            项目配置文件
│  ├── prod.env.js           生产环境变量
│  └── test.env.js           测试环境变量
├── mock              mock数据目录
│  └── hello.js
├── package.json          npm包配置文件，里面定义了项目的npm脚本，依赖包等信息
├── src               源码目录 
│  ├── main.js             入口js文件
│  ├── app.vue             根组件
│  ├── components           公共组件目录
│  │  └── title.vue
│  ├── assets             资源目录，这里的资源会被wabpack构建
│  │  └── images
│  │    └── logo.png
│  ├── routes             前端路由
│  │  └── index.js
│  ├── store              应用级数据（state）状态管理
│  │  └── index.js
│  └── views              页面目录
│    ├── hello.vue
│    └── notfound.vue
├── static             纯静态资源，不会被wabpack构建。
└── test              测试文件目录（unit&e2e）
  └── unit              单元测试
    ├── index.js            入口脚本
    ├── karma.conf.js          karma配置文件
    └── specs              单测case目录
      └── Hello.spec.js


```









## 十二、安装element-ui和axios

### **安装element-ui plus**

> **vue3以上得下载element-puls才行**

**安装：**

```bash
# 安装
# npm install element-ui --save
npm install element-plus --save

```

或者如下方式安装(会自动帮助适配版本)：

```bash
vue add element-plus
# 参考：https://www.jianshu.com/p/02b8616d4fe6
```





**全局引入：**

然后打开main.js,引入依赖

```js
import { createApp } from 'vue'
import App from './App.vue'

// 引入element-plus
import ElementPlus from "element-plus"
import "element-plus/dist/index.css"


// 使用element-plus
createApp(App)
    .use(ElementPlus)
    .mount('#app')

```

按需引入：生产环境建议这种，不建议全局引入



**测试：**

在App.vue中添加button，查看效果

```html
<template>
    <el-row class="mb-4">
        <el-button>Default</el-button>
        <el-button type="primary">Primary</el-button>
        <el-button type="success">Success</el-button>
        <el-button type="info">Info</el-button>
        <el-button type="warning">Warning</el-button>
        <el-button type="danger">Danger</el-button>
    </el-row>
</template>
```

生效了

![image-20230719194923846](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719194923846.png)





### **安装axios**

类似ajax

```
npm install axios --save
```



后面就可以使用`axios.get()`发送请求了





## 十三、页面路由

接下来，我们先定义好路由和页面，因为我们只是做一个简单的博客项目，页面比较少，所以我们可以直接先定义好，然后在慢慢开发，这样需要用到链接的地方我们就可以直接可以使用：
我们在views文件夹下定义几个页面：

- BlogDetail.vue（博客详情页）
- BlogEdit.vue（编辑博客）
- Blogs.vue（博客列表）
- Login.vue（登录页面）

> 可以配置插件:VueHelper(新建vue项目 最上方输入vuet按键tab可直接生成模板)



### 先新建几个基本页面

> 注意:每个页面下方<template><temlate> 里面只能有一个<div></div>  ,vue3开始就可以有多个了

删除多余的页面：

![image-20230719201501986](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719201501986.png)

新建四个组件：

![image-20230719201605027](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719201605027.png)



- BlogDetail.vue（博客详情页）

```vue
<template>
<div>BlogDetail</div>
  
</template>

<script>
export default {
    name: "BlogDetail"

}
</script>

<style>

</style>
```



- BlogEdit.vue（编辑博客）

```vue
<template>
<div>BlogEditor</div>
  
</template>

<script>
export default {
    name: "BlogEditor"

}
</script>

<style>

</style>
```



- Blogs.vue（博客列表）

```vue
<template>
<div>Blogs</div>
  
</template>

<script>
export default {
    name: "Blogs"

}
</script>

<style>

</style>
```



- Login.vue（登录页面）

```vue
<template>
<div>Login</div>
  
</template>

<script>
export default {
    name: "Login"

}
</script>

<style>

</style>
```







### *配置路由：*

1.vue安装router:

```bash
npm install vue-router@next --save
```



2.引入

**src** 文件夹下创建 **router** 文件夹，**router** 文件夹下创建 **index.js** 文件

![image-20230719195329435](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719195329435.png)

为保证代码整洁，可以将routes=[{…}]部分提取到另一js文件:

router/index.js

```js
//该文件专门用于创建整个应用的路由器

//引用
import { createRouter,createWebHashHistory} from "vue-router";


//为保证代码整洁，可以将routes=[{…}]部分提取到另一js文件；或通过api动态加载路由
import routes from './routes'

//创建一个路由器
const router = createRouter({
    routes:routes,
    history:createWebHashHistory(process.env.BASE_URL)
})

export default router






```





router/routes.js 放具体的路由

```js
// import Demo from '@/views/Demo' 
import Login from '@/views/Login' 
import Blogs from '@/views/Blogs' 
import BlogEdit from '@/views/BlogEdit' 
import BlogDetail from '@/views/BlogDetail' 

//创建具体的路由
const routes = [
    {
        path: '/',
        name: 'Index',
        redirect:{name : "Blogs"}
      },
      {
        path: '/blogs',
        name: 'Blogs',
        component: Blogs
      },{
      path: '/Login',
      name: 'Login',
      component: Login
      },{
        path: '/blog/add',
        name: 'BlogAdd',  //name不能重复
        component: BlogEdit
      }, 
    //   {
    //     path: '/Demo',
    //     name: 'Demo',
    //     component: Demo
    //   },
      {
        path: '/blog/:blogid',
        name: 'BlogDetail',
        component: BlogDetail
      } ,{
        // path: '/blog/:blogid/edit', vue2,而vue3中应该edit在前面，id在后面
        path: '/blog/edit/:blogid',
        name: 'BlogEdit', 
        component: BlogEdit
      }
  
]

export default routes
```



3.main.js文件中引入路由

```js
import { createApp } from 'vue'
import App from './App.vue'

// 引入element-plus
import ElementPlus from "element-plus"
import "element-plus/dist/index.css"
import router from "router"

// 使用element-plus
createApp(App)
    .use(ElementPlus)
    .use(router)
    .mount('#app')

```





4.添加路由组件router-view，指定路由插入位置（划重点）

App.vue

```vue
<template>
    <router-view></router-view>   <!--加上路由标签，路由才会有效果-->

</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  // components: {
  //   // HelloWorld
  // }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```







### 登录功能

#### 登录页面编写：

```vue
<template>
  <div>
    <el-container>
      <el-header>
        <img class="mlogo" src="https://geoer.cn/images/avatar.png" alt="logo图片" />
      </el-header>
      <el-main>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="用户姓名" prop="username">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="用户密码" prop="password">
            <el-input v-model="ruleForm.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>
<script>
export default {
  name: "Login",
  data () {
    return {
      ruleForm: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }

        ],
        password: [
          { required: true, message: '请选择密码', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert('submit!');
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm (formName) {
      this.$refs[formName].resetFields();
    }
  }
}
</script>

<style scoped>
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-main {
  /* background-color: #e9eef3; */
  color: #333;
  text-align: center;
  line-height: 160px;
}
.mlogo {
  height: 80%;
}

.demo-ruleForm {
  max-width: 500px;
  margin: 0 auto;
}
</style>

```







#### **vue项目拿到的token常放到localStore里面：**

> 把这个jwt让其他所有组件也能访问到
>
> vue的各个页面不能直接互相通信，可以通过store来数据互通，这个是 保存数据存在本地cookie或者会话session中，其他页面要用数据时就可以从这里面取

新建一个`store`目录，下面新建一个`index.js`:配置完善store/index.js

相当于Java Bean: `state相当于私有属性,mutations相当于set，getter相当于get`

```js
// import Vue from 'vue'
// import Vuex from 'vuex'
import { createStore } from "vuex"

// Vue.use(Vuex)

export default createStore({
  //定义全局参数 其他页面可以直接获取state里面的内容
  state: {
    token: localStorage.getItem("token") , //方法一 localStorage.getItem("token") 
    //反序列化获取session会话中的 userInfo对象
    userInfo:JSON.parse(sessionStorage.getItem("userInfo"))
  },
  mutations: {
    //相当于实体类的set
    SET_TOKEN:(state,token)=>{
      state.token=token//将传入的token赋值 给state的token
      //同时可以存入浏览器的localStorage里面
      localStorage.setItem("token",token)
    },
    SET_USERINFO:(state,userInfo)=>{
      state.userInfo=userInfo//将传入的tuserInfo赋值 给state的userInfo
      //同时可以存入会话的sessionStorage里面 sessionStorage中只能存字符串 不能存入对象所以我们存入序列化 jons串
      sessionStorage.setItem("userInfo",JSON.stringify(userInfo))
    },
    //删除token及userInfo
    REMOVE_INFO:(state)=>{
      state.token = '';
      state.userInfo = {};
      localStorage.setItem("token",'')
      sessionStorage.setItem("userInfo",JSON.stringify(''))
    }
  },
  getters: {
    //相当于get
    //配置一个getUser可以直接获取已经反序列化对象的一个userInfo
   getUser: state=>{
     return state.userInfo;
   },
   getToken: state=>{
    return state.token;
  }
  },
  actions: {
    
  },
  modules: {
    
  }
})


```



然后我们给main.js中引入store和axios

```js
import { createApp } from 'vue'
import App from './App.vue'

// 引入element-plus
import ElementPlus from "element-plus"
import "element-plus/dist/index.css"
import router from "./router/index"
import store from './store'
import axios from 'axios'



// // 使用element-plus
// createApp(App)
//     .use(ElementPlus)
//     .use(router)
//     .use(store)
//     .mount('#app')

const app = createApp(App)
            .use(ElementPlus)
            .use(router)
            .use(store)

app.mount('#app')
//将axios挂载为app的全局自定义属性之后
//每个组件可以通过this直接访问到全局挂载的自定义属性：在组件中发起axios请求：this.$http.get('/users')
app.config.globalProperties.$http = axios

```







#### **登录功能编写：**

在Login.vue中添加请求

```vue
<template>
  <div>
    <el-container>
      <el-header>
        <img class="mlogo" src="https://geoer.cn/images/avatar.png" alt="logo图片" />
      </el-header>
      <el-main>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
          <el-form-item label="用户姓名" prop="username">
            <el-input v-model="ruleForm.username"></el-input>
          </el-form-item>
          <el-form-item label="用户密码" prop="password">
            <el-input v-model="ruleForm.password"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">立即登录</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>
<script>
export default {
  name: "Login",
  data () {
    return {
      ruleForm: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 15, message: '长度在 3 到 15 个字符', trigger: 'blur' }

        ],
        password: [
          { required: true, message: '请选择密码', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
            const _this = this;//获取整个vue的this
            //   alert('submit!');
            //   _this.$http.post("http://localhost:8081/login", _this.ruleForm).then(res => {
            _this.$http.post("/login", _this.ruleForm).then(res => {

            // console.log(res)
            const jwt = res.headers['authorization'];
            const userInfo = res.data.data
            //存储(共享)全局变量jwt和userInfo
            _this.$store.commit("SET_TOKEN", jwt);
            _this.$store.commit("SET_USERINFO", userInfo);
            //登录成功后自动跳转到博客列表
            _this.$router.push("/blogs")
            //获取token和getUser
            // console.log(_this.$store.getters.getToken)
            // console.log(_this.$store.getters.getUser) 

          })
          .catch((err)=>{
            console.log(err);
         })

        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm (formName) {
      this.$refs[formName].resetFields();
    }
  }
}
</script>

<style scoped>
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-main {
  /* background-color: #e9eef3; */
  color: #333;
  text-align: center;
  line-height: 160px;
}
.mlogo {
  height: 80%;
}

.demo-ruleForm {
  max-width: 500px;
  margin: 0 auto;
}
</style>

```







**测试：**

打开页面点击登录
可查看我们的信息已经存入浏览器中



![image-20230719220955962](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230719220955962.png)







#### **配置axios全局拦截**

>  用户名和密码不正确的状态的弹窗效果，全局处理
>
>  前置拦截、后置拦截



 1.src下面创建axios.js，并完善axios.js

![image-20230720092922158](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720092922158.png)

```js
// 引入axios依赖
import axios from 'axios'
// 引入element-ui依赖
import {ElementPlus,ElMessage} from "element-plus"
import router from "./router/index"
import store from './store/index'



//配置默认前缀：也可以在config中配置
axios.defaults.baseURL= "http://localhost:8081/";  //获取连接前缀相当于 http://localhost:8081/。axios发送请求可以直接写后面部分了

//配置前置拦截
// 前置拦截，其实可以统一为所有需要权限的请求装配上header的token信息，这样不需要在使用是再配置，我的小项目比较小，所以，还是免了吧~
axios.interceptors.request.use(config => { 
  return config
})

//配置后置拦截
// res => res.data  取出data数据，将来调用接口的时候直接拿到的就是后台的数据
axios.interceptors.response.use(response=>{
    let res = response.data; 
    if(res.code == 200){
      return response
    }else{ 
        console.log(res)
        ElMessage.error(res.msg)
        //返回一个异常提示就不会继续往下走axios的then了 不+reject的话 res=>的里面 还是会继续走的
        return Promise.reject(response.data.msg)
    }
     // 捕获并处理后台异常信息
  },error=>{
     // 使得异常信息更加友好
    console.log(error) 
    if (error.response.data) { //data不为空时
      error.message = error.response.data.msg
      console.log("-------------------------")
      console.log(error.message)
      console.log("-------------------------")
    }
    if(error.response.status == 401){
      store.commit('REMOVE_INFO')//清空token userinfo
      router.push("/login")  //跳转登录页面
    } 
    ElMessage.error(error.message)
    
    return Promise.reject(error)
  }

)

```







2.main.js中引入

```js
import "./axios.js"
```









### 公共组件Header



要先打开**AccountController**，在logout方法里面返回一个Result 不然会报错：

![image-20230720150033245](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720150033245.png)





1.新建header组件：

![image-20230720143847950](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720143847950.png)

```vue
<template>
  <div class="m-content">
    <h3>欢迎来到我的博客</h3>
    <div class="block">
      <div class="block">
        <el-avatar :size="50" :src="user.avatar"></el-avatar>
      </div>
      <div>{{user.username}}</div>
      <div class="maction">
        <span>
          <el-link>主页</el-link>
        </span>
        <el-divider direction="vertical"></el-divider>
        <span>
          <el-link type="success">发表博客</el-link>
        </span>

        <el-divider direction="vertical"></el-divider>
        <span v-show="!hasLogin">
          <el-link type="primary" @click="login">登录</el-link>
        </span>
        <span v-show="hasLogin">
          <el-link type="danger" @click="logout">退出</el-link>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
    export default {
    name: "Header"
    , data () {
        return {
        user: {
            username: '请先登录',
            avatar: 'https://geoer.cn/images/avatar.png'
        },
        hasLogin: false
        }
    },
    methods: {
        //退出操作
        logout () {
        const _this = this
        //首先调用后端logout接口(因该接口需要认证权限,所以需要传入token)
        //其次调用$store清除用户信息及token
        _this.$http.get("/logout", {
            headers: {
            "Authorization": localStorage.getItem("token")
            }
        }).then(res => {
            _this.$store.commit("REMOVE_INFO")
            _this.$router.push("/login")
        })
        },
        login () { //跳转登录页面进行登录
        this.$router.push("/login")
        }
    },
    //页面创建时即会调用,进而获取用户信息
    created () {
        console.log("=======123=")
        console.log(this.$store.getters.getUser)
        if (this.$store.getters.getUser.username) {//如果username不为空
            this.user.username = this.$store.getters.getUser.username
            this.user.avatar = this.$store.getters.getUser.avatar
            //判断是登录状态还是非登录显示 退出按钮或者登录按钮
            this.hasLogin = true;
        }
    }
    }
</script>

<style>
    .m-content {
    max-width: 960px;
    margin: 0 auto;
    text-align: center;
    }

</style>

```







2.引入

![image-20230720144340003](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720144340003.png)

```vue
<template>
<Header></Header>
<div>Blogs</div>
  
</template>

<script>
import Header from "../components/Header.vue"

export default {
    name: "Blogs",
    //注册组件
    components:{
        Header
    }

}
</script>

<style>

</style>
```





**自行测试登录退出功能**

![image-20230720150750350](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720150750350.png)



![image-20230720150810620](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720150810620.png)







### 博客列表页面开发

使用时间线的样式组件+分页



**时间格式：**

先给实体类Blog加上JsonFormat

```
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime created;

```

![image-20230720152721130](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720152721130.png)



完善Blogs.vue内容：

```vue
<template>
  <div class="mcontaner">
    <Header></Header>
    <div class="block">
      <el-timeline>
        <el-timeline-item :timestamp="blog.created" placement="top" v-for="(blog,key) in blogs" :key=key>
          <el-card>

            <h4>
                <!-- 路由 -->
              <router-link :to="{  name: 'BlogDetail', params: {blogid: blog.id}}">{{blog.title}}</router-link>
            </h4>

            <p>{{blog.description}}</p>
          </el-card>
        </el-timeline-item>

      </el-timeline>

      <el-pagination class="mage" background layout="prev, pager, next" :current-page="currentPage"
        :page-size="pageSize" :total="total" @current-change=page>
      </el-pagination>
    </div>
  </div>
</template>

<script>
//引入header组件
import Header from "../components/Header";

export default {
    name: "Blogs",
    components: { Header },

    data () {
        return {
        blogs: {},
        currentPage: 1,
        total: 0,
        pageSize: 5
        }
    },

    methods: {
        page (currentPage) {
        const _this = this
        _this.$http.get("/blogs?currentPage=" + currentPage).then(res => {
            var data = res.data.data
            _this.blogs = data.records
            _this.currentPage = data.current
            _this.pageSize = data.size
            _this.total = data.total

        })
        }
    },
  created () {
    this.page(1)
  }
}
</script>

<style scoped>
.block {
  margin: 20px;
}

.mage {
  margin: 0 auto;
}
</style>

```









App.vue中统一添加根标签的样式：

![image-20230720153419795](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720153419795.png)















### 博客发表/编辑页面开发



安装markdown编辑器：这里使用` mavon-editor`

```bash
npm install mavon-editor --save
```

<font color="red">mavon-editor不支持wue3啊，VUE3的话可以换成v-md-editor</font>

```bash
npm i @kangc/v-md-editor@next -S

```



为了方便，我这里选择全局注册使用。那么在main.js里面进行注册

> 其实不太推荐直接在main.ts中引入，会显得整个文件特别臃肿
>
> 以后可以在plugins文件夹下引入第三方插件
>
> 我这里先偷个懒吧... 

```js
import { createApp } from 'vue'
import App from './App.vue'

// 引入element-plus
import ElementPlus from "element-plus"
import "element-plus/dist/index.css"
import router from "./router/index"
import store from './store/index'
import axios from 'axios'
import "./axios.js"

import VueMarkdownEditor from '@kangc/v-md-editor';
import '@kangc/v-md-editor/lib/style/base-editor.css';
import vuepressTheme from '@kangc/v-md-editor/lib/theme/vuepress.js';
import '@kangc/v-md-editor/lib/theme/style/vuepress.css';
import Prism from 'prismjs';


VueMarkdownEditor.use(vuepressTheme, {
    Prism,
  });


// // 使用element-plus
const app = createApp(App)
            .use(ElementPlus)
            .use(router)
            .use(store)
            .use(VueMarkdownEditor)

app.mount('#app')
//将axios挂载为app的全局自定义属性之后
//每个组件可以通过this直接访问到全局挂载的自定义属性：在组件中发起axios请求：this.$http.get('/users')
app.config.globalProperties.$http = axios

```



**编写BlogEdit.vue**

```vue
<template>
  <div>
    <Header></Header>
    <div class="m-content">
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">

        <el-form-item label="标题" prop="title">
          <el-input v-model="ruleForm.title"></el-input>
        </el-form-item>

        <el-form-item label="摘要" prop="description">
          <el-input type="textarea" v-model="ruleForm.description"></el-input>
        </el-form-item>

        <el-form-item label="内容" prop="content">
          <v-md-editor v-model="ruleForm.content" height="600px"></v-md-editor>
          <!-- <mavon-editor v-model="ruleForm.content"></mavon-editor> -->


        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
    import Header from "../components/Header";
    export default {
    name: "BlogEdit",
    components: { Header },
    data () {
        return {
        ruleForm: {
            id: '',
            title: '',
            description: '',
            content: ''
        },
        rules: {
            title: [
            { required: true, message: '请输入标题', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 15 个字符', trigger: 'blur' }
            ],

            description: [
            { required: true, message: '请输入摘要', trigger: 'blur' },
            ],
            content: [
            { required: true, message: '请输入内容', trigger: 'blur' },
            ]
        }
        };
    },
    methods: {
        submitForm (formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                const _this = this
                this.$http.post('/blog/edit', this.ruleForm, {
                    headers: {
                    "Authorization": localStorage.getItem("token")
                    }
                }).then(res => {
                    _this.$alert('操作成功', '提示', {
                    confirmButtonText: '确定',
                    callback: action => {
                        _this.$router.push("/blogs")
                    }
                    });
                })
                } else {
                console.log('error submit!!');
                return false;
                }
            });
        },
        resetForm (formName) {
        this.$refs[formName].resetFields();
        }
    },
    //编辑：如果有这个blogid，就获取，进行编辑
    created () {
        //获取动态路由的 blogid
        const blogId = this.$route.params.blogid
        console.log(blogId)
        const _this = this
        if (blogId) {
        this.$http.get("/blog/" + blogId).then(res => {
            console.log(res)
            const blog = res.data.data
            _this.ruleForm.id = blog.id
            _this.ruleForm.title = blog.title
            _this.ruleForm.description = blog.description
            _this.ruleForm.content = blog.content

        })
        }

    }
    }
</script>

    <style scoped>
    .m-content {
    text-align: center;
    }
</style>

```



**Note：**VUE3的话跳转到Detail要把edit放到最后面，顺序和视频里是反过来的。

![image-20230720161422406](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720161422406.png)



![image-20230720161911311](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720161911311.png)









### 博客详情页面开发&编辑按钮权限

博客详情中需要回显博客信息，然后有个问题就是，后端传过来的是博客内容是markdown格式的内容，我们需要进行渲染然后显示出来，这里我们使用一个插件markdown-it，用于解析md文档，然后导入github-markdown-c，所谓md的样式。

```
npm install vue3-markdown-it --save
```



BlogDetail.vue

```vue
<template>
  <div>
    <Header></Header>
    <div class="blog">
      <h2>{{blog.title}}</h2>

      <el-link icon="el-icon-edit" v-if="ownBlog">
        <!-- 按钮设置权限，不是自己的id不显示编辑 -->
        <router-link :to="{name:'BlogEdit',params:{blogid:blog.id}}">
          编辑
        </router-link>
      </el-link>

      <el-divider></el-divider>
      <div class="markdown-body" v-html="blog.content"></div>
    </div>
  </div>
</template>

<script>
// import MarkdownIT from 'vue3-markdown-it';
import 'highlight.js/styles/a11y-dark.css';
import Header from "../components/Header";
export default {
  name: "BlogDetail",
  components: { Header },
  data () {
    return {
      blog: {
        id: '',
        title: '',
        content: '',
        description: ''

      },
      ownBlog: false
    }
  },
  created () {
    //获取动态路由的 blogid
    const blogId = this.$route.params.blogid
    // const _this = this
    if (blogId) {
      this.$http.get("/blog/" + blogId).then(res => {
        const blog = res.data.data
        this.blog.id = blog.id
        this.blog.title = blog.title
        this.blog.description = blog.description

        //MardownIt 渲染
        var MardownIt = require("markdown-it")
        var md = new MardownIt();
        var result = md.render(blog.content)
        this.blog.content = result
        //查看是否是登录人 是则可以编辑
        this.ownBlog = (blog.userId === this.$store.getters.getUser.id)
        console.log(this.ownBlog)
      })
    }

  }
}
</script>

<style scoped>
.blog {
  margin-top: 10px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  width: 100%;
  min-height: 700px;
  padding: 10px;
}
</style>

```













### 路由权限拦截

因为我们是前后端分离的项目。页面已经开发完毕之后，我们来控制一下哪些页面是需要登录之后才能跳转的，如果未登录访问就直接重定向到登录页面，因此我们在src目录下定义一个js文件



 

1.通过之前我们定义页面路由时候的的`meta`信息，指定`requireAuth: true`，需要登录才能访问，因此这里我们在每次路由之前（router.beforeEach）判断token的状态，觉得是否需要跳转到登录页面。

![image-20230720172623732](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720172623732.png)

```js
// import Demo from '@/views/Demo' 
import Login from '@/views/Login' 
import Blogs from '@/views/Blogs' 
import BlogEdit from '@/views/BlogEdit' 
import BlogDetail from '@/views/BlogDetail' 

//创建具体的路由
const routes = [
    {
        path: '/',
        name: 'Index',
        redirect:{name : "Blogs"}
      },
      {
        path: '/blogs',
        name: 'Blogs',
        component: Blogs
      },{
      path: '/Login',
      name: 'Login',
      component: Login
      },{
        path: '/blog/add',   // 注意放在 path: '/blog/:blogId'之前
        name: 'BlogAdd',
        component: BlogEdit,
        meta: {
            requireAuth: true
          }
      }, 
      {
        path: '/blog/:blogid',
        name: 'BlogDetail',
        component: BlogDetail
      } ,{
        // path: '/blog/:blogid/edit', vue2,而vue3中应该edit在前面，id在后面
        path: '/blog/edit/:blogid',
        name: 'BlogEdit', 
        component: BlogEdit,
        meta: {
            requireAuth: true
          }
      }
  
]

export default routes
```



 2.配置一个路由前置拦截

src下面新建一个`permission.js`:

```js
import router from "./router";
// 路由判断登录 根据路由配置文件的参数
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requireAuth)) { // 判断该路由是否需要登录权限
    const token = localStorage.getItem("token")
    console.log("------------" + token)
    if (token) { // 判断当前的token是否存在 ； 登录存入的token
      if (to.path === '/login') {
      } else {
        next()
      }
    } else {
      //如果不存在就跳到登录页面
      next({
        path: '/login'
      })
    }
  } else {
    next()
  }
})


```



然后我们在main.js中import我们的permission.js

![image-20230720173901939](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/image-20230720173901939.png)

 



测试：

退出状态下，我们再去访问/blog/add的时候就会自动重定向到/login





### 增加删除文章功能

**前端：src/views/BlogDetail.vue **

```
<template>
  <div>
    <Header></Header>
    <div class="blog">
      <h2>{{blog.title}}</h2>

      <el-link icon="el-icon-edit" v-if="ownBlog">
        <!-- 按钮设置权限，不是自己的id不显示编辑 -->
        <router-link :to="{name:'BlogEdit',params:{blogid:blog.id}}">
          编辑
        </router-link>
      </el-link>

      <el-link icon="el-icon-delete" v-if="ownBlog" class="linklist">
        <el-button type="danger" round @click="delblog">删除</el-button>
      </el-link>

      <el-divider></el-divider>
      <div class="markdown-body" v-html="blog.content"></div>
    </div>
  </div>
</template>

<script>
// import MarkdownIT from 'vue3-markdown-it';
import 'highlight.js/styles/a11y-dark.css';
import Header from "../components/Header";
export default {
  name: "BlogDetail",
  components: { Header },
  data () {
    return {
      blog: {
        id: '',
        title: '',
        content: '',
        description: ''

      },
      ownBlog: false
    }
  },
   methods: {
    delblog () {
      const blogId = this.$route.params.blogid
      const _this = this
      if (blogId) {
        this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          _this.$http.post(`/blogdel/${blogId}`, null, {
            headers: {
              "Authorization": localStorage.getItem("token")
            }
          }).then(res => {
            this.$message({
              type: 'success',
              message: res.data.data
            });
            _this.$router.push("/blogs")
          })

        }).catch(() => {

          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });


      }
    }
  },

  created () {
    //获取动态路由的 blogid
    const blogId = this.$route.params.blogid
    // const _this = this
    if (blogId) {
      this.$http.get("/blog/" + blogId).then(res => {
        const blog = res.data.data
        this.blog.id = blog.id
        this.blog.title = blog.title
        this.blog.description = blog.description

        //MardownIt 渲染
        var MardownIt = require("markdown-it")
        var md = new MardownIt();
        var result = md.render(blog.content)
        this.blog.content = result
        //查看是否是登录人 是则可以编辑
        this.ownBlog = (blog.userId === this.$store.getters.getUser.id)
        console.log(this.ownBlog)
      })
    }

  }
}
</script>

<style scoped>
.blog {
  margin-top: 10px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  width: 100%;
  min-height: 700px;
  padding: 10px;
}
</style>

```



**后端：**

src/main/java/com/vuelog/controller/BlogController

```java
//删除文章
//@PathVariable动态路由
@RequiresAuthentication  //需要认证之后才能操作
@PostMapping("/blogdel/{id}")
public Result edit(@PathVariable Long id){
  //判断是否是自己的文章(文章id属于自己)
  Assert.isTrue(blogService.getById(id).getUserId().longValue() == ShiroUtil.getProfile().getId().longValue(),"没有删除权限");

  boolean b = blogService.removeById(id);
  //判断是否为空 为空则断言异常
  if(b==true){
    return Result.succ("文章删除成功");
  }else{
    return Result.fail("文章删除失败");
  }
}

```



最后放一下pom.xml的内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.7</version>
<!--        <version>3.1.1</version>-->
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>vueblog1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>vueblog</name>
    <description>vueblog</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api -->
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.1</version>
        </dependency>
        <!-- JSON工具类 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.6</version>
        </dependency>


        <!--        模板引擎-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>LATEST</version>
        </dependency>
        <!--mybatis-plus代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.5.3.1</version>
<!--            <version>3.2.0</version>-->
        </dependency>

        <!-- shiro-redis -->
        <dependency>
            <groupId>org.crazycake</groupId>
            <artifactId>shiro-redis-spring-boot-starter</artifactId>
<!--            <version>3.3.1</version>-->
            <version>3.2.1</version>
<!--            <classifier>jakarta</classifier>-->
        </dependency>
        <!-- hutool工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
<!--            <version>5.8.16</version>-->
            <version>5.3.3</version>
        </dependency>

        <!-- jwt 生成工具 校验工具-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>






    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```



代码仓库：https://github.com/shuai06/VueBlog-prj













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%9F%BA%E4%BA%8Espringboot%E5%92%8Cvue3%E7%9A%84%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E5%8D%9A%E5%AE%A2/  

