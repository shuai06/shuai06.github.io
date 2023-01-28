# Java学习之枚举类


#### 概述

- **枚举类的实现**
- JDK1.5之前需要自定义枚举类
- JDK 1.5新增的**enum关键字**用于定义枚举类
- 若枚举只有一个对象,则可以作为一种单例模式的实现方式。
- **枚举类的属性**
- **枚举类对象的属性不应允许被改动,所以应该使用private final修饰**
- 枚举类的使用private final修饰的属性应该在构造器中为其赋值
- 若枚举类显式的定义了带参数的构造器,则在列出枚举值时也必须对应的传入公数

#### 定义枚举类(两种方式)

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 18:18
 * @功能：枚举类
 *
 * 类的对象只有有限个、确定的。我们称此类为枚举类
 * 当需要定义一组常量时，强烈建议使用枚举类
 * 如果枚举类中只有一个对象，则可以作为单例模式的实现方式。
 *
 *
 * 如何定义枚举类
 * 方法1：jdk5.0之前：之定义枚举类
 * 方法2：jdk5.0，可以使用enum关键字定义枚举类(常用)
 */
public class EnumTest {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring);



        // //jdk5.0，可以使用enum关键字定义枚举类(常用)
        //定义的枚举类继承于java.lang.Enum
        Season1 spring1 = Season1.SPRING;
        System.out.println(spring1);
        System.out.println(Season1.class.getSuperclass());

    }


}


//jdk5.0之前：自定义枚举类
class Season{
    //1.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTOM = new Season("秋天", "秋高气爽");
    public static final Season WINTER = new Season("冬天", "冰天雪地");

    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //4.其他诉求：提供toString()
    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }



}


// //jdk5.0，可以使用enum关键字定义枚举类(常用)
enum Season1{
//    1.提供当前枚举类的帝乡，多个对象之间用,隔开
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "夏日炎炎"),
    AUTOM("秋天", "秋高气爽"),
    WINTER("冬天", "冰天雪地");

    //2.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season1(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final


    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //4.其他诉求：提供toString()

}


```





#### Enum类的主要方法

![Enum类的主要方法](http://image.xpshuai.cn/JavaEnum%E7%B1%BB%E7%9A%84%E4%B8%BB%E8%A6%81%E6%96%B9%E6%B3%95.png)

![image-20210125164002288](C:\Users\19401\AppData\Roaming\Typora\typora-user-images\image-20210125164002288.png)

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 18:18
 * @功能：枚举类
 *
 * 类的对象只有有限个、确定的。我们称此类为枚举类
 * 当需要定义一组常量时，强烈建议使用枚举类
 * 如果枚举类中只有一个对象，则可以作为单例模式的实现方式。
 *
 *
 * 如何定义枚举类
 * 方法1：jdk5.0之前：之定义枚举类
 * 方法2：jdk5.0，可以使用enum关键字定义枚举类(常用)
 *
 *
 * Enum中常用方法：
 * value()  返回枚举类型的对象数据
 * valueOf()  可以把一个字符串转为对应的枚举类对象（要求字符串必须是枚举类对象的"名字"）
 * toString()  返回当前枚举类对象常量的名称
 */
public class EnumTest {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring);



        // //jdk5.0，可以使用enum关键字定义枚举类(常用)
        //定义的枚举类继承于java.lang.Enum
        Season1 spring1 = Season1.SPRING;
        // toString()
        System.out.println(spring1);
        System.out.println(Season1.class.getSuperclass());

        //values()
        Season1[] values = Season1.values();
        for (int i = 0; i < values.length; i++) {
            System.out.println(values[i]);
        }


        // valueOf(String objName): 根据提供objName, 返回枚举类中对象名是objName的对象
        // 如果没有，则抛出异常
        Season1 winter = Season1.valueOf("WINTER");
        System.out.println(winter);

    }


}


//jdk5.0之前：自定义枚举类
class Season{
    //1.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTOM = new Season("秋天", "秋高气爽");
    public static final Season WINTER = new Season("冬天", "冰天雪地");

    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //4.其他诉求：提供toString()
    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }



}


// //jdk5.0，可以使用enum关键字定义枚举类(常用)
enum Season1{
//    1.提供当前枚举类的帝乡，多个对象之间用,隔开
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "夏日炎炎"),
    AUTOM("秋天", "秋高气爽"),
    WINTER("冬天", "冰天雪地");

    //2.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season1(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final


    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

}

```





#### 使用enum关键字定义的枚举类实现接口



```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-24 18:18
 * @功能：枚举类
 *
 * 类的对象只有有限个、确定的。我们称此类为枚举类
 * 当需要定义一组常量时，强烈建议使用枚举类
 * 如果枚举类中只有一个对象，则可以作为单例模式的实现方式。
 *
 *
 * 如何定义枚举类
 * 方法1：jdk5.0之前：之定义枚举类
 * 方法2：jdk5.0，可以使用enum关键字定义枚举类(常用)
 *
 *
 * Enum中常用方法：
 * value()  返回枚举类型的对象数据
 * valueOf()  可以把一个字符串转为对应的枚举类对象（要求字符串必须是枚举类对象的"名字"）
 * toString()  返回当前枚举类对象常量的名称
 *
 *
 *
 *
 * 使用enum关键字定义的枚举类实现接口：
 *情况1：实现接口，在enum类中实现抽象方法
 *情况2：让枚举类的对象分别实现接口中的抽象方法
 *
 */
public class EnumTest {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        System.out.println(spring);



        // //jdk5.0，可以使用enum关键字定义枚举类(常用)
        //定义的枚举类继承于java.lang.Enum
        Season1 spring1 = Season1.SPRING;
        // toString()
        System.out.println(spring1);
        System.out.println(Season1.class.getSuperclass());

        //values()
        Season1[] values = Season1.values();
        for (int i = 0; i < values.length; i++) {
            System.out.println(values[i]);
            values[i].show();
        }


        // valueOf(String objName): 根据提供objName, 返回枚举类中对象名是objName的对象
        // 如果没有，则抛出异常
        Season1 winter = Season1.valueOf("WINTER");
        System.out.println(winter);
        winter.show();  //类实现了这个接口，对象就可以去调用

    }


}


//jdk5.0之前：自定义枚举类
class Season{
    //1.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTOM = new Season("秋天", "秋高气爽");
    public static final Season WINTER = new Season("冬天", "冰天雪地");

    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //4.其他诉求：提供toString()
    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }



}


interface Info{
    void show();
}

// //jdk5.0，可以使用enum关键字定义枚举类(常用)
enum Season1 implements Info{
//    1.提供当前枚举类的帝乡，多个对象之间用,隔开
    SPRING("春天", "春暖花开"){
    @Override
    public void show() {
        System.out.println("春天在哪里");
    }
},
    SUMMER("夏天", "夏日炎炎"){
        @Override
        public void show() {
            System.out.println("夏天在哪里");
        }
    },
    AUTOM("秋天", "秋高气爽"){
        @Override
        public void show() {
            System.out.println("秋天在哪里");
        }
    },
    WINTER("冬天", "冰天雪地"){
        @Override
        public void show() {
            System.out.println("冬天在哪里");
        }
    };

    //2.声明Season对象的属性
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器, 并给对象属性赋值
    private Season1(String seasonName, String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //3.提供当前枚举类的对象的多个对象  public static final


    //4.其他诉求：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }

    //在enum类中实现抽象方法
    @Override
    public void show() {
        System.out.println("这是一个季节");
    }

    //4.其他诉求：提供toString()
}


```







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E6%9E%9A%E4%B8%BE%E7%B1%BB/  

