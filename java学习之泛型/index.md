# Java学习之泛型


### 为什么要有泛型

> 泛型：标签

集合容器类在设计阶段/声明阶段不能确定这个容器到底实际存的是什么类型的对象，所以在**JDK1.5之前只能把元素类型设计为Object，JDK1.5之后使用泛型来解决**。因为这个时候除了元素的类型不确定，其他的部分是确定的，例如关于这个元素如何保存，如何管理等是确定的，因此此**时把元素的类型设计成一个参数，这个类型参数叫做泛型**。`Collection<E>`，`List<E>`，`ArrayList<E>`这个<E>就是类型参数，即泛型。

```
* 举例：
* 中药店，每个出题外面贴着标签
* 超市购物加上很多瓶子，每个瓶子装的是什么，有标签
```



#### 泛型的概念

- <font color=red>所谓泛型，就是允许在定义类、接口时通过一个标识表示类中某个属性的类型或者是某个方法的返回值及参数类型。这个类型参数将在使用时（例如，继承或实现这个接口，用这个类型声明变量、创建对象时）确定（即传入实际的类型参数，也称为类型实参)。</font>
- **从JDK1.5以后，Java引入**了“参数化类型(Parameterized type)”的概念，允许我们在创建集合时再指定集合元素的类型，正如: `List<String>`，这表明该List只能保存字符串类型的对象
- JDK1.5改写了集合框架中的全部接口和类，为这些接口、类增加了泛型支持，从而可以在声明集合变量、创建集合对象时传入类型实参。



**使用与不使用泛型的区别：**

![为何使用泛型](http://image.xpshuai.cn/%E4%B8%BA%E4%BD%95%E4%BD%BF%E7%94%A8%E6%B3%9B%E5%9E%8B.png)







### 在集合中使用泛型

```java
 /** 泛型的使用：
 *1.从JDK1.5以后新增的特性
 *
 *2.在集合中使用泛型：
 *      (1).集合接口或集合类在jdk5.0时都修改为带泛型的结构
 *      (2).在实例化集合类时，可以指明具体的泛型类型
 *      (3).指明完以后，在集合类或接口中凡是定义类或接口时，内部结构(比如方法、构造器、属性等)使用到类的泛型的位置，都指定为实例化的泛型类型
 *          比如：add(E e) --> 实例化时候: add(Integer e)
 *      (4).泛型的类型必须是类，不能是基本数据类型。需要用到基本数据类型的位置，拿包装类替换
 *      (5).如果实例化时，没有指定泛型的类型，默认类型为java.lang.Object类型
 *
 **/


    /*
    * 在集合中不使用泛型的情况
    * */
    @Test
    public void test(){
        ArrayList list = new ArrayList();
        // 存放学生成绩
        list.add(78);
        list.add(99);
        list.add(88);
        list.add(65);
        //问题1：类型不安全
        list.add("Tom");

        for(Object score: list){
            //问题2：强转时，可能出现ClassCastException
            int stuScore = (Integer) score;
            System.out.println(stuScore);
        }

    }

    /*
     * 在集合中使用泛型的情况: 以ArrayList为例
     * */
    @Test
    public void test1(){
        ArrayList<Integer> list = new ArrayList<Integer>();  // 用Integer而不是int，泛型是类型，不能用基本类型所以用包装类
        //这样传入的对象只能是Integer类型的了
        list.add(78);
        list.add(99);
        list.add(88);
        list.add(65);
        // 编译时，就会进行类型检查，保证数据的安全
//        list.add("Tom");

        //方式1：
        for(Integer score: list){
            // 避免了强转操作
            int stuScore = score;
            System.out.println(stuScore);
        }
        //方式2：
        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()){
            int nu = iterator.next();
            System.out.println(nu);
        }

    }

    /*
     * 在集合中使用泛型的情况: 以HashMap为例
     * */
    @Test
    public void test2(){
        Map<String, Integer> map = new HashMap<String, Integer>();
        // key-value两个类型都要指定
        map.put("TOm", 18);
        map.put("Tom", 20);
        map.put("Jerry", 18);
        
        
        //取
        //泛型的嵌套
        // Entry是Map里面的，没有暴露在外面
        Set<Map.Entry<String, Integer>> entries = map.entrySet();
        Iterator<Map.Entry<String, Integer>> iterator = entries.iterator();
        while (iterator.hasNext()){
            Map.Entry<String, Integer> entry = iterator.next();
            String key = entry.getKey();
            Integer val = entry.getValue();
        }




        // jdk7新特性：类型推断
        Map<String, Integer> map2 = new HashMap<>();  //后面可省略



    }

```





### 自定义泛型



自定义的泛型类：

```java
package cn.xpshuai.java1;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 14:28
 * @功能：自定义泛型结构：泛型类、泛型接口、泛型方法
 *
 * T,  E
 * k,v 一般为键值
 */
public class Order<T> {
    String orderName;
    int orderId;


    //类的内部结构就可以使用类的泛型
    T orderT;

    public Order(){
//        T[] arr = new T[10]; //编译不通过， 不能new
//        T[] arr = (T[]) new Object[10]; //这样可以，但是一般不这么用


    }

    public Order(String orderName, int orderId, T orderT) {
        this.orderName = orderName;
        this.orderId = orderId;
        this.orderT = orderT;

    }

    //如下上方法都不是泛型方法
    public T getOrderT(){
        return orderT;
    }

    public void setOrderT(T orderT){
        this.orderT = orderT;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderName='" + orderName + '\'' +
                ", orderId='" + orderId + '\'' +
                ", orderT=" + orderT +
                '}';
    }

    //静态方法中不能使用类的泛型
//    public static void show(T orderT){
//        System.out.println("show");
//    }

//    public static void show2(){
    // 编译不通过
//       try{
//
//       }catch (T t){
//
//       }
//    }


    //泛型方法：在方法中出现了泛型的结构，泛型参数与类的泛型参数没有任何关系
    //换句话说：泛型方达所属的类是不是泛型类都没有关系。
    public <E> List<E> copyFromArrayToList(E[] arr){  // E类型的数组, 前面也要加<E>
        ArrayList<E> list = new ArrayList<>();
        for(E e: arr){
            list.add(e);
        }
        return list;

    }

}

```

使用: 

```java
    @Test
    public void test3(){
        // 使用自定义的泛型类 Order.java
        // 如果定义了泛型类，实例化没有指明类的泛型，则认为此泛型类型为Object类型
        // 要求：如果定义了类是带泛型的，建议在实例化时要指明类的泛型。
//        Order order = new Order();
//        order.setOrderT(12);
//        order.setOrderT("AA");

        Order<String> order = new Order<String>("orderAA", 1001, "我是泛型小实例1");
        order.setOrderT("我是泛型小实例:2");
        System.out.println(order.getOrderT());

	
        //【泛型在继承上的体现】
        //如果子类指明了具体的泛型类型，使用子类实例化的时候就不用写泛型的形式了
        SubOrder sub = new SubOrder();
        //由于子类在继承带泛型的父类时，指明了泛型类型。则实例化子类对象时，不再需要指明泛型。
        sub.setOrderT(111);


        // 子类不具体父类中指明泛型的类型，SubOrder1<T>仍然是泛型类，使用子类实例化的时候仍然写泛型的形式了
        SubOrder1<String> sub1 = new SubOrder1<>();
        sub1.setOrderT("CCC");



    }

```

泛型类的子类：

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 14:40
 * @功能：Order的子类
 */

//这里如果指明了具体的泛型类型，使用子类实例化的时候就不用写泛型的形式了
public class SubOrder extends Order<Integer>{
    //......
}

```

泛型类的子类2:

```java
package cn.xpshuai.java1;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 14:46
 * @功能：这里的子类不具体父类中指明泛型的类型，SubOrder1<T>仍然是泛型类，使用子类实例化的时候仍然写泛型的形式了
 */
public class SubOrder1<T> extends Order<T> {
}

```





**自定义泛型类与泛型接口的注意点：**

1. 泛型类可能有多个参数，此时应将多个参数一起放在尖括号内。比如:<E1,E2,E3>

2. 泛型类声明的构造器如下:`public GenericClass()`。而下面是错误的:`public GenericClass<E>()`

3. 实例化后，操作原来泛型位置的结构必须与指定的泛型类型一致。

4. 泛型不同的引用不能相互赋值（尽管在编译时`ArrayList<String>和ArrayList<Integer>`是两种类型，但是，在运行时只有一个ArrayList被加载到JVM中）。

5. 泛型如果不指定，将被擦除，泛型对应的类型均按照Object处理，但不等价于Object。经验: 泛型要使用一路都用。要不用，一路都不要用。

6. 如果泛型结构是一个接口或抽象类，则不可创建泛型类的对象。

7. jdk1.7，泛型的简化操作:`ArrayList<Fruit> flist = new ArrayList<>()`;

8. 泛型的指定中不能使用基本数据类型，可以使用包装类替换。

9. 在类/接口上声明的泛型，在本类或本接口中即代表某种类型，可以作为非静态属性的类型、非静态方法的参数类型、非静态方法的返回值类型。但在<font color=red>静态方法中不能使用类的泛型</font>。

10. 异常类是不能声明为泛型的(`public calss MyExpection<T> extends Exception`是错误的)

11. 不能使用new E[]。但是可以:`E]elements =(E[])new Object[capacity];`
    参考: ArrayList源码中声明:Object[]elementData，而非泛型参数类型数组。

12. 父类有泛型，子类可以选择保留泛型也可以选择指定泛型类型:

    - 子类不保留父类的泛型: 按需实现
      - 没有类型：擦除
      - 具体类型
    - 子类保留父类的泛型:泛型子类
      - 全部保留
      - 部分保留

    **结论:**子类必须是“富二代”，子类除了指定或保留父类的泛型，还可以增加自己的泛型

```java
    @Test
    public void test4(){
//        泛型不同的引用不能相互赋值
    ArrayList<String> l1 = null;
    ArrayList<Integer> l2 = null;
//    l2 = l1; //不能相互赋值

    }



```

![子父类泛型](http://image.xpshuai.cn/%E5%AD%90%E7%88%B6%E7%B1%BB%E6%B3%9B%E5%9E%8B.png)



**泛型方法(重难点)：**

```java
package cn.xpshuai.java1;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 14:28
 * @功能：自定义泛型结构：泛型类、泛型接口、泛型方法
 *
 * T,  E
 * k,v 一般为键值
 */
public class Order<T> {
    String orderName;
    int orderId;


    //类的内部结构就可以使用类的泛型
    T orderT;

    public Order(){
//        T[] arr = new T[10]; //编译不通过， 不能new
//        T[] arr = (T[]) new Object[10]; //这样可以，但是一般不这么用


    }

    public Order(String orderName, int orderId, T orderT) {
        this.orderName = orderName;
        this.orderId = orderId;
        this.orderT = orderT;

    }

    //如下上方法都不是泛型方法
    public T getOrderT(){
        return orderT;
    }

    public void setOrderT(T orderT){
        this.orderT = orderT;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderName='" + orderName + '\'' +
                ", orderId='" + orderId + '\'' +
                ", orderT=" + orderT +
                '}';
    }

    //静态方法中不能使用类的泛型(当时还没有这个对象)
//    public static void show(T orderT){
//        System.out.println("show");
//    }

//    public static void show2(){
    // 编译不通过
//       try{
//
//       }catch (T t){
//
//       }
//    }


    //泛型方法：在方法中出现了泛型的结构，泛型参数与类的泛型参数没有任何关系
    //换句话说：泛型方达所属的类是不是泛型类都没有关系。
    //泛型方法可以声明为静态的。原因泛型参数是在调用方法时确定的，并非在实例化时确定的
    public <E> List<E> copyFromArrayToList(E[] arr){  // E类型的数组, 前面也要加<E>，不然编译器会认为E是我们自定义的一个类呢
//    public static <E> List<E> copyFromArrayToList(E[] arr){  // 这里可以加static的哦
        ArrayList<E> list = new ArrayList<>();
        for(E e: arr){
            list.add(e);
        }
        return list;

    }

}

```

```java
    //测试泛型方法
    @Test
    public void test5(){
        Order<String> order = new Order<String>("orderAA", 1001, "我是泛型小实例1");
        Integer[] arr = new Integer[]{1,2,3,4};
        // 泛型方法在调用时，指明泛型参数的类型（与泛型类的类型是没有关系的）
        List<Integer> list = order.copyFromArrayToList(arr);
        System.out.println(list);
        
    }
```

![自定义泛型方法](http://image.xpshuai.cn/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B3%9B%E5%9E%8B%E6%96%B9%E6%B3%95.png)





**泛型类和泛型方法情景举例：**

DAO(操作表的共性操作)

```java
package cn.xpshuai.java1;

import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 15:24
 * @功能：模拟
 * DAO: data(base) access object
 *   定义了一些通用的数据库的操作方法
 *
 * 数据库中一个表对应java的一个类
 *
 */
public class DAO <T>{  //

    //删除一条记录
    public void add(T t){

    }

    //删除一条记录
    public boolean remove(int index){
        return false;
    }

    //修改一条记录
    public void update(int index, T t){

    }

    //查询一条记录
    public T getIndex(int index){
        return null;
    }

    //查询多条记录
    public List<T> getList(int index){
        return null;

    }

    //泛型方法
    // 举例:获取表中一共有多少条记录?获取最大的员工入职时间?
    public <E> E getValue(){
        return null;
    }
}

```

子类(只能具体操作某个表的类):

```java
package cn.xpshuai.java1;

//这个继承DAO，写具体的对Customer的数据库的操作
public class CustomerDAO extends DAO<Customer>{  //专门操作Customer表
}

```

子类(某个数据表对应一个类):

```java
package cn.xpshuai.java1;

public class Customer {  //此类对应数据库中的一个表
}

```



操作：

```java
//测试例子(DAO)
@Test
public void testDao(){
    CustomerDAO dao1 = new CustomerDAO();
    dao1.add(new Customer());//
    List<Customer> list = dao1.getList(1);
}
```





### 泛型在继承上的体现

```
类A是类B的父类，G<A>和G<B>不具备子父类关系，二者是并列关系
补充：类A是类B的父类/接口，A<G>与<G>也具备子父类/接口关系
```

```java
package cn.xpshuai.java2;

import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 15:38
 * @功能：
 *
 * 1.泛型在继承方面的体现
 *
 *
 * 2.通配符的使用
 *
 *
 *
 *
 */
public class GenericTest {
    /*
    1.泛型在继承方面的体现
    类A是类B的父类，G<A>和G<B>不具备子父类关系，二者是并列关系
    补充：类A是类B的父类/接口，A<G>与<G>也具备子父类/接口关系

     */
    @Test
    public void test(){
        //在类上的展示, 类似多态的展示
        Object obj = null;
        String str = null;

        obj = str;

        //在数组上的展示
        Object[] arr1 = null;
        String[] arr2 = null;
        arr1 = arr2;

        // 编译不通过
        List<Object> l1 = null;
        List<String> l2 = null;
        //此时的l1和l2的类型不具有子父关系
//        l1 = l2;  //错误
    }

    @Test
    public void test1(){
        List<String> list1 = null;
        ArrayList<String> list2 = null;
        list1 = list2;

    }


}

```







### 通配符的使用

![通配符的使用](http://image.xpshuai.cn/%E9%80%9A%E9%85%8D%E7%AC%A6%E7%9A%84%E4%BD%BF%E7%94%A8.png)

```java
package cn.xpshuai.java2;

import org.junit.Test;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 15:38
 * @功能：
 *
 * 1.泛型在继承方面的体现
 *
 *
 * 2.通配符的使用
 *
 
 */
public class GenericTest {
    /*
    1.泛型在继承方面的体现
    类A是类B的父类，G<A>和G<B>不具备子父类关系，二者是并列关系
    补充：类A是类B的父类/接口，A<G>与<G>也具备子父类/接口关系

     */
    @Test
    public void test(){
        //在类上的展示, 类似多态的展示
        Object obj = null;
        String str = null;

        obj = str;

        //在数组上的展示
        Object[] arr1 = null;
        String[] arr2 = null;
        arr1 = arr2;

        // 编译不通过
        List<Object> l1 = null;
        List<String> l2 = null;
        //此时的l1和l2的类型不具有子父关系
//        l1 = l2;  //错误
    }

    @Test
    public void test1(){
        List<String> list1 = null;
        ArrayList<String> list2 = null;
        list1 = list2;

    }


    /*
    2.通配符的使用
    通配符：?

    类A是类B的父类，且G<A>和G<B>不具备子父类关系，二共同的父类是: G<?>

     */
    @Test
    public void test2(){
        List<Object> l1 = null;
        List<String> l2 = null;

        List<?> list = null;   //作为前两个的公共父类,实现公共的调用
        list = l1;
        list = l2;

        printer(l1);
        printer(l2);


        //
        List<String> list3 = new ArrayList<>();
        list3.add("AA");
        list3.add("BB");
        list3.add("CC");
        list = list3;
        //添加(写入)：对于List<?>，就不能向其内部添加数据了
        //除了添加null之外, 仅此而已
//        list.add("D");
        list.add(null);

        //获取(读取):允许读取数据，读取的数据类型为Object
        Object o = list.get(0);
        System.out.println(o); // 但是是允许读的


    }
    // 加上通配符，这样就可以调用一个公共的方法了
    public void printer(List<?> list){
        Iterator<?> iterator = list.iterator();
        while (iterator.hasNext()){
            Object obj = iterator.next();
            System.out.println(obj);
        }
    }

}

```



**有限制条件的通配符的使用**：

![有限制条件的通配符](http://image.xpshuai.cn/%E6%9C%89%E9%99%90%E5%88%B6%E6%9D%A1%E4%BB%B6%E7%9A%84%E9%80%9A%E9%85%8D%E7%AC%A6.png)

```java
    /*
    3.有限制条件的通配符的使用
        ? extends A:
            G<? extends A> 可以作为G<A>和G<B>的父类，其中B是A的子类
        ? super A:
            G<? super A> 可以作为G<A>和G<B>的父类, 其中G<B>为A的父类


     */
    @Test
    public void test3(){
        //以List为例
        List<? extends Person> l1 = null;  // <=
        List<? super Person> l2 = null;    // >=

        List<Students> l3 = null;
        List<Person> l4 = null;
        List<Object> l5 = null;


        l1 = l3;
        l1 = l4;
//        l1 = l5;

//        l2 = l3;
        l2 = l4;
        l2 = l5;

    }


```

```java
package cn.xpshuai.java2;

public class Students extends Person{
    
}

```





### 泛型应用举例

泛型嵌套：

![泛型嵌套举例](http://image.xpshuai.cn/%E6%B3%9B%E5%9E%8B%E5%B5%8C%E5%A5%97%E4%B8%BE%E4%BE%8B.png)



```bash
list1.forEach(System.out::println)	# jdk8新特性
```



---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E6%B3%9B%E5%9E%8B/  

