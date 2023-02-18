# Java学习之常用类


### String类

#### 基础知识

**String类: 代表字符串**。Java程序中的所有字符串字面值（如"abc"）都作为此类的实例实现。

String是一个final类，代表<font color=red>不可变的字符序列</font>。

字符串是常量，用双引号引起来表示。它们的值在创建之后不能更改。

String对象的字符内容是存储在一个字符数组value[]中的。



**字符串常量池：**

三种JVM：

- Sun公司的HotSpot（使用的较多）
- BEA公司的JRockit
- IBM公司的J9 VM



**Heap 堆**

—个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存分为三部分︰

```bash
Young Generation Space新生区		Young
Tenure generation space养老区		old
Permanent Space永久存储区		   Perm

```



**事实上永久区是划分在方法区的：**

![JVM优化是哪里](http://image.geoer.cn/JVM%E4%BC%98%E5%8C%96%E6%98%AF%E5%93%AA%E9%87%8C.png)

![代](http://image.geoer.cn/%E4%BB%A3.png)





**永久区：**

​    永久存储区是一个常驻内存区域，用于存放JDK自身所携带的Class,Interface 的元数据，也就是说它存储的是运行环境必须的类信息，**被装载进此区域的数据是不会被垃圾回收器回收掉的，关闭JVM才会释放此区域所占用的内存。**
​    如果出现`java.lang.OutOfMemoryError: PermGen space`，说明是Java虚拟机对永久代Perm内存设置不够。一般出现这种情况，都是程序启动需要加载大量的第三方jar包。例如:在一个Tomcat下部署了太多的应用。或者大量动态反射生成的类不断被加载，最终导致Perm区被占满。
Jdk1.6及之前:常量池分配在永久代, Jdk1.6在方法区

Jdk1.7:有，但已经逐步“去永久代”，1.7在堆

Jdk1.8及之后:无，1.8在元空间

![在不用jdk中区别](http://image.geoer.cn/jdk%E6%B0%B8%E4%B9%85%E5%8C%BA.png)







**String对象的创建：**

```java
string str = "hello";

//本质上this.value = new char[0];
String s1 = new String();

//this.value = original.value;
String s2 = new string(String origina1);

//this.value = Arrays.copyof(value，value.length);
string s3 = new String(char[] a);

String s4 = new String(char[]a,int startIndex,int count);

```



**两种创建方式的区别：**

![两种创建方式的区别](http://image.geoer.cn/string%E4%B8%A4%E7%A7%8D%E5%88%9B%E5%BB%BA%E6%96%B9%E5%BC%8F%E7%9A%84%E5%8C%BA%E5%88%AB.jpg)



**字符串的特性：**



![字符串的特性](http://image.geoer.cn/%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%9A%84%E7%89%B9%E6%80%A7.png)



```java
package cn.xpshuai.java1;

import org.junit.Test;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 10:44
 * @功能：String的使用
 *
*/

public class StringTest {
/**
 * 1.String声明为final，不可被继承
 * 2.String实现了Serializable接口：表示字符串是支持序列化的
 *          实现了Comparable接口：表示可以比较大小
 * 3.String内部定义了final char[] value用于存储字符数据
 * 4.String代表不可变的字符序列
 *   体现：
 *       1.当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
         2.当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的地址的value进行赋值
 *       3.当调用replace()修改字符串或字符时，也需要重新指定内存区域赋值，不能使用原有的地址的value进行赋值
 *5.通过字面量方式(区别于new)给一个字符串赋值，此时的字符串值声明在字符串常量池中,字符串常量池中是不会存储相同内容的字符串的
*/
    @Test
    public void test1(){
        String s1 = "asd";  //字面量
        String s2 = "asd";
        System.out.println(s1 == s2); //不是基本数据类型，比较s1和s2的地址池  true

        s1 = "hello";

        System.out.println(s1 == s2); //比较s1和s2的地址池   false
        System.out.println(s1);
        System.out.println(s2);

        String s3 = "asd";
        s3 += "fgh";
        System.out.println(s3); // 拼接了，还得新造一个，不能直接在原来地址上添加
        System.out.println(s2);

        String s4 = "asd";
        String s5 = s4.replace('a', '6'); //也是新造，不能在原地址区进行任何修改
        System.out.println(s4);
        System.out.println(s5);



    }

    /**
     * String的实例化方式：
     方式1：通过字面量定义的方式
     方式2：通过 new + 构造器的方式


      面试题: string s = new String("abc");方式创建对象，在内存中创建了几个对象?
             答：两个，一个是堆空间中new的，另一个是char[]对象的常量池中的数据"abc"

     */
    @Test
    public void test2(){
        //此时的s1和s2的数据是声明在方法区中的字符串常量池中
        String s1 = "java";
        String s2 = "java";
        //通过 new + 构造器的方式：:此时的s3和s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值。
        String s3 = new String("java");
        String s4 = new String("java");

        System.out.println(s1 == s2); //true
        System.out.println(s1 == s3); //false
        System.out.println(s3 == s4);  //false

        // String中equals()比较的是值，==比较的是地址
        // 通过字面量定义的str会存储在字符串常量池中


    }


    /**
     * 结论：1.常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量
     *      2.只要其中有一个是变量，结果就在堆中
     *      3.如果拼接的结果调用intern()方法，返回值就在常量池中
     */
    @Test
    public void test3() {
        //此时的s1和s2的数据是声明在方法区中的字符串常量池中
        String s1 = "java";
        String s2 = "EE";

        String s3 = "javaEE";  //字面量
        String s4 = "java" + "EE";  //字面量的连接，在常量池中
        String s5 = s1 = "EE";  //只要有变量名参与，就不在常量池而在堆空间中了
        String s6 = "java" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4); // true
        System.out.println(s3 == s5); // false
        System.out.println(s3 == s6);// false
        System.out.println(s3 == s7);// false
        System.out.println(s5 == s6);// false
        System.out.println(s5 == s7);// false
        System.out.println(s6 == s7);// false

//        intern()
        String s8 = s5.intern(); //返回值得到的s8使用的常量池中已经存在的"javaEE"
        System.out.println(s3 == s8); // true

    }


}

```



#### 练习

**练习1：**

![分析](http://image.geoer.cn/string%E7%BB%83%E4%B9%A01.jpg)

**面试题2：**

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 14:38
 * @功能：面试题一道， 涉及String与值传递机制
 */
public class CaseTest {
    String str = new String("good");
    char[] cs = {'t', 'e', 's', 't'};
    public void change(String str, char ch[]){
        // 这里形参是String，是引用型数据，但是String是不可变的，这里形参相当于造了一个变量
        // 但是是无法改变String的，所以还是good
        str = "test ok";
        ch[0] = 'b';

        // ch是数组，是引用型数据，是在堆里的，可以修改，t-->b
    }

    public static void main(String[] args) {
        CaseTest ex = new CaseTest();
        ex.change(ex.str, ex.cs);
        System.out.println(ex.str);  // good
        System.out.println(ex.cs); // best
    }
}

```









#### String常用方法

![String常用方法](http://image.geoer.cn/String%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg)



```java
package cn.xpshuai.java1;

import org.junit.Test;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 14:51
 * @功能：String常用方法
 */
public class StringMethodTest {
    @Test
    public void tes1(){
        String s0 = "我是大帅锅";
        String s1 = "    Hello World  ";
        String s2 = "I am A boy.";
        System.out.println(s1.length());  //长度
        System.out.println(s1.charAt(0));  //取索引上的一个值
        System.out.println(s1.isEmpty());  //是否为空

        String s11 = s1.toLowerCase();
        System.out.println(s11); //小写的字符串
        System.out.println(s1); //s1还不变，仍然为原来的
        System.out.println(s1.toUpperCase()); //转换大写

        String s3 = s1.trim();
        System.out.println(s3);  //去除首尾空格（注册用户写输入框的时候）
        System.out.println(s1.equalsIgnoreCase(s11)); //忽略大小写的情况下比较值是否相等
        System.out.println(s1.equals(s2));

        System.out.println(s1.concat("EFG")); //拼接

        //涉及到字符串排序
        System.out.println(s1.compareTo(s1)); //比较字符串大小(逐个比较ASCII码)，相等为0，负数表示前面的数小

        System.out.println(s0.substring(1)); // 从索引位置开始截取
        System.out.println(s0.substring(1, 3)); // 从索引位置开始截取到end索引位置(左闭右开)

    }

    @Test
    public void tes2() {
        String s0 = "我是大帅锅";
        String s1 = "Hellor World";
        String s2 = "I am A boy.";
        String s3 = "am A boy";

        boolean b1 = s1.endsWith("ld");
        System.out.println(b1);
        boolean b2 = s1.startsWith("H"); //是否以指定字符开始
        System.out.println(b2);

        boolean b3 = s1.startsWith("ll", 2); //从索引位置开始，是否以指定字符开始
        System.out.println(b3);

        System.out.println(s2.contains(s3)); //是否包含s3, true

        //返回指定子串在字符串中第一次出现位置的索引(如果没找到，返回-1)
        System.out.println(s1.indexOf("ll"));
        System.out.println(s1.indexOf("ll", 2)); //从指定索引开始找子串(想找子串出现了几次，找到一次之后从上一次找到的索引+子串长度继续往后找)

        //从后往前找
        System.out.println(s1.lastIndexOf("or"));
        System.out.println(s1.lastIndexOf("or", 6)); //从指定索引反向搜索

        // 什么时候indexOf和lastIndexOf返回索引相同？
        // 1.存在唯一的str
        // 2.不存在这个str


    }

    @Test
    public void tes3() {
        String s0 = "我1是大帅锅";
        String s1 = "Hellor World";
        String s2 = "I am A boy.";
        String s3 = "am A boy";

        //替换
        String s11 = s1.replace("or", "XX"); //替换字符串(所有出现的都要替换)
        String s111 = s1.replace('W', '时'); //替换单个字符
        System.out.println(s11);
        System.out.println(s111);

        String str = "12hello34world5java798mysql465";
        String strr = s1.replaceAll("\\d+", ",");  //根据正则替换
        System.out.println(strr);
//        s0.replaceFirst() //只替换第一次

        //匹配正则表达式  boolean
        System.out.println(str.matches("\\d+"));


        //切片 根据正则
        String[] s00 = s0.split("\\d+");
        for (int i = 0; i <s00.length ; i++) {
            System.out.println(s00[i]);
        }
    }
}

```



#### String与其他类型转换

##### 1.String与基本数据类型、包装类之间的转换

![string与其他类型转换](http://image.geoer.cn/string%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2.jpg)

```java
/**
* 1.String与基本数据类型、包装类之间的转换
* String -->基本数据类型、包装类:调用包装类的静态方法：parseXXX(str)
*
*基本数据类型、包装类 -->String：调用String重载的valueOf(xxx)
*
*
*/
@Test
public void test1(){
    String s1 = "123";   //在常量池里
    int n = Integer.parseInt(s1);

    String s2 = String.valueOf(n);
    String s3 = n + "";  //在堆里

}
```



##### 2.String与char[]之间的转换

```java
    /**
     * 2.String与char[]之间转换
     *String -->char[]:调用String的toCharArray()
     *char[] --> String: 调用String的构造器
     *
     */
    @Test
    public void test2(){
        String s1 = "abc123";   //在常量池里
        char[] c1 = s1.toCharArray(); //
        for (int i = 0; i < c1.length; i++) {
            System.out.println(c1[i]);
        }

        char[] arr = new char[]{'h','e','l','l','o'};
        String s2 = new String(arr);
        System.out.println(s2);
```



##### 3.String与字节数组byte[]之间的转换

![String与字节数组byte[]之间的转换](http://image.geoer.cn/String%E4%B8%8E%E5%AD%97%E8%8A%82%E6%95%B0%E7%BB%84byte%5B%5D%E4%B9%8B%E9%97%B4%E7%9A%84%E8%BD%AC%E6%8D%A2.png)

```java
    /**
     * 3.String与字节数组byte[]之间的转换
     * 编码：String --> byte[]:调用String的getBytes()
     *
     * 解码：byte[] --> String: 调用String的构造器
     *
     *
     */
    @Test
    public void test3() throws UnsupportedEncodingException {
        String s1 = "qwe123中国";
        byte[] bytes = s1.getBytes(); //转换为字节，中文采用的就是默认的字符编码集（汉字在utf-8占3个字节）
        System.out.println(Arrays.toString(bytes)); //这么遍历输出也可以

        byte[] gbks = s1.getBytes("gbk"); //使用指定的字符集进行编码（汉字在gnk占2个字节）
        System.out.println(Arrays.toString(gbks)); //这么遍历输出也可以

        String s2 = new String(bytes);
//        String s2 = new String(gbks);  //如果不用前面相同的字符集解码，会乱码
        System.out.println(s2);


    }
    
    
```



一个加了final的拼接问题：

```java
@Test
public void test4(){
    String s1 = "javaEE";
    String s2 = "java";
    String s3 = s2 + "EE";
    System.out.println(s1 == s3); //false

    final String s4 = "java";   //加了final，就是 s4:常量
    String s5 = s4 + "EE";
    System.out.println(s1 == s5); // true
}

```





#### 常见算法题

![string常见算法题.](http://image.geoer.cn/string%E5%B8%B8%E8%A7%81%E7%AE%97%E6%B3%95%E9%A2%98.jpg)





#### StringBuffer与StringBuilder

三者区别：

```java
package cn.xpshuai.java1;

import org.junit.Test;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 16:22
 * @功能：StringBuffer与StringBuilder
 */
public class StringBufferTest {
    /**
     *
     * String、StringBuffer、StringBuilder有何异同？
     * String: 不可变的字符序列：底层使用char[]存储
     * StringBuffer：可变的字符序列：线程安全的，效率偏低：底层使用char[]存储
     * StringBuilder：可变的字符序列：线程不安全，效率高一些，jdk5.0新增：底层使用char[]存储
     *
     *
     *
     * 为啥第一个不可变，后俩可变呢？
     *
     *
     * 【源码分析：】
     * String s = new String(); //new char[0];
     * String s1 = new String("abc"); // char value = new char[]{'a','b','c'};
     *
     * StringBuffer sb1 = new StringBuffer(); // char value = new char[16]; 底层创建了一个长度16的数组
     * System.out.println(sb1);  //长度为：0
     * sb1.append('a'); //value[0] = 'a';
     *sb1.append('b'); //value[1] = 'b';
     *
     * StringBuffer sb2 = new StringBuffer("aba"); // char value = new char["aba".length() + 16]
     *
     * 问题1：
     * System.out.println(sb2.length());  // 3
     *
     * 问题2：
     * 扩容问题：如果要添加的数据底层数组盛不下了，那就扩容底层的数组
     *          默认情况下，扩容为原来容量的2倍，同时将原有数组中的元素复制到新的数组中
     *
     *          因为String不可变，所以优先用StringBuffer和StringBuilder，再考虑有无多线程问题...
     *
     *
     *
     *
     *
     *
     *
     *
     */

    @Test
    public void test1(){
        StringBuffer sb1 = new StringBuffer("aba");
        sb1.setCharAt(0, 'm');
        System.out.println(sb1);
    }


}

```



##### StringBuffer中常用方法

![StringBuffer中常用方法](http://image.geoer.cn/StringBuffer%E4%B8%AD%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg)



```java
package cn.xpshuai.java1;

import org.junit.Test;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 16:22
 * @功能：StringBuffer与StringBuilder
 */
public class StringBufferTest {
    /**
     *
     * String、StringBuffer、StringBuilder有何异同？
     * String: 不可变的字符序列：底层使用char[]存储
     * StringBuffer：可变的字符序列：线程安全的，效率偏低：底层使用char[]存储
     * StringBuilder：可变的字符序列：线程不安全，效率高一些，jdk5.0新增：底层使用char[]存储
     *
     *
     *
     * 为啥第一个不可变，后俩可变呢？
     *
     *
     * 【源码分析：】
     * String s = new String(); //new char[0];
     * String s1 = new String("abc"); // char value = new char[]{'a','b','c'};
     *
     * StringBuffer sb1 = new StringBuffer(); // char value = new char[16]; 底层创建了一个长度16的数组
     * System.out.println(sb1);  //长度为：0
     * sb1.append('a'); //value[0] = 'a';
     *sb1.append('b'); //value[1] = 'b';
     *
     * StringBuffer sb2 = new StringBuffer("aba"); // char value = new char["aba".length() + 16]
     *
     * 问题1：
     * System.out.println(sb2.length());  // 3
     *
     * 问题2：
     * 扩容问题：如果要添加的数据底层数组盛不下了，那就扩容底层的数组
     *          默认情况下，扩容为原来容量的2倍，同时将原有数组中的元素复制到新的数组中
     *
     *          因为String不可变，所以优先用StringBuffer和StringBuilder，再考虑有无多线程问题...
     *
     *
     *
     *
     *
     *
     *
     *
     */

    @Test
    public void test1(){
        StringBuffer sb1 = new StringBuffer("aba");
        sb1.setCharAt(0, 'm');
        System.out.println(sb1);
    }

    /**
     * StringBuffer中常用方法
     *关注：
     * 增:append(xxx)
     * 删:delete(int start, int end)
     * 改:setCharAt(int n, char cn) / replace(int start, int end, String str)
     * 查: charAt(int n)
     * 插入:insert(int offset, xx)
     * 长度:length()
     * 遍历:for +里面用 charAt()   / toString() 查看内容
     */
    @Test
    public void tets2(){
        StringBuffer sb1 = new StringBuffer("aba");
        sb1.append(1);  //添加数据
        sb1.append("aaa");
        sb1.delete(1,3); //删除索引内的字符（左闭右开）
        sb1.replace(1,3, "hello"); //替换索引内的字符（左闭右开）
        sb1.insert(2, "H"); //在索引位置插入数据，后面的数据后移
        sb1.reverse(); //反转本身
        System.out.println(sb1);
        sb1.indexOf("H");  //返回索引
        String ss = sb1.substring(1,3);  //返回一个从start到end索引结束的左闭右开区间的子串
        sb1.charAt(1); //

        sb1.setCharAt(0, 'm');
        System.out.println(sb1);

    }


    @Test
    public void test3(){
        //三者的效率：由高到低：StringBuilder >  StringBuffer > String
    }

}

```



> StringBuilder中常用方法是类似的，只不过加了同步方法







#### 三者效率

**由高到低：**StringBuilder >  StringBuffer > String



### 时间类

#### JDK8之前日期和之间API

![JDK8之前日期和之间API](http://image.geoer.cn/JDK8%E4%B9%8B%E5%89%8D%E6%97%A5%E6%9C%9F%E5%92%8C%E4%B9%8B%E9%97%B4API.jpg)



```java
package cn.xpshuai.java2;

import org.junit.Test;

import java.util.Date;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-19 17:19
 * @功能： JDK8之前日期和时间的API测试
 */
public class DateTimeAPI {
//    1.System类中的currentTimeMillis()
    @Test
    public void test1() {
        long time = System.currentTimeMillis();
        //返回有计算机元年之间以毫秒为单位的时间差, 时间戳
        System.out.println(time);
    }

    /**
     * java.util.Date类
     *       | --- java.sql.Date类(对应数据库中的日期类型的变量)
     *
     * java.util.Date类：
     * 1.两个构造器的使用
     * Date()
     * Date(1611048522028L);
     * 2.两个常用方法：
     * getTime()：获取当前时间的时间戳
     * toString()：显示当前时间
     *
     *
     * java.sql.Date类
     * 1.如何实例化
     * 2.如果有sql.Date，如何转换为util.Date对象 -->多态赋值就行
     * 3.util.Date，如何转换为sql.Date对象(父类往子类转)
     *
     *
     */
    @Test
    public void test2() {
//        构造器1
        Date date1 = new Date();
        System.out.println(date1.toString()); // Tue Jan 19 17:28:03 CST 2021
        System.out.println(date1.getTime());  //long型是值，毫秒数，时间戳

//        构造器2:创建指定毫秒数的Date对象
        Date date2 = new Date(1611048522028L);
        System.out.println(date2.toString()); // Tue Jan 19 17:28:03 CST 2021

        //创建java.sql.Date对象
        java.sql.Date date3 = new java.sql.Date(1611048522028L);
        System.out.println(date3);

        //util.Date，如何转换为sql.Date对象(父类往子类转)
//        情况1
//        Date date4 = new java.sql.Date(1611048522028L);
//        java.sql.Date date5 = (java.sql.Date)date4;

//        情况2：
        Date date6 = new Date();
        java.sql.Date date7 = new java.sql.Date(date6.getTime());

    }
}

```



**java.text.SimpleDateFormat类**

- Date类的API不易于国际化，大部分被废弃了，`java.text.SimpleDateFormat`类是一个不与语言环境有关的方式来格式化和解析日期的具体类。

- 它允许进行**格式化:日期→文本、解析:文本→日期**

- **格式化:**

  ```java
  simpleDateFormat() :	//默认的模式和语言环境创建对象
  public SimpleDateFormat(String pattern):	//该构造方法可以用参数pattern指定的格式创建一个对象，该对象调用:
  public String format(Date date):	//方法格式化时间对象date
  ```

- **解析:**

  ```java
  public Date parse(String source):	//从给定字符串的开始解析文本，以生成一个日期
  ```





**java.util.Calendar类：**

- Calendar是一个抽象基类，主用用于完成日期字段之间相互操作的功能.

- 获取Calendar实例的方法

- 使用`Calendar.getInstance()`方法
- 调用它的子类`GregorianCalendar`的构造器。

- 一个Calendar的实例是系统时间的抽象表示，通过`get(int field)`方法来取得想要的时间信息。比如YEAR、MONTH、DAY_OF_WEEK、HOUR_OF_DAYMINUTE、SECOND

  ```java
  public void set(int field,int value)
  public void add(int field,int amount)
  public final Date getTime()
  public final void setTime(Date date)
  ```

  

- 注意:

- 获取月份时:一月是0，二月是1，以此类推，12月是11
- 获取星期时:周日是1，周二是2，。。。。周六是7



```java
package cn.xpshuai.java2;

import org.junit.Test;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-23 21:35
 * @功能：
 *
 *
 * 1.system的currentTImeMills()
 * 2. java.util.Date和java.sql.Date
 * 3.java.util.SimpleDateFormat
 * 4.Calendar
 */
public class SimpleDateFormatTest {
    /*
    SimpleDateFormat: 对日期Date类的格式化和解析
    格式化：日期-->字符串
    解析: 字符串 --> 日期

     */
    @Test
    public void testSimpleDateFormat() throws ParseException {
        //实例化：使用默认构造器
        SimpleDateFormat sdf = new SimpleDateFormat();
        //格式化
        Date date = new Date();
        String format_Str = sdf.format(date);
        System.out.println(format_Str);

        //解析
        String str = "2020-12-30 上午8:11";
        Date date1 = sdf.parse(str);    //抛异常
        System.out.println(date1);
        System.out.println(date1.getTime());
        System.out.println("**********");


        //实例化：调用带参构造器(开发中这么指定格式的方式用得多)
        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        String str2 = sdf2.format(date);
        System.out.println(str2);
        //解析: 要求字符串必须符合通过构造器参数体现的这个实例的格式
        System.out.println(sdf2.parse("2021-01-23 09:47:37"));

    }


    /*
    Calendar 日历类
     */
    @Test
    public void testCalendar(){
        //1.实例化
        //方式1: 创建其子类(GregorianCalendar)的对象
        //方式2: 调用它的静态方法: getInstance()  -- 常用
        Calendar calendar = Calendar.getInstance();
        System.out.println(calendar.getClass());

        //2.常用方法
        //get()
        int days = calendar.get(Calendar.DAY_OF_MONTH); //当前时间是这个月的第几天
        System.out.println(days);
        System.out.println(calendar.get(Calendar.DAY_OF_YEAR)); // 当前是这个年的第几天

        //set()
        calendar.set(Calendar.DAY_OF_MONTH, 22); //修改值
        int days2 = calendar.get(Calendar.DAY_OF_MONTH);
        System.out.println(days2); //现在本身已经改了(可变的)

        //add()
        calendar.add(Calendar.DAY_OF_MONTH, 3); //在原基础加上三天
        calendar.add(Calendar.DAY_OF_MONTH, -3); //在原基础减去三天

        //getTime()：日历类-->Date
        Date date = calendar.getTime(); //


        //setTime(): Date --> 日历类
        Date da = new Date();
        calendar.setTime(da);
        calendar.getTime();

    }


}

```



#### JDK8中的日期和之间API

**新API出现背景：**

JDK11之后java.util.DAta被启用了了。而Calendar并不比Date好多少。

它们面临的问题是:

可变性: 像日期和时间这样的类应该是不可变的。

偏移性: Date中的年份是从1900开始的，而月份都从0开始。

格式化: 格式化只对Date有用，Calendar则不行。
此外，它们也不是线程安全的;不能处理闰秒等。

**总结:**对日期和时间的操作一直是Java程序员最痛苦的地方之一。





**新时间日期API：**

- 第三次引入的API是成功的，并且Java8中引入的java.time API已经纠正了过去的缺陷，将来很长一段时间内它都会为我们服务。
- Java 8吸收了Joda-Time 的精华，以一个新的开始为Java创建优秀的API.新的java.time中包含了所有关于**本地日期(LocalDate) 、本地时间(LocalTime) 、本地日期时间(LocalDateTime)、时区 (ZonedDateTime和持续时间(Duration）的类。**历史悠久的Date类新增了tolnstant()方法,用于把 Date转换成新的表示形式。这些新增的本地化时间日期API大大简化了日期时间和本地化的管理。

```java
java.time -包含值对象的基础包
java.time.qhrono-提供对不同的日历系统的访问
java.time.format-格式化和解析时间和日期
java.time.temporal-包括底层框架和扩展特性
java.time.zone-包含时区支持的类
```

- LocalDate、LocalTime、LocalDateTime类是其中较重要的几个类，它们的实例是**不可变的对象**，分别表示使用ISO-8601日历系统的日期、时间、日期和时间。它们提供了简单的本地日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。
- LocalDate代表IOS格式(yyyy-MM-dd）的日期,可以存储生日、纪念日等日期
- LocalTime表示一个时间，而不是日期。
- LocalDateTime是用来表示日期和时间的，这是一个最常用的类之一。







![java新时间API的方法](http://image.geoer.cn/java%E6%96%B0%E6%97%B6%E9%97%B4API%E7%9A%84%E6%96%B9%E6%B3%95.jpg)



```java
    /*
    LocalDate, LocalTime, LocalDateTime(使用较多)
    类似于Calendar
     */
    @Test
    public void test_LocalXXX(){
        //实例化1:
        //now() 获取当前日期或时间
        LocalDate ldate = LocalDate.now();
        LocalTime ltime = LocalTime.now();
        LocalDateTime ldatetime = LocalDateTime.now(); //使用的较多

        System.out.println(ldate);
        System.out.println(ltime);
        System.out.println(ldatetime);

        //实例化2: of() 设置指定时间, 参数没有偏移量
        LocalDateTime nowtime = LocalDateTime.of(2020,10,10,8,8,8);
        System.out.println(nowtime);
        System.out.println("*******");


        //getXXX() 获取
        System.out.println(ldatetime.getDayOfMonth()); // 获取时间是本月的第几天
        System.out.println(ldatetime.getDayOfWeek());
        System.out.println(ldatetime.getDayOfYear());
        System.out.println(ldatetime.getMonthValue()); //获取月份
        System.out.println(ldatetime.getMinute());

        //withXXX() 设置
        // 这种设置是不可变的
        LocalDate localDate1 = ldate.withDayOfMonth(10); //修改设置值，修改当前是月的第几天
        System.out.println(localDate1); //它改了
        System.out.println(ldate); //原来的不受影响

        LocalDateTime ll = ldatetime.withHour(5);

        //加
        LocalDateTime lll = ldatetime.plusMonths(3); //在原来基础上加上了三个月
        System.out.println(lll); //加了之后的
        System.out.println(ldatetime); //原来的还是不变的

        //减
        LocalDateTime llll = ldatetime.minusDays(6); //减去6天
        System.out.println(llll);
    }

```





**Instant(瞬时)：**

![Instant](http://image.geoer.cn/java%E6%97%B6%E9%97%B4-instant.jpg)



```java
    Instant只是简单的表示自元年开始的秒
    类似java.util.Date
     */
    @Test
    public void testInstant(){
        //now()  获取本初子午线对应的标准时间
        Instant instant = Instant.now();
        System.out.println(instant); //默认时间是不对的，按UTC时间算是在本初子午线位置的时间，咱在东八区需要加一下

        // 我们采用这种方式，添加时间的偏移量
        OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8)); //东八区
        System.out.println(offsetDateTime);

        //获取毫秒数  --> Date.getTime()
        long mi = instant.toEpochMilli();
        System.out.println(mi);

        //
        //通过给定的毫秒数，获取Instant实例 --> Date(Long millis)
        Instant instant2 = Instant.ofEpochMilli(1611413025604L);
        System.out.println(instant2);

    }


```





**格式化与解析日期或时间：**

![格式化与解析日期或时间](http://image.geoer.cn/%E6%A0%BC%E5%BC%8F%E5%8C%96%E4%B8%8E%E8%A7%A3%E6%9E%90%E6%97%A5%E6%9C%9F%E6%88%96%E6%97%B6%E9%97%B4.jpg)



```java
    /*
    DateTimeFormatter: 格式化或解析日期或时间
    类似于SimpleDateFormat
     */
    @Test
    public void testDateTimeFormatter(){
//        LocalDateTime, LocalDate, LocalTime
        //实例化1：预定义的标准格式
        DateTimeFormatter formatter1 = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        //格式化:日期-->str
        LocalDateTime localDateTime = LocalDateTime.now();
        String str1 = formatter1.format(localDateTime);
        System.out.println(str1);
        //解析: str-->日期
        TemporalAccessor parse = formatter1.parse("2021-01-23T22:51:32.392");


        //实例化2：本地化相关的格式(以某个函数或参数举例)
//        ofLocalizedDateTime,ofLocalizedDate
        DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
        //格式化
        String str2 = formatter2.format(localDateTime);

//        ofLocalizedDate
        DateTimeFormatter formatter3 = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL);
        String str3 = formatter3.format(LocalDate.now());
        System.out.println(str3);

        //解析...

        //实例化3：自定义的格式(使用的最多 -->重点)
        DateTimeFormatter format1 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
        //格式化
        String s1 = format1.format(LocalDateTime.now());
        System.out.println(s1);
        //解析
        TemporalAccessor accessor = format1.parse("2020-8-8 08:08:08");
        System.out.println(accessor);
    }

```



**java其他时间api：**

![java其他时间api](http://image.geoer.cn/java%E5%85%B6%E4%BB%96%E6%97%B6%E9%97%B4api.jpg)



**与传统日期的处理转换：**

![与传统日期的处理转换](http://image.geoer.cn/%E4%B8%8E%E4%BC%A0%E7%BB%9F%E6%97%A5%E6%9C%9F%E7%9A%84%E5%A4%84%E7%90%86%E8%BD%AC%E6%8D%A2.jpg)



> 用到的时候查一下就行





### Java比较器(重要)

> 涉及到对象排序，就要用下面两种，至于用哪种就自己选吧
>
> java.lang.Comparable 一劳永逸，一旦实现，在任何位置都可以比较大小
>
> java.util.Comparator   临时用一下时候的比较



![自然排序1](http://image.geoer.cn/%E8%87%AA%E7%84%B6%E6%8E%92%E5%BA%8F1.jpg)



#### Comparable

```java
package cn.xpshuai.java3;

import org.junit.Test;

import java.util.Arrays;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 17:07
 * @功能： Java比较器
 *
 * Java中的对象，正常情况下只能进行比较；==, !=, 不能使用: >   <
 * 但是在开发场景中，我们需要对多个对象进行排序可以比较大小
 *实现对象排序的方式有两种：
 * - 自然排序: java.lang.Comparable
 * - 定制排序: java.util.Comparator
 *
 *
 * 1.Comparable接口
 *
 *
 * 2.Comparator接口
 */
public class ComparableTest {

    /*
    Comparable使用举例
    1.String等包装类实现了Comparable接口，重写了compareTo()方法,给出了比较两个对象大小的实现
    2.String等包装类重写compareTo()以后，进行了从小到大排列
    3.实现compareTo()的规则:
        如果当前对象this大于形参对象，则返回正整数；
        如果当前对象this小于形参对象，则返回负整数；
        如果当前对象this等于形参对象obj，则返回0。
     */
    @Test
    public void test1(){
        String[] arr = new String[]{"DD", "GG", "BB", "JJ","MM","AA"};
        Arrays.sort(arr); //排序(其实也是String实现了Comparable接口)

    }


}
 
```



#### 自定义类实现Comparable自然排序

自定义类：

```java
package cn.xpshuai.java3;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 17:26
 * @功能：
 */
public class Goods implements Comparable{
    private String name;
    private double price;

    public Goods() {
    }

    public Goods(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Goods{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    //实现这个方法
    //指明商品比较大小的方式(这里自定义比较价格从低到高排序，如果需要，可以继续嵌套比较)
    @Override
    public int compareTo(Object o) {
        if(o instanceof  Goods){
            Goods goods = (Goods)o;
            if(this.price > goods.price){
                return 1;
            }else if(this.price < goods.price){
                return -1;
            }else {
//                return 0;
                 return this.name.compareTo(goods.name); //如果需要，可以继续嵌套比较, 比如价格一样就再比较商品名称
            }
        }
        throw new RuntimeException("传入的数据类型不一致!");
    }
}

```

```java
package cn.xpshuai.java3;

import org.junit.Test;

import java.util.Arrays;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 17:07
 * @功能： Java比较器
 *
 * Java中的对象，正常情况下只能进行比较；==, !=, 不能使用: >   <
 * 但是在开发场景中，我们需要对多个对象进行排序可以比较大小
 *实现对象排序的方式有两种：
 * - 自然排序: java.lang.Comparable
 * - 定制排序: java.util.Comparator
 *
 *
 * 1.Comparable接口
 *
 *
 * 2.Comparator接口
 */
public class ComparableTest {

    /*
    Comparable(自然排序)使用举例
    1.String等包装类实现了Comparable接口，重写了compareTo()方法,给出了比较两个对象大小的实现
    2.String等包装类重写compareTo()以后，进行了从小到大排列
    3.实现compareTo()的规则:
        如果当前对象this大于形参对象，则返回正整数；
        如果当前对象this小于形参对象，则返回负整数；
        如果当前对象this等于形参对象obj，则返回0。
     4.对于自定义类，如果需要排序，可以让自定义类实现Comparable接口，重写compareTo()方法
     在compareTo(obj)写如何排序
     */
    @Test
    public void test1(){
        String[] arr = new String[]{"DD", "GG", "BB", "JJ","MM","AA"};
        Arrays.sort(arr); //排序(其实也是String实现了Comparable接口)

    }

    /*
    【自定义类实现Comparable自然排序】
    这里自定义类是Goods.java

     对于自定义类，如果需要排序，可以让自定义类实现Comparable接口，重写compareTo()方法
     在compareTo(obj)写如何排序
     */
    @Test
    public void test2(){
        Goods[] arr = new Goods[4];
        arr[0] = new Goods("联想", 65);
        arr[1] = new Goods("小米", 12);
        arr[2] = new Goods("华为", 65);
        arr[3] = new Goods("罗技", 101);

        Arrays.sort(arr);  //原始情况下，不能执行，需要在自定义类中实现并重写
        System.out.println(Arrays.toString(arr));  // 继承并重写之后

    }

}

```





#### Comparator接口

![定制排序2](http://image.geoer.cn/%E5%AE%9A%E5%88%B6%E6%8E%92%E5%BA%8F2.jpg)



```java
    /*
    Comparator接口(定制排序)
    1.背景:
    当元素的类型没有实现java.Lang.ComparabLe接口而又不方便修改代码,
    或者实现了java.Lang.comparable接口的排序规则不适合当前的操作,
    那么可以考怎使用Comparator的对象来排序
    2.重写compare(Object o1,0bject o2)方法，比较o1和o2的大小:
        如果方法返回正整数,则表示o1大于o2
        如果方法返回负整数,则表示o1小于o2
        如果方法返回0,则表示o1等于o2

     */
    @Test
    public void test3() {
        String[] arr = new String[]{"DD", "GG", "BB", "JJ","MM","AA"};
        Arrays.sort(arr, new Comparator<String>() { //使用匿名内部类
            @Override
            public int compare(String o1, String o2) { //实现compare方法
                //指明o1和o2怎么排
                if(o1 instanceof String && o2 instanceof String){
                    String s1 = (String)o1;
                    String s2 = (String)o2;

                    return -s1.compareTo(s2); //从大到小来排列
                }

                throw new RuntimeException("输入的类型不一致");
            }
        });
    }
```



#### 自定义类实现Comparator定制排序

自定义类:

```java
// Goods类，同前面
```



```java
    /*
    【自定义类实现Comparator定制排序】
    这里自定义类是Goods.java

     */
    @Test
    public void test4(){
        Goods[] arr = new Goods[4];
        arr[0] = new Goods("联想", 65);
        arr[1] = new Goods("小米", 12);
        arr[2] = new Goods("华为", 65);
        arr[3] = new Goods("罗技", 101);
        arr[3] = new Goods("罗技", 100);

        Arrays.sort(arr, new Comparator<Goods>() {
            //指明商品比较大小的方式(这里自定义, 先按产品名称从低到高，再比较价格从高到低排序)
            @Override
            public int compare(Goods o1, Goods o2) {
                if(o1 instanceof  Goods && o2 instanceof  Goods){
                    Goods goods1 = (Goods)o1;
                    Goods goods2 = (Goods)o2;
                    if(goods1.getName().equals(goods2.getName())){
                        return -Double.compare(goods1.getPrice(), goods2.getPrice()); //如果名称一样，就再比较价格
                    }else {
                        return goods1.getName().compareTo(goods2.getName());
                    }
                }
                throw new RuntimeException("传入的数据类型不一致!");
            }
        });  //原始情况下，不能执行，需要在自定义类中实现并重写
        System.out.println(Arrays.toString(arr));  // 继承并重写之后

    }


```





### System类

![System类](http://image.geoer.cn/javaSystem%E7%B1%BB.jpg)















### Math类

![Math类](http://image.geoer.cn/javaMath%E7%B1%BB.jpg)









### BigInteger与BigDecimal

![BigInteger与BigDecimal](http://image.geoer.cn/javaBigInteger%E4%B8%8EBigDecimal.jpg)















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E5%B8%B8%E7%94%A8%E7%B1%BB/  

