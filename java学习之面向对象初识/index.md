# Java学习之面向对象初识


### 面向对象概述

**面向对象三大特征：**

- 封装（Encapsulation）
- 继承（Inheritance）
- 多态（Polymorphism）

**类和类成员：**属性、方法、构造器(前三个重要)；代码块、内部类

**其他关键字：**this, super, static, final, abstract, interface, package, import



**属性(成员变量)和局部变量：**

```java
* 同：
    *  定义格式：数据类型 变量名 = 变量值
    *  先声明后使用
    *  都有其作用域
    * 
* 异：
    *  1.声明位置不同
    * 		属性:直接定义在{}内
        *  	局部变量：声明在方法内部、方法形参、代码块内、构造器形参、构造器内部的变量
        *  2.关于权限修饰符的不同
        *  	属性：可以在声明属性时候使用权限修饰符
        *  		如：public  private，  缺省， protected
        *  	局部变量：不能用权限修饰符
        *  3.默认初始化值
        *  	属性：类的属性，根据其类型有初始化默认值
        *  	局部变量：没有默认初始化值（意味着，我们在调用局部变量的时候一定要显式赋值）
        *  			特别的，形参在调用时赋值即可
        *  4.二者在内存中加载的位置
        *  	属性：加载到堆空间(非static)
        *  	局部变量：加载到栈空间
```

**方法：**

```java
 *  定义格式：
 *  权限修饰符  返回值类型 方法名(形参列表){
 *  	方法体
 *  }
 *  注意：static、final、abstract来修饰的方法，后面再讲
 *  
 *  1.四种权限修饰符：public、private、缺省、 protected
 *  2.返回值类型；
 *  	有返回值 vs 无返回值(void,通常不需要return，非要写的话就return;)
 *  	return之后不可以跟语句了
```



**类的基本结构：**

```java
package cn.xpshuai.test;

public class PersonTest {
	public static void main(String[] args) {
		
	}
}



class Person{
	// 【属性】 = 成员变量 = field = 字段、域
	String name;
	int age;
	boolean isMarried;
	// 构造器
	public Person(){}
	public Person(String n, boolean im) {
		name = n; isMarried = im;
	}
	
	
	// 【方法】 = 成员方法 = 函数 = method
	public void walk() {
		System.out.println("人走路...");
	}
	public String display() {
		return  "名字：" + name + "婚否：" + isMarried;
	}
	
	// 代码块(出现的较少)
	{
		name = "韩梅梅";
		age = 17;
		isMarried = true;
	}
	
	// 内部类(出现的较少)
	class pet{
		String name;
		float weight;
	}
	
}

```



小练习：

```java
package cn.xpshuai.test;
/*
 * 对Stedent.java的改进
 * */


public class StudentsTest {
	public static void main(String[] args) {
		// 声明 Students类型的数组
		Students[] stus = new Students[20];
		
		// 给数组元素赋值
		for (int i = 0; i < stus.length; i++) {
			stus[i] = new Students();
			//给对象元素的属性赋值
			stus[i].num = i + 1;
			stus[i].state = (int)(Math.random() * (6-1+1)+1);
			stus[i].score = (int)(Math.random() * (100-0+1));
		}
		
		// 实例化类对象
		StudentsTest testStu = new StudentsTest();
		
		//输出指定年级的学生
		testStu.searchState(stus, 3);
		
		// 冒泡从小到大排序学生成绩
		testStu.BubbleSort(stus);
		
		System.out.println("----------------排序完成---------------");
		// 遍历输出显示学生信息(排序之后的)
		testStu.printStutent(stus);
	}
	
	/** 遍历student数组操作的方法
	 * @param stus
	 */
	public void printStutent(Students[] stus) {
		for (int i = 0; i < stus.length; i++) {
			System.out.println(stus[i].info());
		}
		
	}

	/** 找指定年级的学生的方法
	 * 
	 * @author XPS
	 * @param stus 要查找的数组
	 * @param sta  要找的年级
	 */
	public void searchState(Students[] stus, int sta) {
		for (int i = 0; i < stus.length; i++) {
			if(stus[i].state == sta) {
				System.out.println(sta + "年级学生：" + stus[i].info());
			}
		}
		
	}
	
	
	/**
	 *  给Students数组冒泡排序的方法
	 * @param stu
	 */
	public void BubbleSort(Students[] stu) {
		for (int i = 0; i < stu.length-1; i++) {
			for (int j = 0; j < stu.length-1-i; j++) {
				if(stu[j].score > stu[j+1].score) {
					//交换的是Student对象的顺序
					Students temp = stu[j];
					stu[j] = stu[j+1];
					stu[j+1] = temp;
				}
			}
		}
	}
	
}


class Students{
	int num; // 学号
	int state; // 年级
	int score; //成绩
	
	//显示学生信息的方法
	public String info() {
		return "学号：" + num + "，年级" + state + "，成绩：" + score;
	}
}
```





### **匿名对象**

```java
package cn.xpshuai.testc;

public class InstanceTest {
	public static void main(String[] args) {
		Phone p1 = new Phone();
		System.out.println(p1);
		p1.sendEmail();
		
		// 匿名对象（new的对象没有赋值给变量名） -->只能调用一次
		new Phone().sendEmail(); // 没有吧实例化的对象赋值给变量，可以直接调用方法
		new Phone().price = 1999;
		new Phone().showPrice(); // 新new的对象，是另外一个了，不能用第二次了
	
		PhoneMall mall = new PhoneMall();
		mall.show(new Phone());  // 这种情况下使用匿名对象较多
		
	}
}

class Phone{
	double price;
	
	public void sendEmail() {
		System.out.println("发邮件");
	}
	public void playGame() {
		System.out.println("打游戏");
	}
	public void showPrice() {
		System.out.println("价格为：" + price);
	}
}


class PhoneMall{
	public void show(Phone p) {
		p.sendEmail();
		p.playGame();
	}
}
```



### 方法的重载(overload)

> 区分与后面的重写



**重载：**在同一个类中，允许存在—个以上的<font color=red>同名方法</font>，只要它们的<font color=red>参数个数或参数类型不同</font>即可。

**特点：**与返回值类型无关，<font color=red>只看参数列表</font>，且参数列表必须不同(参数个数或参数类型)。调用时，根据方法参数列表的不同来区别。

如下两个同名方法构成了重载：

```java
public class{
   //如下两个同名方法构成了重载
	//反转数组（形参是整数数组）
	public void reverseArr(int[] arr) {
		for (int i = 0; i < arr.length; i++) {
			int tmp = arr[i];
			arr[i] = arr[arr.length-i-1];
			arr[arr.length-i-1] = tmp;
		}
	}
	//反转数组(形参是字符数组)
	public void reverseArr(String[] arr) {
		for (int i = 0; i < arr.length; i++) {
			String tmp = arr[i];
			arr[i] = arr[arr.length-i-1];
			arr[arr.length-i-1] = tmp;
		}
	}
    
    
}
```



### 可变个数的形参（新特性）

> jdk5.0新增的内容

* 	格式：func(数据类型 ... 参数名)
 * 	当调用可变个数形参方法时，传入参数**个数可为 0到多个**
 * 	可变个数形参的方法与本类中方法名相同，形参不同的方法之间**构成重载**
 * 	可变个数形参在方法的形参中，必须声明在**末尾**（形参列表的最后一个位置）
 *  可变个数形参在方法的形参中,最多**只能声明一个**可变形参。

```java
/*
 * 使用场景：sql查询数据时候，where后面的条件中不知道写几个的时候
 * */
public class ArgumentsTest {
	
	public static void main(String[] args) {
		ArgumentsTest a = new ArgumentsTest();
		a.show(11);
		a.show("hello");
		a.show("hello", "world");
		a.show(new String[] {"AAA", "BBB", "CCC"});
	}
	
	public void show(int i){
		
	}
	
	public void show(String s){
		
	}
	
	// 新特性：可变个数形参(都能识别)
	public void show(String ... strs){
		for (int i = 0; i < strs.length; i++) {
			System.out.println(strs[i]);   //调用方法一样
		}
		
	}
//	// 在jdk5.0之前，都是用数组来传递多个参数，和上面的是不可共存的(不构成重载)
//	public void show(String []... strs){
//		
//	}
}

```



### 方法参数的值传递机制

**关于变量的赋值：**

 * 如果变量是基本数据类型：此时赋值的是变量所保存的数据值
 * 如果变量是引用数据类型：此时赋值的是变量所指向的地址



**方法形参的传递机制：**【值传递机制】

 * 如果参数是基本数据类型，此时实参赋给形参的是实参真实存储的数据值。
 * 如果参数是引用数据类型，此时实参赋给形参的是实参存储数据的地址值。

```java
public class ValueTranslation {
	public static void main(String[] args) {
		System.out.println("*********基本数据类型**********"); 
		int m = 10;
		int n = m;
		
		System.out.println(m + "\t" + n);
		
		n = 20;
		System.out.println(m + "\t" + n);  // n变了之后，m是不变的
		
		
		System.out.println("*********引用数据类型**********"); 
		
		Order o1 = new Order();
		o1.id =1001;
		
		Order o2 = o1;
		
		System.out.println(o1.id + "\t" + o2.id);  // 1002
		o2.id =1002;
		System.out.println(o1.id + "\t" + o2.id); // o1也为1002（两个地址值相同，指向堆空间中同一个区域）
		
		
		System.out.println("*********方法形参的值传递**********");
		// 交换两个变量的值
		int i = 10;
		int j = 20;
		System.out.println(i + "\t" + j);
		// 交换两个变量的值的操作
		ValueTranslation test = new ValueTranslation();
		test.swap1(i, j); //调用方法，没换成（是基本数据类型，传递的参数的值，只是在swap作用域里面发生了交换）
		System.out.println(i + "\t" +  j);
		
		System.out.println("*********方法形参的引用数据类型传递**********");
		Data data = new Data();
		data.x = 10;
		data.y = 20;
		System.out.println(data.x + "\t" +  data.y);
		ValueTranslation test2 = new ValueTranslation();
		test2.swap2(data); //调用方法，换成了（是引用数据类型，交换的是堆空间的地址值）
		
		System.out.println(data.x + "\t" +  data.y);
		
		
	}
	
	
	public void swap1(int i, int j) {
		int tmp = i;
		i = j;
		j = tmp;
	}
	
	// 引用传递类型
	public void swap2(Data data) {
		int tmp = data.x;
		data.x = data.y;
		data.y = tmp;
	}
	
}


class Order{
	int id;
	
}


class Data{
	int x;
	int y;
}

```



![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%8F%82%E6%95%B0%E7%B1%BB%E5%9E%8B%E4%B8%BA%EF%BC%9A%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.jpg)

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%8F%82%E6%95%B0%E7%B1%BB%E5%9E%8B%E4%B8%BA%E5%BC%95%E7%94%A8%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)

但是，String作为引用数据类型，是例外的，是不能交换成功的，因为它存在字符串常量池中，会新造一个地址。所以，记下面这个结论：

 * 如果变量是基本数据类型：此时赋值的是变量所保存的数据值
 * 如果变量是引用数据类型：此时赋值的是变量所指向的地址



**自定义封装Arrayutil类：**

```java
package cn.xpshuai.testc;
/*
 * 自定义封装数组的工具类
 */


public class ArayUtil {
	//求数组最大值
	public int getMax(int[] arr) {
		int max = 0;
		for (int i = 0; i < arr.length; i++) {
			if (arr[i] > max) {
				max = arr[i];
			}
		}
		
		return max;
	}
	
	//求数组最小值
	public int getMin(int[] arr) {
		int min = 0;
		for (int i = 0; i < arr.length; i++) {
			if (arr[i] < min) {
				min = arr[i];
			}
		}
		return min;
	}
	
	//求数组总和
	public int getSum(int[] arr) {
		int sum = 0;
		for (int i = 0; i < arr.length; i++) {
			sum += arr[i];
		}
		return sum;
	}
	
	//求数组平均值
	public int getMiddle(int[] arr) {
		return getSum(arr) / arr.length;  // 方法中调用方法
	}
	
	//如下两个同名方法构成了重载（调用哪个取决你参数的类型）
	//反转数组
	public void reverseArr(int[] arr) {
		for (int i = 0; i < arr.length; i++) {
			int tmp = arr[i];
			arr[i] = arr[arr.length-i-1];
			arr[arr.length-i-1] = tmp;
		}
	}
	//反转数组
	public void reverseArr(String[] arr) {
		for (int i = 0; i < arr.length; i++) {
			String tmp = arr[i];
			arr[i] = arr[arr.length-i-1];
			arr[arr.length-i-1] = tmp;
		}
	}
	
	//复制数组(区别于数组变量的赋值：arr1=arr)
	public int[] copyArr(int[] arr) {
		int[] arr2 = new int[arr.length];
		for (int i = 0; i < arr2.length; i++) {
			arr2[i] = arr[i];
		}
		return arr2;
	}
	
	// 数组遍历
	public void printArr(int[] arr) {
		System.out.print("{");
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + ", ");
		}
		System.out.print("}");
	}
	
	
	// 数组排序(冒泡为例), 不用return
	public void sortArr(int[] arr) {
		for (int i = 0; i < arr.length-1; i++) {
			for (int j = 0; j < arr.length-1-i; j++) {
				if(arr[j] > arr[j+1]) {
					//交换的是Student对象的顺序
//					int temp = arr[j];
//					arr[j] = arr[j+1];
//					arr[j+1] = temp;
					
					// 错误的。不能传递基本数据类型
//					swap(arr[j], arr[j+1]);
					// 正确的
					swap(arr, j, j+1);
				}
			}
		}
	}
	
	public void swap(int[] arr, int i, int j) {
		int tmp = arr[i];
		arr[i] = arr[j];
		arr[j] = tmp;
	}
	
	//查找指定元素(这里 线性查找)
	public int getIndex(int[] arr, int dest){
		for (int i = 0; i < arr.length; i++) {
			if(arr[i] == dest) {
				return 1; // 返回索引值
			}
		}
		return -1;  //返回-1表示没找到
	}
	
	
	/// ...
}

```



**画内存图**

1.内存结构：

​			栈（局部变量）

  		  堆（new出来的结构：对象、数组）

2.变量：成员变量&局部变量（方法内、方法形参、构造器内、构造器形参、代码块内）





### <font color=red>封装性</font>

隐藏对象内部的复杂性，只对外公开简单的接口。便于外界调用，从而提高系统的可扩展性、可维护性。通俗的说，**把该隐藏的隐藏起来，该暴露的暴露出来**。这就是封装性的设计思想。

封装性与隐藏：

```java
 在对对象发属性进行赋值时，赋值操作要受到数据类型和存储范围的制约。除此之外没有其他制约条件。但是，在实际问题中，我们往往需要给属性赋值
 *加入额外的限制条件，这个条件就不能再属性声明时候体现，我们只能通过方法进行限制条件的添加。
 * 同时我们需要避免用户再使用"对象.属性"方式进行赋值，则需要将属性声明为private
 * -->此时针对属性，就体现了封装性
```

封装性体现：
```java
1.我们将类的属性私有化(private),同时，提供公共的(public)方法来获取(getXXX)和设置(setXXX)此属性的值
2.不对外暴露私有方法
3.单例模式
4. ... ...

```

权限修饰符：
```java
 * 封装性的体现需要权限修饰符来配合：
 * 1.java规定的四种权限(从小到大排序)：private、缺省(即default，不用写的时候)、protected、public
 * 详细见笔记
 * 2.四种权限可以用来修饰类及类的内部结构：属性、方法、构造器、内部类
 * 3.具体的，四种权限都可以用来修饰内部结构：属性、方法、构造器、内部类
 * 修饰类的话：只能用public 和 缺省
```

**封装性总结：**

Java提供了4种权限修饰符来修饰类及类的内部结构，体现类及类的内部结构在被调用时的可见性的大小。

```java
public class AnimalTest {
	public static void main(String[] args) {
		Animal a = new Animal();
		//a.name = "大黄";
		//a.age = 2;
		//a.legs = 4;  //禁止用户直接访问属性(权限修饰符 private)，让他访问setLegs()
		//a.legs = -4;  // 负数肯定是不符合实际的，应该加限制只让输入正偶数
		a.setName("大黄");
		a.setAge(2);
		a.setLegs(6);
		a.show();
		
		
		// 测试权限
		OrderTest o1 = new OrderTest();
		o1.b = 111;
		o1.c = 222;
		// o1.a   调不了了(出了类，私有属性和方法就不能被调用)
		o1.test2();
		o1.test3();
		// o1.test1();  调不了
		
				
	}
}


class Animal{
	private String name;
	private int age;
	private int legs; //腿个数, 私有的，禁止暴露在外面
	
	//对属性的设置
	// set, 把处理逻辑卸载里面封装起来
	public void setLegs(int l) {
		if(l >= 0 && l % 2 == 0) {
			legs = l;
		}else {
			legs = 0;
		}
	}
	
	//对属性的获取：get方法, 返回属性值
	public int getLegs() {
		return legs;
	}
	
	public void setAge(int a){
		if(a >= 0) {
			age = a;
		}else {
			age = 0;
		}
	}
	public int getAge() {
		return age;
	}
	
	public void setName(String n){
		name = n;
	}
	public String getName() {
		return name;
	}
	
	
	public void eat() {
		System.out.println("动物进食");
	}
	public void show() {
		System.out.println("动物的名字" + name + "，年龄：" + age + "，腿个数：" + legs);

	}
}

```



权限测试例子：

```java
public class OrderTest2 {  // 不同包下，可以起同名的类
	public static void main(String[] args) {
		// 测试权限
		// 出了Order类所属的包之后，私有、缺省的结构都不可以调用了
		OrderTest o1 = new OrderTest();
		//o1.b = 111;  // 缺省的也调用不了了
		o1.c = 222;
		// o1.a   调不了了(出了类，私有属性和方法就不能被调用)
		//o1.test2();
		o1.test3();
		// o1.test1();  调不了
		
	}
}

```

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/java%E6%9D%83%E9%99%90%E4%BF%AE%E9%A5%B0%E7%AC%A6.png)





### 构造器

**特征：**

- 它具有与类相同的名称
- 它不声明返回值类型(与声明为void不同)
- 不能被static、final、synchronized、abstract、native修饰，不能有return语句返回值



**作用：**创建对象，初始化对象的属性

**注意：**

 * 如果没有显式的定义类构造器的话，则系统**默认**提供一个**空参**的构造器
 * 定义构造器的格式：**权限修饰符 类名(形参列表){}**
 * 构造器 **不是方法**
 * 一个类中定义多个构造器构成重载
 * 一旦显式的定义类的构造器，系统就不再提供默认的空参构造器
 * 一个类中，**至少**会有**一个**构造器

如下：

```java
public class PeopleTest {
	public static void main(String[] args) {
		// 创建类对象
		People p = new People("Tom", 18);  //形参在创建对象时候就赋值了属性
		p.walk();
	}
}


class People{
	String name;
	int age;
	
	// 构造器
	public People(){
		System.out.println("构造器1被调用...");
	}
	// 构造器的重载
	public People(String n){
		System.out.println("构造器1被调用...");
		name = n;    //形参在创建对象时候就赋值了属性
	}
	public People(String n, int a){
		System.out.println("构造器1被调用...");
		name = n;    //形参在创建对象时候就赋值了属性
		age = a;
	}

	public void walk() {
		System.out.println("人走路...");
	}

}
```



**总结-属性赋值过程(按优先顺序)：**

1. 默认初始化
2. 显式初始化
3. 构造器中初始化
4. 通过`对象.属性`或`对象.方法`的方式赋值



### JavaBean

JavaBean：是一种Java语言写成的可重用组件。



**所谓javaBean，是指符合如下标准的Java类:**

- 类是公共(public)的
- 有一个无参的公共的构造器
- 有属性，且有对应的getXXX、setXXX方法

**满足这样三个条件的一个类，就叫JavaBean**



> 也可以通过反射来造对象





### UML类图

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/UML%E7%B1%BB%E5%9B%BE.jpg)





### this关键字

- this表示当前对象，可以调用类的属性、方法和构造器
- 它在方法内部使用，即这个方法所属对象的引用
- 它在构造器内部使用，表示该构造器正在初始化的对象

```java
package cn.xpshuai.www;
/*
 * this关键字
 * 
 * this修饰属性和方法
 * this理解为：当前对象 或 当前正在创建的对象(构造器中)
 * 
 * 【在类的方法中】，我们可以使用"this.属性"或"this.方法"的方式，调用当前对象属性或方法。但是,通常情况下,我们都选择省略"this."。
* 特殊情况下，如果方法的形参和类的属性同名时，我们必须显式使用"this.变量"的方式，来表名次变量是属性而非形参
* 同样，【在构造器中】，我们可以使用"this.属性"或"this.方法"的方式，调用当前正在创建的对象属性或方法。但是,通常情况下,我们都选择省略"this."。I
* 特殊情况下，如果方法的形参和类的属性同名时，我们必须显式使用"this.变量"的方式，来表名次变量是属性而非形参
 *
 *
 *
 * 【this调用构造器】
 * 1.我们在类的构造器中，可以使用显式的使用"this(形参列表);"方式，调用本类中其他的构造器
 * 2.构造器中不能使用"this(形参列表);"方式调用自己
 * 3.如果一个类中有n和构造器，则最多有n-1个构造器中使用了"this(形参列表);"
 * 4."this(形参列表);"必须声明在当前构造器的首行
 * 5.构造器内部不能同时调用两个"this(形参列表);"
 * 
 */
public class ThisTEst {
	public static void main(String[] args) {
		People2 p1 = new People2("Tom", 12);
		System.out.println(p1.getAge());
		
	}
}

class People2{
	String name;
	int age;
	
	// 构造器（如果写下面的带参数的构造器，必须先写上空的构造器）
	public People2(){
		System.out.println("构造器1被调用...");
		this.eat(); // 使用this调用方法
	}
	
	public People2(String name){
		this();  //调用上面的构造器1
		System.out.println("构造器2被调用...");
		this.name = name;    
	}
	
	public People2(String name, int age){
		this(name);  //调用上面的构造器2，方式冗余
//		this.name = name;   // 那么下面就不需要这行的name了 
		this.age = age;
		System.out.println("构造器3被调用...");
	}

	
	//set
	public void setName(String name) {
		this.name = name;   //使用this(当前对象)区分：当前对象的name=name
	}
	//get
	public String getName() {
		return this.name;
	}
	
	public void setAge(int age) {
		this.age = age;
	}
	public int getAge() {
		return this.age;
	}


	public void walk() {
		System.out.println("人走路...");
	}

}

```



Eclipse快速生成get和set等方法：`alt + seift +s`快捷键就可以看到啦

Eclipse快速生成get和set等方法：`alt +insert `快捷键就可以看到啦



### package

```java
 * package关键字的使用：
 * 1.为了实现更好项目中类的管理，提出了“包”的概念
 * 2.使用package声明类或者接口所属的包，声明在源文件的首行
 * 3.包，属于标识符，遵循标识符命名规范(小写)、见名知意
 * 4.每"."一次，就代表一层文件目录
 * 
 * 补充：同一个包下，不能命名同名的接口、类, 不同包下可以命名同名的接口、类
```







### <font color=red>继承性</font>

- Java**只支持单继承和多层继承**，不允许多重继承
- Object类是所有类的根父类



**简单例子如下：**

测试类：

```java
package cn.xpshuai.java;
/* 
 * 继承好处：
 * 1.减少代码冗余，提高代码复用
 * 2.便于功能拓展
 * 3.为了以后多态使用，提供了前提
 * 
 * 格式： class A extends B{}
 * A:子类、派生类、subclass
 * B：父类、超类、基类、superclass
 * 
 * 体现：一旦子类A继承父类B以后，子类A中就获取了父类B中声明的所有的 属性、方法
 * 特别的，父类中声明为私有的属性和方法，子类仍获取到了，只是由于封装性影响-->使得子类不能直接调用而已
 * 
 * 子类继承父类以后，还可以声明自己特有的属性或方法:实现功能的拓展。
 * 
 * 
 * 
 * 规定：
 * 1.一个类只能有一个父类
 * 2.Java只支持单继承和多层继承
 * 3.直接父类、间接父类。子类继承父类以后，就获取了直接父类以及所有间接父类中声明的属性和方法
 * 
 * 
 * 如果我们没有显式的声明一个类的父类的话，则此类继承于java.lang.Object类
 * 所有的Java类，都直接或间接的继承Object类
 */

public class ExtendsTest {
	public static void main(String[] args) {
		Person p1 = new Person("张三", 18);
		p1.eat();
		
		Student s1 = new Student("GIS");
		s1.setName("张三三");
		s1.setAge(22);
		// 父类中的方法，子类中也可以调用 来实现复用
		s1.show();
//		System.out.println(s1.getMajor());
	}
}

```

父类：

```java
package cn.xpshuai.java;

public class Person {
	private String name;  //私有的也继承到了，通过getXXX方法来获取
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
	
	public void eat() {
		System.out.println("吃");
	}
	
	public void sleep() {
		System.out.println("睡觉");
	}
}

```

子类：

```java
public class Student extends Person{    // extends Person ：表示继承Person类
	private String major;
	
	//构造方法
	public Student() {
		super();
	}
	
	public Student(String major) { 
		super();
		this.major = major;
	}
	// get
	public String getMajor() {
		return major;
	}
	// set
	public void setMajor(String major) {
		this.major = major;
	}

	public void study() {
		System.out.println("学习");
	}
	
	public void show() {
		System.out.println("姓名：" + getName() + ", 年龄：" + getAge() + "，专业：" + getMajor());
	}
    	//重写父类方法
    @Override
	public void eat() {
		System.out.println("学生应该多吃");
	}
}

```



#### 方法的重写(override 、 overwrite)

> 与前面的"重载"区分

**定义：**在子类中可以根据需要对从父类中继承来的方法进行改造，也称为方法的<font color=red>重置、覆盖</font>。在程序执行时，子类的方法将覆盖父类的方法（子类根据需要对从父类继承来的方法进行改造）。

**要求：**

1. 子类重写的方法必须和父类被重写的方法具有<font color=red>相同的方法名称、参数列表</font>
2. 子类重写的方法的返回值类型<font color=red>不能大于</font>父类被重写的方法的返回值类型
3. 子类重写的方法使用的访问权限<font color=red>不能小于</font>父类被重写的方法的访问权限
   - 子类不能重写父类中声明为private权限的方法
4. 子类方法抛出的异常不能大于父类被重写方法的异常



**注意：**

子类与父类中同名同参数的方法必须同时声明为<font color=red>非static的</font>(即为重写)

若同时声明为static的〈不叫重写），因为static方法是属于类的，子类无法覆盖父类的方法。

```java
 * 【重写】override 、 overwrite
 * 子类根据需要对从父类继承来的方法进行改造
 * 
 * 形式：权限修饰符   (可选：static/final) 返回值类型   方法名(形参列表)  throws 异常的类型{
 * 	//方法体
 * }
 * 
 * 1.必须与父类被重写的方法具有“相同的方法名称、参数列表”
 * 2.子类重写的方法的返回值类型“不能大于”父类被重写的方法的返回值
 * 			>父类被重写的方法的返回值类型是void，则子类重写的方法的返回值类型只能是void
 *			>父类被重写的方法的返回值类型是A类型，则子类重写的方法的返回值类型可以是A类或A类的子类
 *			>父类被重写的方法的返回值类型是基:本据类型，则子类重写的方法的返回值类型必须是相同的基本数据类型
 * 3.子类重写的方法使用的当问权限“不能小于”父类被重写的方法的访问权限
 * 			>子类不能重写父类中private的方法
 * 4.子类方法抛出的异常不能大于父类被重写方法的异常
 * 			>
 * 
 * 子类与父类中同名同参数的方法必须同时声明为非static的(即为重写)，或者同时声明为static的（不是重写）。因为static方法是属于类的，子类无法覆盖父类的方法。
 * 
```



#### super关键字

- 子类重写了父类的方法之后，可以用super调用父类中的方法
- super可以调用：属性、方法、构造器



**super调用属性和方法：**

* 我们可以在子类的方法或构造器中。通过使用`super.属性`或`super.方法`的方式，显式的调用父类中声明的属性或方法。但是，通常情况下，我们习惯省略`super. `
 * 特殊情况: 当子类和父类中定义了同名的属性时，我们要想在子类中调用父类中声明的属性，则必须显式的使用`super.属性`的方式，表明调用的是父类中声明的属性。
 * 特殊情况: 当子类重写了父类中的方法以后，我们想在子类的方法中调用父类中被重写的方法时，则必须显式的使用`super.方法`的方式，表明调用的是父类中被重写的方法。

 **super调用父类构造器：**

 * 我们可以在子类的构造器中显式的使用`super(形参列表)`的方式，调用父类中声明的指定的构造器
 * `super(形参列表)`的使用，必须声明在<font color=red>子类构造器的首行!</font>
 * 我们在类的构造器中，针对于`this(形参列表)`或`super(形参列表)`<font color=red>只能二选一</font>，不能同时出现
 * 在构造器的首行，没有显式的声明`this(形参列表)`或`super(形参列表)`，则<font color=red>默认调用的是父类中空参的构造方法</font>
 * 在类的多个构造器中，至少有一个类的构造器中使用了`super(形参列表)`，调用父类中的构造器

**子类对象实例化的全过程：**

```java
1．从结果上来看:(继承性)
     * 子类继承父类以后，就获取了父类中声明的属性或方法。
     * 创建子类的对象，在堆空间中，就会加载所有父类中声明的属性。
         
2．从过程上来看;
     * 当我们通过子类的构造器创建子类对象时，我们一定会直接或间接的调用其父类的构造器，进而调用父类的父类的构造器,直到调用了java.lang.0bject类中空参的构造器为止。正因为加载过所有的父类的结构，所以才可以看到内存中有父类中的结构，子类对象才可以考虑进行调用。
     
* 明确:虽然创建子类对象时，调用了父类的构造器，但是自始至终就创建过一个对象，即为new的子类对象。
```

如子类：

```java
public class Student extends Person{    // extends Person 继承Person类
	private String major;
	private int id = 1002;  //学号
	
	//构造方法
	public Student() {
		super();
	}
	
	public Student(String name, int age, String major) {
		super(name, age);  // super调用父类构造器
		this.major = major;
	}


	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public Student(String major) { 
		super();
		this.major = major;
	}
	// get
	public String getMajor() {
		return major;
	}
	// set
	public void setMajor(String major) {
		this.major = major;
	}

	public void study() {
		System.out.println("学习");
	}
	
	//重写父类方法
	@Override
	public void eat() {
		System.out.println("学生应该多吃有营养的食物...");
		super.sleep();
	}
	
	public void show() {
		System.out.println("姓名：" + getName() + ", 年龄：" + getAge() + "，专业：" + getMajor());
		System.out.println(this.getId());  // 子类id
		System.out.println(super.getId());		// 父类id
	}
	
	
}

```

测试类：

```java
public class ExtendsTest {
	public static void main(String[] args) {
		Person p1 = new Person("张三", 18);
		p1.eat();
		
		Student s1 = new Student("GIS");
		s1.setName("张三三");
		s1.setAge(22);
		// 父类中的方法，子类中也可以调用 来实现复用
		s1.show();
		s1.eat();
		
		Student s2 = new Student("小李", 18, "CS");  //super调用父类构造器, 可以直接在这初始化
		s2.show();
		
	}
}
```







### <font color=red>多态性(Polymorphism)</font>

**对象的多态性：**父类的引用指向子类的对象（可以直接应用在抽象类和接口上）



Java引用变量有两个类型：

- 编译时类型：由声明该变量时候使用的类型决定
- 运行时类型：由实际赋给该变量的对象决定

**简称：编译看左，运行看右**

#### 虚拟方法

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/java%E8%99%9A%E6%8B%9F%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8.png)



**问：多态是编译时行为还是运行时行为？**

**答：运行时行为**





#### instanceof操作符

`x instanceof A`：检验x是否为类A的对象，返回值为boolean类型

- 要求x所属的类与类A必须是子类和父类的关系，否则编译错误
- 如果x属于类A的子类B(间接父类子类关系)，也可以使用



instanceof情景：为了避免在向下转型时出现ClassCastException的异常，我们在向下转型之前，先进行instanceof判断



#### toString()

输出样式，可以手动重写子类中的该方法



### finalize()

对象回收之前会调用这个方法：finalize()， 子类可重写

```java
//子类中
class B{
    @Override
    protecred void finalize() throws Throwable{
        System.out.println("对象被释放--> "  = this);
    }

	...
}

//强制性释放空间
System.gc();
    
    
```





### == 与 equals()

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/qeuals%E5%92%8C%3D%3D%E5%8C%BA%E5%88%AB.png)



```java

import java.sql.Date;

/*
 * 
 * == 与equals()
 * 
 * 
 * 面试题： == 与equals() 区别？
 * 【== 运算符】
 1.可以使用在基本数据类型变量和引用数据类型变量中
 2.如果比较的是基本数据类型变量:比较两个变量保存的数据是否相等(不一定类型要相同)
        如果比较的是引用数据类型变量，比较两个对象的地址值是否相同.即两个引用是否指向同一个对象实体

【equals()方法】
是一个方法，而非运算符
只能用于引用数据类型
像String、 Date、File、包装类等都重写了0bject类中的equals()方法。重写以后，比较的不是两个引用的地址是否相同，而是比较两个对象的T体内容是否相同。



【重写Object类中的equals()】
通常情况下，我们自定义的类如果使用equals()的话，也通常是比较两个对象的"实体内容"是否相同。那么，我们就需要对Object类中的equals()进行重写
重写的原则：比较两个对象的实体内容是否相同.

 * 
 */
public class EqualsTest {
	public static void main(String[] args) {
		int i = 10;
		char c = 10;
		System.out.println( c == i);  // true
		
		char c1 = 'A'; // ascii码也为65
		char c2 = 65;   
		System.out.println(c1 == c2); //true
		
		
		Customer customer1 = new Customer("xxx");
		Customer customer2 = new Customer("xxx");
		
		System.out.println(customer1 == customer2);   // false
		
		
		// String也是引用型数据
		String s1 = new String("asd");
		String s2 = new String("asd");
		System.out.println(s1 == s2);   // false
		System.out.println(s1.equals(s2));   // true   调用的是String中的equals()，和==作用是相同的
		System.out.println(customer1.equals(customer2));   // false  调用的是Object中的equals, 自定义之后变为true了
		
		
		Date date1 = new Date(2354534654L);
		Date date2 = new Date(2354534654L);
		System.out.println(date1.equals(date2));  //true

	}
}

class Customer{
	String name;
	int age;

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

	public Customer() {
	}
	
	public Customer(String name) {
		this.name = name;
	}
	
	public Customer(String name, int age) {
		this.name = name;
		this.age = age;
	}

	//自己手动重写Object类中的equals()
	// 重写原则：
//	@Override
//    public boolean equals(Object obj) {
//		System.out.println("Customer的equals()...");
//		if(this == obj) {
//			return true;
//		}
//		if(obj instanceof Customer) {
//			Customer cust = (Customer)obj; //强转
//			//比较两个对象的每个属性是否都相同
//			return this.age == cust.age && this.name.equals(cust.name);
//		}else {
//			return false;
//		}
//    }
	
	//自动重写的
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Customer other = (Customer) obj;
		if (age != other.age)
			return false;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		return true;
	}

}

```



重写equals()方法的原则：

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E9%87%8D%E5%86%99qauals%28%29.png)







### 包装类

#### 单元测试

步骤如下：

```java
package cn.xpshuai.java3;

import org.junit.Test;

/*
 * Java中JUnit单元测试
 * 
 * 步骤：
 * 1.选中当前工程–右键选择: build path - add libraries - JUnit 4 -下一步
 * 2.创建Java类，进行单元测试
 * 		测试的Java类要求：
 * 				1.此类是public的
 * 				2.此类提供公共是无参数的构造器
 * 3.此类中声明单元测试方法：
 * 		此时的单元测试方法：方法权限为public，没有返回值类型。没有形参
 * 4.此单元测试方法需要声明注解：@Test, 并import org.junit.Test;
 *
 * 5.声明好单元测试方法后，就可以在方法中写相关测试代码
 * 6.写完代码后，双击方法名：右键-->Run As --> JUnit Test
 *
 * 说明：
 * 1.如果执行结果没有任何异常:绿条
 * 2.... 异常：红条
 *
 *
 */
public class JUnitTest {
	int num = 10;
	
	@Test
	public void testEquals() {
		String s1 = new String("y");
		String s2 = new String("yyy");
		System.out.println(s1 == s2);
		System.out.println(s1.equals(s2));
		
		System.out.println(num); // 可以直接调用属性和方法
	}
	
	@Test
	public void testToString() {
		System.out.println(num); 
	}
}

```





#### 包装类(Wrapper)的使用

> 让基本数据类型也具有类的特征，封装起来

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/java%E5%8C%85%E8%A3%85%E7%B1%BB.png)



**基本类型、包装类、String三种类型相互转换：**

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E4%B8%8E%E5%8C%85%E8%A3%85%E7%B1%BB%E4%B8%8EString%E8%BD%AC%E6%8D%A2.png)

```java
package cn.xpshuai.java3;
/*
 * 
 * 
 * 
 */
import org.junit.Test;

public class WrapperTest {
	public static void main(String[] args) {
		
		
	}
	
	
	// 1.基本数据类型转换为包装类：调用包装类的构造器
	@Test
	public void test1() {
		int num1 = 10;
		
		Integer in1 = new Integer(num1);
		System.out.println(in1.toString());
		
		Integer in2 = new Integer("132");
		System.out.println(in2.toString());
		
		
		Float float1 = new Float("11.1");
		System.out.println(float1);
		
		
		Boolean boolean1 = new Boolean("tRuE");
		Boolean boolean2 = new Boolean("true123");
		System.out.println(boolean1);  // true
		System.out.println(boolean2);  // false  比较特别
		System.out.println(boolean2);  // false  比较特别
		
		Order order = new Order();
		System.out.println(order.isMan);  // null  
		System.out.println(order.isFemale);  // false  
	}
	
	
	
	// 2.包装类转换为基本数据类型：调用包装类的xxxValue()
	@Test
	public void test2() {
		Integer in1 = new Integer(12); // 对象
		int i1 = in1.intValue();  // 变成基本变量了
		System.out.println(i1 + 11);
	}
	
	
	// JDK5.0新特性：自动装箱与拆箱，用不着（.基本数据类型转换为包装类：调用包装类的构造器）
	@Test
	public void test3() {
		int num1 = 10;
		// 基本数据类型-->包装类的对象
//		method1(num1);
		
		// 新特性：自动装箱
		int num2 = 11; //自动装箱
		Integer in2 = num2;
		
		boolean isFlag = true;  //自动装箱
		Boolean isFlag2 = isFlag;
		
		
		// 新特性：自动拆箱（包装类转换为基本数据类型）
		int num3 = in2; // 自动拆箱
		
	}
	public void method1(Object obj) {
		System.out.println(obj);
	}
	
	
	// 3.基本数据类型、包装类 -->转换为String类型
	@Test
	public void test4() {
		// 方式1：连接运算
		int num1 = 10;
		String str1= num1 + "";
		//方式2：调用String重载的valueof(xxx xxx)
		float f2 = 12.3f;
		String string = String.valueOf(f2);
		System.out.println(string);  // "12.3"
		
		Double d1 = new Double(12.5);
		String string2 = String.valueOf(d1);
		System.out.println(string2);
	}
	
	
	// 3.String类型 -->转换为基本数据类型、包装类：调用包装类的parseXXX方法
		@Test
		public void test5() {
			String str1 = "123";
			//记住
			int num2 = Integer.parseInt(str1);
			System.out.println(num2); //123
			
		}
	
	
}

class Order{
	Boolean isMan;
	boolean isFemale;
}


```





Vector()可以代替数组，后续再说...





### 关键字：static

无论产生了多少个对象，只希望某些特定的数据在内存空间中只有一份

static可以用来修饰：属性、方法、代码块、内部类

**static修饰属性：**静态变量

```java
属性按照是够使用static修饰分为：静态属性 vs 非静态属性(实例变量)
    
实例变量：创建的多个对象，每个对象都独立拥有一套类中的非静态属性。当修改其中一个非静态属性时不会影响另一个对象中相同属性值的修改
    
静态属性：创建了类的多个对象，公用同一个静态变量，当通过某一个对象修改该静态变量时，会导致其他对象调用该静态变量时是修改过的
 * 
 * 		(1)静态变量随着类的加载而加载: "类.静态变量"方式进行调用
 * 		(2)静态变量的加载要早于对象的创建
 * 		(3)由于类只会记载一次，则静态变量在内存中也会只存在一份：存在方法区的静态域
 * 		
举例：System.out     Math.PI  
```

**static修饰方法：**

```java
静态方法随着类的加载而加载: "类.静态方法"方式进行调用
	静态方法中，只能调用静态的方法或属性
    非静态方法中，既可以调用非静态的方法或属性，也可以调用静态的方法或属性
		
在静态的方法内，不能使用this关键字、super关键字
关于静态属性和静态方法的使用，大家都从生命周期的角度去理解。
```





```java
/*
 * 在开发中，如何确定一个属性是否需要声明为static：
 * 	 >属性是可以被多个对象所共享的，不会随着对象的不同而本同的。
 
 *  在开发中，如何确定一个方法是否需要声明为static：
 * 	>操作静态属性的方法，通常设置为static的
 * 	>工具类中的方法，习惯上声明为static的,比如 Math, Collection, Arrats
 */
public class StaticClass {
	public static void main(String[] args) {
		Chinese.nation = "CHN";
		
		Chinese c1 = new Chinese();
		c1.name = "姚明";
		c1.age = 40;
//		c1.nation = "CHN";
		
		Chinese c2 = new Chinese();
		c2.name = "马龙";
		c2.age = 30;
		
		System.out.println(c2.nation);
		// 直接 "类.方法"  调用
		Chinese.show();
	}
}



class Chinese{
	String name;
	int age;
	static String nation;
	
	public static void show() {
		System.out.println("我是一个中国人！！！");
		Chinese.nation ="CHN";
//		Name = "YYY";
		
	}
	public void show2() {
		System.out.println("我是一个中国人2！！！");
		Chinese.nation ="CHN";
		System.out.println(this.name);
		
	}
}

```



### 类变量和实例变量内存解析

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/%E7%B1%BB%E5%8F%98%E9%87%8F%E5%92%8C%E5%AE%9E%E4%BE%8B%E5%8F%98%E9%87%8F%E5%86%85%E5%AD%98%E8%A7%A3%E6%9E%901.png)



### 单例模式

```java
package cn.xpshuai.java4;
/*
 * 单例(SingleTon)设计模式：
 * 1.定义：某个类只能存在一个实例
 * 2.如何实现
 * 		饿汉式单例模式  vs 懒汉式单例模式
 * 
 * 3.区分饿汉式、懒汉式
 * 饿汉式： 坏处：对象加载时间过长。
 * 		好处：是线程安全的
 *  懒汉式：好处：延迟对象创建
 *  	目前下面的写法坏处：线程不安全 -->多线程章节再修改
 * 
 * 4.单例模式优点：减少了系统的开销
 * 
 * 5.单例模式使用场景：
 * 
 */
public class SingleTonTest {
	public static void main(String[] args) {
		// 饿汉式测试
		// bank1和bank2是一个对象
		Bank bank1 = Bank.getIntance();
		Bank bank2 = Bank.getIntance();
		System.out.println(bank1 == bank2);
		
		
		// 懒汉式测试
		Order order1 = Order.getInstance();
		Order order2 = Order.getInstance();
		System.out.println(order1 == order2);
	}
}

// 【饿汉式单例模式】
class Bank{
	// 1.私有化类的构造器：为了避免在外面类调用该构造器
	private Bank() {

	}
	
	// 2.内部创建类的对象
	// 4.要求此对象也必须是static的
	private static Bank instance1 = new Bank();  //先创建
	
	//3.提供公共静态方法，返回类的对象(类似getXXX方法)
	public static Bank getIntance() {
		return instance1;
	}
}


//【懒汉式单例模式】
class Order{
//	1.私有化类的构造器
	private Order(){
	}
	
//	2.声明当前类对象（没有初始化）
//	4.此对象也必须声明为static
	private static Order instance2 = null;
	
//	3.声明public、static的返回当前类对象的方法
	public static Order getInstance() {
		if(instance2 == null) {
			instance2 = new Order();
		}
		return instance2;
	}
}

```



> 一个.java源文件只能有一个public的类





### 代码块

```java
package cn.xpshuai.java4;
/*
 * 类成员4：代码块（或初始化块）
 * 
 * 1.代码块作用：用来初始化类、对象
 * 2.如果有修饰的话，只能用static来修饰
 * 3.分类：静态代码块 vs 非静态代码块
 
 * 静态代码块：
 * 	>内部可以有输出语句
 *  >随着类的加载而执行，且只执行一次
 *  >作用：初始化类的信息
 *  >如果一个类中定义多个静态代码块，按照声明优先顺序执行
 *  >静态代码块要优先于非静态代码块的执行
 * 	>只能调用静态结构，不能调用非静态属性和方法
 * 
 
 * 非静态代码块：
 * >内部可以有输出语句
 * >随着对象的创建而执行，每创建一个对象就执行一次非静态代码块
 * >作用：可以在创建对象时对对象的属性进行初始化
 * >如果一个类中定义多个非静态代码块，按照声明优先顺序执行
 * >可以调用非静态属性和方法以及静态属性和方法
  *       非静态代码块执行顺序先于构造器
 * 
 * 
 * 对属性可以赋值的位置：按顺序排
 * 1.默认初始化
 * 2.显式初始化 / 3.在代码块中赋值
 * 4.构造器初始化
 * 5.有对象之后，通过"对象.属性" 或 "对象.方法"方式进行赋值
 * 
 * 
 * 
 * 开发中使用代码块(使用频率不高)：
 * 
 * 
 * 由父及子，先静后构
 * 
 * 
 */
public class BlockTest {
	public static void main(String[] args) {
		String desc = Person.desc;
		Person p1 = new Person();
		System.out.println(desc);
	}
}


class Person{
	//属性
	String name;
	int age;
	static String desc = "我是一个人";
	
	//构造器
	public Person() {
	}
	public Person(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
	//非静态代码块代码块
	{
		System.out.println("hello非静态代码块1");
		// 调用非静态结构
		age = 1; //
		eat();
		// 调用静态结构
		info();
		desc = "hhhhhh";
		
		
	}
	{
		System.out.println("hello非静态代码块2");
		age = 1; //
		
	}
	//静态代码块
	static{
		System.out.println("hello静态代码块1");
		// 只能调用静态结构
		info();
		desc = "重新赋值，我是一个爱学习的人1";
	}
	static{
		System.out.println("hello静态代码块1");
		desc = "重新赋值，我是一个爱学习的人2";
	}
	
		
	//方法
	public void eat() {
		System.out.println("人吃饭");
	}
	
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + "]";
	}
	
	public static void info() {
		System.out.println("我是一个快乐的人");
	}
}
```





### final 关键字

final可以修饰的结构：类、方法、变量

```java
package cn.xpshuai.java4;
/*
 *关键字：final
 *
 * final可以修饰的结构：类、方法、变量
 * 
 * 【final修饰类：】
 * 	该类加了final后，不能被继承了
 * 		比如：String,  System, StringBuffer
 * 
 * 【 final修饰方法：】
 * 	该方法不能被重写了
 * 		比如：Object中的getClass()
 * 
 * 【 final修饰变量：】
 * 此时的"变量"就称为是一个常量，不能再修改了
 * 		final修饰属性：可以考虑赋值的位置有：显式初始化、代码块中初始化、构造器中初始化
 * 
 * 	 【 final修饰局部变量：】
 * 		方法内声明的局部变量、形参
 * 		尤其是final修饰形参时候，表示该形参为一个常量，当我们调用时候为这个形参赋值为一个实参，以后，方法内只能进行调用不能修改
 * 
 * 
 * static final: 用来修饰 属性 -->全局常量(接口中常用)、方法 -->
 * 
 * 
*/
public class FinalTest {
	public static void main(String[] args) {
		AA aa = new AA();
		aa.setDown(10);
	}
}


final class FinalA{
	// 该类不能再被继承了
}

class AA{
	// 该属性不能再被修改了
	final String NAME = "aaa";
	final int WIDTH;
	final int HEIGHT;
//	final int DOWN;
	{
		WIDTH =10; 
	}
	public AA(){
		HEIGHT = 10; 
	}
	public AA(int n){
		HEIGHT = n; 
	}
	
	public final void show() {
		//此方法不能被重写了
	}
	
	public void setDown() {
		final int NUM = 11; // final之后就变成常量了
	}
	public void setDown(final int num) {
		System.out.println(num); //修饰形参时候，方法内只能进行调用不能修改
	}
}

```



### 抽象类与抽象方法（重要）

> abstract

有时将一个父类设计得非常抽象，以至于它没有具体的实例，这样的类叫做抽象类。

```java
package cn.xpshuai.java5;
/*
 * abstract 
 * 可以修饰的结构：类、方法
 * abstract修饰，表示不能造对象了
 * 
 * abstract修饰类:抽象类
 * 		>此类不能实例化
 * 		>抽象类中一定有构造器，便于子类实例化时候调用
 * 		>开发中，都会提供抽象类的子类，让子类对象实例化并完成操作
 * 
 * 
 * 
 * abstract修饰方法:抽象方法
 * 		>只有方法的声明，没有方法体
 * 		>如果类中包含有抽象方法，那么该类一定是抽象类。反之抽象类中不一定要有抽象方法
 * 		>若子类重写了父类中所有抽象方法中，才能实例化对象
 * 		  若子类没有重写父类中的所有的抽象方法，则此子类也是一个抽象类，需要使用abstract修饰
 * 
 * 
 * 抽样的应用场景举例：
 * 
 * 
 * 注意点：
 * 1.abstract不能用来修饰：属性、构造器等
 * 2.abstract不能用来修饰私有方法、静态方法、final的方法、final的类
 * 
 * 
 */
public class AbstractTest {
	public static void main(String[] args) {
//		Person p1 = new Person();   // abstract之后就不能实例化对象了
//		p1.eat();
	}
}


abstract class Person { // abstract之后就不能实例化了
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
	
	
	public void eat() {
		System.out.println("人：吃饭");
	}
	
	public void walk() {
		System.out.println("人：走路");
	}
	
	abstract public void play(); //抽象方法（只有方法的声明，没有方法体）子类中必须得实现这个方法
	
}

class Student extends Person{
	public Student(String name, int age) {
		super(name, age);  // 
	}
	
	//F1.重写该方法
	@Override
	 public void play() {
		 System.out.println("学生玩");
	 }
	 //F2:把子类变成抽象类
}




```



**创建抽象类的匿名子类对象**

```java
package cn.xpshuai.java5;
/*
 *  抽象类的匿名子类
 */
public class PersonTest {
	public static void main(String[] args) {
		method(new Student("tom", 19));  //匿名对象
		
		Worker worker = new Worker();
		method1(worker);  //非匿名的类和对象
		
		method1(new Worker());  //非匿名的类, 匿名的对象
		
		//创建了一个匿名子类的对象：p
		Person p = new Person() {  
			@Override    //重写匿名方法即可
			public void play() {
				System.out.println("匿名重写");
			}
			
		};
		method1(p); 
		
		
		//创建匿名子类的匿名对象（适合用一次的情况）
		method1(new Person() {
			@Override
			public void play() {
				System.out.println("玩好玩的东西");
			}
		});
	
	}
	
	public static void method(Student s) {

	}
	public static void method1(Person p) {

	}
}

class Worker extends Person{
	
	@Override
	public void play() {
		System.out.println("工人娱乐");
	}
}

```



**多态的应用：**模板方法设计模式



### 接口(Interface)（重要）



```java
package cn.xpshuai.java5;
/*
 * 接口关键字的使用：
 * 1.使用Interface 关键字定义
 * 类可以实现多个接口
 * 一定程度解决了类的单继承的局限性
 * 2.Java中，接口和类是并列的两个结构
 * 
 * 3.定义接口：定义接口中的成员
 * 	3.1 JDK7及以前：只能够定义全局常量和抽象方法
 * 		>全局常量：public static final的, 但是书写时可以省略不写
 * 		>抽象方法：public abstract的
 * 
 * 
 * 		
 * 
 * 	3.2 JDK8: 除了定义全局常量和抽象方法之外，还可以定义：静态方法、默认方法
 * 					
 * 			
 * 
 * 
 * 
 * 4.接口中不能定义构造器！！！意味着接口不可以实例化
 * 
 * 5.开发中，接口通过类去"实现"(implements)的方式来使用
 * 			如果实现类覆盖了接口中所有抽象方法，则该实现类有实例化
 * 			如果实现类没有覆盖接口中所有的抽象方法，则此实现类仍为一个抽象类
 * 			抽象类和接口中，叫实现，不叫重写
 * 
 * 
 * 6.Java类可以实现多个接口--->弥补了Java单继承性的局限性问题
 * 	格式：class AA extends BB implements CC, DD{  }
 * 
 * 
 * 7.接口和接口之间：也叫继承，可以多继承
 * 
 * 
 * 8.接口是具体使用，体现了多态性
 * 9.接口，实际上可以看做是一种规范
 * 
 * 面试题：抽象类与接口有哪些异同？
 * 
 * 
 * 接口应用：代理模式、工厂模式(一种设计模式)
 * 
 */
public class InterfaceTest {
	public static void main(String[] args) {
		Plane plane = new Plane();
		plane.fly();
	}
}


interface AttackAble{
	void attack();
}

interface Flyable{
	//全局常量
	public static final int MAX_SPEED = 7900;
	int MIN_SPEED = 1; //省略了public static final 
	
	// 抽象方法
	public abstract void fly();
	void stop();  // 省略了public abstract
		
}


class Plane implements Flyable{
	@Override  // 这叫：实现，不叫重写
	public void fly() {
		System.out.println("飞机通过引擎起飞");
	};
	@Override
	public void stop() {
		System.out.println("驾驶员减速停止");
	};
	
}

//只能还叫抽象类，要么就把抽象方法都实现了
abstract class Kite implements Flyable{
	@Override  // 这叫：实现，不叫重写
	public void fly() {
		System.out.println("风筝通过风起飞");
	};
	
}

//子弹类（	实现多个接口）
class Bullet extends Object implements Flyable, AttackAble{   //先写继承父类，后写实现具体的接口

	@Override
	public void attack() {
		
	}

	@Override
	public void fly() {
		
	}

	@Override
	public void stop() {
		
	}
	
}

interface AA{
	void method1();
}

interface BB{
	void method2();
}

interface CC extends BB,AA{
	
}

```

```java
package cn.xpshuai.java5;
/*JDK8: 除了定义全局常量和抽象方法之外，还可以定义：静态方法、默认方法
 * 
 * 
 * 
 */
public class Java8Test {
//	public static void method1() {
//		System.out.println("北京");
//	}
	public static void main(String[] args) {
		SubClass sub = new SubClass();
		//知识点1：接口中定义的静态方法，只能通过接口来调用
		CompareA.method1();
		//知识点2：通过实现类的对象，可以调用接口中默认方法，可重写
		sub.method2(); //没有method1()
		//知识点3：如果子类(或实现类)继承的父类和实现接口中有声明了同名同参数的方法
		// 子类在没有重写此方法的情况下，则调用的是父类中此方法 --> 类优先原则
		sub.method3();
		//知识点4：如果实现类实现了多个接口，而这个接口中定义了同名参数的默认方法
		//那么在实现类没有父类，且没有重写此方法的情况下，报错-->接口冲突
		// 这就需要我们必须实现此类中重写此方法
	}
}


class SubClass extends SuperClas implements CompareA,CompareB{
	@Override
	public void method2() {
		System.out.println("compareA:上海222222");
	}
	
	//知识点5：如何在子类(实现类)的方法中调用父类、接口中被重写的方法
	public void myMethod() {
		method3(); //调自己的
		super.method3(); //调父类的
		//调接口中的默认方法
		CompareA.super.method3();
		CompareB.super.method3();
	}
	
	
	
}
```





### 内部类（用的少）

> 一种类成员

```java
package cn.xpshuai.java5;
/*
 * 类的内部成员之五: 内部类
 * 1.Java中允许将一个类A声明在另一个类B中，则类A就是内部类，类B称为外部类
 * 2.内部类分类： 成员内部类(静态、非静态) vs	局部内部类(方法内、代码块内、构造器内)
 * 
 * 3.成员内部类：
 * 		一方面，作为外部类的成员：
 * 			>调用外部类的结构
 * 			>可以被static修饰
 * 			>可以被权限修饰符修饰
 * 
 * 
 * 		另一方面，作为一个类：
 * 			>类内可以定义属性、方法、构造器
 * 			>可以被final修饰，表示此类不能被继承
 * 			>可被abstract修饰，表示不能被实例化
 * 
 * 4.1 如何实例化成员内部类对象？
 * 
 * 4.2 如何在成员内部类中区分调用外部类的结构？
 * 		
 * 4.3 开发中，局部内部类的使用
 * 		
 * 
 * 
 * 
 */
public class InnerClassTest {
	public static void main(String[] args) {
		//创建Food实例(静态的成员内部类)
		People.Food food = new People.Food();
		food.show();
		
		//创建Food2实例(非静态的成员内部类)
//		People.Food2 food2 = new People.Food2();
		People people = new People();
		People.Food2 food2 = people.new Food2("大白菜");
		food2.show();
	}
}


class People{
	String name;
	int age;
	public void eat() {
		System.out.println("people eat");
	}
	
	// 静态成员内部类
	static class Food{ // abstract 
		String name;
		int age;
		public void show() {
			System.out.println("food1");
		}
//		eat();
	}
	// 非静态成员内部类
	class Food2{
		String name;

		public Food2(String name) {
			this.name = name;
		}
		
		public void show() {
			System.out.println(name + "food22");
		}
//		People.this.eat();  //调用外部类属性(非静态)
		
		public void display(String name) {
			System.out.println(name);  // 形参
			System.out.println(this.name);  //内部类的属性
			System.out.println(People.this.name); //外部类的属性
			
			
			
		}
		
		
	}
	
	
	
	public void method() {
		//局部内部类
		class AA{
			
		}
	} 
	
	{
		//局部内部类
		class BB{
			
		}
		
	}
	
	// 构造器
	public People() {
		//局部内部类
		class CC{
			
		}
	}
	
	//返回一个实现了Comparable接口的类的对象
	public Comparable getComparable() {
		//创建一个实现了Comparable接口的类：局部内部类
		class MyComparable implements Comparable{
			@Override
			public int compareTo(Object obj) {
				return 0;
			}
		}
		return new MyComparable();
	}	
}
```

局部内部类使用注意点：

```java
public class Test{
    	public void method() {
		//局部内部类的方法中，如果调用局部内部类所声明的方法中的局部变量的话，
		//要求此局部变量声明为final
		//局部变量: jdk7之前需要加final，jdk8后可省略，但默认也是的
		int num = 11;
		
		//局部内部类
		class AA{
			public void show() {
				//只能用，不能修改局部变量，其实是一个副本
//				num = 22;  //报错
				System.out.println(num);
			}
		}
	} 
    
    
}
```



> 内容逻辑有些混乱，仅为个人学习笔记......





---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8B%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E5%88%9D%E8%AF%86/  

