# Java学习之集合




> 这章很重要的哦



### Java集合框架概述

- 为了方便对多个对象的操作，就需要对对象进行存储
- 且使用Array存储对象方面存在一定**弊端**。而Java集合就像一种容器，可以动态地把**多个对象**的引用放入容器
- 数组在内存存储方面的特点:
    - 数组初始化以后，长度就确定了；
    - 数组声明的类型，就决定了进行元素初始化时的类型。
- 数组在存储数据方面的弊端:
    - 数组初始化以后，长度就不可变了，不便于扩展；
    - 数组中提供的属性和方法少，不便于进行添加、删除、插入等操作，且效率不高。同时无法直接获取存储元素的个数；
    - 数组存储的数据是有序的、可以重复的。---->存储数据的特点单一



**Java集合可分为Collection和 Map 两种体系:** 

- Collection接口:单列数据，定义了存取一组对象的方法的集合
- List: 元素有序、可重复的集合
- Set: 元素无序、不可重复的集合
- Map接口:双列数据，保存具有映射关系“key-value对”的集合



**Java集合框架概述：**

![collection接口继承树](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/collection%E6%8E%A5%E5%8F%A3%E7%BB%A7%E6%89%BF%E6%A0%91.png)



![map接口继承树](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/map%E6%8E%A5%E5%8F%A3%E7%BB%A7%E6%89%BF%E6%A0%91.png)





```java
/**
 *学完之后，最基本要知道，什么类型的数据适合用什么样的集合存储
 *
 * 一、集合框架的概述
 * 1.集合、数组都是对多个数据进行存储操作的结构，简称Java容器。
 *此时的存储，主要指的是内存层面的存储，不涉及持久化的存储(.txt，jpg，avi，数据库...)
 *
 * 2.数组在存储多个数据方面的特点：
 *  > 一旦初始化，其长度就确定了
 *  > 数组一旦定义好，其元素的类型也就确定了,如元素类型:String[] arr2; int[] arr1; Object[] arr2;
 *
 * 3.相比集合，数组在存储多个数据方面的特点：
 *  > 一旦初始化，其长度就确定了, 无法扩容
 *  > 数组中提供的方法非常有限，对于添加删除插入元素的操作非常不方便，同时效率不高
 *  > 获取数组中实际元素的个数，没有现成的属性或方法
 *  > 数组存储数据的特点: 有序、可重复，对于无序、不可重复的需求不可满足
 *
 *
 *  二、集合框架
 *  | ----- Collection接口: 单列数据，定义了存取一组对象的方法的集合
 *        |---- List接口("动态数据"): 存储元素有序、可重复的数据
 *             |---- ArrayList、LinkedList、Vector(这三个面试多)
 *        |---- Set接口(类似高中讲的"集合":无序、确定性、互异性): 存储元素元素无序、不可重复
 *             |---- HashSet、LinkedHashSet、TreeSet
 * | ----- Map接口: 双列数据，保存具有映射关系的"key-value"对的集合(类似高中函数: y = f(x)，不同key可以指向相同的value)
 *             |---- HashMap(面试问得多)、LinkedHashMap、TreeMap、Hashtable、Properties
 *             
 *
 * 
 *
 */
```





### <font color=red>Collection接口</font>

> 单列数据，定义了存取一组对象的方法的集合



#### Collection接口的方法



```java
/* 
*三、Collection接口中的方法
 * 
 * 结论：
 *向Collection接口的试下类的接口中添加数据obj时，要求boj所在类要重写equals()
 */

public class MyCollectionTest {
/*
    Collection接口中的方法
 */
    @Test
    public void test1(){
        //这里使用ArrayList用多态的方法来做的测试
        Collection coll = new ArrayList();  // ArrayList 是有序的

        //1.add(Object e): 将元素e添加到集合中
        coll.add("AA");
        coll.add("BB");
        coll.add(123); //自动装箱
        coll.add(new Date());
        coll.add(new Person("Tom", 20));

        //2.size(): 获取添加的元素的个数
        System.out.println(coll.size());

        //3.addAll: 把另一集合中的元素添加到当前的集合中
        Collection coll2 = new ArrayList();
        coll.add(456);
        coll.add("CC");
        coll.addAll(coll2);
        System.out.println(coll);


        //4.isEmpty(): 判断当前集合是否为空(是否有元素, size是否为0)
        System.out.println(coll.isEmpty());

        //5.clear():清空集合元素
        coll2.clear();
        System.out.println(coll2);

        //6.contains(Object obj): 判断当前集合是否包含obj, 判断的是内容而不是地址
        //在判断时，会调用obj对象所在类的equals()，要求我们自己重写一下
        boolean contains = coll.contains(123);
        System.out.println(contains);
        System.out.println(coll.contains(new String("Tom"))); // true, 判断的是内容,  调用的equals方法
        System.out.println(coll.contains(new Person("Tom", 20))); // false, 但是如果我们重写了Person的equals方法，然后就可以按照自己的想法变为true了

        // 7.containsAll(Collection coll): 判断形参中的所有元素是否都存在于当前集合中
        // 跟contains的判断是相等的，都跟实现类中的equals()有关
        Collection coll3 = Arrays.asList(123,456);
        coll.containsAll(coll3);

        // 8.remove(Object obj): 从集合中移除某个元素, 成功返回true
        // 仍然需要重写equals()方法，因为在移除之前需要判断元素是否在集合中
        coll.remove(123);
        coll.remove(new Person("Tom", 20)); // 在Person中需重写equals()

//        9.removeAll(Collection coll4): 从当前集合中移除某个集合中的元素(差集)
        coll.removeAll(coll3);
        System.out.println(coll);

//        10.retainAll(Collection coll4):返回布尔型的值，修改当前集合为两个集合的交集
        Collection coll4 = Arrays.asList(132,789,123);
        coll.retainAll(coll4);

//        11.equals(Object bj): 比较两个集合是否相等，依次逐个元素比较(这里ArrayList是有序的，所以顺序不一样也不行)
        System.out.println(coll.equals(coll3));

//        12.hashCode(): 返回当前对象的hash值
        System.out.println(coll.hashCode());

//        13.集合 --> 数组: toArray()
        Object[] arr = coll.toArray();
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }

//        14.数组 --> 集合: 调用Arrays类的静态方法asList()
        List<String> li = Arrays.asList(new String[]{"AA", "BB", "CC"});
        System.out.println(li);
        //Note: 使用时候注意
        List arr1 = Arrays.asList(new int[]{111,222}); //这种写法会被当做一个元素
        List arr2 = Arrays.asList(111,222); //直接写这就行
        List arr3 = Arrays.asList(new Integer[]{111,222}); //或者用Integer
        System.out.println(arr1.size());
        System.out.println(arr2.size());
        System.out.println(arr3.size());

//        15.iterator: 返回Iterator接口的实例，用于遍历集合元素，下一部分来测试


    }

}


```





#### lterator迭代器接口

- lteralor对象称为迭代器(设计模式的一种)，主要用于遍历Collection集合中的元素。
- GOF给迭代器模式的定义为:提供一种方法访问一个容器(container)对象中各个元素，而又不需暴露该对象的内部细节。**迭代器模式，就是为容器而生**。类似于“公交车上的售票员”、“火车上的乘务员”、“空姐”。
- Collection接口继承了`java.lang.lterable`接口，该接口有一个`iterator()`方法，那么所有实现了Collection接口的集合类都有一个iterator()方法，用以返回一个实现了lterator接口的对象。
- **Iterator仅用于遍历集合，lterator木身并不提供承装对象的能力**。如果需要创建lterator对象，则必须有一个被迭代的集合。
- **集合对象每次调用iterator()方法都得到一个全新的迭代器对象**，默认游标都在集合的第一个元素之前。

![迭代器的执行原理](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E8%BF%AD%E4%BB%A3%E5%99%A8%E7%9A%84%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86.png)



**Iterator迭代器接口remove()方法**

```java
Iterator iterator2 = coll.iterator();
while (iterator2.hasNext()){
    Object obj = iterator2.next();  
    if("Tom".equals(obj)){
        iterator2.remove();  
    }
}
```



**Note:**

- lterator可以删除集合的元素，但是是遍历过程中通过迭代器对象的remove方法，不是集合对象的remove方法。

- 如果还未调用next)或在上一次调用next方法之后已经调用了remove方法，再调用remove都会报llegalStateException。

  

```java
/*
    集合元素的遍历操作，使用Iterator(迭代器接口)
    1.内部的方法：hasNext()与next()配合去使用
    2.每次调用iterator()，就会是一个全新的迭代器对象，指针永远指向开头
    3.内部定义了remove(),可以在遍历的时候，删除集合中的元素。此方法不同于集合直接使用remove()
    4.在调用next()之前，不要调用remove()；移除集合中某元素的操作, 不能写两次
 */
    @Test
    public void test2(){
        Collection coll = new ArrayList();
        coll.add("AA");
        coll.add("BB");
        coll.add(123);
        coll.add(new Date());
        coll.add(new Person("Tom", 20));

        Iterator iterator = coll.iterator();   // 装东西的还是原来的集合，Iterator并不是容器，只是迭代器
        // next()
//        System.out.println(iterator.next());
        // 不推荐这么写
//        for (int i = 0; i < coll.size(); i++) {
//            System.out.println(iterator.next());
//        }

        // 【推荐写法】:
        // hasNext() 与 next()配合
        while (iterator.hasNext()){
            // next(): 1.指针先下移；2.将下移以后集合位置上的元素返回
            System.out.println(iterator.next());
        }


        //错误写法1:
//        while (iterator.next() != null){  //这里先指针下移了，所以会跳着输出，且会出现NoSuchElementException
//            System.out.println(iterator.next());
//        }

        //错误写法2：
        // 每次调用iterator()，就会是一个全新的迭代器对象，指针永远指向开头
//        while ((coll.iterator()).hasNext()){  //死循环，不断输出第一个元素
//            System.out.println(iterator.next());
//        }


        //使用Iterator中的remove()
        Iterator iterator2 = coll.iterator();
        while (iterator2.hasNext()){
            Object obj = iterator2.next();  // 在调用next()之前，不要调用remove()
            if("Tom".equals(obj)){
                iterator2.remove();  //移除集合中某元素的操作, 不能写两次
            }
        }
        //然后再查看一下remove之后的元素
        Iterator iterator3 = coll.iterator();
        while (iterator3.hasNext()){
            System.out.println(iterator3.next());
        }

    }

```



#### foreach循环遍历集合和数组元素

```java
/*
    foreach 循环遍历集合和数组元素
    jdk5新增

     */
@Test
public void test3(){
    Collection coll = new ArrayList();
    coll.add("AA");
    coll.add("BB");
    coll.add(123);
    coll.add(new Date());
    coll.add(new Person("Tom", 20));

    //        foreach 循环遍历集合
    // 格式: for(集合/数组中元素的类型 局部变量: 集合/数组对象)
    // 内部仍然调用了迭代器
    for(Object obj: coll){
        System.out.println(obj);

    }


    // foreach 循环遍历数组
    int[] arr1 = new int[]{11,22,33,44,55};
    for(int i: arr1){
        System.out.println(i);
    }


    //练习题：
    java.lang.String[] arrr = new java.lang.String[]{"MM", "MM", "MM"};
    //F1:普通for循环
    for (int i = 0; i < arrr.length; i++) {
        arrr[i] = "GG";
    }

    for (int i = 0; i < arrr.length; i++) {
        System.out.println(arrr[i]);   // GG
    }

    // F2:foreach
    for(java.lang.String s: arrr){
        s ="GG";  //取出来，给s，s把它给改了，不会修改原有数组的值
    }

    for (int i = 0; i < arrr.length; i++) {
        System.out.println(arrr[i]);   // MM
    }


}

```





### 

#### Collection子接口一: <font color=red>List(重点)</font>

> 元素有序、可重复的集合
>
> 看做"动态数组"，以后可以用这个来代替数组啦~



**List接口概述：**

- 鉴于Java中数组用来存储数据的局限性，我们通常使用List替代数组
- List集合类中元素有序、且可重复，集合中的每个元素都有其对应的顺序索引。
- List容器中的元素都对应一个整数型的序号记载其在容器中的位置，可以根据序号存取容器中的元素。
- JDK API中List接口的实现类常用的有: ArrayList、LinkedList和Vector。

![JavaList实现类2-LinkedList](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/JavaList%E5%AE%9E%E7%8E%B0%E7%B1%BB2-LinkedList.png)



**源码分析(自己看)**



**List接口常用方法：**

 除去Collection这种的15个，还有下面几个

```java
【总结：常用方法】
    增：add(Object obj)
    删：remove(int index) /remove(Object obj)
    改：set(int index, Object ele)
    查：get(int index)
    插：add(index, Object ele)
    长度：size()
    遍历：1. Iterator迭代器
         2. foreach
         3. 普通的循环
```

如下：

```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.util.*;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-26 16:44
 * @功能： Collection子接口一: List
 *
 * 一、list源码框架
*        |---- List接口: 存储元素有序、可重复的数据 --> "动态"数组，替换原有数组
*             |---- ArrayList: 作为List接口的主要实现类(一般就用它):线程不安全的(但是用的时候一般都扔到synchronizedList()里面就线程安全了)，效率高：底层使用Object[]存储
 *            |----LinkedList: 其次使用的, 对于频繁的插入和删除操作，使用此类效率比ArrayList高，底层使用双向链表存储
 *            |----Vector: 作为List接口的古老实现类: 线程安全的，效率低：底层使用Object[] elemData存储
 *
 * List常用的接口异同：
 * - ArrayList
 * - LinkedList
 * - Vector
 * 同：都实现了list接口，存储数据的特点相同：存储有序的、可重复的数据
 * 不同：
 *
 *
 * 二、源码分析：
*   1.ArrayList:
 *   jdk7情况下:
 *   ArrayList list = new ArrayList(); //底层创建了长度是10的Object[]数组elementData
 *   list.add(123);//elementData[0]=new Integer(123);
 *   ...
 *   list.add(11);//list.add(11);//如果此次的添加导致底层elementData数组容量不够，则扩容。默认情况下，扩容为原来的容量的1.5倍, 同时需要将原有数组中的数据复制到新的数组中
 *   结论：建议开发中使用带参的构造器: ArrayList list = new ArrayList(int capacity);
 *
 *   jdk8下变化:
 * ArrayList coll1 = new ArrayList();//底层object[]elementData初始化为{}.并没有创建长度为10的数组
 * 第一次调用add()时，才创建长度为10的数组
 * 后续的添加和扩瞳操作与jdk7无异
 *
 * 小结：jdk7中的ArrayList的对象的创建类似于单例的饿汉式，而jdk8中的ArrayList的对象的创建类似于单例的蓝汉式，节省了内存
 *
 *
*   2.LinkedList:
 *LinkedList coll2 = new LinkedList();
 *
 *
*   3.Vector:
 *
 *
 *
 *
 * 三、list接口中常用方法
 * 除去Collection这种的15个，还有下面几个
 *
 */
public class ListTest {
    @Test
    public void test1(){
        ArrayList coll1 = new ArrayList();
        LinkedList coll2 = new LinkedList();
        Vector coll3 = new Vector();

    }

    /*
    list接口中常用方法
    除去Collection这种的15个，还有下面几个


   【总结：常用方法】
    增：add(Object obj)
    删：remove(int index) /remove(Object obj)
    改：set(int index, Object ele)
    查：get(int index)
    插：add(index, Object ele)
    长度：size()
    遍历：1. Iterator迭代器
         2. foreach
         3. 普通的循环


     */
    @Test
    public void test2(){
        ArrayList l1 = new ArrayList(); //就以ArrayList为例
        l1.add(132);
        l1.add(132);
        l1.add("aaa");
        l1.add(new Person("Lisa", 18));
        System.out.println(l1);

        // void add(int index, Object obj): 在指定位置插入元素
        l1.add(2, "BB");

        // boolean addAll():在index位置开始将ollention的元素插入
        List a1 = Arrays.asList(1, 2, 3);
        l1.addAll(a1);
        System.out.println(l1.size());

        // Object get(int index): 获取指定位置元素
        System.out.println(l1.get(0));

        // int indexOf(): 获取元素首次出现的位置，如果不存在返回-1
        int index = l1.indexOf(132);
        System.out.println(index);
        // int lastIndexOf(): 获取元素末次出现的位置，如果不存在返回-1
        System.out.println(l1.lastIndexOf("BB"));

        // Object remove(int index): 移除指定index位置的元素，并返回此元素
        Object b = l1.remove(0);
        System.out.println(b);
        System.out.println(l1);

        //Object set(int index, Object ele): 返回指定idnex位置的元素为ele
        l1.set(1, "CCC");
        System.out.println(l1);

        // List subList(int fromIndex, int toIndex): 从集合中返回一个左闭右开的子集合
        List l2 = l1.subList(1,6);  // 不会对本身list造成影响
        System.out.println(l2);



        // 遍历
//        F1: Iterator迭代器
        Iterator iterator = l1.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }

//        F2: foreach
        for(Object obj: l1){
            System.out.println(obj);
        }

//        F3:普通for循环
        for (int i = 0; i < l1.size(); i++) {
            System.out.println(l1.get(i));
        }
    }


    @Test
    public void test3(){
        
    }

}

```



**面试题：**

区分list中remove()方法

```java
@Test
public void test1() {
    List list = new ArrayList();
    list.add(1);
    list.add(2);
    list.add(3);
    updateList(list);
    System.out.println(list);
}

public void updateList(List list){
    //        list.remove(2); // 这样remove的是索引为2的元素
    list.remove(new Integer(2)); //这样通过封装类的方式, remove的是元素值为2的元素
}
```





#### Collection子接口二:<font color=red>Set</font>

> 元素无序、不可重复的集合
>
> 类似高中讲的"集合": 无序性、确定性、互异性



**Set接口概述：**

- Set接口是Collection的子接口，set接口没有提供额外的方法
- Set集合不允许包含相同的元素，如果试把两个相同的元素加入同一个Set集合中，则添加操作失败。
- Set判断两个对象是否相同不是使用==运算符，而是根据equals()方法



![image-20210127110226609](C:\Users\19401\AppData\Roaming\Typora\typora-user-images\image-20210127110226609.png)

**Set实现类之一: Hashset**

- HashSet是 Set接口的典型实现，大多数时候使用Set集合时都使用这个实现类;

- HashSet 按Hash算法来存储集合中的元素，因此具有很好的存取、查找、删除性能。

- HashSet具有以下特点:

- 不能保证元素的排列顺序
- HashSet不是线程安全的
- 集合元素**可以是null**

- **HashSet集合判断两个元素相等的标准**:两个对象通过 hashCode()方法比较相等，并且两个对象的equals()方法返回值也相等。

- 对于存放在Set容器中的对象，**对应的类一定要重写equals()和hashCode(Objectobj)方法，以实现对象相等规则**。即:“**相等的对象必须具有相等的散列码**”。

  

**重写hashCode()方法的基本原则:** 

- 在程序运行时，同一个对象多次调用hashCode()方法应该返回相同的值。
- 当两个对象的equals()方法比较返回 true 时，这两个对象的hashCode()方法的返回值也应相等。
- 对象中用作equals()方法比较的Field，都应该用来计算 hashCode值。



**Set实现类之二: LinkedkashSet: **

- LinkedHashSet是 HashSet的子类
- LinkedHashSet根据元素的hashCode值来决定元素的存储位置，但它同时使用**双向链表**维护元素的次序，这使得元素看起来是**以插入顺序保存**的。
- **LinkedHashSet插入性能略低于HashSe**t，但在迭代访问Set里的全部元素时有很好的性能。
- LinkedHashSet不允许集合元素重复。



**Set实现类之三:TreeSet: **

- TreeSet是 SortedSet接口的实现类，TreeSet可以确保集合元素处于排序状态。

- TreeSet底层使用**红黑树**结构存储数据

- TreeSet两种排序方法: **自然排序**和**定制排序**。默认情况下，TreeSet采用自然排序。

- 新增的方法如下:(了解)

  ```java
  Comparator comparator()
  Object first()
  Object last()
  Object lower(Object e)
  Object higher(Object e)
  SortedSet subSet(fromElement, toElement)
  SortedSet headSet(toElement)
  SortedSet tailSet(fromElement)
  ```



代码如下：

```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.util.*;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-27 10:14
 * @功能：
 *
 * 一、set接口的框架结构(使用的有点少)
 *  *  | ----- Collection接口: 单列数据，定义了存取一组对象的方法的集合
     *    |---- Set接口(类似高中讲的"集合":无序、确定性、互异性): 存储元素元素无序、不可重复
     *        |---- HashSet: 作为set接口的主要实现类；线程不安全，可存储null值
 *                 |---- LinkedHashSet: 作为hashset的子类: 遍历其内部数据时，可以按照添加的顺序遍历
 *            |---- TreeSet: 可以按照添加对象的指定属性进行排序
 *
 *
 * Note:
 * Set接口中没有额外定义新的方法，使用的都是collection中声明过的方法。
 *要求:重写的hashCode()和equals()尽可能保持一致性:相等的对象必须具有相等的散列码
 *     重写两个方法的小技巧：对象中用作equals()方法比较的 Field，都应该用来计算hashCode值
 *对于频繁的遍历操作，LinkedHashSet效率优于HashSet
 *
 *
 *
 * 二、set的无序性和不可重复性
 * 以HashSet为例说明:
 *1.无序性
 *无序性 != 随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的hash值决定的
 *
 *
 * 2.不可重复性:保证添加的元素按照equals()判断时，不能返回true。即相同的元素只能添加一个
 *
 *
 * 二、添加元素的过程(以HashSet为例)
 * 我们向HashSet中添加元素a,首先调用元素α所在类的hashcode()方法，计算元素α的哈希值，
 * 此哈希值接着通过某种算法计算出在HashSet底层数组中的存放位置（即为:索引位置)，
 * 判断教组此位置上是否已经有元素:
 *     如果此位置上没有其他元素，则元素a添加成功。--->情况1
 *     如果此位置上有其他元素b(或以链表形式存在的多个元素〉，则比较元素a与元素b的hash值:
 *         如果hash值不相同，则元素α添加成功。--->情况2
 *         如果hash值相同，进而需要调用元素α所在类的equals()方法:
 *             equals()返回true,元素α添加失败
 *             equals()返回false,则元素a添加成功。--->情况2
 * 对于添加成功的情况2和情况3而言:元素α与已经存在指定索引位置上数据以链表的方式存储。
 * jdk 7∶元素a放到数组中，指向原来的元素。
 * jdk 8 :原来的元素在数组中，指向元素d
 *总结：七上八下
 *
 *HashSet底层：数组+链表
 *
 *
 *LinkedHashSet:
 *
 *
 *TreeSet
 *
 *
 *
 */
public class SetTest {
    @Test
    public void test1(){
        //1.无序性
        // 2.不可重复性
        Set set1 = new HashSet(); //以hashset为例
        set1.add(132);
        set1.add(465);
        set1.add("AA");
        set1.add(new Person("Jack", 18));
        set1.add(666);

        Iterator iterator = set1.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());  // 顺序跟添加的顺序是一样的
        }



        // LinkedHashSet的使用
        // LinkedHashSet作为HashSet的子类，在添加数据的同时，每个数据还维护了两个引用，记录此数据和后一个数据
        //优点：对于频繁的遍历操作，效率优于HashSet
        Set set2 = new LinkedHashSet(); //LinkedHashSet
        set2.add(132);
        set2.add(465);
        set2.add("AA");
        set2.add(new Person("Jack", 18));
        set2.add(666);

        Iterator iterator2 = set1.iterator();
        while (iterator2.hasNext()){
            System.out.println(iterator2.next());  // 顺序跟添加的顺序是一样的
        }


    }


    /*
    1.向TreeSet中添加的数据，要求是相同类的对象
    2.两种排序方式：自然排序(实现Comparable接口)、定制排序(Comparator)
    3.自然排序中，比较两个对象是否相同的标准为：compareTo()返回0，不再是equals()
    4.自然排序中，比较两个对象是否相同的标准为：compare()返回0，不再是equals()

     */
    @Test
    public void test2(){
//        TreeSet
        TreeSet set = new TreeSet();
//        set.add(123);
//        set.add(456);
//        set.add(5);
//        set.add(56);
//        set.add(-6);
//        set.add("AAA"); //不能添加不同类的对象

          set.add(new Person("Tom", 18));
          set.add(new Person("Tom", 21));  //两个相同的
          set.add(new Person("Lisa", 20));
          set.add(new Person("Jack", 17));
          set.add(new Person("Jerry", 25));

          //自然排序
          Iterator iterator = set.iterator();
          while (iterator.hasNext()){
              System.out.println(iterator.next());
          }


          // 定制排序：Comparator
          Comparator com = new Comparator() {
              //按照年龄从小到大排序
              @Override
              public int compare(Object o1, Object o2) {
                  if (o1 instanceof Person && o2 instanceof Person){
                      Person p1 = (Person)o1;
                      Person p2 = (Person)o2;
                      return Integer.compare(p1.getAge(), p2.getAge());

                  }else{
                      throw new RuntimeException("输入的数据类型不匹配");
                  }
              }
          };

          TreeSet set2 = new TreeSet();
          set2.add(new Person("Tom", 18));
          set2.add(new Person("Tom", 21));  //两个相同的
          set2.add(new Person("Lisa", 20));
          set2.add(new Person("Jack", 17));
          set2.add(new Person("Jerry", 18));
    }
}

```



```java
 package cn.xpshuai.java1;


import java.util.Objects;

class Person implements Comparable{
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
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

    public void walk(){
        System.out.println("走路");
    }

    public void eat(){
        System.out.println("吃饭");
    }


    @Override
    public boolean equals(Object o) {
        System.out.println("调用了equals方法");
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age &&
                Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    //重写、按照姓名从小到大排序
    @Override
    public int compareTo(Object o) {
        if(o instanceof Person){
            Person person = (Person)o;
//            return this.name.compareTo(person.name);

            int compare = this.name.compareTo(person.name);
            if (compare != 0){
                return compare;
            }else { //二级排序, 若姓名相同则再比较age
                return Integer.compare(this.age, person.age);
            }
        }else{
            throw new RuntimeException("输入的类型不匹配");
        }
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







![set实现类-HashSet](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/set%E5%AE%9E%E7%8E%B0%E7%B1%BB-HashSet.png)



![编译器中hashCode的重写](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E7%BC%96%E8%AF%91%E5%99%A8%E4%B8%ADhashCode%E7%9A%84%E9%87%8D%E5%86%99.png)

![TreeSet红黑树](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/TreeSet%E7%BA%A2%E9%BB%91%E6%A0%91.png)





**练习1：**

HashSet可以用来去重

![HashSet去重练习](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/HashSet%E5%8E%BB%E9%87%8D%E7%BB%83%E4%B9%A0.png)





HashSet先HashCode再equals()





### <font color=red>Map接口(重点)</font>

> 双列数据，保存具有映射关系的"key-value"对的集合
>
> 类似高中函数: y = f(x)，不同key可以指向相同的value



**Map接口继承树：**

![map接口继承树](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/map%E6%8E%A5%E5%8F%A3%E7%BB%A7%E6%89%BF%E6%A0%91.png)





**Map结构的理解:** 

![Map结构的理解](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/Map%E7%BB%93%E6%9E%84%E7%9A%84%E7%90%86%E8%A7%A3.png)



**常见面试题：**

```java
谈谈你对HashMap中put/get方法的认识?如果了解再谈谈HashMap的扩容机制?默认大小是多少?什么是负载因子(或填充比)?什么是春吐临界值(或阈值、threshold)?
```



**HashMap源码中重要常量：**

![HashMap源码中重要常量](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/HashMap%E6%BA%90%E7%A0%81%E4%B8%AD%E9%87%8D%E8%A6%81%E5%B8%B8%E9%87%8F.png)

![HashMap内部类Node,Entry](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/HashMap%E5%86%85%E9%83%A8%E7%B1%BBNode%2CEntry.png)



**Map中常用的方法:** 

![Map接口中常用的方法](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/Map%E4%B8%AD%E5%B8%B8%E7%94%A8%E7%9A%84%E6%96%B9%E6%B3%95.png)



```
package cn.xpshuai.java1;

import org.junit.Test;

import java.util.*;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-27 16:22
 * @功能：
 *
 *
 *一、Map的结构体系
 * | ----- Map接口: 双列数据，保存具有映射关系的"key-value"对的集合(类似高中函数: y = f(x)，不同key可以指向相同的value)
 *      |---- HashMap(面试问的多, 平时也常用): 作为Map的主要实现类; 线程不安全的，效率高；存储null的key和value
 *          |---- LinkedHashMap:保证在遍历map元素时，可以按照添加的循序实现遍历
 *                              原因：在原有的HashMap的基础上，添加了一堆指针，指向前一个和后一个元素
 *                              对于频繁的遍历操作，此类执行效率高于HashMap
 *      |---- TreeMap: 保证按照添加的key-value进行排序，实现排序遍历(此时考虑key的自然排序和定制排序)
 *      |---- Hashtable: 作为古老的实现类；线程安全的，效率低；不能存储null的key和value
 *          |---- Properties: 常用来处理配置文件。key和value都是String类型
 *
 *
 *  HashMap的底层：数组+链表(jdk7及之前)
 *                 数组+链表+红黑树(jdk8)
 *
 *
 *二、Map结构的理解
 * Map中的key: 无序的、不可重复的，使用Set存储所有的key  ---> key所在的类要重写equaLs( )和hashcode() （以HashMap为例）
 * Map中的value：无序的、可重复的，使用Collection存储所有的value  -->vaLue所在的类要重写equals()
 * 一个键值对：key-value构成了一个Entry对象
 * Map中的Entry: 无序的、不可重复的，使用Set存储所有的entry
 *
 *
 *三、HashMap的底层实现原理
 * jdk7为例：
 * HashMap map = new HashMap();
 * 在实例化以后，底层创建了长度为16的以为数组Entry[] table
 * ... 可能已经执行了多次put ...
 * map.push(key1, value1);
 * 首先，计算key1所在类的hashCode()，计算key1的哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置
 * 如果此位置上的数据为空，此时的key1-value1添加成功  ---> 情况1
 * 如果此位置上的数据不为空(此位置存在一个或多个数据(以链表形式存在)), 比较key1和已经存在的一个或多个数据的hash值：
 *      如果key1的hash值与已存在的数据的hash值都不相同，此时key1-value1添加成功       ---> 情况1
 *      如果key1的hash值与已存在的数据的某一个数据(key2-value2)的hash值都不相同，继续比较: 调用key1所在类的equals()方法，比较:
 *          如果equals()返回false:此时key1-value1添加成功     ---> 情况3
 *          如果equals()返回true:使用value1替换相同key的value2值，
 *          key1-value1添加成功
 * 补充：关于情况2和情况3: 此时key1-value1和原来的数据以链表的方式存储
 *
 * 在不断的添加过程中，会涉及到扩容问题, 当超出临界值(且要存放的位置非空)时，扩容。默认的扩容方式为扩容到原来容量的2倍，并将原有数据复制过来
 *
 * jdk8 相较于jdk7，在底层实现方面的不同；
     * 1.new HashMap()： 底层没有创建一个长度为16的数组
     * 2.jdk8底层的数据是 Node[], 而非Entry[]
     * 3.首次调用put()方法时，底层创建长度为16的数组
     * 4.jdk7底层结构只有：数组+链表, jdk8中底层结构:数组+链表+红黑树
     *      当数组的某一个索引位置上的元素以链表形式存在的数据个数>8 且当前数组的长度>64时，
     *       此时此索引位置上所有数据改为红黑树存储
 *
 *
 *HashMap源码分析:
 * jdk7
 * jdk8
 *
 *
 * DEFAULT_INITIAL_CAPACITY : HashMap的默认容量，16
 * DEFAULT_LOAD_FACTOR: HashMap的默认加载因子:0.75
 * threshold:扩容的临界值，=容量*填充因子:16* 0.75 =>12
 * TREEIFY_THRESHOLD: Bucket中链表长度大于该默认值，转化为红黑树:8
 * MIN_TREEIFY_CAPACITY:桶中的Node被树化时最小的hash表容量:64
 *
 *
 *
 * 面试题:
 * 1.HashMap的底层实现原理(重点)
 * 2.HashMap和Hashtable区别
 * 3.CurrentHashMap与Hashtable区别
 *
 *
 *
 * 四、LinkedHashMap的底层实现:
 *    static class Entry<K,V> extends HashMap.Node<K,V> {
 *         Entry<K,V> before, after;  //能够记录添加的元素的先后顺序
 *         Entry(int hash, K key, V value, Node<K,V> next) {
 *             super(hash, key, value, next);
 *         }
 *     }
 *
 *
 *
 * 五、Map中常用的方法
 *
 *
 * 六、TreeMap
 * //向TreeMap中添加key-value，要求key必须是由同一个类创建的对象
 * //因为要按照key进行排序:自然排序、定制排序
 *
 *
 *
 */
public class MapTest {
    @Test
    public void test1(){
    Map map = new HashMap();
//    map = new HashMap();
        map.put(null, 123);



    }

    @Test
    public void test2(){
        HashMap map = new HashMap();
//    map = new HashMap();
        map.put(null, 123);

    }
    @Test
    public void test3(){
        HashMap map = new HashMap();
        map = new LinkedHashMap(); // 记录了添加的顺序
        map.put(123, "AA");
        map.put(124, "BB");
        map.put(567, "CC");
        System.out.println(map);

//        LinkedHashMap
    }

    /*
    Map中常用的方法
     */
    @Test
    public void tes4(){
        // Object put(Object key, Object value):将制定键值对添加(修改)到当前map对象中

        // void putAll(Map m): 将m中所有键值对存放到当前map中
        // Object remove(Object key): 移除指定key的键值对，并返回value
        // void clear(): 清空当前map中的所有数据

        HashMap map = new HashMap();
        //put 添加
        map.put(123, "AA");
        map.put("BB", "222");
        map.put(567, "CC");
        map.put(56, "CCC");
        // put修改
        map.put(56, "566666");

        //putAll()
        HashMap map2 = new HashMap();
        map2.put(1, "AA");
        map2.put(2, "AB");
        map.putAll(map2);

        //remove
        Object val = map.remove("BB");
        System.out.println(val);
        System.out.println(map);

        //clear
//        map.clear();
        System.out.println(map.size());

        //元素查询的操作:
        //object get(0bject key):获取指定key对应的vaLue
        System.out.println(map.get(45));
        //boolean containsKey (object key):是否包含指定的key
        System.out.println(map.containsKey(45));
        //boolean containsValue(object value):是否包含指定的value
        // int size():返回map 中key-value对的个数
        //boolean isEmpty():判断当前map是否为空
        System.out.println(map.isEmpty());
        //boolean equals(object obj):判断当前map和参数对象obj是否相等
        System.out.println(map.equals(map2));


//        元视图操作的方法(遍历):
//        set keySet():返回所有key构成的Set集合
        // 遍历所有key
        Set set = map.keySet();
        Iterator iterator = set.iterator(); //变成set了，就可以用迭代器了
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }

//        collection values():返回所有value构成的collection集合set entryset():返回所有key-value对构成的Set集合
        Collection values = map.values();
        for(Object obj: values){ // 变成collections了，就可以用foreach了
            System.out.println(obj);
        }

        //遍历所有的key-value:
        // 方式1：entrySet()
        Set entrySet = map.entrySet();
        Iterator iterator1 = entrySet.iterator();
        while (iterator1.hasNext()){
            Object obj = iterator1.next();
            // entrySet集合中的元素都是entry
            Map.Entry entry = (Map.Entry)obj;
            System.out.println(entry.getKey() + " --- " + entry.getValue());
        }
        //方式2：
        Set set2 = map.keySet();
        Iterator iterator2 = set2.iterator(); //变成set了，就可以用迭代器了
        while (iterator2.hasNext()){
            Object key = iterator2.next();
            System.out.println(key+ "<--->" + map.get(key));
        }



    }


    @Test
    public void test5(){
        //向TreeMap中添加key-value，要求key必须是由同一个类创建的对象
        // 因为要按照key进行排序:自然排序、定制排序
        // //TreeMap自然排序
        TreeMap map = new TreeMap();
        Person p1 = new Person("TOm", 18);
        Person p2 = new Person("Jerry", 18);
        Person p3 = new Person("Jack", 20);
        Person p4 = new Person("Rose", 21);
        Person p5 = new Person("Lura", 22);
        map.put(p1, 98);
        map.put(p2, 99);
        map.put(p3, 100);
        map.put(p4, 88);
        map.put(p5, 92);
        Set entrySet = map.entrySet();
        Iterator iterator1 = entrySet.iterator();
        while (iterator1.hasNext()){
            Object obj = iterator1.next();
            // entrySet集合中的元素都是entry
            Map.Entry entry = (Map.Entry)obj;
            System.out.println(entry.getKey() + " --- " + entry.getValue());
        }


        // //TreeMap定制排序
        TreeMap map2 = new TreeMap(new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                // 比较年龄
                if (o1 instanceof Person && o2 instanceof Person){
                    Person p1 = (Person)o1;
                    Person p2 = (Person)o2;
                    return Integer.compare(p1.getAge(), p2.getAge());
                }
                throw new RuntimeException("输入的类型不匹配");
            }
        });
        Person p11 = new Person("TOm", 18);
        Person p21 = new Person("Jerry", 18);
        Person p31 = new Person("Jack", 20);
        Person p41 = new Person("Rose", 21);
        Person p51 = new Person("Lura", 22);
        map2.put(p11, 98);
        map2.put(p21, 99);
        map2.put(p31, 100);
        map2.put(p41, 88);
        map2.put(p51, 92);
        Set entrySet2 = map2.entrySet();
        Iterator iterator2 = entrySet2.iterator();
        while (iterator2.hasNext()){
            Object obj = iterator2.next();
            // entrySet集合中的元素都是entry
            Map.Entry entry2 = (Map.Entry)obj;
            System.out.println(entry2.getKey() + " --- " + entry2.getValue());
        }



    }
}

```



**Map实现类之四: Hashtable**

- Hashtable是个古老的 Map 实现类，JDK1.0就提供了。不同于HashMap,Hashtable是线程安全的。
- Hashtable实现原理和lHashMap相同，功能相同。底层都使用哈希表结构，查询速度快，很多情况下可以互用。
- 与HashMap不同，Hashtable不允许使用null作为 key和l value
- 与HashMap一样，Hashtable 也不能保证其中Key-Value 对的顺序
- Hashtable判断两个key相等、两个value相等的标准，与HashMap一致。



**Map实现类之五:Properties**

- Properties类是Hashtable的子类，该对象用于处理属性文件
- 由于属性文件里的key、value 都是字符串类型，所以 Properties里的key和l value都是字符串类型
- 存取数据时，建议使用`setProperty(String key,String value)`方法和`getProperty(String key)`方法
  

```java
package cn.xpshuai.java1;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 8:54
 * @功能：
 *
 *  |---- Properties: 常用来处理配置文件。key和value都是String类型
 */
public class PropertiesTest {
    public static void main(String[] args) throws Exception {
    FileInputStream fis = null;
    try{
        Properties properties = new Properties();
        //默认在当前工程下:xxx.properties
        fis = new FileInputStream("XXX.properties");  //文件流
        properties.load(fis);  //加载流对应的文件
        String name = properties.getProperty("name"); //获取属性key
        String passwd = properties.getProperty("passwd");
        System.out.println(name + passwd);
        fis.close();
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        if(fis != null){
            fis.close();
        }
    }


    }
}

```







### Collections工具类

- Collections是一个操作**Set、list和lMap等集合**的工具类
- Collections中提供了一系列**静态的方法**对集合元素进行排序、查询和修改等操作，还提供了对集合对象设置不可变、对集合对象实现同步控制等方法
- 排序操作:(均为static方法)

```java
reverse(List):反转List中元素的顺序
shuffle(List):对List集合元素进行随机排序
sort(List):根据元素的自然顺序对指定List集合元素按升序排序
sort(List，Comparator):根据指定的Comparator产生的顺序对List集合元素进行排序wap(List，int，int):将指定list集合中的i处元素和j处元素进行交换
```



**Collections常用方法**
查找、替换

```java
Object max(Collection):根据元素的自然顺序，返回给定集合中的最大元素Object max(Collection，Comparator):根据Comparator指定的顺序，返回给定集合中的最大元素
oobject min(Collection)
oObject min(Collection，Comparator)
int frequency(Collection，Object):返回指定集合中指定元素的出现次数
void copy(List dest,List src):将src中的内容复制到dest中
boolean replaceAll(List list，Object oldVal，Object newVal):使用新值替换List对象的所有旧值
```

![Collections常用方法-同步控制](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/Collections%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95-%E5%90%8C%E6%AD%A5%E6%8E%A7%E5%88%B6.png)



```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-28 9:07
 * @功能：Collections:操作Collection、Map的工具类
 *
 *
 * 面试题:Collection 和Collections区别？
 *
 *
 *
 * 常用方法：
 * reverse(List):反转List中元素的顺序
 * shuffle(list):对List集合元素进行随机排序
 * sort(List):根据元素的自然顺序对指定List集合元素按升序排序
 * sort(List，Comparator):根据指定的Comparator产生的顺序对List集合元素进行排序
 * swap(List， int，int):将指定list集合中的i处元素和j处元素进行交换
 * object max(Collection):根据元素的自然顺序，返回给定集合中的最大元素
 * object max(collection，Comparator):根据Comparator指定的顺序，返回给定集合中的最
 * object min(collection)
 * object min(colLection，Comparator)
 * int frequency(collection，object):返回指定集合中指定元素的出现次数
 * void copy(List dest,List src):将src中的内容复制到dest中
 * boolean replaceAlL(List list，object oldval，object newVal):使用新值替换List对
 *
 *
 *
 */
public class CollectionsTest {
    @Test
    public void test(){
        List list = new ArrayList();
        list.add(132);
        list.add(55);
        list.add(666);
        list.add(666);
        list.add(-8);
        list.add(100);
        list.add(99);

        Collections.reverse(list);
        System.out.println(list); // list也改变了顺序哦

        Collections.shuffle(list);
        System.out.println(list);

        Collections.sort(list);
        Collections.swap(list, 1,2); //交换两个索引的位置
        System.out.println(list);

        Collections.frequency(list,666); //出现次数


        // copy(List dst, List src): 将src的内容复制到dst中
        //易错点: dst的长度不能比src小
        //错误写法
//        List dst = new ArrayList();
//        Collections.copy(dst, list);
//        System.out.println(dst);
        //比较标准的写法
        List dst = Arrays.asList(new Object[list.size()]); // n个null
        Collections.copy(dst, list);
        System.out.println(dst);


        //synchronizedXXX()方法
        //Collections类中提供了多个synchronizedXxx()方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题
        ///返回的List1即为线程安全的List
        List list1 = Collections.synchronizedList(list);


    }
}

```





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E9%9B%86%E5%90%88/  

