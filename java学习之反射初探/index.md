# Java学习之反射初探


### Java反射机制概述

- Reflection（反射）是被视为**动态语言**的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。
- 加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。**这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为: 反射。**



![Java反射概述](http://image.xpshuai.cn/Java%E5%8F%8D%E5%B0%84%E6%A6%82%E8%BF%B0.png)



![动态vs静态语言](http://image.xpshuai.cn/%E5%8A%A8%E6%80%81vs%E9%9D%99%E6%80%81%E8%AF%AD%E8%A8%80.png)

有了反射，就有了Java动态的特性





**Java反射机制提供的功能：**

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解
- 生成动态代理
  
  



**反射相关的主要API；**

- java.lang.Class:代表一个类
- java.lang.reflect.Method:代表类的方法
- java.lang.reflect.Field:代表类的成员变量
- java.lang.reflect.Constructor:代表类的构造器
- ......



反射代码：

```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-04 22:38
 * @功能：
 *
 * 总结：公开的代码都不安全
 *
 *疑问： 通过直接new对象的方式或反射的方式都可疑调用公共的结构，开发中到底用哪个？
 * 建议用new的方法
 * 什么时候会使用反射的方式。反射的特征:动态性
 *
 * 疑问：反射机制与面向对象中的封装性是不是矛盾的呢？
 *不矛盾，封装性是建议你调不调；反射解决的是能不能调
 *
 *
 */
public class ReflectionTest {
    //反射之前，对于Person类的操作
    @Test
    public void test(){
        //1.实例化类对象
        Person p1 = new Person("tom", 12);
        //2.可以通过对象，调用内部属性、方法
        p1.age = 10;
        System.out.println(p1);
        p1.setAge(11);
        System.out.println(p1);
        p1.show();

        //在 Person类外部，不可以通过Person类的对象调用其内部私有结构(比如私有属性、私有方法、私有构造器等)。


    }


    /*
    反射之后，对于Person的操作
     */
    @Test
    public void test1() throws Exception {
        Class clazz = Person.class;
        //1.通过反射创建了Person类的对象
        Constructor cons = clazz.getConstructor(String.class, int.class);
        Object obj = cons.newInstance("Tom", 12); //造了个对象, 多态的形式
        Person p = (Person)obj; //知道类型的话可以强转
        System.out.println(p.toString()); //


        //2.通过反射，调用对象指定的属性、方法
        //调属性
        Field age = clazz.getDeclaredField("age");
        age.set(p, 18);
        System.out.println(p.toString());
        //调用方法
        Method show = clazz.getDeclaredMethod("show");
        show.invoke(p);  //调用

        System.out.println("*******************");
        //通过反射，可以调用Person类(运行时类)的私有结构（私有方法、私有属性、私有构造器）
        //调用私有构造器
        Constructor cons1 = clazz.getDeclaredConstructor(String.class);
        cons1.setAccessible(true);
        Person p1 = (Person) cons1.newInstance("Jerry");
        System.out.println(p1);

        //调用私有属性
        Field name = clazz.getDeclaredField("name");
        name.setAccessible(true);
        name.set(p1, "hanmeimei");
        System.out.println(p1);

        //调用私有方法
        Method na = clazz.getDeclaredMethod("showNation", String.class); //方法名，咱叔
        na.setAccessible(true);
//        na.invoke(p1, "China"); //调用
        String nation = (String) na.invoke(p1, "China"); //接收返回值。调用, 相当于p1.show("China")
        System.out.println(nation);

    }


    @Test
    public void test2(){

    }


    @Test
    public void test3(){

    }
}


```

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-04 22:38
 * @功能：
 */
public class Person {
    private String name;
    public int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    private Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void show(){
        System.out.println("show方法");
    }

    private String showNation(String nation){
        System.out.println("我的国籍是：" + nation);
        return nation;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```





### 理解Class类并<font color=red>获取Class实例</font>

#### 对Class类的理解

```java
 * 关于java.lang.Class的理解：
 *  Class clazz = Person.class;  -->反射的源头
 *  1.类的加载过程
 *  程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)，
 *  接着我们使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中，此过程就成为类的加载。
 *  加载到内存中的类，我们就称为运行时类，此运行时类，就作为Class的一个实例
 *  （类本身也是对象，是Class的对象 ）
 *  换句话说，Class的实例就对应着一个运行时类
 *  加载搭配内存中的运行时类，会缓存一定的时间，在此时间之内，我们可以通过不同的方式
    来获取此运行时类
 * 
 *
 *
 *
 * --- 万事万物皆对象：对象.xxx, File, URK, 反射, 前端, 数据库操作
```



#### 获取Class实例的4种方式

```java
    /*
     * 关于java.lang.Class的理解：
     *  Class clazz = Person.class;  -->反射的源头
     *  1.类的加载过程
     *  程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)，
     *  接着我们使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中，此过程就成为类的加载。
     *  加载到内存中的类，我们就称为运行时类，此运行时类，就作为Class的一个实例
     *  （类本身也是对象，是Class的对象 ）
     *  换句话说，Class的实例就对应着一个运行时类
     * 加载搭配内存中的运行时类，会缓存一定的时间，在此时间之内，我们可以通过不同的方式
     * 来获取此运行时类
     *
     * --- 万事万物皆对象：对象.xxx, File, URK, 反射, 前端, 数据库操作
     *
     *
     * 获取Class实例的4种方式，如下（前三种方式需要掌握）
     */
    @Test
    public void test2() throws ClassNotFoundException {
        //方式1：调用运行时类的属性: .class
        Class<Person> clazz = Person.class;   //加上泛型
        System.out.println(clazz);

        //方式2：通过运行时类的对象.调用getClass()
        Person p1 = new Person();
        Class clazz2 = p1.getClass();

        //方式3(使用最多)：调用Class的静态方法: forName(String classPath)
        Class clazz3 = Class.forName("cn.xpshuai.java1.Person");

        System.out.println(clazz == clazz2);  // true
        
        
        //方式4(使用较少)：使用类的加载器：ClassLoader
        ClassLoader classLoader = ReflectionTest.class.getClassLoader();
        Class clazz4 = classLoader.loadClass("cn.xpshuai.java1.Person");


    }

```





#### 哪些类型可以有class对象?

1. class: 
   外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类
2.  interface: 接口
3. []: 数组
4. enum: 枚举
5. annotation: 注解@interface
6. primitive type:基本数据类型
7. void



**Class实例可以是哪些结构的说明：**

![Class实例可以是哪些结构](http://image.xpshuai.cn/Class%E5%AE%9E%E4%BE%8B%E5%8F%AF%E4%BB%A5%E6%98%AF%E5%93%AA%E4%BA%9B%E7%BB%93%E6%9E%84.png)











### 类的加载与ClassLoader的理解

#### **类的加载过程：**



![类的加载过程](http://image.xpshuai.cn/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B.jpg)



例子：

![类的加载过程例子](http://image.xpshuai.cn/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B%E4%BE%8B%E5%AD%90.png)



#### **类的加载器：**

![类的加载器](http://image.xpshuai.cn/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E5%99%A8.png)



![ClassLoader](http://image.xpshuai.cn/ClassLoader.png)









#### 使用ClassLoader读取配置文件

```java
    /*
    Properties: 用来读取配置文件。
     */
    @Test
    public void test4() throws IOException {
        Properties pros = new Properties();
        //此时的文件默认在当前的module下
        //读取配置文件的方式1：
//        FileInputStream fis = new FileInputStream("jdbc.properties");
//        FileInputStream fis = new FileInputStream("src\\dbc.properties");
//        pros.load(fis); //加载输入流

        //读取配置文件的方式2（常用）：
        //此时的文件默认在前的module下的src目录下
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        InputStream is = classLoader.getResourceAsStream("jdbc.properties");
        pros.load(is);

        String user = pros.getProperty("user");
        String pass = pros.getProperty("pass");
        System.out.println(user + pass);



    }

```











### <font color=red>创建运行时类的对象</font>（重点）



**通过反射，创建对应的运行时类的对象:**

```java
@Test
public void test() throws IllegalAccessException, InstantiationException, ClassNotFoundException {
    Class clazz = Class.forName("cn.xpshuai.java1.Person");

    // newInstance(): 创建对应的运行时类的对象, 内部调用了运行时类的空参构造器
    //要想此方法正常的创建运行时类的对象，要求：
    //1.运行时类必须提供空参的构造器
    //2.空参的构造器的访问权限得够，通常设置为public

    //在javabean中要求提供一个public的空参构造器，原因：
    //1.便于通过反射，创建运行时类的对象
    //2.便于子类继承此运行时类时，默认调用super()时，保证父类有此构造器
    Object obj = clazz.newInstance();
    System.out.println(obj);



}
```





**举例体会反射的动态性：**

```java
    //体会反射的动态性
    @Test
    public void test1() throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        int num = new Random().nextInt(3); // 0,1,2
        String classPath = "";
        switch (num){
            case 0:
                classPath = "java.util.Date";
                break;
            case 1:
                classPath = "java.lang.Object";
                break;

            case 2:
                classPath = "cn.xpshuai.java1.Person";
                break;
        }

        Object obj = getInstance(classPath);
        System.out.println(obj);


    }

    //此方法创建一个指定类的对象，classPath: 指定类的全类名
    public Object getInstance(String classPath) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Class clazz = Class.forName(classPath);
        return clazz.newInstance();

    }
```









### 获取运行时类的完整结构(了解即可)

先创建一下丰富的Person类的结构(java2包里面)：

Person.java

```java
package cn.xpshuai.java2;

import java.io.IOException;

/**
提供结构更丰富的Person类
 */

@MyAnnotation(value = "Hi")
public class Person extends Creature<String> implements Comparable<String>,MyInterface{
    private String name;
    int age;
    public int id;

    @MyAnnotation(value = "abc")
    public Person() {
        System.out.println("Person的空参构造器");
    }
    private Person(String name) {
        this.name = name;
    }
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String display(String interests, int age) throws IOException {
        return interests + age;
    }

    @MyAnnotation
    private void show(String nation){
        System.out.println("show方法,我的国籍是：" + nation);
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public void info() {
        System.out.println("info方法，我是一个人");

    }

    @Override
    public int compareTo(String o) {
        return 0;
    }
}

```

Creature.java

```java
package cn.xpshuai.java2;

import java.io.Serializable;


public class Creature<T> implements Serializable {
    private char gender;
    public double weight;

    public void breath(){
        System.out.println("生物呼吸");
    }

    public void eat(){
        System.out.println("生物吃东西");
    }

}

```

MyInterface.java

```java
package cn.xpshuai.java2;


public interface MyInterface {
    void info();
}

```

MyAnnotation.java

```java
package cn.xpshuai.java2;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;


@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value() default "hello";

}

```



测试代码(java3包里面)：

```java
package cn.xpshuai.java3;

import cn.xpshuai.java2.Person;
import org.junit.Test;

import java.lang.reflect.Field;
import java.lang.reflect.Modifier;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-05 15:28
 * @功能：获取当前运行时类的属性结构
 */
public class FieldTest {
    @Test
    public void test(){
        Class clazz = Person.class;

        //获取属性结构
        //getFields(): 获取当前运行时类及其父类中声明为public访问权限的属性
        Field[] fields = clazz.getFields();
        for(Field f: fields){
            System.out.println(f);
        }

        //获取声明的属性结构
        //getDeclaredFields(): 获取当前运行时类当中声明的所有属性(与权限无关)，不包含父类中的属性
        Field[] declaredFields = clazz.getDeclaredFields();
        for(Field f: declaredFields){
            System.out.println(f);
        }

        System.out.println("******************");
    }

    //权限修饰符  数据类型  变量名 = ...
    @Test
    public void test1(){
        Class clazz = Person.class;
        Field[] declaredFields = clazz.getDeclaredFields();
        for(Field f: declaredFields){
            //1.权限修饰符
            //返回的是数字(在Modifier类中有)
            int modifiers = f.getModifiers();
//            System.out.println(modifiers);
            System.out.print(Modifier.toString(modifiers) + "\t"); //翻译回来

            //2.数据类型
            Class<?> types = f.getType();
            System.out.print(types.getName() + "\t");

            //3.变量名
            String fname = f.getName();
            System.out.print(fname);

            System.out.println();

        }

    }


}

```

```java
package cn.xpshuai.java3;

import cn.xpshuai.java2.Person;
import org.junit.Test;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-05 15:43
 * @功能：获取当前运行时类的方法结构
 
 框架 = 注解+反射+涉及模式
 */
public class MethodTest {
    @Test
    public void test(){
        Class<Person> clazz = Person.class;

        //获取方法结构
        //getMethods():获取当前运行时类及其父类中声明为public访问权限的方法
        Method[] methods = clazz.getMethods();
        for(Method m: methods){
            System.out.println(m);
        }

        //getDeclaredMethods():获取当前运行时类当中声明的所有方法(与权限无关)，不包含父类中的方法
        Method[] dmethods = clazz.getDeclaredMethods();
        for(Method m: dmethods){
            System.out.println(m);
        }




    }

    /*
    获取运行时类的内部结构

    @Xxx
    权限修饰符  返回值类型  方法名(参数类型1 形参名1, ...) throws XxxException{}
     */
    @Test
    public void test1(){

        Class clazz = Person.class;
        Method[] dmethods = clazz.getDeclaredMethods();
        for(Method m: dmethods){
            //1.获取方法声明的注解
            Annotation[] annotations = m.getAnnotations();
            for(Annotation a: annotations){
                System.out.println(a);  // @cn.xpshuai.java2.MyAnnotation(value=hello)
            }

            //2.权限修饰符
            System.out.print(Modifier.toString(m.getModifiers()) + "\t");

            //3.返回值类型
            System.out.print(m.getReturnType().getName() + "\t");

            //4.方法名
            System.out.print(m.getName());
            System.out.print("(");

            //5.形参列表
            Class[] parameterTypes = m.getParameterTypes();
            if(!(parameterTypes == null || parameterTypes.length == 0)){
                for(int i=0; i< parameterTypes.length; i++){
                    if(i == parameterTypes.length-1){
                        System.out.print(parameterTypes[i].getName());
                        break;
                    }
                    System.out.print(parameterTypes[i].getName() +  ", ");

                }
            }
            System.out.print(")");

            //6.抛出的异常
            Class[] exceptionTypes = m.getExceptionTypes();
            if(!(exceptionTypes == null || exceptionTypes.length == 0)){
                System.out.println(" throws ");
                for (int i = 0; i < exceptionTypes.length; i++) {
                    if(i == parameterTypes.length-1){
                        System.out.println(exceptionTypes[i].getName());
                    }
                        System.out.println(exceptionTypes[i].getName() + ", ");
                }
            }

            System.out.println();

        }



    }

    @Test
    public void test2(){

    }



}

```

```java
package cn.xpshuai.java3;

import cn.xpshuai.java2.Person;
import org.junit.Test;

import java.lang.annotation.Annotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-05 16:13
 * @功能：
 */
public class OtherTest {
    /*
    获取构造器结构
     */
    @Test
    public void test(){
        Class clazz = Person.class;

        // getConstructors(): 当前运行时类中声明为public的构造器
        Constructor[] constructors = clazz.getConstructors();
        for(Constructor c: constructors){
            System.out.println(c);
        }

        System.out.println("*************");

        // getDeclaredConstructors(): 当前运行时类中的所有的构造器
        Constructor[] declaredConstructors = clazz.getDeclaredConstructors();
        for(Constructor c: declaredConstructors){
            System.out.println(c);
        }

    }


    /*
    比较重要
    获取运行时类的父类及父类的泛型
    代码：逻辑性代码(可以平时搜集保留,到时候复制粘贴) vs 功能性代码
     */
    @Test
    public void test1(){
        Class clazz = Person.class;

        // getSuperclass(): 获取运行时类的父类
        Class superclass = clazz.getSuperclass();
        System.out.println(superclass);  // class cn.xpshuai.java2.Creature

        // getGenericSuperclass(): 获取运行时类的带泛型的父类
        Type genericSuperclass = clazz.getGenericSuperclass();
        System.out.println(genericSuperclass);  // cn.xpshuai.java2.Creature<java.lang.String>

        // 基于getGenericSuperclass(), 强转然后...: 获取运行时类的带泛型的父类的泛型
        Type genericSuperclass2 = clazz.getGenericSuperclass();
        //做强转
        ParameterizedType paramType = (ParameterizedType)genericSuperclass2;
        //获取泛型类型
        Type[] actualTypeArguments = paramType.getActualTypeArguments();
        System.out.println(actualTypeArguments[0]); // 形式是这样的   cn.xpshuai.java2.Creature<java.lang.String>
        System.out.println(actualTypeArguments[0].getTypeName()); //这里就一个，就没for循环. 可以getTypeName()得到 java.lang.String形式
        System.out.println(((Class)actualTypeArguments[0]).getName()); // 也可以强转为Class然后getName(),    java.lang.String


    }


    /*
    比较重要（动态代理中会用）
    获取运行时类实现的接口
     */
    @Test
    public void test2(){
        Class clazz = Person.class;

        // 获取运行时类实现的接口
        Class[] interfaces = clazz.getInterfaces();
        for(Class c: interfaces){
            System.out.println(c);
        }

        System.out.println("********");

        // 获取运行时类实现的接口
        Class[] interfaces1 = clazz.getSuperclass().getInterfaces();
        for(Class c: interfaces1){
            System.out.println(c);
        }

    }


    /*
    获取运行时类所在的包
     */
    @Test
    public void test3(){
        Class clazz = Person.class;

        Package aPackage = clazz.getPackage();
        System.out.println(aPackage);

    }

    
    /*
    比较重要
    获取运行时类声明的注解
    后面，主要是根据注解获取到注解的内容，判断你要干什么
 */
    @Test
    public void test4(){
        Class clazz = Person.class;

        // 获取运行时类声明的注解
        Annotation[] annotations = clazz.getAnnotations();
        for(Annotation a: annotations){
            System.out.println(a);  // @cn.xpshuai.java2.MyAnnotation(value=Hi)
        }

    }

    @Test
    public void test5(){


    }
}

```









### <font color=red>调用运行时类的指定结构</font>(重要，掌握)

> 主要是调 方法、其次是属性和构造器



```java
package cn.xpshuai.java3;

import cn.xpshuai.java2.Person;
import org.junit.Test;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-06 9:37
 * @功能：

调用运行时类的指定结构(重要，掌握)
> 主要是调方法、其次是属性和构造器

 */
public class ReflectionTest {

    /*
    这个不需要掌握
     */
    @Test
    public void test() throws NoSuchMethodException, NoSuchFieldException, IllegalAccessException, InstantiationException {
        Class clazz = Person.class;
        System.out.println(clazz);
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();
        System.out.println(p);

        //获取指定的属性(通常不采用此方法，因为非public的无法获取)
        Field id = clazz.getField("id");

        //设置当前属性的值
//        set():参数1->指明设置哪个对象的属性   参数2->将此属性值设置为多少
        id.set(p, 1001);

        //获取当前属性的值
//        get(): 参数1-->获取哪个对象的当前属性
        int pid = (int)id.get(p);
        System.out.println(pid);
    }

    /*
需要掌握
标准的，操作运行时类中的指定的属性
 */
    @Test
    public void test1() throws Exception{
        Class clazz = Person.class;
        System.out.println(clazz);
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();


        //这个常用！！！
        //1.getDeclaredField(String fieldName):获取指定变量名的属性
        Field name = clazz.getDeclaredField("name");
        //2.保证当前属性可访问
        name.setAccessible(true);  //权限不够的，设置允许方法
        //3.获取、设置指定对象的属性值
        name.set(p, "Jerry");

        System.out.println(name.get(p));
        System.out.println(p);

    }


    /*
需要掌握
标准的，操作运行时类中的指定的方法
*/
    @Test
    public void test2() throws Exception{
        Class clazz = Person.class;
        System.out.println(clazz);
        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();


        //这个常用！！！
        //1.getDeclaredMethod(String methodName):获取指定变量名的属性
        //参数1：指明获取的方法的名称。 参数2：指明获取的方法的形参列表
        Method show = clazz.getDeclaredMethod("show", String.class);

        //2.保证当前方法可访问
        show.setAccessible(true);  //权限不够的，设置允许方法

        //3.调用invoke()执行: 参数1：方法的调用者，  参数2：给方法形参赋值的实参
        //invoke()的返回值即为对应类中调用的方法的返回值
//        show.invoke(p, "CHN");
        Object returnVal = show.invoke(p, "CHN");
        System.out.println(returnVal);



        System.out.println("**************");
        //调用静态方法
//        private static void showDesc()
        Method sd = clazz.getDeclaredMethod("showDesc");
        sd.setAccessible(true);
        //调用，第一个参数写当前class或null
        sd.invoke(Person.class);  //如果该运行时类的方法没有返回值，则此invoke的返回值为null
//        sd.invoke(null); //写null也行，丝毫不影响

    }

        /*
操作运行时类中的指定的构造器（不太常用，因为常用newInstance()来实例化）
*/

    @Test
    public void test3() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Class clazz = Person.class;

        //1.获取指定的构造器（参数：只需指明参数列表）
        Constructor constructor = clazz.getDeclaredConstructor(String.class);

        //2.保证此构造器可访问
        constructor.setAccessible(true);

        //3.创建运行时类的对象
        Person per = (Person)constructor.newInstance("Tom");
        System.out.println(per);

    }
}

```



















### 反射的应用: 动态代理

**代理设计模式的原理:**
使用一个代理将对象包装起来,然后用该代理对象取代康始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。

之前为大家讲解过代理机制的操作，属于静态代理，特征是代理类和目标对象的类都是在编译期间确定下来，不利于程序的扩展。同时，每一个代理类只能为一个接口服务，这样一来程序开发中必然产生过多的代理。**最好可以通过一个代理类完成全部的代理功能**。



**动态代理**是指客户通过代理类来调用其它对象的方法，并且是在程序运行时根据需要动态创建目标类的代理对象。

**动态代理使用场合:**

- 调试
- 远程方法调用



**动态代理相比于静态代理的优点:**
抽象角色中（接口）声明的所有方法都被转移到调用处理器一个集中的方法中处理，这样，我们可以更加灵活和统一的处理众多的方法。



先看一下之前学过的静态代理：

```java
package cn.xpshuai.java4;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-06 13:21
 * @功能：静态代理的例子
 *
 * 特点：代理类和被代理类在编译期间，就确定了下来
 */

interface ClothFactory{
    void produceCloth();

}


//代理类
class ProxyClothFactory implements ClothFactory{
    private ClothFactory factory; // 就用被代理类对象进行实例化

    public ProxyClothFactory(ClothFactory factory){
        this.factory = factory;
    }

    @Override
    public void produceCloth() {
        System.out.println("代理工厂做一些准备工作...");
        factory.produceCloth();
        System.out.println("代理工厂做一些后续的收尾工作...");
    }
}

//被代理类
class NikeClothFactory implements ClothFactory{
    @Override
    public void produceCloth() {

        System.out.println("Nike工厂生产运动服");

    }
}


//测试
public class StaticProxyTest {
    public static void main(String[] args) {
        //创建被代理类的对象
        ClothFactory nike = new NikeClothFactory();   //多态的形式
        //创建代理类的对象
        ProxyClothFactory proxyClothFactory = new ProxyClothFactory(nike);
        //代理类的对象调方法
        proxyClothFactory.produceCloth();


    }
}

```



动态代理的举例(要求，明白这个逻辑就行，不需要死记代码)：

```java
package cn.xpshuai.java4;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-06 13:33
 * @功能：动态代理的举例
 */

interface Human{
    String getBelief();
    void eat(String food);
}

//被代理类
class SuperMan implements Human{

    @Override
    public String getBelief() {
        return "I believe I can fly.";
    }

    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃：" + food);
    }
}


/*

要想实现动态代理，需要解决的问题？
1.如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
2.当通过代理类的对象调用方法时，如何动态的去调用被代理类中的同名方法

 */

class ProxyFactory{
    //调用此方法，返回一个代理类的对象，解决问题1
    public static Object getProxyInstance(Object obj){ //obj：被代理类的对象
        MyInvocationHandler handler = new MyInvocationHandler();

        handler.bind(obj);

        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }

}

class MyInvocationHandler implements InvocationHandler{
    private Object obj; //需要使用被代理类的对象进行赋值

    public void bind(Object obj){
        this.obj = obj;
    }

    //当我们通过代理类的对象，请用方法a时，就会自动调用如下的犯法:invoke()
   //将被代理类要指定的方法a的功能就声明在invoke()中
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        // obj: 被代理类的对象
        Object returnVal = method.invoke(obj, args);
        //上述方法的返回值就作为当前类中的invoke()的返回值
        return returnVal;
    }
}


//测试
public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        // proxyInstance: 代理类的对象
        Human proxyInstance = (Human)ProxyFactory.getProxyInstance(superMan);
        //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("麻辣烫");


        //只需要提供被代理类和接口就行啦
        System.out.println("*************");
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        ClothFactory proxyClothInstance1 = (ClothFactory)ProxyFactory.getProxyInstance(nikeClothFactory);
        proxyClothInstance1.produceCloth();

    }

}

```



![动态代理辅助理解](http://image.xpshuai.cn/%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E8%BE%85%E5%8A%A9%E7%90%86%E8%A7%A3.png)





**AOP与动态代理举例：**

> 面向切面编程

![AOP1](http://image.xpshuai.cn/AOP1.png)

![AOP2](http://image.xpshuai.cn/AOP2.png)





```java
package cn.xpshuai.java4;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @author: 剑胆琴心
 * @create: 2021-02-06 13:33
 * @功能：动态代理的举例
 */

interface Human{
    String getBelief();
    void eat(String food);
}

//被代理类
class SuperMan implements Human{

    @Override
    public String getBelief() {
        return "I believe I can fly.";
    }

    @Override
    public void eat(String food) {
        System.out.println("我喜欢吃：" + food);
    }
}


/*

要想实现动态代理，需要解决的问题？
1.如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象
2.当通过代理类的对象调用方法时，如何动态的去调用被代理类中的同名方法

 */

class ProxyFactory{
    //调用此方法，返回一个代理类的对象，解决问题1
    public static Object getProxyInstance(Object obj){ //obj：被代理类的对象
        MyInvocationHandler handler = new MyInvocationHandler();

        handler.bind(obj);

        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }

}

class MyInvocationHandler implements InvocationHandler{
    private Object obj; //需要使用被代理类的对象进行赋值

    public void bind(Object obj){
        this.obj = obj;
    }

    //当我们通过代理类的对象，请用方法a时，就会自动调用如下的犯法:invoke()
   //将被代理类要指定的方法a的功能就声明在invoke()中
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        HumanUtil util = new HumanUtil();
        util.method1();


        // method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
        // obj: 被代理类的对象
        Object returnVal = method.invoke(obj, args);

        util.method2();



        //上述方法的返回值就作为当前类中的invoke()的返回值
        return returnVal;
    }
}


//测试
public class ProxyTest {
    public static void main(String[] args) {
        SuperMan superMan = new SuperMan();
        // proxyInstance: 代理类的对象
        Human proxyInstance = (Human)ProxyFactory.getProxyInstance(superMan);
        //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
        String belief = proxyInstance.getBelief();
        System.out.println(belief);
        proxyInstance.eat("麻辣烫");


        //只需要提供被代理类和接口就行啦
        System.out.println("*************");
        NikeClothFactory nikeClothFactory = new NikeClothFactory();
        ClothFactory proxyClothInstance1 = (ClothFactory)ProxyFactory.getProxyInstance(nikeClothFactory);
        proxyClothInstance1.produceCloth();

    }

}


//测试AOP代理的通用方法
class HumanUtil{
//    通用方法1
    public void method1(){
        System.out.println("通用方法1");

    }

//    通用方法
    public void method2(){
        System.out.println("通用方法2");

    }



}
```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E5%8F%8D%E5%B0%84%E5%88%9D%E6%8E%A2/  

