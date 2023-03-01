# Java学习之多线程




### 基本概念

![image-20210116145142703](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%A4%9A%E7%BA%BF%E7%A8%8B_%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5.png)



**JVM内存结构：**

![image-20210116145803616](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%A4%9A%E7%BA%BF%E7%A8%8B_JVM.png)



**单核CPU和多核CPU的理解：**

- 单核：其实是一种假的多线程。因为在一个时间单元内，也只能执行一个线程的任务。
- 多核：如果是多核的话，才能更好的发挥多线程的效率。（现在的服务器都是多核的)
- 一个Java应用程序java.exe，其实至少有三个线程：main()主线程，gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。



- **并行：**多个CPU同时执行多个任务。比如:多个人同时做不同的事。

- **并发：**一个CPU(采用时间片)同时执行多个任务。比如:秒杀、多个人做同一件事



**使用多线程的优点：**

1. 提高应用程序的响应。对图形化界面更有意义，可增强用户体验。

2. 提高计算机系统CPU的利用率

3. 改善程序结构。将既长又复杂的进程分为多个线程，独立运行，利于理解和修改

   

**何时需要多线程：**

- 程序需要同时执行两个或多个任务。
- 程序需要实现一些需要等待的任务时，如用户输入、文件读写操作、网络操作、搜索等。
- 需要一些后台运行的程序时。







### 线程的创建和使用

> 共四种：1.继承与Thread类；2.实现Runnable接口； 3.实现Callable接口； 4.线程池



Java语言的JVM允许程序运行多个线程，它通过`java.lang.Thread`类来体现。

**Thread类的特性：**

每个线程都是通过某个特定Thread对象的`run()`方法来完成操作的，经常把`run()`方法的主体称为线程体
通过该Thread对象的`start()`方法来启动这个线程，而非直接调用run()



#### <font color=red>方法1：继承于Thread类</font>

**步骤：**

1. 创建一个继承与Thraed的子类
2. 重写Thread类的run()方法-->将此线程要执行的操作写在该方法内
3. 创建Thread类的子对象
4. 通过此对象调用start()方法



例子：多线程遍历100以内所有偶数

```
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create 2021-01-16 14:21
 *

程序：指一段静态的代码
进程：正在运行的一个程序，是资源分配的单位，系统会为每个进程分配Wie不同的内存区域是动态的。
线程：调度和执行的单位，每个线程拥有独立的运行栈和程序计数器，线程切换的开销小
      进程可以细化为线程，线程是一个陈旭内部的一条执行路径
      多个线程操作共享的系统资源(进程中的方法区和堆)可能带来安全隐患。



 【创建多线程】
【方式1】：继承Thread类
 1.创建一个继承与Thraed的子类
 2.重写Thread类的run()方法-->将此线程要执行的操作写在该方法内
 3.创建Thread类的子对象
 4.通过此对象调用start()方法
 例子：遍历100以内所有偶数


 *
 */
public class MultiThreadsTest {
    public static void main(String[] args) {
        //3.创建Thread类的子对象
        MyThread1 myThread1 = new MyThread1(); // alt + enter 自动创建对象
        //4.通过此对象调用start()方法：(1)启动当前线程 (2)调用当前线程的run()
        myThread1.start();
//        问题1：不能通过直接调用run()的方式启动线程
//        myThread1.run();
//        问题2：再启动一个线程，遍历100以内偶数，不可以还让已经start()的线程去执行
//               但是可以new一个新的线程的对象，再去启动
        MyThread1 myThread2 = new MyThread1();
        myThread2.start();

        //如下操作仍然是在main线程中执行的
        for (int i = 0; i < 100; i++) {
            System.out.println("hello");  //hello在哪输出是不确定的
            System.out.println(Thread.currentThread().getName() + ":" + i);  //查看线程名

        }

    }
}


//1.创建一个继承与Thraed的子类
class MyThread1 extends Thread{

    //2.重写Thread类的run()方法
    @Override
    public void run() {
        //方法体里面，要写要做的事
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}

```



例子：创建两个分线程，其中一个遍历100以内偶数，另外一个遍历100以内基数(两个线程做的事情不一样)

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create 2021-01-16 16:11
 * 练习：创建两个分线程，其中一个遍历100以内偶数，另外一个遍历100以内基数(两个线程做的事情不一样)
 */
public class ThreadCase {
    public static void main(String[] args) {
        //可以写两个，分别调用
        ForDouble m1 = new ForDouble();
        ForSingle m2 = new ForSingle();

        m1.start();
        m2.start();


        // 或者：如果只用一次的话，可以创建Thread类的匿名子类的方法
        new Thread(){
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    if(i % 2 == 0){
                        System.out.println(Thread.currentThread().getName() + "-- 偶数 --" + i);
                    }
                }
            }
        }.start();
        new Thread(){
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    if(i % 2 != 0){
                        System.out.println(Thread.currentThread().getName() + "-- 偶数 --" + i);
                    }
                }
            }
        }.start();
        



    }
}


class ForDouble extends Thread{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + "-- 偶数 --" + i);
            }
        }
    }
}

class ForSingle extends Thread{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 != 0){
                System.out.println(Thread.currentThread().getName() + "-- 奇数 --" + i);
            }
        }
    }
}

```



不完善的例子2：继承Thread方式，实现多线程三个窗口售票，总票数为100张例子（待实现线程安全问题）

```java
package cn.xpshuai.java1;

/**
 * @author: xps
 * @create 2021-01-16 17:30
 * @功能：继承Thread方式，实现多线程三个窗口售票，总票数为100张例子
 */
public class ThreadCase2 {
    public static void main(String[] args) {
        Windows w1 = new Windows();
        Windows w2 = new Windows();
        Windows w3 = new Windows();

        w1.setName("窗口1");
        w2.setName("窗口2");
        w3.setName("窗口3");

        w1.start();
        w2.start();
        w3.start();
    }

}


class Windows extends Thread{
//    private int tickets = 100; //这样只是实例的变量，会卖300张
    private static int tickets = 100; //加了static也不对，需要解决线程安全问题
    @Override
    public void run() {
        while (true){
            if (tickets > 0){
                System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
                tickets--;
            }else {
                break;
            }
        }
    }
}

```







#### <font color=red>方法2：实现Runnable接口的方式</font>

**步骤：**

1. 创建一个实现了Runnable接口的类

2. 实现Runnable中的抽象方法: run()

3. 创建实现类的对象

4. 将此对象作为参数传递到Thread类的构造器中，创建Thread的对象

5. 通过Thread的对象调用start()

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create 2021-01-16 17:48
 * @功能：
 *
 *
 * 【创建多线程的方式2】：实现Runnable接口的方式
 *  1.创建一个实现了Runnable接口的类
 *  2.实现Runnable中的抽象方法: run()
 *  3.创建实现类的对象
 *  4.将此对象作为参数传递到Thread类的构造器中，创建Thread的对象
 *  5.通过Thread的对象调用start()
 *
 */
public class MultiThreadTest2 {
    public static void main(String[] args) {

//        3.创建实现类的对象
        MyThread11 m11 = new MyThread11();
//        4.将此对象作为参数传递到Thread类的构造器中，创建Thread的对象
        Thread t1 = new Thread(m11);//类似于多态的思想
        t1.setName("线程1");

//        5.通过Thread的对象调用start():(1)启动线程 (2)调用当前线程的run()-->调用了Runnable类型中的target的run
        t1.start();

        //再启动一个线程,直接new个Thread，共用同一个Runnable的实例
        Thread t2 = new Thread(m11);
        t2.setName("线程2");
        t2.start();

    }
}

//1.创建一个实现了Runnable接口的类
class MyThread11 implements Runnable{
//    2.实现Runnable中的抽象方法: run()
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}
```



不完善的例子：用实现Runnable接口的方式，实现多线程三个窗口售票，总票数为100张例子（待实现线程安全问题）

```java
package cn.xpshuai.java1;

/**
 * @author: xps
 * @create 2021-01-16 17:57
 * @功能：用实现Runnable接口的方式，实现现多线程三个窗口售票，总票数为100张例子
 * 待解决，仍然存在线程安全问题
 */
public class ThreadCase3 {
    public static void main(String[] args) {
        Windows2 w = new Windows2();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }

}



class Windows2 implements Runnable{
    private int tickets = 100; //不用加static了，但还会有线程安全问题
    @Override
    public void run() {
        while (true){
            if (tickets > 0){
                System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
                tickets--;
            }else {
                break;
            }
        }
    }
}
```



#### 两种创建方式比较

```java
开发中，第二种实现Runnable接口的方式比较好
     如果用第一种方式，若该类已经存在一个父类，而Java又没有多继承,...
     第二种方式，不仅没有类的单继承性的局限性，而且多个线程天然的就适合并可以共享数据
联系：Thread类本身也实现了Runnable接口，两种方式都需要重写run()，将程序逻辑写在run()中
```





#### 线程中常用的方法



![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/java%E7%BA%BF%E7%A8%8B%E4%B8%AD%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95.jpg)

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create 2021-01-16 16:47
 * @功能：测试Thread中的常用方法
 *
 *
 * 1.start() :启动当前线程；调用当前线程的run()方法
 * 2.run():通常需要重写Thread类的此方法，将创建的线程要自行的操作声明在此方法中
 * 3.currentThread()：静态方法，返回指定当前代码的线程
 * 4.getName():获取当前线程的名字
 * 5.setName():设置当前线程的名字
 * 6.yield():线程让步，暂停。释放当前CPU的执行
 * 7.join():在线程A中调用线程B的join()，此时线程A就进入阻塞，直到线程B完全执行完之后，线程A才结束阻塞状态
 * 8.sleep(long millitime): 让当前线程睡眠指定毫秒数(期间当前线程是阻塞状态)， 还是个静态方法
 * 9.stop():强制结束当前线程（不推荐使用）
 * 10.isAlive():判断当前线程知否存活
 *
 *
 *
 * 【线程的优先级】
 * MAX_PRIORITY:10
 * MIN_PRIORITY:1
 * NORM_PRIORITY:5
 *
 * 涉及的方法：
 * getPriority()：返回线程优先级
 * setPriority(int newPriority):改变线程的优先级
 *
 * 说明：
 * 线程创建时候继承父线程的优先级
 *低优先级只是获得调度的概率低，并非一定是在高优先级线程之后才被调用
 *
 */

class HelloThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if(i % 2 != 0){
                try {
                    sleep(1000); //抛出异常，需要try包裹起来(不能throws，因为不能比父类大，父类没有抛异常)
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + Thread.currentThread().getPriority() +  "-- 奇数 --" + i);
            }

            if(i % 20 ==0){
//                this.yield();
                yield();
            }
        }
    }

//    //也可以通过构造器方式给线程命名
//    public HelloThread(String name){
//        super(name);
//    }
}



public class ThreadMethod {
    public static void main(String[] args) throws InterruptedException {
//        HelloThread h1 = new HelloThread("线程线程1");  //通过构造器方式给线程命名
        HelloThread h1 = new HelloThread();
        h1.setName("线程1");
        h1.setPriority(Thread.MAX_PRIORITY); //设置分线程的优先级

        h1.start();

        //给主线程命名
        Thread.currentThread().setName("主线程");
        for (int i = 0; i < 100; i++) {
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + Thread.currentThread().getPriority() + ":" + i);
            }
            if(i == 20){
                //调用了另外一个线程的join方法，等他执行完再继续回来执行
                h1.join();   //需要加 throws
            }
        }
        h1.isAlive();

    }

}

```



#### 线程的调度

- 时间片
- 抢占式：高优先级的线程抢占CPU



**Java的调度方法**

- 同优先级线程组成先进先出队列(先到先服务），使用时间片策略
- 对高优先级，使用优先调度的抢占式策略



**线程的优先级：**

* `MAX_PRIORITY`:10
* `MIN_PRIORITY`:1
* `NORM_PRIORITY`:5



**涉及的方法：**

* `getPriority()`：返回线程优先级
* `setPriority(int p)`:改变线程的优先级



**说明：**

* 线程创建时候继承父线程的优先级
* 低优先级只是获得调度的概率低，并非一定是在高优先级线程之后才被调用



### 线程的分类

- 守护线程
- 用户线程



它们在几乎每个方面都是相同的，唯一的区别是判断JVM何时离开。

守护线程是用来服务用户线程的，通过在`start()`方法前调用
`thread.setDaemon(true)`可以把一个用户线程变成一个守护线程。

Java垃圾回收就是一个典型的守护线程。
若JVM中都是守护线程，当前JVM将退出。

形象理解：兔死狗烹，鸟尽弓藏







### 线程的生命周期

> 这个生命周期流程图需要记住！！！

![image-20210117085742806](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpg)







### 线程的同步(重难点)

> 解决的是线程安全的问题

**问题的提出：**

多个线程执行的不确定性引起执行结果的不稳定
多个线程对账本的共享，会造成操作的不完整性，会破坏数据。





#### 解决线程安全问题共有三种方式：

##### <font color=red>1.同步代码块</font>

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 9:14
 * @功能：使用同步代码块，解决前面通过实现Runnable方法实现的3个窗口同时卖100张票出现重票、错票的的线程安全问题

 问题：出现了重票、错票-->出现了线程的安全问题
 原因：当某个线程操作车票过程中，尚未操作完成时，其他线程参与进来，也操作车票
 解决：当一个线程在操作贡献数据的时候，其他线程不能参与进来，直到线程A操作完其他线程才可以开始操作共享数据，
        即是线程A出现了阻塞，也不能被改变。加锁。

 在Java中：通过同步机制来解决线程的安全问题

 方式1：同步代码块
    synchronized(同步监视器){
        //需要被同步的代码：即操作共享数据(多个线程共同操作的变量)的代码(不能包含代码多了，也不能包含代码少了-->只包含操作的代码)
    }
    说明：同步监视器-->俗称：锁。任何一个类的对象都可以充当锁
                  要求：多个线程必须共用同一把锁(唯一的)

    同步的方式：解决了线程的安全问题 -- 好处
               操作同步代码时，只能有一个线程参与，其他线程等待，相当于是一个单线程的过程，效率低  --不足之处

    补充: 在实现Runnable接口创建多线程的方法中，我们可以考虑使用this作为同步监视器



  方式2：同步方法
        如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明为同步的
        synchronized 操作方法_name(){
                //操作共享数据的代码块
        }
     关于同步方法的总结:
     1.同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
     2．非静态的同步方法，同步监视器是:this
        静态的同步方法，同步监视器是:当前类本身

 */
public class ThreadCase4 {
    public static void main(String[] args) {
        Windows2 w = new Windows2();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }

}



class Windows2 implements Runnable{
    private int tickets = 100;  //共享数据
//    Object obj = new Object();  //这里的对象是唯一的就行(多个线程用的是同一个)

    @Override
    public void run() {
        while (true){ // synchronized不能包多，比如不能把while(true)包进来-->否则就成了只有一个窗口卖票了
            //
            synchronized(Windows2.class){  //监视器方式1：这里可以用当前类作为监视器(类也是对象)，是唯一的(只会加载一次)
//            监视器方式2：synchronized (obj){ //这里不能用this(代表t1,t2,t3对象，不唯一)作为同步监视器
                if (tickets > 0){
                    try {
                        Thread.sleep(100); //先加了sleep强制阻塞，让出现错票的概率提升了
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
                    tickets--;
                }else {
                    break;
                }
            }
        }
    }
}

```

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 9:47
 * @功能： 使用同步代码块，解决前面通过继承Thread方式实现的3个窗口同时卖100张票出现重票、错票的的线程安全问题
    说明：在通过继承Thread创建多方式，慎用this作为同步监视器，考虑使用当前类作为同步监视器（考虑是不是唯一的）


 */
public class ThreadCase5 {
    public static void main(String[] args) {
        Windows w1 = new Windows();
        Windows w2 = new Windows();
        Windows w3 = new Windows();

        w1.setName("窗口1");
        w2.setName("窗口2");
        w3.setName("窗口3");

        w1.start();
        w2.start();
        w3.start();
    }

}


class Windows extends Thread{
    private static int tickets = 100; //加了static也不对，需要解决线程安全问题
//    private static Object obj = new Object(); //不唯一，所以加static

    @Override
    public void run() {
        while (true){
            //  监视器方式1：为了更简便,用当前对象来作为监视器，这里的this：唯一的Window对象
            synchronized(this){ // 监视器方式2：synchronized(obj)
                if (tickets > 0){
                    try {
                        Thread.sleep(100); //使得错票概率变大
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
                    tickets--;
                }else {
                    break;
                }
            }
        }
    }
}


```



##### <font color=red>2.同步方法</font>

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 10:04
 * @功能：使用同步方法，解决通过Runnable实现的抢票的安全问题
 *
 *
 *
 *【关于同步方法的总结】:
 * 1.同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
 * 2．非静态的同步方法，同步监视器是:this
 *    静态的同步方法，同步监视器是:当前类本身
 *
 *
 *
 *
 */
public class ThreadCase6 {
    public static void main(String[] args) {
        Windows3 w = new Windows3();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }

}



class Windows3 implements Runnable{
    private int tickets = 100;  //共享数据

    @Override
    public void run() {
        while (true){
                show();
            }
        }

    // 同步监视器：this，只是省略了没写
    private synchronized void show(){ //把操作代码单独提出来，不能多提也不能少提
        if (tickets > 0){
            try {
                Thread.sleep(100); //先加了sleep强制阻塞，让出现错票的概率提升了
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
            tickets--;
        }
    }
}

```

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 10:10
 * @功能： 使用同步方法，解决通过继承Thread类的方式实现的抢票的安全问题
 */
public class ThreadCase7 {
    public static void main(String[] args) {
        Windows4 w1 = new Windows4();
        Windows4 w2 = new Windows4();
        Windows4 w3 = new Windows4();

        w1.setName("窗口1");
        w2.setName("窗口2");
        w3.setName("窗口3");

        w1.start();
        w2.start();
        w3.start();
    }

}


class Windows4 extends Thread{
    private static int tickets = 100; //加了static也不对，需要解决线程安全问题

    @Override
    public void run() {
        while (true){
            show();
        }
    }

    private static synchronized void show(){  //把方法写出静态的，监视器唯一(为Windows.class)，正确
//    private synchronized void show(){   按照前面的方式直接加synchronized是不行的(因为监视器不是唯一的同一个)，错误的写法
        if (tickets > 0){
            try {
                Thread.sleep(100); //使得错票概率变大
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + tickets);
            tickets--;
        }
    }

}

```



**将单例模式的懒汉式，改成线程安全的：**

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 10:27
 * @功能：使用同步机制，将单例模式的懒汉式，改成线程安全的
 */
public class BankTest {
    
}


class Bank{
    private Bank(){

    }
    private static Bank instance = null;
    //懒汉式

    public static  Bank getInstance() { //同步锁为当前类本身(Bank.class)
//方式1(同步代码块/同步方法)：效率稍差
//        synchronized(Bank.class){
//            if(instance == null){
//                instance = new Bank();
//            }
//            return instance;
//        }
        //方式2：效率较高
        if(instance == null){ //双重检查
            synchronized(Bank.class){
                if(instance == null){
                    instance = new Bank();
                }
            }
        }

        return instance;
    }
    
}
```



##### <font color=red>3.Lock锁方式</font>



**死锁的问题：**

**死锁：**

不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁  
出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续  

**解决方法：**

- 专门的算法、原则
- 尽量减少同步资源的定义>尽量避免嵌套同步



死锁例子：

```java
package cn.xpshuai.java2;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 10:42
 * @功能：出现了死锁
 *
 *
 *1.死锁的理解:不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁
 * 
 *2.说明:
 * 1）出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续
 * 2）我们使用同步时,要避免出现死锁。|
 *
 */
public class LuckTest {
    public static void main(String[] args) {
        StringBuffer s1 = new StringBuffer();
        StringBuffer s2 = new StringBuffer();

        // 用匿名方式写继承Thread类的方式
        new Thread(){
            @Override
            public void run() {
                synchronized (s1){
                    s1.append("a");
                    s2.append("1");

                    try {
                        Thread.sleep(100);  //增加死锁出现的概率
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (s2){
                        s1.append("b");
                        s2.append("2");

                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }.start();

        // 用匿名方式写实现Runnable接口的方式
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (s2){
                    s1.append("c");
                    s2.append("3");

                    try {
                        Thread.sleep(100);  //增加死锁出现的概率
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (s2){
                        s1.append("d");
                        s2.append("4");

                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }).start();
    }
}

```



**Lock锁方式解决线程安全问题：**

![image-20210117110948119](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/Lock%E9%94%81.png)

```java
package cn.xpshuai.java2;

import java.util.concurrent.locks.ReentrantLock;

/**
 * @author: 剑胆琴心
 * @create 2021-01-17 11:10
 * @功能：解决线程安全问题的方式三：Lock锁 -- JDK5.0新增
 *

步骤在下面的例子中：
1.实例化一个ReentrantLock对象(若设置为true，为公平，先进先出；不写就是false)
2.调用lock()锁定方法
3.调用解锁方法unlock()



面试题：synchronized与lock异同
 同：二者都可以解决线程安全问题
 异：synchronized机制在执行完相应的同步代码之后，自动释放同步监视器
     Lock需要手动的启动同步(lock())，同时结束同时也需要手动的实现(unlock())

 建议优先用Lock这种手动方式
 优先使用顺序：
    Lock →同步代码块（已经进入了方法体，分配了相应资源）→同步方法（在方法体之外)


 *
 */
public class LuckTest2 {
    public static void main(String[] args) {
        Window w = new Window();

        Thread t1 = new Thread(w);
        Thread t2 = new Thread(w);
        Thread t3 = new Thread(w);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }



}


class Window implements Runnable{
    private int ticket = 100;

//    1.实例化一个ReentrantLock对象(若设置为true，为公平，先进先出；不写就是false)
    private ReentrantLock lock = new ReentrantLock(true);

    @Override
    public void run() {
        while (true){
            try{
                //2.调用lock()锁定方法
                lock.lock();
                if (ticket > 0){
                    try {
                        Thread.sleep(100); //先加了sleep强制阻塞，让出现错票的概率提升了
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + ":卖票，票号为：" + ticket);
                    ticket--;
                }else {
                    break;
                }
            }finally {
    //            3.调用解锁方法unlock()
                lock.unlock();
            }
        }
    }
}

```



**synchronized与Lock，二者对比：**

1.  Lock是显式锁（手动开启和关闭锁，别忘记关闭锁），synchronized是隐式锁，出了作用域自动释放
2.  Lock只有代码块锁，synchronized有代码块锁和方法锁
3. 使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性(提供更多的子类)



**优先使用顺序:**
Lock →同步代码块（已经进入了方法体，分配了相应资源）→同步方法(在方法体之外)|











### 线程的通信

> 其实就是几个方法的使用



```java
package cn.xpshuai.java3;

/**
 * @author: 剑胆琴心
 * @create： 2021-01-17 14:12
 * @功能：线程通信的例子，一个线程打印偶数，一个线程打印奇数, 交替打印
 *
 *
 *设计到的三个方法：
 * wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器
 * notify(): 一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
 * notify():一旦执行此方法,就会唤醒所有被wait的线程。
 *
 * 说明：
 * 1.wait(),notify(),notify()以上三个方法必须出现在同步代码块或者同步方法当中，不能用在lock()的情况下
 * 2.这三个方法的调用者必须是同步代码块或者同步方法当中的同步监视器，否则会出异常
 * 3.这三个方法是定义在java.lang.Object中的
 *
 */
public class NumTest {
    public static void main(String[] args) {
        Number n = new Number();
        Thread t1 = new Thread(n);
        Thread t2 = new Thread(n);

        t1.setName("线程1");
        t2.setName("线程2");

        t1.start();
        t2.start();

    }
}


class Number implements Runnable{
    private int number = 1;
//    Object obj = new Object();

    @Override
    public void run() {
        while (true){
            synchronized (this){
//            synchronized (obj){
                notify(); // 2.唤醒另一个线程
//                this.notify(); // 2.唤醒另一个线程
//                obj.notify(); // 如果同步代码块中同步监视器为Object，调用这个方法也必须为obj
//                notifyAll();  // 唤醒其他所有线程

                if(number <= 100){
                    System.out.println(Thread.currentThread().getName() + ":" + number);  //查看线程名
                    number++;

                    try {
                        wait();  //1.让调用wait()的线程进入阻塞状态
//                        obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }else {
                    break;
                }
            }
        }
    }
}


```



**面试题：**sleep()和wait的异同

```java
1.相同点:一旦执行方法，都可以使得当前的线程进入阻塞状态。
2.不同点:
	1）两个方法声明的位置不同:Thread类中声明sleep() , object类中声明wait()
	2）调用的要求不同: sleep()可以在任何需要的场景下调用。而wait()必须使用在同步代码块或同步方法中调用
	3) 关于是否释放同步监视器，如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁(同步监视器)

```



**经典例题：生产者/消费者问题**

```java
package cn.xpshuai.java3;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-17 14:42
 * @功能：生产者/消费者问题的例子
 *
 *
 * 生产者(Productor)将产品交给店员(CLerk)，而
 * 消费者(Customer)从店员处取走产品，
 * 店员一次只能持有固定数量的产品(比如:20)，
 * 如果生产者试图生产更多的产品，店员会叫生产者停一下，
 * 如果店中有空位放产品了再通知生产者继续生产，
 * 如果店中没有产品了，店员会告诉消费者等一下，
 * 如果店中有产品了再通知消费者来取走产品。
 *
 *
 * 涉及：同步机制、多线程通信
 *
 *
 */
public class ProducerCustomer {
    public static void main(String[] args) {
        Clerk clerk = new Clerk();  //这个对象自始至终只造了一个

        Producer p1 = new Producer(clerk);
        p1.setName("生产者1");

        Customer c1 = new Customer(clerk);
        c1.setName("消费者1");
        
        p1.start();
        c1.start();

        

    }
}

class Clerk{
    private int productCount = 0;

    //生产产品
    public synchronized void produceProduct()  { //用同步方法
        if(productCount < 20){
            productCount++;
            System.out.println(Thread.currentThread().getName() + ":开始生产第" + productCount + "个产品");
            notify();  //唤醒消费者
        }else {
            //等待
            try{
                wait();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }

    }

    //消费产品
    public synchronized void consumeProduct(){
        if(productCount > 0){
            System.out.println(Thread.currentThread().getName() + ":开始消费第" + productCount + "个产品");
            productCount--;
            notify();  //唤醒生产者
        }else {
            //等待
            try{
                wait();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }

    }
}


class Producer extends Thread{ //生产者
    private Clerk clerk;

    public Producer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ":开始生产产品...");
        while (true){
            clerk.produceProduct();
        }
    }
}


class Customer extends Thread{ //消费者
    private Clerk clerk;

    public Customer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ":开始消费产品...");
        while (true){
            clerk.consumeProduct();
        }
    }
}

```





### JDK5.0新增线程创建方式

> 2种方法



#### <font color=red>方法3：实现Callable接口</font>

与使用`Runnable`相比，`Callable`功能更强大些：

- 相比重写`run()`方法，可以有返回值(重写`call()`方法)
- 方法可以抛出异常
- 支持泛型的返回值
- 需要借助`FutureTask`类，比如获取返回结果



**Future接口：**

- 可以对具体Runnable、Callable任务的执行结果进行取消、查询是否完成、获取结果等；
- `FutrueTask`是Futrue接口的**唯一**的实现类
- FutureTask同时实现了Runnable,Future接口。它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值





```java
package cn.xpshuai.java3;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-17 15:0
 * @功能：创建线程的方式三: 实现Callable接口 -- JDK5.0新增
 *
 *
 *步骤：
 *1.创建一个Callable的实现类
 *2.实现call方法，将此线程需要执行的操作声明在call()中
 *3.创建Callable接口实现类的对象
 *4.将此Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
 *5.将FutureTask的对象作为参数，传递到Thread类的构造器中，创建Thread对象，并调用start()
 *6.获取Callable中call()方法的返回值
 *
 *
 * 如何理解实现Callable接口的方式创建多线程，比实现Runnable接口创建多线程方式强大？
 * 1.call()可以由返回值
 * 2.call()可以抛出异常，被外面的操作捕获，获取异常的信息
 * 3.Callable是支持泛型的
 *
 *
 *
 * */

public class CallableTest {
    public static void main(String[] args) {
//        3.创建Callable接口实现类的对象
        NumThread n1 = new NumThread();
//        4.将此Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
        FutureTask futureTask = new FutureTask(n1);
//        FutureTask<Integer> futureTask = new FutureTask<Integer>(n1);
//        5.将FutureTask的对象作为参数，传递到Thread类的构造器中，创建Thread对象，并调用start()
        new Thread(futureTask).start();

        try {
//            6.获取Callable中call()方法的返回值
            //get()返回值即为FutureTask构造器参数Callable实现类重写的的call()的返回值
            Object sum = futureTask.get(); //获取回调方法的返回值, 如果对返回值不感兴趣可以不用调用这个get()
//            Integer sum = futureTask.get();
            System.out.println("总和为：" + sum);

        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }


    }
}


//1.创建一个Callable的实现类
class NumThread implements Callable{
//class NumThread implements Callable<Integer>{
//    2.实现call方法，将此线程需要执行的操作声明在call()中
    @Override
    public Object call() throws Exception {
//    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0){
                System.out.println(i);
                sum += i;
            }
        }
        return sum;
    }
}

```





#### <font color=red>方法4：线程池</font>

> 开发中，常用这个方法



**背景: **经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。

**思路:** 提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。类似生活中的公共交通工具。

**好处:**

- 提高响应速度（减少了创建新线程的时间)
- 降低资源消耗（重复利用线程池中线程,不需要每次都创建)
- 便于线程管理
- corePoolSize: 核心池的大小
- maximumPoolSize: 最大线程数
- keepAliveTime: 线程没有任务时最多保持多长时间后会终止
- ... ...

![image-20210117153400642](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/java_ExecutorService.png)





```java
package cn.xpshuai.java3;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-17 15:37
 * @功能：创建线程的方式4：线程池
 *
 * 【步骤】
 * 1.提供指定线程数量的线程池
 * 2.执行指定的线程的操作，需要提供实现了Runnable接口或Callable接口实现类的对象
 * 3.关闭线程池
 *
 * 【好处】
 * - 提高响应速度（减少了创建新线程的时间)
 * - 降低资源消耗（重复利用线程池中线程,不需要每次都创建)
 * - 便于线程管理
 *
 *
 *
 * 新手本例子可能出现的问题：http://www.voidcn.com/article/p-zkcigdhf-bud.html
 *
 */
public class ThreadPool {
    public static void main(String[] args) {
//        1.提供指定线程数量的线程池
        ExecutorService service = Executors.newFixedThreadPool(10);   // newFixedThreadPool方法是类Executors的静态方法.

//        设置线程池的属性, 这里
        System.out.println(service.getClass());  // class java.util.concurrent.ThreadPoolExecutor
        ThreadPoolExecutor service1 = (ThreadPoolExecutor)service; // 必须强转为这种形式才能设置属性
//        service1.setCorePoolSize(15);
//        service1.setKeepAliveTime(1000);

//        2.执行指定的线程的操作，需要提供实现了Runnable接口或Callable接口实现类的对象
        service.execute(new NumberThread());  //适合使用于Runnable
        service.execute(new NumberThread2());  //适合使用于Runnable
//        service.submit(Callable callable); //适合使用于Callable

//        3.关闭线程池
        service.shutdown();
    }
}

class NumberThread implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i <= 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + "偶数：" + i);

            }
        }
    }
}

class NumberThread2 implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i <= 100; i++) {
            if (i % 2 != 0) {
                System.out.println(Thread.currentThread().getName() + "奇数：" + i);

            }
        }
    }
}




```













---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E5%A4%9A%E7%BA%BF%E7%A8%8B/  

