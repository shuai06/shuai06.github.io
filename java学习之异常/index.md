# Java学习之异常




### 常见异常



- Error：Java虚拟机无法解决的严重问题，如StackOverflowErroe等，一般不编写针对性代码进行处理

- Exception：其它因编程错误或偶然的外在因素导致的一般性问题，可以伎用针对性的代码进行处理。例如：空指针访问、试图读取不存在的文件、网络连接中断、数组角标越界等



**分类：**

- 编译时异常
- 运行时异常



捕获错误最理想的是在编译期间。



### 异常处理1：try-catch-finally

```java
package cn.xpshuai.day12;
/*
异常处理机制

Error: JVM系统内部错误,一般不不编写针对性代码进行处理
        比如栈溢出，OOM（堆空间溢出）
Exception:
        编译时异常：编译器会自动变红叉, javac会异常，不会生成字节码文件都
        运行时异常：

异常的处理:抓抛模型
过程一:"抛":程序在正常执行的过程中，一旦出现异常，就会在异常代码处生成一个对应异常类的对象。
并将此对象抛出。
    一旦抛出对象以后，其后的代码就不再执行。

过程二:"抓":可以理解为异常的处理方式: 1.try-catch-finally  2.throws




try-catch-finally使用
try{
    可能出现异常的代码
}catch(异常类型1 变量名1){
}catch(异常类型2 变量名2){
}catch(异常类型3 变量名3){
}finally{
    一定会执行的代码放在这里
}

说明：
1.finally是可选的。
2．使用try将可能出现异常代码包装起来，在执行过程中，一旦出现异常，就会生成一个对应异常类的对象，根据此对象
的类型，去catch中进行匹配
3.catch中的异常类型如果没有子父类关系，则谁声明在上，谁声明在下无所谓。
  catch中的异常类型如果满足子父类关系，则要求子类一定声明在父类的上面。否则，报错

4.常用的异常对象处理的方式:
    String getMessage()
    printstackTrace()


5.在try结构中声明的变量，再出了try结构以后，就不能再被调用

6.体会:使用try-catch-finally处理编译时异常，是得程序在编译时就不再报错，但是运行时仍可能报错。
      相当于我们使用try-catch-finally将一个编译时可能出现的异常,延迟到运行时出现。


7.finally中的是一定会执行的代码，即使catch中又出现异常了，try中有return语句，catch中有return语句等情况。


8.像数据库连接、输入输出流、网络编程Socket等资源，JVMN是不能自动的回收的，我们需要自己手动的进行资源的
释放。此时的资源释放，就需要声明在finally中。

9. try-catch可以相互嵌套

10.开发中，由于运行时异常比较常见，所以我们通常就不针对运行时异常编写try-catch-finally针对于编译时异常，我们说一定要考虑异常的处理。
运行时异常不一定非要try-catch，但编译时异常一定要加

 */

import org.junit.Test;
import org.junit.internal.runners.statements.ExpectException;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Date;
import java.util.Scanner;

public class ExceptionTest {

    // Error
    @Test
    public void testError() {
//        testError();  //栈溢出
//        Integer[] arr = new Integer[1024*1024*1024];  //OOM 堆空间溢出
    }


    @Test
    public void testException() {
        String a;
        try{
            a = "123";
            System.out.println(a.charAt(6)); // java.lang.StringIndexOutOfBoundsException
        }catch (StringIndexOutOfBoundsException e){
//            System.out.println("出现了StringIndexOutOfBoundsException异常");
            System.out.println(e.getMessage());
//            e.printStackTrace();
        }catch (Exception e2){
            System.out.println("出现了其他异常2");
        }finally {
            System.out.println("finally继续执行其他语句");
        }
    }

    @Test
    public void testException2() {
//       Object obj = new Date();
//       String str = (String) obj;   // java.lang.ClassCastException

//        Scanner scanner = new Scanner(System.in);
//        int score  = scanner.nextInt();
//        System.out.println(score);
//        scanner.close();

//        int a = 10;
//        int b = 0;
//        System.out.println(a/b);  // java.lang.ArithmeticException:
    }

    @Test
    public void testTryCatch() {
        FileInputStream fis = null;
        try {
                File file = new File("test.txt");
                fis = new FileInputStream(file);
                int data = fis.read();
                while (data != -1){
                    System.out.println((char)data);
                    data = fis.read();
            }

        }catch (FileNotFoundException e){
            e.getMessage();
        }catch (IOException e){
            e.getMessage();
        }catch (Exception e){
            e.getMessage();
        }finally {
            try{  // try-catch可以相互嵌套
                if(fis != null)  //避免空指针异常
                    fis.close(); //连接不会自动释放
            }catch (Exception e){
                e.getMessage();
            }
            System.out.println("finally");
        }

    }

}

```





### 异常处理2：throws + 异常类型

```java
package cn.xpshuai.day12;
/*

1. "throws + 异常类型"写在方法的声明处。指明此方法执行时，可能会抛出的异常类型。
一旦当方法体执行时，出现异常，仍会在异常代码处生成一个异常类的对象，此对象满足throws后异常类型时,就会被抛出。异常代码后续的代码,就不再执行!

2．体会:try-catch-finally:真正的将异常给处理掉了。
throws的方式只是将异常抛给了方法的调用者。并设有真正将异常处理掉。

3.子类抛出的异常不能大于父类的
子类重写的方法抛出的异常类型不大于父类被重写的方法抛出的异常类型

4.开发中如何选择使用try-catch-finally 还是使用throws?
4.1 如果父类中被重写的方法没有throws方式处理异常，则子类重写的方法也不能使用throws，意味着如果
子类重写的方法中有异常，必须使用try-catch-finally方式处理。
4.2 执行的方法a中，先后又调用了另外的几个方法，这几个方法是递进关系执行的。我们建议这几个方法使用throw的方式进行处理。而执行的方法a可以考虑使用try-catch-finally方式进行处理。|






 */
import org.junit.Test;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

public class ThrowsTest {
    public static void main(String[] args) {
        try{
            method2();
        }catch (Exception e){
            e.getMessage();
        }
    }

    // 向上通报，抛给上面
//    @Test
    public static void method1() throws FileNotFoundException, IOException {
        File file = new File("test1.txt");
        FileInputStream fis = new FileInputStream(file);
        int data = fis.read();
        while (data != -1) {
            System.out.println((char) data);
            data = fis.read();
        }
    }

    // 这里调用method1了，所以method1的异常就抛到method2了
    public static void method2()  throws FileNotFoundException, IOException{
        method1();
        // method2处理不了，也往上抛

    }



}

```



### 手动抛出异常：throw

```java
class Student{
    private int id;
    public void register(int id) throws Exception{
        if (id >0){
            this.id = id;
        }else{
            //手动抛出异常对象
            throw new Exception("输入数据非法"); //编译和运行时都有了，throws给上层处理
//            throw new RuntimeException("输入数据非法！"); // 运行时异常

        }
    }
}
```



### 用户自定义异常类

```java
/*
用户自定义异常类
1.继承现有异常结构：RuntimeException . Exception
2.提供全局常量：serialVersionUID
3.提供重载的构造器

 */
public class SelfMakeExpection extends Exception{
    static final long serialVersionUID = -7034897190745777777L;

    public SelfMakeExpection() {
        super();
    }

    public SelfMakeExpection(String msg) {
        super(msg);
    }
}

```

调用自定义异常类：

```java
class Student{
    private int id;
    public void register(int id) throws Exception{
        if (id >0){
            this.id = id;
        }else{
            //手动抛出异常对象
//            throw new Exception("输入数据非法"); //编译和运行时都有了，throws给上层处理
//            throw new RuntimeException("输入数据非法！"); // 运行时异常
             throw new SelfMakeExpection("自定义异常类：出现错误啦~~~");
        }
    }
}

```



---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E5%BC%82%E5%B8%B8/  

